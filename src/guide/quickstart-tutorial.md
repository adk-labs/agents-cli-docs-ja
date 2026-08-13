# チュートリアル: 最初のエージェントを構築する

*コーディングエージェントを使用してエージェントを構築、評価、デプロイしたい初心者向け。*

このチュートリアルでは、Agents CLI in Agent Platform の体験全体を紹介します — コーディングエージェントと会話するだけで、エージェントがあなたの代わりに ADK エージェントを構築、評価、デプロイします。

作成するのは **caveman compressor (原始人風圧縮機)** です: 冗長なテキストを受け取り、簡潔で原始人スタイルの要約に圧縮するエージェントです。[caveman](https://github.com/JuliusBrussee/caveman) に着想を得ています。

エンドツーエンドでの動作イメージは以下の通りです:

![agents-cli demo](https://raw.githubusercontent.com/google/agents-cli/assets/agents-cli-demo.gif)

---

## セットアップ

ご自身で実行するコマンドはこれだけです。それ以外はすべてコーディングエージェント経由で処理されます。

```bash
uvx google-agents-cli setup
```

その後、お使いのコーディングエージェント — [Antigravity CLI](https://antigravity.google/)、[Claude Code](https://docs.anthropic.com/en/docs/claude-code)、[Codex](https://github.com/openai/codex)、その他 — を開きます。

---

## 1. スキャフォールド

コーディングエージェントに以下のように伝えます:

> *「agents-cliを使って、冗長なテキストを簡潔で技術的な用語に圧縮する原始人スタイルのエージェントを作成してください」*

コーディングエージェントは `google-agents-cli-workflow` および `google-agents-cli-scaffold` スキルをアクティブ化します。エージェントは以下の処理を行います:

- 確認のための質問 (デプロイターゲット、セーフティ制約など)
- エージェントの目的をキャプチャした `.agents-cli-spec.md` に仕様を保存
- プロジェクトをスキャフォールド:

```
agents-cli create caveman-agent --prototype --yes
cd caveman-agent && agents-cli install
```

これで、ボイラープレートのエージェントコード、テスト、評価セットが含まれる実用的なプロジェクトが作成されます。

---

## 2. 構築

コーディングエージェントが `app/agent.py` を編集し、デフォルトのエージェントを洞窟男圧縮エージェントに置き換えます。ADK パターンのために `google-agents-cli-adk-code` スキルを使用します。

エージェントの定義は概ね以下のようになります:

```python title="app/agent.py"
root_agent = Agent(
    name="caveman_agent",
    model=Gemini(model="gemini-3.6-flash"),
    instruction="""You caveman compressor. Human give long words,
    you make short. Rules:
    - No articles. No filler. No fluff.
    - Short grunts. Simple words.
    - Keep technical terms but grunt around them.
    - Funny but meaning stays.

    Example input:  "I would like to deploy the application to production"
    Example output: "Me deploy. Production. Now."
    """,
)
```

その後、コーディングエージェントが動作確認 (スモークテスト) を行います:

```
agents-cli run "Please help me understand the deployment options available for my project"
```

出力結果:

```
Deploy options: Agent Runtime, Cloud Run, GKE. Pick one. Ship.
```

---

## 3. 評価

コーディングエージェントに以下のように伝えます:

> *「caveman エージェントの評価テストを作成して実行してください」*

コーディングエージェントは `google-agents-cli-eval` スキルをアクティブ化し、以下を行います:

- テストケース (圧縮品質、技術用語の保持、原始人トーン) を含む `tests/eval/datasets/caveman-dataset.json` を作成
- 評価の実行:

```bash
agents-cli eval generate
agents-cli eval grade
```

ケースが失敗した場合は、修正内容をコーディングエージェントに伝えます:

> *「挨拶テストへの返答が丁寧すぎます。もっと原始人らしくしてください。」*

コーディングエージェントは指示 (instruction) を調整し、評価を再実行して、品質しきい値をクリアするまで反復します。

評価機能は `generate` や `grade` にとどまりません — `eval dataset synthesize`、`eval compare`、`eval analyze`、および `eval optimize` は、合成ケース生成、回帰差分、失敗クラスタリング、プロンプトの自動チューニングをカバーします。すべての機能については [評価ガイド](evaluation.md#beyond-generate-and-grade) を参照してください。

---

## 4. デプロイ

コーディングエージェントに以下のように伝えます:

> *「これを Cloud Run にデプロイしてください」*

コーディングエージェントは `google-agents-cli-deploy` スキルをアクティブ化し、以下を行います:

- デプロイインフラストラクチャを追加:

```
agents-cli scaffold enhance --deployment-target cloud_run
```

- デプロイを実行:

```
agents-cli deploy
```

これで洞窟男エージェントが本番環境で稼働し、出力に Cloud Run の URL が表示されます。

---

## 5. 可観測性

Cloud Trace はデフォルトで有効になっているため、セットアップは不要です。Google Cloud Console で [Trace explorer](https://console.cloud.google.com/traces) を開き、エージェントにリクエストを送信してください。各 LLM 呼び出しやツール実行のスパンが表示されます。

さらに進んで、本番環境でエージェントが処理する実際のプロンプトとレスポンスを検証したい場合は、コーディングエージェントに伝えます:

> *「エージェントの可観測性インフラストラクチャをセットアップしてください」*

コーディングエージェントは `infra single-project` を実行し、サービスアカウント、GCS バケット、BigQuery データセットをプロビジョニングして、デプロイ済みサービスを更新してそれらを使用するようにします。検証手順と詳細オプションについては [可観測性ガイド](observability/index.md) を参照してください。

---

## 内部で起きたこと

各プロンプトによって内部でトリガーされた処理のまとめです:

| あなたの指示 | コーディングエージェントの処理 |
|----------|----------------------|
| *「caveman 圧縮エージェントを作成して」* | プロジェクトをスキャフォールドし、エージェントコードを記述、ローカルでテスト |
| *「評価を作成して実行して」* | データセットを作成し、`generate` と `grade` で評価を実行 |
| *「Cloud Run にデプロイして」* | デプロイターゲットを追加し、Cloud Run へデプロイ |
| *「可観測性をセットアップして」* | サービスアカウント、GCS バケット、BigQuery データセットをプロビジョニング |

スキルにより、コーディングエージェントは各ステップで正しい判断を行うコンテキストを得ます (使用すべき ADK パターン、評価の構造化方法、渡すべきデプロイターゲットフラグなど)。

---

## 次のステップ

より複雑なエージェントの構築に挑戦してみてください:

- ツールの追加 — *「Google Search ツールを追加して、洞窟男が時事ニュースについて呟けるようにして」*
- マルチエージェント — *「他のエージェントと対話できる A2A エージェントを作成して」* (`adk` テンプレートを使用 — A2A は組込済み)
- RAG — *「ドキュメントに関する質問に答えるエージェントを構築して」* (RAG サンプルをクローンして学習 — [エージェントテンプレート](templates.md#rag-retrieval-augmented-generation) を参照)

すべてのオプションについては [エージェントテンプレート](templates.md) を参照するか、完全なワークフローについて [開発ガイド](development.md) へ進んでください。
