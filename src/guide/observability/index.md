# 可観測性

すべての `agents-cli` プロジェクトには、**Cloud Trace** にトレースを自動的にエクスポートする OpenTelemetry 計装（instrumentation）が備わっています。これにより、以下が提供されます:

- **分散トレース** — LLM 呼び出しやツールの実行を通じてリクエストの流れを追跡します。
- **レイテンシ分析** — スパンの期間を分析してパフォーマンスのボトルネックを特定します。
- **エラーの可視化** — トレースがエラーをキャプチャし、障害が発生した場所を特定するのに役立ちます。
- **設定不要** — すべての環境で開梱後すぐに機能します。

ADK ベースのエージェントの場合、**プロンプト・レスポンス ログ** はモデルのインタラクション全体（プロンプト、レスポンス、トークン）をキャプチャし、**GCS** (JSONL) および **BigQuery** の `completions` テーブルにアップロードします。これはログバケットが設定されている場合（`LOGS_BUCKET_NAME` + `OTEL_INSTRUMENTATION_GENAI_*` アップロード変数）に有効になり、Terraform でプロビジョニングされたデプロイメントではデフォルトで有効になります。

> **2つの独立した層。** プロンプト・レスポンス ログ（GCS/BigQuery completions）は **完全なコンテンツ** をキャプチャします。コンテンツが **Cloud Trace スパン / Cloud Logging イベント** に *表示されるか* どうかは、`OTEL_INSTRUMENTATION_GENAI_CAPTURE_MESSAGE_CONTENT`（デフォルトは `NO_CONTENT` — コンテンツはトレース/イベントから **除外** されます）および `ADK_CAPTURE_MESSAGE_CONTENT_IN_SPANS=false` によって個別に入力・制御されます。したがって、デフォルトでは **GCS/BigQuery には完全なコンテンツが保存され、トレースにはコンテンツが含まれません。**

### 環境ごとのロギング動作

| 環境 | Cloud Trace スパン | プロンプト・レスポンス ログ (GCS/BigQuery) |
|---|---|---|
| **ローカル** (`agents-cli playground`) | 有効、コンテンツなし | オフ (`LOGS_BUCKET_NAME` なし) |
| **デプロイ済み** (Terraform プロビジョニング) | 有効、コンテンツなし | **オン — 完全なプロンプト/レスポンス** |
| **デプロイ済み** (単なる `agents-cli deploy`、バケットなし) | 有効、コンテンツなし | オフ (`LOGS_BUCKET_NAME` なし) |

---

## Cloud Trace

デフォルトの可観測性手法です。セットアップと使用方法については [Cloud Trace](cloud-trace.md) を参照してください。

---

## BigQuery Agent Analytics

高度なアナリティクス向け — 会話全体のパターンクエリ、トークン使用量ダッシュボード、本番トラフィックに対する LLM-as-a-judge スコアリングなど。プロジェクト作成時に `--bq-analytics` フラグを使用してオプトインします。

詳細については [BigQuery Agent Analytics](bq-agent-analytics.md) を参照してください。
