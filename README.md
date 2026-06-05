# Shoka（書架）— My Learning Library Template

**学んだ技術を一冊ずつ書架に収めて育てる、自分だけの学習まとめサイトのテンプレート。**
記事は Claude に `/create` と頼んで増やし、`/update` で直し、`/org` で整頓して、`/publish`（または `--push`）で
GitHub Pages に公開します。中身は **単一の `index.html` ＋ 記事HTML** だけ。ビルド不要・依存なしで動きます。

> これは**記事ゼロの初期テンプレート**です。`Use this template` か fork で自分のリポジトリを作り、
> Claude に最初の一冊を書いてもらうところから始めてください。

## まず使い始める（3分）

1. このリポジトリの **`Use this template` → `Create a new repository`**（または fork）で自分のリポジトリを作る。
2. ローカルに clone する。
3. **GitHub Pages を有効化**（下の手順）。空の書架が公開されます。
4. Claude に `skills/learning-library/SKILL.md` を読ませ、`/create [学びたいこと] --push` で最初の一冊を作って公開。

## 構成

```
.
├── index.html        トップページ（学習ハブ）。蔵書データは末尾の ARTICLES 配列【真実】
├── catalog.json      検索用カタログ【派生】。/org が ARTICLES から再生成（手で編集しない）
├── .nojekyll         GitHub Pages で素のまま配信させる（必須・空ファイル）
├── README.md
├── skills/
│   └── learning-library/
│       └── SKILL.md  /create・/update・/org・/publish を再現する運用スキル
└── pages/            記事HTML（1記事 = 自己完結HTML）。棚(shelf)別サブディレクトリに整頓
    ├── db/        データの土台（Database & Query）
    ├── backend/   バックエンド & API（Backend & API）
    ├── design/    ソフトウェア設計（Software Design）
    ├── infra/     クラウド & インフラ（Cloud & Infra）
    ├── dist/      分散データシステム（Distributed Data）
    └── tools/     開発ツール（Dev Tools）
```

- 初期状態は **記事ゼロ**。`pages/<棚>/` は空（`.gitkeep` のみ）、`ARTICLES` も空配列。
- 棚（分野）は `index.html` の `SHELVES` で定義。自由に増減・改名できます。
- 記事リンクはすべて**相対パス**（`pages/<shelf>/<slug>.html`）。ユーザーサイト・プロジェクトサイトどちらでも動く。
- 記事は自己完結（内部アンカー＋外部リンクのみ、記事間リンクなし）なので、棚をまたぐ移動でも壊れない。
- トップは検索ボックス＋分野フィルタ＋タグ絞り込みを備え、記事が100件規模でも快適。

## GitHub Pages で公開する

1. リポジトリの **Settings → Pages** を開く。
2. **Build and deployment → Source** を **Deploy from a branch** にする。
3. **Branch** を `main`、フォルダを `/ (root)` にして **Save**。
4. 数十秒〜数分で公開される。
   - ユーザーサイト（リポジトリ名が `<ユーザー名>.github.io`）→ `https://<ユーザー名>.github.io/`
   - プロジェクトサイト → `https://<ユーザー名>.github.io/<リポジトリ名>/`
5. 各記事は `…/pages/<shelf>/<slug>.html`。

> `.nojekyll` を必ず含めること。GitHub Pages の既定（Jekyll）による余計な処理を無効化し、
> ファイルをそのまま配信させるため。

## 記事を増やす・直す（Claude）

新しいチャットの冒頭に `skills/learning-library/SKILL.md` の内容を貼る（またはスキルとして読み込ませる）と、
次の合言葉が使える。**実行時は最新の index.html（更新なら対象記事も）を一緒にアップロードする。**

| コマンド | 用途 |
|---|---|
| `/create [書籍名 または 学習内容] [--push]` | 新記事を `pages/<棚>/` に作り、ハブ（ARTICLES）と catalog.json に追記 |
| `/update [識別ワード or file名] [変更内容] [--push]` | catalog.json で対象記事を一意に特定し、変更範囲を確認のうえ外科的に編集 |
| `/org [--push]` | `pages/` を棚別サブディレクトリに整頓し、catalog.json を再生成して参照パスを揃える |
| `/publish [メッセージ]` | 変更を commit & push し、公開サイトに反映（git リポジトリのある環境専用） |

`/update` の第1引数は正確なファイル名でなくてよい。記事を一意に特定できる言葉でも解決する。
例: `/update sql指南書 〇〇に変更して` → 該当記事を特定して編集。

`/create`・`/update`・`/org` はファイルを提示するところまで（自動pushはしない）。公開したいときは
末尾に **`--push`**（`-p`）を付けると提示後そのまま commit & push まで一気通貫。あるいは後から **`/publish`**
を叩いても反映できる。例: `/create TCP/IP 入門 --push` / `/update sql指南書 〇〇に変更して --push` / `/org --push`

## 新しい記事を手で追加する場合

1. 記事HTMLを `pages/<shelf>/<slug>.html`（棚に合うサブディレクトリ）に置く。
2. `index.html` の `ARTICLES:START 〜 ARTICLES:END` の間にメタ情報を1件追記（テンプレは SKILL.md 参照）。
3. Claude に `/org` を頼んで `catalog.json` を再生成（または手で1件追記）。
4. コミット → push。

---

このテンプレートは [Shoka](https://github.com/kb-daikigoto/shoka) を記事ゼロの初期状態に整えたものです。
自由に作り変えて、あなたの「書架」を育ててください。
