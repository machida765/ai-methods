# AIを活用したYouTube動画制作の効率的なやり方（2026年10月版）

> 作成日: 2026年10月1日
> 対象読者: AIを使ってYouTube動画（長尺・ショート）の制作時間を短縮したい個人クリエイター、企業の動画担当者、エンジニア寄りの発信者
> 注意: AIツールの機能・料金・提供状況は数か月単位で変わります。本レポートで「要確認」としたものは、執筆時点で一次情報を確認できなかった項目です。導入前に必ず公式サイトで最新情報を確認してください。

---

## 目次

- [3分で分かる要約](#summary)
- [第1章 なぜ今「AI × YouTube制作」なのか：2026年の前提整理](#ch1)
- [第2章 工程別ツールマップ](#ch2)
  - [2-1 企画・リサーチ](#ch2-1)
  - [2-2 台本](#ch2-2)
  - [2-3 音声（ナレーション・キャラクターボイス）](#ch2-3)
  - [2-4 動画生成](#ch2-4)
  - [2-5 画像生成](#ch2-5)
  - [2-6 編集](#ch2-6)
  - [2-7 字幕](#ch2-7)
  - [2-8 サムネイル](#ch2-8)
  - [2-9 ショート切り抜き](#ch2-9)
  - [2-10 分析](#ch2-10)
- [第3章 ジャンル別の構築パイプライン](#ch3)
  - [3-1 ゆっくり／ずんだもん解説](#ch3-1)
  - [3-2 顔出しなし（ナレーション＋素材）チャンネル](#ch3-2)
  - [3-3 ショート量産（ただし“量産型”にしない）](#ch3-3)
  - [3-4 顔出し・トーク系の時短](#ch3-4)
  - [3-5 企業・教育チャンネル](#ch3-5)
- [第4章 自動化の仕組み](#ch4)
  - [4-1 Claude Code / Cursor + Skills](#ch4-1)
  - [4-2 MCP（YouTube Data API系・ffmpeg系・ファイル系）](#ch4-2)
  - [4-3 Remotion / ffmpeg によるプログラマブル動画](#ch4-3)
  - [4-4 n8n / Make によるワークフロー自動化](#ch4-4)
  - [4-5 プロンプト集](#ch4-5)
- [第5章 YouTubeのAIコンテンツ方針と法的注意点](#ch5)
  - [5-1 合成コンテンツ（AI使用）の開示](#ch5-1)
  - [5-2 収益化ポリシー：非オリジナル（量産型）／再利用コンテンツ](#ch5-2)
  - [5-3 著作権・音声ライセンスの注意](#ch5-3)
- [第6章 情報サイト・発信者（日本語／英語）](#ch6)
- [第7章 チェックリスト](#ch7)
- [第8章 導入ロードマップ](#ch8)
- [付録 参考URL一覧と要確認事項](#appendix)

---

<a id="summary"></a>
## 3分で分かる要約

**結論：2026年のAI動画制作は「全部AIに作らせる」ではなく、「人間が価値（視点・検証・体験）を出し、AIが作業（下調べ・下書き・音声・素材・編集・字幕・切り抜き・分析）を肩代わりする」形が最も効率的で、かつ収益化ポリシー上も安全です。**

1. **制作工程の8割はAIで短縮できる。** 企画（vidIQ・YouTube Studioのインスピレーション機能・ChatGPT/Claude/Gemini/Perplexity）、台本（LLM）、音声（ElevenLabs、VOICEVOX等）、素材（Veo 3.1、Kling 3.0、Runway Gen-4.5、Hailuo/MiniMax、画像生成AI）、編集（Premiere Pro、DaVinci Resolve、CapCut、Vrew、Descript）、字幕（Whisper系・Vrew）、ショート化（Opus Clip等）、分析（YouTube Studio、vidIQ、TubeBuddy）まで、各工程に実用レベルのツールがそろっています。
2. **2026年の大きな変化は3つ。**
   - **YouTubeが2026年7月に「非オリジナル（inauthentic）コンテンツ」の中身を3分類で明確化**しました（①汎用的・反復的・テンプレート的なコンテンツ、②不快・扇情的なコンテンツ、③健康・金融・法律・政治などのセンシティブな話題を扱うAIペルソナ）。ポリシーの新設ではなく「明確化」ですが、AIでの“量産”チャンネルにとっては事実上の警告です。判定はチャンネル単位です。
   - **AIラベルの自動付与。** YouTubeは2026年5月から、創作者が未申告でも写実的なAI使用をシステムが検出した場合に自動でラベルを付ける運用を開始しました。長尺はプレーヤー直下、ショートは動画上にオーバーレイ表示されます。YouTube公式は「ラベル自体はおすすめ表示や収益化に影響しない」と説明しています。
   - **動画生成AIの勢力図が変化。** OpenAIのSoraアプリは2026年4月26日に終了、APIも2026年9月24日で提供終了と報じられています（複数の比較記事による。OpenAI公式発表の原文は要確認）。現時点ではGoogle Veo 3.1、Kling 3.0、Runway Gen-4.5、MiniMax（Hailuo）H3、ByteDance Seedance 2.x などが主な選択肢です。
3. **日本語ボイスは規約確認が最重要。** VOICEVOXのずんだもんは「VOICEVOX:ずんだもん」のクレジット表記で商用利用可能です。一方、**にじボイスは2026年2月4日でサービス終了**と発表されており（ITmedia、2025年11月21日報道）、既存動画や今後の利用条件は要確認です。
4. **自動化の主役は「AIエージェント＋コード」。** Claude CodeやCursorに、YouTube Data API系MCP、ffmpeg/動画編集系MCP、ファイルシステムMCPを接続し、Remotion（Reactで動画を作るフレームワーク）やffmpegで「台本→音声→字幕→動画→アップロード下書き」までをスクリプト化できます。ノーコード派はn8nやMakeでRSS・スプレッドシート・API連携を組むのが近道です。
5. **効率化の勝ち筋は「型」と「差分」。** フォーマット（構成・テロップ・BGM・サムネの型）はテンプレート化して自動化し、毎回の「中身」（独自の検証、データ、体験、意見）は人間が出す。この分担が「量産型」判定を避けながら制作時間を短縮する唯一の現実解です。
6. **導入は4段階で進める。** ①既存ワークフローに字幕・台本補助だけ入れる → ②音声・素材生成で外注を置き換える → ③スクリプト／MCPで半自動化 → ④分析を回してフォーマットを改善、の順が失敗しにくいです（第8章）。

---

<a id="ch1"></a>
## 第1章 なぜ今「AI × YouTube制作」なのか：2026年の前提整理

### 1-1 制作コストの構造が変わった

従来のYouTube動画制作は、10分の解説動画1本あたり「リサーチ2〜4時間、台本2〜3時間、収録1時間、編集4〜8時間、サムネ1時間、字幕1〜2時間」程度かかるのが一般的でした（個人差が大きく、あくまで目安です）。このうち、**AIで大幅に短縮しやすいのは「調べる」「書き起こす」「並べる」「文字を打つ」作業**です。逆に短縮しにくいのは「何を言うべきかを決める」「事実を確認する」「面白さを判断する」作業です。

AIを使った効率化の本質は、後者に人間の時間を集中させることです。たとえば次のような分担が典型です。

| 工程 | AIに任せる部分 | 人間が担う部分 |
|---|---|---|
| 企画 | トレンド抽出、競合動画の要約、キーワード候補 | テーマの選定、切り口・独自性の決定 |
| リサーチ | 資料検索、要約、論点整理 | 一次情報の確認、ファクトチェック |
| 台本 | 構成案、下書き、言い回しの調整 | 主張・体験・意見の追加、最終校閲 |
| 音声 | TTSでの読み上げ、ノイズ除去 | 読み間違い修正、抑揚の最終確認 |
| 素材 | 画像・動画・BGMの生成、ストック検索 | 権利確認、世界観の統一 |
| 編集 | カット、テロップ自動化、無音削除 | テンポ・演出の判断 |
| サムネ | 背景・要素の生成、文言案 | 最終デザイン、クリック率の検証 |
| 分析 | データ取得、レポート作成 | 改善策の意思決定 |

### 1-2 2026年の「AIスロップ」問題とプラットフォームの反応

生成AIの普及により、YouTubeには低品質な自動生成動画（英語圏では “AI slop” と呼ばれます）が急増しました。これに対してYouTubeは、2025年7月15日に「repetitious content（反復コンテンツ）」ポリシーを「inauthentic content（非オリジナル／非真正コンテンツ）」に改称し、「大量生産・反復的なコンテンツも含む」ことを明確化しました。さらに2026年7月には、YouTubeのTrust & Safety担当VPであるMatt Halprin氏がCreator Insiderの動画（聞き手はCreator LiaisonのRene Ritchie氏）で、非オリジナルコンテンツを3分類で具体的に説明しました（TechCrunch 2026年7月20日、Tubefilter 2026年7月13日、NetInfluencerの報道による）。

ここで重要なのは、**YouTubeは「AIを使うこと」自体を禁止していない**点です。AIを活用していても、オリジナルで実質的な価値のある動画は収益化の対象です。問題視されるのは、「テンプレート的で、動画ごとの違いがほとんどなく、制作者の独自の洞察や視点が加わっていない」コンテンツです。詳しくは[第5章](#ch5)で扱います。

### 1-3 効率化の3原則

本レポート全体を通じて、次の3原則を前提にします。

1. **型はAI、中身は人間。** 構成テンプレート、テロップスタイル、BGMの選定ルールなどの「型」は自動化します。一方、各動画の論点・検証結果・体験談などの「中身」は人間が責任を持ちます。
2. **一次情報と検証を必ず挟む。** LLMはもっともらしい誤情報（ハルシネーション）を生成します。数字・固有名詞・日付・引用は必ず一次ソースで確認する工程をワークフローに組み込みます。
3. **権利と開示を最初に設計する。** 音声・画像・BGM・動画素材のライセンス、YouTubeのAI使用開示、キャラクター利用規約のクレジット表記などは、後から直すのが最も高コストです。テンプレートの段階で組み込みます。

### 1-4 本レポートの使い方

- まず[第2章](#ch2)で工程ごとのツールを把握し、自分のボトルネック工程を特定してください。
- [第3章](#ch3)で自分のジャンルに近いパイプラインを選び、[第4章](#ch4)で自動化の度合いを決めます。
- 公開前に[第5章](#ch5)と[第7章](#ch7)のチェックリストを確認してください。
- 導入の順番は[第8章](#ch8)のロードマップを参照してください。

---

<a id="ch2"></a>
## 第2章 工程別ツールマップ

この章では、制作工程ごとに代表的なツールと使い方のコツを整理します。料金や上限はプランの改定が頻繁なため、原則として記載せず、必要に応じて「要確認」とします。

<a id="ch2-1"></a>
### 2-1 企画・リサーチ

#### 主なツール

| ツール | 用途 | ポイント |
|---|---|---|
| YouTube Studio（リサーチ／インスピレーション系タブ） | 視聴者が検索しているキーワード、コンテンツギャップの把握 | 自チャンネルの視聴者データに基づくため最も信頼性が高い。AIによる企画提案機能の提供範囲・日本語対応は要確認 |
| vidIQ | キーワード調査、競合分析、トレンド、AIコーチ | ニッチの選定や「次に何を作るか」の判断材料に強い |
| TubeBuddy | キーワード調査、タイトル／サムネのA/Bテスト、一括編集 | 既存動画のメタデータ最適化に便利 |
| ChatGPT / Claude / Gemini | 企画ブレスト、構成案、競合動画の要約 | Webブラウジングや深掘りリサーチ機能（Deep Research系）で下調べを短縮 |
| Perplexity | 出典付きの検索・要約 | 出典URLを辿ってファクトチェックしやすい |
| NotebookLM（Google） | 手元資料（PDF・URL・動画）を読み込ませて要約・Q&A | 資料ベースの解説動画のリサーチに向く。音声概要機能も参考になる |
| Googleトレンド | 検索関心の推移 | 季節性・急上昇の確認 |

#### 効率的なやり方

1. **「需要」と「供給」を分けて調べる。** 需要（検索ボリューム、視聴者の質問、コメント欄の要望）はYouTube Studioやvidiq等で、供給（既存動画の質と量）は検索結果の上位動画を見て判断します。需要が大きく、供給の質が低いテーマが狙い目です。
2. **競合動画の「字幕」を要約させる。** 上位動画の文字起こし（自動字幕）をLLMに要約させ、「どの論点が扱われ、何が欠けているか」を一覧化すると、差別化ポイントがすぐに見えます。ただし他者動画の内容をそのまま再構成するのは著作権・オリジナリティの両面で問題があるため、あくまで「欠けている論点」を探す目的に限定します。
3. **コメント欄を企画の源泉にする。** 自チャンネルのコメントをYouTube Data API（`commentThreads.list`）で取得し、LLMで「質問」「不満」「要望」に分類すると、視聴者起点の企画リストが作れます（実装例は[第4章](#ch4)）。

#### 企画用プロンプト例

```text
あなたはYouTubeの企画編集者です。
以下の条件で動画企画を10案出してください。

# チャンネル
- ジャンル: 個人投資家向けの「制度・税金」解説（顔出しなし、ナレーション）
- 視聴者: 30〜50代の会社員、投資歴1〜5年
- 既存の人気動画: 「新NISAの売却タイミング」「iDeCoの受け取り方」

# 条件
- 各案について「タイトル案（32文字以内）」「視聴者の悩み」「独自の切り口」「必要な一次情報（官公庁・公式資料）」を表で出す
- 既存の上位動画でよく見る切り口と、そうでない切り口を区別する
- 断定的な投資助言にならないテーマを優先する
- 確証がない数値は「要確認」と書く
```

<a id="ch2-2"></a>
### 2-2 台本

#### 主なツール

- **Claude（Anthropic）**：長文の構成・文体の一貫性に強く、台本の推敲に向きます。
- **ChatGPT（OpenAI）**：アイデア出し、フック（冒頭の掴み）案の大量生成に便利です。
- **Gemini（Google）**：Google検索・YouTubeとの連携、長い資料の読み込みに強みがあります。
- **Notion AI / Googleドキュメントの生成AI機能**：チームでの台本管理と共同編集に。

各モデルの最新バージョン名（例：Claude、GPT、Geminiの最新世代）は更新が速いため、本レポートでは特定バージョンを推奨しません（最新は要確認）。

#### 台本作成の型（解説動画）

YouTubeの解説動画では、次の構成が定番です。

1. **フック（0〜15秒）**：結論の予告、意外な事実、視聴者の悩みの代弁
2. **前提共有（15〜60秒）**：この動画で分かること、対象者
3. **本論（3〜5ブロック）**：1ブロック＝1論点。各ブロックの最後に小さな結論
4. **具体例・検証**：自分で試した結果、データ、図解（ここが独自性の核）
5. **まとめ＋次の行動**：要点3つ、関連動画への誘導

#### 台本プロンプト例（構成→執筆→校閲の3段階）

```text
## STEP1: 構成案
以下のリサーチメモをもとに、10分（約3,000字）の解説動画の構成案を作ってください。
- 各ブロックに「目的」「話す要点」「画面に出す図・素材の案」「想定尺（秒）」を付ける
- 冒頭15秒のフックを3パターン提案
- 私自身の検証パート（下記メモ）を必ず中盤に入れる

[リサーチメモ]
...
[自分の検証メモ]
...
```

```text
## STEP2: 本文執筆
STEP1の構成案Bで台本を書いてください。
- 話し言葉。1文は40字以内を目安
- 専門用語は初出時に一言で言い換える
- 数字・固有名詞には【出典: 】の空欄を付け、私が後で埋められるようにする
- 断定できない事柄は「〜とされています」「〜と報じられています」とする
- 最後に「この台本で事実確認が必要な箇所」を箇条書きで列挙
```

```text
## STEP3: 校閲
あなたは厳しい校閲者です。次の台本について、
1) 事実誤認の可能性がある箇所
2) 論理の飛躍
3) 視聴者が離脱しそうな冗長部分
4) 読み上げ音声（TTS）で誤読しそうな語（読み仮名を提案）
を表で指摘してください。修正版の全文は不要です。
```

STEP3の「TTSで誤読しそうな語」の抽出は、後工程（音声）の手戻りを大きく減らします。VOICEVOXやゆっくりムービーメーカー等のユーザー辞書に登録する語のリストとしてそのまま使えます。

#### ゆっくり・ずんだもん向けの掛け合い台本

```text
以下の解説台本を、「ずんだもん（質問役・語尾は『〜のだ』）」と
「四国めたん（解説役・丁寧語）」の掛け合いに変換してください。
- 出力形式はCSV: speaker,text,emotion,scene_note
- 1セリフは50字以内
- ずんだもんの誤解→めたんの訂正、という流れを各ブロックに1回入れる
- 実在の人物・団体を揶揄しない
- キャラクターの公式ガイドラインに反する表現（過度な暴力・性的表現等）を避ける
```

CSV形式で出力させておくと、後述の自動化スクリプト（VOICEVOX APIで一括合成→字幕ファイル生成）にそのまま流し込めます。

<a id="ch2-3"></a>
### 2-3 音声（ナレーション・キャラクターボイス）

#### 主なツール

| ツール | 特徴 | 商用利用・注意点 |
|---|---|---|
| **ElevenLabs** | 自然な多言語TTS。最新モデル「Eleven v3」は日本語を含む70以上の言語に対応し、音声タグ（感情・ため息など）や複数話者の対話生成（Text to Dialogue API）に対応。正式提供（GA）済みと公式ブログに記載 | プランにより商用利用条件・ボイスクローンの可否が異なる（要確認）。他人の声のクローンは本人の同意が必須 |
| **VOICEVOX** | 無料・オープンソースの日本語TTS。ずんだもん、四国めたん、春日部つむぎ等多数のキャラクター | ソフトウェア自体は商用・非商用可。音声は**各キャラクターの規約**に従う。ずんだもんは「VOICEVOX:ずんだもん」のクレジット表記で商用・非商用可。クレジットなしの商用利用は1キャラクターあたり40万円（税別）の契約が必要（東北ずん子公式「音源利用ガイドライン」） |
| **にじボイス**（Algomatic） | アニメ調の感情豊かな日本語ボイス | **2026年2月4日にサービス終了と発表**（ITmedia 2025年11月21日）。日本俳優連合から一部ボイスが声優の声に酷似しているとの削除要請を受け、同社は「法的な権利侵害は確認されなかった」としつつ終了を決定。終了後の既存音声の利用条件は要確認 |
| **CoeFont** | 日本語TTS、多数の声 | 商用条件はプランごとに要確認 |
| **A.I.VOICE / VOICEPEAK** | 買い切り型の日本語音声合成ソフト | 製品ごとに商用ライセンス条件あり（要確認） |
| **AquesTalk（ゆっくりボイス）** | いわゆる「ゆっくり」の音声 | 商用利用（収益化動画を含む）にはライセンス購入が必要とされるケースがある。最新条件は株式会社アクエスト公式で要確認 |
| **Adobe Podcast（Enhance Speech）** | 録音音声のノイズ除去・音質改善 | 自分の声を使う場合の時短に有効 |
| **Descript（Overdub等）** | 自分の声の修正・差し替え | 自分の声の登録が前提 |

#### 効率化のコツ

1. **ユーザー辞書を育てる。** チャンネルで頻出する固有名詞・専門用語の読みを辞書登録しておくと、毎回の修正時間がゼロに近づきます。VOICEVOXはエンジンAPIでユーザー辞書を操作できます（`/user_dict_word` エンドポイント。仕様は公式のAPIドキュメントで要確認）。
2. **台本側で読みを制御する。** 数字（「1,000万円」）や英字略語（「ETF」）は、台本段階で「せんまんえん」「イーティーエフ」のように読み仮名を併記するルールにすると誤読が減ります。
3. **セリフ単位で合成する。** 1セリフ＝1音声ファイルにしておくと、修正時に該当セリフだけ再合成でき、字幕のタイミングも音声長から自動計算できます。
4. **クローン音声は「自分の声」に限定する。** 他人（有名人・声優・知人）の声をクローンするのは、肖像権・パブリシティ権・不正競争の観点で高リスクです。YouTubeも「肖像検出（likeness detection）」機能を拡張しており、声のなりすましの検出対象化が進んでいると報じられています（詳細な提供範囲は要確認）。

#### VOICEVOXエンジンAPIでの合成例（Python）

VOICEVOXエンジンはローカルで起動すると `http://127.0.0.1:50021` でHTTP APIを提供します。音声合成は「`audio_query` でクエリ作成 → `synthesis` で合成」の2段階です。

```python
# voicevox_batch.py
import csv, json, pathlib, requests

ENGINE = "http://127.0.0.1:50021"
SPEAKERS = {"zundamon": 3, "metan": 2}  # スタイルIDは環境で /speakers を叩いて要確認

def synth(text: str, speaker_id: int, out: pathlib.Path, speed=1.1):
    q = requests.post(f"{ENGINE}/audio_query",
                      params={"text": text, "speaker": speaker_id}).json()
    q["speedScale"] = speed
    q["prePhonemeLength"] = 0.1
    q["postPhonemeLength"] = 0.1
    wav = requests.post(f"{ENGINE}/synthesis",
                        params={"speaker": speaker_id},
                        data=json.dumps(q),
                        headers={"Content-Type": "application/json"})
    out.write_bytes(wav.content)

out_dir = pathlib.Path("build/voice"); out_dir.mkdir(parents=True, exist_ok=True)
with open("script.csv", encoding="utf-8") as f:
    for i, row in enumerate(csv.DictReader(f)):
        synth(row["text"], SPEAKERS[row["speaker"]], out_dir / f"{i:04d}_{row['speaker']}.wav")
```

> スタイルID（例：ずんだもんノーマル＝3）は、VOICEVOXの公式リポジトリの一覧に記載がありますが、バージョンによって変わる可能性があるため、実行環境で `GET /speakers` を呼び出して確認してください。

#### ElevenLabs APIでのナレーション生成例

```bash
# ElevenLabs Text to Speech（エンドポイント・モデルIDは公式ドキュメントで要確認）
curl -X POST "https://api.elevenlabs.io/v1/text-to-speech/${VOICE_ID}" \
  -H "xi-api-key: ${ELEVENLABS_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "[calm] 今日は、AIで動画制作を効率化する方法を解説します。",
    "model_id": "eleven_v3"
  }' \
  --output narration.mp3
```

角括弧の音声タグ（`[calm]` など）はEleven v3の機能です。使えるタグは声や文脈によって異なると公式に記載されているため、短い文で試してから本番に使うのが安全です。

<a id="ch2-4"></a>
### 2-4 動画生成

#### 2026年10月時点の主要モデル

以下は複数の比較記事（pixflow.net、imagine.art、melies.co、dreamega.ai 等、2026年5〜9月更新）を突き合わせた整理です。仕様・料金は頻繁に変わるため、利用前に公式で要確認です。

| モデル | 提供元 | 特徴（報道・比較記事ベース） | YouTubeでの使いどころ |
|---|---|---|---|
| **Veo 3.1** | Google | ネイティブ音声（セリフ・効果音・環境音）同時生成、最大4K、参照画像対応。4/6/8秒クリップ。軽量版「Veo 3.1 Lite」も登場と報じられる | Bロール、イメージ映像、短いドラマ演出。YouTube Shorts内の生成機能（Dream Screen等）でも利用可能とされる |
| **Sora 2** | OpenAI | 写実性が高いが、**Soraアプリは2026年4月26日に終了、APIは2026年9月24日で終了**と複数記事が報道（OpenAI公式原文は要確認） | 新規プロジェクトでの採用は非推奨 |
| **Runway Gen-4 / Gen-4.5** | Runway | カメラ制御、参照画像によるキャラクター一貫性、編集機能と一体化 | 広告・ブランド動画、同一キャラクターの複数ショット |
| **Kling 3.0 / O3** | Kuaishou（快手） | 3〜15秒、4K版あり、ネイティブ音声、マルチショット生成。コストパフォーマンスが高いと評価 | シネマティックなBロール、動きの多いシーン |
| **Hailuo（MiniMax H3）** | MiniMax | 表現力のある動き、2K出力、最初と最後のフレーム指定 | アクション、商品映像 |
| **Seedance 2.x** | ByteDance | 最大20〜30秒程度の長めのクリップ、ネイティブ音声と報じられる | 長めのワンカット |
| **Wan 3.0** | Alibaba | 長めのクリップ、オープン系モデルの流れを汲む（ライセンスは要確認） | ローカル／低コスト運用 |
| **Pika / Luma** | Pika Labs / Luma AI | SNS向けの手軽さ | ショートのアクセント |

#### 動画生成AIの効率的な使い方

1. **「全編生成」ではなく「Bロール生成」に使う。** 10分の解説を全て生成映像で作るのは、コスト・一貫性・ポリシー（反復的コンテンツ）の面で非効率です。説明を補助する5〜8秒のイメージ映像を、要所に差し込む使い方が費用対効果に優れます。
2. **画像→動画（Image to Video）で一貫性を担保する。** まず画像生成AIでキーフレームを作り、それを動画化すると、キャラクターや色調をそろえやすくなります。
3. **プロンプトを構造化する。** 被写体・動作・カメラ・光・スタイル・音を分けて書くと、再現性が上がります。
4. **生成物にはAI使用の開示を。** 写実的な人物・出来事・場所をAIで生成した場合は、YouTube Studioの「AI使用（AI use）」で開示が必要です（[第5章](#ch5-1)）。

#### 動画生成プロンプトのテンプレート

```text
[Subject] 30代の日本人女性、オフィスカジュアル、ノートPCに向かう
[Action] 画面を見て小さくうなずき、メモを取る
[Camera] ミディアムショット、ゆっくりとしたドリーイン、35mm相当
[Lighting] 午後の自然光、窓からのやわらかい逆光
[Style] 落ち着いたドキュメンタリー調、彩度控えめ
[Audio] キーボードの打鍵音、遠くのオフィス環境音、セリフなし
[Duration] 8秒 / 16:9
[Negative] テキスト、ロゴ、歪んだ手指
```

```text
# ショート用（9:16）
縦型9:16、6秒。夜の東京の交差点を真上から俯瞰するタイムラプス風。
人の流れが光の線になる。カメラは静止。サイバー感のある青とマゼンタ。
環境音のみ。字幕・文字は入れない。
```

#### 料金感覚の注意

比較記事では1秒あたり数十円〜百数十円相当（1080p、ドル建て換算）といった試算が掲載されていますが、クレジット制・サブスク・API従量課金が混在しており、単純比較は困難です。**月の生成秒数の上限を決めてから**ツールを選ぶのが実務的です（具体的な料金は要確認）。

<a id="ch2-5"></a>
### 2-5 画像生成

| ツール | 用途 | 注意点 |
|---|---|---|
| ChatGPT（GPT系の画像生成） | 図解風イラスト、文字入り画像、サムネ素材 | 日本語文字の描画精度は改善しているが、誤字チェックは必須 |
| Gemini（Imagen系／画像編集機能） | 画像生成・部分編集 | 生成画像には透かし（SynthID）が付く場合がある |
| Midjourney | 世界観のあるアート、背景 | 商用利用条件はプランで要確認 |
| Adobe Firefly | 商用利用を意識した生成、Photoshopとの連携 | Adobeは学習データの権利処理を前面に出している（補償条件等は要確認） |
| Stable Diffusion系 / FLUX系 | ローカル生成、LoRAでの画風固定 | モデルごとのライセンスが異なる（要確認） |
| Canva（AI機能） | サムネ・図解の量産テンプレート | テンプレート素材のライセンス確認 |
| Ideogram | 文字入りデザイン | 商用条件要確認 |

**効率化のコツ**

- **スタイルガイドを1枚作る。** 色（HEX）、フォント、人物のテイスト、背景の質感を決め、プロンプトの冒頭に毎回入れます。
- **図解はAIに「描かせる」より「コードで作る」。** グラフや比較表はmatplotlib、Mermaid、HTML/CSS、Remotionなどで生成した方が正確です。数字の入った図をAI画像生成で作ると、数値が崩れることがあります。
- **人物の写実画像は避けるか開示する。** 実在人物に似た画像はトラブルの元です。

<a id="ch2-6"></a>
### 2-6 編集

| ツール | AI機能の例 | 向いている人 |
|---|---|---|
| **Adobe Premiere Pro** | 文字起こしベース編集、自動キャプション、生成拡張（Generative Extend）、音声強調、シーン編集検出など | 業務利用、Adobe製品との連携重視 |
| **DaVinci Resolve**（Blackmagic Design） | 無料版でも高機能。Studio版でAI機能（文字起こし、字幕生成、音声分離、マジックマスク、スマートリフレーム等） | コスパ重視、カラー・音声までこだわる人 |
| **CapCut** | 自動字幕、テンプレート、背景除去、テキスト読み上げ、縦型化 | ショート中心、スマホ編集 |
| **Vrew** | 音声認識で自動字幕＋テキスト編集でカット、AI音声、テキストから動画生成 | 字幕・トーク動画の時短。日本語ユーザーが多い |
| **Descript** | 文字起こしを編集すると動画が編集される、フィラー除去、Studio Sound、AIエージェント的な編集支援 | ポッドキャスト・トーク系、英語中心だが日本語対応状況は要確認 |
| **ゆっくりムービーメーカー（YMM4）** | ゆっくり／VOICEVOX系キャラの立ち絵・口パク・字幕を一体で扱える | ゆっくり／ずんだもん解説 |
| **AviUtl / AviUtl ExEdit2** | 日本の定番フリー編集ソフト。プラグインで拡張 | 既存ノウハウ資産を活かしたい人（最新版の状況は要確認） |

**編集の時短テクニック**

1. **「テキストベース編集」を基本にする。** Premiere Pro、DaVinci Resolve、Descript、Vrewはいずれも文字起こしを編集してカットできる機能を持ちます。言い間違い・無音・フィラーの削除はテキスト上で済ませます。
2. **ジェットカット（無音削除）を自動化する。** ffmpegの `silencedetect` で無音区間を検出し、自動でカットする方法もあります（[第4章](#ch4-3)）。
3. **テロップのスタイルをプリセット化する。** Premiereの「エッセンシャルグラフィックス」、DaVinciの「Text+」テンプレート、YMM4のアイテムテンプレートなどで、話者ごとのスタイルを固定します。
4. **BGM・効果音は「ライブラリ＋ルール」で。** 「導入はこの曲、解説パートはこの曲、まとめはこの曲」と決めれば選曲時間がゼロになります。

<a id="ch2-7"></a>
### 2-7 字幕

| ツール | 特徴 |
|---|---|
| YouTube自動字幕 | 無料。精度は向上しているが固有名詞に弱い。公開後に編集可能 |
| Vrew | 日本語の自動字幕と分割が速い。SRT出力可 |
| Premiere Pro / DaVinci Resolve の文字起こし | 編集ソフト内で完結 |
| OpenAI Whisper（オープンソース）/ faster-whisper / whisper.cpp | ローカルで高精度な文字起こし。SRT/VTT出力 |
| ElevenLabs Scribe（v2） | 音声認識API。2026年1月にScribe v2が発表（公式ブログの記載より） |
| YouTubeの自動吹き替え（Auto dubbing） | 多言語の音声トラックを自動生成する機能。対象チャンネル・対応言語は要確認 |

**ベストプラクティス**

- **台本があるなら「音声認識」より「強制アライメント」。** TTSで作った音声なら、台本テキストと音声長から字幕タイミングを計算できるので誤字ゼロになります。
- **字幕は1行最大16〜20文字・2行以内**を目安にし、句読点や意味の切れ目で改行します（LLMに改行位置を提案させるのも有効）。
- **多言語字幕はSRTを翻訳してアップロード。** LLMで翻訳する際は「タイムコード行を変更しない」ことを明記します。

```text
次のSRTを英語に翻訳してください。
- 番号行とタイムコード行は一切変更しない
- 1字幕あたり42文字以内に収め、超える場合は自然に要約
- 固有名詞は下記の対訳表に従う
[対訳表] ずんだもん=Zundamon, 新NISA=new NISA
[SRT]
...
```

<a id="ch2-8"></a>
### 2-8 サムネイル

サムネイルはクリック率（CTR）に直結するため、AIは「案出し」と「素材作り」に使い、最終判断はデータで行うのが鉄則です。

- **文言案**：LLMに「動画の約束（視聴後に得られるもの）を8〜13文字で」と制約して20案出させます。
- **素材**：背景・オブジェクトを画像生成AIで作り、Photoshop・Canva・Figmaで合成します。
- **テスト**：YouTube Studioの「テストと比較（Test & Compare）」機能でサムネイルのA/Bテストができます（タイトルのテスト対応範囲は要確認）。TubeBuddyにもA/Bテスト機能があります。
- **一貫性**：チャンネル全体で配色・フォント・顔／キャラの位置を統一すると、ブラウジング時の認知が上がります。

```text
サムネイル文言を20案。
- 動画テーマ: 「AIで台本作成時間を1/3にした手順」
- 8〜13文字、数字を含む案を半分以上
- 煽り表現・誇大表現（「絶対」「100%」など）は禁止
- 各案に「狙う感情（好奇心/不安解消/得したい）」を付ける
```

<a id="ch2-9"></a>
### 2-9 ショート切り抜き

| ツール | 特徴（公式・解説記事ベース） |
|---|---|
| **Opus Clip（OpusClip）** | 長尺から見どころを自動検出し、9:16にリフレーム、字幕・トランジションを付与。「ClipAnything」は自然言語指定で映像・音声・感情を解析して切り出し。日本語の文字起こし・字幕に対応。バイラルスコアは有料プラン以上（料金詳細は要確認） |
| vidIQ（クリップ機能） | 簡易的な切り抜き機能を提供と紹介されている（詳細は要確認） |
| CapCut | 長尺から自動でショートを生成する機能あり（提供地域・名称は要確認） |
| Descript / Premiere Pro / DaVinci Resolve | 自動リフレーム（被写体追従での縦型化） |
| YouTube Studio | 長尺動画から「ショートを作成」機能（既存動画の一部を使ってショート化） |

**切り抜き運用の注意**

- **他人の動画の切り抜きは、権利者の許諾（切り抜きガイドライン）がある場合に限る。** 多くの日本の配信者・VTuber事務所が切り抜きガイドラインを公開していますが、収益化の可否・クレジット表記・禁止事項はそれぞれ異なります。
- **YouTubeの「再利用されたコンテンツ（reused content）」ポリシー**に注意。他者のコンテンツを大きな改変・解説なしに使ったチャンネルは収益化できません（[第5章](#ch5-2)）。
- 自分の長尺からの切り抜きは問題ありませんが、**同じ構成のショートを延々と並べると「反復的」と見られる**可能性があるため、フック・テロップ・切り口に変化を付けます。

<a id="ch2-10"></a>
### 2-10 分析

| ツール | できること |
|---|---|
| **YouTube Studio（アナリティクス）** | インプレッション、CTR、視聴維持率、トラフィックソース、視聴者層。最も正確な一次データ。リサーチタブで視聴者の検索傾向 |
| **YouTube Analytics API / Reporting API** | 上記データのプログラム取得。自動レポートやダッシュボード化に |
| **vidIQ** | キーワードスコア、競合チャンネル比較、AIコーチ、日々の企画提案 |
| **TubeBuddy** | キーワード調査、A/Bテスト、タグ・メタデータの一括管理、ベストな投稿時間の提案 |
| Looker Studio / スプレッドシート | API経由で取得したデータの可視化 |

**分析をAIで効率化するやり方**

1. 週1回、YouTube Analytics APIで「動画別のインプレッション、CTR、平均視聴率、視聴維持率の30秒地点の値」を取得し、CSVに保存。
2. LLMに「上位3本と下位3本の違いを、タイトル・サムネ文言・冒頭15秒の台本から推測し、次の動画で試す仮説を3つ」と依頼。
3. 仮説を次の動画で1つだけ変えて検証（複数を同時に変えない）。

```text
以下は直近10本の動画データ（CSV）と、各動画の冒頭15秒の台本です。
1) CTRと平均視聴率の相関
2) 視聴維持率が30秒地点で急落している動画の共通点
3) 次の3本で検証すべき仮説（1本につき1変数のみ変更）
を出してください。データから言えないことは「データ不足」と明記すること。
```

---

<a id="ch3"></a>
## 第3章 ジャンル別の構築パイプライン

ここでは、代表的なジャンルごとに「どのツールを、どの順番で、どこまで自動化するか」を具体的に示します。いずれも**人間が価値を足す工程（★）**を明示しています。★を省くと、収益化ポリシー上の「非オリジナルコンテンツ」に近づきます。

<a id="ch3-1"></a>
### 3-1 ゆっくり／ずんだもん解説

日本独自の人気フォーマットで、キャラクター音声の掛け合いで解説する形式です。顔出し・声出し不要で始めやすい反面、**参入者が非常に多く、テンプレート的な動画が量産されやすいジャンル**でもあります。

#### パイプライン

```text
[1] テーマ選定 ★（独自の視点・切り口を決める）
      ↓  vidIQ / YouTube Studio リサーチ / LLMでの競合分析
[2] リサーチ ★（一次資料を読む、数字を確認）
      ↓  Perplexity / NotebookLM / 公式資料
[3] 台本（解説文）→ 掛け合いCSVに変換
      ↓  Claude / ChatGPT（第2章のプロンプト）
[4] 校閲 ★（事実確認、キャラ規約の確認、誤読チェック）
      ↓
[5] 音声合成（VOICEVOX / AquesTalk）
      ↓  ユーザー辞書を適用、セリフ単位でwav出力
[6] 動画組み立て（YMM4 またはRemotion/ffmpeg）
      ↓  立ち絵・口パク・字幕・背景・BGM
[7] 図解・素材の挿入 ★（オリジナルの図表、自作のデータ可視化）
      ↓
[8] サムネ作成（テンプレ＋文言A/Bテスト）
      ↓
[9] 公開設定（AI使用の開示判断、クレジット表記、チャプター）
```

#### ポイント

- **クレジット表記の自動挿入。** 概要欄テンプレートに「VOICEVOX:ずんだもん」「VOICEVOX:四国めたん」等を必ず含めます。ずんだもんの公式ガイドラインでは「動画サイトの場合は説明画面や動画内のクレジットなど、ユーザーが気になって見にいった際にわかる程度のところに記載」とされています。
- **立ち絵素材の規約も確認。** 音声だけでなく、立ち絵（イラスト）にも作者ごとの利用規約があります。商用可否・改変可否・クレジット要否を素材ごとに管理表にまとめます。
- **AquesTalk（ゆっくり）は商用ライセンスに注意。** 収益化する場合のライセンス要否は、アクエスト社の最新規約で確認してください（要確認）。
- **「反復的」判定を避ける工夫。** 毎回同じ背景・同じ掛け合いパターン・同じ結論の流れにならないよう、①独自の図解、②自分で調べた一次データ、③視聴者コメントへの回答パート、④シリーズごとに異なる演出、などの「差分」を意図的に入れます。YouTubeが例示する非収益化対象には「同じ状況に同じキャラクターが置かれ、同じ結末を繰り返す動画」が含まれています。
- **AI使用の開示。** キャラクター音声によるアニメ調の解説は、一般に「写実的なAIコンテンツ」には当たらないと考えられますが、実在人物の声に似せた音声や、写実的な生成映像を挿入した場合は開示が必要です。判断に迷う場合は開示する方が安全です。

#### 制作時間の目安（10分動画）

| 工程 | 手作業中心 | AI活用後（目安） |
|---|---|---|
| リサーチ | 3時間 | 1.5時間（一次資料確認は残る） |
| 台本 | 3時間 | 1時間 |
| 音声・誤読修正 | 2時間 | 0.5時間（辞書が育てばさらに短縮） |
| 動画組み立て | 5時間 | 1.5〜2時間 |
| サムネ | 1時間 | 0.5時間 |

上記は筆者による一般的な目安で、実測データではありません。

<a id="ch3-2"></a>
### 3-2 顔出しなし（ナレーション＋素材）チャンネル

雑学・歴史・テクノロジー解説・ニュース解説・ドキュメンタリー風など、ナレーションと映像素材で構成するジャンルです。英語圏では “faceless channel” と呼ばれます。

#### パイプライン

```text
[1] 企画 ★：自分が詳しい・継続的に調べられるテーマに絞る
[2] リサーチ ★：一次資料・書籍・論文・公的統計
[3] 台本：LLMで構成→執筆→校閲（第2章の3段階プロンプト）
[4] ナレーション：
      A) 自分の声で収録 → Adobe Podcast / DaVinciで音質改善（推奨）
      B) ElevenLabs等のTTS（自分の声のクローン or ライセンス済みボイス）
[5] 映像素材：
      - ストック映像（Pexels, Storyblocks, Artgrid 等。ライセンス確認）
      - 生成映像（Veo 3.1 / Kling / Runway）をBロールとして要所に
      - 自作の図解・地図・年表（Remotion/After Effects/Keynote）★
[6] 編集：Premiere / DaVinci。台本の段落ごとにマーカーを打ち素材を配置
[7] 字幕：台本から生成（TTSの場合）または文字起こし
[8] サムネ・タイトル：A/Bテスト
[9] 公開：写実的な生成映像を使った場合は「AI使用」を「はい」に
```

#### ポイント

- **ナレーションは可能なら自分の声。** TTSの品質は非常に高くなっていますが、視聴者の信頼や「人間の視点」が伝わりやすいのは本人の声です。TTSを使う場合も、台本に自分の経験・意見を明確に入れます。
- **センシティブな話題でのAIペルソナは避ける。** 健康・金融・法律・政治について、AIで作った架空の「専門家」に語らせるチャンネルは、2026年7月に明確化された非収益化カテゴリに該当します。こうした分野では、実名の発信者として責任を持つか、出典を明示した解説に徹します。
- **生成映像の比率を意識する。** 全編が生成映像のスライドショー的な動画は「テンプレート的」と判断されやすくなります。ストック映像・自作図解・生成映像を組み合わせ、映像そのものが説明の役に立っている状態を目指します。
- **素材管理の自動化。** 台本の各段落に `[B-ROLL: 港に入る蒸気船, 1900年代, 白黒]` のようなタグを入れておくと、LLMで素材検索クエリや生成プロンプトに自動変換できます。

```text
以下の台本の各段落について、Bロール案を出してください。
出力はJSON配列: {"paragraph_id", "type": "stock|generate|diagram", "query_or_prompt", "duration_sec"}
- 実在の人物・事件の写実的な生成は禁止（diagramかstockにする）
- 図解(diagram)は数値や年表など、正確さが必要なものに使う
```

<a id="ch3-3"></a>
### 3-3 ショート量産（ただし“量産型”にしない）

YouTube Shortsは短時間で制作でき、新規視聴者の獲得に強い一方、**最も「テンプレート量産」に陥りやすい形式**です。「量産」は「同じものを大量に作る」ではなく、「1つの素材・知見から、切り口の異なる複数の作品を効率よく作る」と捉え直す必要があります。

#### パターンA：長尺からの切り出し（推奨）

```text
[1] 長尺動画（自分のもの）を公開
[2] Opus Clip / Descript / Premiere の自動リフレームで候補を20本抽出
[3] ★ 人間が5本に絞る（フックが強い、単体で完結する）
[4] ★ 冒頭1〜2秒のフック文言を付け直す（LLMで案出し→選定）
[5] 字幕スタイル・BGMをショート用テンプレートで統一
[6] 長尺への関連動画リンクを設定
```

#### パターンB：ショート専用の企画

```text
[1] 「1ショート＝1知見」のネタ帳をスプレッドシートで管理 ★
[2] LLMで30〜45秒の台本に整形（フック/本題/オチ）
[3] 音声：自分の声 or TTS
[4] 映像：Remotionテンプレート（テロップアニメ＋図解）＋実写/生成Bロール
[5] 自動レンダリング → 予約投稿（下書きまで自動、公開判断は人間）★
```

#### 「量産型」にしないためのチェック

- 直近10本を連続で見たとき、**「名前や画像を差し替えただけ」に見えないか**（YouTubeは「キャラクター名や設定、画像だけが変わるAI生成ストーリー」も非収益化の例として挙げていると報じられています）。
- 各ショートに**その動画でしか得られない情報や視点**があるか。
- スクロールするテキスト・スライドショーだけで、ナレーションや解説がない形式になっていないか。
- 感情を煽る目的だけの演出（動物を危険に晒して救うなどの「やらせ」的演出）をしていないか。

<a id="ch3-4"></a>
### 3-4 顔出し・トーク系の時短

顔出しやトーク系はAIで「作る」より、AIで「削る・整える」のが中心です。

```text
[1] 収録前：LLMで話す論点リスト（台本ではなく箇条書き）を作成
[2] 収録：カメラ＋マイク。言い直しは気にせず話す
[3] 文字起こし：Premiere / DaVinci / Descript / Vrew
[4] テキスト編集：言い間違い・フィラー・無音をテキスト上で削除
[5] 音声：Enhance Speech / Studio Sound 等でノイズ除去
[6] テロップ：自動字幕 → 強調したい箇所だけ装飾テロップ
[7] Bロール：要所に画面録画・図解・生成映像
[8] ショート：Opus Clip等で切り出し → ★ 人間が選定
[9] チャプター・概要欄：文字起こしからLLMで生成 → ★ 確認
```

```text
以下の文字起こし（タイムスタンプ付き）から、
1) YouTubeチャプター（最初は必ず0:00、各3語〜12語）
2) 概要欄の要約（200字）
3) ショートに向く区間の候補5つ（開始・終了時刻と理由）
を作ってください。発言にない内容を付け加えないこと。
```

<a id="ch3-5"></a>
### 3-5 企業・教育チャンネル

企業の製品解説、社内研修、オンライン講座などでは、**正確性・ブランド一貫性・多言語化**が重要です。

- **AIアバター**：HeyGen、Synthesia等のAIアバターで講師動画を作るケースがあります。実在社員のアバター化は本人の書面同意を取り、退職時の取り扱いも決めておきます。
- **多言語展開**：字幕翻訳、ElevenLabs等の吹き替え、YouTubeの自動吹き替え・多言語音声トラック機能を活用します。
- **レビュー体制**：法務・広報の確認工程をワークフローに組み込み、AI生成物の誤りが公開されないようにします。
- **データ取り扱い**：社外秘資料を外部AIサービスに入力する場合は、学習利用の有無（オプトアウト設定）や契約条件を確認します。

---

<a id="ch4"></a>
## 第4章 自動化の仕組み

この章では、エンジニア寄りのクリエイター向けに「コードとAIエージェントで制作工程をつなぐ」方法を解説します。全体像は次の通りです。

```text
┌──────────── 企画DB（Notion / スプレッドシート / Markdown）────────────┐
│                                                                     │
│  Claude Code / Cursor（AIエージェント）                              │
│    ├─ Skills: 台本作成・校閲・サムネ文言・メタデータ生成の手順書       │
│    ├─ MCP: YouTube Data/Analytics API, ffmpeg/動画編集, ファイル, Web │
│    └─ スクリプト: VOICEVOX/ElevenLabs合成, Remotionレンダリング       │
│                                                                     │
│  n8n / Make（定期実行・外部サービス連携・通知）                       │
└─────────────────────────────────────────────────────────────────────┘
          ↓ 出力: mp4, srt, サムネ画像, メタデータ(JSON) → YouTubeへ「非公開/下書き」でアップロード
          ↓ ★ 人間が確認して公開
```

**原則：自動化するのは「下書き（非公開アップロード）」まで。公開ボタンは人間が押す。** これにより、誤情報・権利侵害・不適切表現の公開を防げます。

<a id="ch4-1"></a>
### 4-1 Claude Code / Cursor + Skills

#### Skillsとは

Claude Codeの「Skills（Agent Skills）」は、特定タスクの手順・ルール・テンプレートをまとめたフォルダ（`SKILL.md` と補助ファイル）で、エージェントが必要に応じて読み込んで使う仕組みです。Cursorにも、ルール（`.cursor/rules`）やスキルに相当する仕組みがあります（最新の仕様は各公式ドキュメントで要確認）。

動画制作では、次のようなSkillを用意すると、毎回の指示が短くなり品質が安定します。

| Skill名（例） | 内容 |
|---|---|
| `yt-research` | リサーチ手順、一次情報の優先順位、出典の書き方 |
| `yt-script` | 台本の構成テンプレート、文体ルール、尺の計算式 |
| `yt-factcheck` | 数字・固有名詞・日付の確認手順、要確認タグの付け方 |
| `yt-voice` | VOICEVOX/ElevenLabsでの合成手順、辞書、話者ID |
| `yt-render` | Remotion/ffmpegでのレンダリング手順、出力仕様 |
| `yt-metadata` | タイトル・概要欄・タグ・チャプター・クレジット・AI開示の判断 |
| `yt-shorts` | 長尺→ショートの選定基準、フック作成ルール |

#### Skillの例（`.claude/skills/yt-script/SKILL.md`）

```markdown
---
name: yt-script
description: YouTube解説動画の台本を作成・推敲する。台本、構成、ナレーション原稿の依頼で使う。
---

# YouTube台本作成スキル

## 入力
- `projects/<slug>/research.md`（リサーチメモ。出典URL付き）
- `projects/<slug>/my_notes.md`（制作者自身の検証・意見。必須）

## 手順
1. research.md と my_notes.md を読み、論点を5つ以内に絞る
2. `templates/structure.md` の構成で `outline.md` を作る（各ブロックに想定秒数）
3. ユーザーに outline.md の承認を求める（承認前に本文を書かない）
4. 承認後 `script.md` を書く
   - 1文40字以内、話し言葉
   - 数字・固有名詞には出典の脚注を付ける。出典がないものは【要確認】
   - my_notes.md の内容を必ず1ブロック以上で使う
5. `yt-factcheck` スキルで自己校閲し、`factcheck.md` に結果を出力
6. TTS誤読リストを `reading.csv`（語,読み）に出力

## 禁止事項
- 医療・金融・法律について断定的な助言をしない
- 他チャンネルの台本の言い回しを流用しない
- 実在人物の発言を創作しない
```

#### プロジェクト構成の例

```text
youtube-factory/
├── .claude/
│   └── skills/
│       ├── yt-script/SKILL.md
│       ├── yt-factcheck/SKILL.md
│       ├── yt-render/SKILL.md
│       └── yt-metadata/SKILL.md
├── .mcp.json                 # プロジェクト用MCP設定
├── CLAUDE.md                 # チャンネル全体のルール（トーン、禁止事項、クレジット）
├── templates/
│   ├── structure.md
│   ├── description.md        # 概要欄テンプレ（クレジット・出典・AI開示メモ）
│   └── thumbnail_style.md
├── projects/
│   └── 2026-10-ai-editing/
│       ├── research.md
│       ├── my_notes.md
│       ├── script.md
│       ├── dialogue.csv
│       ├── build/            # 音声・字幕・中間ファイル
│       └── out/              # 完成mp4, srt, thumbnail.png, metadata.json
├── remotion/                 # Remotionプロジェクト
└── scripts/
    ├── voicevox_batch.py
    ├── make_srt.py
    └── upload_draft.py
```

#### CLAUDE.md の例（チャンネル憲法）

```markdown
# チャンネル運用ルール

- チャンネル: 「AIで仕事を速くする」解説（日本語、顔出しなし、ずんだもん＋めたん）
- 目的: 視聴者が動画を見た直後に1つ行動できること
- 必須: 各動画に「制作者が実際に試した結果」パートを入れる
- 概要欄に必ず入れる: VOICEVOX:ずんだもん / VOICEVOX:四国めたん / 出典リスト
- 写実的な生成映像を使ったら metadata.json の "ai_disclosure": true
- アップロードは必ず privacyStatus: "private"。公開は人間が行う
- 不明な情報は【要確認】と書き、推測で埋めない
```

#### エージェントへの指示例

```text
projects/2026-10-ai-editing で、yt-script スキルに従って outline.md を作って。
research.md の出典のうち、公式ドキュメント以外のものは信頼度を「中」として扱って。
outline ができたら止まって私に見せて。
```

```text
script.md が承認済み。次をやって:
1. dialogue.csv に変換（ずんだもん=質問役、めたん=解説役）
2. scripts/voicevox_batch.py で音声合成（reading.csv をユーザー辞書に登録してから）
3. scripts/make_srt.py で字幕生成
4. remotion で 1920x1080 / 30fps でレンダリング
5. out/ に mp4・srt・metadata.json を出力
途中でエラーが出たら、勝手に仕様を変えずに報告して。
```

<a id="ch4-2"></a>
### 4-2 MCP（YouTube Data API系・ffmpeg系・ファイル系）

MCP（Model Context Protocol）は、AIエージェントに外部ツールやデータを接続するための標準プロトコルです。Claude Code、Cursor、その他のMCP対応クライアントで共通して使えます。

#### 動画制作で使えるMCPの種類

| 種類 | 例 | できること |
|---|---|---|
| YouTube Data / Analytics API系 | GitHub上の `mrchevyceleb/youtube-mcp`、`itayshmool/shmool-youtube-mcp` など（コミュニティ製） | 動画一覧・詳細取得、アップロード、メタデータ更新、サムネ設定、プレイリスト、コメント、アナリティクス |
| ffmpeg／動画編集系 | `KyaniteLabs/mcp-video`（コミュニティ製） | トリム、結合、テキスト合成、音声追加、字幕焼き込み、シーン検出、Whisper文字起こし、Remotion連携など |
| ファイル系 | 公式リファレンス実装のFilesystem MCPサーバー | 指定ディレクトリ内のファイル読み書き（Claude Code/Cursorは標準でファイル操作可能なため、他クライアント向け） |
| Web検索・取得系 | Fetch MCP、各種検索API系 | リサーチ、出典確認 |
| ドキュメント／DB系 | Notion、Google Drive、スプレッドシート系MCP | 企画DB・台本管理 |
| 音声系 | ElevenLabsが提供するMCPサーバー（提供状況は要確認） | TTS生成 |

> **注意：コミュニティ製MCPの安全性。** 上記のYouTube系・動画系MCPは個人・小規模組織が公開しているもので、公式製品ではありません。OAuthのトークンを渡す前に、ソースコードを確認する、権限（スコープ）を最小限にする、削除系のアクションを無効化する、などの対策を取ってください。リポジトリの継続性も要確認です。

#### YouTube MCPの設定例（Claude Code `.mcp.json`）

`mrchevyceleb/youtube-mcp` のREADMEに記載された形式に基づく例です。事前にGoogle Cloud ConsoleでYouTube Data API v3とYouTube Analytics APIを有効化し、OAuth 2.0クライアントIDを作成します。

```json
{
  "mcpServers": {
    "youtube": {
      "command": "node",
      "args": ["/absolute/path/to/youtube-mcp/dist/index.js"],
      "env": {
        "GOOGLE_CLIENT_ID": "${GOOGLE_CLIENT_ID}",
        "GOOGLE_CLIENT_SECRET": "${GOOGLE_CLIENT_SECRET}"
      }
    },
    "video": {
      "command": "uvx",
      "args": ["mcp-video"]
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/home/me/youtube-factory"]
    }
  }
}
```

Claude CodeではCLIからも追加できます（`mcp-video` のREADMEに記載の例）。

```bash
claude mcp add mcp-video -- uvx mcp-video
claude mcp list
```

> 環境変数の展開記法（`${...}`）の対応はクライアントにより異なります。シークレットをリポジトリにコミットしないよう、`.mcp.json` を `.gitignore` に入れるか、ユーザースコープの設定に置いてください。

#### OAuthスコープの考え方

`shmool-youtube-mcp` のREADMEでは、`youtube`、`youtube.upload`、`youtube.readonly`、`yt-analytics.readonly` のスコープが挙げられています。運用では次のように分けると安全です。

- **分析専用エージェント**：`youtube.readonly` と `yt-analytics.readonly` のみ
- **アップロード用**：`youtube.upload`（下書きアップロード）を追加
- **フル管理**：`youtube`（削除も可能になるため常用しない）

なお、YouTube Data API v3にはプロジェクトごとの1日あたりのクォータ（割り当て）があり、動画アップロードは消費ユニットが大きい操作です。具体的な数値は公式ドキュメントで要確認です。また、未審査のAPIプロジェクトからアップロードした動画は非公開に制限される場合がある旨が公式に記載されていたことがあり、最新の監査要件は要確認です。

#### MCPを使ったエージェント指示例

```text
youtube MCPで自分のチャンネルの直近20本の動画を取得し、
各動画の views, CTR, averageViewPercentage を一覧にして analytics/2026-10-01.csv に保存。
上位5本と下位5本のタイトルの特徴を比較して、次の企画への示唆を3つ書いて。
書き込み系のアクション（更新・削除）は一切使わないこと。
```

```text
video MCPで out/main.mp4 から以下を作って:
- 0:00〜0:03 の無音部分をトリム
- build/subs.srt を焼き込んだ版を out/main_subbed.mp4 として出力
- 2:10〜2:55 を 9:16 にリフレームして out/short_01.mp4
処理後、それぞれの尺と解像度を報告して。
```

```text
youtube MCPで out/main.mp4 を privacyStatus=private でアップロードし、
metadata.json のタイトル・概要欄・タグ・カテゴリを設定、out/thumbnail.png をサムネに。
アップロード後の動画IDとStudioのURLだけ教えて。公開はしないこと。
```

<a id="ch4-3"></a>
### 4-3 Remotion / ffmpeg によるプログラマブル動画

#### Remotionとは

Remotionは、Reactコンポーネントで動画を記述し、MP4などにレンダリングできるフレームワークです。`useCurrentFrame()` で現在フレームを取得し、`interpolate()` や `spring()` でアニメーションを作ります。テロップ・図解・グラフ・ランキング・字幕アニメーションなど、**「型が決まっていて中身が変わる」映像**の自動生成に非常に向いています。

> ライセンス注意：Remotionは個人や小規模チームは無料で使える一方、一定規模以上の企業では有償の会社ライセンスが必要とされています。条件の詳細は公式サイトで要確認です。

#### セットアップ

```bash
npx create-video@latest   # テンプレートを選んでプロジェクト作成
cd my-video
npx remotion studio       # ブラウザでプレビュー
npx remotion render MyComp out/video.mp4
# 縦型（ショート）
npx remotion render ShortComp out/short.mp4 --width=1080 --height=1920
```

#### 字幕＋キャラ立ち絵の掛け合い動画コンポーネント例

```tsx
// remotion/src/Dialogue.tsx
import {AbsoluteFill, Audio, Img, Sequence, staticFile,
        useCurrentFrame, interpolate} from "remotion";

export type Line = {
  speaker: "zundamon" | "metan";
  text: string;
  audio: string;        // build/voice/0001_zundamon.wav
  durationInFrames: number;
};

const Subtitle: React.FC<{text: string; speaker: Line["speaker"]}> = ({text, speaker}) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 6], [0, 1], {extrapolateRight: "clamp"});
  const color = speaker === "zundamon" ? "#3a7d2c" : "#b03a7a";
  return (
    <div style={{
      position: "absolute", bottom: 60, left: 120, right: 120, opacity,
      fontSize: 54, fontWeight: 800, color: "#fff", textAlign: "center",
      WebkitTextStroke: `10px ${color}`, paintOrder: "stroke fill",
    }}>{text}</div>
  );
};

export const Dialogue: React.FC<{lines: Line[]}> = ({lines}) => {
  let from = 0;
  return (
    <AbsoluteFill style={{backgroundColor: "#f5f1e8"}}>
      <Img src={staticFile("bg/desk.png")} style={{width: "100%"}} />
      {lines.map((l, i) => {
        const seq = (
          <Sequence key={i} from={from} durationInFrames={l.durationInFrames}>
            <Audio src={staticFile(l.audio)} />
            <Img src={staticFile(`chara/${l.speaker}.png`)}
                 style={{position: "absolute", bottom: 0,
                         [l.speaker === "zundamon" ? "left" : "right"]: 0, height: 700}} />
            <Subtitle text={l.text} speaker={l.speaker} />
          </Sequence>
        );
        from += l.durationInFrames;
        return seq;
      })}
      <Audio src={staticFile("bgm/main.mp3")} volume={0.08} loop />
    </AbsoluteFill>
  );
};
```

```tsx
// remotion/src/Root.tsx（尺は lines.json から計算）
import {Composition} from "remotion";
import {Dialogue} from "./Dialogue";
import lines from "../public/lines.json";

const total = lines.reduce((s: number, l: any) => s + l.durationInFrames, 0);

export const RemotionRoot = () => (
  <Composition id="Main" component={Dialogue}
    durationInFrames={total} fps={30} width={1920} height={1080}
    defaultProps={{lines}} />
);
```

`lines.json` は、VOICEVOXで生成したwavの長さ（秒）×30でフレーム数を計算して作ります。

```python
# scripts/make_lines.py
import csv, json, wave, pathlib
FPS = 30
rows = list(csv.DictReader(open("dialogue.csv", encoding="utf-8")))
lines = []
for i, r in enumerate(rows):
    wav = pathlib.Path(f"build/voice/{i:04d}_{r['speaker']}.wav")
    with wave.open(str(wav)) as w:
        sec = w.getnframes() / w.getframerate()
    lines.append({"speaker": r["speaker"], "text": r["text"],
                  "audio": f"voice/{wav.name}",
                  "durationInFrames": int(sec * FPS) + 6})  # 0.2秒の間
json.dump(lines, open("remotion/public/lines.json", "w", encoding="utf-8"),
          ensure_ascii=False, indent=2)
```

#### SRT字幕の自動生成

```python
# scripts/make_srt.py
import json
FPS = 30
def ts(f):
    s = f / FPS
    return f"{int(s//3600):02d}:{int(s%3600//60):02d}:{int(s%60):02d},{int((s%1)*1000):03d}"
lines = json.load(open("remotion/public/lines.json", encoding="utf-8"))
t, out = 0, []
for i, l in enumerate(lines, 1):
    out.append(f"{i}\n{ts(t)} --> {ts(t + l['durationInFrames'] - 6)}\n{l['text']}\n")
    t += l["durationInFrames"]
open("out/subs.srt", "w", encoding="utf-8").write("\n".join(out))
```

#### ffmpegの定番レシピ

```bash
# 1) 無音区間の検出（-30dB以下が0.5秒以上）
ffmpeg -i raw.mp4 -af silencedetect=noise=-30dB:d=0.5 -f null - 2> silence.log

# 2) 字幕の焼き込み（日本語フォント指定）
ffmpeg -i main.mp4 -vf "subtitles=subs.srt:force_style='FontName=Noto Sans JP,FontSize=28,Outline=3'" \
  -c:a copy main_subbed.mp4

# 3) 16:9 → 9:16（中央クロップ）
ffmpeg -i main.mp4 -ss 00:02:10 -to 00:02:55 \
  -vf "crop=ih*9/16:ih,scale=1080:1920" -c:a aac short_01.mp4

# 4) BGMを-20dB程度でミックスし、ナレーション中は自動で下げる（サイドチェーン）
ffmpeg -i voice.wav -i bgm.mp3 -filter_complex \
  "[1:a]volume=0.3[b];[b][0:a]sidechaincompress=threshold=0.03:ratio=8[bd];[0:a][bd]amix=inputs=2:duration=first" \
  mixed.wav

# 5) ラウドネス正規化（配信向けに-14 LUFS前後を目安にする例）
ffmpeg -i mixed.wav -af loudnorm=I=-14:TP=-1.5:LRA=11 final.wav

# 6) 複数クリップの結合（同一コーデック）
printf "file '%s'\n" clip_*.mp4 > list.txt
ffmpeg -f concat -safe 0 -i list.txt -c copy joined.mp4

# 7) サムネ候補をシーンチェンジから抽出
ffmpeg -i main.mp4 -vf "select='gt(scene,0.4)',scale=1280:-1" -vsync vfr thumbs/%03d.jpg
```

#### Whisperでの文字起こし（顔出し・トーク系）

```bash
pip install faster-whisper
```

```python
from faster_whisper import WhisperModel
model = WhisperModel("large-v3", device="cuda", compute_type="float16")
segments, _ = model.transcribe("raw.mp4", language="ja", vad_filter=True)
with open("raw.srt", "w", encoding="utf-8") as f:
    for i, s in enumerate(segments, 1):
        def t(x): return f"{int(x//3600):02d}:{int(x%3600//60):02d}:{int(x%60):02d},{int(x%1*1000):03d}"
        f.write(f"{i}\n{t(s.start)} --> {t(s.end)}\n{s.text.strip()}\n\n")
```

モデル名・推奨設定は faster-whisper の最新READMEで要確認です。

#### 下書きアップロード（YouTube Data API、Python）

```python
# scripts/upload_draft.py
import json
from googleapiclient.discovery import build
from googleapiclient.http import MediaFileUpload
from google_auth_oauthlib.flow import InstalledAppFlow

SCOPES = ["https://www.googleapis.com/auth/youtube.upload"]
creds = InstalledAppFlow.from_client_secrets_file("client_secret.json", SCOPES).run_local_server(port=0)
yt = build("youtube", "v3", credentials=creds)
meta = json.load(open("out/metadata.json", encoding="utf-8"))

req = yt.videos().insert(
    part="snippet,status",
    body={
        "snippet": {"title": meta["title"], "description": meta["description"],
                    "tags": meta["tags"], "categoryId": "27"},  # 27=Education
        "status": {"privacyStatus": "private", "selfDeclaredMadeForKids": False},
    },
    media_body=MediaFileUpload("out/main.mp4", chunksize=-1, resumable=True),
)
res = None
while res is None:
    _, res = req.next_chunk()
print("uploaded:", res["id"])
```

> AI使用（合成コンテンツ）の開示をAPIで設定できるかどうか（`status` 配下の該当フィールドの有無）は、本レポート作成時点で一次情報を確認できていません（要確認）。確実なのはYouTube Studioのアップロード画面「属性」→「AI使用」で設定する方法です。下書きアップロード後、人間がStudioで開示設定と最終確認を行う運用にすると安全です。

<a id="ch4-4"></a>
### 4-4 n8n / Make によるワークフロー自動化

コードを書かずに（または少しだけ書いて）工程をつなぐなら、n8n（セルフホスト可能なワークフロー自動化ツール）やMake（旧Integromat）が便利です。

#### ワークフロー例1：企画ネタの自動収集（毎朝）

```text
[Schedule Trigger 毎朝7:00]
  → [RSS Read] 業界ニュース・公式ブログ数件
  → [HTTP Request] YouTube Data API search.list（キーワード、直近24時間、viewCount順）
  → [AI Agent / OpenAI / Anthropic ノード] 「チャンネルに合うネタか」を判定し、切り口を1行で
  → [Google Sheets] ネタ帳に追記（URL、要約、スコア、切り口）
  → [Slack / Discord] 上位3件を通知
```

LLMノードのプロンプト例：

```text
あなたは「AIで仕事を速くする」チャンネルの編集者です。
次のニュース/動画について JSON で返してください:
{"fit": 0-10, "angle": "当チャンネルならではの切り口(40字)", "need_check": ["確認が必要な事実"]}
- 一次情報が確認できないものは fit を3以下に
- 他チャンネルの動画をなぞるだけの切り口は不可
入力: {{ $json.title }} / {{ $json.description }} / {{ $json.link }}
```

#### ワークフロー例2：台本承認 → 音声・動画の自動生成

```text
[Google Sheets Trigger] status列が「台本承認」に変わった行
  → [Google Docs] 台本テキスト取得
  → [Code(JS)] セリフ単位に分割
  → [HTTP Request] ElevenLabs TTS（またはローカルのVOICEVOXエンジン）
  → [Execute Command / SSH] レンダリングサーバーで `npx remotion render ...`
  → [Google Drive] mp4・srtを保存
  → [Sheets] status を「レビュー待ち」に更新、DriveのURLを記入
  → [Slack] 担当者にレビュー依頼
```

n8nのExecute Commandノードはセルフホスト環境で使えるノードです（クラウド版での可否は要確認）。重いレンダリングは別サーバーに切り出し、Webhookで完了通知を受ける構成が安定します。

#### n8n Codeノードの例（セリフ分割）

```javascript
// 入力: $json.script（「ずんだもん：…」「めたん：…」形式）
const lines = $json.script.split(/\n+/).filter(Boolean);
return lines.map((line, i) => {
  const m = line.match(/^(ずんだもん|めたん)[：:](.+)$/);
  if (!m) return null;
  return { json: {
    index: i,
    speaker: m[1] === "ずんだもん" ? 3 : 2,   // VOICEVOXスタイルID（要確認）
    text: m[2].trim(),
  }};
}).filter(Boolean);
```

#### ワークフロー例3：公開後の分析レポート（毎週）

```text
[Schedule 毎週月曜9:00]
  → [HTTP Request] YouTube Analytics API reports.query
       metrics=views,estimatedMinutesWatched,averageViewPercentage,subscribersGained
       dimensions=video, startDate=7日前, endDate=昨日
  → [AI] 週次サマリーと「次に試す1つの仮説」
  → [Notion] 週次レポートページ作成
  → [Slack] 要約を投稿
```

#### Makeを使う場合

Makeでも同様に、Google Sheets・HTTP・OpenAI/Anthropic・YouTube（Makeの公式YouTubeモジュール）・Google Driveのモジュールをつないで構成できます。YouTubeモジュールで可能な操作の範囲（アップロード、メタデータ更新など）は最新の公式ドキュメントで要確認です。

#### 自動化ツール選びの目安

| 観点 | n8n | Make | Claude Code / Cursor＋スクリプト |
|---|---|---|---|
| 学習コスト | 中 | 低〜中 | 中〜高（コードが読めると強い） |
| セルフホスト | 可能 | 不可（SaaS） | ローカル実行 |
| 重い処理（レンダリング） | 外部サーバー連携 | 外部サーバー連携 | ローカルで直接 |
| 柔軟性 | 高 | 中 | 最高 |
| 向いている用途 | 定期実行・通知・DB連携 | 素早い連携・非エンジニア | 制作そのものの自動化 |

実務では「Claude Code／Cursorで制作パイプラインを作り、n8n／Makeで定期実行と通知を担う」という併用が扱いやすい構成です。

<a id="ch4-5"></a>
### 4-5 プロンプト集

#### タイトル案

```text
動画の内容（下記）から、YouTubeタイトルを15案。
- 全角32文字以内、検索キーワード「{kw}」を前半に
- 誇大表現・釣り表現（内容にない約束）は禁止
- 3案ずつ「疑問形」「数字」「ベネフィット」「意外性」「比較」の型で
[内容要約]
```

#### 概要欄

```text
以下のテンプレートを埋めて概要欄を作成。
---
{1〜2文の要約}

▼目次
{チャプター（0:00から）}

▼参考資料
{script.mdの出典を箇条書き。URLは原文のまま、作らない}

▼使用素材・クレジット
VOICEVOX:ずんだもん / VOICEVOX:四国めたん
{BGM・素材のクレジット}

▼制作について
本動画は台本作成・音声合成・一部映像の生成にAIツールを使用しています。
内容は制作者が確認・検証しています。
---
```

#### ショートのフック

```text
次の30秒台本の冒頭1.5秒で言う「フック」を10案。
- 12文字以内
- 結論の先出し / 視聴者の誤解の指摘 / 数字 の3型
- 本編で回収しない約束はしない
```

#### 品質ゲート（公開前のAIセルフチェック）

```text
あなたはYouTubeポリシー担当のレビュアーです。以下の台本・メタデータ・使用素材リストを確認し、
1) 写実的なAI生成/改変の有無 → 「AI使用」開示が必要か
2) 非オリジナルコンテンツ（テンプレート的、反復的、AIペルソナがセンシティブ分野で助言）に該当する恐れ
3) 再利用コンテンツ（他者素材を大きな改変なしに使用）の恐れ
4) 著作権・音声ライセンス・クレジット表記の漏れ
5) 誤解を招くタイトル・サムネ
を「OK/要修正/要人間判断」で表にしてください。最終判断は人間が行う前提です。
```

---

<a id="ch5"></a>
## 第5章 YouTubeのAIコンテンツ方針と法的注意点

> 本章はポリシーの要約であり、法的助言ではありません。最新の正式な規定はYouTubeヘルプセンターで必ず確認してください。

<a id="ch5-1"></a>
### 5-1 合成コンテンツ（AI使用）の開示

#### 基本ルール

YouTubeヘルプ「Disclosing use of GenAI content」によると、**写実的に見えるコンテンツをAIで意味のある形で改変・生成した場合、クリエイターは開示が必要**です。

- 開示方法：YouTube Studioでアップロードする際、「属性（Attributes）」の「AI使用（AI use）」で「はい」を選択。
- 表示：開示すると動画にラベルが付きます。2026年の改定で、写実的なAIコンテンツのラベルは**長尺ではプレーヤー直下（概要欄の上）**、**ショートでは動画上のオーバーレイ**に表示される形式に統一されました（YouTube公式ブログ「Improving AI labels for viewers and creators」）。写実的でない、アニメ調、または軽微な改変の場合は、概要欄の展開部分に表示されます。

#### 開示が必要な例・不要な例（一般的な整理）

| 開示が必要になりやすい | 一般に不要とされやすい |
|---|---|
| 実在人物が言っていないことを言っているように見せる（顔・声の合成） | 台本作成・アイデア出しにAIを使った |
| 実在の場所・出来事の映像を改変する | 自動字幕・文字起こし |
| 写実的な架空の出来事・人物・場所を生成する | 色補正、ノイズ除去、美肌などの軽微な補正 |
| 本物と見分けにくい生成映像をBロールに使う | 明らかに非現実的なアニメ・ファンタジー表現 |

境界事例は多いため、**迷ったら開示する**のが実務上の推奨です。

#### 自動ラベル（2026年5月〜）

YouTubeは2026年5月から、クリエイターが開示を指定していなくても、システムが**大きな写実的AI使用を検出した場合に自動でラベルを付与**する運用を始めました。誤判定の場合は多くのケースでStudioから修正できますが、次の場合はラベルが恒久的に残ります。

- YouTube自身のAIツール（VeoやDream Screenなど）で作ったコンテンツ
- 完全生成AIであることを示すC2PA（Content Credentials）メタデータを含むコンテンツ

また、YouTube公式のRene Ritchie氏は解説動画で「ラベル自体は、おすすめ表示や収益化に影響しない」と述べています。一方で、ヘルプページには、**継続的に開示しないクリエイターには、ラベルの手動付与やコンテンツ削除、YPP停止などのペナルティが科される可能性**がある旨が記載されています。

#### 実務上のポイント

- 動画生成AIの出力にはC2PAメタデータや電子透かし（Googleの場合はSynthID等）が含まれている場合があります。再エンコードしても検出される可能性があるため、「隠せばバレない」という考えは捨てましょう。
- 概要欄に「制作にAIを使用しています」と書くことは、Studioでの開示の代わりにはなりません。両方行うのが丁寧です。

<a id="ch5-2"></a>
### 5-2 収益化ポリシー：非オリジナル（量産型）／再利用コンテンツ

YouTubeパートナープログラム（YPP）の「チャンネル収益化ポリシー」には、AI活用チャンネルに直接関係する2つのポリシーがあります。

#### (1) 非オリジナルコンテンツ（inauthentic content）

- 2025年7月15日、「反復的なコンテンツ（repetitious content）」から改称され、**大量生産・反復的なコンテンツを含む**ことが明確化されました。YouTubeはこれを「従来から収益化対象外であったものの明確化」と説明しています。
- 収益化するコンテンツは「自分のオリジナル作品であること」「大量生産・汎用的・反復的・操作的でないこと（視聴回数を得るためだけでなく、視聴者の楽しみや学びのために作られていること）」が求められます。
- **2026年7月の明確化（3分類）**（2026年7月16日に展開と報道）：
  1. **汎用的・反復的なコンテンツ**：テンプレートで作られたように見える、同じチャンネルの動画を続けて見ると繰り返しに感じる。例として、教育的価値や解説がほとんどなく動画間の変化が乏しいもの、同じ状況に同じキャラクターを置いて同じ結末を繰り返すもの、ナレーション・解説がほとんどない画像スライドショーや定型ストーリー・スクロールテキスト、「制作者独自の洞察や視点を加えず、汎用的なテンプレートで大量生産された印象を与えるAI生成コンテンツ」などが挙げられています。既に広く出回っている内容をなぞるだけのチュートリアルも該当しうると説明されています。
  2. **不快・扇情的なコンテンツ（unsatisfying / off-putting）**：視聴回数を得るためだけに感情を操作する、不快にさせる内容（例：動物を苦境に置いてから救出する演出）。
  3. **センシティブな話題を扱うAIペルソナ**：健康、法律、金融、政治などについて、人間の専門家のように見せかけて助言するAI生成のペルソナ。
- 判定は**チャンネル全体**で行われ、AI・CGIなどの使用有無にかかわらず適用されると説明されています。YPPから外れた場合、**21日以内に異議申し立て**、**90日後に再申請**が可能と報じられています（最新の手続きはヘルプで要確認）。

#### (2) 再利用されたコンテンツ（reused content）

- 他者のコンテンツ（他チャンネルの動画、テレビ番組、映画、他SNSの投稿など）を、**大きな改変・独自の解説や付加価値なしに**再アップロードしたものが中心のチャンネルは収益化できません。
- 解説、批評、パロディ、教育目的で大きく編集・付加価値を加えたものは対象外になりうるとされていますが、著作権法上の適法性とは別問題です。
- 切り抜きチャンネルは、権利者のガイドラインに従ったうえで、**独自の編集・解説・構成**を加えることが前提です。

#### AI活用チャンネルが取るべき対策

| リスク | 対策 |
|---|---|
| テンプレート的 | フォーマットは固定しても、論点・図解・検証・演出は毎回変える。シリーズごとに構成を見直す |
| 独自性不足 | 「制作者が実際に試した」「独自に集計した」「視聴者の質問に答えた」パートを必ず入れる |
| AIペルソナ | センシティブな分野では架空の専門家キャラを使わない。キャラクターを使う場合も「解説キャラ」であることを明示し、出典を示す |
| スライドショー化 | ナレーションで解説し、映像は説明の役に立つものにする |
| 再利用 | 他者素材は権利処理済みのもの、または引用要件を満たす範囲に限定 |
| 投稿頻度の過剰 | 品質を維持できる頻度に抑える。短期間に似た動画を大量投稿しない |

<a id="ch5-3"></a>
### 5-3 著作権・音声ライセンスの注意

#### 生成AIと著作権（日本）

- 日本の著作権法第30条の4はAI学習（情報解析）について広く利用を認めていますが、**生成・利用段階**では通常の著作権侵害の判断（類似性・依拠性）が適用される、というのが文化庁の「AIと著作権に関する考え方について」（2024年）の整理です。最新の見解は文化庁サイトで要確認です。
- **既存キャラクター・作品に似せた生成**（「〇〇風」「特定作家の画風」での出力をそのまま使う）は、類似性・依拠性が認められると侵害になりえます。
- **AI生成物の著作物性**：人間の創作的寄与が乏しいAI生成物は著作物と認められない可能性があり、他者に無断使用されても保護を主張しにくい点にも注意。

#### 音声ライセンスのチェックポイント

1. **ソフトウェアのライセンスとキャラクター（音声ライブラリ）の規約は別。** VOICEVOX自体は商用可でも、各キャラクターの規約に従う必要があります。
2. **クレジット表記の要否と形式。** 例：ずんだもんは「VOICEVOX:ずんだもん」。クレジットなし商用は別契約（1キャラクター40万円＋税）。
3. **キャラクターのイメージを損なう利用の禁止。** 多くのキャラクター規約には、公序良俗に反する利用、政治・宗教的利用、特定の個人・団体の誹謗中傷などの禁止条項があります。各規約を個別に確認してください。
4. **サービス終了時の扱い。** にじボイスのようにサービスが終了する場合、過去に生成した音声の利用継続条件を確認し、記録（規約のスクリーンショット・利用日）を残しておきます。
5. **声のクローンは本人の同意が前提。** 他人の声を無断でクローンすることは、パブリシティ権・名誉毀損・不正競争などのリスクがあります。日本では声の権利保護を求める動き（日本俳優連合の活動など）も活発です。
6. **BGM・効果音。** YouTubeオーディオライブラリ、商用可のフリー素材、AI音楽生成サービス（Suno、Udio、ElevenLabs Musicなど）は、それぞれ商用条件・クレジット要否・Content IDとの関係が異なります（要確認）。AI生成音楽を使った場合でも、他者のContent ID登録曲に類似して申し立てを受ける可能性はゼロではありません。

#### 素材管理表のテンプレート

```csv
asset_id,type,source,license,commercial_ok,credit_required,credit_text,url_or_tool,checked_date,note
V001,voice,VOICEVOX,キャラ規約,yes,yes,VOICEVOX:ずんだもん,https://zunko.jp/con_ongen_kiyaku.html,2026-10-01,
B001,video,Veo 3.1,Google利用規約,要確認,no,,Gemini/Flow,2026-10-01,写実的→AI開示
M001,music,YouTubeオーディオライブラリ,ライブラリ条件,yes,曲による,,YouTube Studio,2026-10-01,
```

---

<a id="ch6"></a>
## 第6章 情報サイト・発信者（日本語／英語）

ここでは、本レポート作成時に実在・内容を確認できたもの、または広く知られた公式情報源に絞って掲載します。個人の発信者は活動状況が変わりやすいため、フォロー前に最新の発信内容を確認してください。

### 公式・一次情報

| 名称 | 内容 | URL |
|---|---|---|
| YouTubeヘルプ：チャンネル収益化ポリシー | 非オリジナル・再利用コンテンツの公式定義 | https://support.google.com/youtube/answer/1311392 |
| YouTubeヘルプ：GenAIコンテンツの開示 | AI使用開示の手順と対象 | https://support.google.com/youtube/answer/14328491 |
| YouTubeヘルプ：「コンテンツの作成方法」の表示 | ラベル・C2PAの扱い | https://support.google.com/youtube/answer/15447836 |
| YouTube公式ブログ（AIラベル改善の記事） | 2026年のラベル表示変更・自動検出 | https://blog.youtube/news-and-events/improving-ai-labels-viewers-creators/ |
| Creator Insider（YouTube公式の制作者向けチャンネル） | ポリシー解説。Rene Ritchie氏（Creator Liaison）が出演 | チャンネルURLは要確認 |
| YouTube Data API ドキュメント | API仕様・クォータ | https://developers.google.com/youtube/v3 |
| ElevenLabs ドキュメント（モデル一覧） | Eleven v3等の対応言語 | https://elevenlabs.io/docs/overview/models |
| VOICEVOX 音声モデル利用規約 | キャラごとの規約へのリンク | https://github.com/VOICEVOX/voicevox_vvm |
| 東北ずん子・ずんだもん 音源利用ガイドライン | クレジット・商用条件 | https://zunko.jp/con_ongen_kiyaku.html |
| OpusClip 公式 | 機能・対応言語 | https://www.opus.pro/ |
| Remotion 公式 | ドキュメント・ライセンス | https://www.remotion.dev/ |
| n8n 公式ドキュメント | ノード・セルフホスト | https://docs.n8n.io/ |
| 文化庁「AIと著作権」 | 日本の著作権の考え方 | 文化庁サイト内（個別URLは要確認） |

### ニュース・解説メディア

| 名称 | 言語 | 特徴 |
|---|---|---|
| ITmedia AI＋ | 日 | 国内AIニュース。にじボイス終了などを報道 |
| Tubefilter | 英 | YouTube・クリエイター業界ニュース。2026年7月のポリシー明確化を詳報 |
| TechCrunch | 英 | テック全般。YouTubeのAIスロップ対策を報道 |
| NetInfluencer | 英 | クリエイター経済ニュース |
| Mashable | 英 | YouTubeのAIスロップ収益化ポリシーの解説記事あり |
| Uravation（株式会社Uravationのメディア） | 日 | Opus Clipの日本語解説（2026年9月更新）など |

### コミュニティ・リポジトリ

| 名称 | 内容 |
|---|---|
| GitHub: mrchevyceleb/youtube-mcp | YouTube管理用MCPサーバー（コミュニティ製） |
| GitHub: itayshmool/shmool-youtube-mcp | YouTube Data/Analytics APIのMCPラッパー（コミュニティ製） |
| GitHub: KyaniteLabs/mcp-video | ffmpeg＋Remotionの動画編集MCP（コミュニティ製） |
| GitHub: VOICEVOX | VOICEVOX本体・エンジン・音声モデル |

### 発信者について

- **Rene Ritchie（英）**：YouTubeのCreator Liaison。ポリシーやラベル変更を公式に解説しています（本レポートで内容を確認）。
- **ヒホ（廣芝和之）氏（日）**：VOICEVOXの開発者として知られています。開発・規約関連の情報はVOICEVOX公式サイト／GitHubで確認するのが確実です（個人SNSのURLは要確認）。
- その他、日本語のAI動画制作系YouTuber・X発信者は多数いますが、活動状況や情報の正確性にばらつきがあるため、本レポートでは個人名の推薦は控えます。選ぶ際は「出典を示しているか」「ツールの宣伝（アフィリエイト）に偏っていないか」「ポリシーや規約に言及しているか」を基準にしてください。

---

<a id="ch7"></a>
## 第7章 チェックリスト

### 7-1 導入前チェック

- [ ] 自分の制作工程で一番時間がかかっている工程を計測した
- [ ] チャンネルの「独自の価値」（体験・専門性・検証・視点）を1文で言える
- [ ] 使うAIツールの商用利用条件・クレジット要否を確認した
- [ ] 社外秘・個人情報をAIに入力する場合のルールを決めた
- [ ] APIキー・OAuthトークンの保管場所（.env、シークレット管理）を決めた

### 7-2 制作ごとのチェック（公開前）

**内容**
- [ ] 台本の数字・固有名詞・日付・引用を一次情報で確認した
- [ ] 「要確認」タグが残っていない
- [ ] 制作者自身の検証・意見・体験が含まれている
- [ ] 医療・金融・法律・政治で断定的な助言をしていない／AIペルソナに語らせていない

**権利**
- [ ] 音声キャラのクレジットを概要欄（または動画内）に記載した
- [ ] 立ち絵・BGM・効果音・ストック映像のライセンスを素材管理表に記録した
- [ ] 他者の動画・画像・音声を、許諾または適法な範囲でのみ使用した
- [ ] 実在人物の顔・声を無断で合成していない

**YouTubeポリシー**
- [ ] 写実的なAI生成・改変があれば、Studioで「AI使用」を「はい」にした
- [ ] 直近の動画と並べて「テンプレートの差し替え」に見えない
- [ ] タイトル・サムネが内容と一致している（誇大・釣りでない）
- [ ] ショートの場合、単体で価値が完結している

**技術**
- [ ] 音量（ラウドネス）が適切、BGMがナレーションを邪魔していない
- [ ] 字幕の誤字・タイミングを確認した
- [ ] TTSの誤読を確認した
- [ ] チャプター（0:00始まり）を設定した

### 7-3 自動化のチェック

- [ ] 自動化は「非公開アップロード」までで止め、公開は人間が行う
- [ ] MCP・APIのスコープは最小限（削除権限を常用しない）
- [ ] コミュニティ製MCPのソースコードを確認した
- [ ] APIクォータ超過時のリトライ・通知を設定した
- [ ] 生成物・中間ファイルのバックアップがある
- [ ] エージェントが仕様を勝手に変えないよう、指示に「エラー時は報告」と明記した

### 7-4 月次の振り返り

- [ ] 視聴維持率・CTRの上位／下位動画を比較した
- [ ] 1本あたりの制作時間とAIツール費用を記録した
- [ ] YouTubeのポリシー・ツールの規約変更を確認した（ヘルプセンター、公式ブログ）
- [ ] プロンプト・Skill・テンプレートを改善した

---

<a id="ch8"></a>
## 第8章 導入ロードマップ

一気に全自動化を目指すと、品質低下・ポリシー違反・ツール費用の膨張が起きがちです。次の4段階で、効果を確かめながら進めることを推奨します。期間ではなく「完了条件」で次の段階に進みます。

### ステージ1：補助AIの導入（既存ワークフローを崩さない）

- **やること**：台本の構成案・校閲、自動字幕（Vrew／Premiere／DaVinci）、チャプター・概要欄の下書きをAIに任せる
- **ツール**：ChatGPT／Claude／Gemini、Vrew、YouTube Studio
- **完了条件**：1本あたりの制作時間を計測し、短縮効果を確認できた。台本の事実確認フローが定着した

### ステージ2：素材・音声の置き換え

- **やること**：ナレーションのTTS化（または自分の声の音質改善）、Bロールの画像・動画生成、サムネ素材の生成
- **ツール**：ElevenLabs／VOICEVOX、Veo 3.1／Kling／Runway／Hailuo、画像生成AI、Canva／Photoshop
- **完了条件**：素材管理表とクレジットのテンプレートができ、AI使用開示の判断基準をチームで共有できた

### ステージ3：半自動パイプライン化

- **やること**：Claude Code／CursorにSkillsとMCPを設定し、「台本→音声→字幕→レンダリング→非公開アップロード」をスクリプト化。ショート切り出しを仕組み化
- **ツール**：Claude Code／Cursor、Remotion、ffmpeg、YouTube Data API（MCP）、Opus Clip
- **完了条件**：台本承認から非公開アップロードまで、人手の作業が「確認」だけになった。エラー時の手順が文書化された

### ステージ4：データ駆動の改善ループ

- **やること**：n8n／Makeで分析レポートを定期実行し、仮説→検証のサイクルを回す。タイトル・サムネのA/Bテストを常時運用
- **ツール**：YouTube Analytics API、vidIQ／TubeBuddy、n8n／Make、Notion／スプレッドシート
- **完了条件**：毎週1つの仮説を検証し、テンプレート・Skillに反映する仕組みが回っている

### よくある失敗と回避策

| 失敗 | 回避策 |
|---|---|
| 全工程をAI任せにして動画が似通い、収益化審査で落ちる | 「独自パート」を必須項目としてテンプレート化する |
| 誤情報の公開でコメント欄が荒れる | 事実確認ステップと「要確認」タグ運用 |
| ツールを増やしすぎて費用が膨らむ | 工程ごとに1ツールに絞り、月次で費用対効果を確認 |
| サービス終了・規約変更で過去動画が問題に | 規約のスクリーンショットと利用日を記録、代替ツールを把握 |
| 自動アップロードで誤って公開 | privacyStatusは常にprivate、公開は人間 |

---

<a id="appendix"></a>
## 付録 参考URL一覧と要確認事項

### 本レポートで参照した主な情報源（2026年10月1日時点で確認）

- YouTubeヘルプ「YouTube channel monetization policies」 https://support.google.com/youtube/answer/1311392
- YouTubeヘルプ「Disclosing use of GenAI content」 https://support.google.com/youtube/answer/14328491
- YouTubeヘルプ「Understanding ‘How this content was made’ disclosures」 https://support.google.com/youtube/answer/15447836
- YouTube Blog「Improving AI labels for viewers and creators」 https://blog.youtube/news-and-events/improving-ai-labels-viewers-creators/
- YouTube動画「New! Simplified AI Labels & Auto-Detection」 https://www.youtube.com/watch?v=r99O5TAKM1E
- TechCrunch（2026年7月20日） https://techcrunch.com/2026/07/20/youtube-clarifies-policies-around-ai-slop-and-upsetting-videos/
- Tubefilter（2026年7月13日） https://www.tubefilter.com/2026/07/13/youtube-inauthentic-content-monetization-policy-update/
- NetInfluencer https://www.netinfluencer.com/youtube-trust-and-safety-chief-details-three-categories-of-content-barred-from-monetization/
- Mashable https://mashable.com/life/youtube-ai-slop-monetization-policy
- ITmedia AI＋（2025年11月21日、にじボイス終了） https://www.itmedia.co.jp/aiplus/article/2511/21/1251121121/
- Algomatic「『にじボイス』一部キャラクターの提供終了に関するお知らせ」 https://algomatic.jp/news/notice_nijivoice_20251111
- VOICEVOX 音声モデル利用規約 https://github.com/VOICEVOX/voicevox_vvm
- 東北ずん子 音源利用ガイドライン https://zunko.jp/con_ongen_kiyaku.html
- ElevenLabs Models https://elevenlabs.io/docs/overview/models
- ElevenLabs Blog「Eleven v3」 https://elevenlabs.io/blog/eleven-v3
- OpusClip https://www.opus.pro/
- Uravation「Opus Clipとは？」（2026年9月） https://uravation.com/media/opus-clip-complete-guide-2026/
- 動画生成AI比較：https://pixflow.net/blog/best-ai-video-generator/ 、https://www.imagine.art/insights/best-ai-video-generation-models 、https://melies.co/compare/ai-video-models 、https://www.dreamega.ai/blog/best-ai-video-models-2026
- MCP：https://github.com/mrchevyceleb/youtube-mcp 、https://github.com/itayshmool/shmool-youtube-mcp 、https://github.com/KyaniteLabs/mcp-video

### 要確認事項の一覧

| 項目 | 状況 |
|---|---|
| Soraアプリ終了（2026年4月26日）・API終了（2026年9月24日） | 複数の比較記事で報道。OpenAI公式発表の原文は未確認 |
| にじボイス終了後の既存生成音声の利用条件 | 未確認 |
| AquesTalk（ゆっくり）の商用ライセンス最新条件 | 未確認 |
| YouTube Data APIでAI使用開示を設定できるか | 未確認（Studioでの設定を推奨） |
| YouTube Data APIのクォータ値・アップロード時の監査要件 | 未確認 |
| YouTube Studioのリサーチ／インスピレーション機能、タイトルA/Bテストの日本での提供範囲 | 未確認 |
| 各ツールの料金・商用条件（ElevenLabs、Midjourney、Runway、Kling等） | 頻繁に変わるため都度確認 |
| Remotionの会社ライセンスの条件 | 公式サイトで確認が必要 |
| YouTubeの肖像（声を含む）検出機能の提供範囲 | 報道ベース、詳細未確認 |
| Creator InsiderチャンネルのURL、文化庁資料の個別URL | 未確認 |

---

*本レポートは2026年10月1日時点の公開情報に基づいて作成しました。AIツールとプラットフォームの方針は変化が速いため、定期的な見直しを推奨します。*

