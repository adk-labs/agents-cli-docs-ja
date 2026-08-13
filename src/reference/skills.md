# スキルリファレンス

スキルとは、`agents-cli setup` を介してコーディングエージェント（Antigravity CLI、Claude Code、GitHub Copilot）にインストールされるコンテキストファイルです。生成されたエージェントプロジェクトの操作に関するドメイン固有のガイダンスを提供します。

```bash
agents-cli setup      # すべてのスキルをインストール
agents-cli update     # スキルを再インストール / 更新
```

---

## `google-agents-cli-adk-code`

エージェントタイプ、ツール定義、オーケストレーションパターン、コールバック、および状態管理に関するクイックリファレンスを提供します。

---

## `google-agents-cli-deploy`

デプロイメントワークフロー、サービスアカウント、ロールバック、および本番インフラストラクチャをカバーします。Google ADK (Agent Development Kit) スキルスイートの一部です。

---

## `google-agents-cli-eval`

評価のライフサイクル全体をカバーします: データセットスキーマ、トレースの生成と採点、実行結果の比較、失敗クラスタの分析、メトリクスの検出、プロンプト最適化、LLM-as-judge 設定、および一般的な失敗の原因。Google ADK (Agent Development Kit) スキルスイートの一部です。

---

## `google-agents-cli-observability`

Cloud Trace、プロンプト・レスポンス ログ、BigQuery Agent Analytics、サードパーティ統合（AgentOps、Phoenix、MLflow など）、およびトラブルシューティングをカバーします。Google ADK (Agent Development Kit) スキルスイートの一部です。

---

## `google-agents-cli-publish`

ADK 対 A2A 登録モード、プログラムによる使用および対話型の使用、フラグリファレンス、デプロイメントメタデータからの自動検出、およびトラブルシューティングをカバーします。Google ADK (Agent Development Kit) スキルスイートの一部です。

---

## `google-agents-cli-scaffold`

`agents-cli scaffold create`、`scaffold enhance`、`scaffold upgrade` コマンド、テンプレートオプション、デプロイターゲット、およびプロトタイプ優先ワークフローをカバーします。

---

## `google-agents-cli-workflow`

常にアクティブ — 完全なワークフロー（スキャフォールド、ビルド、評価、デプロイ、パブリッシュ、監視）、コード保持ルール、モデル選択ガイダンス、および ADK または任意のエージェント開発向けのトラブルシューティング手順を提供します。
