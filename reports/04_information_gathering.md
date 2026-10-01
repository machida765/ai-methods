# AIを活用した情報収集の効率的なやり方 ― ツール選定・自動化・検証・情報源までの実践ガイド（2026年10月版）

> 作成日: 2026年10月1日  
> 対象読者: AI・テック分野の動向を追うビジネスパーソン、エンジニア、研究者、企画・マーケティング担当者  
> 表記ルール: 本文中の製品仕様・料金・機能はWeb検索で確認できた範囲で記載し、確認しきれなかったもの・変動が激しいものには **『要確認』** と付記しています。導入前に必ず公式情報を確認してください。

---

## 目次

- [3分で分かる要約](#summary)
- [第1章 なぜ「AIで情報収集」を設計し直すべきなのか](#ch1)
- [第2章 情報収集の全体アーキテクチャ（5層モデル）](#ch2)
- [第3章 主要ツールの比較と使い分け](#ch3)
  - [3.1 AI検索エンジン（Perplexity / Felo / Genspark）](#ch3-1)
  - [3.2 Deep Research（ChatGPT / Gemini / Claude / Perplexity）](#ch3-2)
  - [3.3 ソース限定型リサーチ（NotebookLM＝Gemini Notebook）](#ch3-3)
  - [3.4 論文向けツール（Elicit / Consensus / SciSpace ほか）](#ch3-4)
  - [3.5 フィード・既読管理（Feedly AI / Readwise Reader / Glasp）](#ch3-5)
  - [3.6 総合比較表と「迷ったらこれ」](#ch3-6)
- [第4章 仕組み化：収集→要約→配信→蓄積の自動パイプライン](#ch4)
  - [4.1 RSS＋LLM要約の基本形（Python）](#ch4-1)
  - [4.2 n8n / Make / Zapier での自動化](#ch4-2)
  - [4.3 Slack / Discord への定期ダイジェスト配信](#ch4-3)
  - [4.4 Obsidian / Notion へのナレッジ蓄積](#ch4-4)
  - [4.5 個人RAGの構築](#ch4-5)
- [第5章 MCPサーバーとSkillsで「調べるエージェント」を作る](#ch5)
- [第6章 情報の信頼性検証（ハルシネーション対策とファクトチェック）](#ch6)
- [第7章 AI分野を追うための情報源・発信者（日英）](#ch7)
- [第8章 目的別レシピ](#ch8)
- [第9章 チェックリスト集](#ch9)
- [付録 プロンプト集・用語集](#appendix)

---

<a id="summary"></a>
## 3分で分かる要約

**結論：AI時代の情報収集は「検索の上手さ」ではなく「パイプラインの設計」で差がつく。**

1. **ツールは役割で使い分ける。** 単発の調べ物は *AI検索（Perplexity、Felo、Genspark）*、数十〜数百ソースを横断する調査は *Deep Research（ChatGPT／Gemini／Claude／Perplexity）*、手元の資料を深掘りするなら *NotebookLM（2026年7月に「Gemini Notebook」へ改称）*、論文は *Elicit／Consensus／SciSpace*、日々の定点観測は *Feedly AI／Readwise Reader／Glasp* が担当、という分業が基本形です。
2. **「毎日見に行く」をやめて「届く」仕組みにする。** RSS・ニュースレター・arXiv・Hacker Newsなどを集約し、LLMで要約・スコアリングしてSlack／Discordに毎朝配信します。n8n（セルフホスト可）、Make、Zapierのいずれでも30〜60分程度で最小構成を作れます（所要時間は構成によります）。
3. **読んだものは「後で検索できる形」で蓄積する。** Obsidian（Markdown）またはNotion（データベース）に、要約・出典URL・取得日・タグを付けて保存します。蓄積がたまったら個人RAG（ベクトル検索＋LLM）で「自分の知識ベースに質問できる」状態にします。
4. **MCPとSkillsでエージェントに調べ物をさせる。** Brave Search、Exa、Tavily、Firecrawl、Fetch などのMCPサーバーをClaude Desktop／Claude Code／Cursor等につなぎ、「調査手順」をSKILL.md（Agent Skills、2025年12月にオープン標準として公開）として定義すると、毎回同じ品質で調査させられます。
5. **検証は省略しない。** AIの出力は「下書き」であり「仮説」です。重要な主張は *一次情報（論文・公式発表・法令・決算資料）に遡り*、引用先に本当にその記述があるかを確認します。本レポート第6章の「3段階ファクトチェック」をテンプレート化してください。
6. **情報源は「少数の良質なキュレーター＋一次情報」に絞る。** ニュースレター（例: Import AI、The Batch、Latent Space など）、arXiv／Hugging Face Papers、Hacker News、公式ブログを軸に、X・YouTube・ポッドキャストは補助として使います。
7. **目的別レシピを持つ。** 業界調査・競合調査・論文サーベイ・毎朝のニュースの4パターンについて、手順・プロンプト・チェックリストを第8〜9章に用意しました。

> **1行で言うと：** 「AI検索で素早く当たりを付け、Deep Researchで広く集め、NotebookLMで深く読み、一次情報で裏を取り、自動化で毎日届け、ナレッジベースに貯めて再利用する」。

---

<a id="ch1"></a>
## 第1章 なぜ「AIで情報収集」を設計し直すべきなのか

### 1.1 情報量の爆発と「追いつけなさ」

AI分野に限っても、arXivのcs.AI／cs.CL／cs.LGカテゴリには毎日数百本規模の論文が投稿され、主要ベンダー（OpenAI、Google、Anthropic、Meta、xAI、Mistral、国内各社など）は月に何度もモデルや機能を更新しています。さらにX（旧Twitter）、YouTube、ポッドキャスト、ニュースレター、Discordコミュニティなど、発信チャネル自体が増え続けています。

この状況では、従来型の「気になったらGoogle検索」「ブックマークして後で読む」というやり方は破綻しがちです。典型的な失敗パターンは次のとおりです。

- **後で読む地獄:** 「後で読む」に数百件たまり、結局読まない。
- **タイムライン依存:** Xのアルゴリズムが選んだ情報だけを見て、話題の偏りに気付かない。
- **孤立した知識:** 読んだ記事の内容を3か月後に思い出せず、また検索し直す。
- **AI出力の鵜呑み:** チャットAIの回答をそのまま資料に貼り、後で誤りが発覚する。

### 1.2 AIが変えた3つのこと

生成AIは情報収集のボトルネックを次の3点で緩和しました。

| 従来のボトルネック | AIによる変化 | 新たに生じたリスク |
|---|---|---|
| 検索キーワードを考える・10本のリンクを開いて読む | AI検索が複数ソースを読み、出典付きで要約 | 要約の誤り、出典と本文の不一致 |
| 長文（論文・決算・規制文書）を読む時間 | 要約・Q&A・翻訳が数秒で可能 | 重要な但し書きの欠落 |
| 情報の整理・分類・タグ付け | LLMによる自動分類・スコアリング | 分類基準のブラックボックス化 |

つまり **「集める・読む・整理する」コストは劇的に下がった一方、「正しさを確かめる」コストの重要性は相対的に上がった** のです。効率化の本質は、浮いた時間を検証と思考に再投資することにあります。

### 1.3 本レポートの基本方針

本レポートでは次の原則で情報収集を設計します。

1. **Pull（取りに行く）とPush（届く）を分ける。** 定点観測はPush化し、深掘りだけPullで行う。
2. **ツールに役割を割り当てる。** 「なんでもChatGPT」ではなく、得意分野で使い分ける。
3. **出典を必ず保存する。** 要約だけを保存しない。URL・取得日・原文の該当箇所をセットで残す。
4. **再現可能な手順にする。** プロンプト・Skill・ワークフローとして明文化し、誰がやっても同じ品質にする。
5. **人間は判断に集中する。** 何を調べるか、何を信じるか、どう使うかは人間が決める。

---

<a id="ch2"></a>
## 第2章 情報収集の全体アーキテクチャ（5層モデル）

効率的な情報収集は、次の5層で捉えると設計しやすくなります。

```text
┌────────────────────────────────────────────────────┐
│ ⑤ 活用層   : 資料作成・意思決定・発信（レポート、スライド、社内共有） │
├────────────────────────────────────────────────────┤
│ ④ 蓄積層   : Obsidian / Notion / Readwise / 個人RAG              │
├────────────────────────────────────────────────────┤
│ ③ 検証層   : 一次情報確認・クロスチェック・ファクトチェック         │
├────────────────────────────────────────────────────┤
│ ② 処理層   : LLM要約・翻訳・分類・スコアリング・重複排除           │
├────────────────────────────────────────────────────┤
│ ① 収集層   : RSS / ニュースレター / arXiv / HN / AI検索 / Deep Research │
└────────────────────────────────────────────────────┘
```

### 2.1 各層の役割と代表ツール

| 層 | 目的 | 代表的なツール・手段 | 自動化の余地 |
|---|---|---|---|
| ① 収集 | 漏れなく・偏りなく集める | Feedly、RSSリーダー、Perplexity、Deep Research、Exa/Tavily API、arXiv API | 高い |
| ② 処理 | 読む量を減らす | GPT／Gemini／Claude API、Feedly AI、Readwise Ghostreader | 高い |
| ③ 検証 | 誤りを除く | 一次情報、Google Scholar、公式ドキュメント、ファクトチェック手順 | 中（人間の確認が必須） |
| ④ 蓄積 | 再利用可能にする | Obsidian、Notion、Readwise、Glasp、ベクトルDB | 高い |
| ⑤ 活用 | 成果に変える | NotebookLM、Claude Projects、スライド生成 | 中 |

### 2.2 「時間軸」での整理

情報収集のタスクは、時間軸で大きく3つに分けられます。

- **デイリー（毎日・5〜15分）:** ニュースダイジェストの確認、重要トピックへのスター付け。→ 完全自動化＋人間は既読処理だけ。
- **ウィークリー（毎週・30〜60分）:** 週次まとめ、気になった論文や記事の精読、ナレッジベースへの整理。→ 半自動化。
- **プロジェクト（随時・数時間〜数日）:** 業界調査、競合調査、論文サーベイ。→ Deep Research＋NotebookLM＋人間の検証。

### 2.3 コスト感の目安

個人で一通りそろえる場合、次のような構成が一般的です（料金は2026年10月時点で各社が頻繁に改定しているため **『要確認』**）。

- 無料で始める構成: Perplexity無料枠＋Gemini無料枠＋NotebookLM（Gemini Notebook）無料枠＋Feedly無料版＋Obsidian（無料）＋n8nセルフホスト
- 標準構成: 主要チャットAIの有料プラン1つ（月20ドル前後）＋Readwise Reader＋API利用料（月数ドル〜）
- ヘビー構成: 複数のチャットAI上位プラン＋Feedly Pro+/Enterprise＋Exa/Tavily/Firecrawl APIの有料枠

> **ポイント:** 有料プランを複数契約する前に、「自分の情報収集のどの層がボトルネックか」を特定してください。多くの人のボトルネックは①収集ではなく、②処理と④蓄積です。

---

<a id="ch3"></a>
## 第3章 主要ツールの比較と使い分け

本章では、2026年10月時点で主要な情報収集系AIツールを「何に向いているか」という観点で比較します。各ツールは毎月のように機能追加や名称変更が行われているため、細かな仕様・料金は **『要確認』** として扱ってください。

<a id="ch3-1"></a>
### 3.1 AI検索エンジン（Perplexity / Felo / Genspark）

#### Perplexity

- **概要:** 質問に対してWeb検索を行い、出典番号付きで回答する「回答エンジン」の代表格。通常検索、Pro Search（より多段階の検索）、Deep Research、Spaces（プロジェクト単位でファイルや指示を保持）、Discover（ニュースフィード）などを備えます。
- **強み:** 出典の提示が明快で、クリックして一次情報に飛びやすいUI。2026年の比較記事でも「速度と引用の透明性」で高く評価されています（例: Nesyona、Presenc AIの比較記事。いずれも第三者評価であり、評価方法の妥当性は要確認）。
- **弱み:** 回答が「要約寄り」になり、深い統合・考察は弱い傾向。検索結果に引っ張られるため、SEO記事やまとめサイトが混じることがある。
- **向く用途:** 「最新の○○は何か」「Aの公式発表はいつか」などの事実確認、調査の初動での当たり付け。
- **使いこなしのコツ:**
  - 「公式サイト・一次情報を優先して」「2026年以降の情報に限定して」と条件を明示する。
  - Focus／ソース指定（Academic、ソーシャル等。名称はUI更新で変わるため要確認）を使い分ける。
  - Spacesに「回答は日本語、出典は英語一次情報優先、不明な点は不明と書く」といったカスタム指示を入れておく。

```text
【Perplexity用プロンプト例：事実確認】
次の主張が正しいか確認してください。
主張：「◯◯社は2026年◯月に△△を発表した」
条件：
- 公式発表（プレスリリース、公式ブログ、IR資料）を最優先の根拠にする
- 発表日・発表主体・具体的な内容を表で示す
- 一次情報が見つからない場合は「一次情報未確認」と明記する
- 報道ベースの情報と公式情報を区別する
```

#### Felo

- **概要:** 日本発（東京）のAI検索サービス。多言語横断検索（日本語で質問して英語・中国語などのソースを検索し、日本語で回答）を強みとし、マインドマップ生成、AIスライド生成、トピック（ナレッジ整理）機能などを持ちます（Felo公式ブログの記述に基づく。機能の詳細・料金は要確認）。
- **強み:** 日本語ユーザーが海外ソースに当たりやすい。調査結果をスライドやマインドマップに変換できるため、社内共有までが速い。
- **弱み:** 公式ブログの比較記事は自社に有利な比較である点に注意。高度な推論・長文レポートは Deep Research系に劣る場面がある（体感ベースの評価であり要確認）。
- **向く用途:** 海外動向の日本語での把握、調べた内容をそのまま資料化したい場面。

#### Genspark

- **概要:** 米国パロアルトのGenspark社による「AIエージェント型ワークスペース」。当初はSparkpage（検索結果を1枚のAI生成ページに集約）で知られましたが、2026年4月8日発表の「AI Workspace 4.0」で、デスクトップクライアント「Genspark Claw for Desktop」（ローカルファイル操作・Browser Use）、PowerPoint／Excel／Word向けプラグイン、Advanced Workflows などを打ち出しています（Genspark公式ブログ、Business Wireのプレスリリースで確認）。
- **強み:** 調査→スライド・表・文書作成までをエージェントが一気通貫で行う。複数サイトからのデータ収集やページ監視をBrowser Useで任せられる。
- **弱み:** エージェントの自律度が高い分、どのソースをどう使ったかの追跡が難しくなりがち。ローカルファイルやブラウザ操作の権限付与にはセキュリティ上の検討が必要。
- **向く用途:** 調査結果をすぐ成果物（スライド・表）にしたいとき、定型的なWebデータ収集。

#### AI検索3種の使い分け

| 観点 | Perplexity | Felo | Genspark |
|---|---|---|---|
| 主な価値 | 出典付きの高速回答 | 多言語横断・資料化 | エージェントによる作業代行 |
| 出典の追いやすさ | ◎ | ○ | △〜○（タスクによる） |
| 日本語UX | ○ | ◎ | ○ |
| 成果物生成 | △（ページ機能等あり） | ○（スライド・マインドマップ） | ◎（スライド・シート・文書） |
| おすすめ場面 | 事実確認・初動調査 | 海外動向の日本語把握 | 調査＋資料作成の一括処理 |

<a id="ch3-2"></a>
### 3.2 Deep Research（ChatGPT / Gemini / Claude / Perplexity）

「Deep Research」は、AIが調査計画を立て、数十〜数百のWebページを読み、数分〜数十分かけて長文レポートを書く機能の総称です。2024年12月にGeminiが先行し、2025年2月にChatGPT（2月3日）とPerplexity（2月15日）、2025年4月にClaude（Research機能）が続きました（aitoolradar.io の比較記事に基づく。日付は各社公式発表で要確認）。

#### 各社の特徴（2026年の比較記事からの整理）

| 項目 | ChatGPT Deep Research | Gemini Deep Research | Claude Research | Perplexity Deep Research |
|---|---|---|---|---|
| 所要時間の目安 | 長め（10〜30分程度との報告） | 中（5〜15分程度） | 中（5〜20分程度） | 短（3〜10分程度） |
| 強み | 網羅性・レポートの長さ、調査計画の編集、サイト制限 | Google検索の広さ、Drive・NotebookLM等をソースに指定可能、Docsへのエクスポート | 矛盾する情報の統合・推論、社内ツール連携（Google Workspace等）と組み合わせた調査 | 速さ、引用の検証しやすさ、無料枠 |
| 弱み | 時間がかかる、上位プラン以外は回数制限 | 技術的・推論的な内容では他に劣るとの評価も | ソース数が比較的少なめとの評価 | 統合・考察の深さ |
| 向く用途 | 本番の調査レポート、学術・技術の総合整理 | 大量の調査、Workspace中心の業務 | 社内資料＋Webの統合、意思決定支援 | 速報性のあるテーマ、初動調査 |

> 上表の所要時間・評価は Presenc AI、Nesyona、aitechrankings.com、aitoolradar.io など第三者の2026年比較記事の記述をまとめたもので、各社の公式値ではありません。ベースモデル名（GPT-5系、Gemini 3系、Claude Opus 4系など）は記事ごとに記述が異なり、頻繁に更新されるため **『要確認』** とします。

#### Deep Researchを使いこなす5つのコツ

1. **調査の「問い」を構造化して渡す。** 目的、対象範囲（期間・地域・業界）、除外条件、出力形式、想定読者を明示します。
2. **調査計画を確認・修正する。** ChatGPTやGeminiは実行前に計画を提示するので、ここで観点の抜け漏れを修正します。計画段階の修正が最もコスト効率が高いです。
3. **ソースの制約を指定する。** 「公式発表・政府統計・査読論文を優先」「まとめサイト・アフィリエイト記事は除外」と明記します。
4. **複数のDeep Researchを並走させる。** 同じ問いをChatGPTとGemini（あるいはClaude）に投げ、結論が一致する部分と食い違う部分を比較すると、検証すべき箇所が浮かび上がります。
5. **レポートを「そのまま使わない」。** 主要な数値・主張ごとに出典を開き、第6章の手順で検証してから使います。

```markdown
## Deep Research 依頼テンプレート

### 目的
（例）2027年度の新規事業検討のため、国内の「生成AIを使った議事録・会議支援SaaS」市場の現状を把握したい。

### 調査範囲
- 期間: 2024年1月〜2026年9月の情報を中心に
- 地域: 日本市場（比較対象として米国の主要プレイヤーも少し）
- 対象: 法人向けSaaS。個人向けアプリは除外

### 知りたいこと（優先順）
1. 主要プレイヤー（10社程度）と提供機能・価格帯・ターゲット
2. 市場規模の推計（出典と推計方法を明記。推計が複数あれば並記）
3. 導入企業の事例と課題（セキュリティ、精度、コスト）
4. 規制・ガイドラインの動向（個人情報保護、録音の同意など）

### ソースの条件
- 優先: 公式サイト、プレスリリース、IR資料、官公庁資料、調査会社の公開サマリー
- 注意: 比較サイト・アフィリエイト記事は補助的にのみ使用し、その旨を明記
- 不明な点は推測せず「不明」と書く

### 出力形式
- 冒頭に要点5行
- 比較表（プレイヤー×機能×価格×出典URL）
- 各主張の末尾に出典番号
- 最後に「確度が低い情報」「追加調査が必要な論点」のリスト
```

<a id="ch3-3"></a>
### 3.3 ソース限定型リサーチ（NotebookLM＝Gemini Notebook）

#### 名称変更と主な新機能（2026年）

Googleは **2026年7月16日、NotebookLMを「Gemini Notebook」に改称** しました（Google公式ブログ「NotebookLM is now Gemini Notebook」で確認）。独立した製品としては継続しつつ、Geminiアプリや検索のAI Modeとの連携を強める方針です。同発表では、ノートブックごとに「安全なクラウドコンピュータ」を持たせ、ソースに基づいてコードを書いて実行しデータ分析できる機能が、まずAI Ultra等の上位プランから順次展開されると説明されています。本レポートでは便宜上、従来名と併記して「NotebookLM（Gemini Notebook）」と書きます。

2026年中の主なアップデート（Google公式ブログ、Google Workspace Updatesブログ、CNET、XDAなどで確認できたもの）:

- スライド（Slide Deck）の指示による修正、PPTXエクスポート
- インフォグラフィックの10種類のスタイル選択
- フラッシュカード・クイズの進捗保存
- EPUBファイルのアップロード対応
- 会話履歴の保存（共有ノートブックでも自分の会話は自分だけに表示）
- Cinematic Video Overviews（アニメーション付き動画解説）
- 白紙の状態から、チャットでWeb上の関連ソースを探してノートブックに追加する機能
- PDFレポート、スプレッドシートなど多様な形式での出力

> 各機能の提供範囲（無料／有料、地域、言語）は段階的展開のため **『要確認』**。

#### NotebookLM（Gemini Notebook）の位置付け

最大の特徴は **「指定したソースに基づいて回答する」** ことです。Web全体を検索するAIと比べて、回答の根拠が手元の資料に限定されるため、ハルシネーションのリスクを下げやすく、回答中の引用をクリックすると該当箇所に飛べます。

**向く用途:**

- 論文・白書・決算資料・規制文書など、長文の一次資料を読み込んで質問する
- 社内資料と公開資料を一つのノートブックに集め、比較・統合する
- 音声解説（Audio Overview）で通勤中に資料の概要をつかむ
- 調査結果を学習用のクイズやフラッシュカードにする

**使いこなしのコツ:**

1. **1ノートブック＝1テーマ** に絞る。ソースを混ぜすぎると回答がぼやける。
2. **ソースの質を管理する。** 一次資料を中心に入れ、二次資料には「二次資料」とわかる名前を付ける。
3. **ノート機能で中間成果を保存する。** 良い回答はノートとして保存し、後でソースとして再利用する。
4. **「ソースに書かれていないことは書かない」と明示する。**

```text
【NotebookLM用プロンプト例：比較表の作成】
アップロードした3社の統合報告書をもとに、以下の観点で比較表を作成してください。
観点：生成AI関連の投資額、具体的な取り組み、KPI、リスク認識
ルール：
- ソースに記載がない項目は「記載なし」とし、推測で埋めない
- 各セルに引用元（ソース名とページ・該当箇所）を付ける
- 数値は単位と対象年度を必ず併記する
```

<a id="ch3-4"></a>
### 3.4 論文向けツール（Elicit / Consensus / SciSpace ほか）

学術情報の収集では、一般的なAI検索よりも論文データベースに特化したツールが有効です。

| ツール | 主な機能 | 強み | 注意点 |
|---|---|---|---|
| **Elicit** | 研究課題に関連する論文の検索、論文からの情報抽出（表形式）、系統的レビュー支援 | 複数論文から「対象・手法・結果」などを列として抽出し一覧化できる | 抽出の正確性は原文で要確認。料金体系・上限は要確認 |
| **Consensus** | Yes/No型の問いに対し、論文群がどう結論付けているかを集約 | 「研究の合意度」をつかみやすい | 問いの立て方で結果が変わる。医療・健康分野は特に原典確認必須 |
| **SciSpace** | 論文PDFの読解支援（Copilot）、文献検索、要約、関連論文探索 | 難解な論文の数式や専門用語を質問しながら読める | 解説の誤りに注意 |
| **Semantic Scholar** | 論文検索、引用関係、TLDR要約、API | 無料で使えるAPIがあり自動化に向く | 分野による網羅性の差 |
| **Google Scholar** | 論文検索、被引用数 | 網羅性が高い | AI要約は限定的。API公式提供なし（要確認） |
| **arXiv / Hugging Face Papers / alphaXiv** | プレプリント、日次の注目論文、論文へのコメント | AI分野の最新動向を最速で追える | プレプリントは査読前 |
| **Connected Papers / ResearchRabbit** | 引用ネットワークの可視化 | 関連研究の地図を作れる | 最新論文の反映に時差がある場合あり（要確認） |

**論文ツールの使い分けの型:**

1. **探索:** Elicit／Consensus／Semantic Scholarで関連論文を20〜50本ピックアップ
2. **地図作り:** Connected Papers／ResearchRabbitで重要論文（ハブ）を特定
3. **精読:** SciSpaceやNotebookLM（Gemini Notebook）に重要論文を入れて質問しながら読む
4. **整理:** ZoteroやObsidianに書誌情報・要約・自分のコメントを保存

> 注意: AIが提示する論文の **書誌情報（著者・年・タイトル・DOI）は必ず実在確認** してください。存在しない論文を生成する「引用ハルシネーション」は、汎用チャットAIで特に起こりやすい典型的な失敗です。DOIをdoi.orgで解決できるか、arXiv IDが実在するかを確認するのが最も簡単な検証方法です。

<a id="ch3-5"></a>
### 3.5 フィード・既読管理（Feedly AI / Readwise Reader / Glasp）

#### Feedly AI（Leo）

- **概要:** 老舗RSSリーダーFeedlyのAI機能群。トピック・企業・トレンドの追跡、優先度付け、ノイズ除去、要約を行います。現在は脅威インテリジェンス（Threat Intelligence）と市場インテリジェンス（Market Intelligence）の法人向け製品に注力しており、AI Feeds、AI Agents、Insights Cards、Boards、ニュースレター作成などの機能があります（Feedly公式サイト・Changelogで確認）。
- **2026年の動き:** 2026年4月15日のChangelogで、Slack／Microsoft Teams連携時の要約をユーザーが自由に定義できる「カスタムサマリー」、Cmd+Kで横断移動できる「Go To」、ニュースレターのワンクリック英訳などが発表されています。
- **向く用途:** 業界・競合・技術トピックの定点観測、チームへのアラート配信。
- **注意:** 個人向けプランと法人向けプランで使えるAI機能が大きく異なります（要確認）。

#### Readwise Reader

- **概要:** 記事、ニュースレター、PDF、EPUB、RSS、YouTube（字幕付き）、X投稿などを一か所に集めて読む「後で読む＋RSSリーダー」。ハイライトはReadwise本体に同期され、Obsidian・Notionなどへエクスポートできます。
- **AI機能「Ghostreader」:** 2026年8月の「Reader Public Beta Update #14」で **Global Ghostreader**（Web／デスクトップ）が発表され、ライブラリ全体を対象に引用付きで回答したり、タグ付け・移動などの操作を行えるようになりました。「Research Topic」「Triage Inbox/Feed」「Find Similar」などの再利用可能な **Skills** を備え、自作も可能です。同アップデートでは Reader の **MCP／CLI** にも触れられています（Readwise公式ドキュメント・ブログで確認。詳細仕様は要確認）。
- **向く用途:** 「読む」体験の中心。保存した記事群への横断質問、ハイライトの知識化。

#### Glasp

- **概要:** Webページ・PDF・YouTube字幕にハイライトを付け、ソーシャルに共有できるハイライトツール。他のユーザーのハイライトから良質な記事を発見できるのが特徴です。
- **2026年の動き:** 2026年7月のニュースレター「The Highlights」によれば、拡張機能v2.1.2で「Suggested Highlights（AIが重要文を提案）」「AI Overview」を追加、Firefox版を公開、Obsidianプラグインで要約・ブックマークも同期可能になり、インポート元（Matter、Raindrop.io、Diigo、Wallabag等）とエクスポート先（Zotero、Tana、Workflowy、Roam Research、Anki等）が拡充されたとされています。
- **向く用途:** 無料〜低コストでハイライト習慣を作る、YouTube字幕のハイライト、他者のハイライトからの発見。

#### 3ツールの関係

```text
Feedly（何が起きているか検知）
   │ 重要記事を送る
   ▼
Readwise Reader / Glasp（読む・ハイライトする）
   │ ハイライトと要約を同期
   ▼
Obsidian / Notion（知識として蓄積・リンク）
   │ 埋め込み・検索
   ▼
個人RAG / NotebookLM（質問して再利用）
```

<a id="ch3-6"></a>
### 3.6 総合比較表と「迷ったらこれ」

| やりたいこと | 第一候補 | 第二候補 | 理由 |
|---|---|---|---|
| 事実をすばやく確認したい | Perplexity | Felo / Gemini | 出典の追いやすさ |
| 30分以上かけて網羅的なレポートが欲しい | ChatGPT Deep Research | Gemini Deep Research | 網羅性と計画編集 |
| 社内資料とWebを組み合わせて考えたい | Claude Research（＋Projects） | Gemini Deep Research（Drive連携） | 連携ソースとの統合 |
| 手元のPDF群を深く読みたい | NotebookLM（Gemini Notebook） | Claude Projects | ソース限定・引用ジャンプ |
| 論文の研究動向を知りたい | Elicit / Consensus | Semantic Scholar＋NotebookLM | 論文DB特化 |
| 業界ニュースを毎日追いたい | Feedly AI | RSS＋LLM自作（第4章） | 定点観測と配信 |
| 読んだものを貯めて再利用したい | Readwise Reader | Glasp＋Obsidian | ハイライトの知識化 |
| 調査結果をそのままスライドにしたい | Genspark | Felo / NotebookLM | 成果物生成 |
| 海外情報を日本語でつかみたい | Felo | Perplexity（日本語指示） | 多言語横断 |

> **迷ったらこれ（個人の最小構成）:** Perplexity（無料）＋ Gemini（Deep Research・NotebookLM込みのプラン）＋ Readwise Reader ＋ Obsidian。チームなら Feedly＋Slack連携＋Notionを追加。

---

<a id="ch4"></a>
## 第4章 仕組み化：収集→要約→配信→蓄積の自動パイプライン

ツールを手で使うだけでは、忙しい週には情報収集が止まってしまいます。本章では「放っておいても毎朝要約が届き、ナレッジベースに貯まる」仕組みを、コード・設定例付きで解説します。

### 全体像

```text
[情報源]                [処理]                   [配信]            [蓄積]
RSS / Atom   ─┐
arXiv API    ─┤      ┌─ 重複排除            ┌─ Slack       ┌─ Obsidian (Markdown)
Hacker News  ─┼──▶  ├─ LLMで要約・翻訳  ──▶ ├─ Discord ──▶ ├─ Notion DB
ニュースレター─┤      ├─ 重要度スコアリング   └─ メール      └─ ベクトルDB (RAG)
YouTube字幕  ─┘      └─ タグ付け
```

<a id="ch4-1"></a>
### 4.1 RSS＋LLM要約の基本形（Python）

まずは最小構成として、RSSを取得し、LLMで日本語要約・スコアリングし、Markdownに出力するスクリプトです。OpenAI互換APIを想定していますが、Gemini・Claude等のSDKにも置き換えられます（モデル名は利用時点の最新を公式ドキュメントで確認してください）。

```python
# digest.py  ― RSS → LLM要約 → Markdownダイジェスト
# pip install feedparser openai python-dateutil
import feedparser, json, hashlib, os, datetime as dt
from pathlib import Path
from openai import OpenAI

FEEDS = {
    "Hacker News (100pt+)": "https://hnrss.org/newest?points=100",
    "arXiv cs.CL": "https://rss.arxiv.org/rss/cs.CL",
    "Simon Willison": "https://simonwillison.net/atom/everything/",
    # 公式ブログ等のRSS URLは各サイトで確認して追加する
}
SEEN_FILE = Path("seen.json")
MODEL = os.environ.get("DIGEST_MODEL", "gpt-4o-mini")  # 利用可能な最新モデルに変更
client = OpenAI()

SYSTEM = """あなたはAI分野のリサーチアシスタントです。
与えられた記事情報だけを根拠に、日本語で要約してください。
記事に書かれていないことを推測で補わないこと。"""

USER_TMPL = """以下の記事を評価してJSONで返してください。
{{"summary_ja": "3文以内の日本語要約",
  "importance": 1-5の整数（5=業界に大きな影響）,
  "tags": ["最大3つ"],
  "why": "重要度の理由を1文"}}

タイトル: {title}
URL: {link}
本文抜粋: {summary}
"""

def load_seen():
    return set(json.loads(SEEN_FILE.read_text())) if SEEN_FILE.exists() else set()

def summarize(entry):
    msg = USER_TMPL.format(title=entry.title, link=entry.link,
                           summary=getattr(entry, "summary", "")[:3000])
    res = client.chat.completions.create(
        model=MODEL,
        response_format={"type": "json_object"},
        messages=[{"role": "system", "content": SYSTEM},
                  {"role": "user", "content": msg}],
    )
    return json.loads(res.choices[0].message.content)

def main():
    seen, items = load_seen(), []
    for source, url in FEEDS.items():
        for e in feedparser.parse(url).entries[:20]:
            key = hashlib.sha1(e.link.encode()).hexdigest()
            if key in seen:
                continue
            seen.add(key)
            try:
                r = summarize(e)
            except Exception as ex:
                print("skip:", e.link, ex); continue
            items.append({"source": source, "title": e.title, "link": e.link, **r})

    items.sort(key=lambda x: x["importance"], reverse=True)
    today = dt.date.today().isoformat()
    lines = [f"# AIデイリーダイジェスト {today}\n"]
    for it in items:
        if it["importance"] < 3:
            continue
        lines.append(f"## {'★'*it['importance']} {it['title']}")
        lines.append(f"- 出典: [{it['source']}]({it['link']})")
        lines.append(f"- 要約: {it['summary_ja']}")
        lines.append(f"- タグ: {', '.join(it['tags'])} / 理由: {it['why']}\n")
    Path(f"digest-{today}.md").write_text("\n".join(lines), encoding="utf-8")
    SEEN_FILE.write_text(json.dumps(list(seen)))

if __name__ == "__main__":
    main()
```

**設計上のポイント:**

- **重複排除（seen.json）** を必ず入れる。同じ記事を毎日要約するとAPI費用が無駄になります。
- **本文抜粋のみで要約** しているため、要約は「何についての記事か」を知るためのものと割り切ります。重要記事は原文を読む前提です。
- **JSON出力（構造化出力）** にしておくと、後段でSlack整形・Notion登録・フィルタが楽になります。
- **重要度の閾値** で配信量を制御します。最初は3以上、慣れたら4以上に絞るとよいでしょう。
- 定期実行は `cron`（例: `0 7 * * * cd ~/digest && python digest.py`）や GitHub Actions の `schedule` で行えます。

```yaml
# .github/workflows/digest.yml  ― GitHub Actionsで毎朝7時(JST)に実行
name: daily-digest
on:
  schedule:
    - cron: "0 22 * * *"   # UTC 22:00 = JST 7:00
  workflow_dispatch:
jobs:
  run:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.12" }
      - run: pip install feedparser openai python-dateutil requests
      - run: python digest.py
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
      - run: |
          git config user.name "digest-bot"
          git config user.email "digest-bot@users.noreply.github.com"
          git add seen.json digest-*.md && git commit -m "digest" || true
          git push
```

> 公開リポジトリで運用する場合は、ダイジェスト本文に有料記事の内容などを含めないよう注意してください（著作権・利用規約）。プライベートリポジトリを推奨します。

#### ニュースレターをRSS化する

ニュースレターはメールで届くため、そのままでは自動処理しにくいものです。主な方法は次のとおりです。

1. **Readwise Reader / Feedly のニュースレター専用アドレス** に転送・登録する（両サービスとも受信用アドレス機能あり。詳細は要確認）。
2. **Kill the Newsletter!** のようなメール→Atom変換サービスを使う（運営状況は要確認）。
3. **Gmailのフィルタ＋Apps Script** でラベル付きメールを抽出し、処理パイプラインへ送る。

```javascript
// Google Apps Script：ラベル「newsletter」のメールを要約対象としてWebhookに送る
function forwardNewsletters() {
  const threads = GmailApp.search('label:newsletter is:unread newer_than:1d');
  const url = PropertiesService.getScriptProperties().getProperty('WEBHOOK_URL');
  threads.forEach(t => {
    t.getMessages().forEach(m => {
      const payload = {
        from: m.getFrom(),
        subject: m.getSubject(),
        date: m.getDate().toISOString(),
        body: m.getPlainBody().slice(0, 20000)
      };
      UrlFetchApp.fetch(url, {method: 'post', contentType: 'application/json',
                              payload: JSON.stringify(payload)});
    });
    t.markRead();
  });
}
```

<a id="ch4-2"></a>
### 4.2 n8n / Make / Zapier での自動化

ノーコード／ローコードの自動化ツールを使うと、コードを書かずに同様のパイプラインを構築できます。

| 項目 | n8n | Make（旧Integromat） | Zapier |
|---|---|---|---|
| ホスティング | クラウド版＋セルフホスト可（ライセンス条件は要確認） | クラウドのみ | クラウドのみ |
| 料金の考え方 | 実行回数ベース（クラウド）、セルフホストは自前サーバー代 | オペレーション数ベース | タスク数ベース |
| AI連携 | AI Agentノード、LangChain系ノード、各社LLMノード | OpenAI等のモジュール、AI関連機能 | AI by Zapier、Zapier Agents等（名称は要確認） |
| 強み | 自由度が高い、コードノード、自前データを外に出さない構成が可能 | 視覚的なシナリオ設計、分岐が直感的 | 連携アプリ数が最多クラス、設定が最も簡単 |
| 向く人 | エンジニア、データを社外に出したくない組織 | ノンエンジニアで複雑な分岐を組みたい人 | とにかく早く動かしたい人 |

#### n8n：毎朝のAIダイジェスト・ワークフロー

ノード構成の例です。

```text
[Schedule Trigger 毎朝6:30]
   → [RSS Read ×N（各フィード）]
   → [Merge]
   → [Remove Duplicates / Code（既読チェック）]
   → [Filter：直近24時間]
   → [OpenAI（またはAnthropic / Gemini）ノード：要約＋スコアJSON]
   → [Filter：importance >= 3]
   → [Sort：importance降順]
   → [Code：Slack Block Kit形式に整形]
   → [Slack：チャンネルへ投稿]
   → [Notion：データベースに1件ずつ登録]
```

コードノード（JavaScript）の整形例:

```javascript
// n8n Code ノード：要約済みアイテムを1つのSlackメッセージにまとめる
const items = $input.all().map(i => i.json);
const top = items.sort((a, b) => b.importance - a.importance).slice(0, 10);
const blocks = [
  { type: "header", text: { type: "plain_text", text: `🗞 AIダイジェスト ${new Date().toLocaleDateString('ja-JP')}` } },
  ...top.flatMap(it => [
    { type: "section", text: { type: "mrkdwn",
      text: `*<${it.link}|${it.title}>*\n${'★'.repeat(it.importance)}  ${it.summary_ja}\n_${it.tags.join(' / ')}_` } },
    { type: "divider" }
  ])
];
return [{ json: { blocks } }];
```

> ※上記コードのヘッダー絵文字はSlack表示用の例です。不要なら削除してください。

LLMノードに渡すシステムプロンプト例:

```text
あなたはAI業界アナリストです。入力された記事のタイトルと抜粋のみを根拠に評価します。
出力は必ず次のJSONのみ：
{"summary_ja": string, "importance": 1-5, "tags": string[], "category": "モデル|製品|研究|規制|資金調達|その他"}
評価基準：
5 = 主要ベンダーの新モデル・重大な規制・業界構造の変化
4 = 実務に影響する新機能・重要論文
3 = 知っておくと良い動向
2以下 = 宣伝・重複・既報
抜粋に情報が不足している場合は importance を下げ、summary_jaに「詳細不明」と書く。
```

#### Make：シナリオの例

```text
[RSS > Watch RSS feed items]（複数フィードはRouterで並列）
  → [Tools > Set variable]（本文を3000字に切り詰め）
  → [OpenAI > Create a Completion / Chat]（JSON出力）
  → [JSON > Parse JSON]
  → [Filter：importance >= 4]
  → [Slack > Create a Message] ＋ [Notion > Create a Database Item]
```

#### Zapier：最短構成

```text
Trigger: RSS by Zapier（New Item in Multiple Feeds）
Action 1: ChatGPT / AI by Zapier（要約・分類）
Action 2: Filter by Zapier（重要度でフィルタ）
Action 3: Digest by Zapier（1日分をためて朝にまとめて放出）
Action 4: Slack（Send Channel Message）
```

ZapierのDigest機能を使うと「1記事ごとに通知」ではなく「1日分まとめて1通」にできるため、通知疲れを防げます（機能仕様は要確認）。

<a id="ch4-3"></a>
### 4.3 Slack / Discord への定期ダイジェスト配信

#### Slack（Incoming Webhook）

```python
import os, requests

def post_slack(markdown_text: str):
    url = os.environ["SLACK_WEBHOOK_URL"]
    # Slackのmrkdwnは標準Markdownと記法が異なる（太字は *text*、リンクは <url|text>）
    requests.post(url, json={"text": markdown_text}, timeout=10).raise_for_status()
```

#### Discord（Webhook）

```python
import os, requests

def post_discord(items):
    url = os.environ["DISCORD_WEBHOOK_URL"]
    embeds = [{
        "title": it["title"][:256],
        "url": it["link"],
        "description": it["summary_ja"][:4000],
        "footer": {"text": f"重要度 {it['importance']} / {', '.join(it['tags'])}"}
    } for it in items[:10]]          # 1メッセージあたりのembed上限（10件）に注意
    requests.post(url, json={"content": "本日のAIダイジェスト", "embeds": embeds},
                  timeout=10).raise_for_status()
```

#### 配信設計のベストプラクティス

- **1日1回・朝に集約**（リアルタイム通知は重大ニュースのみ別チャンネル）。
- **チャンネルを分ける:** `#ai-digest-daily`（全般）、`#ai-alert`（重要度5のみ）、`#paper-digest`（論文）。
- **スレッドで議論:** ダイジェストの各項目にスレッドでコメントする文化を作ると、チームの知見が蓄積されます。
- **リアクションでフィードバック:** 👍／👎（あるいは任意のスタンプ）を集計し、スコアリングプロンプトの改善に使う。
- **週次まとめ:** 金曜夕方に「今週の重要度4以上」をLLMで再要約して配信する。

```text
【週次まとめ用プロンプト】
以下は今週配信したAIダイジェストの項目（重要度4以上）です。
1. 今週の主要トピックを3〜5個にグルーピングし、それぞれ3行で説明
2. 「来週以降に注視すべき論点」を3つ
3. 各トピックに該当する元記事のURLを列挙
入力にない情報は追加しないでください。
```

<a id="ch4-4"></a>
### 4.4 Obsidian / Notion へのナレッジ蓄積

#### Obsidian：Markdownで「自分専用のWiki」を作る

Obsidianはローカルのフォルダ（Vault）にMarkdownファイルを保存するノートアプリです。テキストファイルなので、スクリプトから直接書き込め、Gitでのバージョン管理やLLMへの投入も容易です。

推奨フォルダ構成:

```text
Vault/
├── 00_Inbox/          # 自動取り込み（ダイジェスト、Readwise同期）
├── 10_Sources/        # 記事・論文ごとのノート（1ソース1ノート）
├── 20_Topics/         # トピックノート（例：RAG、AIエージェント、規制）
├── 30_Projects/       # 調査プロジェクト
├── 90_Templates/
└── 99_Archive/
```

1ソース1ノートのテンプレート（Templaterプラグイン等で利用）:

```markdown
---
title: "{{title}}"
source_url: "{{url}}"
source_type: article   # article / paper / video / podcast / official
author: "{{author}}"
published: {{published_date}}
retrieved: {{date}}
tags: [ai/agents, 要検証]
reliability: unverified   # unverified / checked / primary
---

## 要約（AI生成・未検証）
- 

## 重要な主張と根拠
| 主張 | 根拠（原文の該当箇所） | 検証状況 |
|---|---|---|
|  |  |  |

## 自分のコメント・示唆

## 関連ノート
- [[ ]]
```

ポイントは **「AI生成の要約」と「自分の考え」と「検証状況」を分けて書く** ことです。数か月後に読み返したとき、どこまで信用してよいかが一目でわかります。

Dataviewプラグインを使えば、検証待ちのノート一覧を自動で作れます。

````markdown
```dataview
TABLE source_type, retrieved, source_url
FROM "10_Sources"
WHERE reliability = "unverified"
SORT retrieved DESC
LIMIT 30
```
````

Readwiseの公式Obsidianプラグイン、Glaspのプラグインを使うと、ハイライトが自動で `00_Inbox` に同期されます。

#### Notion：データベースでチーム共有

チームで共有するならNotionのデータベースが便利です。推奨プロパティ:

| プロパティ | 型 | 用途 |
|---|---|---|
| タイトル | Title | 記事名 |
| URL | URL | 出典 |
| 情報源 | Select | HN / arXiv / 公式ブログ / ニュース 等 |
| カテゴリ | Select | モデル / 製品 / 研究 / 規制 / 資金調達 |
| 重要度 | Number | 1〜5（LLM付与＋人間修正） |
| 要約 | Text | AI要約 |
| 検証状況 | Status | 未検証 / 確認済 / 一次情報 |
| 取得日 | Date | 自動 |
| 担当者 | Person | 深掘り担当 |

Notion APIへの登録例:

```python
# pip install notion-client
import os
from notion_client import Client

notion = Client(auth=os.environ["NOTION_TOKEN"])
DB_ID = os.environ["NOTION_DB_ID"]

def add_item(it, retrieved_iso):
    notion.pages.create(
        parent={"database_id": DB_ID},
        properties={
            "タイトル": {"title": [{"text": {"content": it["title"][:200]}}]},
            "URL": {"url": it["link"]},
            "カテゴリ": {"select": {"name": it.get("category", "その他")}},
            "重要度": {"number": it["importance"]},
            "要約": {"rich_text": [{"text": {"content": it["summary_ja"][:1900]}}]},
            "検証状況": {"status": {"name": "未検証"}},
            "取得日": {"date": {"start": retrieved_iso}},
        },
    )
```

> Notion APIのバージョンやデータベース／データソースの扱いは更新されることがあるため、実装時は公式のAPIリファレンスを確認してください（要確認）。

<a id="ch4-5"></a>
### 4.5 個人RAGの構築

蓄積が数百〜数千ノートになると、「前に読んだあの記事、何だったっけ？」に答えてくれる **個人RAG（Retrieval-Augmented Generation）** が威力を発揮します。

#### 選択肢

| 方式 | 例 | 難易度 | 特徴 |
|---|---|---|---|
| 既製サービス | NotebookLM（Gemini Notebook）、Readwise Global Ghostreader、Claude Projects | 低 | すぐ使える。データは各サービスに置く |
| Obsidianプラグイン | Smart Connections、Copilot for Obsidian 等（要確認） | 低〜中 | Vault内で完結。ローカルLLMも選べる場合あり |
| 自作（ローカル） | Python＋Chroma/LanceDB＋埋め込みモデル | 中 | 自由度が高く、データを手元に置ける |
| MCP経由 | Obsidian MCP / Notion MCP＋Claude等 | 中 | エージェントが必要に応じて検索 |

#### 自作RAGの最小実装（Obsidian Vaultを検索）

```python
# rag.py ― Obsidian VaultをChromaにインデックスして質問する最小構成
# pip install chromadb openai python-frontmatter
import os, glob, frontmatter, chromadb
from openai import OpenAI

VAULT = os.path.expanduser("~/Vault")
client = OpenAI()
db = chromadb.PersistentClient(path="./rag_db")
col = db.get_or_create_collection("vault")
EMB_MODEL = "text-embedding-3-small"   # 利用時点のモデル名を確認
CHAT_MODEL = "gpt-4o-mini"             # 同上

def chunks(text, size=800, overlap=150):
    i = 0
    while i < len(text):
        yield text[i:i+size]; i += size - overlap

def embed(texts):
    r = client.embeddings.create(model=EMB_MODEL, input=texts)
    return [d.embedding for d in r.data]

def index():
    for path in glob.glob(f"{VAULT}/**/*.md", recursive=True):
        post = frontmatter.load(path)
        parts = list(chunks(post.content))
        if not parts: continue
        ids = [f"{path}::{i}" for i in range(len(parts))]
        metas = [{"path": path, "url": str(post.get("source_url", "")),
                  "reliability": str(post.get("reliability", ""))} for _ in parts]
        col.upsert(ids=ids, documents=parts, embeddings=embed(parts), metadatas=metas)

def ask(q, k=6):
    res = col.query(query_embeddings=embed([q]), n_results=k)
    ctx = "\n\n".join(
        f"[{i+1}] ({m['path']} / {m['url']} / 信頼度:{m['reliability']})\n{d}"
        for i, (d, m) in enumerate(zip(res["documents"][0], res["metadatas"][0])))
    prompt = f"""次の資料だけを根拠に質問に答えてください。
資料にない内容は「手元の資料には見当たりません」と答えること。
主張ごとに [番号] で出典を示すこと。信頼度が unverified の資料に基づく主張には（未検証）と付けること。

# 資料
{ctx}

# 質問
{q}"""
    r = client.chat.completions.create(model=CHAT_MODEL,
                                       messages=[{"role": "user", "content": prompt}])
    return r.choices[0].message.content

if __name__ == "__main__":
    import sys
    if sys.argv[1] == "index": index()
    else: print(ask(" ".join(sys.argv[1:])))
```

**実運用での改善ポイント:**

- **チャンク分割を見出し単位に:** Markdownの `##` 見出しで区切ると文脈が保たれやすい。
- **ハイブリッド検索:** ベクトル検索に加え、BM25などのキーワード検索を併用すると固有名詞・型番に強くなる。
- **メタデータフィルタ:** 「2026年以降」「reliability=primary のみ」などで絞り込む。
- **差分インデックス:** ファイルの更新日時やハッシュで変更分だけ再インデックスする。
- **ローカルLLM:** 機密情報を扱う場合は、Ollama等でローカルの埋め込みモデル・LLMを使う構成も可能（性能とのトレードオフ）。

---

<a id="ch5"></a>
## 第5章 MCPサーバーとSkillsで「調べるエージェント」を作る

### 5.1 MCPとSkillsの関係

- **MCP（Model Context Protocol）:** AIアプリケーション（Claude Desktop、Claude Code、Cursor、VS Code、ChatGPTの一部機能など。対応状況は要確認）と外部ツール・データをつなぐ標準プロトコル。「検索する」「ページを取得する」「Notionに書く」といった **手足** を提供します。
- **Agent Skills（SKILL.md）:** Anthropicが開発し、2025年12月18日にオープン標準として公開したフォーマット（Anthropic公式エンジニアリングブログ、agentskills/agentskillsリポジトリで確認）。「どういう手順で、どのツールを使って、どんな品質基準で仕事をするか」という **手順書・ノウハウ** をフォルダにまとめます。

つまり **MCP＝道具、Skill＝道具の使い方を書いたマニュアル** です。両者を組み合わせると、「毎回同じ品質で調査するエージェント」を作れます。

### 5.2 情報収集に有用なMCPサーバー

| MCPサーバー | 提供元 | 主な機能 | 要APIキー | 用途 |
|---|---|---|---|---|
| **Brave Search** | Brave公式（github.com/brave/brave-search-mcp-server） | Web・ニュース・画像・動画・ローカル検索、要約 | 要（Brave Search API） | 汎用Web検索。プライバシー重視 |
| **Exa** | Exa公式（github.com/exa-labs/exa-mcp-server） | `web_search_exa`、`web_fetch_exa`、高度な検索（ドメイン・日付フィルタ）、`agent_run` | ホスト版は匿名でもレート制限付きで利用可、OAuth/APIキーで上限緩和 | 意味検索、類似ページ探索、技術情報 |
| **Tavily** | Tavily公式（github.com/tavily-ai/tavily-mcp） | search / extract / map / crawl | 要 | エージェント向け検索、サイトのクロール |
| **Firecrawl** | Firecrawl公式（github.com/firecrawl/firecrawl-mcp-server） | スクレイピング、クロール、検索、構造化抽出 | 要 | Webページを綺麗なMarkdownに変換 |
| **Fetch** | MCP公式リファレンス実装（modelcontextprotocol/servers） | URLを取得してMarkdown化 | 不要 | 一次情報ページの取得 |
| **arXiv** | コミュニティ製（例: arxiv-mcp-server。リポジトリは要確認） | 論文検索・取得・要約 | 不要 | 論文サーベイ |
| **YouTube字幕** | コミュニティ製（複数実装あり。要確認） | 動画の字幕（トランスクリプト）取得 | 実装による | 講演・ポッドキャスト動画の要約 |
| **Notion** | Notion公式（makenotion/notion-mcp-server、およびホスト型MCP。要確認） | ページ・DBの検索・作成・更新 | OAuth / トークン | 調査結果の保存 |
| **Obsidian** | コミュニティ製（Local REST APIプラグイン経由の実装等。要確認） | Vault内ノートの検索・読み書き | プラグインのAPIキー | 個人ナレッジの参照・追記 |
| **Filesystem / Memory / Git** | MCP公式リファレンス実装 | ローカルファイル、知識グラフ型メモリ、Git操作 | 不要 | ローカル資料・メモの管理 |

> MCPサーバーはサードパーティ製が多く、品質・保守状況・セキュリティにばらつきがあります。**公式提供のものを優先** し、コミュニティ製は利用前にソースコード・権限・最終更新日を確認してください。上表のコミュニティ製リポジトリ名は特定のものを推奨する意図ではなく、**『要確認』** です。

### 5.3 設定例（Claude Desktop / Cursor 等の `mcpServers` 形式）

多くのMCPクライアントは次のようなJSON形式で設定します（ファイルの場所はクライアントごとに異なるため各ドキュメントを参照）。

```json
{
  "mcpServers": {
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@brave/brave-search-mcp-server", "--transport", "stdio"],
      "env": { "BRAVE_API_KEY": "YOUR_BRAVE_API_KEY" }
    },
    "exa": {
      "url": "https://mcp.exa.ai/mcp"
    },
    "tavily": {
      "command": "npx",
      "args": ["-y", "tavily-mcp@latest"],
      "env": { "TAVILY_API_KEY": "YOUR_TAVILY_API_KEY" }
    },
    "firecrawl": {
      "command": "npx",
      "args": ["-y", "firecrawl-mcp"],
      "env": { "FIRECRAWL_API_KEY": "YOUR_FIRECRAWL_API_KEY" }
    },
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch"]
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/you/Vault"]
    }
  }
}
```

- Brave Search の設定例（`npx -y @brave/brave-search-mcp-server`、環境変数 `BRAVE_API_KEY`）は公式リポジトリのREADMEに基づきます。
- Exa のホスト型エンドポイント `https://mcp.exa.ai/mcp` は公式READMEに記載があります。`?tools=web_search_exa,web_fetch_exa` のように有効化するツールを指定できます。リモートURL形式（`"url"`）に対応しているかはクライアントにより異なります（要確認）。
- Tavily はリモートMCP（`https://mcp.tavily.com/mcp/`）も提供しています。APIキーをURLクエリに含める方式が案内されていますが、キーの漏えいに注意してください。
- Firecrawl のパッケージ名（`firecrawl-mcp`）と環境変数名は、公式リポジトリで最新を確認してください（要確認）。

Claude Code の場合はCLIで追加できます。

```bash
# Claude Code にMCPサーバーを追加する例（オプションは公式ドキュメントで要確認）
claude mcp add brave-search -e BRAVE_API_KEY=xxxx -- npx -y @brave/brave-search-mcp-server
claude mcp add --transport http exa https://mcp.exa.ai/mcp
claude mcp add fetch -- uvx mcp-server-fetch
```

### 5.4 検索系MCPの使い分け

| 状況 | 推奨 | 理由 |
|---|---|---|
| 一般的なニュース・事実確認 | Brave Search | 通常の検索エンジンに近い結果、ニュース検索あり |
| 「この記事に似たものを探す」「技術ブログ・論文寄り」 | Exa | ニューラル（意味）検索、類似検索、日付・ドメインフィルタ |
| エージェントに要約済みの検索結果を渡したい | Tavily | LLM向けに整形された結果、extract・crawl |
| 特定ページを丁寧に読む | Fetch / Firecrawl | 本文をMarkdown化。JSレンダリングが必要ならFirecrawl |
| サイト全体を調べる（ドキュメント、価格ページ群） | Firecrawl / Tavily crawl | クロールと構造化抽出 |

> **コスト管理:** 検索APIは呼び出し回数課金が一般的です。エージェントは検索を何十回も繰り返すことがあるため、Skill側で「検索は最大N回」「同じクエリを繰り返さない」と上限を定めておきましょう。

### 5.5 Skillsの設計例

Agent Skillsの仕様では、スキルは `SKILL.md` を含むディレクトリで、YAMLフロントマターに `name`（必須・64文字以内・小文字英数字とハイフン）と `description`（必須・1024文字以内）を書きます。任意で `license`、`compatibility`、`metadata`、`allowed-tools`（実験的）を指定できます。エージェントは起動時に name と description だけを読み込み、必要になったときに本文、さらに必要なら `scripts/` や `references/` のファイルを読み込みます（段階的開示）。本文は5,000トークン未満が推奨されています（agentskills仕様で確認）。

#### 例1：ファクトチェック付きWeb調査スキル

```text
skills/
└── web-research-verified/
    ├── SKILL.md
    ├── references/
    │   ├── source-tiers.md      # 情報源の信頼度ランク表
    │   └── report-template.md   # 出力テンプレート
    └── scripts/
        └── check_urls.py        # 出典URLの生存確認
```

```markdown
---
name: web-research-verified
description: Web上の最新情報を複数の検索ツールで調べ、一次情報で裏を取ったうえで出典付きの日本語レポートを作成する。「調べて」「最新動向」「ファクトチェック」「出典付きで」などの依頼で使用する。
compatibility: Brave Search / Exa / Fetch のMCPサーバーが必要
---

# 検証付きWeb調査

## 手順
1. 依頼を「調査の問い」「期間」「対象範囲」「出力形式」に分解し、曖昧なら仮定を明記する。
2. 検索クエリを日本語・英語で各3〜5本作る（固有名詞は英語表記も併用）。
3. Brave Searchで広く、Exaで技術記事・類似記事を探す。検索は合計15回まで。
4. 有望なURLは Fetch で本文を取得し、要点と該当箇所の引用（原文短く）をメモする。
5. 各主張について references/source-tiers.md に従い情報源ランクを付ける。
6. 重要な主張（数値・日付・発表内容）は、ランクAの一次情報で確認できるまで「未確認」とする。
7. scripts/check_urls.py で出典URLの生存を確認する。
8. references/report-template.md の形式で出力する。

## 禁止事項
- 取得していないページの内容を書かない。
- URL・人名・論文名を推測で作らない。見つからなければ「見つからなかった」と書く。
- 二次情報だけで断定しない。

## 出力の最後に必ず含めるもの
- 「確認済み」「未確認」「矛盾あり」に分けた主張リスト
- 使用した検索クエリ一覧
```

`references/source-tiers.md` の例:

```markdown
# 情報源ランク
- A（一次情報）: 公式発表・プレスリリース、公式ドキュメント、論文本文、法令・官公庁資料、決算・IR資料、本人の発言（録画・原文）
- B（信頼できる二次情報）: 主要報道機関、専門メディアの署名記事、査読付きレビュー
- C（参考情報）: 個人ブログ、技術ブログ、SNS投稿、比較サイト
- D（使用不可）: 出典不明のまとめ、AI生成と思われる無署名記事、アフィリエイト目的が明らかなページ
ルール: 数値・日付・発表内容はAで確認する。Bのみの場合は「報道ベース」と明記する。
```

`scripts/check_urls.py` の例:

```python
import sys, requests
for url in sys.argv[1:]:
    try:
        r = requests.head(url, allow_redirects=True, timeout=10,
                          headers={"User-Agent": "Mozilla/5.0 (url-check)"})
        if r.status_code >= 400:   # HEADを拒否するサイト向けにGETで再試行
            r = requests.get(url, allow_redirects=True, timeout=10, stream=True)
        print(f"{r.status_code}\t{url}")
    except Exception as e:
        print(f"ERR\t{url}\t{e}")
```

#### 例2：arXiv論文サーベイスキル

```markdown
---
name: arxiv-survey
description: 指定テーマのarXiv論文を検索し、重要論文の要点・手法・結果を比較表にまとめる。論文サーベイ、先行研究調査、「最近の研究動向」の依頼で使用する。
---

# arXiv論文サーベイ

## 手順
1. テーマから英語キーワードを5つ以上作り、同義語も含める。
2. arXiv MCP（なければ Fetch で export.arxiv.org のAPI）で直近12か月の論文を検索する。
3. タイトルとアブストラクトから関連度を1〜5で評価し、上位15本を選ぶ。
4. 各論文について「課題・手法・データセット・主な結果・限界」を抽出する。アブストラクトにない項目は「本文未確認」とする。
5. arXiv ID とタイトルの組を必ず記録し、存在しないIDを出力しない。
6. 研究の流れ（系譜）を3〜5のクラスタに分けて説明する。

## 出力
- 比較表（arXiv ID / タイトル / 年月 / 手法 / 結果 / 限界）
- クラスタごとの解説
- 「査読前（プレプリント）である」旨の注意書き
```

arXivの公式API（Atom形式）を直接使う場合のクエリ例:

```bash
curl "http://export.arxiv.org/api/query?search_query=all:%22retrieval%20augmented%20generation%22&sortBy=submittedDate&sortOrder=descending&max_results=20"
```

#### 例3：動画・ポッドキャスト要約スキル

```markdown
---
name: video-digest
description: YouTube動画やポッドキャストの字幕を取得し、タイムスタンプ付きで要点を日本語にまとめる。「この動画を要約」「講演のポイント」などの依頼で使用する。
---

# 動画要約
1. YouTube字幕MCPで字幕を取得する。字幕がない場合は取得できない旨を伝えて終了する。
2. 話題の切れ目でセクションに分け、各セクションに開始時刻を付ける。
3. 発言者の主張と、事実として語られた情報（数値・日付・製品名）を分けて記載する。
4. 事実情報は「要検証」として一覧化する（動画内の発言は一次情報ではあるが正確とは限らない）。
5. 最後に「この動画を見るべき人／飛ばしてよい人」を1行で書く。
```

#### 例4：ナレッジ保存スキル（Obsidian / Notion）

```markdown
---
name: save-to-knowledge-base
description: 調査結果や読んだ記事を、出典・取得日・検証状況付きでObsidianまたはNotionに保存する。「保存して」「ナレッジに追加」「メモしておいて」で使用する。
---

# ナレッジ保存
- 保存先: 個人メモはObsidian（10_Sources/）、チーム共有はNotionの「AIリサーチDB」
- 必須メタデータ: title, source_url, source_type, published, retrieved, tags, reliability
- reliability は、一次情報で確認済みなら primary、二次情報で確認済みなら checked、それ以外は unverified
- 既存ノートと重複する場合は新規作成せず、既存ノートに追記する（先にタイトルとURLで検索）
- 要約は「AI生成」と明記する
```

### 5.6 MCP利用時のセキュリティ注意点

- **プロンプトインジェクション:** 取得したWebページに「以前の指示を無視して…」といった文言が仕込まれている可能性があります。Fetch等で取得した内容は「データ」であり「指示」ではないことをSkillに明記し、書き込み系ツール（Notion更新、ファイル削除等）は承認制にします。
- **最小権限:** FilesystemはVaultなど特定フォルダに限定、NotionはインテグレーションのアクセスをDB単位で限定します。
- **APIキー管理:** 設定ファイルをGitにコミットしない。URLにキーを含める方式はログに残りやすい点に注意。
- **データの持ち出し:** 機密資料を外部検索APIのクエリに含めないよう注意します。

---

<a id="ch6"></a>
## 第6章 情報の信頼性検証（ハルシネーション対策とファクトチェック）

### 6.1 AI情報収集で起こる典型的な誤り

| 誤りの種類 | 内容 | 例 | 主な対策 |
|---|---|---|---|
| 事実の捏造 | 存在しない事実・数値を生成 | 架空の市場規模、架空の発表日 | 一次情報で確認 |
| 引用ハルシネーション | 実在しない論文・URL・人物を提示 | それらしいDOI、架空の著者 | DOI・URLの実在確認 |
| 引用の不一致 | 出典は実在するが、その記述がない | リンク先に該当数値がない | 引用先の該当箇所を確認 |
| 古い情報 | 学習データ時点や古い記事の情報 | 改定前の料金・旧モデル名 | 日付を確認、公式の最新ページを見る |
| 文脈の欠落 | 但し書き・条件を落とす | 「一部地域のみ」「ベータ版」が消える | 原文の前後を読む |
| 二次情報の連鎖 | 誤報が複数記事にコピーされ「多数派」に見える | 同じ誤りが10記事に | 情報の出どころ（起点）を辿る |
| 宣伝・利害の混入 | 自社比較記事・アフィリエイト | 「A社が最良」とするA社ブログ | 発信者の利害を確認 |

### 6.2 プロンプト段階でできる対策

```text
【ハルシネーション抑制のためのシステム指示例】
- 検索・取得した資料に書かれていることだけを根拠にしてください。
- 各主張の直後に出典（URLと該当箇所の短い引用）を付けてください。
- 資料から確認できない場合は「確認できませんでした」と書き、推測で補わないでください。
- 推測を述べる場合は「推測：」と明示し、事実と分けてください。
- 数値には単位・対象期間・発表主体を必ず付けてください。
- 情報の日付が古い（12か月以上前）場合はその旨を注記してください。
```

ただし、プロンプトだけで誤りを防ぐことはできません。**最終的な防波堤は人間による検証** です。

### 6.3 3段階ファクトチェック手順

#### ステップ1：トリアージ（どの主張を検証するか決める）

すべてを検証するのは非現実的です。次の主張を優先します。

- 数値（市場規模、性能スコア、価格、ユーザー数、資金調達額）
- 日付・時系列（発表日、施行日、リリース日）
- 固有名詞（人名、社名、製品名、論文名）
- 結論を左右する主張（「AはBより優れている」「規制で禁止された」）
- 意外性の高い主張（直感に反するものほど誤りの可能性が高い）

#### ステップ2：一次情報への遡り

```text
主張 → AIが示した出典 → その出典が引用している元 → … → 起点（一次情報）
```

| 主張の種類 | 一次情報の例 |
|---|---|
| 企業の発表・製品機能 | 公式ブログ、プレスリリース、公式ドキュメント、Changelog |
| 業績・財務 | 決算短信、有価証券報告書、10-K/10-Q、IR資料 |
| 法規制 | 法令本文（e-Gov法令検索、EUR-Lex等）、官公庁の公式発表・ガイドライン |
| 研究結果 | 論文本文（アブストラクトだけでなく実験条件・限界の節） |
| 統計 | 政府統計（e-Stat等）、国際機関のデータ |
| 発言 | 本人の投稿原文、講演の録画、インタビュー全文 |

**遡りのテクニック:**

- 記事内の「〜によると」を探し、そのリンクを辿る。リンクがなければ発表主体の公式サイトで検索する。
- 数値は、数値そのものと単位でサイト内検索する（`site:` 演算子の併用）。
- 論文はDOIまたはarXiv IDで本文を開き、図表の数値と照合する。
- 公式ページが更新されている場合は、Internet Archive（Wayback Machine）で当時の版を確認する。

#### ステップ3：クロスチェックと記録

- **独立した2つ以上の情報源で一致するか**（同じ元記事のコピーは「独立」ではない）。
- **別のAIに反証させる:** 「この主張の反例・反論・誤りの可能性を探してください」と依頼する。
- **結果を記録:** ノートの `reliability` を更新し、確認した一次情報のURLと確認日を残す。

```text
【反証用プロンプト例】
以下は別のAIが作成した調査レポートの要点です。
あなたの役割は「批判的なレビュアー」です。
1. 事実誤認の可能性がある主張を列挙し、理由を述べてください
2. 出典が二次情報のみの主張を指摘してください
3. 反対の見解や異なる数値を示す情報源を探してください
4. 各指摘に対して、確認すべき一次情報の種類を提案してください
同意できる部分を褒める必要はありません。
```

### 6.4 ファクトチェック記録テンプレート

```markdown
| # | 主張 | AI出典 | 一次情報 | 一致 | 確認日 | 備考 |
|---|---|---|---|---|---|---|
| 1 | NotebookLMは2026年7月にGemini Notebookへ改称 | 比較記事 | Google公式ブログ | ○ | 2026-10-01 | 発表日 7/16 |
| 2 | ○○の市場規模は△△億円 | まとめ記事 | 調査会社の公開サマリー | △ | 2026-10-01 | 対象範囲の定義が異なる |
| 3 | ○○論文で精度95% | AI回答 | 該当論文が見つからない | × | 2026-10-01 | 引用ハルシネーションの疑い |
```

### 6.5 情報源そのものを評価する視点

- **誰が書いたか:** 署名の有無、専門性、所属。
- **なぜ書いたか:** 宣伝、販売、PV目的、研究成果の発表など、利害関係。
- **いつ書いたか:** 公開日・更新日。AI分野では半年前の情報でも古い場合がある。
- **何に基づくか:** データ・一次情報へのリンクの有無。
- **他と比べてどうか:** 同じテーマの他の情報源との整合性。

> AI分野では特に、**ベンチマークスコア** の扱いに注意が必要です。評価条件（プロンプト、試行回数、ツール使用の有無）が異なる数値を並べて比較している記事が多く、公式発表のスコアであっても条件を確認しないと比較できません。

---

<a id="ch7"></a>
## 第7章 AI分野を追うための情報源・発信者（日英）

ここでは、AI分野の動向を追うための代表的な情報源を挙げます。いずれも広く知られたものを選んでいますが、更新頻度・有料化・運営状況は変わりうるため、購読前に公式サイトで確認してください。**X（旧Twitter）のアカウント名やURLは、なりすましも多いため、本人の公式サイト等からリンクを辿って確認してください**（本レポートではハンドル名の記載を控えます）。

### 7.1 一次情報（最優先）

| 種類 | 情報源 | ポイント |
|---|---|---|
| 公式ブログ・ニュース | OpenAI、Google（DeepMind／Google AI／Gemini）、Anthropic、Meta AI、Microsoft、NVIDIA、Mistral AI、xAI、国内各社（例: Sakana AI、Preferred Networks、ELYZA など） | 製品発表・モデル公開の起点。RSSがあれば必ず購読 |
| Changelog・リリースノート | 各API・製品のリリースノート、GitHubのReleases | 機能変更・料金改定の一次情報 |
| 論文 | arXiv（cs.AI / cs.CL / cs.LG / cs.CV）、主要国際会議（NeurIPS、ICML、ICLR、ACL等）の論文 | 研究の一次情報。プレプリントは査読前 |
| 政策・規制 | 日本: 内閣府・総務省・経済産業省・デジタル庁・AISI（AIセーフティ・インスティテュート）等の公表資料。海外: EUのAI Act関連文書、米国の行政機関の発表 | 規制動向は必ず原文で |

### 7.2 論文・研究動向のキュレーション

- **arXiv RSS／API:** カテゴリ単位で購読可能。量が多いのでLLMでのフィルタ前提。
- **Hugging Face Daily Papers（huggingface.co/papers）:** コミュニティが注目論文を投票で選ぶ日次リスト。
- **alphaXiv:** arXiv論文にコメント・議論できるサービス。
- **Semantic Scholar:** 推薦フィード、著者フォロー、API。
- **Papers with Code:** かつて定番でしたが、2025年にサービス終了・移行が報じられています（現状は要確認）。

### 7.3 ニュースレター（英語）

| 名称 | 発行者 | 特徴 |
|---|---|---|
| Import AI | Jack Clark（Anthropic共同創業者） | 研究・政策の論点を深く解説。週刊 |
| The Batch | DeepLearning.AI（Andrew Ng） | 週次のAIニュースと解説 |
| Latent Space | swyx、Alessio Fanelli | AIエンジニア向け。ポッドキャストも |
| Interconnects | Nathan Lambert | オープンモデル・RLHF・ポストトレーニングの深掘り |
| One Useful Thing | Ethan Mollick（ペンシルベニア大学ウォートン校） | AIの実務・教育での活用 |
| Simon Willison's Weblog | Simon Willison | LLMツールの実践的な検証記事。更新頻度が高い |
| Ben's Bites | Ben Tossell | 起業家・ビルダー向けの日次〜週次ダイジェスト（形態は要確認） |
| TLDR AI | TLDR | 短い要約の日刊ニュースレター |
| The Rundown AI | Rowan Cheung | 一般向けの日刊ニュース |
| Last Week in AI | Skynet Today系 | 週次まとめ。ポッドキャストもあり |

### 7.4 ポッドキャスト・YouTube（英語）

- **ポッドキャスト:** Latent Space、Dwarkesh Podcast（研究者・経営者への長尺インタビュー）、Lex Fridman Podcast、Hard Fork（The New York Times）、Last Week in AI、No Priors（要確認）、The Cognitive Revolution（要確認）
- **YouTube:** Andrej Karpathy（LLMの仕組みを解説する長尺講義）、Two Minute Papers、Yannic Kilcher（論文解説）、AI Explained、3Blue1Brown（ニューラルネットの数理的解説）
- **活用法:** 長尺動画は第5章の `video-digest` スキルで要約し、気になるセクションだけ視聴する。

### 7.5 コミュニティ・アグリゲーター

- **Hacker News（news.ycombinator.com）:** 技術者コミュニティ。コメント欄に専門家の指摘が集まることが多い。`hnrss.org` でポイント閾値付きRSSを作れる。
- **Reddit:** r/MachineLearning、r/LocalLLaMA（ローカルLLM）など。
- **GitHub Trending:** 新しいOSSツールの発見。
- **Hugging Face:** 新モデル・データセット・Spacesのトレンド。
- **LMArena（旧Chatbot Arena）等のリーダーボード:** モデル比較の参考（評価方法の特性を理解した上で）。

### 7.6 日本語の情報源

| 種類 | 情報源 | 特徴 |
|---|---|---|
| ニュースメディア | ITmedia AI+、日経クロステック、Impress Watch系、GIGAZINE、ASCII.jp | 国内外のAIニュースを日本語で。速報性が高い |
| ビジネス系 | 日本経済新聞、東洋経済オンライン、NewsPicks | 産業・投資・政策の観点 |
| 技術コミュニティ | Zenn、Qiita、note（例: npaka氏による各種AIツール・APIの解説記事） | 実装・検証記事。品質にばらつきがあるので著者を見て選ぶ |
| 企業技術ブログ | 国内IT企業・AIスタートアップのテックブログ | 実運用の知見 |
| 研究者・発信者 | 松尾豊氏（東京大学）とその研究室関係者、今井翔太氏、深津貴之氏、清水亮氏 など | 解説・論評・実践知。アカウントは公式プロフィール経由で確認（要確認） |
| 公的機関 | AISI（AIセーフティ・インスティテュート）、IPA、総務省・経産省のAI関連ガイドライン | 規制・ガバナンスの一次情報 |
| ポッドキャスト | 国内のAI・テック系ポッドキャスト各種 | 具体的な番組名は更新が多いため要確認 |

> 日本語の情報は英語圏の発表から数時間〜数日遅れることが多いため、**速報は英語の一次情報、解釈と国内文脈は日本語メディア** という使い分けが効率的です。

### 7.7 情報源ポートフォリオの作り方

「全部追う」のではなく、次の比率を目安に絞ります。

- **一次情報（40%）:** 主要ベンダー公式ブログ5〜10、arXivの関心カテゴリ（LLMでフィルタ）
- **良質なキュレーター（40%）:** ニュースレター3〜5本（英2〜3、日1〜2）
- **コミュニティ（15%）:** Hacker News（100ポイント以上）、Hugging Face Papers
- **SNS・動画（5%）:** 信頼する発信者を10〜20人程度に絞ったリスト

3か月ごとに「実際に役立った情報源」を棚卸しし、使っていないものは解除します。

---

<a id="ch8"></a>
## 第8章 目的別レシピ

### レシピ1：業界調査（例：国内の生成AI導入支援市場）

**所要時間の目安:** 半日〜2日（深さによる）

1. **問いの設計（15分）:** 目的・範囲・期間・アウトプット形式を決める。第3章のDeep Research依頼テンプレートを使う。
2. **当たり付け（15分）:** Perplexity／Feloで主要プレイヤー、関連キーワード、主要な調査レポート名を洗い出す。
3. **広く集める（30〜60分・並行実行）:** ChatGPT Deep ResearchとGemini Deep Researchに同じ依頼を投げる。
4. **一次資料の収集（1〜2時間）:** 両レポートの出典から一次情報（IR資料、官公庁資料、調査会社の公開サマリー）をダウンロード。
5. **深く読む（1〜2時間）:** NotebookLM（Gemini Notebook）に一次資料を入れ、比較表・論点整理を作る。
6. **検証（1時間）:** 数値・日付・固有名詞を第6章の手順で確認。2つのDeep Researchで食い違った箇所を重点確認。
7. **まとめ（1時間）:** 「確度の高い事実」「推計・見解」「不明点」に分けてレポート化。

```text
【業界構造の整理プロンプト（NotebookLM用）】
ソースをもとに、この業界のバリューチェーンを
「基盤モデル提供 → インフラ → 開発・導入支援 → SaaS → ユーザー企業」
の層で整理し、各層の主要プレイヤー、収益モデル、参入障壁を表にしてください。
ソースにない情報は「ソースに記載なし」とすること。
```

### レシピ2：競合調査（例：自社SaaSの競合5社）

1. **競合リストの確定:** AI検索で候補を出し、人間が最終決定する。
2. **定点観測の設定:** 各社の公式ブログ・リリースノート・プレスリリースのRSSをFeedlyまたは自作パイプラインに登録。RSSがない場合はFirecrawl／Tavilyのクロール、またはページ監視で変更検知。
3. **プロファイル作成:** Firecrawlで各社の製品ページ・価格ページ・導入事例をMarkdown化し、NotebookLMまたはClaude Projectsに投入。
4. **比較表:** 機能・価格・ターゲット・導入事例・採用情報（注力領域の推測材料）を比較。
5. **差分ウォッチ:** 週次で「先週からの変更点」をLLMに抽出させ、Slackに配信。

```python
# 競合ページの変更検知（前回取得分との差分をLLMに要約させる）
import difflib, hashlib, pathlib, requests

def snapshot(url):
    # 実運用ではFirecrawl等で本文をMarkdown化してから比較するとノイズが減る
    return requests.get(url, timeout=20).text

def diff_and_store(name, url):
    p = pathlib.Path(f"snapshots/{name}.txt"); p.parent.mkdir(exist_ok=True)
    new = snapshot(url)
    old = p.read_text() if p.exists() else ""
    p.write_text(new)
    if hashlib.md5(old.encode()).digest() == hashlib.md5(new.encode()).digest():
        return None
    return "\n".join(difflib.unified_diff(old.splitlines(), new.splitlines(), lineterm="", n=0))[:8000]
# 差分テキストをLLMに渡し「価格・機能・訴求の変更点」を要約させる
```

```text
【競合差分の要約プロンプト】
以下は競合A社の価格ページの前回からの差分です。
1. 価格・プラン構成の変更
2. 機能の追加・削除
3. 訴求メッセージの変化
を箇条書きで示し、自社への示唆を1〜2行で述べてください。
差分にない変更を推測しないでください。HTMLの装飾的な変更は無視してください。
```

> 競合サイトのクロールは、各サイトの利用規約とrobots.txtを守り、アクセス頻度を抑えてください。

### レシピ3：論文サーベイ（例：LLMエージェントの評価手法）

1. **キーワード設計:** 英語キーワードと同義語を作る（例: "LLM agent evaluation", "agent benchmark", "tool-use evaluation"）。
2. **探索:** Elicit／Consensus／Semantic Scholarで20〜50本、arXivで直近6〜12か月の新着を追加。第5章の `arxiv-survey` スキルを使うとエージェントに一次選別を任せられる。
3. **地図作り:** 被引用数の多い論文・サーベイ論文を起点に、Connected Papers等で関連を可視化。
4. **精読:** 重要論文10本程度をNotebookLM／SciSpaceに入れ、「手法・データセット・評価指標・限界」を表に。
5. **書誌確認:** 全論文のarXiv ID／DOIの実在を確認し、Zoteroに登録。
6. **統合:** 研究の流れ、未解決課題、自分の研究・業務への示唆をまとめる。

```text
【論文比較表の作成プロンプト】
アップロードした論文それぞれについて、以下を表にしてください。
列：論文（著者, 年）/ 解く課題 / 提案手法 / 評価データセット・ベンチマーク / 主な結果（数値は表・図の番号付き）/ 著者が述べる限界
- 本文から確認できない項目は「記載なし」
- 数値は論文中の表番号を必ず併記
最後に、手法を3〜4のグループに分類し、その基準を説明してください。
```

### レシピ4：毎朝のニュース（15分ルーティン）

**前日夜〜早朝（自動）:**

- 6:30 n8n／GitHub Actionsが RSS・HN・arXiv・ニュースレターを収集し、LLMで要約・スコアリング
- 7:00 Slack `#ai-digest-daily` に重要度3以上のトップ10を配信、重要度5は `#ai-alert` にも
- 同時にNotionデータベース／Obsidianの `00_Inbox` に保存

**朝（人間・15分）:**

1. （5分）ダイジェストを流し読みし、気になる項目にリアクション
2. （5分）重要度5の項目だけ一次情報（公式発表）を開いて確認
3. （5分）深掘りしたいものをReadwise Readerに送る／担当者を割り振る

**週末（30分）:**

- 週次まとめを読み、トピックノートを更新
- スコアリングの外れ（重要なのに低評価、など）を確認し、プロンプトを調整

```text
【朝のダイジェストを読んだ後の深掘りプロンプト（Perplexity等）】
次のニュースについて、(1) 公式発表の原文、(2) 発表内容の要点、
(3) 前回の発表からの変更点、(4) 実務への影響、を調べてください。
ニュース：「（ダイジェストの見出しとURL）」
公式発表が見つからない場合は、その旨を明記してください。
```

---

<a id="ch9"></a>
## 第9章 チェックリスト集

### 9.1 仕組みづくりチェックリスト

- [ ] 情報収集の目的（何の意思決定に使うか）を書き出した
- [ ] 一次情報源（公式ブログ・リリースノート・arXivカテゴリ）をRSS登録した
- [ ] ニュースレターを専用アドレス／ラベルに集約した
- [ ] LLM要約・スコアリングのパイプライン（n8n／Make／Zapier／自作）が毎朝動いている
- [ ] 重複排除と重要度フィルタを入れている
- [ ] Slack／Discordの配信チャンネルを用途別に分けた
- [ ] Obsidian／Notionに出典URL・取得日・検証状況付きで保存されている
- [ ] APIキーをコードやリポジトリに直書きしていない
- [ ] 月次のAPI費用を確認する仕組みがある
- [ ] 3か月ごとに情報源を棚卸しする予定を入れた

### 9.2 調査実行チェックリスト

- [ ] 問い・範囲・期間・出力形式を明文化した
- [ ] 日本語と英語の両方で検索した
- [ ] 2つ以上のツール（例: Deep Research 2種）で結果を比較した
- [ ] 調査計画の段階で観点の抜け漏れを確認した
- [ ] 一次情報をダウンロードし、ソース限定ツールで精読した
- [ ] 結論と根拠の対応表を作った

### 9.3 ファクトチェック・チェックリスト

- [ ] 数値・日付・固有名詞・結論を左右する主張を抽出した
- [ ] 各主張の出典リンクを開き、該当記述が実在することを確認した
- [ ] 一次情報（公式・論文・法令・IR）まで遡った
- [ ] 論文のDOI／arXiv ID、URLの実在を確認した
- [ ] 独立した2つ以上の情報源で一致を確認した
- [ ] 情報の日付を確認し、古い情報には注記した
- [ ] 発信者の利害関係を確認した
- [ ] 確認できない情報に「要確認」「未確認」と明記した
- [ ] 別のAIまたは同僚に反証レビューを依頼した

### 9.4 MCP／エージェント運用チェックリスト

- [ ] 公式提供のMCPサーバーを優先し、コミュニティ製はコードと権限を確認した
- [ ] 書き込み系ツールは承認制にした
- [ ] ファイルアクセスは特定フォルダに限定した
- [ ] Skillに「取得していない内容を書かない」「URLを捏造しない」を明記した
- [ ] 検索回数の上限をSkillで定めた
- [ ] 取得コンテンツ内の指示に従わない（プロンプトインジェクション対策）ことを明記した

---

<a id="appendix"></a>
## 付録 プロンプト集・用語集

### A. すぐ使えるプロンプト集

```text
【1. 調査の問いを磨く】
私は「（テーマ）」について調べたいと考えています。目的は「（目的）」です。
調査を始める前に、(1) 問いを具体化するための質問を5つ、(2) 想定される論点の一覧、
(3) 調べるべき一次情報の種類、を提案してください。

【2. 用語の地図を作る】
「（分野）」の主要概念を20個挙げ、それぞれ1行で定義し、概念同士の関係（上位・下位・対立）を
Mermaidのグラフ記法で示してください。定義は一般的なものに限り、出典があれば付けてください。

【3. 長文資料の要約（構造化）】
この資料を、(1) 結論、(2) 根拠となるデータ、(3) 前提条件・限界、(4) 筆者の立場・利害、
の4つに分けて要約してください。各項目に資料内の該当箇所を示してください。

【4. 英語記事の日本語ブリーフィング】
次の英語記事を、日本のビジネスパーソン向けに日本語で300字に要約し、
日本市場への示唆を2点述べてください。原文の固有名詞は英語表記を併記してください。

【5. 矛盾の検出】
次の2つのレポートで主張が食い違う箇所を表にし、どちらの根拠がより一次情報に近いかを評価してください。
```

### B. 用語集

| 用語 | 説明 |
|---|---|
| AI検索（回答エンジン） | 検索結果をAIが読み、出典付きで回答を生成する検索サービス |
| Deep Research | AIが調査計画を立て、多数のWebページを読み込んで長文レポートを作る機能 |
| RAG | Retrieval-Augmented Generation。検索で取得した資料を根拠にLLMが回答する手法 |
| 埋め込み（Embedding） | 文章を意味的な類似度を計算できるベクトルに変換したもの |
| MCP | Model Context Protocol。AIアプリと外部ツール・データをつなぐ標準プロトコル |
| Agent Skills | SKILL.mdを中心に手順・スクリプト・参考資料をまとめた、エージェントの再利用可能な能力パッケージ |
| ハルシネーション | AIが事実に基づかない内容をもっともらしく生成すること |
| 一次情報 | 当事者が直接発信した情報（公式発表、論文本文、法令、決算資料など） |
| プロンプトインジェクション | 外部コンテンツに埋め込まれた指示でAIの挙動を乗っ取る攻撃 |

### C. 本レポートで確認した主な情報（2026年10月1日時点のWeb検索による）

- Google公式ブログ「NotebookLM is now Gemini Notebook」（2026年7月16日）、「Do your best research with NotebookLM」、Google Workspace Updates（2026年3月）
- Genspark公式ブログ「Introducing Genspark AI Workspace 4.0」、Business Wireのプレスリリース（2026年4月8日）
- Feedly Changelog（2026年4月15日）、Feedly AIページ
- Readwise「Reader Public Beta Update #14」（2026年8月）、Readwise Docs「Global Ghostreader」
- Glasp「The Highlights — July 2026」
- Anthropic「Equipping agents for the real world with Agent Skills」（2025年12月18日にオープン標準化を追記）、agentskills/agentskills の仕様
- GitHub: brave/brave-search-mcp-server、exa-labs/exa-mcp-server、tavily-ai/tavily-mcp、firecrawl/firecrawl-mcp-server
- Deep Research比較（第三者記事）: Presenc AI、Nesyona、aitechrankings.com、aitoolradar.io、diyai.io

> 料金、プランごとの利用上限、ベースモデル名、コミュニティ製MCPサーバーの所在、個人のSNSアカウントなどは本レポートでは確定させていません。利用前に各公式情報で **要確認** です。
