# 評価データセットの移行

[Gemini Enterprise Agent Platform GenAI Eval SDK](https://docs.cloud.google.com/gemini-enterprise-agent-platform/optimize/evaluation/agent-evaluation) を中心に評価機能が再構築される前に `agents-cli` の使用を開始した場合、以前の ADK `EvalSet` スキーマを使用した評価ファイルが `tests/eval/evalsets/` 配下にある可能性があります。それらのファイルは `agents-cli eval generate` や関連コマンドによって読み込まれなくなりました。このページでは、それらを新しいフォーマットに変換する手順を説明します。

プロジェクトに `tests/eval/evalsets/` ディレクトリがない場合、必要な作業はありません。

---

## 自動移行

`agents-cli scaffold upgrade` はレガシーな `*.evalset.json` ファイルを検出し、それらを新しいフォーマットに自動的に変換します。変換は以下のルールに従います: 新しいファイルを `tests/eval/datasets/` 配下に書き込み、すでに存在する送信先はスキップし、削除前に検証できるようにレガシーディレクトリをそのまま残します。`eval generate` は次回の実行時にライブラリエージェントから `agent_data.agents` を生成するため、マイグレーターはスタブを書き込みません。

手動で変換を行いたい場合は、このページの以降のセクションでスキーマの変更について説明します。

---

## 変更された理由

新しい評価機能 (`eval generate`、`eval grade`、`eval dataset synthesize`、`eval compare`、`eval analyze`、`eval metric list`、`eval optimize`) は、Gemini Enterprise Agent Platform GenAI Eval SDK の `EvaluationDataset` / `EvalCase` 型に基づいています。プラットフォーム自体のスキーマを採用することで、`agents-cli` が 2 つの異なるデータ形状間を橋渡しすることなく、Agent Platform のより広範な評価機能セット（組み込みおよびカスタムメトリクス、LLM-as-judge 採点、データセット合成、回帰比較、失敗モード分析、およびプロンプト最適化）を活用できるようになります。

---

## 変更点の概要

| | 旧 (ADK `EvalSet`) | 新 (Agent Platform `EvaluationDataset`) |
|---|---|---|
| ディレクトリ | `tests/eval/evalsets/` | `tests/eval/datasets/` |
| ファイル名 | `*.evalset.json` | `*-dataset.json` |
| デフォルトファイル | `basic.evalset.json` | `basic-dataset.json` |
| スキーマソース | `google.adk.evaluation` | `vertexai._genai.types.EvaluationDataset` |

`agents-cli eval generate` はデフォルトで `tests/eval/datasets/basic-dataset.json` を探します。別のファイルを指定するには `--dataset PATH` を使用します。

---

## スキーマの変更

> **2つの有効な入力形状。** 各新しいフォーマットの評価ケースは、以下の **いずれか1つ** を提供する必要があります:
>
> - **形状 A — 単一プロンプトケース:** トップレベルの `prompt` フィールド（単一のユーザーメッセージ）。ケースが単一ターンのユーザー質問である場合に使用します。
> - **形状 B — 継続会話ケース（"N+1" パターン）:** ターンがユーザーメッセージで終わる `agent_data` ブロック。`agents-cli eval generate` はその後に次のエージェントのレスポンスを追加します。
>
> 旧 `EvalSet` スキーマの *単一ターン* ケースは **形状 A** にマッピングされます。旧 *複数ターン* ケースは **形状 B** にマッピングされます: 記録された以前のターンは `agent_data.turns` となり、エージェントに応答させたいユーザーメッセージで終わります。以下のセクションで両方を示します。

### エンベロープ

外側のラッパーはよりシンプルになりました。`eval_set_id`、`name`、および `description` は廃止され、`eval_cases` のみが残ります。

**変更前:**
```json
{
  "eval_set_id": "basic_eval",
  "name": "Basic Agent Evaluation",
  "description": "Sample evaluation set for testing core agent functionality.",
  "eval_cases": [ ... ]
}
```

**変更後:**
```json
{
  "eval_cases": [ ... ]
}
```

### 単一ターンケース

ケースごとに 3 つの変更点があります:

- `eval_id` → `eval_case_id`。
- 最初のターンの `conversation[0].user_content` がトップレベルの `prompt` に引き上げられます。
- `session_input` は削除されます。エージェント状態の初期化は評価データで宣言するのではなく、エージェントコード (`app/agent.py`) 内に移動します。

**変更前:**
```json
{
  "eval_id": "greeting",
  "conversation": [
    {
      "user_content": {
        "parts": [{"text": "Hello, what can you help me with?"}]
      }
    }
  ],
  "session_input": {
    "app_name": "app",
    "user_id": "eval_user",
    "state": {}
  }
}
```

**変更後:**
```json
{
  "eval_case_id": "greeting",
  "prompt": {
    "role": "user",
    "parts": [{"text": "Hello, what can you help me with?"}]
  }
}
```

プロンプトに `"role": "user"` が追加されていることに注意してください — これは Agent Platform の `Content` 型で必須です。

### 複数ターンケース (形状 B)

旧スキーマでは、複数ターンの会話は `conversation` 配下のターンのリストでした。新スキーマでは、それらは **形状 B** にマッピングされます: 以前のターンは `agent_data.turns` 配下に配置され、その履歴の最後のユーザーメッセージに対して `eval generate` が応答します（個別のトップレベル `prompt` はありません）。

`agent_data.turns[].events` 配下の各エントリは、`author` (`"user"` または `agent_data.agents` で宣言されたエージェント ID のいずれか) と `content` (ユーザーターンの場合は `role: "user"`、エージェントターンの場合は `role: "model"`) を持つイベントです。

**変更前 (2ターンの会話):**
```json
{
  "eval_id": "follow_up",
  "conversation": [
    {
      "user_content": {
        "parts": [{"text": "Book a flight to Paris."}]
      },
      "final_response": {
        "parts": [{"text": "What dates are you flying?"}]
      }
    },
    {
      "user_content": {
        "parts": [{"text": "Next Monday, returning Friday."}]
      }
    }
  ]
}
```

**変更後 (形状 B):**
```json
{
  "eval_case_id": "follow_up",
  "agent_data": {
    "agents": {
      "flight_booker": {
        "agent_id": "flight_booker",
        "agent_type": "llm_agent",
        "description": "Books flights and answers itinerary questions.",
        "instruction": "Help the user book flights. Ask clarifying questions about dates, origin, and passenger count before calling any booking tool.",
        "tools": [
          {
            "function_declarations": [
              {"name": "search_flights", "description": "Search available flights."},
              {"name": "book_flight",    "description": "Book a flight by ID."}
            ]
          }
        ],
        "sub_agents": []
      }
    },
    "turns": [
      {
        "turn_index": 0,
        "events": [
          {
            "author": "user",
            "content": {
              "role": "user",
              "parts": [{"text": "Book a flight to Paris."}]
            }
          },
          {
            "author": "flight_booker",
            "content": {
              "role": "model",
              "parts": [{"text": "What dates are you flying?"}]
            }
          },
          {
            "author": "user",
            "content": {
              "role": "user",
              "parts": [{"text": "Next Monday, returning Friday."}]
            }
          }
        ]
      }
    ]
  }
}
```

> **`agent_data.agents` について。** `agents` マップは、評価対象のエージェントシステムのトポロジーを宣言します: エージェント ID でキー指定され、各エントリはそのエージェントの設定 — `agent_type`、`description`、`instruction`、`tools` (`google.genai.types.Tool` で使用されるのと同じ形状の、エージェントが呼び出す可能性のある関数宣言)、および `sub_agents` を保持します。各イベントの `author` は `"user"` またはこのマップに存在するエージェント ID のいずれかであり、採点中にマルチエージェントシステムが適切なサブエージェントにレスポンスやツール呼び出しを帰属させる方法となります。`tools` ブロックを使用すると、採点者はエージェントが適切なパラメータで適切なツールを選択したかを確認できるため、エージェントに呼び出し可能なツールがある場合は必ず含めてください。単一エージェントプロジェクトの場合は上記のように 1 つのエントリのみを宣言できます。マルチエージェントシステムの場合は各エージェントをリストし、`sub_agents` を使用してトポロジーを表現します。ここに示されている値は例示用です — ご自身のプロジェクトに合わせて調整してください。

`eval generate` はこの履歴に対してエージェントを実行し、その返答を次のエージェントイベントとして追加し、`eval grade` に渡す準備が整った追加済みトレースを生成します。

旧ケースの **最後** のターン（採点対象となるターン）に模範解答を表現するために `final_response` が設定されていた場合、それは異なる概念です — `agent_data.turns` に混ぜるのではなく、トップレベルの `reference` フィールドに配置してください。過去の実際のレスポンスはターン履歴に入り、最終ユーザーメッセージに対する目標解答は `reference` に入ります。

---

## ステップバイステップの変換

単一のファイル `tests/eval/evalsets/basic.evalset.json` の場合:

1. 新しいディレクトリを作成: `mkdir -p tests/eval/datasets`
2. ファイルをコピー: `cp tests/eval/evalsets/basic.evalset.json tests/eval/datasets/basic-dataset.json`
3. エディタで `tests/eval/datasets/basic-dataset.json` を開く
4. トップレベルから `eval_set_id`、`name`、および `description` を削除
5. `eval_cases` 配下の各エントリについて、`eval_id` を `eval_case_id` に変更し、**形状 A** または **形状 B** を選択:
    - **単一ターン (形状 A):** 唯一のターンの `user_content` をトップレベルの `prompt` に移動し、`"role": "user"` を追加します。`conversation` 配列と `session_input` ブロックを削除します。
    - **複数ターン (形状 B):** `agent_data.agents`（エージェント ID からその `AgentConfig` へのマップ）でエージェントトポロジーを宣言し、最後のエントリがエージェントに応答させたいユーザーメッセージである `agent_data.turns[0].events` リストを構築します。以前の各ターンの `user_content` を `author: "user"` (`role: "user"`) を持つイベントに変換し、記録されたエージェントのレスポンスを、`author` が応答エージェントの ID (`role: "model"`) であるイベントに変換します。`conversation` 配列と `session_input` ブロックを削除します。形状 B のトップレベル `prompt` は設定**しない**でください。
6. 保存し、`agents-cli eval generate` で検証します — 自動的にファイルが見つかるはずです。

すべてが正常に動作したら、古い `tests/eval/evalsets/` ディレクトリを削除します。

### 複数ファイル

各 `*.evalset.json` について上記の手順を繰り返します。`tests/eval/datasets/` 内のファイル名は `*-dataset.json` の規約に従う必要があります（したがって `flight_booking.evalset.json` は `flight_booking-dataset.json` になります）。

---

## 検証

```bash
agents-cli eval generate
```

デフォルト名 (`basic-dataset.json`) を使用した場合、`eval generate` は自動的にそれを取得します。その他のファイル名の場合:

```bash
agents-cli eval generate --dataset tests/eval/datasets/your-file-dataset.json
```

正常に実行されると、`agents-cli eval grade` に渡すことができるデータの入ったトレースファイルが生成されます。
