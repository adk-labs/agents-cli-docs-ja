# CI/CD と本番環境

プルリクエスト時にテストを実行し、マージ時にステージングにデプロイし、手動承認を経て本番環境に昇格させる CI/CD パイプラインをセットアップします。

![プロトタイプから本番環境へ](../assets/prototype_to_prod.png)

---

## パイプラインの仕組み

1. **CI パイプライン** (プルリクエスト時にトリガー):
    - ユニットテストおよび統合テストを実行します。

2. **ステージング CD パイプライン** (`main` へのマージ時にトリガー):
    - コンテナをビルドし、Artifact Registry にプッシュします。
    - **ステージング環境** にデプロイします。
    - 自動負荷テストを実行します。

3. **本番デプロイメント** (ステージング成功後にトリガー):
    - 進行する前に **手動承認** が必要です。
    - ステージングでテストされた同じコンテナイメージをデプロイします。

---

## パイプラインのセットアップ

単一のコマンドを実行して、インフラストラクチャのプロビジョニングと CI/CD の設定を行います:

```bash
agents-cli infra cicd \
  --staging-project my-staging-project \
  --prod-project my-prod-project
```

これにより以下が処理されます:

- ステージングおよび本番環境向けの Terraform による **インフラストラクチャプロビジョニング**
- 選択したランナー (Cloud Build または GitHub Actions) での **CI/CD 設定**
- GitHub への **リポジトリ接続**

### CI/CD ランナーの検出

| ランナー | 検出方法 |
|--------|-------------------|
| GitHub Actions | プロジェクト内の `wif.tf` から自動検出されます。キーレス認証のために Workload Identity Federation を使用します。 |
| Google Cloud Build | Terraform 設定から自動検出されます。GitHub への Cloud Build 接続をセットアップします。 |

### オプション

| フラグ | 説明 |
|------|-------------|
| `--staging-project` | ステージング環境の GCP プロジェクト ID (必須) |
| `--prod-project` | 本番環境の GCP プロジェクト ID (必須) |
| `--cicd-project` | CI/CD リソース用の個別プロジェクト (デフォルト: prod) |
| `--dev-project` | 開発プロジェクト (オプション、開発インフラもプロビジョニング) |
| `--repository-name` | GitHub リポジトリ名 |
| `--repository-owner` | GitHub リポジトリの所有者 |
| `--local-state` | GCS の代わりにローカルの Terraform 状態を使用 |
| `--create` | 新しい GitHub リポジトリを作成 (既存リポジトリを使用する場合は省略) |

---

## Terraform 変数

パイプラインは `deployment/terraform/variables.tf` で定義された Terraform 変数を使用します:

| 変数 | 説明 |
|----------|-------------|
| `project_name` | リソース命名のベース名 |
| `prod_project_id` | 本番環境の Google Cloud プロジェクト ID |
| `staging_project_id` | ステージング環境の Google Cloud プロジェクト ID |
| `cicd_runner_project_id` | CI/CD パイプラインが実行される Google Cloud プロジェクト ID |
| `region` | Google Cloud リージョン (デフォルト: `us-west1`) |
| `repository_name` | GitHub リポジトリ名 |
| `repository_owner` | GitHub のユーザー名または組織名 |
| `app_sa_roles` | アプリケーションサービスアカウントのロール |
| `cicd_roles` | CI/CD ランナーサービスアカウントのロール |

---

デプロイが完了したら、`agents-cli publish gemini-enterprise` を使用してエージェントを Gemini Enterprise に登録します。すべてのオプションを表示するには `--help` を付けて実行してください。
