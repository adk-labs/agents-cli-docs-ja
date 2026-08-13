# ユースケース

Agents CLI は、コーディングエージェントに提示する説明に基づいてエージェントをスキャフォールディング、評価、デプロイします。以下のようなエージェントの構築に利用できます:

- **定期実行ボット:** RSS フィードからデータを取得し、LLM で結果を要約して、Cloud Scheduler のトリガーで Google Chat やメールに発行します。
- **調査エージェント:** ログを読み取り、デプロイをトレースし、過去のインシデントと結果を関連付けて根本原因分析を作成します。
- **ナレッジエージェント:** 会話、メール、設計ドキュメントをインデックス化し、類似のテーマが再発した際に過去の決定事項を検索できるようにします。
- **A2A マルチエージェントシステム:** インシデント対応、コード移行、監査などの領域でスペシャリストエージェントを調整・連携させます。

---

## パターンを選択する

<table class="use-case-grid">
<tr>
<td align="center" width="33%"><h3><a href="#日次ニュースボット">日次ニュースボット</a></h3></td>
<td align="center" width="33%"><h3><a href="#業界動向ウォッチ">業界動向ウォッチ</a></h3></td>
<td align="center" width="33%"><h3><a href="#自己調整型サポート">自己調整型サポート</a></h3></td>
</tr>
<tr>
<td align="center"><h3><a href="#技術調査エージェント">技術調査エージェント</a></h3></td>
<td align="center"><h3><a href="#回帰検知器">回帰検知器</a></h3></td>
<td align="center"><h3><a href="#組織のナレッジメモリ">組織のナレッジメモリ</a></h3></td>
</tr>
<tr>
<td align="center"><h3><a href="#組織ナレッジナビゲーター">組織ナレッジナビゲーター</a></h3></td>
<td align="center"><h3><a href="#デューデリジェンス">デューデリジェンス</a></h3></td>
<td align="center"><h3><a href="#セキュリティ監査">セキュリティ監査</a></h3></td>
</tr>
<tr>
<td align="center"><h3><a href="#rfp-レスポンス生成">RFP レスポンス生成</a></h3></td>
<td align="center"><h3><a href="#インシデント対応オーケストレーション">インシデント対応</a></h3></td>
<td align="center"><h3><a href="#分散コード移行">分散コード移行</a></h3></td>
</tr>
</table>

!!! note "現時点で未対応"

    - **リアルタイム音声およびビデオ**
    - **Python 以外のエージェント** (Go、Java、TypeScript)
    - **マルチクラウド デプロイ** — Google Cloud に特化。他のクラウドとのやり取りにはカスタムのインフラストラクチャやスキルが必要になる場合があります

---

## 初級 (Beginner)

エージェント間連携を伴わない単一エージェントパターン。最初のプロジェクトに最適です。

### 日次ニュースボット

*初級 · `adk`*

設定された一連の RSS フィードからニュースの見出しを取得し、LLM で最も関連性の高い記事を選択して、Google Chat やメールに発行します。Cloud Scheduler でスケジュール実行します。

```
Build me a daily news bot that pulls these RSS feeds, summarizes the top 5 stories, and posts to Google Chat every morning.
```

スケジュール実行と展開については [デプロイ](deployment.md) および [CI/CD](cicd.md) を参照してください。

### 業界動向ウォッチ

*初級 · `adk`*

業界全体の公開リリースノート、ドキュメントの更新、求人情報、カンファレンストークを追跡します。リリースされた機能や採用傾向を表面化させます。週ごとのレビューのために、クエリ可能なストアに調査結果を保存します。

```
Track these companies' public docs, releases, and job postings daily. Surface shipped features and hiring trends.
```

---

## 中級 (Intermediate)

フィードバックループ、RAG (検索拡張生成)、または本格的なツール統合を備えた単一エージェント。

### 自己調整型サポート

*中級 · `adk`*

会話ごとに評価を実行し、知識や動作のギャップを特定して、弱かった回答に対して新しい評価ケースをドラフトします。顧客が実際に尋ねる質問に合わせてカバレッジが適応します。

```
Build a support agent that runs eval after each conversation, drafts new eval cases for weak answers, and surfaces documentation gaps.
```

[評価ガイド](evaluation.md) では評価・修正ループについて解説しています。本番環境のトレースをリプレイするには [可観測性](observability/index.md) と組み合わせます。

### 技術調査エージェント

*中級 · `adk`*

「先月、決済サービスのレイテンシが増加したのはなぜか？」といった質問を受け付けます。ログを読み取り、デプロイをトレースし、過去のインシデントと関連付けます。タイムラインと根本原因分析を作成します。

```
Build an investigation agent. I ask questions like "why did X break last week" and it pulls from logs, deploy history, and past incidents to produce a writeup.
```

### 回帰検知器

*中級 · `adk`*

現在のメトリクスやログパターンを、過去のインシデント前のシグネチャと比較します。現在の動作が既知の機能退行パターンと一致した場合に予防的な Issue を起票します。夜間スケジュールで実行します。

```
Build an agent that runs nightly, looks for metric/log patterns that match historical pre-incident signatures, and files preventive bugs.
```

### 組織のナレッジメモリ

*中級 · RAG レシピ*

決定記録のために Google Chat、メール、設計ドキュメント、会議メモをインデックス化します。提案が再浮上した際 (例:「セッションに Redis を使用する」など)、元のスレッドとチームが到達した決定事項を表示します。

```
Build a RAG agent that indexes Google Chat, email, and design docs nightly. Surface past decisions when someone proposes something we've already discussed.
```

RAG はクローンして学習するレシピです — 適応させるサンプルについては [エージェントテンプレート](templates.md#rag-retrieval-augmented-generation) を参照してください。

### 組織ナレッジナビゲーター

*中級 · RAG レシピ · Gemini Enterprise*

Google ドライブ、Google Chat、メールへの権限付きアクセス権を持って Gemini Enterprise にデプロイします。「本番データベースへのアクセス権を取得するにはどうすればよいか？」といった質問に対し、ドキュメント化されたプロセスと実際の運用の現状の両方を添えて回答します。

```
Build a RAG agent for new-hire questions that knows both official docs and how things actually work. Publish it to Gemini Enterprise.
```

登録の詳細については [`google-agents-cli-publish`](../reference/skills.md) スキルを参照してください。

---

## 上級 (Advanced)

長時間実行されるワークフローまたはマルチエージェントオーケストレーション。専用のインフラストラクチャと拡張された開発が必要です。

### デューデリジェンス

*上級 · RAG レシピ*

約 50 万行の対象コードベースをインデックス化します。技術的負債、セキュリティの脆弱性、ライセンスのコンプライアンス、デプロイの複雑さを分析します。行番号、依存関係グラフ、CVE 参照を含むリスクレポートを作成します。数日間にわたる分析には、Agent Runtime の拡張セッションとチェックポイント機能が効果的です。

```
Build a due-diligence agent that indexes a target codebase, runs security and license scans, and produces a risk report with citations.
```

### セキュリティ監査

*上級 · `adk`*

コードベース全体のデータフローをマッピングし、GDPR、HIPAA、SOC2 への準拠を検証します。取り込みから削除までの機密データをトレースします。設定された保持ポリシーを超えてユーザーデータを保持しているアナリティクスログなどのギャップにフラグを立てます。

```
Build a compliance-audit agent that traces sensitive data flows across our codebase and flags retention/policy gaps with file:line citations.
```

監査トレイルの完全性を追跡するには [BigQuery Agent Analytics](observability/bq-agent-analytics.md) を使用します。

### RFP レスポンス生成

*上級 · RAG レシピ*

過去のプロジェクト記録、現在のリソース状況、料金モデルからデータを取得します。タイムラインと予算を見積もります。技術的アプローチをドラフトします。人間がレビューするための提案パッケージを作成します。

```
Build a RAG agent that drafts RFP responses by pulling from past proposals, current resourcing, and pricing models.
```

### インシデント対応オーケストレーション

*上級 (A2A) · `adk`*

障害発生時にスペシャリストエージェントを並行して実行します: 1つは最近の変更を二分探索し、1つはサービス間のエラーを関連付け、1つは過去のインシデントを検索し、1つは顧客向けコミュニケーションをドラフトします。並行調査により、順次トラブルシューティングと比較して原因特定までの時間を短縮できます。

```
Build an A2A multi-agent system for incident response. Specialists for bisection, error correlation, past-incident lookup, and customer comms — coordinated in parallel.
```

[`adk` テンプレート](templates.md) (A2A 組込済み) は A2A プロトコルを公開します。各スペシャリストはサービスとして実行され、コーディネーターが実行をプロデュースします。

### 分散コード移行

*上級 (A2A) · `adk`*

大規模なフレームワーク移行のためにスペシャリストエージェントを実行します: 1つはデータモデルを処理し、1つは API コントラクトを処理し、1つはテストを処理し、1つは検証を処理します。スペシャリストは A2A 経由で調整し、破壊的変更に関する調査結果を共有します。多数の並行スペシャリストインスタンスを実行する場合は GKE が推奨ランタイムです。

```
Build A2A specialist agents for a large framework migration: data models, API contracts, tests, validation.
```

---

## 次のステップ

- [チュートリアル: 最初のエージェントを構築する](quickstart-tutorial.md) — コーディングエージェントを使用した構築、評価、デプロイ
- [プロジェクト構造](project-structure.md) — 生成された各ファイルの役割を理解する
- [エージェントテンプレート](templates.md) — `adk` テンプレートと RAG クローン＆学習レシピ
- [開発ガイド](development.md) — 開発ワークフローの詳細
- [CLI リファレンス](../cli/index.md) — すべてのコマンドとフラグ
