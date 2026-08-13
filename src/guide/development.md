# 開発ガイド

このガイドでは、構築対象の定義から本番環境でのモニタリングに至るまで、開発ワークフロー全体をカバーします。コーディングエージェントが `google-agents-cli-workflow` スキルを通じて使用するのと同じフェーズに従います。

---

## フェーズ 0: 理解 (Understand)

コードを書く前に、構築するものを定義します。

コーディングエージェントを使用している場合、エージェントが自動的にこれらの質問を投げかけます。手動で作業している場合は、自分で回答を用意します:

1. **エージェントはどのような問題を解決するか？** — コアとなる目的と機能
2. **外部 API やデータソースは必要か？** — ツール、統合、認証の要件
3. **セーフティに関する制約事項は？** — エージェントが行っては*ならない*こと
4. **推奨されるデプロイ先は？** — まずプロトタイプを作成するか、完全なデプロイ (Agent Runtime、Cloud Run、GKE) か？

現在のディレクトリの `.agents-cli-spec.md` に回答を保存します (概要、ユースケースの例、必要なツール、制約事項、成功基準)。

---

## フェーズ 1: スキャフォールド (Scaffold)

テンプレートから新しいプロジェクトを作成します:

```bash
agents-cli create my-agent
```

作成中にエージェントテンプレート (`adk`) とデプロイターゲットを選択します。(RAG はクローン＆学習レシピです — [テンプレート](templates.md#rag-retrieval-augmented-generation) を参照してください。) インフラストラクチャの決定を行わずに迅速にプロトタイピングする場合:

```bash
agents-cli create my-agent --prototype --yes
```

後から `agents-cli scaffold enhance` を使用してデプロイのサポートを追加できます。

すべてのオプションについては [エージェントテンプレート](templates.md) を参照してください。

---

## フェーズ 2: 構築と反復 (Build & Iterate)

### コーディングエージェントを使用する場合

コーディングエージェントを開き、ワークロースキルをアクティブ化します:

```
/google-agents-cli-workflow
```

構築したい内容を説明します。コーディングエージェントはインストールされたスキルを使用して、エージェントロジックの記述、ツールの作成、変更のテストを行います — これらはすべて ADK のベストプラクティスに従います。

### 手動で作業する場合

`app/agent.py` 内のエージェントロジックを編集し、以下でテストします:

- `agents-cli playground` — ホットリロード機能を備えた ADK Web プレイグラウンドを `localhost:8080` で起動
- `agents-cli run "your prompt"` — ターミナルからの簡単な動作確認 (スモークテスト)

### コード品質 (Code Quality)

```bash
agents-cli lint                                # Ruff によるチェックとフォーマット
uv run pytest tests/unit tests/integration     # ユニットテストおよび統合テストの実行
```

### パッケージ管理 (Package Management)

[uv](https://docs.astral.sh/uv/) を使用して依存関係を追加・削除します:

- `uv add <package>`
- `uv remove <package>`

---

## フェーズ 3: 評価 (Evaluate)

構造化された評価を実行してエージェントの挙動を検証します。これには内部で [GenAI Eval SDK](https://docs.cloud.google.com/gemini-enterprise-agent-platform/optimize/evaluation/agent-evaluation) が使用されます。

```bash
agents-cli eval generate
agents-cli eval grade
```

エージェントが安定して評価をクリアするまでに、評価・修正ループの **5〜10回以上の反復** が予想されます。まず 1〜2 個のコアな評価ケースから始め、不具合を修正したのち、カバレッジを拡大してください。

メトリクス、データセットスキーマ、および詳細な方法論については [評価ガイド](evaluation.md) を参照してください。

---

## フェーズ 4: デプロイ (Deploy)

評価のしきい値を満たしたら、Google Cloud へデプロイします。

1. **デプロイターゲットの追加** (`--prototype` で開始した場合):

    ```bash
    agents-cli scaffold enhance --deployment-target cloud_run
    ```

2. **デプロイ**:

    ```bash
    agents-cli deploy
    ```

!!! tip
    可観測性機能 (プロンプト/レスポンスのロギング、コンテンツログ) を有効にするには、デプロイ後に `agents-cli infra single-project` を実行します。詳細については [可観測性ガイド](observability/index.md) を参照してください。

ステージング環境、承認ゲート、CI/CD を備えた本番パイプラインについては、[デプロイ](deployment.md) および [CI/CD & 本番環境](cicd.md) を参照してください。

---

## フェーズ 5: 公開 (Publish - オプション)

デプロイされたエージェントを Gemini Enterprise に登録します:

```bash
agents-cli publish gemini-enterprise
```

すべてのエージェントでこれが必要なわけではありません — Gemini Enterprise 経由で配布する場合にのみ実行します。

---

## フェーズ 6: 可観測性 (Observe)

本番環境でエージェントを監視します。Cloud Trace はすべてのデプロイ済みエージェントでデフォルトで有効になっており、構成は不要です。

- **Cloud Trace** — 分散トレーシング、レイテンシ分析、エラーの可視化
- **BigQuery Agent Analytics** — トークン使用量、会話パターン、LLM-as-judge スコアリングのためのオプトイン拡張アナリティクス

セットアップと使用方法については [可観測性ガイド](observability/index.md) を参照してください。

---

すべてのコマンドとフラグについては [CLI リファレンス](../cli/index.md) を参照してください。各フェーズでコーディングエージェントが使用するスキルの詳細については [スキル](../reference/skills.md) を参照してください。
