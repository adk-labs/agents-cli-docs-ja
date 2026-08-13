# 評価ガイド

構造化された評価を実行して、エージェントが適切なツールを呼び出し、高品質なレスポンスを生成し、エッジケースを適切に処理できるか確認します。内部では、評価の採点に [Gemini Enterprise Agent Platform GenAI Eval SDK](https://docs.cloud.google.com/gemini-enterprise-agent-platform/optimize/evaluation/agent-evaluation) を使用しています。

!!! note "以前の agents-cli からアップグレードしますか？"
    以前のバージョンによる `tests/eval/evalsets/*.evalset.json` ファイルがプロジェクト内に残っている場合は、新しいフォーマットについて [評価データセットの移行](../reference/eval-dataset-migration.md) を参照してください。

---

## 最初の評価を実行する

プロジェクトには、`tests/eval/datasets/basic-dataset.json` のデフォルトデータセットと `tests/eval/eval_config.yaml` のメトリクス設定が含まれています。以下を実行してください:

```bash
agents-cli eval generate
agents-cli eval grade
```

出力には、設定されたメトリクスに対する各評価ケースのスコアが表示されます。

```bash
# カスタムデータセットおよび異なるメトリクスで実行
agents-cli eval generate --dataset tests/eval/datasets/custom-dataset.json --output custom_traces/
agents-cli eval grade --metrics general_quality --traces custom_traces/

# ローカルでの実行ではなく、デプロイ済みエージェントを評価
agents-cli eval generate --url https://my-agent.run.app --app-name app
agents-cli eval grade
```

---

## 評価ケースの記述とメトリクスの選択

評価ケースのスキーマおよび利用可能なメトリクスの詳細については、[Gemini Enterprise Agent Platform 評価ドキュメント](https://docs.cloud.google.com/gemini-enterprise-agent-platform/optimize/evaluation/agent-evaluation) を参照してください。

### 利用可能なメトリクスリファレンス

エージェントの機能やタスクに応じて、豊富な組込メトリクスから選択できます。利用可能なメトリクスの完全なリストを表示するには、以下を実行します:

```bash
agents-cli eval metric list
```

#### 一般的なメトリクス一覧

よく使われる組み込みメトリクス ID の短いリファレンスです。説明付きの完全なセットについては `agents-cli eval metric list` を使用してください。

| メトリクス ID | 採点対象 |
|---|---|
| `general_quality` | 自動生成されたコンテンツベースの基準による全体的なレスポンス品質。非エージェント評価の推奨される開始点。 |
| `text_quality` | 言語的側面: 流暢さ、一貫性、文法。 |
| `instruction_following` | レスポンスが特定の制約や指示にどれだけ従っているか。 |
| `tool_use_quality` | ツールの選択、パラメータの精度、およびステップ順序の正確さ（単一ターン）。 |
| `multi_turn_tool_use_quality` | 複数ターンの会話におけるツール呼び出しの技術的およびセマンティックな正確さ。 |
| `multi_turn_trajectory_quality` | ターンにわたるシーケンシャルロジック、効率性、およびエラー回復の堅牢性。 |
| `multi_turn_task_success` | 複数ターンの会話全体でユーザーの目標が達成されたかどうか。 |
| `final_response_quality` | 最終レスポンスと中間ツール使用状況の包括的な評価。 |
| `final_response_reference_free` | 参照回答なしの最終レスポンス品質（カスタムルーブリックが必要）。 |
| `final_response_match` | エージェントの最終レスポンスを提供されたゴールデン参照回答と比較。 |
| `hallucination` | レスポンスをアトミックな主張に分割し、ツールから返されたコンテキストに対してそれぞれを検証。 |
| `grounding` | 提供されたコンテキストに対する事実性と一貫性。 |
| `safety` | セーフティポリシー（PII、ヘイトスピーチ、危険なコンテンツ、嫌がらせ、性的コンテンツ）への準拠。 |

### 評価設定 (`eval_config.yaml`)

`eval_config.yaml` ファイルは、実行するメトリクスを指定し、評価の採点に使用するカスタムメトリクスを定義します。

```yaml
metrics_to_run:
  - response_under_500_chars

custom_metrics:
  - name: response_under_500_chars
    custom_function: |
      def evaluate(instance: dict) -> dict:
          response = instance.get("response") or {}
          text = "".join(
              p.get("text", "") for p in (response.get("parts") or []) if p.get("text")
          )
          passed = len(text) <= 500
          return {
              "score": 1.0 if passed else 0.0,
              "explanation": f"Final response is {len(text)} chars (limit 500).",
          }
  - name: response_quality_rubric
    prompt_template: |
      Rate the agent's response 1-5 for helpfulness and accuracy.
      Prompt: {prompt}
      Final response: {response}
      Full trace (for tool-call and reasoning context): {agent_data}
      Return JSON: {"score": <1|2|3|4|5>, "explanation": "<reason>"}
    judge_model: gemini-3.6-flash
    judge_model_sampling_count: 3
```

各カスタムメトリクスは、**コード実行メトリクス** または **LLM-as-a-Judge メトリクス** (`LLMMetric`) スキーマのいずれかに準拠している必要があります:
- **コード実行メトリクス**: 評価のためにカスタム Python コードを実行する際に使用します。`name` と `custom_function` (`def evaluate(instance):` シグネチャを含む) が必要です。デフォルトでは、関数は **CLI プロセス内でローカルに** 実行されます — GCP プロジェクトやリージョンは不要ですが、ユーザー提供のコードは CLI の権限で実行されます。GCP プロジェクトおよびリージョンの設定が必要な Vertex AI のサンドボックス環境である `CodeExecutionMetric` (サーバー側) を選択するには、`"execution": "remote"` を追加します。
- **LLM-as-a-Judge メトリクス**: LLM ジャッジを使用してレスポンスを評価する際に使用します。`name` と `prompt_template` が必要です。オプション項目には `rubric_group_name`、`judge_model`（例: `gemini-3.6-flash`）、および `judge_model_sampling_count`（`1` 〜 `32`）が含まれます。

### 一般的なシナリオのクイックリファレンス

- **カスタム関数ツールを持つエージェント** — `tool_use_quality`（単一ターン用）または `multi_turn_tool_use_quality` + `multi_turn_trajectory_quality`（複数ターン用）を使用します。
- **RAG エージェント** — `grounding` + `hallucination` + `safety` を使用します。
- **会話型アシスタント** — `general_quality` または `multi_turn_general_quality` を使用します。
- **目標指向型エージェント** — `multi_turn_task_success` を使用します。

---

## 評価・修正のループ (Eval-Fix Loop)

評価は反復的なプロセスです。エージェントが安定して合格するまでに 5〜10 回以上のサイクルが必要です。

1. **最も重要な動作をカバーする 1〜2 個のコア評価ケースを記述する**
2. **実行**: `agents-cli eval generate` を実行し、続いて `agents-cli eval grade` を実行
3. **結果を確認する** — どのケースが失敗し、その理由は何か
4. **修正** — エージェントの指示、ツール、またはロジックを調整
5. **再実行**: `agents-cli eval generate` と `agents-cli eval grade` を実行
6. **拡張** — コアケースが合格したら、エッジケースや新しいシナリオを追加

---

## `generate` と `grade` を超えて

`generate` と `grade` は内部ループを形成しますが、評価機能には知っておくべきコマンドが他にもいくつかあります。それぞれ、評価セットアップが成熟するにつれて利用する個別のステップです。

### `agents-cli eval dataset synthesize`

ローカルの ADK エージェントを検査し、それに対する複数ターンの会話シナリオを生成することでデータセットを起動します — 入力ファイルは不要です。新しいエージェントの評価をコールドスタートする場合や、すべてのケースを手動で記述せずにカバレッジを拡大する場合に便利です。生成された各ケースには、開始ユーザーメッセージ、会話プラン、および LLM ベースのユーザーシミュレーターに対してシナリオを再生することで生成された完全なエージェントトレースが含まれます。

```bash
agents-cli eval dataset synthesize --count 10
```

`--instruction`（例: `"Scenarios where the user changes their mind"`）および `--environment-context`（例: `"Today is Monday. Flights to Paris are available."`）を使用して、生成される内容をコントロールします。出力は、編集、コミット、および `eval grade` に直接フィードバックできる通常の `*-dataset.json` ファイルです（トレースはすでに生成されているため、`eval generate` をスキップできます）。

### `agents-cli eval compare`

変更によって実際に改善されたかどうかを確認できるように、2 つの採点結果を並べて比較します。

```bash
agents-cli eval compare baseline_results.json candidate_results.json
```

一般的な使用法は、評価・修正ループ中に「修正前」の実行と「修正後」の実行を比較することです。

### `agents-cli eval analyze`

採点結果ファイルから失敗モードをテーマ別にクラスタリングし、個々のケースを目視するのではなく、*どのような問題が発生しているか* を視覚化できます。

```bash
agents-cli eval analyze --eval-result grade_results.json
```

### `agents-cli eval metric list`

SDK がサポートするすべての組み込みメトリクスと、それぞれの短い説明を出力します。上記の一般的なメトリクステーブル以外の利用可能な項目を知りたい場合の出発点となります。

### `agents-cli eval optimize`

評価が整備されたら、`eval optimize` を使用して、評価結果に基づいてエージェントのプロンプトを自動的に調整します。

```bash
agents-cli eval optimize
```

実行にはデータセットのサイズとメトリクスの複雑さに応じて数分から数時間かかるため、何度も繰り返し実行するものではありません。よりシンプルなアプローチ（プロンプトの手動書換、メトリクスの調整、失敗ケースの手動修正など）を一通り試した後に使用してください。

---

評価ケースのスキーマ、メトリクス、およびユーザーシミュレーションの詳細については、[Gemini Enterprise Agent Platform 評価ドキュメント](https://docs.cloud.google.com/gemini-enterprise-agent-platform/optimize/evaluation/agent-evaluation) を参照してください。
