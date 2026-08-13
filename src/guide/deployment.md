# デプロイメント

CI/CD パイプラインを使用して、開発環境または本番環境にエージェントをデプロイします。

![プロトタイプから本番環境へ](../assets/prototype_to_prod.png)

---

## インフラストラクチャ対デプロイメント

**インフラストラクチャ** (`agents-cli infra`) は、エージェントが必要とするクラウドリソース（サービスアカウント、IAM バインディング、API、テレメトリバケット、Terraform 状態）をプロビジョニングします。これは準備を行うステップであり、エージェントを実行するわけではありません。

**デプロイメント** (`agents-cli deploy`) は、エージェントコードを受け取り、プロビジョニングされたインフラストラクチャ上に配置します — コンテナをビルドし、レジストリにプッシュし、サービスを開始します。

一般的なフロー: 最初にインフラストラクチャをプロビジョニングし、その上にデプロイします。

---

## 開発環境へのデプロイ

デプロイを実行するための最もシンプルな手順:

**1. 開発プロジェクトの設定:**

```bash
gcloud config set project YOUR_DEV_PROJECT_ID
```

**2. エージェントのデプロイ:**

```bash
agents-cli deploy
```

このコマンドは、`agents-cli-manifest.yaml`（`create_params` 内）から `deployment_target` を読み取り、適切なフローに振り分けます:

| `deployment_target`  | 発生する処理                                  |
|----------------------|-----------------------------------------------|
| `agent_runtime`      | Agent Runtime デプロイメント（完全管理型）       |
| `cloud_run`          | `gcloud beta run deploy` (Cloud Run 上のコンテナ) |
| `gke`                | Terraform + Docker ビルド + `kubectl apply`     |

デプロイ先ターゲットはプロジェクト作成時に設定されます:

```bash
agents-cli create my-agent -d cloud_run    # または agent_runtime, gke
```

既存プロジェクトのデプロイ先ターゲットを変更するには、`scaffold enhance` を使用します:

```bash
agents-cli scaffold enhance -d cloud_run
```

利用可能なすべてのオプションを確認するには、`agents-cli scaffold enhance --help` を実行してください。

!!! tip
    可観測性機能（プロンプト・レスポンス ログ、コンテンツ ログ）を有効にするには、デプロイ後に `agents-cli infra single-project` を実行します。Terraform はテレメトリリソースをプロビジョニングし、それらを使用するようにサービスを更新します。詳細については [可観測性ガイド](observability/index.md) を参照してください。

**動作の検証:**

```bash
agents-cli deploy --list    # デプロイメントの一覧表示
agents-cli deploy --status  # デプロイメントステータスの確認
```

---

## デプロイターゲット

### Agent Runtime

*`agents-cli create my-agent -d agent_runtime` で選択、または `agents-cli-manifest.yaml` の `create_params.deployment_target: agent_runtime` で指定。*

完全管理型ランタイム: `Dockerfile`（スキャフォールディング済み）を提供するだけで、Agent Engine がコンテナをビルドして実行します — クラスターやサービスの運用は不要です:

```bash
agents-cli deploy --project my-gcp-project --region us-east1
```

Docker ビルド引数やコンテナポートを指定できます。ビルド済みの `--image` はサポートされていません（Agent Runtime は常に Dockerfile からビルドします）:

```bash
agents-cli deploy --build-args KEY=VALUE --port 8080
```

非同期デプロイメントの確認:

```bash
agents-cli deploy --no-wait     # 開始してすぐに復帰
agents-cli deploy --status      # 後から進捗を確認
```

### Cloud Run

*`agents-cli create my-agent -d cloud_run` で選択、または `agents-cli-manifest.yaml` の `create_params.deployment_target: cloud_run` で指定。*

ソースからコンテナをビルドし、Cloud Run サービスとしてデプロイします:

```bash
agents-cli deploy --project my-gcp-project --region us-east1
```

リソース制限の上書き:

```bash
agents-cli deploy --memory 8Gi --port 8080
```

ソースからビルドする代わりに、ビルド済みのイメージをデプロイ:

```bash
agents-cli deploy --image gcr.io/my-project/my-agent:v1
```

!!! tip
    `agents-cli` のフラグで公開されていない高度な Cloud Run デプロイメント機能が必要な場合は、`--dry-run`（または `-n`）を使用して完全な `gcloud` コマンドを出力します。それをコピーして必要に応じて追加の引数を追加できます。

### GKE

*`agents-cli create my-agent -d gke` で選択、または `agents-cli-manifest.yaml` の `create_params.deployment_target: gke` で指定。*

Terraform と kubectl を使用して GKE クラスターにデプロイします:

```bash
agents-cli deploy --cluster-name my-cluster --project my-gcp-project
```

---

## 次のステップ

- [CI/CD と本番環境](cicd.md) — ステージングおよび本番環境向けの自動化パイプラインのセットアップ
- [可観測性](observability/index.md) — デプロイされたエージェントのモニタリング
