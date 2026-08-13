# エージェントテンプレート

`agents-cli` はエージェントテンプレートからプロジェクトを作成します。各テンプレートは、用途に応じた適切な依存関係、ツール、およびプロジェクト構造を備えた動作可能なエージェントを提供します。

---

## 利用可能なテンプレート

| テンプレート | 説明 | ユースケース |
|---|---|---|
| `adk` | ADK を使用した ReAct エージェント | ツール利用を伴う汎用会話型エージェント |

> **RAG はテンプレートではありません** — クローンして学習するためのレシピです。以下の [RAG](#rag-retrieval-augmented-generation) を参照してください。

### adk

デフォルトのテンプレートです。[Agent Development Kit](https://google.github.io/adk-docs/) を使用し、サンプルツールを含んだ ReAct エージェントを作成します。ADK が初めての方や、汎用エージェントを構築する場合は、ここから始めてください。

```bash
agents-cli create my-agent --agent adk
```

すべての Python ADK エージェントは、開梱後すぐに [Agent-to-Agent (A2A) プロトコル](https://a2a-protocol.org) を提供します — A2A ルート（エージェントカード + JSON-RPC）は自動的にマウントされます。エージェントが他のフレームワーク（LangGraph、CrewAI など）で構築されたエージェントと相互運用する必要がある場合や、分散マルチエージェントシステムを構築する場合に使用してください。個別のテンプレートや手動で記述する A2A コードは不要です。

## RAG (Retrieval-Augmented Generation)

RAG はテンプレートでは**ありません** — クローンして学習するためのレシピです。ベースとなる `adk` プロジェクトをスキャフォールディングし、[google/adk-samples](https://github.com/google/adk-samples) 内の RAG サンプルのいずれかを学習・適用して、リトリーバーと `infra/terraform/` をプロジェクトにコピーします:

- **`rag-vector-search`** — カスタム取り込みパイプライン（エンベディング、類似性検索）を備えた Vertex AI Vector Search 2.0。
- **`rag-agent-search`** — 完全管理型の GCS データコネクタを備えた Agent Platform Search (Discovery Engine) — バケットにファイルを格納するだけで、取り込みコードを記述する必要はありません。

ADK コードスキルの `references/samples.md` には、主要ファイルとともに両方が記載されており、各サンプルの `AGENTS.md` が学習および適用のガイドとなります。プロビジョニングと取り込みは、サンプル自体の `Makefile`（`make setup-infra`、`make data-ingestion`）から実行します。
