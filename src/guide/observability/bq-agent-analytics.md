# BigQuery Agent Analytics プラグイン

*エージェントの動作、トークン使用量、および会話パターンに関して SQL ベースのアナリティクスを希望するチーム向け。*

## 概要

BigQuery Agent Analytics プラグインは、詳細なエージェントイベントを BigQuery に直接ログ記録することで、強化された可観測性を提供します。これにより、エージェントの動作、インタラクション、および経時的なパフォーマンスに関する豊富な SQL ベースの分析が可能になります。

これは **オプトイン** 機能であり、**ADK ベースのエージェント** でのみ利用可能です。

---

## 使用するタイミング

以下を行う必要がある場合に、このプラグインを有効にします:

*   **BigQuery の高度な LLM 機能を活用する** — `AI.Search`、`AI.Score`、`AI.Generate_text` を介して、エージェントのセマンティック分析（会話のセマンティックなグループ化、順位付け、エラーの特定、または LLM-as-judge を使用した評価）を行います。
*   **BigQuery の会話型アナリティクスを活用する** — 別の会話型エージェントを使用してエージェントを分析し、複雑な SQL クエリを手動で記述する必要をなくします。
*   **カスタムダッシュボードとレポートの作成** — エージェントのパフォーマンス、ツール使用状況、およびトークン消費量に関するレポートを作成します。
*   **構造化されたクエリ可能な履歴の保持** — 監査、ファインチューニング、または他のビジネスデータとの結合のために、エージェントイベントの履歴を保持します。

常に有効な [Cloud Trace テレメトリ](cloud-trace.md) と比較して、このプラグインはオフライン分析用に設計された構造化テーブル形式でより粒度の細かいデータを提供します。

---

## 前提条件

*   **ADK ベース** のテンプレート（例: `adk`）で生成されたエージェントプロジェクト。
*   `google-adk` バージョン `>=1.21.0`（プラグインを有効にすると自動的に追加されます）。
*   BigQuery API および BigQuery Storage API が有効になっている Google Cloud プロジェクト（通常は Terraform で処理されます）。

---

## プラグインの有効化

プロジェクト作成時に `--bq-analytics` フラグを使用します:

```bash
agents-cli create my-agent \
  -a adk \
  -d cloud_run \
  --bq-analytics
```

このフラグにより、`app/agent.py` にプラグインの初期化コードが含まれ、Terraform で環境変数が設定されます。

---

## 設定

プラグインは `app/agent.py` ファイル内で設定されます:

```python
from google.adk.plugins.bigquery_agent_analytics_plugin import (
    BigQueryAgentAnalyticsPlugin,
    BigQueryLoggerConfig,
)

bq_config = BigQueryLoggerConfig(
    enabled=True,
    gcs_bucket_name=os.environ.get("BQ_ANALYTICS_GCS_BUCKET"),
    connection_id=os.environ.get("BQ_ANALYTICS_CONNECTION_ID"),
    log_multi_modal_content=True,
    max_content_length=500 * 1024,
    table_id="agent_events",
)

bq_analytics_plugin = BigQueryAgentAnalyticsPlugin(
    project_id=os.environ.get("GOOGLE_CLOUD_PROJECT"),
    dataset_id=os.environ.get("BQ_ANALYTICS_DATASET_ID", "adk_agent_analytics"),
    table_id=bq_config.table_id,
    config=bq_config,
    location=os.environ.get("GOOGLE_CLOUD_LOCATION", "US"),
)

app = App(
    name="my-agent",
    root_agent=root_agent,
    plugins=[bq_analytics_plugin],
)
```

**主な `BigQueryLoggerConfig` オプション:**

*   **`enabled`**: プラグインを切り替えます。
*   **`gcs_bucket_name`** (任意): 大容量/バイナリコンテンツをオフロードするための GCS バケット。マルチモーダルデータでのみ必要です。
*   **`connection_id`** (任意): GCS アクセス用の BigQuery Connection ID。マルチモーダルデータでのみ必要です。
*   **`log_multi_modal_content`**: コンテンツパーツを処理して GCS にオフロードするかどうか。
*   **`max_content_length`**: テキストを GCS にオフロードする閾値。
*   **`table_id`**: BigQuery テーブル名（デフォルトは `agent_events`）。
*   **`event_allowlist`** / **`event_denylist`**: ログに記録するイベントタイプをフィルタリングします。
*   **`batch_size`**: 書き込み前にバッチ処理する行数。

---

## インフラストラクチャ

Terraform (`agents-cli infra single-project`) でデプロイされた場合:

*   **データセット:** `{project_name}_telemetry` という名前の BigQuery データセットが作成されます。
*   **GCS バケット** (任意): コンテンツオフロード用の `{project_id}-{project_name}-logs`。
*   **BigQuery 接続** (任意): BigQuery から GCS にアクセスするための `{project_name}-genai-telemetry`。
*   **テーブル:** 最初のエントリが発生した際に、プラグインによって `agent_events` テーブルが **自動作成** されます。

---

## クエリの例

必要に応じて `YOUR_PROJECT_ID` と `YOUR_AGENT_NAME` を置き換えてください。

**最近のイベント:**

```sql
SELECT *
FROM `YOUR_PROJECT_ID.YOUR_AGENT_NAME_telemetry.agent_events`
ORDER BY timestamp DESC
LIMIT 100;
```

**ツール呼び出しとエラー:**

```sql
SELECT
  timestamp,
  JSON_VALUE(content, '$.tool') AS tool_name,
  JSON_VALUE(content, '$.args') AS tool_args,
  status,
  error_message
FROM `YOUR_PROJECT_ID.YOUR_AGENT_NAME_telemetry.agent_events`
WHERE event_type IN ('TOOL_COMPLETED', 'TOOL_ERROR')
ORDER BY timestamp DESC;
```

**LLM トークン使用量:**

```sql
SELECT
  agent,
  JSON_VALUE(attributes, '$.model') AS model,
  SUM(CAST(JSON_VALUE(attributes, '$.usage_metadata.prompt') AS INT64)) AS total_prompt_tokens,
  SUM(CAST(JSON_VALUE(attributes, '$.usage_metadata.completion') AS INT64)) AS total_completion_tokens
FROM `YOUR_PROJECT_ID.YOUR_AGENT_NAME_telemetry.agent_events`
WHERE event_type = 'LLM_RESPONSE'
  AND JSON_VALUE(attributes, '$.usage_metadata.prompt') IS NOT NULL
GROUP BY agent, model;
```
