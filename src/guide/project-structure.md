# プロジェクト構造

*生成されたエージェントプロジェクトのレイアウトを理解したい開発者向け。*

`agents-cli create my-agent --prototype --yes` を実行すると、すぐに実行可能なプロジェクトが生成されます。このページでは、各ファイルの役割を説明します。

---

## ディレクトリ構造

```
my-agent/
├── app/                          # エージェントコード
│   ├── __init__.py               # Appを登録（`app`をエクスポート）
│   ├── agent.py                  # エージェント定義 — 指示、モデル、ツール
│   ├── fast_api_app.py           # FastAPIサーバー — テレメトリ設定、フィードバック/A2Aルート
│   └── app_utils/                # ユーティリティ
│       ├── services.py           # 共有セッションおよびアーティファクトサービス
│       ├── a2a.py                # A2Aルート配線
│       └── typing.py             # リクエスト/レスポンス Pydantic モデル
│
├── tests/
│   ├── eval/                     # 評価テストケース
│   │   ├── datasets/
│   │   │   └── basic-dataset.json    # デフォルト評価ケース
│   │   └── eval_config.yaml          # 評価メトリクス設定
│   ├── integration/
│   │   └── test_agent.py         # 統合テスト（エージェントをエンドツーエンドで実行）
│   └── unit/
│       └── test_dummy.py         # ユニットテストのプレースホルダー
│
├── pyproject.toml                # プロジェクト設定および依存関係
├── agents-cli-manifest.yaml      # agents-cli の設定
├── GEMINI.md                     # コーディングエージェント向けガイドファイル
├── Makefile                      # ショートカットコマンド（make dev、make eval など）
├── .env                          # 環境変数（プロジェクト ID、ロケーション）
└── uv.lock                       # ロックされた依存関係バージョン
```

---

## 主要ファイル

### `app/agent.py`

ここにはエージェントの処理が定義されます。デフォルトテンプレートは以下のようになっています：

```python title="app/agent.py"
from google.adk.agents import Agent
from google.adk.apps import App
from google.adk.models import Gemini
from google.genai import types

MODEL = "gemini-3.6-flash"


def get_weather(query: str) -> str:
    """Web検索をシミュレートします。天気情報の取得に使用します。"""
    if "sf" in query.lower() or "san francisco" in query.lower():
        return "It's 60 degrees and foggy."
    return "It's 90 degrees and sunny."


def get_current_time(query: str) -> str:
    """市の現在時刻の取得をシミュレートします。"""
    # ... 実装
    return f"The current time is ..."


root_agent = Agent(
    name="root_agent",
    model=Gemini(
        model=MODEL,
        retry_options=types.HttpRetryOptions(attempts=3),
    ),
    instruction="You are a helpful AI assistant.",
    tools=[get_weather, get_current_time],
)

app = App(
    root_agent=root_agent,
    name="app",  # エージェントディレクトリ名と一致する必要があります
)
```

4つの主要なコンポーネント:

1. **ツール関数** — ドキュメント文字列（docstring）を持つ通常の Python 関数。ドキュメント文字列は、LLM にツールをいつ使用するかを伝えます。
2. **`Agent`** — モデル、指示（システムプロンプト）、ツールを統合します。
3. **`App`** — 配信（serving）用にエージェントをラップします。`name` はディレクトリ名（`app`）と一致させる必要があります。
4. **モデル** — デフォルトは `gemini-3.6-flash` です。`agent.py` の上部にある `MODEL` 定数を変更して調整できます。

### `pyproject.toml`

Python プロジェクトのメタデータと依存関係が含まれています:

```toml title="pyproject.toml"
[project]
name = "my-agent"
version = "0.0.1"
requires-python = ">=3.11"
dependencies = [
    "google-adk[gcp]>=2.0.0,<3.0.0",
    # ... その他の依存関係
]
```

### `agents-cli-manifest.yaml`

agents-cli プロジェクトのメタデータと設定が含まれています:

```yaml title="agents-cli-manifest.yaml"
name: my-agent
agent_directory: app
create_params:
  deployment_target: none
  session_type: in_memory
```

- **`agent_directory`** — `agents-cli` コマンドにエージェントコードの場所を伝えます。
- **`create_params`** — プロジェクトがどのように作成されたかを記録します。設定を保持したまま `agents-cli scaffold upgrade` を実行する際に使用されます。

### `tests/eval/datasets/basic-dataset.json`

デフォルトの評価ケースです。各ケースはユーザーメッセージとそれを実行するためのセッションコンテキストを定義します。完全なスキーマについては [評価ガイド](evaluation.md) を参照してください。

### `GEMINI.md`

コーディングエージェント（Antigravity CLI、Claude Code など）が自動的に読み込むガイドファイルです。ADK パターン、コーディング規約、ワークフローガイドラインなど、プロジェクト固有の指示が含まれています。コーディングエージェントによるプロジェクトの取り扱いをカスタマイズしたい場合を除き、このファイルを直接読んだり編集したりする必要はありません。

### `.env`

ローカル開発用の環境変数です:

```bash title=".env"
GOOGLE_CLOUD_PROJECT=your-project-id
GOOGLE_CLOUD_LOCATION=us-east1
```

これらは実行時にエージェントによって読み込まれます。Google Cloud プロジェクトに合わせて設定するか、Gemini API キーを使用する場合は空のままにしてください。

---

## デプロイメントインフラストラクチャを含む場合

デプロイ先ターゲットを指定してプロジェクトを作成した場合（または `agents-cli scaffold enhance` で追加した場合）、追加のディレクトリが表示されます:

```
my-agent/
├── deployment/
│   └── terraform/
│       ├── dev/              # 開発環境用 Terraform
│       ├── staging/          # ステージング環境用 Terraform
│       ├── prod/             # 本番環境用 Terraform
│       └── variables.tf      # 共有変数
│
├── .github/                  # GitHub Actions CI/CD（選択した場合）
│   └── workflows/
│       ├── pr_checks.yaml
│       ├── staging.yaml
│       └── deploy-to-prod.yaml
│
└── .cloudbuild/              # Cloud Build CI/CD（選択した場合）
    ├── pr_checks.yaml
    ├── staging.yaml
    └── deploy-to-prod.yaml
```

### 後からインフラストラクチャを追加する

プロトタイプから開始し、必要に応じてインフラストラクチャを追加できます:

```bash
# Cloud Run デプロイメントの追加
agents-cli scaffold enhance --deployment-target cloud_run

# 変更を適用せずにプレビュー
agents-cli scaffold enhance --deployment-target cloud_run --dry-run
```
