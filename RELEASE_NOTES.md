# リリースノート

このプロジェクトのすべての主要な変更点はこのファイルに記録されます。

## [1.3.1] - 2026-08-04

- agent-plugins.org プラグインマニフェストを追加
- CICD コマンドのバックオフ処理をよりスマートに改善
- `setup` 中の軽微な誤りエラーメッセージを回避するため npx skills 1.5.9 へロールバック
  - https://github.com/google/agents-cli/issues/59

## [1.3.0] - 2026-07-31

- A2A 0.3 クライアント用互換性レイヤーを備えた、全体にわたる A2A 1.0 使用へのアップデート
- Cloud Run デプロイの再試行処理を改善
- `infra cicd` コマンドの再試行処理をストリームライン化
- setup および update 時の `npx skills` 出力フォーマットを修正
- Agent Runtime デプロイ開始時に Cloud Logging の URL を出力
- eval generate/run に `--concurrency` および `--header` フラグを追加
- `agent_runtime` と併用した際の `--session-type` 周りのメッセージを微調整

## [1.2.1] - 2026-07-23

- ヤンクされた `opentelemetry-resourcedetector-gcp` バージョンに起因する `uv sync` 時のインポート問題を修正
- `--url` を使用した `eval generate` 用のオプション HTTP ベースパスを追加 (開発中の機能)

## [1.2.0] - 2026-07-21

- **クラウドテレメトリが CLI およびデプロイ全体で ADK の `otel_to_cloud` へ移行しました。** `playground` および `run` に ADK の現在の `--otel_to_cloud` を転送する `--otel-to-cloud` フラグが追加されました。従来の `--trace-to-cloud` は非表示の機能的なエイリアスとして残り、使用時に警告を表示します。デプロイ側では、Agent Runtime が `otel_to_cloud` 経由でエクスポートするようになり (`GOOGLE_CLOUD_AGENT_ENGINE_ENABLE_TELEMETRY` に依存)、生成されたプロジェクトはランタイムではなく Terraform でクラウドテレメトリを宣言的に構成します。同梱された `google-adk` は `otel-gcp` エクストラ付きの `>=2.2.0` に移行し、可観測性スキルもこれに合わせて更新されました。
- スキャフォールドのデフォルトモデルが `gemini-3.6-flash` になりました。
- **Agent Analytics が Agent Runtime 上の GenAI 完了ログをキャプチャするようになりました。** BigQuery テレメトリログシンクがデプロイターゲットごとに構成されるため、Agent Runtime デプロイは GenAI リクエスト/レスポンスログを BigQuery にルーティングし、完了ビューが空になるのを防ぎます。
- **ADK コードスキルに Managed Agents のガイドを追加しました。** 同梱の ADK チートシートに `ManagedAgent` を使用するタイミング、Gemini API と Agent Platform (GEAP) のセットアップ、実行可能な作成および使用例をカバーする Managed Agents セクションが追加されました。

## [1.1.0] - 2026-07-10

- **新しいエージェントのための対話型ブレインストーミング。** ワークロースキルの Phase 0 が対話型のブレインストーミングダイアログとなり、コードを書く前にエージェントの仕様を形成し、質問できない場合は確認用に仮定を表示するようになりました。
- `eval generate` および `eval grade` が、結果に影響しないサードパーティの警告やプログレスバーで出力を汚さなくなりました。失敗時には引き続き表示され、デバッグ用に再有効化も可能です。
- 生成されたプロジェクトでスキャフォールドされたデフォルトの評価メトリクスモジュールの名前が、実装するメトリクスに合わせて `tests/eval/response_quality.py` (以前は `metrics.py`) に変更されました。
- CLI 全体にわたる広範な Windows 互換性の修正。
- ユーザー向けヒントのコマンド名タイポを修正し、正しい `agents-cli` コマンドを指すようにしました。
- 同梱スキルの刷新 (RAG サンプルが `google/adk-samples` 内の `core/python/` を指すように変更など)。

## [1.0.0] - 2026-06-30

**Agents CLI 1.0 — 一般提供 (GA) 開始。** 初の GA リリースです。Google Cloud 上で ADK エージェントをスキャフォールディング、評価、デプロイするための安定した本番対応 CLI です。

- 再デプロイ時に、指定されていない設定をリセットするのではなく、Agent Runtime および Cloud Run 上の既存のデプロイスペックを保持するようになりました。
- Agent Runtime デプロイ時、ソースのパッケージング時に `.gcloudignore` および `.gitignore` を尊重するようになり、アップロードに無視されたファイルが含まれなくなりました。
- RAG がクローンして学習するレシピになりました。`google/adk-samples` の `rag-vector-search` / `rag-agent-search` サンプルから開始してください (ワークロースキル経由で表示)。`agentic_rag` テンプレート、`--datastore` フラグ、および `infra datastore` / `data-ingestion` コマンドは削除され、リダイレクトを表示するようになりました。
- 生成されたプロジェクトで Python 環境構成が単一のテンプレート化された `.env` ファイルに統合されました。
- 評価コマンドが評価メタデータのイントロスペクション時に ADK ツールセットを許容するようになり、ツールセットを使用するエージェントでメタデータ収集が失敗しなくなりました。
- GKE Cloud Build デプロイがログストリーミング制限に対して耐性を持つようになり、ビルドログストリームが切り詰められた場合でも失敗しなくなりました。
- 同梱スキルの刷新: RAG サンプルが adk-samples `main` の `core/` を指すようになり、常時アクティブなワークロースキルが共通化およびスリム化され、ADK コードガイドに Agent Runtime でのデバッグ用に `streaming_agent_run_with_events` が記載されました。

## [0.6.1] - 2026-06-28
- `publish gemini-enterprise` はデフォルトで ADK 経由で Agent Runtime デプロイを登録するようになり、Gemini Enterprise がネイティブかつ確実に呼び出せるようになりました。Cloud Run および GKE では引き続き A2A 登録がデフォルトです。Agent Runtime で A2A を要求すると警告が表示され ADK が推奨されます。A2A エージェントを再公開しても重複登録が作成されなくなり、A2A エージェントカードは最初のデプロイ時に正しい公開 URL を運ぶようになりました。
- `agents-cli update` は、誤解を招く緑色の「Skills updated.」バナーを常に表示するのではなく、スキルの更新に失敗した場合にゼロ以外のステータスコードで終了し、明確にレポートするようになりました。また、Windows PowerShell で失敗メッセージが誤った色でレンダリングされる問題も修正されました。
- テンプレート化されたすべてのエージェントの生成プロジェクト `uv.lock` ファイルを刷新し、同梱の `google-adk` を 2.2.0 から 2.3.0 にアップデートしました。

## [0.6.0] - 2026-06-23
- Agent Runtime デプロイが、単一の統合コンテナアプリから ADK Web、A2A、および Reasoning Engine を処理するようになりました。
- Cloud Trace スパンが LLM プロンプトとレスポンスをキャプチャしなくなり、機密コンテンツがトレースに含まれないようになりました。
- 同梱スキルの刷新: 正確性の修正、重複排除、常時アクティブなワークローガイドの軽量化、および ADK コードチートシートへの a2ui のドキュメント化。

## [0.5.1] - 2026-06-18
- Windows での run および playground コマンドを修正
  - https://github.com/google/agents-cli/issues/34
  - https://github.com/google/agents-cli/issues/35
  - これらを発見し報告してくれた @Abdullah-k0de に感謝します！
- 障害調査ガイド内の古くなった GCS バケットを修正
- publish スキルに Agent Registry フリート管理を追加

## [0.5.0] - 2026-06-15
- `deploy` が Agent Runtime および Cloud Run 用のマシン形状パラメータをフラグとして公開するようになりました。
- `deploy` に `--service-name` オーバーライドを追加しました。
- `run` がセッションフッターにコピー＆ペースト可能な再開コマンドを出力するようになりました。
- `run` が通常の実行時に再利用されたローカルサーバーを削除しなくなりました。
- `scaffold upgrade` が `uvx` 経由で旧バージョンのテンプレートを構築するようになりました。
- スキルのセットアップ/更新が大きな `npx` 出力でハングしなくなりました (パイプバッファのデッドロック)。
- プロジェクトルートの通知が、コマンドが実際にディレクトリを変更したときにのみ印刷されるようになりました。
- 同梱スキルおよび生成されたプロジェクトの README における既存の不正確さを修正しました。
- ソースコードが公開 GitHub リポジトリに公開されました: https://github.com/google/agents-cli

## [0.4.0] - 2026-06-10
- スキャフォールドされた Python テンプレートが **ADK 2.0 GA** を使用するようになりました。新しい `adk`、`adk_a2a`、および `agentic_rag` プロジェクトは `google-adk[gcp]>=2.0.0,<3.0.0` に固定されます。`[gcp]` エクストラは OpenTelemetry GCP エクスポートを復元し、BigQuery クライアントをバンドルするため、個別の `[bigquery-analytics]` エクストラは不要になりました。Cloud Run および GKE 上の Cloud SQL セッションは 2.0 でも動作し続けます。同梱された ADK コーディングスキルとそのリファレンスドキュメントが 2.0 用に刷新されました。
  - https://github.com/google/agents-cli/issues/24
- Agent Runtime デプロイが Cloud Run の動作に合わせ、`--update-env-vars` 経由で渡されたユーザー指定の `AGENT_VERSION` (または `NUM_WORKERS`) を上書きしなくなりました。「version not found」警告に、設定すべき `pyproject.toml` のフィールド名が表示されるようになりました。
- Cloud Trace 可観測性ガイド内の古い `deployment/terraform/dev/` パスを修正し、現在の `single-project` terraform レイアウトに合わせました。

## [0.3.1] - 2026-06-04
- `eval generate` が `VertexAiSearchTool` などの組み込みツールを使用する ADK 2.x プロジェクトで動作するようになりました。SDK 修正が含まれる `google-cloud-aiplatform` の下限を 1.156.0 に引き上げました。
  - https://github.com/google/agents-cli/issues/27
- `agents-cli setup` 経由でインストールされたスキルが Antigravity から認識できるようになりました。グローバルスキルが Antigravity スキルディレクトリにミラーリングされます。
  - https://github.com/google/agents-cli/issues/26
- `update` がサイレントに失敗するのではなく、エラーを明確に表示するようになりました。
- エージェントのデプロイ時、壊れたまたは不正な `deployment_metadata.json` をクラッシュせずに許容するようになりました。
- デプロイのタイムスタンプがタイムゾーンを認識するようになりました。
- 不正な `AGENTS_CLI_EXPERIMENTS` 値によって CLI がクラッシュしなくなりました。
- `agents-cli install` が `--locked` で実行されるようになり、乖離した `uv.lock` はサイレントに新しい依存関係バージョンを解決するのではなく、迅速に失敗するようになりました。

## [0.3.0] - 2026-05-29
### 破壊的変更
- 評価データフォーマットが ADK `EvalSet` から Vertex AI `EvaluationDataset` に変更されました。既存の `tests/eval/evalsets/*.evalset.json` ファイルは `agents-cli eval generate` 等で読み込まれなくなりました。変換方法については [評価データセットの移行](docs/src/reference/eval-dataset-migration.md) を参照してください。`scaffold upgrade` はレガシーファイルが検出された場合に通知を出力するようになりました。

### 評価 - Quality Flywheel (プレビュー)
- LLM 駆動のユーザーシミュレーションデータセット生成用に `eval dataset synthesize` を追加しました。
- `EvaluationDataset` 全体でエージェント推論を実行しトレースを出力する `eval generate` を追加しました。
- 組み込みまたはカスタムメトリクスに基づいてエージェントトレースをスコア付けする `eval grade` を追加しました。
- Vertex AI Eval Service 上でエンドツーエンドのクラウド側評価実行を送信する `eval submit` を追加しました。
- 完了したクラウド評価実行から結果を取得する `eval results` を追加しました。
- 採点済み結果の失敗モード分析用に `eval analyze` を追加しました。
- 組み込みの評価メトリクスを探索する `eval metric list` を追加しました。
- Quality Flywheel ワークフロー (dataset, generate, grade, analyze, optimize) をカバーするように `eval` スキルをエンドツーエンドで書き直しました。

### その他
- スキルの軽微な一貫性修正

## [0.2.1] - 2026-05-28
- --dry-run のエイリアスとして --dryrun を追加
- よりスマートなスキルインストール
  - https://github.com/google/agents-cli/issues/23
- パフォーマンス向上のための資格情報のキャッシュ
- gcloud なしで動作するように is_authenticated を修正
  - https://github.com/google/agents-cli/issues/16
- エージェントランタイムのデプロイエラー表示をより明確に修正
- 不要になった gcloud コマンドから 'beta' を削除
- 壊れていたドキュメントリンクを修正
- deploy でのエクスポート試行前にロックファイルが欠落している場合は自動生成
  - https://github.com/google/agents-cli/issues/17

## [0.2.0] - 2026-05-15
- agent-cli プロジェクト構成を言語非依存の agents-cli-manifest.yaml ファイルに移動しました
  - pyproject.toml に埋め込まれた古い構成は `agents-cli scaffold upgrade` で自動移行できます
- `eval optimize` コマンドを追加しました
- deploy に --network-attachment および --dns-peering-* フラグを追加しました
- 起動パフォーマンスに関するその他の改善
- ターミナルエンコーディングに関連するクラッシュを回避
  - https://github.com/google/agents-cli/issues/15 を修正
- 特に Windows 向けのツールパス解決をスマート化
  - https://github.com/google/agents-cli/issues/14 を修正
- 依存関係バージョンのロックを更新
  - https://github.com/google/agents-cli/issues/13 を修正
- Claude および Gemini CLI プラグインサポートのマニフェストサポートを追加
- スキャフォールディング、強化、および/またはアップグレード時に適切な構成メタデータを保持する際の一部のバグを修正
- ドキュメントおよびスキルのその他の修正
- https://google.github.io/agents-cli/ に Agents CLI ライフサイクルのビジュアル解説ページを追加
- 不要なテンプレートコードをクリーンアップ

## [0.1.3] - 2026-05-06
- `infra` コマンドのデフォルトを terraform apply ではなく terraform plan に変更
- Cloud Shell などの同様の環境で動作するように `playground` を修正し、基礎となるコマンドについて透明性を向上
- スキルを更新して cloud sql ロールの必要性をカバー
- バグ報告を容易にするため `agents-cli info` に OS 情報を表示するように変更
- `run` が `--start-server` で要求された場合にのみバックグラウンドサーバーを起動するように変更
- ADC 認証の表示文字列をより明確に改善
- 壊れていたドキュメントリンクを修正
- agent_runtime の欠落していたターゲットの説明を修正

## [0.1.2] - 2026-04-29
- ドキュメントおよび画像の修正
- プロジェクトメタデータの修正
- completions_view BigQuery SQL でマルチホップトレースを保持
- セットアップ中にレガシー ADK スキルを検出
- インラインアーティファクトを .google-agents-cli/artifacts/ に保存
- Windows シェル対話のいくつかの問題を修正
- `deploy` の未処理のパススルー引数を削除し、スキルと --help テキストを更新
- 認証が古くなった際に agents-cli がユーザーを認証済みとみなす問題を修正
- エラー時にローカル `run` サーバーを自動停止

## [0.1.1] - 2026-04-22
- パフォーマンスの向上 (特に CLI の起動時間)
- ドキュメントのクリーンアップ

## [0.1.0] - 2026-04-21
- 初回パブリックリリース
