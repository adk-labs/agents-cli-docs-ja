<div align="center">
  <img src="https://raw.githubusercontent.com/google/agents-cli/main/docs/src/assets/logo_sm.png" alt="agents-cli logo" width="120" />
  <h1><code>agents-cli</code></h1>
  <p>Gemini Enterprise Agent Platform上でエージェントを構築するためのCLIおよびスキル。</p>

  <p>
    <a href="#はじめに">はじめに</a> &nbsp;&nbsp;|&nbsp;&nbsp;
    <a href="#エージェントスキル">スキル</a> &nbsp;&nbsp;|&nbsp;&nbsp;
    <a href="#cliコマンド">コマンド</a> &nbsp;&nbsp;|&nbsp;&nbsp;
    <a href="https://pypi.org/project/google-agents-cli/">PyPI</a> &nbsp;&nbsp;|&nbsp;&nbsp;
    <a href="https://github.com/google/agents-cli/issues">Issues</a> &nbsp;&nbsp;|&nbsp;&nbsp;
    <a href="https://google.github.io/agents-cli/">Docs</a> &nbsp;&nbsp;|&nbsp;&nbsp;
    <a href="https://github.com/google/agents-cli/blob/main/RELEASE_NOTES.md">リリースノート</a> &nbsp;&nbsp;|&nbsp;&nbsp;
    <a href="https://github.com/google/agents-cli">Star us</a>
  </p>
</div>

---

お気に入りのコーディングアシスタントを、Google Cloud上でのエージェント構築およびデプロイのエキスパートに変えましょう。

**Agents CLI in Agent Platform** (`agents-cli`) は、エンタープライズグレードのエージェントを構築、拡張、ガバナンス、最適化するためのスキルとコマンドをコーディングエージェントに提供します。これにより、すべてのCLIやサービスを自分で学習する必要がなくなります。

**以下のツールとシームレスに連携します:**
[Antigravity CLI](https://antigravity.google/) &nbsp;•&nbsp; [Claude Code](https://docs.anthropic.com/en/docs/claude-code) &nbsp;•&nbsp; [Codex](https://github.com/openai/codex) &nbsp;•&nbsp; *およびその他のコーディングエージェント。*

## はじめに

**前提条件:** Python 3.11+、[uv](https://docs.astral.sh/uv/getting-started/installation/)、および [Node.js](https://nodejs.org/en/download)。

### 1. インストール

```bash
uvx google-agents-cli setup
```

<details>
<summary>スキルのみを追加する場合 — あとはコーディングエージェントが処理します</summary>

```bash
npx skills add google/agents-cli
```

</details>

### 2. コーディングエージェントを開く

[Antigravity CLI](https://antigravity.google/)、[Claude Code](https://docs.anthropic.com/en/docs/claude-code)、[Codex](https://github.com/openai/codex)、またはお好みのコーディングエージェントを起動します。

### 3. 最初のエージェントを構築する

コーディングエージェントに構築を依頼します — 例: *「agents-cliを使って、冗長なテキストを簡潔で技術的な用語に圧縮する原始人スタイルのエージェントを作成してください」*

ステップバイステップの手順については、[チュートリアル全文](https://google.github.io/agents-cli/guide/quickstart-tutorial/)を参照してください。

**[公式ドキュメントを閲覧する →](https://google.github.io/agents-cli/)**

---

## エージェントスキル

| スキル | コーディングエージェントが学習する内容 |
|-------|-------------------------------|
| `google-agents-cli-workflow` | 開発ライフサイクル、コード保持ルール、モデル選択 |
| `google-agents-cli-adk-code` | ADK Python API — エージェント、ツール、オーケストレーション、コールバック、状態 |
| `google-agents-cli-scaffold` | プロジェクトのスキャフォールディング — `create`、`enhance`、`upgrade` |
| `google-agents-cli-eval` | 評価方法論 — メトリクス、データセット、LLM-as-judge、適応型ルーブリック |
| `google-agents-cli-deploy` | デプロイ — [Agent Runtime](https://docs.cloud.google.com/gemini-enterprise-agent-platform/scale)、[Cloud Run](https://cloud.google.com/run)、[GKE](https://cloud.google.com/kubernetes-engine)、CI/CD、シークレット |
| `google-agents-cli-publish` | Gemini Enterpriseへの登録 |
| `google-agents-cli-observability` | 可観測性 (Observability) — Cloud Trace、ロギング、サードパーティ統合 |

---

## CLIコマンド

| コマンド | 概要 |
|---------|-------------|
| `agents-cli setup` | CLIおよびスキルをコーディングエージェントにインストール |
| `agents-cli scaffold <name>` | 新しいエージェントプロジェクトを作成 |
| `agents-cli eval generate` | 評価データセットでエージェントを実行し, トレースを生成 |
| `agents-cli eval grade` | 生成されたトレースに対してエージェント評価を実行 |
| `agents-cli deploy` | Google Cloudへデプロイ |
| `agents-cli publish gemini-enterprise` | Gemini Enterpriseに登録 |

<details>
<summary>すべてのコマンドを表示</summary>

| コマンド | 説明 |
|---------|-------------|
| `agents-cli login` | Google CloudまたはAI Studioで認証 |
| `agents-cli login --status` | 認証ステータスを表示 |
| **Scaffold** | |
| `agents-cli scaffold <name>` | 新しいエージェントプロジェクトを作成 |
| `agents-cli scaffold enhance` | 既存プロジェクトにデプロイ、CI/CD、またはRAGを追加 |
| `agents-cli scaffold upgrade` | プロジェクトを新しいagents-cliバージョンにアップグレード |
| **Develop** | |
| `agents-cli run "prompt"` | 単一プロンプトでエージェントを実行 |
| `agents-cli install` | プロジェクトの依存関係をインストール |
| `agents-cli lint` | コード品質チェックを実行 (Ruff) |
| **Evaluate** | |
| `agents-cli eval generate` | 評価ケース全体でエージェントの推論を実行 |
| `agents-cli eval grade` | 生成されたトレースをメトリクスに基づいて採点 |
| `agents-cli eval dataset synthesize` | ローカルエージェント用のマルチターン評価シナリオを合成 |
| `agents-cli eval compare` | 2つの評価結果ファイルを比較 |
| `agents-cli eval analyze` | 採点結果から失敗モードをクラスタリング |
| `agents-cli eval metric list` | 利用可能なメトリクスを一覧表示 |
| `agents-cli eval optimize` | 評価データを使用してエージェントプロンプトを自動チューニング |
| **Deploy & Publish** | |
| `agents-cli deploy` | Google Cloudへデプロイ |
| `agents-cli publish gemini-enterprise` | Gemini Enterpriseに登録 |
| `agents-cli infra single-project` | 単一プロジェクトのインフラストラクチャをプロビジョニング |
| `agents-cli infra cicd` | CI/CDパイプラインおよびステージング/本番インフラストラクチャをセットアップ |
| **Data** | |
| `agents-cli infra datastore` | RAG用のデータストアインフラストラクチャをプロビジョニング |
| `agents-cli data-ingestion` | データ取り込みパイプラインを実行 |
| **Other** | |
| `agents-cli info` | プロジェクト構成とCLIバージョンを表示 |
| `agents-cli update` | すべてのIDEにスキルを強制再インストール |

</details>

## 仕組み

<div align="center">
  <a href="https://youtu.be/ECYKo70pPNc">
    <img src="https://img.youtube.com/vi/ECYKo70pPNc/maxresdefault.jpg" alt="agents-cli demo video" width="100%" />
  </a>
</div>

---

## アーキテクチャ

`agents-cli`が構築するGoogle Cloudエージェントスタック:

![Architecture](https://raw.githubusercontent.com/google/agents-cli/main/docs/src/assets/architecture.png "Architecture")

## FAQ (よくある質問)

**これは Antigravity CLI、Claude Code、Codex の代替ツールですか？**<br>
いいえ。**`agents-cli`はコーディングエージェントの*ための*ツールであり、コーディングエージェント自体ではありません。** コーディングエージェントがGoogle Cloud上でADKエージェントを構築、評価、デプロイする作業を効率化するためのCLIコマンドとスキルを提供します。

**直接 `adk` を使用するのとどう違いますか？**<br>
[ADK](https://adk.dev) はエージェントフレームワークです。`agents-cli`はコーディングエージェントに、ADKエージェントをエンドツーエンドで構築、評価、デプロイするためのスキルとツールを提供します。

**Google Cloudは必要ですか？**<br>
ローカル開発 (`create`、`run`、`eval`) の場合は不要です。[AI Studio APIキー](https://aistudio.google.com/apikey)を使用して、ローカルで[ADK](https://adk.dev)とともにGeminiを実行できます。デプロイやクラウド機能を利用する場合は必要です。

**既存のエージェントプロジェクトで使用できますか？**<br>
はい。`agents-cli scaffold enhance` を使用すると、既存のプロジェクトにデプロイとCI/CDを追加できます。

**コーディングエージェントなしで `agents-cli` を使用できますか？**<br>
はい。CLIはスタンドアロンで動作します。ターミナルから直接 `agents-cli scaffold`、`eval`、`deploy` などのすべてのコマンドを実行できます。スキルはコーディングエージェントがこれらの作業を行うのを容易にするためのものです。

**他のスキルで `agents-cli` を拡張するにはどうすればよいですか？**<br>
`agents-cli` のスキルは、エージェント構築のライフサイクル (スキャフォールディング、ADKコードパターン、評価、デプロイ、公開、可観測性) をカバーしています。関連するその他の関心事については、併せて別のスキルスイートをインストールできます。たとえば、[agent-skills](https://github.com/addyosmani/agent-skills) は一般的なソフトウェアエンジニアリングワークフロー (アイデア出し、仕様ゲート、計画、コードレビュー) をカバーし、[google/skills](https://github.com/google/skills) はGoogle Cloudの基礎 (BigQuery、Cloud Run、Firebase、GKE) をカバーしています。

## フィードバック

コミュニティのために `agents-cli` を改善できるよう、皆様からのご意見をお待ちしております。

- **バグ報告 & 機能リクエスト:** [Issueを作成](https://github.com/google/agents-cli/issues/new) — 優先してほしいものには 👍 を付けてください
- **成果物の共有:** 皆様のプロジェクトについての報告をお待ちしています！エージェントの共有やフィードバックの提供については <a href="mailto:agents-cli@google.com">agents-cli@google.com</a> までご連絡ください

## 貢献

貢献の最良の方法はフィードバックです。[Issue](https://github.com/google/agents-cli/issues) を通じてバグ報告、機能リクエスト、アイデアを共有していただくことで、ロードマップの策定に直接ご協力いただけます。

詳細については[貢献ガイド](CONTRIBUTING.md)を参照してください。

## 利用規約

`agents-cli` は Google Cloud API を利用します。エージェントをデプロイすると、ご自身の Google Cloud プロジェクト内にリソースがデプロイされ、それらのリソースに対して責任を負うことになります。詳細については [Google Cloud サービス利用規約](https://cloud.google.com/terms/service-terms) をご確認ください。
