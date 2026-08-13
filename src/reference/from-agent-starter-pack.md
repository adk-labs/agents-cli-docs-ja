# Agent Starter Pack からの移行

`agents-cli` は Agent Starter Pack (ASP) の後継です。同じ基盤の上に構築され、重要な改善が加えられています。

---

## 変更点

**コーディングエージェント優先。** ASP は対話型 CLI を実行する人間向けに構築されました。agents-cli はコーディングエージェント向けに構築されています — ADK、評価、デプロイ、および可観測性に関する深いコンテキストを提供する 7 つのバンドルスキルを備えています。すべてのコマンドはターミナルからもそのまま機能します。

**Makefile を CLI に置き換え。** ASP は `make` ターゲット（`make dev`、`make eval`、`make deploy`）を使用していました。agents-cli はそれらを、フラグ、ヘルプテキスト、および構造化出力を備えたライフサイクル全体をカバーする統合 CLI に置き換えます。

**新機能。** `agents-cli` には、ASP に存在しなかったコマンドが追加されています: `playground`、`run`、`deploy`、完全な評価機能 (`eval generate`、`eval grade`、`eval dataset synthesize`、`eval compare`、`eval analyze`、`eval metric list`、`eval optimize`)、`lint`、`login`、およびスキル管理 (`setup`、`update`)。

### コマンドのマッピング

| Agent Starter Pack | agents-cli |
|---|---|
| `create` | `create` (`scaffold create` のエイリアス) |
| `enhance` | `scaffold enhance` |
| `upgrade` | `scaffold upgrade` |
| `setup-cicd` | `infra cicd` |
| `register-gemini-enterprise` | `publish gemini-enterprise` |

### 設定キー

設定は `pyproject.toml` 内の `[tool.agent-starter-pack]` から専用の `agents-cli-manifest.yaml` に移動しました:

**変更前 (`pyproject.toml`)**
```toml
[tool.agent-starter-pack]
agent_directory = "app"

[tool.agent-starter-pack.create_params]
deployment_target = "cloud_run"
```

**変更後 (`agents-cli-manifest.yaml`)**
```yaml
name: my-agent
agent_directory: app
create_params:
  deployment_target: cloud_run
```

### テンプレートのカバレッジ

agents-cli は `adk` テンプレート (Python) をサポートしており、すべての ADK エージェントに A2A が組み込まれています — 独立していた `adk_a2a` テンプレートは `adk` に統合されました。RAG はテンプレートではなくクローンして学習するためのレシピです（以前の `agentic_rag` テンプレートは削除されました。代わりに `rag-vector-search` / `rag-agent-search` サンプルを適用してください）。ASP には agents-cli でまだ利用できない追加のテンプレート（`adk_go`、`adk_java`、`adk_ts`、`adk_live`、`custom_a2a`）がありました。これらに対するサポートも計画されています。

### 変更されない点

- **テンプレート** — 同じ `adk` エージェントテンプレート（RAG はクローン・学習レシピになりました）、同じデプロイターゲット、同じセッションストレージオプション
- **プロジェクト構造** — 生成されたプロジェクトは同じレイアウトを持ち、`app/agent.py` のコードは変更されません
- **Terraform** — `deployment/terraform/` 以下の同じ Infrastructure as Code
- **CI/CD パイプライン** — 同じ Cloud Build および GitHub Actions 設定

---

## 既存プロジェクトの移行

既存の ASP プロジェクトとの完全な互換性があります。必要な唯一の変更は `pyproject.toml` 内の設定セクションの名前を変更することです。

**ステップ 1: agents-cli のインストール**

```bash
uvx google-agents-cli setup
```

**ステップ 2: 設定セクションの名前変更**

```bash
sed -i '' 's/tool.agent-starter-pack/tool.agents-cli/g' pyproject.toml
```

次回設定が読み込まれると、`agents-cli-manifest.yaml` への自動移行がトリガーされ、`pyproject.toml` から `tool.agents-cli` セクションが削除されます。

**ステップ 3: 検証**

```bash
agents-cli info
```

これによりプロジェクト設定が表示され、agents-cli がそれを読み取れることが確認できます。エージェントコード、テスト、Terraform、および CI/CD パイプラインはすべて以前と同様に機能します。

!!! note "`tests/eval/evalsets/` 配下に既存の評価ケースがありますか？"
    ASP のデフォルトエージェントテンプレートには、ADK `EvalSet` スキーマを使用した `basic.evalset.json` が含まれていました。agents-cli の評価機能は `tests/eval/datasets/` から異なるフォーマットを読み込みます。変換については [評価データセットの移行](eval-dataset-migration.md) を参照してください。
