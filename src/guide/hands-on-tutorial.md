# チュートリアル: 手動ワークフロー

*コーディングエージェントを使用せず、すべてのコマンドをご自身で入力したい開発者向け。*

このチュートリアルでは、コーディングエージェントを使用せず、すべてのコマンドをご自身で入力して ADK エージェントを構築、テスト、評価する手順を解説します。

!!! tip
    コーディングエージェントに作業を任せたいですか？代わりに [チュートリアル: 最初のエージェントを構築する](quickstart-tutorial.md) を参照してください。

---

## 構築するもの

デフォルトのエージェントテンプレート (天気を調べたり時刻を伝えたりするアシスタント) から始めて、新しいペルソナとカスタムツールを追加してカスタマイズします。

## 前提条件

- Python 3.11+ および [uv](https://docs.astral.sh/uv/getting-started/installation/) のインストール
- 認証のセットアップ — [Gemini API キー](authentication.md#option-a-gemini-api-key-google-ai-studio) または [Google Cloud 認証情報](authentication.md#option-b-google-cloud-vertex-ai)

---

## 1. プロジェクトの作成

```bash
agents-cli create my-first-agent --prototype --yes
cd my-first-agent
agents-cli install
```

- `--prototype` は Terraform と CI/CD をスキップします — エージェントコード、テスト、評価セットのみが生成されます。
- `--yes` はデフォルト値 (ADK テンプレート、インメモリセッションストレージ) を自動承認します。
- `agents-cli install` は `uv sync` 経由ですべての Python 依存関係をインストールします。

---

## 2. プロジェクトの構造を確認

プロジェクトには以下が含まれています:

```
my-first-agent/
├── app/
│   ├── __init__.py       # アプリの登録
│   ├── agent.py          # エージェント定義 — ロジックを記述する場所
│   └── app_utils/        # テレメトリおよびユーティリティコード
├── tests/
│   ├── eval/
│   │   ├── datasets/
│   │   │   └── basic-dataset.json   # 評価用テストケース
│   │   └── eval_config.yaml         # メトリクス構成
│   ├── integration/
│   │   └── test_agent.py
│   └── unit/
│       └── test_dummy.py
├── pyproject.toml        # プロジェクト構成と依存関係
└── GEMINI.md             # コーディングエージェント用ガイドファイル
```

重要なファイルは `app/agent.py` です。開くと、2つのツール関数 (`get_weather`, `get_current_time`) とエージェント定義が含まれていることが分かります:

```python title="app/agent.py"
root_agent = Agent(
    name="root_agent",
    model=Gemini(model="gemini-3.6-flash"),
    instruction="You are a helpful AI assistant designed to provide accurate and useful information.",
    tools=[get_weather, get_current_time],
)
```

各ファイルの詳細な役割については [プロジェクト構造](project-structure.md) を参照してください。

---

## 3. エージェントをローカルで実行

ADK Web プレイグラウンドを起動します:

```bash
agents-cli playground
```

ブラウザで [http://localhost:8080](http://localhost:8080) を開きます。チャットインターフェースが表示されます。以下を送信してみてください:

> What's the weather in San Francisco?

エージェントは `get_weather` ツールを呼び出し、*「It's 60 degrees and foggy in San Francisco.」* のような返答を返します。

!!! tip
    プレイグラウンドはホットリロードに対応しています — `app/agent.py` の変更を保存すると即座に反映されます。

---

## 4. ターミナルからのテスト

ブラウザを使用せずにテストすることもできます:

```bash
agents-cli run "What's the weather in San Francisco?"
```

これにより、単一のプロンプトが送信され、エージェントのレスポンスが印刷されます。

---

## 5. エージェントのカスタマイズ

エージェントにパーソナリティを与えてみましょう。`app/agent.py` を開き、instruction (指示) を変更します:

```python title="app/agent.py"
root_agent = Agent(
    name="root_agent",
    model=Gemini(
        model="gemini-3.6-flash",
        retry_options=types.HttpRetryOptions(attempts=3),
    ),
    instruction="""You are a cheerful weather reporter who speaks in short, 
    punchy sentences. Always include a fun weather-related pun in your responses. 
    When asked about time, relate it back to weather somehow.""",
    tools=[get_weather, get_current_time],
)
```

ファイルを保存します。プレイグラウンドがまだ起動していれば自動的にリロードされます。同じ質問を再度試すと、返答のトーンが変わっていることが確認できます。

---

## 6. カスタムツールの追加

単語数をカウントするツールを追加してみましょう。`app/agent.py` 内の `root_agent` 定義の上部に以下の関数を追加します:

```python title="app/agent.py"
def count_words(text: str) -> str:
    """Count the number of words in the given text.

    Args:
        text: The text to count words in.

    Returns:
        A string with the word count.
    """
    word_count = len(text.split())
    return f"The text contains {word_count} words."
```

次に、エージェントの `tools` リストに登録します:

```python
    tools=[get_weather, get_current_time, count_words],
```

テストを実行します:

```bash
agents-cli run "How many words are in: The quick brown fox jumps over the lazy dog"
```

エージェントは `count_words` を呼び出し、単語数を返答します。

!!! tip
    ADK のツールは普通の Python 関数です。**docstring** が LLM に表示される説明文になるため、モデルがいつどのようにツールを使うべきかを明確に記述してください。

ツールの追加に関する詳細は [ADK Tools ドキュメント](https://google.github.io/adk-docs/tools/) を参照してください。

---

## 7. 評価の実行

評価により、エージェントが正しく動作することを確認できます。プロジェクトの `tests/eval/datasets/basic-dataset.json` にはデフォルトのデータセットが用意されています:

```json title="tests/eval/datasets/basic-dataset.json"
{
  "eval_cases": [
    {
      "eval_case_id": "greeting",
      "prompt": {
        "role": "user",
        "parts": [{"text": "Hello, what can you help me with?"}]
      }
    }
  ]
}
```

各評価ケースはユーザーメッセージを定義します。評価システムはエージェントにメッセージを送信し、`eval_config.yaml` で指定されたメトリクスを使用してレスポンスを採点します。

実行します:

```bash
agents-cli eval generate
agents-cli eval grade
```

出力には、構成されたメトリクスに対する各評価ケースのスコアが表示されます。

テストケースの作成、メトリクスの追加、評価修正ループ、その他の評価機能 (`eval dataset synthesize`、`eval compare`、`eval analyze`、`eval metric list`、`eval optimize`) を含む完全な評価ワークフローについては [評価ガイド](evaluation.md) を参照してください。

---

## 8. Google Cloud へのデプロイ

エージェントが評価を通過したら、デプロイします。まず、デプロイターゲットを追加します (プロトタイププロジェクトには含まれていません):

```bash
agents-cli scaffold enhance --deployment-target cloud_run
```

Google Cloud プロジェクトを設定してデプロイします:

```bash
gcloud config set project YOUR_DEV_PROJECT_ID
agents-cli deploy
```

実行状態を確認します:

```bash
agents-cli deploy --status
```

!!! note
    デプロイには [Google Cloud 認証情報](authentication.md#option-b-google-cloud-vertex-ai) が必要です。Agent Runtime、GKE、その他のオプションについては [デプロイガイド](deployment.md) を参照してください。

---

## 9. エージェントの監視 (可観測性)

Cloud Trace はデフォルトで有効になっています — 構成は不要です。エージェントにリクエストを数回送信したのち、Google Cloud Console で [Trace explorer](https://console.cloud.google.com/traces) を開きます。各 LLM 呼び出しやツール実行のスパンとレイテンシの内訳が表示されます。

### コンテンツログの表示

本番環境でエージェントが処理する実際のプロンプトとレスポンスを検証するには、可観測性インフラストラクチャをプロビジョニングします:

```bash
agents-cli infra single-project --project YOUR_DEV_PROJECT_ID
```

これにより Terraform が実行され、専用のサービスアカウント、GCS バケット、BigQuery データセットが作成され、デプロイ済みサービスがこれらを使用するように更新されます。

検証手順、完全なコンテンツのキャプチャ、および BigQuery Agent Analytics については [可観測性ガイド](observability/index.md) を参照してください。

---

## 実施した作業のまとめ

| ステップ | 概要 |
|------|--------------|
| `agents-cli create --prototype --yes` | エージェントコード、テスト、評価セットを含むプロジェクトを作成 |
| `agents-cli playground` | 対話型テスト用の ADK プレイグラウンドを起動 |
| `agents-cli run "..."` | ターミナルからエージェントをテスト |
| `agent.py` の編集 | ペルソナをカスタマイズし、ツールを追加 |
| `agents-cli eval generate` に続く `agents-cli eval grade` | 構造化評価によりエージェントの挙動を検証 |
| `agents-cli deploy` | エージェントを Google Cloud にデプロイ |
| Trace explorer + コンテンツログ | トレーシングを確認し、プロンプト/レスポンスログをセットアップ |

---

## 次のステップ

- [ADK カスタムツール](https://google.github.io/adk-docs/tools/) — より多くのツールパターンと応用的な使い方
- [評価ガイド](evaluation.md) — より優れた評価の記述とメトリクスの理解
- [デプロイガイド](deployment.md) — Agent Runtime、GKE、シークレット、および CI/CD
- [可観測性ガイド](observability/index.md) — BigQuery Agent Analytics、サードパーティ統合
