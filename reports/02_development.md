# AIを活用したソフトウェア開発の効率的なやり方 ― 2026年10月版 実践レポート

> 作成日: 2026年10月1日
> 対象読者: AIコーディングツールを業務に本格導入したい個人開発者・チームリード・エンジニアリングマネージャー
> 注意: AI開発ツールの領域は月単位で仕様・価格・名称が変わります。本文中の **『要確認』** は、執筆時点で一次情報を十分に確認できなかった、または変化が特に速い情報です。導入前に必ず公式ドキュメントを確認してください。

---

## 目次

- [3分で分かる要約](#summary)
- [第1章 2026年のAI開発の全体像](#ch1)
- [第2章 AIコーディングツール比較と使い分け](#ch2)
- [第3章 ルールファイル設計（AGENTS.md / CLAUDE.md / .cursor/rules）](#ch3)
- [第4章 Agent Skills（SKILL.md）の設計](#ch4)
- [第5章 サブエージェント・hooks・カスタムコマンド](#ch5)
- [第6章 有用なMCPサーバーと設定例](#ch6)
- [第7章 ワークフロー（仕様駆動開発・TDD×AI・並列エージェント・worktree・レビュー自動化・CI連携）](#ch7)
- [第8章 コスト管理](#ch8)
- [第9章 セキュリティ（プロンプトインジェクション・秘密情報）](#ch9)
- [第10章 品質担保](#ch10)
- [第11章 情報源・発信者ガイド（日英）](#ch11)
- [第12章 すぐ試せるチェックリスト](#ch12)
- [第13章 段階的導入ロードマップ](#ch13)
- [付録 参考URL一覧](#appendix)

---

<a id="summary"></a>
## 3分で分かる要約

1. **主役は「補完」から「エージェント」に移った。** 2026年のAI開発は、エディタ内で1行ずつ補完してもらう段階をとうに過ぎています。いまの中心は、タスクを渡すとファイルを読み、コマンドを実行し、テストを回し、プルリクエスト（PR）まで作る「コーディングエージェント」です。Cursor、Claude Code、OpenAI Codex、GitHub Copilot（coding agent / CLI）、Gemini CLI、Devin（旧Windsurfを含む）などが該当します。
2. **ツール選びより「コンテキスト設計」のほうが効く。** 同じモデルを使っても、リポジトリに置くルールファイル（`AGENTS.md`、`CLAUDE.md`、`.cursor/rules`）、再利用できる手順書（Agent Skills / `SKILL.md`）、外部情報への接続（MCP）、そして自動検証（テスト、lint、hooks）を整えているかどうかで成果は大きく変わります。
3. **Agent Skillsが共通の標準になりつつある。** `SKILL.md`（YAMLフロントマター＋Markdown本文）は、Claude Code・Codex・Cursorなどで共通に読めるオープン標準として整備されてきました（仕様: agentskills.io）。必要になったときだけ全文を読み込む「段階的開示」の仕組みなので、コンテキストを圧迫せずに多くのノウハウを持たせられます。
4. **効率化の鍵は「人間が検証しやすい単位に切ること」。** 仕様を先に書く（仕様駆動）、テストを先に書く（TDD×AI）、小さなPRに分ける、git worktreeやクラウドエージェントで並列に回す、AIレビューと人間レビューを組み合わせる――が定番の型です。
5. **コストとセキュリティは最初から設計に入れる。** 多くのツールは定額制から「定額＋使用量（トークン・クレジット）課金」に移っており、並列実行するほど費用が増えます。リポジトリ内の文書、Issue、Webページ、MCPの応答に紛れ込むプロンプトインジェクションや、秘密情報の流出は現実のリスクです。最小権限、サンドボックス、秘密情報のスキャン、人間による承認を組み合わせて守ります。
6. **導入は段階的に。** 「個人の補完・チャット利用」→「ルールファイル整備」→「エージェントに小タスクを任せる」→「Skills・MCP・hooksで標準化」→「並列・クラウド・CI連携」→「組織で計測・ガバナンス」の順に進めると、失敗が少なくなります（[第13章](#ch13)）。

**最初にやること3つ:** (1) リポジトリ直下に `AGENTS.md` を作り、ビルド・テスト・lintのコマンドとコーディング規約を書く。(2) 小さめのバグ修正を1件、エージェントに「テストを先に書いて失敗を確認し、その後修正」と指示して任せてみる。(3) 秘密情報を `.env` に分け、エージェントから読めないよう除外設定する。

---

<a id="ch1"></a>
## 第1章 2026年のAI開発の全体像

### 1.1 三つの利用形態

AI開発ツールは、使い方で大きく三つに分けられます。

| 形態 | 典型例 | 人間の関与 | 向くタスク |
|---|---|---|---|
| 補完・インライン編集 | Copilotの補完、CursorのTab | 常時（1行〜数十行単位） | 定型コード、リネーム、小修正 |
| 対話型ローカルエージェント | Cursor Agent、Claude Code、Codex CLI、Gemini CLI、Cline/Roo、Aider | 数分〜数十分ごとに確認 | 機能追加、リファクタ、デバッグ |
| 非同期クラウドエージェント | Cursor Cloud Agents、Codex（クラウド）、Copilot coding agent、Devin、Claude Codeのクラウド実行 | タスクを渡してPRで受け取る | 明確に定義された中小タスクの並列処理 |

ここに、プロトタイプを素早く作る **アプリ生成系**（v0、Bolt.new、Lovable、Replit Agentなど）が加わります。

### 1.2 「効率的」とは何か

AI導入で効率化を測るとき、「コードを書く速さ」だけを見ると失敗しがちです。AIは大量のコードを一瞬で書けるので、ボトルネックはすぐに **レビュー・検証・統合** に移ります。2025〜2026年に各所で語られてきた教訓をまとめると、次のようになります。

- **生成速度 ≠ 開発速度。** レビューできない量のコードが出てくると、技術的負債が溜まるだけです。
- **検証の自動化が生産性の上限を決める。** テスト、型チェック、lint、E2Eが充実したリポジトリほど、エージェントの自律度を上げられます。
- **タスクの切り方が成果を決める。** 「何をもって完了とするか」が明確なタスクほど、エージェントの成功率は上がります。
- **コンテキストは資産になる。** ルール・Skills・ドキュメントは一度書けば全員と全エージェントが使えます。

### 1.3 2026年に起きた主な変化（執筆時点で確認できた範囲）

- **Agent Skillsの標準化:** `SKILL.md` 形式がオープン標準（agentskills.io）として公開され、Claude Code、OpenAI Codex、Cursorの公式ドキュメントがこの標準への準拠を明記しています。Cursorは `.claude/skills/` や `.codex/skills/` も互換のため読み込むとドキュメントに書いています。
- **WindsurfからDevin Desktopへ:** 複数の二次情報によると、Cognitionは2026年6月にWindsurfエディタを「Devin Desktop」に改名し、Devinのクラウドセッション・CLIと料金プランを一本化しました（**要確認**: 正確な日付とプラン内容は公式サイトで確認してください）。
- **ACP（Agent Client Protocol）の普及:** Zedが提唱したエディタとエージェントをつなぐプロトコルで、JetBrains、Gemini CLI、Copilot CLIなどが採用したと報じられています（**要確認**）。
- **料金の使用量課金化:** 主要ツールの多くが、固定のリクエスト数から、トークンやクレジットに基づく従量部分を含む料金体系に移っています（具体額は[第8章](#ch8)）。


---

<a id="ch2"></a>
## 第2章 AIコーディングツール比較と使い分け

> 価格・プラン名・対応モデルは頻繁に変わります。下表の価格は2026年8〜9月時点の二次情報を含むため、すべて **要確認** として扱ってください。

### 2.1 一覧比較

| ツール | 形態 | 強み | 弱み・注意 | 参考価格（個人・要確認） |
|---|---|---|---|---|
| **Cursor** | VS Code系IDE＋CLI＋クラウドエージェント | Tab補完の質、Agentモード、複数モデル選択、Rules/Skills/hooks/MCP、Cloud Agents、Bugbot（PRレビュー） | 使用量課金のため、高性能モデルを多用すると費用が膨らむ | Pro $20/月、上位にPro+・Ultra |
| **Claude Code** | ターミナルCLI（IDE拡張・デスクトップ・Web版あり） | 自律的に長いタスクをこなす力、サブエージェント、hooks、Skills、プラグイン、ヘッドレス実行（`-p`）、GitHub Actions連携 | ターミナル操作に慣れが必要。基本はAnthropicのモデル | Pro $20/月、Max $100〜/月 |
| **OpenAI Codex** | CLI（OSS）＋IDE拡張＋クラウド＋アプリ | ChatGPTのプランに含まれる、クラウドで並列実行してPR作成、`AGENTS.md` の普及を主導、Skills対応 | 機能の名称や構成の変化が速い | ChatGPT Plus $20/月〜 |
| **GitHub Copilot** | IDE補完＋Chat＋coding agent＋CLI＋コードレビュー | GitHubとの統合（IssueをアサインするとPRが来る）、企業向けの管理機能、価格の予測しやすさ | 最先端の機能はCursorやClaude Codeに遅れることがある | Pro $10/月、Pro+ $39/月 など |
| **Devin Desktop（旧Windsurf）/ Devin** | IDE＋CLI＋自律クラウドエージェント | Slack・Linear・Jiraからタスクを渡せる、IDEとクラウドで1つの契約 | 改名直後のため、機能名や移行状況が流動的 | Pro $20/月、Max $200/月（要確認） |
| **Gemini CLI** | ターミナルCLI（OSS） | 無料枠が大きい、長いコンテキスト、Google Cloud連携、拡張機能 | 複雑なタスクの安定性はモデル世代による | 個人Googleアカウントで無料枠あり（要確認） |
| **Cline / Roo Code** | VS Code拡張（OSS） | 好きなAPI・モデルを持ち込める（BYOK）、Plan/Actの切替、モード定義、MCP連携 | APIの従量課金を自分で管理する必要がある | 拡張は無料＋API費用 |
| **Aider** | ターミナルCLI（OSS） | gitと深く統合（変更ごとに自動コミット）、repo map、多数のモデルに対応、軽量 | GUIなし。大規模な自律タスクには工夫が必要 | 無料＋API費用 |
| **v0 / Bolt.new / Lovable / Replit Agent** | ブラウザでのアプリ生成 | UIプロトタイプやMVPを数分で作れる、デプロイまで一気通貫 | 生成物を本番品質に育てるには、結局エンジニアの手が要る | 無料枠＋月額（要確認） |

その他の選択肢: **Zed**（ACP対応の高速エディタ）、**JetBrains AI Assistant / Junie**、**Amazon Q Developer / Kiro**（仕様駆動を前面に出すAWSのIDE。名称や提供形態は要確認）、**OpenCode**（OSSのターミナルエージェント）、**Continue**（OSSのIDE拡張）、**Kilo Code**（Cline/Roo系の派生、要確認）など。

### 2.2 それぞれの特徴を少し詳しく

#### Cursor
VS Codeをフォークした「AIネイティブIDE」です。Tab補完、Agent、Plan（計画を立ててから実装するモード）、背景で動くCloud Agents、PRを自動レビューするBugbot、CLI（`cursor-agent`、名称は要確認）を備えています。プロジェクトルールは `.cursor/rules/*.mdc`（または `AGENTS.md`）、Skillsは `.cursor/skills/` または `.agents/skills/` に置き、MCPは `.cursor/mcp.json` で設定します。複数のベンダーのモデルを切り替えられるので、「計画は推論の強いモデル、実装は速いモデル」のような使い分けがしやすいのが特徴です。

#### Claude Code
Anthropic製のエージェント型CLIです。`CLAUDE.md`（階層的に読み込まれるメモリ）、`.claude/agents/`（サブエージェント）、`.claude/skills/`、`.claude/commands/`（カスタムスラッシュコマンド。現在はSkillsへの統合が進んでいる）、`settings.json` のhooksと権限設定、プラグイン・マーケットプレイス、`claude -p` によるヘッドレス実行、Claude Code GitHub Actionsなど、拡張点が非常に豊富です。長い自律作業に強く、「ターミナルで動くシニアエンジニア」のように使えます。公式ドキュメントによると、Skillsは標準に加えて、呼び出し制御・サブエージェントでの実行・動的なコンテキスト注入などの独自拡張を持っています。

#### OpenAI Codex
CLI版（OSS、Rust製）、IDE拡張、クラウド版（ChatGPTから起動してPRを受け取る）、デスクトップアプリがあります。`AGENTS.md` を読み、Skillsは `.agents/skills/` を作業ディレクトリからリポジトリルートまで遡って探します（公式ドキュメントより）。`$スキル名` での明示呼び出しや、`agents/openai.yaml` による暗黙呼び出しの制御（`allow_implicit_invocation`）にも対応しています。サンドボックスと承認モードの設定がはっきりしているのも特徴です。

#### GitHub Copilot
すでにGitHub Enterpriseを使っている組織にとって、最も導入しやすい選択肢です。IssueをCopilotにアサインすると、GitHub Actions上の環境で作業してドラフトPRを作る「coding agent」、PRの自動レビュー、ターミナル用のCopilot CLI、`.github/copilot-instructions.md` と `AGENTS.md` の読み込み、MCP対応などがあります。管理者がポリシーでモデルや機能を制御できる点が、企業では大きな利点です。

#### Devin Desktop（旧Windsurf）/ Devin
Cognitionの製品群です。旧Windsurfの「Cascade」に代わって、エージェントの指揮画面（Agent Command Center）を中心にしたIDEになったと報じられています（要確認）。クラウドのDevinはSlackやLinearから「チケットを渡す」使い方に向いています。仕様が明確で、文脈への依存が少ない作業を並列でこなすのが得意です。

#### Gemini CLI
GoogleのOSSのターミナルエージェントです。`GEMINI.md` でコンテキストを与え、MCPと拡張機能（extensions）に対応しています。無料枠が大きいので、個人の学習や大きなコードベースの読解・要約に向きます。GitHub Actions連携（要確認）も提供されています。

#### Cline / Roo Code
VS Code拡張のOSSエージェントです。好きなAPIキー（Anthropic、OpenAI、OpenRouter、ローカルモデルなど）を使え、Plan/Actの切替や、操作ごとの承認がはっきりしているため、「AIが何をしているか」を把握したまま使えます。Rooは「モード」（Architect、Code、Debugなど）をカスタム定義できるのが特徴です。ルールは `.clinerules` / `.roo/rules`（要確認）に置きます。

#### Aider
gitを中心に据えたOSSのCLIです。変更ごとに意味のあるコミットメッセージで自動コミットするので、`git diff` と `git reset` で気軽にやり直せます。`--architect` モード（推論モデルが設計し、編集モデルが書く）や、各社モデルの成績を比べる公開ベンチマーク（Aider Polyglot Leaderboard）でも知られています。

#### v0 / Bolt.new / Lovable
「動くもの」を最短で見せたいときに使います。v0（Vercel）はReact/Next.js＋shadcn/uiのUI生成に強く、Bolt.new（StackBlitz）はブラウザ内でフルスタックを動かせ、LovableはSupabase連携を含むアプリ生成とデプロイが得意です。現実的な使い方は、**生成したコードをGitHubに書き出し、以降はCursorやClaude Codeで本番品質に育てる** という二段構えです。

### 2.3 使い分けの指針

| 状況 | 推奨 |
|---|---|
| 個人開発で、まず1本選ぶなら | Cursor（IDE派）か Claude Code（ターミナル派）。ChatGPTを契約済みならCodexも有力 |
| GitHub中心の企業で、ガバナンスを重視 | GitHub Copilot（Business/Enterprise）を基盤にし、必要に応じて他ツールを追加 |
| 長く自律的なリファクタや調査 | Claude Code / Codex CLI / Cursor Agent（推論の強いモデル） |
| 明確な小タスクを大量に並列処理 | クラウドエージェント（Cursor Cloud Agents、Codexクラウド、Copilot coding agent、Devin） |
| モデルやAPIを自由に選びたい・費用を細かく管理したい | Cline / Roo Code / Aider / OpenCode（BYOK） |
| コストを抑えて試したい・巨大なコードベースを読みたい | Gemini CLI |
| デモ・MVP・UIの叩き台 | v0 / Bolt.new / Lovable → GitHubに書き出して通常開発へ |
| オフライン・機密性の高い環境 | ローカルモデル（Ollamaなど）＋Aider/Cline/Continue。ただし性能とのトレードオフあり |

**複数ツールの併用は普通のことです。** ルールを `AGENTS.md` に、手順を `SKILL.md` に集めておけば、ツールを替えても資産を持ち越せます。これがツールに縛られないための最大のコツです。

### 2.4 モデル選びの考え方

- **計画・設計・難しいデバッグ:** 推論の強い最上位モデル（各社のフラッグシップ。具体的なモデル名とバージョンは変化が速いので要確認）。
- **定型的な実装・テスト追加・大量の小修正:** 速くて安いモデル。
- **レビュー:** 実装したのとは別系統のモデルに見せると、見落としを拾いやすくなる（「別の目」効果）。
- **ベンチマークは参考程度に:** SWE-bench Verified、Terminal-Bench、Aider Polyglotなどがよく引用されますが、自分のリポジトリでの成功率は自分で測るのが一番確実です。


---

<a id="ch3"></a>
## 第3章 ルールファイル設計（AGENTS.md / CLAUDE.md / .cursor/rules）

### 3.1 ルールファイルの役割と種類

ルールファイルは、エージェントがセッションを始めるたびに読む「プロジェクトの常識」です。人間の新メンバー向けオンボーディング資料に近いものですが、**簡潔さ・具体性・コマンドの正確さ** がより強く求められます。

| ファイル | 主な対応ツール | 配置 |
|---|---|---|
| `AGENTS.md` | Codex、Cursor、Copilot、Devin、Gemini CLI（設定次第）、Aider（設定次第）など多数 | リポジトリルート。サブディレクトリにも置ける（近いものが優先） |
| `CLAUDE.md` | Claude Code | ルート、サブディレクトリ、`~/.claude/CLAUDE.md`（個人用） |
| `.cursor/rules/*.mdc` | Cursor | フロントマターで適用条件（常時 / globs / エージェント判断 / 手動）を指定 |
| `.github/copilot-instructions.md` | GitHub Copilot | `.github/instructions/*.instructions.md` でパス別の指示も可 |
| `GEMINI.md` | Gemini CLI | ルートおよび階層 |
| `.clinerules` / `.roo/rules/` | Cline / Roo Code | ディレクトリ形式も可（要確認） |

**おすすめの構成:** 中身は `AGENTS.md` を正本とし、`CLAUDE.md` からは `@AGENTS.md` で取り込む（Claude Codeは `@path` でのインポートに対応）か、シンボリックリンクにします。こうすると二重管理を防げます。

```markdown
<!-- CLAUDE.md -->
@AGENTS.md

## Claude Code固有
- サブエージェント `test-runner` を使ってテスト失敗を調査すること
- 破壊的なgit操作（push --force, reset --hard）は必ず確認を取ること
```

### 3.2 良いAGENTS.mdの実例

```markdown
# AGENTS.md

## プロジェクト概要
B2B向け請求書管理SaaS。Next.js 15 (App Router) + TypeScript + Prisma + PostgreSQL。
モノレポ構成: `apps/web`（フロント）, `apps/api`（Hono）, `packages/db`（Prisma）, `packages/ui`。

## よく使うコマンド
- 依存インストール: `pnpm install`
- 開発サーバー: `pnpm dev`（web: 3000, api: 8787）
- 単体テスト: `pnpm test`（特定ファイル: `pnpm test -- path/to/file.test.ts`）
- 型チェック: `pnpm typecheck`
- Lint/整形: `pnpm lint --fix`
- E2E: `pnpm e2e`（事前に `pnpm db:reset` が必要）
- DBマイグレーション作成: `pnpm --filter db prisma migrate dev --name <name>`

## 完了の定義（Definition of Done）
変更を完了とする前に必ず以下を実行し、すべて成功させること:
1. `pnpm typecheck`
2. `pnpm lint`
3. 変更箇所に関係するテスト
新しいロジックには必ずテストを追加する。

## コーディング規約
- `any` 禁止。外部入力は zod でバリデーションする
- サーバー側のエラーは `AppError`（`packages/core/errors.ts`）で投げる
- 日付は `date-fns` を使い、`moment` は使わない
- UIは `packages/ui` の既存コンポーネントを優先する。新規作成前に検索すること
- コメントは「なぜ」を書く。「何を」は書かない

## やってはいけないこと
- `packages/db/prisma/migrations/` 内の既存ファイルを編集しない
- `.env*` ファイルを読まない・出力しない
- 依存パッケージの追加は、理由を説明してから行う
- 公開APIのレスポンス形式を変える場合は `docs/api-changes.md` に追記する

## アーキテクチャのメモ
- 認証は `apps/api/src/middleware/auth.ts`。テナント分離は全クエリで `tenantId` 必須
- 金額は整数（最小通貨単位）で扱う。浮動小数点禁止

## PR
- タイトルは Conventional Commits 形式（feat:, fix:, refactor: ...）
- 説明に「変更内容」「テスト方法」「リスク」を書く
```

### 3.3 書き方の原則

1. **短く保つ。** ルールファイルは毎回コンテキストに入ります。数百行を超えるようなら、詳しい内容はSkillsや `docs/` に移し、「詳細は `docs/testing.md` を参照」と書いて参照させます。
2. **コマンドは正確に、コピペで動く形で。** エージェントの失敗の多くは「テストの実行方法が分からない」「間違ったパッケージマネージャーを使う」ことから起きます。
3. **禁止事項は理由と一緒に書く。** 理由があると、エージェントは似た状況にも応用できます。
4. **ミスが起きたら追記する。** エージェントが同じ間違いを2回したら、ルールに1行足します。ルールファイルは「失敗の記録」として育てるものです。
5. **自明なことは書かない。** 「きれいなコードを書く」のような一般論は効果が薄く、トークンの無駄です。
6. **ネストを活用する。** `apps/web/AGENTS.md` にフロント固有のルールを置けば、その配下で作業するときだけ適用されます。

### 3.4 Cursorの `.cursor/rules` 実例

```markdown
---
description: Reactコンポーネントを作成・編集するときの規約
globs: apps/web/**/*.tsx
alwaysApply: false
---

- Server Componentをデフォルトにし、状態やイベントが必要な場合のみ `"use client"` を付ける
- スタイルは Tailwind。任意値（`w-[123px]`）は極力避け、デザイントークンを使う
- データ取得は `apps/web/lib/queries/` の関数経由で行う
- 新規コンポーネントには Storybook の story を追加する
- 参考実装: @apps/web/components/invoice/InvoiceTable.tsx
```

適用条件の使い分け:

- `alwaysApply: true` … 全体の方針（短く）
- `globs` 指定 … ファイル種別ごとの規約
- `description` のみ … エージェントが関連すると判断したときに読む（Skillsとの役割の重なりが大きく、Cursorは動的ルールからSkillsへの移行用に `/migrate-to-skills` を用意しています）
- 手動（`@rule名`）… たまにしか使わない手順

### 3.5 Copilotのパス別指示

```markdown
---
applyTo: "apps/api/**/*.ts"
---
# API実装ルール
- ルーティングは Hono の `app.route()` で分割する
- すべてのハンドラで `zValidator` を使って入力を検証する
- DBアクセスは `packages/db` のリポジトリ層経由のみ
```
（`.github/instructions/api.instructions.md`。フロントマターの仕様は要確認）

---

<a id="ch4"></a>
## 第4章 Agent Skills（SKILL.md）の設計

### 4.1 Skillsとは

Agent Skillsは、「特定のタスクのやり方」をまとめたフォルダです。中心となる `SKILL.md` に加え、スクリプト、テンプレート、参考資料を同梱できます。オープン標準の仕様（agentskills.io/specification）では、次のように定められています。

- 必須フィールド: `name`（最大64文字、小文字英数字とハイフンのみ、ディレクトリ名と一致）、`description`（最大1024文字。何をするか、いつ使うか）
- 任意フィールド: `license`、`compatibility`、`metadata`、`allowed-tools`（実験的）
- **段階的開示:** 起動時には `name` と `description`（約100トークン）だけを読み込み、使うと判断したときに本文（5,000トークン未満を推奨）を、必要に応じて `scripts/`・`references/`・`assets/` を読み込む

**ルールとの違い:** ルールは「常に守ること」、Skillsは「必要なときに引き出す手順書」です。毎回読ませる必要のない長い手順は、Skillsに移すとコンテキストを節約できます。

### 4.2 配置場所（執筆時点の公式ドキュメントより）

| ツール | プロジェクト | ユーザー |
|---|---|---|
| Claude Code | `.claude/skills/<name>/SKILL.md` | `~/.claude/skills/` |
| Codex | `.agents/skills/`（作業ディレクトリからルートまで探す） | ユーザー・管理者・システムの各場所 |
| Cursor | `.cursor/skills/` または `.agents/skills/`（`.claude/skills/`、`.codex/skills/` も互換で読み込む） | `~/.cursor/skills/` など |

呼び出し方: Claude Code・Cursorは `/skill-name`、Codexは `$skill-name` または `/skills`。自動で呼び出されないようにするには、Claude Code・Cursorでは `disable-model-invocation: true`、Codexでは `agents/openai.yaml` の `allow_implicit_invocation: false` を使います。

### 4.3 実例1: リリースノート作成スキル

```text
.agents/skills/release-notes/
├── SKILL.md
├── scripts/
│   └── collect_commits.sh
└── assets/
    └── template.md
```

```markdown
---
name: release-notes
description: 前回のタグから現在までのコミットとマージ済みPRを集計し、日本語のリリースノートを作成する。「リリースノート」「changelog」「変更履歴」を求められたときに使う。
---

# リリースノート作成

## 手順
1. `scripts/collect_commits.sh` を実行して、前回タグ以降のコミット一覧を取得する
2. Conventional Commits の type ごとに分類する（feat → 新機能, fix → 修正, perf → 改善）
3. `chore`, `ci`, `test` のみのコミットは除外する
4. 破壊的変更（`!` または `BREAKING CHANGE`）は先頭の「⚠ 互換性のない変更」節にまとめる
5. `assets/template.md` の形式で `CHANGELOG.md` の先頭に追記する
6. ユーザー向けの言葉で書く。内部の関数名は出さない

## 注意
- タグが存在しない場合は、ユーザーに起点を確認する
- PR番号は `(#123)` 形式でリンクする
```

```bash
#!/usr/bin/env bash
# scripts/collect_commits.sh
set -euo pipefail
last_tag=$(git describe --tags --abbrev=0 2>/dev/null || echo "")
if [ -z "$last_tag" ]; then
  echo "NO_TAG"; exit 0
fi
git log "${last_tag}..HEAD" --pretty=format:'%h|%s|%an' --no-merges
```

### 4.4 実例2: DBマイグレーション安全手順（手動起動のみ）

```markdown
---
name: db-migration
description: Prismaスキーマを変更しマイグレーションを作成する手順。テーブル・カラムの追加、変更、削除を伴う作業で使う。
disable-model-invocation: true
---

# 安全なDBマイグレーション

## 原則
- 本番データに影響する変更は「拡張 → 移行 → 縮小」（expand/contract）の3段階に分ける
- カラム削除やリネームを1回のマイグレーションで行わない

## 手順
1. `packages/db/prisma/schema.prisma` を編集
2. `pnpm --filter db prisma migrate dev --name <説明的な名前>` を実行
3. 生成されたSQLを読み、以下をチェック:
   - [ ] `DROP` を含んでいないか（含む場合は作業を止めてユーザーに確認）
   - [ ] 大きなテーブルへの `NOT NULL` 追加にデフォルト値があるか
   - [ ] インデックス作成は `CONCURRENTLY` が必要か
4. `pnpm --filter db test` を実行
5. PR説明に「ロールバック手順」を必ず書く

詳しいチェックリストは [references/checklist.md](references/checklist.md) を参照。
```

### 4.5 実例3: UI変更をブラウザで検証するスキル（Playwright MCPと連携）

```markdown
---
name: verify-ui
description: フロントエンドの変更後に、開発サーバーを起動しブラウザで実際の画面を確認してスクリーンショットを撮る。UI・CSS・コンポーネントを変更したら完了前に使う。
compatibility: Playwright MCP サーバーが設定されていること
---

# UI検証

1. 開発サーバーが動いていなければ `pnpm dev` をバックグラウンドで起動し、`http://localhost:3000` が応答するまで待つ
2. Playwright MCP で対象ページを開く
3. 以下を確認する:
   - コンソールエラーが出ていないこと
   - 変更した要素が表示され、クリックなどの操作ができること
   - 幅 375px（モバイル）と 1280px（デスクトップ）の両方で崩れがないこと
4. スクリーンショットを `artifacts/` に保存し、PR説明に添付する
5. 問題があれば修正して1からやり直す（最大3回。それでも直らなければ状況を報告）
```

### 4.6 Skills設計のコツ

- **`description` がすべてを決める。** エージェントは説明文だけを見て使うかどうかを決めます。「何をするか」と「どんなときに使うか（トリガーになる言葉）」の両方を書きます。
- **1スキル1目的。** 何でもできるスキルは呼び出されにくく、誤って呼び出されやすくもなります。
- **決定的な処理はスクリプトに。** 集計、変換、検証のような「毎回同じ結果になるべき処理」はシェルやPythonのスクリプトにして、LLMには判断だけを任せます。
- **本文は短く、詳細は `references/` に。** 段階的開示を活かします。
- **副作用のあるスキル（デプロイ、マイグレーション、外部への投稿）は手動起動にする。**
- **チームで共有する。** `.agents/skills/` をリポジトリに入れてレビュー対象にします。外部のスキルを取り込むときは、スクリプトの中身を必ず読みます（[第9章](#ch9)）。
- **Claude Codeのプラグインでの配布:** 複数のスキル・コマンド・hooks・MCP設定をまとめてプラグインにし、マーケットプレイス経由で配れます。Codexにもプラグイン形式があります（詳細は要確認）。


---

<a id="ch5"></a>
## 第5章 サブエージェント・hooks・カスタムコマンド

### 5.1 サブエージェント

サブエージェントは、メインのエージェントから仕事を切り出して、**独立したコンテキストで実行する** 専門エージェントです。利点は次の3つです。

1. **コンテキストの節約:** 大量のログ読みや検索を任せ、要約だけをメインに返させる
2. **専門化:** レビュー専用、テスト専用などで、プロンプトとツール権限を絞れる
3. **並列化:** 独立した調査を同時に進められる

#### Claude Codeのサブエージェント定義例（`.claude/agents/code-reviewer.md`）

```markdown
---
name: code-reviewer
description: コード変更をレビューする専門家。実装が終わった直後や、PRを作る前に積極的に使う。
tools: Read, Grep, Glob, Bash
model: inherit
---

あなたはシニアエンジニアのレビュアーです。`git diff main...HEAD` を確認し、以下の観点でレビューしてください。

## 観点（優先度順）
1. 正しさ: ロジックの誤り、境界値、null/undefined、競合状態
2. セキュリティ: 入力検証、認可漏れ（tenantIdのチェック）、秘密情報のハードコード
3. テスト: 変更に見合うテストがあるか、テストが実装の詳細に依存しすぎていないか
4. 保守性: 既存のパターンとの一貫性、重複

## 出力形式
- 🔴 必須修正 / 🟡 推奨 / 🟢 任意 に分類
- 各指摘に `ファイル:行` と修正案を付ける
- 問題がなければ「指摘なし」とだけ書く。褒め言葉は不要

コードは編集しないこと。
```

#### テスト調査用サブエージェント（`.claude/agents/test-runner.md`）

```markdown
---
name: test-runner
description: テストを実行し、失敗の原因を調査して要約する。テストが失敗したとき、または変更後の確認に使う。
tools: Bash, Read, Grep
model: haiku
---

1. 指定されたテスト（指定がなければ `pnpm test`）を実行する
2. 失敗したテストごとに、エラーメッセージ・関係するソース箇所・推定原因を3行以内でまとめる
3. 長いログ全文は返さない。メインエージェントが修正に必要な情報だけを返す
```

（`model` に指定できる値は要確認。安いモデルを割り当てるとコストを抑えられます。）

Cursorでもサブエージェント（`.cursor/agents/` など、仕様は要確認）や、Taskツールによる並列の子エージェントが使えます。Codexにも同様の仕組みが追加されつつあります（要確認）。

**設計の指針:**
- サブエージェントの出力は「要約」に絞る（メインのコンテキストを汚さない）
- レビュー役には編集権限を与えない
- 3〜5種類程度から始める（調査、テスト、レビュー、ドキュメント）。多すぎると使い分けが崩れます

### 5.2 hooks

hooksは、エージェントのライフサイクル（ツール実行の前後、セッション開始、プロンプト送信時、停止時など）に **決定的なスクリプトを差し込む** 仕組みです。「LLMにお願いする」のではなく「必ず実行される」点が重要で、ルールファイルの「お願い」を「強制」に変えられます。

#### Claude Codeのhooks例（`.claude/settings.json`）

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [
          { "type": "command", "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/format.sh" }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/guard-bash.py" }
        ]
      },
      {
        "matcher": "Read|Edit|Write",
        "hooks": [
          { "type": "command", "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/protect-secrets.sh" }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          { "type": "command", "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/verify.sh" }
        ]
      }
    ]
  }
}
```

フォーマッタを走らせるhook:

```bash
#!/usr/bin/env bash
# .claude/hooks/format.sh — 編集されたファイルを自動整形
input=$(cat)
file=$(echo "$input" | jq -r '.tool_input.file_path // empty')
[ -z "$file" ] && exit 0
case "$file" in
  *.ts|*.tsx|*.js|*.json|*.md) npx prettier --write "$file" >/dev/null 2>&1 ;;
  *.py) ruff format "$file" >/dev/null 2>&1 ;;
esac
exit 0
```

危険なコマンドをブロックするhook（終了コード2でブロックし、stderrの内容がエージェントに返る）:

```python
#!/usr/bin/env python3
# .claude/hooks/guard-bash.py
import json, re, sys

data = json.load(sys.stdin)
cmd = data.get("tool_input", {}).get("command", "")

DENY = [
    r"rm\s+-rf\s+/(?!tmp)",
    r"git\s+push\s+.*--force",
    r"git\s+reset\s+--hard",
    r"curl[^|]*\|\s*(ba)?sh",
    r"\bprintenv\b|\benv\b\s*$",
    r"cat\s+.*\.env",
]
for pat in DENY:
    if re.search(pat, cmd):
        print(f"ブロック: ポリシー違反のコマンドです ({pat})。別の方法を検討してください。", file=sys.stderr)
        sys.exit(2)
sys.exit(0)
```

秘密ファイルを保護するhook:

```bash
#!/usr/bin/env bash
# .claude/hooks/protect-secrets.sh
path=$(jq -r '.tool_input.file_path // empty')
if [[ "$path" =~ (\.env|\.pem|id_rsa|credentials\.json|secrets/) ]]; then
  echo "秘密情報を含む可能性のあるファイル ($path) へのアクセスは禁止されています" >&2
  exit 2
fi
exit 0
```

完了前に検証を強制するhook（Stop）:

```bash
#!/usr/bin/env bash
# .claude/hooks/verify.sh — 型チェックとlintが通るまで停止させない
input=$(cat)
# 無限ループ防止: すでにStop hookで継続中なら終了を許可
if [ "$(echo "$input" | jq -r '.stop_hook_active')" = "true" ]; then exit 0; fi
if ! out=$(pnpm -s typecheck 2>&1 && pnpm -s lint 2>&1); then
  echo "型チェックまたはlintが失敗しています。修正してから完了してください:" >&2
  echo "$out" | tail -n 40 >&2
  exit 2
fi
exit 0
```

（イベント名・入力JSONのフィールド名・終了コードの意味は、Claude Codeのhooksドキュメントで要確認。バージョンで追加・変更があります。）

#### Cursorのhooks例（`.cursor/hooks.json`）

Cursorもエージェントのhooks（シェル実行前、ファイル編集後、停止時など）に対応しています。形式の例です（キー名は要確認）。

```json
{
  "version": 1,
  "hooks": {
    "beforeShellExecution": [{ "command": ".cursor/hooks/guard-shell.sh" }],
    "afterFileEdit": [{ "command": ".cursor/hooks/format.sh" }],
    "stop": [{ "command": ".cursor/hooks/notify.sh" }]
  }
}
```

Cursorには `/create-hook` という組み込みスキルもあり、対話しながらhooksを作れます（公式ドキュメントに記載）。

#### hooksの活用アイデア

| タイミング | 用途 |
|---|---|
| 編集後 | フォーマッタ、lintの自動修正、変更ファイルの単体テスト |
| コマンド実行前 | 危険コマンドの拒否、本番環境へのアクセスのブロック |
| ファイル読み取り前 | 秘密ファイルの保護 |
| プロンプト送信時 | 秘密情報らしき文字列の検出、チケット情報の自動付与 |
| セッション開始時 | 現在のブランチ、未完了TODO、直近のCI結果をコンテキストに注入 |
| 停止時 | 検証の強制、デスクトップ通知・Slack通知 |

### 5.3 カスタムコマンド（スラッシュコマンド）

よく使うプロンプトを `/コマンド名` で呼び出せるようにする仕組みです。Claude Codeでは `.claude/commands/*.md`、Cursorでは `.cursor/commands/*.md` が使われてきましたが、現在はどちらも **Skillsへの統合が進んでいます**（`disable-model-invocation: true` のスキルは、実質的にスラッシュコマンドと同じ）。新しく作るならSkills形式が無難です。

#### 例: Issueを実装する `/fix-issue`（Claude Code形式）

```markdown
---
description: GitHub Issueを読み、テストファーストで修正してPRを作成する
argument-hint: <issue番号>
allowed-tools: Bash(gh:*), Bash(git:*), Bash(pnpm:*), Read, Edit, Write, Grep, Glob
---

Issue #$ARGUMENTS を解決してください。

1. `gh issue view $ARGUMENTS` で内容を確認し、不明点があれば作業前に質問する
2. `git checkout -b fix/issue-$ARGUMENTS`
3. 関連コードを調査し、修正方針を3〜5行で示す
4. 再現するテストを先に書き、失敗することを確認する
5. 修正し、テスト・型チェック・lintを通す
6. Conventional Commits 形式でコミットする
7. `gh pr create` で PR を作成し、本文に `Closes #$ARGUMENTS` を含める
```

#### 例: 計画だけを作る `/plan`

```markdown
---
name: plan
description: 実装に入る前に計画書を作成する
disable-model-invocation: true
---

コードは変更しないでください。以下の形式で `docs/plans/<日付>-<トピック>.md` を作成します。

1. 目的と完了条件（検証可能な形で）
2. 影響範囲（ファイル・モジュール）
3. 実装ステップ（各ステップは1コミット程度の大きさ）
4. テスト計画
5. リスクと未解決の質問

作成後、未解決の質問をユーザーに提示して終了してください。
```

#### 例: コミットを整える `/commit`

```markdown
---
name: commit
description: ステージ済みの変更から Conventional Commits 形式のメッセージを作ってコミットする
disable-model-invocation: true
---
1. `git diff --staged` を確認する。何もステージされていなければ報告して終了する
2. 変更が複数の論理単位を含む場合は、分割を提案する
3. `type(scope): 要約` の1行目（50文字以内）と、「なぜ」を説明する本文を書く
4. `git commit` を実行する（`--no-verify` は使わない）
```


---

<a id="ch6"></a>
## 第6章 有用なMCPサーバーと設定例

### 6.1 MCPとは

MCP（Model Context Protocol）は、Anthropicが2024年11月に公開した、AIアプリケーションと外部のツールやデータをつなぐオープンなプロトコルです。現在は主要なAIコーディングツールのほぼすべてが対応しています。接続方式には、ローカルでプロセスを起動する **stdio** と、リモートサーバーに接続する **Streamable HTTP**（OAuth認証に対応するものが多い）があります。

**原則:** MCPサーバーはツールの定義だけでコンテキストを消費し、さらに権限も持ちます。**本当に使うものだけを、必要な権限で** 有効にしてください。

### 6.2 おすすめMCPサーバー

| サーバー | 用途 | 提供元・形態（要確認を含む） |
|---|---|---|
| **GitHub MCP** | Issue/PRの閲覧・作成、コード検索、Actionsの結果確認 | GitHub公式（`github/github-mcp-server`）。リモート版（`https://api.githubcopilot.com/mcp/`）とローカル版あり |
| **Playwright MCP** | ブラウザ操作、E2E検証、スクリーンショット、アクセシビリティツリーの取得 | Microsoft（`@playwright/mcp`） |
| **Chrome DevTools MCP** | コンソール・ネットワーク・パフォーマンスの調査 | Google（`chrome-devtools-mcp`） |
| **Context7** | ライブラリの最新ドキュメントをバージョン指定で取得し、古いAPIの幻覚を減らす | Upstash（`@upstash/context7-mcp`、リモート版あり） |
| **Sentry MCP** | エラー・スタックトレースを取得して修正につなげる | Sentry公式（リモート `https://mcp.sentry.dev/mcp`） |
| **Linear MCP** | Issueの取得・更新、仕様の参照 | Linear公式（リモート `https://mcp.linear.app/mcp`。URLは要確認） |
| **Supabase MCP** | テーブル設計、SQL実行、マイグレーション、ログ | Supabase公式（`@supabase/mcp-server-supabase`）。読み取り専用モード推奨 |
| **PostgreSQL MCP** | スキーマ確認、読み取りクエリ | 参照実装はアーカイブ済みのため、コミュニティ版やDBベンダー製を選ぶ（要確認） |
| **Figma MCP** | デザインのフレーム・変数・コンポーネントを取得し実装に活かす | Figma公式（Dev Mode MCP / リモート版。要確認） |
| **Notion / Atlassian(Jira・Confluence) MCP** | 仕様書や社内ドキュメントの参照 | 各社公式（リモート） |
| **Cloudflare / Vercel / AWS MCP** | デプロイ状況、ログ、インフラ情報 | 各社公式（多数あり、要確認） |
| **Filesystem / Git / Fetch** | 参照実装 | `modelcontextprotocol/servers` |

### 6.3 設定例

#### Cursor（`.cursor/mcp.json`）

```json
{
  "mcpServers": {
    "github": {
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": { "Authorization": "Bearer ${env:GITHUB_PAT}" }
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    },
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    },
    "sentry": {
      "url": "https://mcp.sentry.dev/mcp"
    }
  }
}
```

#### Claude Code（CLIで追加、またはプロジェクトの `.mcp.json`）

```bash
# リモート（HTTP）サーバー
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
claude mcp add --transport http linear https://mcp.linear.app/mcp
# ローカル（stdio）サーバー
claude mcp add playwright -- npx -y @playwright/mcp@latest
# チーム共有したい場合は --scope project で .mcp.json に書き出す
claude mcp add --scope project context7 -- npx -y @upstash/context7-mcp
```

```json
{
  "mcpServers": {
    "supabase": {
      "command": "npx",
      "args": [
        "-y", "@supabase/mcp-server-supabase@latest",
        "--read-only",
        "--project-ref=${SUPABASE_PROJECT_REF}"
      ],
      "env": { "SUPABASE_ACCESS_TOKEN": "${SUPABASE_ACCESS_TOKEN}" }
    },
    "postgres-readonly": {
      "command": "npx",
      "args": ["-y", "<信頼できるPostgres MCPパッケージ>", "postgresql://readonly_user@localhost:5432/app_dev"]
    }
  }
}
```

#### Codex（`~/.codex/config.toml`）

```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]

[mcp_servers.playwright]
command = "npx"
args = ["-y", "@playwright/mcp@latest"]

[mcp_servers.figma]
url = "http://127.0.0.1:3845/mcp"   # Figmaデスクトップ版のローカルMCP（URLは要確認）
```

#### VS Code / Copilot（`.vscode/mcp.json`）

```json
{
  "servers": {
    "github": { "type": "http", "url": "https://api.githubcopilot.com/mcp/" },
    "playwright": { "type": "stdio", "command": "npx", "args": ["-y", "@playwright/mcp@latest"] }
  }
}
```

（各ツールの設定キーやパスはバージョンで変わることがあります。要確認。）

### 6.4 MCPを使ったワークフローの例

- **Sentry → 修正:** 「Sentryで直近24時間に最も多い未解決エラーを調べ、原因を特定し、再現テストを書いて修正して」
- **Figma → 実装:** 「選択中のFigmaフレームを、`packages/ui` の既存コンポーネントとデザイントークンを使って実装し、Playwrightでスクリーンショットを撮って見比べて」
- **Linear → PR:** 「Linearの ENG-123 を読み、仕様の不明点を挙げてから実装計画を作って」
- **Context7で幻覚を防ぐ:** 「Next.jsのドキュメントをContext7で確認してから、キャッシュの設定を書いて」

### 6.5 MCPの選び方と運用

- **公式・有名どころを優先する。** 出所の分からないMCPサーバーは、任意コードを実行するのと同じリスクがあります。
- **バージョンを固定する。** `@latest` は便利ですが、サプライチェーン攻撃に弱くなります。チームで使うものはバージョンを固定します。
- **読み取り専用から始める。** DB、本番クラウド、決済などは読み取り権限・開発環境だけにします。
- **ツール数を絞る。** 多くのサーバーは、有効にするツールを選べます（GitHub MCPのtoolsetsなど）。
- **トークンは環境変数で。** 設定ファイルに直接書かず、`${env:...}` などで参照します。

---

<a id="ch7"></a>
## 第7章 ワークフロー

### 7.1 基本サイクル: 探索 → 計画 → 実装 → 検証 → コミット

Anthropicの「Claude Code: Best practices for agentic coding」をはじめ、多くの実践者が勧めているのが、この流れを明確に分けることです。

1. **探索（Explore）:** 「まだコードは書かないで。関係するファイルを読んで、現状の仕組みを説明して」
2. **計画（Plan）:** Planモードなどで計画を作り、人間がレビューする。ここで方向性を直すのが最も安上がりです
3. **実装（Code）:** 計画に沿って、小さなステップで進める
4. **検証（Verify）:** テスト、型、lint、画面での確認。エージェント自身に確認させる
5. **コミット（Commit）:** 意味のある単位でコミットし、PRの説明を書かせる

### 7.2 仕様駆動開発（Spec-Driven Development）

「いい感じに作って」のような曖昧なプロンプト（いわゆるvibe coding）は、プロトタイプには有効ですが、本番開発では手戻りの元になります。仕様駆動開発では、**先に仕様・設計・タスク分解を文書にし、それを正本としてエージェントに実装させます**。GitHubのSpec Kit（`specify` CLI）、AWSのKiro（requirements → design → tasks）、Tsumikiなどの日本発ツール（要確認）がこの考え方を取り入れています。

#### ディレクトリ構成の例

```text
specs/
└── 2026-10-invoice-export/
    ├── requirements.md   # 何を・なぜ（ユーザーストーリー、受け入れ基準）
    ├── design.md         # どうやって（API、データモデル、画面、代替案）
    └── tasks.md          # 実装タスク（チェックボックス、1タスク≒1PR）
```

#### requirements.md の例（EARS記法を取り入れた受け入れ基準）

```markdown
# 請求書CSVエクスポート

## 背景
経理担当者が会計ソフトへ取り込むため、月次で請求書一覧をCSVで出力したい。

## ユーザーストーリー
経理担当者として、期間を指定して請求書をCSVで出力したい。それにより会計ソフトへの手入力をなくしたい。

## 受け入れ基準
- WHEN ユーザーが期間を指定して「CSV出力」を押したとき、THE SYSTEM SHALL 期間内の請求書をCSVでダウンロードさせる
- THE SYSTEM SHALL 文字コードを UTF-8（BOM付き）にする
- IF 対象件数が 10,000 件を超える場合、THEN THE SYSTEM SHALL 非同期で生成し、完了をメールで通知する
- THE SYSTEM SHALL 自テナントの請求書のみを出力する

## 対象外
- PDF一括出力
```

#### tasks.md の例

```markdown
- [ ] 1. `GET /invoices/export` のハンドラとzodスキーマ（同期・1万件以下）
  - テスト: 期間フィルタ、テナント分離、BOM
- [ ] 2. CSVシリアライザ（`packages/core/csv.ts`）とエスケープ処理
- [ ] 3. 非同期ジョブ（キュー登録、S3保存、メール通知）
- [ ] 4. フロントの出力ボタンと期間ピッカー
- [ ] 5. E2Eテスト
```

**運用のポイント:**
- 仕様書の作成そのものもAIに手伝わせる（「この要件の曖昧な点を10個挙げて」が有効）
- 人間は requirements と design のレビューに時間をかけ、tasks以降はエージェントに任せる比重を上げる
- 仕様と実装のずれを防ぐため、実装中に分かったことは仕様書に書き戻させる
- 仕様書が大きくなりすぎたら、それ自体が「タスクが大きすぎる」というサインです

### 7.3 TDD × AI

テスト駆動開発は、AI時代にむしろ価値が上がりました。テストは「完了の定義」を機械で検証できる形にしたものであり、エージェントが自分で正誤を判断するための最良の材料だからです。

#### 推奨プロンプトの型

```text
これからTDDで進めます。
1. 受け入れ基準に対応するテストを tests/invoice-export.test.ts に書いてください。
   まだ実装は書かないでください（モックで実装を作ることもしないでください）。
2. テストを実行し、期待どおりに失敗することを確認してください。
3. テストをコミットしてください。
4. テストを変更せずに、テストが通る実装を書いてください。
5. すべて通ったら、リファクタリングの余地を提案してください。
```

**注意点:**
- **テストの改ざんに注意する。** エージェントはテストを通すために、テストを弱めたり `skip` したりすることがあります。「テストを変更しないこと」と明示し、hooksやCIで `.skip` / `only` を検出するのが有効です。
- **実装とテストを同じコンテキストで書かせない工夫。** テストを書くサブエージェントと実装するエージェントを分けると、「実装に合わせたテスト」になりにくくなります。
- **プロパティベーステストやミューテーションテストも効果的。** fast-check、Hypothesis、Strykerなどで、テストそのものの質を確かめられます。
- **E2Eはエージェントの「目」になる。** Playwrightのテストやplaywright MCPを使って、UIの変更を自分で確認させます。


### 7.4 並列エージェントとgit worktree

エージェントが作業している間、人間は待つしかありません。そこで、**複数のエージェントを同時に走らせ、人間はレビューとタスクの割り振りに集中する** やり方が広がっています。ローカルではgit worktreeで作業ディレクトリを分けるのが基本です。

#### git worktreeの基本

```bash
# 機能ごとに別ディレクトリ・別ブランチを作る
git worktree add ../app-feat-export -b feat/invoice-export
git worktree add ../app-fix-1234   -b fix/issue-1234

# それぞれでエージェントを起動（例: tmuxの別ペイン）
cd ../app-feat-export && claude
cd ../app-fix-1234   && codex

# 一覧と後片付け
git worktree list
git worktree remove ../app-fix-1234
git branch -d fix/issue-1234
```

#### worktreeを作るヘルパースクリプト

```bash
#!/usr/bin/env bash
# scripts/wt.sh <branch-name> — worktree作成と初期セットアップ
set -euo pipefail
name="$1"
dir="../$(basename "$PWD")-${name//\//-}"
git worktree add "$dir" -b "$name"
cp .env.example "$dir/.env"          # 本物の秘密は入れない
cd "$dir"
pnpm install --frozen-lockfile
# ポートが衝突しないよう、ブランチ名からポート番号を決める
port=$(( 3000 + $(echo "$name" | cksum | cut -d' ' -f1) % 1000 ))
echo "PORT=$port" >> .env
echo "worktree: $dir (PORT=$port)"
```

**並列化で気をつけること:**
- **ぶつからないタスクを選ぶ。** 同じファイルを触るタスクを並列にすると、マージで苦労します
- **ポート、DB、キャッシュを分ける。** worktreeごとに開発サーバーのポートやDBスキーマを変えます（Docker Composeのプロジェクト名を分けるなど）
- **人間のレビュー能力が上限。** 同時に3〜5本程度が現実的、という声が多いです。それ以上はレビューが追いつかず品質が落ちます
- **同じタスクを複数のエージェントに解かせて良い方を選ぶ** やり方（best-of-N）も有効です。Cursorなどは複数モデルで同時に実行して比較する機能を持っています（要確認）

ツール側の対応: Cursorは並列エージェントにworktreeを内部で使い、Claude Codeは `--worktree` などのオプション（要確認）を持ち、Codexアプリやconductor、Crystalのような外部ツール（要確認）もworktreeでの並列実行を管理できます。

### 7.5 クラウドエージェント（非同期エージェント）

クラウドエージェントは、クラウド上の隔離されたVMやコンテナでリポジトリをクローンして作業し、結果をPRとして返します。ノートPCを閉じても作業が続き、ローカル環境を汚さず、何本でも並列に走らせられるのが利点です。

| サービス | 起動方法（例） |
|---|---|
| Cursor Cloud Agents | エディタ、Web、Slack、GitHub、Linear、API |
| OpenAI Codex（クラウド） | ChatGPT、Codexアプリ、GitHubのPRコメント（`@codex`） |
| GitHub Copilot coding agent | IssueをCopilotにアサイン、エージェントパネル |
| Devin | Slack、Linear、Jira、Web |
| Claude Code（Web/クラウド、GitHub Actions） | Web、モバイル、`@claude` メンション |

**成功させるための条件:**
1. **環境構築を自動化する。** 依存関係のインストール、DBの起動、シードの投入をスクリプト化します。Copilotなら `.github/workflows/copilot-setup-steps.yml`、Cursorなら `.cursor/environment.json`、Codexならセットアップスクリプトを使います（各名称は要確認）
2. **タスクを「チケット品質」で書く。** 背景、受け入れ基準、関連ファイル、検証方法を書きます
3. **ネットワークと秘密情報を最小限にする。** クラウドエージェントのネットワーク許可リストと、渡す秘密情報を絞ります
4. **PRは必ず人間がレビューする。** ブランチ保護で、エージェントのPRを直接mainにマージできないようにします

#### Copilot coding agent用セットアップの例

```yaml
# .github/workflows/copilot-setup-steps.yml
name: "Copilot Setup Steps"
on: workflow_dispatch
jobs:
  copilot-setup-steps:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    services:
      postgres:
        image: postgres:16
        env: { POSTGRES_PASSWORD: postgres }
        ports: ["5432:5432"]
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: pnpm }
      - run: pnpm install --frozen-lockfile
      - run: pnpm db:migrate
        env: { DATABASE_URL: postgresql://postgres:postgres@localhost:5432/postgres }
```

### 7.6 コードレビュー自動化

AIによるレビューは、「人間のレビューの前に機械的な指摘を片付ける」のに最も効果があります。

**選択肢:**
- **ツール内蔵:** Cursor Bugbot、GitHub Copilot code review、Codexの `@codex review`、Claude Codeの `/code-review` スキル（同梱スキルとしてドキュメントに記載）
- **専用サービス:** CodeRabbit、Graphite（Diamond）、Greptile、Qodo（旧CodiumAI）など（料金・機能は要確認）
- **自作:** GitHub ActionsでClaude Code / Codex CLIをヘッドレス実行する

#### Claude Code GitHub Actionsでの自動レビュー例

```yaml
# .github/workflows/ai-review.yml
name: AI Review
on:
  pull_request:
    types: [opened, synchronize]
permissions:
  contents: read
  pull-requests: write
jobs:
  review:
    if: github.event.pull_request.head.repo.full_name == github.repository  # フォークからのPRでは動かさない
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            このPRをレビューしてください。REVIEW.md の観点に従い、
            重大度の高い指摘（バグ、セキュリティ、データ破損）のみをインラインコメントで書いてください。
            スタイルの指摘はlinterに任せるので不要です。
          claude_args: "--max-turns 10 --allowedTools Read,Grep,Glob,Bash(git diff:*)"
```

（アクションの入力パラメータ名はバージョンで変わるため要確認。）

#### レビュー観点ファイル（`REVIEW.md`）の例

```markdown
# レビュー観点
## 必ず指摘する
- 認可チェック漏れ（tenantId条件のないクエリ）
- N+1クエリ、トランザクション境界の誤り
- エラーの握りつぶし（空のcatch）
- 金額の浮動小数点演算
- テストのない新規分岐

## 指摘しない
- 命名の好み、import順（linterで担保）
- 既存コードの問題で、今回の差分に含まれないもの
```

**運用のコツ:**
- **ノイズを減らす。** 指摘が多すぎると誰も読まなくなります。「重大度の高いものだけ」「確信度が低いものは書かない」と指示します
- **AIレビューは承認条件にしない。** あくまで補助です。最終的な承認は人間が行います
- **誤検知をフィードバックする。** 誤った指摘のパターンを `REVIEW.md` の「指摘しない」に追加していきます

### 7.7 CI連携

CIは、エージェントにとっての「最後の安全網」であると同時に、「エージェントを起動する場所」にもなります。

**パターン:**
1. **CIの失敗をエージェントに直させる:** CIが落ちたら、ログを付けてエージェントに修正PRを作らせる
2. **定期実行の保守タスク:** 依存関係の更新に伴う破壊的変更への対応、不安定なテスト（flaky test）の調査、ドキュメントの更新
3. **Issueのトリアージ:** 新しいIssueにラベルを付け、重複を検出し、再現手順の不足を指摘する

#### Codex CLIをCIで非対話実行する例

```yaml
# .github/workflows/fix-ci.yml（概念例。コマンドやオプションは要確認）
name: Auto-fix lint
on:
  workflow_dispatch:
jobs:
  fix:
    runs-on: ubuntu-latest
    permissions: { contents: write, pull-requests: write }
    steps:
      - uses: actions/checkout@v4
      - run: npm i -g @openai/codex
      - run: |
          codex exec --sandbox workspace-write \
            "pnpm lint を実行し、自動修正できないエラーを修正して。テストは変更しないこと。"
        env: { OPENAI_API_KEY: "${{ secrets.OPENAI_API_KEY }}" }
      - run: |
          git switch -c bot/lint-fix-${{ github.run_id }}
          git commit -am "fix: lint errors (automated)" && git push -u origin HEAD
          gh pr create --fill --label automated
        env: { GH_TOKEN: "${{ github.token }}" }
```

#### Claude Codeのヘッドレス実行

```bash
# ログを要約させる（パイプ入力）
cat build.log | claude -p "このビルドログから失敗の根本原因を3行で要約して" --output-format json

# 権限を絞った自動実行
claude -p "flakyなテストを特定して報告して" \
  --allowedTools "Bash(pnpm test:*),Read,Grep" --max-turns 20
```

**CI連携の安全原則:**
- フォークからのPRや外部ユーザーのコメントで、秘密情報を持つワークフローを起動しない（`pull_request_target` の誤用に注意）
- `GITHUB_TOKEN` の権限を最小にする
- エージェントが作ったPRも、通常のCIとレビューを必ず通す
- 実行回数・ターン数・時間に上限を設ける


---

<a id="ch8"></a>
## 第8章 コスト管理

### 8.1 料金体系の傾向（2026年秋）

二次情報（2026年8月時点の比較記事など）によると、主要ツールの個人向け料金は次のとおりです。**すべて要確認**です。

| ツール | エントリー | 上位 | チーム | 超過分の課金 |
|---|---|---|---|---|
| Cursor | Pro $20/月 | Ultra $200/月 | 席あたり課金 | プラン額相当の使用枠、超過はAPI料金 |
| Claude Code | Pro $20/月 | Max $100〜$200/月 | Team / Enterprise | セッションと週ごとの使用上限。APIキー利用なら従量 |
| OpenAI Codex | ChatGPT Plus $20/月 | Pro | Business / Enterprise | ChatGPTプランの枠＋クレジット |
| GitHub Copilot | Pro $10/月 | Pro+ $39/月 など | Business $19/席、Enterprise $39/席 | プレミアムリクエストやクレジット（体系は要確認） |
| Devin（Desktop/CLI/Cloud） | Pro $20/月 | Max $200/月 | Teams $80/月〜＋席料金 | 日次・週次のトークン枠、超過はAPI料金 |
| Gemini CLI | 無料枠あり | Google AIの各プラン | Code Assist Standard/Enterprise | API従量 |
| Cline / Roo / Aider | 無料（OSS） | ― | ― | 使ったAPIの従量のみ |

共通する傾向は、**「定額で使い放題」から「定額＋使用量」へ** の移行です。エージェントの1タスクは、補完の数百〜数千倍のトークンを使うことがあるためです。

### 8.2 コストを左右する要因

1. **モデルの単価:** 最上位モデルと軽量モデルでは、単価が数倍〜十数倍違います
2. **コンテキストの長さ:** 大きなファイルや長いログ、多数のMCPツール定義を毎ターン送ると、入力トークンが膨らみます
3. **ターン数:** テストの失敗と修正を繰り返すループ。完了条件が曖昧だと長引きます
4. **並列数:** 並列やbest-of-Nは、そのまま倍の費用になります
5. **プロンプトキャッシュ:** 同じプレフィックス（ルールファイルなど）はキャッシュが効きやすく、単価が下がります

### 8.3 具体的な節約術

- **モデルを使い分ける:** 計画とレビューは上位モデル、実装や定型作業は中位モデル、ログの要約やテスト実行は軽量モデル（サブエージェントにモデルを指定）
- **コンテキストを掃除する:** タスクが変わったら新しいセッションを始める（Claude Codeなら `/clear`、長くなったら `/compact`）。不要なMCPサーバーを無効にする
- **ルールファイルを短く保つ:** 毎ターン送られるものほど、効果の割に高くつきます
- **出力を絞る:** `pnpm test 2>&1 | tail -50` のように、ログ全文を読ませない。テストランナーは失敗したものだけを出す設定にする
- **タスクを明確にする:** 完了条件の曖昧さが、ターン数を増やす最大の原因です
- **使用量を見える化する:** Cursorの使用量ダッシュボード、Claude Codeの `/cost` や `ccusage`（コミュニティ製CLI）、OpenAIやAnthropicのAPI使用量画面、組織の管理画面
- **予算の上限とアラートを設ける:** APIキーごとの上限、チームの月次上限、超過時の通知

### 8.4 ROIの考え方

費用が月数百ドルになっても、エンジニアの人件費と比べれば小さいことが多い一方、**「使われずに払っている席」** と **「レビューされずに積み上がるコード」** は純粋な損失です。次のような指標で効果を測ります。

- リードタイム（Issue作成からマージまで）、PRあたりのレビュー時間
- 変更失敗率、マージ後のバグ件数（DORAメトリクス）
- エージェントのPRの採用率（マージされた割合）、手直しの量
- 開発者アンケート（満足度、認知負荷）

注意点として、METRが2025年に発表したランダム化比較試験では、慣れたOSS開発者がAIツールを使ったところ、**本人は速くなったと感じていたのに、実際には作業時間が約19%長くなった** という結果が報告されています（当時のツールとモデルによる結果であり、現在にそのまま当てはまるとは限りません）。主観ではなく計測で判断することが大切です。

---

<a id="ch9"></a>
## 第9章 セキュリティ（プロンプトインジェクション・秘密情報）

### 9.1 主なリスク

| リスク | 内容 | 例 |
|---|---|---|
| **プロンプトインジェクション** | エージェントが読むデータに埋め込まれた指示に従ってしまう | Issue本文、PRコメント、READMEの隠しテキスト、Webページ、MCPの応答、依存パッケージ内の文書 |
| **データ流出** | 秘密情報やコードが外部に送られる | 画像URLやリンクに情報を埋め込む、`curl` での送信、MCP経由の書き込み |
| **秘密情報の漏洩** | `.env` やトークンがコンテキストやログ、コミットに入る | エージェントが `.env` を読んでログに出す、テストにトークンをハードコード |
| **破壊的操作** | 誤ってデータやリポジトリを壊す | `rm -rf`、`git push --force`、本番DBへの `DROP` |
| **サプライチェーン** | 存在しないパッケージ名を提案され、攻撃者がそれを登録する（slopsquatting）、悪意あるMCP・スキル・拡張機能 | `npm install` での不審なパッケージ、ツール説明文を細工したMCP（tool poisoning） |
| **ルールファイルの汚染** | `AGENTS.md` やルールに不可視文字などで悪意ある指示を仕込む | 「Rules File Backdoor」として報告された手口（2025年） |

### 9.2 「致命的な三要素（lethal trifecta）」

Simon Willisonは、エージェントが次の3つを同時に持つと、データ流出がほぼ防げなくなると指摘しています。

1. **プライベートなデータへのアクセス**（ソースコード、秘密情報、社内文書）
2. **信頼できないコンテンツへの接触**（外部Issue、Webページ、メール）
3. **外部と通信する手段**（ネットワーク、PRやコメントの作成、Webリクエスト）

**対策の基本は、このうち少なくとも1つを断つことです。** たとえば、外部のIssueを処理するエージェントには秘密情報を渡さない、あるいはネットワークを許可リスト方式で制限する、といった具合です。

### 9.3 具体的な対策

#### 権限とサンドボックス

- **承認モードを適切に:** 最初は「コマンド実行ごとに承認」から始め、信頼できるコマンドだけを許可リストに入れる
- **YOLOモード（全自動）はサンドボックス内だけで:** Dockerやdevcontainer、クラウドVMなど、壊れても良く、秘密情報のない環境に限る
- **ネットワークを制限:** Codexのサンドボックス設定、クラウドエージェントの通信許可リストなど

Claude Codeの権限設定の例（`.claude/settings.json`）:

```json
{
  "permissions": {
    "allow": [
      "Bash(pnpm test:*)",
      "Bash(pnpm lint:*)",
      "Bash(pnpm typecheck)",
      "Bash(git status)",
      "Bash(git diff:*)"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Bash(curl:*)",
      "Bash(git push:*)"
    ]
  }
}
```

Codexの設定の例（`~/.codex/config.toml`、キー名は要確認）:

```toml
approval_policy = "on-request"
sandbox_mode = "workspace-write"

[sandbox_workspace_write]
network_access = false
```

#### 秘密情報の扱い

- `.env` は `.gitignore` に入れたうえで、エージェントの読み取り対象からも外す（Cursorの `.cursorignore`、Claude Codeの `deny` 設定、hooks）
- 開発環境では、権限の弱い開発用キーやダミー値を使う。本番のキーをローカルに置かない
- gitleaks、trufflehog、GitHubのsecret scanning（push protection）を使ってコミット前に検出する
- 秘密情報は1Password CLIやDoppler、クラウドのSecret Managerから、必要なときだけ注入する

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.24.0   # バージョンは要確認
    hooks:
      - id: gitleaks
```

#### プロンプトインジェクション対策

- **信頼できない入力を扱うエージェントと、強い権限を持つエージェントを分ける**
- **外部への書き込み（PR作成、コメント、Slack投稿）は人間の承認を挟む**
- **MCPサーバーは信頼できるものに限り、バージョンを固定する。** ツールの説明文が更新されたら差分を確認する
- **外部から取り込むSkills・ルール・プラグインは、中身（特に `scripts/`）を読んでから入れる**
- **不可視のUnicode文字を検出する:** ルールファイルやプロンプトに入り込んだゼロ幅文字などをCIでチェックする

```bash
# ルールファイルに不可視文字が含まれていないか確認する（CIに組み込める）
if grep -rnP '[\x{200B}-\x{200F}\x{202A}-\x{202E}\x{2060}-\x{2064}\x{FEFF}]' \
     AGENTS.md CLAUDE.md .cursor/rules .agents .claude 2>/dev/null; then
  echo "不可視文字が見つかりました"; exit 1
fi
```

#### 依存関係

- エージェントが追加したパッケージは、名前・提供元・ダウンロード数・作成日を確認する（「ルールファイルに依存追加は理由の説明を必須」と書く）
- lockfileを必須にし、`npm ci` / `pnpm install --frozen-lockfile` を使う
- Dependabot、Renovate、Socket、Snykなどで監視する

### 9.4 組織としてのガバナンス

- **データの取り扱い:** 各ツールのビジネスプランで「学習に使わない（ゼロデータ保持など）」の設定を確認し、契約書面で担保する（プライバシーモードなど）
- **許可するツールとモデルの一覧を作る**
- **監査ログ:** 誰がどのエージェントで何をしたかを追えるようにする
- **ガイドラインを作る:** 機密レベルごとに、使ってよいツールを決める
- **OWASP Top 10 for LLM Applications** などの公開フレームワークを参考にする

---

<a id="ch10"></a>
## 第10章 品質担保

### 10.1 「検証できるものしか任せない」

AIが書いたコードの品質は、**検証の仕組みの強さ** で決まります。

| 層 | 手段 | エージェントへの効果 |
|---|---|---|
| 静的解析 | 型（TypeScript strict、mypy）、lint、フォーマッタ | 即座に機械的な誤りを検出し、自分で直せる |
| 単体テスト | Vitest、Jest、pytest など | ロジックの正しさを自分で確認できる |
| 統合・E2E | Playwright、Testcontainers | 実際の動作を確認できる |
| 視覚的な確認 | スクリーンショット、ビジュアルリグレッション | UIの崩れを検出できる |
| セキュリティ | Semgrep、CodeQL、依存関係スキャン | 危険なパターンを検出できる |
| 人間のレビュー | PRレビュー | 設計の妥当性、要件との一致、保守性 |

### 10.2 AIが書いたコードで起きがちな問題

- **存在しないAPIやオプションの使用（幻覚）:** Context7などで最新ドキュメントを参照させ、型チェックで検出する
- **既存の仕組みの再発明:** 「新しく作る前に既存のユーティリティを検索すること」をルールに書く
- **過剰な防御的コードや冗長なコメント:** レビュー観点に入れる
- **テストの弱体化:** テストファイルの変更をPRでハイライトし、`skip` を検出する
- **エラーの握りつぶし:** 空の `catch` や、エラーをログに出して続行するだけのコードをlintで禁止する
- **大きすぎる差分:** 1つのPRを400行程度以内などに目安を設け、超えたら分割させる
- **セキュリティの穴:** 認可の抜け、SQLインジェクション、XSS。テナント分離などは専用のテストで守る

### 10.3 人間のレビューのポイント

AIの差分をレビューするとき、次の点に重点を置くと効率的です。

1. **要件を満たしているか**（仕様書の受け入れ基準と照合）
2. **テストは妥当か**（テストが仕様を表しているか、実装をなぞっているだけではないか）
3. **設計は既存の流儀に合っているか**
4. **境界や異常系の扱い**
5. **自分で説明できるか** ―― 説明できないコードはマージしない。これが最も重要なルールです

### 10.4 「理解の負債」を防ぐ

AIに任せるほど、チームがコードを理解していない状態（理解の負債）が溜まりやすくなります。

- 重要な変更には、エージェントに「設計の判断理由」をPRの説明やADR（Architecture Decision Record）に書かせる
- 定期的に、エージェントにコードベースの解説ドキュメントを更新させ、人間が読む
- 新人には、AIを使う前に、AIの出力を「レビューする側」から経験させる
- 核心となるドメインロジックは、人間が主導して書く（あるいは細かくレビューする）

---

<a id="ch11"></a>
## 第11章 情報源・発信者ガイド（日英）

> 個人の発信者の活動状況やアカウントは変わることがあります。ここに挙げる人物やサイトは、執筆者が知る範囲でAI開発の分野で広く参照されているものです。URLやハンドルのうち確信が持てないものは **要確認** としています。

### 11.1 公式ブログ・ドキュメント（一次情報）

| 情報源 | URL | 見どころ |
|---|---|---|
| Anthropic Engineering | https://www.anthropic.com/engineering | エージェント設計、Claude Codeのベストプラクティス、コンテキストエンジニアリング |
| Claude Code ドキュメント | https://code.claude.com/docs | Skills、hooks、サブエージェント、MCP、設定 |
| OpenAI Developers（Codex） | https://developers.openai.com/codex | Codex CLI・クラウド、Skills、AGENTS.md |
| Cursor ブログ / Changelog / Docs | https://cursor.com/blog 、 https://cursor.com/changelog 、 https://cursor.com/docs | 新機能、Rules、Skills、hooks、Cloud Agents |
| GitHub Blog / Changelog | https://github.blog 、 https://github.blog/changelog | Copilot coding agent、MCP、Spec Kit |
| Google Developers Blog / Gemini CLI | https://github.com/google-gemini/gemini-cli | Gemini CLIのリリースと拡張 |
| Model Context Protocol | https://modelcontextprotocol.io | 仕様、SDK、サーバー一覧 |
| Agent Skills 仕様 | https://agentskills.io | `SKILL.md` の標準仕様 |
| AGENTS.md | https://agents.md | AGENTS.md の説明と対応ツール |
| Cognition（Devin） | https://cognition.ai/blog 、 https://devin.ai | Devin / Devin Desktop の発表 |
| Aider | https://aider.chat | 使い方、LLMリーダーボード |

### 11.2 英語圏の発信者・メディア

- **Simon Willison** ― https://simonwillison.net 。LLMとツールの実験を日々記録しているブログ。プロンプトインジェクションや「lethal trifecta」などセキュリティの論考でも知られます。CLIツール `llm` の作者。
- **Latent Space（swyx、Alessio Fanelli）** ― https://www.latent.space 。AIエンジニア向けのポッドキャストとニュースレター。AI Engineerカンファレンス（AI Engineer World's Fair など）とも関わりが深い。
- **Andrej Karpathy** ― X: @karpathy 、YouTube。「vibe coding」という言葉を広めた人物で、LLMの仕組みの解説動画で有名。
- **Addy Osmani** ― https://addyosmani.com 。Google所属。AI支援開発の実践について多くの記事や書籍を書いています。
- **Harper Reed** ― https://harper.blog 。「My LLM codegen workflow atm」など、仕様を先に作るワークフローの記事が広く読まれました。
- **Armin Ronacher** ― https://lucumr.pocoo.org 。Flaskの作者。エージェント型コーディングの実践記を書いています。
- **Geoffrey Huntley** ― https://ghuntley.com 。エージェントをループで回す手法（「Ralph」など）の論考で知られます（要確認）。
- **Gergely Orosz（The Pragmatic Engineer）** ― https://newsletter.pragmaticengineer.com 。業界の動向や現場への取材記事。
- **Kent Beck** ― https://tidyfirst.substack.com 。TDDの提唱者が、AIとの「augmented coding」を論じています。
- **Martin Fowler のサイト（Birgitta Böckeler ほか）** ― https://martinfowler.com 。生成AIと開発に関する連載記事。
- **Hamel Husain** ― https://hamel.dev 。LLMアプリの評価（evals）の実践。
- **Thorsten Ball** ― ニュースレター「Register Spill」。Sourcegraph/Ampでエージェント開発に携わる（所属は要確認）。
- **METR** ― https://metr.org 。AIの能力や開発者生産性への影響に関する研究（[第8章](#ch8)で紹介した調査）。
- **YouTube:** Anthropic・OpenAI・Cursor・GitHubの公式チャンネル、AI Engineerの講演アーカイブ、IndyDevDan（Claude Codeなどエージェント活用の解説。要確認）など。
- **コミュニティ:** Hacker News（https://news.ycombinator.com ）、Reddit の r/ClaudeAI・r/cursor・r/LocalLLaMA、各ツールのDiscordや公式フォーラム。

### 11.3 日本語の情報源

- **Zenn** ― https://zenn.dev 。トピック「Claude Code」「Cursor」「MCP」「AIエージェント」「Codex」などで実践記事が非常に多く集まります。企業のテックブログ（Publication）も多い。
- **Qiita** ― https://qiita.com 。入門記事や設定例が豊富。Advent Calendarの時期は特にまとまった情報が集まります。
- **DevelopersIO（クラスメソッド）** ― https://dev.classmethod.jp 。新機能を試した記事が発表直後に出ることが多い。
- **各社テックブログ** ― 自社でのAIエージェント導入事例（例: 大手Web企業やSaaS企業のブログ）。社名検索で「Claude Code 導入」「Cursor 全社導入」などを探すと見つかります。
- **mizchi（みずち）** ― Zenn・Xで、AIコーディングエージェントの使い方や設計についての論考を多く発信（アカウントは要確認）。
- **Findy Tools / Findy の調査記事** ― 開発ツールの比較やアンケート（要確認）。
- **connpass** ― https://connpass.com 。「AI駆動開発」「Claude Code」などのテーマで勉強会が多数開かれています。
- **X（旧Twitter）** ― 日本語圏でもツールの開発元の社員や実践者が多く発信しています。特定の個人は活動状況が変わりやすいため、Zennの人気記事の著者をたどってフォローするのが確実です。
- **書籍** ― AIエージェントを使った開発や、MCP、プロンプトエンジニアリングの書籍が多数出版されています。刊行が速いので、発行日が新しいものを選びましょう（具体的な書名は要確認）。

### 11.4 情報収集のコツ

- **一次情報（公式Changelog）を週1回確認する。** 機能の名称や仕様はすぐ変わります。RSSに登録しておくと楽です
- **「やってみた」記事は日付を見る。** 半年前の記事は、設定方法が古くなっている可能性があります
- **ベンチマークの数値より、自分のリポジトリでの成功率を重視する**
- **チーム内で「今週の発見」を共有する場を作る。** Slackのチャンネルや週次の短い会など

---

<a id="ch12"></a>
## 第12章 すぐ試せるチェックリスト

### 12.1 今日やること（個人・30分〜1時間）

- [ ] 主に使うツールを1つ決めてインストールする（Cursor / Claude Code / Codex / Copilot）
- [ ] リポジトリ直下に `AGENTS.md` を作る（コマンド、完了の定義、禁止事項）。エージェントに「このリポジトリを調べて `AGENTS.md` の下書きを作って」と頼んでも良い
- [ ] `CLAUDE.md` を使うなら `@AGENTS.md` を取り込む形にする
- [ ] `.env` などの秘密ファイルを、エージェントの読み取り対象から外す
- [ ] 小さなバグ修正を1件、「探索 → 計画 → テスト先行 → 実装 → 検証」の流れで任せてみる
- [ ] 結果の差分を自分で読み、説明できることを確かめてからコミットする

### 12.2 今週やること

- [ ] Context7とPlaywright MCP（フロントエンドなら）を導入する
- [ ] よく使う手順を1つ `SKILL.md` にする（例: リリースノート、PR作成、UI検証）
- [ ] 編集後にフォーマッタを走らせるhookと、危険なコマンドを止めるhookを入れる
- [ ] git worktreeを使って、2つのタスクを並列で走らせてみる
- [ ] 使用量ダッシュボードで、1週間の費用とタスク数を確認する
- [ ] エージェントが2回同じミスをしたら、`AGENTS.md` に1行足す

### 12.3 チームでやること

- [ ] `AGENTS.md`・Skills・hooks・MCP設定をリポジトリに入れてレビュー対象にする
- [ ] PRのAIレビュー（Bugbot、Copilot review、CodeRabbit、Claude Code Actionなど）を1つ試す
- [ ] クラウドエージェントのセットアップスクリプト（環境構築の自動化）を用意する
- [ ] ブランチ保護と必須チェックで、AIのPRもCIとレビューを必ず通るようにする
- [ ] 秘密情報のスキャン（gitleaks、push protection）を有効にする
- [ ] 使ってよいツール・モデル・データの範囲をガイドラインにまとめる
- [ ] 計測する指標（リードタイム、PR採用率、変更失敗率、費用）を決める

### 12.4 プロンプトの型（コピーして使える）

```text
【探索】
まだコードは変更しないでください。<機能> に関係するファイルを調べ、
現在の処理の流れ、関係するモジュール、テストの場所を説明してください。

【計画】
<要件> を実装する計画を立ててください。変更するファイル、ステップ、
テスト方針、リスク、不明点を挙げてください。不明点があれば実装前に質問してください。

【実装（TDD）】
計画のステップ1だけを実装してください。先にテストを書き、失敗を確認してから
実装してください。既存のテストは変更しないでください。
完了したら `pnpm typecheck && pnpm lint && pnpm test` を実行して結果を報告してください。

【レビュー依頼】
`git diff main...HEAD` を、バグ・セキュリティ・テスト不足の観点でレビューしてください。
重要度の高い順に、ファイルと行を示して指摘してください。

【振り返り】
このセッションで、あなたが間違えた点や、事前に知っていれば早く終わった情報は何ですか？
AGENTS.md に追記すべき内容を3行以内で提案してください。
```

---

<a id="ch13"></a>
## 第13章 段階的導入ロードマップ

期間ではなく、**各段階の到達条件** で進み具合を判断します。

### ステージ0: 準備
- **やること:** ツールの選定、データ取り扱いの確認（学習に使われない設定、契約）、試行メンバーの選出
- **到達条件:** 使ってよいツールとデータの範囲が文書になっている

### ステージ1: 個人での補完・チャット利用
- **やること:** 補完、コードの解説、小さな関数の生成、テストの下書き
- **到達条件:** 試行メンバーが日常的に使い、役立つ場面と役立たない場面を説明できる
- **落とし穴:** 出力をそのまま貼り付ける。説明できないコードをマージしない、を徹底する

### ステージ2: コンテキストの整備
- **やること:** `AGENTS.md`（＋各ツール向けの取り込み）、テストとlintのコマンドを整える、秘密ファイルの除外
- **到達条件:** 新しいセッションのエージェントが、説明なしでビルドとテストを正しく実行できる

### ステージ3: エージェントに小タスクを任せる
- **やること:** 「探索 → 計画 → TDD → 検証」の型で、バグ修正や小機能を任せる。PRは人間がレビュー
- **到達条件:** エージェントのPRの過半が、軽微な手直しでマージされる
- **指標:** PRの採用率、手直しの量、1タスクあたりの費用

### ステージ4: 標準化（Skills・MCP・hooks）
- **やること:** よく使う手順をSkillsに、外部連携をMCPに、必須の検証をhooksにする。リポジトリで共有しレビューする
- **到達条件:** チームの誰が使っても、同じ品質の手順でエージェントが動く

### ステージ5: 並列化・クラウド・CI連携
- **やること:** git worktreeでの並列実行、クラウドエージェントへの委任、AIによるPRレビュー、CIの失敗の自動修正、定期的な保守タスク
- **到達条件:** 人間の主な仕事が「タスクの定義」と「レビュー」になり、リードタイムが計測上で短くなっている
- **落とし穴:** レビューが追いつかないほど並列にすること。同時並列数はレビュー能力に合わせる

### ステージ6: 組織的な運用とガバナンス
- **やること:** 費用の予算管理、監査ログ、セキュリティポリシー（権限・サンドボックス・ネットワーク制限）、社内での知見共有、ツールの定期的な見直し
- **到達条件:** 費用対効果を数字で説明でき、インシデントへの対応手順がある

### 推奨する体制
- **推進役（チャンピオン）** を各チームに1人置き、ルール・Skillsの整備と知見の共有を担当してもらう
- **「AI活用の振り返り」** を定例にする（何がうまくいき、何に失敗したか）
- **ツールは半年ごとに見直す。** この分野は変化が速く、半年前の最適解が今も最適とは限りません。ただし、`AGENTS.md`・Skills・テストといった資産はツールを替えても残ります

---

<a id="appendix"></a>
## 付録 参考URL一覧

本文作成時に確認した主な情報源です（2026年10月1日時点。リンク先の内容は更新されている可能性があります）。

- Agent Skills 仕様: https://agentskills.io/specification
- Claude Code Skills ドキュメント: https://code.claude.com/docs/en/skills
- OpenAI Codex Agent Skills: https://developers.openai.com/codex/skills
- Cursor Agent Skills: https://cursor.com/docs/context/skills
- Windsurf から Devin Desktop への改名と料金（二次情報）:
  - https://agentcode.ai/blog/windsurf-vs-devin-pricing
  - https://chatforest.com/builders-log/windsurf-devin-desktop-rebrand-devin-local-acp-builder-guide/
  - https://continuumcode.ai/guides/devin-ai/
- 各ツールの料金比較（二次情報、要確認）: https://coworker.ai/blog/windsurf-pricing
- Model Context Protocol: https://modelcontextprotocol.io
- AGENTS.md: https://agents.md

> **最後に:** AIを活用した開発で効率を上げる本質は、「AIに何をさせるか」よりも **「AIが正しく動ける環境をどう作るか」** にあります。明確な仕様、速く信頼できるテスト、簡潔なルール、再利用できるSkills、最小権限の接続、そして説明できないコードはマージしない文化。これらはツールが替わっても価値を失わない、チームの資産です。
