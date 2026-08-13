# はじめに

**Agents CLI in Agent Platform** は、Google Cloud 上で AI エージェントを構築、評価、デプロイするための CLI およびスキルパッケージです。エージェントは Google の [Agent Development Kit (ADK)](https://google.github.io/adk-docs/) を使用して構築され、Agents CLI はスキャフォールディング、評価、デプロイ、可観測性など、周辺のすべての作業を処理します。

2通りの方法で動作します:

1. **コーディングエージェントを使用する** — Antigravity CLI、Claude Code、Codex などにスキルをインストールします。コーディングエージェントはこれらのスキルを使用して、各ステップで適切な判断を下します。
2. **コーディングエージェントを使用しない** — ターミナルから直接 CLI コマンドを実行します。すべてのコマンドはスタンドアロンで機能します。

Agents CLI には、ADK ライフサイクル全体にわたる深い知識をコーディングエージェントに提供する **7つのスキル** が同梱されています:

| スキル | コーディングエージェントが学習する内容 |
|-------|-------------------------------|
| `google-agents-cli-workflow` | 開発ライフサイクル、コード保持、モデル選択 |
| `google-agents-cli-adk-code` | ADK Python API — エージェント、ツール、オーケストレーション、コールバック |
| `google-agents-cli-scaffold` | プロジェクトのスキャフォールディング — `create`、`enhance`、`upgrade` |
| `google-agents-cli-eval` | 評価ライフサイクル — データセット、メトリクス、生成/採点、比較、分析、最適化 |
| `google-agents-cli-deploy` | デプロイ — Agent Runtime、Cloud Run、GKE、CI/CD |
| `google-agents-cli-publish` | Gemini Enterpriseへの登録 |
| `google-agents-cli-observability` | Cloud Trace、ロギング、サードパーティ統合 |

---

## 前提条件

**必須:** [Python 3.11+](https://www.python.org/downloads/)、[uv](https://docs.astral.sh/uv/getting-started/installation/)、[Node.js](https://nodejs.org/en/download) (スキルインストール用)

**オプション (デプロイ用):** [Google Cloud SDK](https://cloud.google.com/sdk/docs/install)、[Terraform](https://developer.hashicorp.com/terraform/downloads)

---

## インストール

```bash
uvx google-agents-cli setup
```

これにより、CLI とコーディングエージェント用のコンテキストアウェアなスキルがインストールされます。

??? info "その他のインストール方法"
    **pipx:** `pipx install google-agents-cli && agents-cli setup`

    **venv + pip:** `pip install google-agents-cli && agents-cli setup`

    **スキルのみ:** `npx skills add google/agents-cli`

**対応プラットフォーム:** macOS、Linux、および Windows (WSL 2)。ネイティブ Windows は公式にはサポートされていません。

---

## 認証

`gcloud` ですでに認証されている場合、そのままで動作します — Agents CLI はアプリケーションデフォルト認証情報 (ADC) を自動的に読み込みます。

それ以外の場合、最も手軽な方法は [AI Studio](https://aistudio.google.com/apikey) から Gemini API キーを取得することです:

```bash
export GEMINI_API_KEY="your-key-here"
```

詳細については [認証](authentication.md) を参照してください。

---

## コーディングエージェントで開発を開始する

=== "Antigravity CLI"

    1. **Antigravity CLI を開く**

        IDE またはターミナルから Antigravity を起動します。

    2. **スキルがインストールされていることを確認する**

        Agents CLI スキルがお使いの環境で利用可能であることを確認します。

    3. **構築を依頼する**

        ```
        Build a support agent that answers questions from our docs
        ```

        Antigravity はインストールされたスキルを使用して、エージェントのスキャフォールディング、構築、評価を行います。

=== "Claude Code"

    1. **Claude Code を開く**

        ```bash
        claude
        ```

    2. **スキルがインストールされていることを確認する**

        ```
        /skills
        ```

        `google-agents-cli-workflow` やその他の Agents CLI スキルが一覧表示されていることを確認します。

    3. **構築を依頼する**

        ```
        Build a support agent that answers questions from our docs
        ```

        Claude はインストールされたスキルを使用して、エージェントのスキャフォールディング、構築、評価を行います。

=== "Codex"

    1. **Codex を開く**

        ```bash
        codex
        ```

    2. **スキルがインストールされていることを確認する**

        Agents CLI スキルがお使いの環境で利用可能であることを確認します。

    3. **構築を依頼する**

        ```
        Build a support agent that answers questions from our docs
        ```

        Codex はインストールされたスキルを使用して、エージェントのスキャフォールディング、構築、評価を行います。

=== "その他のエージェント"

    Agents CLI は [スキル (skills)](https://agentskills.io/what-are-skills) をサポートする任意のコーディングエージェントで動作します。

    1. **スキルをインストールする**

        ```bash
        uvx google-agents-cli setup
        ```

    2. **スキルが認識されていることを確認する**

        エージェントが `google-agents-cli-workflow` やその他の Agents CLI スキルを認識できているか確認します。多くのエージェントは `/skills` コマンドまたは設定パネルでこれを提供しています。

    3. **構築を依頼する**

        ```
        Build a support agent that answers questions from our docs
        ```

        スキルがインストールされ認識されていれば、エージェントは自動的にそれらを使用します。

---

## コマンドを自分で入力したい場合

ターミナルからワークフロー全体を実行することもできます — コーディングエージェントは不要です。

```bash
# 最小限のエージェントプロジェクトを作成
agents-cli create my-agent --prototype --yes

# 依存関係をインストールし、開発用プレイグラウンドを起動
cd my-agent
agents-cli install
agents-cli playground
```

これにより、ホットリロード機能を備えた ADK Web プレイグラウンドが `http://localhost:8080` で起動します。

完全な手順については、[手動ワークフローチュートリアル](hands-on-tutorial.md) を参照してください。

---

## デモ

<div align="center">
  <iframe width="100%" height="450" src="https://www.youtube.com/embed/ECYKo70pPNc" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

---

## 次のステップ

- [チュートリアル: 最初のエージェントを構築する](quickstart-tutorial.md) — コーディングエージェントを使用した構築、評価、デプロイ
- [チュートリアル: 手動ワークフロー](hands-on-tutorial.md) — すべてのコマンドを自分で入力して実行
- [ユースケース](use-cases.md) — 実際に構築されているエージェントパターンのヒント
- [プロジェクト構造](project-structure.md) — 生成された各ファイルの役割を理解する
- [エージェントテンプレート](templates.md) — `adk` テンプレートと RAG クローン＆学習レシピ
- [開発ガイド](development.md) — 開発ワークフローの詳細
- [CLI リファレンス](../cli/index.md) — すべてのコマンドとフラグ

---

!!! tip "Agent Starter Pack から移行しますか？"
    [移行ガイド](../reference/from-agent-starter-pack.md) を参照してください。

!!! note "成果物の共有"
    Agents CLI で面白いものを構築しましたか？ぜひお聞かせください！[agents-cli@google.com](mailto:agents-cli@google.com) までプロジェクトをお知らせください。
