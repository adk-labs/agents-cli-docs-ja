# 認証

agents-cli は複数のツールの基礎として機能し、それぞれが独自の認証要件を持っています。このページでは、何をなぜ認証しているのかを明確に理解できるように、3つの異なるレベルに分解して説明します。

---

## レベル 1: コーディングエージェントの認証

お使いのコーディングエージェント (Antigravity CLI、Claude Code、Codex など) は、機能するために固有の認証を必要とします。**agents-cli はこれを管理しません** — 各エージェントが独自の認証情報を処理します。

| コーディングエージェント | 認証方法 |
|-------------|---------------------|
| [Antigravity CLI](https://antigravity.google/) | Google アカウント |
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | Anthropic アカウントまたは API キー |
| [Codex](https://github.com/openai/codex) | OpenAI API キー |

セットアップ手順については、お使いのコーディングエージェントのドキュメントを参照してください。これは agents-cli とは独立しています。

---

## レベル 2: モデルの認証

あなたが*構築している*エージェントは、レスポンスを生成するために LLM を呼び出します。これにはコーディングエージェントとは別の認証情報が必要です。

ADK は [複数のモデルプロバイダ](https://adk.dev/agents/models/) (Gemini、Claude、LiteLLM、Ollama など) をサポートしています。[Gemini モデル](https://adk.dev/agents/models/google-gemini/) における最も一般的な 2つのセットアップは以下の通りです。

### オプション A: Gemini API キー (Google AI Studio)

Google Cloud プロジェクトは不要です。

1. [AI Studio](https://aistudio.google.com/apikey) にアクセスし、API キーを作成します。
2. 環境変数を出力/設定します:

    ```bash
    # スキャフォールドされた Python プロジェクトには .env が含まれています。AI Studio を使用するには、それを編集します:
    #   GOOGLE_* の行をコメントアウトし、GEMINI_API_KEY のコメントを解除します (GOOGLE_API_KEY も受け付けられます)。
    GEMINI_API_KEY="your-key-here"
    ```

3. ファイルを保存します — `agents-cli` の開発コマンドを実行する際、`.env` が自動的に読み込まれます。

!!! note
    API キーはローカル開発コマンド (`dev`、`run`、`eval`) をサポートします。Google Cloud へのデプロイにはレベル 3 の認証が必要です。

### オプション B: Google Cloud (Vertex AI)

Vertex AI モデル、エンタープライズ機能、およびデプロイに必要です。

```bash
agents-cli login -i
# または直接: gcloud auth application-default login
```

これにより OAuth のためにブラウザが開き、アプリケーションデフォルト認証情報 (ADC) がセットアップされます。

プロジェクトとロケーションを設定します:

```bash
gcloud config set project YOUR_PROJECT_ID
export GOOGLE_CLOUD_LOCATION="us-east1"
export GOOGLE_GENAI_USE_VERTEXAI=TRUE
```

---

## レベル 3: デプロイの認証

レベル 2 のオプション B (Vertex AI) をセットアップした場合、デプロイ用の認証は完了しています — 同じ ADC 認証情報を使用します。モデルへのアクセスに加えて、ADC は以下を解放します:

- `agents-cli deploy` — Agent Runtime、Cloud Run、または GKE へのデプロイ
- `agents-cli infra single-project` / `agents-cli infra cicd` — Terraform によるインフラストラクチャおよび CI/CD のプロビジョニング

デプロイには、請求処理が有効で適切な IAM 権限を持つ Google Cloud プロジェクトが必要です (ターゲットにより異なります)。

---

## ステータスの確認

```bash
agents-cli login --status
```

アクティブな認証方法と現在のプロジェクトが表示されます。
