# ライフサイクル

Agents CLI は1つの重要なコンセプトに基づいています。それは**「ノートブック上で良好に動く状態」**から**「本番環境での稼働」**までのループです。このページはその全体像を示すマップです。

## 1つの調査例を見る

障害復旧エージェントを想像してみてください。本番稼働から1週間が経過し、アラートが発報されました:

<div id="lifecycle-anim-transcript" class="lifecycle-anim" aria-label="Auto-playing transcript of an outage investigation"></div>

この調査にかかった時間は **4.3秒** です。*エージェント自体*に特別な点はありません — ほとんどのエージェントフレームワークで表現可能です。特別なのはその周辺のすべての仕組みです。破滅的な修復を推奨した場合には出荷を許可しない評価ルーブリック、ランブック検索が誤ったセクションを返すのを検出する CI チェック、明日何か問題が発生した際にこの調査をそのままリプレイできるトレースなどです。

これがループの真価です。

## 循環する4つのCLI動詞

<div id="lifecycle-anim-loop" class="lifecycle-anim" aria-label="The four CLI verbs in a continuous loop"></div>

`scaffold`、`eval`、`deploy`、`observe` — これらが永遠に循環します。あなたが仕様を記述し、このループがリリースすべきでないものを検知し、テストをパスしたものをデプロイし、次の反復がよりスマートになるようその後の挙動を可視化します。

## ループがない場合に発生する問題

ほとんどのエージェントのデモはプロンプトの作成で終わります。巧みな指示を書き、モデルがノートブック上で見栄えのする結果を返し、それをチーム用にスクリーンショットします。しかし、本番環境へのデプロイには現実世界の課題が伴います。

| | ループがない場合 | Agents CLI を使用する場合 |
|---|---|---|
| **ハルシネーションによる修復** | 発覚は顧客側での発生後 | 評価ルーブリックがマージ前に PR をブロック |
| **ツール API の変更** | 午前2時の障害対応、エージェントはサイレントに破損 | CI 統合テストがスキーマの乖離を検出 |
| **本番環境での誤用** | リプレイ不可、テレメトリなし | Cloud Trace + BigQuery 分析により1時間以内に表面化 |
| **喋りすぎなツールによるコスト急増** | 来月の請求書が最初のアラート | ツールごとのスパンカウントによりループを数時間で発見 |

## 8つのフェーズ

ゆっくりたどると、ループは8つのフェーズに拡張されます。各フェーズには [スキル](../reference/skills.md) にエンコードされたベストプラクティスがあり、コーディングエージェントが最適な選択を行えるようサポートします。

| # | フェーズ | 概要 | CLI動詞 | スキル | 詳細 |
|---|---|---|---|---|---|
| 0 | **仕様 (Spec)** | `.agents-cli-spec.md` を記述します。他のフェーズはここから派生します。 | — | `google-agents-cli-workflow` | [開発ガイド](development.md) |
| 1 | **スキャフォールド (Scaffold)** | 仕様を本番構成のプロジェクト (~72ファイル) に変換します。 | `scaffold create` | `google-agents-cli-scaffold` | [テンプレート](templates.md) |
| 2 | **構築 (Build)** | エージェント本体 (モデル、指示、ツール、`App` ラッパー) を作成します。 | — | `google-agents-cli-adk-code` | [プロジェクト構造](project-structure.md) |
| 3 | **オーケストレーション (Orchestrate)** | 1つのエージェントがチームに拡大した際、スペシャリストを構成します。 | — | `google-agents-cli-adk-code` | [プロジェクト構造](project-structure.md) |
| 4 | **評価 (Evaluate)** | デプロイごとにデータセットに対してエージェントを採点します。 | `eval generate`、`eval grade`、さらに `eval dataset synthesize`、`eval compare`、`eval analyze`、`eval metric list`、`eval optimize` | `google-agents-cli-eval` | [評価](evaluation.md) |
| 5 | **デプロイ (Deploy)** | Agent Runtime、Cloud Run、または GKE にリリースします。 | `deploy` | `google-agents-cli-deploy` | [デプロイ](deployment.md) |
| 6 | **公開 (Publish)** | 他のエージェントがこれを見つけられるよう Gemini Enterprise に登録します。 | `publish` | `google-agents-cli-publish` | [CI/CD](cicd.md) |
| 7 | **可観測性 (Observe)** | Cloud Trace + BigQuery 分析。本番データが明日のデータセットを補給します。 | — | `google-agents-cli-observability` | [可観測性](observability/index.md) |

### 0 · 仕様 (Spec)

`.agents-cli-spec.md` にエージェントのツール、制約事項、成功基準を定義します。ライフサイクルの残りのすべてのフェーズがこのファイルを参照します (スキャフォールドのフラグ、評価ルーブリック、セーフティガードレール、本番環境で監視するトレース属性など)。白紙から始める必要はありません。[Agent Garden](https://docs.cloud.google.com/gemini-enterprise-agent-platform/build/agent-garden) で目的に近い既存テンプレートを閲覧し、カスタマイズしてください。

一般的な仕様書は、1画面に収まる程度の Markdown です:

```markdown
# .agents-cli-spec.md — outage-recovery-bot

## Tools

| Tool                                    | Backing service       |
| --------------------------------------- | --------------------- |
| `query_logs(service, severity)`         | Cloud Logging         |
| `check_metrics(service, metric)`        | Cloud Monitoring      |
| `search_runbook(query)`                 | Vector Search         |

## Constraints

1. 参照したランブックのセクションを常に引用すること。
2. 観察された症状に対してランブックが明示的に認可していない限り、
   破滅的な修復手順を絶対に推奨しないこと。

## Success criteria

- インシデントの 80% 以上で、根本原因が正解データと一致する診断を得ること
- 推奨事項の 100% がランブックのセクションを引用していること
- ランブックの認可なしの破滅的な推奨事項が 0 件であること
```

### 1 · スキャフォールド (Scaffold)

1つのコマンドで仕様を読み込み、プロジェクトを出力します (エージェントコード、テスト、評価ボイラープレート、Terraform、CI/CD ワークフロー、デプロイマニフェスト)。フラグは不要についているわけではなく、各フラグが選択したライフサイクルに合わせてスキャフォールドを拡張または縮小します。

<div id="lifecycle-anim-scaffold" class="lifecycle-anim" aria-label="Scaffold wizard — toggle flags, watch the command and file count update"></div>

完全なセットアップでは、エージェントコード、評価ボイラープレート、Terraform、GitHub Actions ワークフロー、デプロイマニフェストなど **約72個のファイル** が生成されます。不要な部分をスキップしてスリム化することも可能です。全リストについては [テンプレート](templates.md) を参照してください。

### 2 · 構築 (Build)

すべての ADK エージェントは、モデル、指示 (instruction)、ツール一覧、およびそれらをラップする `App` の4つの要素に集約されます。エージェントの本体はわずか30行程度のコードであり、重要な処理はツール内部で行われます。

```python
from google.adk.agents import Agent
from google.adk.apps import App
from google.adk.models import Gemini

root_agent = Agent(
    name="root_agent",
    model=Gemini(model="gemini-3.6-flash"),
    instruction="You are an SRE outage-recovery assistant...",
    tools=[query_logs, check_metrics, search_runbook],
)

app = App(root_agent=root_agent, name="app")
```

Gemini に限定されるわけではありません — モデルの行を ADK がサポートする任意のプロバイダに変更できます ([Model Garden](https://cloud.google.com/model-garden) では Anthropic Claude、OpenAI GPT などがカバーされています)。ライフサイクルの残りの部分は、プロバイダに関係なく同一に動作します。

ステートフルなエージェントは、Agent Platform の追加の2つの機能を活用できます:

- **マネージドセッションストレージ**: 再起動後も生存し、水平スケール可能な会話状態を提供します — インメモリのデフォルトの代わりに、スキャフォールド時に `--session-type agent_platform_sessions` を指定して選択します。
- **[Memory Bank](https://cloud.google.com/agent-builder/docs/memory)**: セッションを跨いだ*長期*メモリを提供します (SRE ボットが「これは前四半期のインシデントに似ている」と認識するなど)。`from google.adk.memory import VertexAiMemoryBankService` 経由で接続すると、ユーザー、セッション、またはアプリに紐づく永続ストレージが利用可能になります。

単一の HTTP リクエストに収まらないワークフロー (長時間の調査、マルチステップのバッチ処理) の場合、Agent Runtime がエージェントの状態を保持するため、デプロイや再起動によって進行状況が失われることはありません。

<div id="lifecycle-anim-models" class="lifecycle-anim" aria-label="Same prompt, three model providers — illustrative side-by-side"></div>

以下は、同じエージェント本体が別のインシデントにエンドツーエンドで対応する例です:

<div id="lifecycle-anim-playground" class="lifecycle-anim" aria-label="Inline playground — payments triage scenario, click to step through"></div>

### 3 · オーケストレーション (Orchestrate)

単一のエージェント構成は問題の規模が小さい場合に有効です。本番環境の現実的なエージェントは、オーケストレーターがそれぞれの狭いツール領域を持つスペシャリストに作業をルーティングする**チーム**へと成長します。

<div id="lifecycle-anim-team" class="lifecycle-anim" aria-label="Team diagram — orchestrator routes work to investigator, diagnoser, and remediator"></div>

チーム分割は、評価、デプロイ、可観測性の観点から3つの理由で役立ちます。プロンプトを小さくすることで各エージェントの信頼性が向上し、ツール領域を分割することでエージェントごとのガードレールを適用でき、トレースによってどのサブエージェントが誤った判断をしたかを正確に特定できます。

チームが複数プロセスにまたがる場合、あるいは自チームが所有していないエージェントを呼び出す場合は、ワイヤーフォーマットとして **[A2A プロトコル](https://a2a-protocol.org/)** を使用します。A2A はすべての ADK エージェントに組み込まれているため、通常通りスキャフォールドするだけです (`--agent adk`)。Agents CLI で構築されたかどうかにかかわらず、すべての A2A 互換エージェントがあなたのエージェントを呼び出せ、またあなたのエージェントから呼び出すことができます。

### 4 · 評価 (Evaluate)

これはほとんどのエージェントデモで省略されがちなフェーズです。`agents-cli eval generate` に続けて `agents-cli eval grade` を実行することで、稼働中のエージェントに対してデータセットを実行し、LLM ジャッジにルーブリックに基づいて各レスポンスを評価させ、自信を持って説明できるスコアを取得できます。

<div id="lifecycle-anim-eval" class="lifecycle-anim" aria-label="Eval-fix loop — click 'apply fix' to see one case flip from failing to passing"></div>

`agents-cli eval grade` ループの反復は 5〜10回以上になることが予想されます。修正を行うたびにスコアが向上し、再実行して、しきい値を超えた時点でデプロイします。以下は、ルーブリックが最も頻繁に検出する4つの失敗モードです。

<div id="lifecycle-anim-failures" class="lifecycle-anim" aria-label="Common agent failures and the eval rubric that catches each"></div>

メトリクス、データセットスキーマ、および詳細な方法論については [評価ガイド](evaluation.md) を参照してください。

### 5 · デプロイ (Deploy)

同じエージェントコードを3つの異なる環境にデプロイできます。`agents-cli deploy` は、スキャフォールド時に指定されたターゲットに基づいて処理を実行します。**ターゲットを選択して `--dry-run` が出力する内容と後続のステップを確認してください:**

<div id="lifecycle-anim-deploy" class="lifecycle-anim" aria-label="Deploy target picker — choose a runtime to see the dry-run + pipeline"></div>

```bash
agents-cli deploy --dry-run        # パイプラインをプレビュー
agents-cli deploy                  # デプロイを実行
agents-cli deploy --no-wait        # 即座に復帰。後で --status で確認
```

各ターゲットは周辺の本番環境用プリミティブを継承します:

- **エージェントごとのサービスアカウント** — `agents-cli deploy --agent-identity` で有効化すると、デプロイされたエージェントは固有の GCP アイデンティティとして実行されます。通常の IAM を使用して、実際に呼び出せる対象 (BigQuery データセット、バケット、API) を制限できます。破滅的な修復をブロックする評価ルーブリックにはセーフティネットがあります。アイデンティティに権限がない場合、エージェントは物理的に `kubectl delete` を実行できません。
- **[Identity-Aware Proxy (IAP)](https://cloud.google.com/iap)** — `--iap` フラグを使用して、Cloud Run デプロイを Google Workspace SSO の保護下に置くことができます。社内専用エージェントがパブリックインターネットに露出する懸念がなくなります。
- **[Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation)** — スキャフォールドされた `pr_checks.yaml` は WIF 経由で GitHub Actions を GCP に対して認証するため、リポジトリ内にサービスアカウントキーを保持する必要がありません。

ターゲットごとの詳細な手順については [デプロイ](deployment.md) を参照してください。

### 6 · 公開 (Publish)

エージェントをデプロイすると URL 経由でアクセス可能になります。公開 (Publish) は別ステップであり、他のエージェント (またはカタログを閲覧する人間) が見つけられるように Gemini Enterprise に登録する処理です。

<div id="lifecycle-anim-publish" class="lifecycle-anim" aria-label="The agent's listing in Gemini Enterprise after publish"></div>

2つの登録モードがあります: **ADK** (デプロイされた Agent Runtime インスタンスを公開) および **[A2A](https://a2a-protocol.org/)** (A2A 互換の HTTP エンドポイントを公開、ADK は不要 — 任意のフレームワークで構築されたエージェントで動作)。

### 7 · 可観測性 (Observe)

エージェントが本番稼働すると、すべての呼び出しによって Cloud Trace スパンが送信されます。すべてのツール呼び出し、モデル生成、サブエージェントへのハンドオフが可視化されます。**下のスパンにホバーして属性を確認してください。**

<div id="lifecycle-anim-trace" class="lifecycle-anim" aria-label="Trace waterfall — bars draw in left-to-right showing the orchestrator and its sub-agents; hover to inspect"></div>

可観測性は本番環境で動作するすべてのエージェントにとって不可欠です。評価で見落とした機能退行、喋りすぎなツールによるコスト急増、ユーザーによるセーフティプロンプトの回避などをキャッチするのに役立ちます。スキャフォールド時に `--bq-analytics` を有効にすると、すべてのプロンプトとレスポンスがオフライン分析用に BigQuery に保存されます。

同じデータがループを締めくくります。本番環境のトラフィックが明日のデータセットを供給します。評価スコアは継続的に再計算されるため、機能退行は数ヶ月ではなく数日で表面化します。

<div id="lifecycle-anim-rolling" class="lifecycle-anim" aria-label="Rolling production eval score over the last ten days, with annotated regression and deploy events"></div>

詳細なセットアップについては [可観測性](observability/index.md) を参照してください。

## 2つの実行方法

<div class="lc-tabs-bare" markdown>

=== "コーディングエージェントに依頼する"

    標準的なアプローチです。コーディングエージェントがスキルを読み込み、各フェーズで適切な CLI コマンドを選択します。

    ```
    Build me an outage-recovery agent. It should investigate incidents
    using logs, metrics, and runbooks, and recommend remediations
    that cite a runbook section. Deploy it to Agent Runtime.
    ```

    コーディングエージェントは以下の手順を実行します:

    1. ツールと制約事項を記述した `.agents-cli-spec.md` を作成
    2. `agents-cli scaffold create … --agent adk --deployment-target agent_runtime` を実行 (RAG はクローン＆学習レシピです — RAG サンプルを適応させてください。[テンプレート](templates.md#rag-retrieval-augmented-generation) を参照)
    3. エージェント本体とツールを作成
    4. データセットケースを作成
    5. `agents-cli eval generate` に続けて `agents-cli eval grade` を実行し、スコアがしきい値を超えるまで `eval grade` で反復処理
    6. `agents-cli deploy` を実行
    7. トレース + アナリティクスを接続し、URL を提示

=== "CLI を自分で操作する"

    すべてのコマンドはスタンドアロンで動作します。自分でコマンドを入力したい場合は、コーディングエージェントを省略できます。

    ```bash
    # フェーズ 1: スキャフォールド
    agents-cli scaffold create outage-recovery-bot \
      --agent adk \
      --deployment-target agent_runtime \
      --cicd-runner github_actions \
      --bq-analytics
    cd outage-recovery-bot && agents-cli install

    # フェーズ 2-3: 構築 & オーケストレーション (app/agent.py を編集)
    agents-cli playground       # :8080 でローカル Web プレイグラウンドを起動

    # フェーズ 4: 評価
    agents-cli eval dataset synthesize --count 10  # オプション: データセットのコールドスタート
    agents-cli eval generate
    agents-cli eval grade                          # 評価スコアがしきい値を超えるまで繰り返す
    agents-cli eval compare prev.json latest.json  # 修正が実際に効果があったか確認
    agents-cli eval analyze --eval-result latest.json  # 残りの失敗モードをクラスタリング
    agents-cli eval optimize                       # オプション: 評価データを使用してプロンプトを自動チューニング

    # フェーズ 5: デプロイ
    agents-cli deploy --dry-run
    agents-cli deploy

    # フェーズ 6: 公開 (オプション)
    agents-cli publish gemini-enterprise
    ```

    エンドツーエンドの完全な手順については、[手動ワークフローチュートリアル](hands-on-tutorial.md) を参照してください。

</div>

## さらに詳しく知るには

- [テンプレート](templates.md) — スキャフォールドテンプレート (`adk`) と RAG クローン＆学習レシピ
- [プロジェクト構造](project-structure.md) — 生成された各ファイルの役割
- [開発ガイド](development.md) — 日常のワークフロー
- [評価ガイド](evaluation.md) — データセットスキーマと評価・修正ループ
- [デプロイ](deployment.md) — ターゲットごとの解説
- [CI/CD & 本番環境](cicd.md) — PR から本番環境への完全なパス
- [可観測性](observability/index.md) — Cloud Trace、BigQuery アナリティクス、サードパーティツール
- [CLI リファレンス](../cli/index.md) — すべてのコマンドとフラグ
