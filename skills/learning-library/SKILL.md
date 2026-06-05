---
name: learning-library
description: >-
  個人の技術学習サイト「学習書架（My Learning Library）」を運用するスキル。ユーザーが
  「/create [書籍名・学習内容]」と送ったら新しい学習記事(単一HTML)を生成して pages/<棚>/ に置き、
  トップページ index.html の ARTICLES 配列に1件追記し catalog.json を更新する。「/update [識別ワード
  または file名] [変更内容]」と送ったら、catalog.json で対象記事を一意に特定し、変更範囲を先に確認した
  うえで該当箇所だけを外科的に編集する。「/org」と送ったら pages/ を棚別サブディレクトリに整頓し
  catalog.json（検索用の派生インデックス）を再生成して index.html の参照パスを揃える。「/publish」と送ったら
  変更を commit して push し公開サイトに反映する（git リポジトリのある環境専用）。技術書・技術トピックの
  学習記事を作る／既存記事を直す／学習ハブ(index)を更新する、GitHub Pages 用の自己完結HTMLを
  作る、といった依頼では、明示的に「スキルを使って」と言われなくても必ずこのスキルを使うこと。
  「学習記事を作って」「この本でまとめて」「記事を直して」「ハブに追加して」等も対象。
---

# 学習書架（Learning Library）運用スキル

個人の技術学習サイトを育てるためのスキル。記事は単一HTML（自己完結）、目録は `index.html` の
`ARTICLES` 配列。GitHub Pages にそのまま置ける相対リンク構成。

> **重要（状態の前提）**: Claude はチャットをまたいでファイルを記憶しない。`/create`・`/update`
> 実行時は、ユーザーに**最新の index.html（および更新対象の記事HTML）をアップロードしてもらい**、
> それを「現在の正」として編集する。アップロードが無ければ、まず依頼すること。

## ディレクトリ構成

```
/
├── index.html              トップページ（学習ハブ）。蔵書データ = ARTICLES 配列【真実】
├── catalog.json            検索用カタログ【派生】。/org が ARTICLES から再生成。手で編集しない
├── README.md
├── .nojekyll               GitHub Pages で素のまま配信させる空ファイル
├── skills/learning-library/SKILL.md   このスキル
└── pages/                  記事HTML（1記事 = 自己完結HTML）。棚(shelf)別サブディレクトリに整頓
    ├── db/        <slug>.html      データの土台
    ├── backend/   <slug>.html      バックエンド & API
    ├── design/    <slug>.html      ソフトウェア設計
    ├── infra/     <slug>.html      クラウド & インフラ
    ├── dist/      <slug>.html      分散データシステム
    └── tools/     <slug>.html      開発ツール

    重厚なトピックは「シリーズ」として複数HTMLに分割してよい（→「シリーズ記事」節）:
    pages/<shelf>/<series>-index.html  目次ページ（入口）
    pages/<shelf>/<series>-vol1.html … -volN.html  各巻
```

リンクはすべて相対パス（`pages/<shelf>/<slug>.html`）。ルート公開でもプロジェクト公開でも動く。
記事は自己完結（内部アンカー＋外部リンクのみ、記事間の相互リンクなし）なので、棚をまたぐ移動でも壊れない。
**例外**: シリーズ記事は「シリーズ単位で自己完結」とし、同一シリーズの兄弟・目次間だけは相互リンク可（→「シリーズ記事」節）。
shelf は SHELVES の id（`db`/`backend`/`design`/`infra`/`dist`/`tools`）と一致させる。

---

## コマンド: `/create [書籍名 または 学習内容] [--push]`

1. **レベル確認（必要なときだけ）**: トピックの難度・対象が曖昧なら、着手前に選択式で2つ確認する。
   自明な場合は聞かずに進めてよい。
   - 現在のレベル: ①ほぼ初めて ②基礎はある ③実務でそこそこ ④熟練（教えられる）
   - 目標レベル: ①全体像をつかむ ②実務で使える ③仕組みまで深く ④専門家レベルで応用
2. **裏取り**: URLがあれば web_fetch、現在の事実（価格・最新仕様・役職等）は web_search で確認。
3. **frontend-design スキルを読む**（`/mnt/skills/public/frontend-design/SKILL.md`）。
4. **棚を決める**: トピックに合う shelf を SHELVES から選ぶ（無ければ新設）。
5. **記事を生成**: 下記「記事デザイン規約」に従い、`pages/<shelf>/<slug>.html` に自己完結HTMLを作成。
   重厚で独立した大テーマが複数あるトピックは、単一HTMLにせず**シリーズ（複数HTML）に分割**してよい
   （→「シリーズ記事」節。目次ページ＋各巻＋代表カード1枚の構成にする）。
6. **ハブに追記**: アップロードされた `index.html` の `ARTICLES:START 〜 ARTICLES:END` の間に、
   新しいオブジェクトを**1件だけ**追記（`file:"pages/<shelf>/<slug>.html"`）。既存要素・他の箇所は変更しない。
   新分野なら `SHELVES` にも棚を1つ追加。`st-upd`（最終更新月）は更新してよい。
7. **catalog.json を更新**: 新記事のエントリ（slug/file/shelf/title/book/cat/tags/aliases）を1件追記。
   `aliases` には slug に加え、書名の略称や日本語ニックネーム（例: 「SQL指南書」）を入れて後で言葉で呼べるようにする。
8. **提示**: `pages/<shelf>/<slug>.html`・更新済み `index.html`・`catalog.json` を present_files で出す。
9. 引数に `--push` があれば、続けて `/publish`（commit & push）まで実行する。

### slug 命名
英小文字＋ハイフン。例: `refactoring.html`, `tcp-ip.html`。シリーズ物は `name-vol1.html`。
配置は `pages/<shelf>/<slug>.html`。

### ARTICLES 1件テンプレ（index.html に貼る形）
```js
{file:"pages/<shelf>/<slug>.html", shelf:"<SHELVESのid>", accent:"#RRGGBB", cat:"CATEGORY",
 title:"表示タイトル", book:"書名/出典", levelFrom:"初級", levelTo:"中級",
 desc:"1〜2文の要約。",
 tags:["タグ1","タグ2","タグ3"],
 vol:"全N巻 第X巻"}   // ← シリーズでなければ vol は省略
```
既存 shelf id: `db`(データの土台) / `backend`(バックエンド&API) / `design`(ソフトウェア設計) /
`infra`(クラウド&インフラ) / `dist`(分散データシステム) / `tools`(開発ツール)。
新分野が必要なら SHELVES に `{id, label, sub}` を追加する。

---

## コマンド: `/update [識別ワード または file名] [変更内容] [--push]`

第1引数は**正確なファイル名でなくてよい**。記事内容を一意に特定できる言葉（書名の略称・分野・
日本語ニックネーム等）でも対象を解決する。例: `/update sql指南書 〇〇に変更して` → `sql-mastery` を特定。

意図しない箇所を変えないことが最優先。

1. **対象を解決（catalog.json で名寄せ）**: 第1引数を `catalog.json` の各記事に対して照合する。
   照合キー: `slug` / `file` / `title` / `book` / `cat` / `tags` / `aliases`（部分一致・大文字小文字や
   全半角の揺れは吸収する）。
   - **一意に1件** → その記事を対象として確定。どのファイルに解決したかを必ず1行で明示してから進む
     （例: 「`sql指南書` → `pages/db/sql-mastery.html`（達人に学ぶ SQL）として扱います」）。
   - **複数ヒット** → 候補を列挙し、どれかを選択式で確認してから進む。
   - **0件** → 近い候補を提示しつつ、対象を質問する。勝手に1つに決めない。
   - `catalog.json` が無い／古い疑いがあるときは先に `/org` を促す。
2. 解決した対象ファイルを受け取る（アップロード必須。無ければファイル名を伝えて依頼）。
3. **変更計画を先に提示**: 「変更する点」「変更しない点」を箇条書きで明文化する。
4. 指示が曖昧・複数解釈できるなら、**着手せず**に質問を重ねて確定させる。
5. 確定後、`str_replace` で**該当箇所だけ**を編集。全体の再生成はしない
   （明示的に「作り直して」と言われた場合を除く）。
6. タイトル・書名・分野など catalog.json に載る属性を変えたときは `catalog.json` の該当エントリも直す。
7. 更新後ファイル（必要なら catalog.json も）を present_files で出し、変更点を一覧で報告。
8. 引数に `--push` があれば、続けて `/publish`（commit & push）まで実行する。`--push` は変更内容に含めない。

---

## コマンド: `/org [--push]`

`pages/` を棚別サブディレクトリに整頓し、検索用カタログ `catalog.json` を再生成する。
`/update` を「言葉」で叩けるようにするための土台。**真実は index.html の ARTICLES**で、
catalog.json と物理配置はそこから導く派生。冪等（既に整っていれば catalog の再生成だけ）。

1. **読み込み**: アップロードされた最新 `index.html` の `SHELVES` と `ARTICLES`、および `pages/` の現状を把握する。
2. **整頓（物理移動）**: 各記事を `pages/<shelf>/<slug>.html` に配置する（shelf は ARTICLES の各 `shelf`）。
   既に正しい場所なら動かさない。記事は自己完結なので内部リンクの書き換えは不要。
3. **参照更新**: `index.html` の各 `file:` を新パス `pages/<shelf>/<slug>.html` に揃える
   （`str_replace` で該当文字列だけ。ARTICLES のそれ以外の属性は触らない）。
4. **catalog.json を再生成**: ARTICLES を正として全エントリを書き出す。各記事に付ける項目:
   `slug` / `file` / `shelf` / `title` / `book` / `cat` / `tags`（＋シリーズは `vol`）に加え、
   **`aliases`**（言葉で呼ぶための別名: slug・書名の略称・日本語ニックネーム。例: 「SQL指南書」「DDIA3」）。
   既存 aliases は活かしつつ、明らかな呼び名を補う。
5. **提示**: 移動の一覧（from→to）、更新後 `index.html`、再生成した `catalog.json` を出し、差分を報告する。
6. 引数に `--push` があれば、続けて `/publish`（commit & push）まで実行する。

> 注意: 物理パスを変えると公開URL（`…/pages/<shelf>/<slug>.html`）も変わる。ブックマークし直しが要る旨を一言添える。

---

## コマンド: `/publish [コミットメッセージ(省略可)]`

生成・編集した内容を**コミットして公開サイトに反映**する。`/create`・`/update`・`/org` はファイルを提示する
ところまで。その成果を公開したいときに続けて `/publish` を叩く（自動pushはしない＝公開は必ずユーザーの明示操作）。

**初回（まだどこにも公開していないプロジェクト）は、リポジトリ作成と GitHub Pages 有効化まで面倒を見て
「初デプロイ」まで完了させる。2回目以降は commit & push だけ。** `gh`（GitHub CLI、認証済み）が前提。

1. **状態を判定**:
   - git リポジトリか（`.git` の有無）／`origin` リモートがあるか／GitHub Pages が有効か
     （`gh api repos/<owner>/<repo>/pages` が 200 なら有効、404 なら未設定）。
   - これで **初回デプロイ**（リポジトリ未作成 or Pages 未設定）か **通常公開**（両方済み）かを見分ける。
2. **変更点を提示**: `git status -s` で今回コミットされる差分を一覧し、何を公開するかを1行で要約する。
3. **整合チェック（推奨）**: 直前が `/create`・`/update`・`/org` なら、`index.html` の `file:` と
   実ファイル配置・`catalog.json` が一致しているかを軽く確認（ズレていれば `/org` を先に促す）。
4. **コミット**: 必要なら `git init` → 既定ブランチを `main` に。`git add -A` → メッセージでコミット。
   メッセージ省略時は内容から簡潔に生成（例: `Add article: <title>` / `Update <slug>: <変更要約>` /
   `Reorganize pages and refresh catalog` / 初回は `init: my learning library`）。
5. **公開**:
   - **初回デプロイ**:
     a. `origin` が無ければ GitHub リポジトリを新規作成して push する
        （`gh repo create <repo名> --public --source=. --remote=origin --push`）。
        リポジトリ名の既定はカレントディレクトリ名。**外向きの操作なので、リポジトリ名・公開範囲(public/private)
        を提示して了承を得てから**作成する。
     b. GitHub Pages を有効化する（未設定のときだけ）:
        `gh api -X POST repos/<owner>/<repo>/pages -f "source[branch]=main" -f "source[path]=/"`。
     c. 既に `origin` はあるが Pages だけ未設定なら、push したうえで b. だけ行う。
   - **通常公開**: 既定ブランチ（通常 `main`）へ push する。
6. **反映確認**: 公開URL（`https://<owner>.github.io/<repo>/`、記事は `…/pages/<shelf>/<slug>.html`）を伝える。
   初回は GitHub Pages の初回ビルドに数十秒〜数分かかる。可能なら 200 応答を待ってから知らせる。

> 破壊的・外向きの操作（リポジトリ作成・公開・push）なので、何をどこへ出すかを提示してから実行する。曖昧なら確認を取る。
> `gh` が無い／未認証なら実行せず、`gh auth login` と README の「GitHub Pages で公開する」手順を案内する。

### `--push` オプション（`/create`・`/update`・`/org` 共通）

各コマンドの末尾に `--push`（別名 `-p`）を付けると、**提示まで終えたあと続けて `/publish` 相当を実行**して
公開まで一気通貫で行う。付けなければ従来どおり提示で止まる（公開は別途 `/publish`）。

- 例: `/create TCP/IP 入門 --push` / `/update sql指南書 〇〇に変更して --push` / `/org --push`
- `--push` は変更指示の一部ではなくフラグ。第2引数（変更内容）の解釈からは除外する。
- フラグ付きでも手順は変えない。**生成・編集 → 整合チェック → 提示**を済ませてから公開する
  （提示を飛ばしてはいけない。公開前に何を出すかは必ず示す）。
- **初めてそのプロジェクトで `--push`／`/publish` を打ったときは、`/publish` の「初回デプロイ」に従って
  リポジトリ作成＋Pages 有効化まで行い、サイトを公開状態にする。**
- `gh` が無い・未認証ならその旨を伝えてファイル提示で止める。
- コミットメッセージは内容から自動生成（`/publish` の規定に従う）。

---

## シリーズ記事（1トピックを複数HTMLに分割する）

重厚な書籍・学習内容は、単一HTMLに収めず複数HTMLの「シリーズ」に分割してよい。分割するかは
都度Claudeが判断する（後述の基準）。狙いは **1ファイルを軽く保ち、各巻の記事品質を担保しやすくする**こと。

### いつ分割するか — 「概念の継ぎ目」で割る（サイズでは割らない）
- 判断軸は KB やサイズではなく「**独立して読んで価値がある大テーマが複数あるか**」。
  本がもともと独立した大テーマの束（例: DDIA の「複製」「整合性」…）なら分割の価値あり。
- 1トピックの深掘りにすぎず、途中で割ると論理が分断されるなら、長くても
  **単一HTML + Part** でまとめる（デザイン規約の「長い記事は Part で束ねる」）。
- **1つの章の主張（1つの論理）を2ファイルに割らない**。割るのは章・大テーマの境界だけ。
- 迷ったら既定は「割らない（単一HTML）」。

### 構成 — 入口は「目次ページ」1枚 ＋ 各巻 ＋ 兄弟ナビ
分割するシリーズは次で構成する（すべて同じ `pages/<shelf>/` 内）:
- **目次ページ** `pages/<shelf>/<series>-index.html` … シリーズの入口。全体像・読む順序の
  ロードマップ・各巻への入口リンクを置く。**ハブからはここに入る**。「どこから読めばいいか」はここで解決。
- **各巻** `pages/<shelf>/<series>-vol1.html` … `-volN.html`。番号順。各巻は通常記事と同じ自己完結HTML。
- **兄弟ナビ**: 各巻の上下に「← 第X巻 / 第Y巻 →」と「シリーズ目次へ」を置く。
  同一フォルダ内の相対リンクなので公開URLが変わっても壊れない。
  → 自己完結原則は「**シリーズ単位で自己完結**」に緩める。記事間リンク禁止の例外は
     **同一シリーズの兄弟・目次に限る**（他シリーズ・他棚へはリンクしない）。

### ハブ(index.html)での見せ方 — 代表カード1枚
ARTICLES にはシリーズにつき **代表カードを1件だけ**置き、`file:` は目次ページを指す。
巻ごとのカードは並べない（一覧が膨れない）。代表カードに `series` 配列で巻マニフェストを持たせる:
```js
{file:"pages/<shelf>/<series>-index.html", shelf:"<id>", accent:"#RRGGBB", cat:"CATEGORY",
 title:"シリーズ名", book:"書名/出典", levelFrom:"初級", levelTo:"中級",
 desc:"1〜2文の要約。", tags:["タグ1","タグ2"], vol:"全N巻シリーズ",
 series:[
   {vol:1, file:"pages/<shelf>/<series>-vol1.html", title:"第1巻のタイトル", aliases:["略称1"]},
   {vol:2, file:"pages/<shelf>/<series>-vol2.html", title:"第2巻のタイトル", aliases:["略称2"]}
 ]}
```
`series` はハブのレンダラが読まない追加フィールド（カードは目次ページ1枚として描画される）。
**真実は変わらず ARTICLES**。`series` は /org が catalog.json を展開するための巻マニフェストを兼ねる。

### catalog.json への展開（/org が再生成）
ARTICLES の代表カードから、catalog に **シリーズ親1件 ＋ 各巻の子N件**を書き出す:
- 親: `slug:<series>` / `file:目次ページ` / `isSeries:true` / `parts:N` ＋ 通常の title・book・cat・tags・aliases
- 子: `slug:<series>-vol1` / `file:各巻` / `partOf:<series>` / `vol:1` ＋ aliases（「DDIA1」等の呼び名）
これで `/update DDIA …`（シリーズ全体）も `/update DDIA3 …`（特定の巻）も言葉で解決できる。

### /create でシリーズを作るとき
1. 棚を決めた後、分割が妥当と判断したら、目次ページ ＋ 全巻HTML を生成し、各巻に兄弟ナビを入れる。
2. ARTICLES には代表カード1件（`series` マニフェスト付き）を追記。
3. catalog.json は親＋子を書き出す（または /org に再生成させる）。
4. 提示時に「**目次ページから読む**」ことを1行で案内する。

### /update でシリーズを直すとき
- 第1引数がシリーズ親に解決 → 「どの巻か／目次ページか／全巻共通か」を確認してから着手。
- 特定の巻に解決 → その巻だけ外科編集。巻タイトル等を変えたら ARTICLES の `series` 該当要素と
  catalog の子エントリも揃える。
- 巻を追加/削除したら、目次ページ・兄弟ナビ・ARTICLES の `series`・catalog をまとめて揃える。

### 既存 DDIA について
DDIA は現状「巻別カード4枚」の旧方式。動作はするので無理に触らない。新方式（代表カード＋目次ページ）へ
寄せたいときは `/org` 実行時にユーザー合意のうえ移行する（必須ではない）。

---

## 記事デザイン規約（シリーズで体裁を揃える）

- **1記事 = 単一の自己完結HTML**（重厚なトピックは複数HTMLの「シリーズ」に分割可 →「シリーズ記事」節）。
  CSS/JS は全てインライン。外部依存は Google Fonts のみ。
  ブラウザストレージ不使用。日本語。対象は初級→中級を基本に「他資料が要らない網羅度」。
- 共通の骨格:
  - 左に sticky な番号付き目次（IntersectionObserver で現在地ハイライト）
  - 上部にスクロール進捗バー、右下に「トップへ戻る」
  - ヒーロー（バッジ＋kicker＋大見出し＋lede＋メタchip）
  - セクションは 01,02… の等幅番号。長い記事は Part で束ねる
  - 各章冒頭に「一言でいうと」ボックス。理屈と具体例・コードを交互に
  - コールアウト 5種: 理屈(why)/要点(point)/補足(tip)/注意(warn)/危険(danger)
  - コードはダーク背景・トークン色分け・コピーボタン。CLIは `$` プロンプト
  - 図は inline SVG（角丸ボックス＋矢印マーカー＋アクセント色）。捏造数値・存在しない概念は不可
  - 比較は table、概念整理は cards、末尾にチートシート＋用語集
- **トピックごとに個性を変える**: アクセント色・フォントを変え「シリーズ感はあるが各々識別できる」状態に。
  汎用AI調フォント（Inter/Roboto/Arial 等）は避ける。日本語対応フォント＋等幅をGoogle Fontsから。
  既使用の目安（重複回避）: DB設計=teal+Shippori Mincho / SQL=indigo+Zen Old Mincho /
  ソフト設計=rose+Zen Maru Gothic / DDIA=ダーク+cyan&amber+Noto Serif JP / ハブ=parchment+Kaisei Tokumin。
- 著作権配慮: 原文の長い引用・逐語コピーをしない。自分の言葉で噛み砕く。
  末尾に「簡略化／仕様は変わりうる、最新は公式ドキュメントで」の注記。
- 温かく簡潔な口調。過剰な前置き・後書きはしない。

---

## GitHub Pages デプロイ（要約）

1. リポジトリに `index.html` / `catalog.json` / `pages/` / `.nojekyll` / `README.md` / `skills/` を配置。
2. push。
3. Settings → Pages → Source: *Deploy from a branch*、Branch `main` / `/(root)`。
4. 公開: ユーザーサイト `https://<user>.github.io/`、記事 `…/pages/<shelf>/<slug>.html`。

詳しい手順は README.md を参照。

## クイックリファレンス

| コマンド | 用途 | 必要な添付 |
|---|---|---|
| `/create [トピック] [--push]` | 新記事を `pages/<棚>/` に作りハブと catalog に追加 | 最新 index.html（＋catalog.json） |
| `/update [ワード or file] [内容] [--push]` | catalog で対象を名寄せして手直し | 最新 index.html か catalog.json（特定後に対象ファイル） |
| `/org [--push]` | `pages/` を棚別に整頓し catalog.json を再生成 | 最新 index.html＋pages/ 一式 |
| `/publish [メッセージ]` | 変更を commit & push して公開（初回はリポジトリ作成＋Pages有効化まで） | `gh` 認証済み |

> `--push`（`-p`）を `/create`・`/update`・`/org` の末尾に付けると、提示後そのまま公開まで一気通貫。

合言葉: **「真実は1つ、ビューは派生」**。目録の真実は `index.html` の `ARTICLES`。
`catalog.json` と `pages/<shelf>/` 配置はそこから導く派生で、`/org` がいつでも作り直せる。
