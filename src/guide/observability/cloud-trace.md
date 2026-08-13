# Cloud Trace

*エージェントをデプロイし、トレースが正しく動作することを確認してテレメトリデータを検証したい開発者向け。*

![可観測性モニタリングフロー](../../assets/observability.png)

Cloud Trace はすべての `agents-cli` プロジェクトでデフォルトで有効になっています。このガイドでは、機能の確認方法とテレメトリデータのクエリ方法を示します。

---

## デプロイメントでのトレースの検証

開発環境にデプロイした後、テレメトリデータが流れ込んでいるか確認します:

### 1. デプロイとテストトラフィックの生成

```bash
gcloud config set project YOUR_DEV_PROJECT_ID
agents-cli deploy
```

エージェントにテストリクエストをいくつか送信します。

### 2. トレースの表示

Google Cloud コンソールを開き、**Trace > トレース エクスプローラ** に移動します。リクエストごとのトレースが表示され、LLM 呼び出しやツールの実行を示すスパンが表示されます。

### 3. プロンプト・レスポンス ログの検証 (任意)

GCS および BigQuery へのプロンプト・レスポンス ログ記録は Terraform (`agents-cli infra single-project` または `agents-cli infra cicd`) によってプロビジョニングされ、ログバケットとデータセットが作成されて `LOGS_BUCKET_NAME` が設定されます — そこで自動的に有効になります。単なる `agents-cli deploy` ではこれらのリソースは作成**されない**ため、以下の検証は Terraform でプロビジョニングされたデプロイメントにのみ適用されます。

```bash
PROJECT_ID="your-dev-project-id"
PROJECT_NAME="your-project-name"

# GCS 内のテレメトリファイルを確認
gsutil ls gs://${PROJECT_ID}-${PROJECT_NAME}-logs/completions/

# BigQuery でテレメトリをクエリ
bq query --use_legacy_sql=false \
  "SELECT * FROM \`${PROJECT_ID}.${PROJECT_NAME}_telemetry.completions\` LIMIT 10"
```

データが表示されない場合:

1. サービスアカウントに `storage.objectCreator` ロールが付与されているか確認します。
2. デプロイ環境の環境変数に `LOGS_BUCKET_NAME` が設定されているか確認します。
3. テレメトリセットアップの警告がないか、Cloud Logging のアプリケーションログを確認します。

---

## ローカルでプロンプト・レスポンス ログを有効にする

デフォルトでは、`agents-cli playground` はプロンプト・レスポンス ログ**なし**で実行されます。テレメトリは宣言型です（ランタイムの `setup_telemetry()` は存在しません）。そのためローカルで completions ログを有効にするには、Terraform がデプロイ済みエージェントに設定するのと同じ環境変数を設定します（ADK エージェントのみ）:

```bash
export LOGS_BUCKET_NAME="your-dev-project-id-your-project-name-logs"   # バケット名のみ、gs:// なし
export OTEL_INSTRUMENTATION_GENAI_CAPTURE_MESSAGE_CONTENT="NO_CONTENT"  # ログにフルコンテンツを含める場合は EVENT_ONLY
export OTEL_INSTRUMENTATION_GENAI_COMPLETION_HOOK="upload"
export OTEL_INSTRUMENTATION_GENAI_UPLOAD_BASE_PATH="gs://your-dev-project-id-your-project-name-logs/completions"
export OTEL_INSTRUMENTATION_GENAI_UPLOAD_FORMAT="jsonl"
export OTEL_SEMCONV_STABILITY_OPT_IN="gen_ai_latest_experimental"
agents-cli playground
```

---

## デプロイメントでプロンプト・レスポンス ログを無効にする

デプロイ環境で無効にするには、`deployment/terraform/single-project/service.tf` を編集します:

```hcl
env {
  name  = "OTEL_INSTRUMENTATION_GENAI_CAPTURE_MESSAGE_CONTENT"
  value = "false"
}
```

次に適用します:

```bash
cd deployment/terraform/single-project
terraform apply -var-file=vars/env.tfvars
```

---

## 設定リファレンス

| 変数 | 値 | 目的 |
|---|---|---|
| `LOGS_BUCKET_NAME` | GCS バケット**名** (`gs://` なし) | プロンプト・レスポンス ログに必須。設定されていない場合、ロギングは無効になります。 |
| `OTEL_INSTRUMENTATION_GENAI_CAPTURE_MESSAGE_CONTENT` | `NO_CONTENT`, `EVENT_ONLY`, `SPAN_ONLY`, `SPAN_AND_EVENT` | **トレース/イベントのみ** のコンテンツを制御します (バケット設定時に常にフルコンテンツをキャプチャする GCS/BigQuery completions には影響しません)。実験的 semconv (`OTEL_SEMCONV_STABILITY_OPT_IN` を設定): `NO_CONTENT` = スパン/イベントにコンテンツなし (デフォルト); `EVENT_ONLY` = Cloud Logging イベントにコンテンツあり; `SPAN_*` = トレーススパンにコンテンツあり。**`true`/`false` は無効** — 拒否され `NO_CONTENT` にフォールバックします。 |
| `OTEL_INSTRUMENTATION_GENAI_COMPLETION_HOOK` | `upload` | コンプレッションレコードのアップロードを有効化 |
| `OTEL_INSTRUMENTATION_GENAI_UPLOAD_BASE_PATH` | `gs://<bucket>/completions` | コンプレッションレコードの送信先 |
| `OTEL_INSTRUMENTATION_GENAI_UPLOAD_FORMAT` | `jsonl` | アップロード形式 |
| `OTEL_SEMCONV_STABILITY_OPT_IN` | `gen_ai_latest_experimental` | GenAI completion/upload semconv に必要 |
| `ADK_CAPTURE_MESSAGE_CONTENT_IN_SPANS` | `false`, `true` | プロンプト/レスポンスのコンテンツをトレーススパンから除外 (`false`、デフォルトで設定) |
