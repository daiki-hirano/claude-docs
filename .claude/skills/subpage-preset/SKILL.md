---
name: subpage-preset
description: 下層ページ（about、recruit など）の雛形一式（Pug・ページ用CSS・PC/SP用モジュールCSS・ページ用imagesディレクトリ）を作成するときに使用する。「下層ページを追加」「○○ページを作成」「下層のプリセットを作って」と依頼されたときは必ず使用する。
---

# 下層ページプリセット

下層ページの雛形を一式作成するスキル。ソースはすべて `_develop/src/` 配下。以下のパスは `_develop/src/` からの相対パスで記載する。

## 1. 作成前に決める内容

プロンプトに指定がない項目は、**作成前にユーザーに確認する**（推測で埋めない）。

| 項目               | 内容                               | 備考                                                                  |
| ------------------ | ---------------------------------- | --------------------------------------------------------------------- |
| ディレクトリ名     | `about` など                       | Pug / CSS / images のすべてで一致させる                               |
| ページ名（日本語） | `pageTitle` に使う                 |                                                                       |
| description        | `<meta name="description">` の文章 | ユーザーに確認し、入力された文章を使う。**未入力の場合は空文字 `''`** |

- `title` は下層では指定しない（`pageTitle` のみ定義し、サイト名との結合は `_base.pug` に任せる）。

## 2. 作成するもの（1階層のページ）

`about` ページの例。`about` の部分をディレクトリ名に置き換える。

| 種類                     | パス                          |
| ------------------------ | ----------------------------- |
| Pug                      | `pug/about/index.pug`         |
| ページ用 CSS             | `css/about.css`               |
| PC 用モジュール CSS      | `css/_modules/_about/_pc.css` |
| SP 用モジュール CSS      | `css/_modules/_about/_sp.css` |
| ページ用画像ディレクトリ | `images/about/`               |

`public_html/` はビルドで出力されるため、直接編集しない。

### Pug（`pug/about/index.pug`）

基本構造は下記のコード例のとおり（実装例: `pug/about/index.pug`）。**ページごとに変更するのは `description`、`canonicalURL`、`pageTitle`**。

```pug
extends ../utils/_base.pug

block append meta
  -
    var description = '';
    var canonicalURL = baseURL + '/about/';

block append page
  - var pageTitle = '会社案内'

block append mainClass
  - var mainClass = 'subpage-container'

block append stylesheet
  link(rel="stylesheet", href=assetPath + 'css/about.css')

block main
```

- `block append mainClass` は `main` に `.subpage-container`（header の高さ分の `padding-top`、`common.css` で定義）を付与する。下層ページには必ず入れる。
- `description` は確認した文章を入れる。未入力なら `''` のまま。
- `block append stylesheet` で、ページ用 CSS を読み込む（`common.css` は `_base.pug` 側で読み込み済みのため `append` を使う）。
- `block main` の中に、そのページの本文を追加していく。

### CSS

`css/about.css`:

```css
@import "./_config/_variables.css" layer(common);
@import "./_config/_media.css" layer(common);

@import "./_modules/_about/_pc.css" layer(page);
@import "./_modules/_about/_sp.css" layer(page);
```

`css/_modules/_about/_pc.css`:

```css
@media (--pc) {
}
```

`css/_modules/_about/_sp.css`:

```css
@media (--sp) {
}
```

- `@media (--pc)` / `@media (--sp)` は1ファイルにつき1つだけ。以降のスタイルはこの中に追記する。
- 共通スタイルは `common.css` に含まれているため、ページ用 CSS には書かない。

### 画像

- `images/about/` ディレクトリを作成する（`.gitkeep` は置かない）。
- 画像の扱い（`+img` / `+pic`、WebP の生成）はプロジェクトルートの `AGENTS.md` の「画像」に従う。

## 3. 階層が深いページ（例: `recruit/requirements/`）

階層ごとに Pug・CSS・画像を分ける。`recruit/requirements/` の場合は以下。

| 種類                     | パス                                                     |
| ------------------------ | -------------------------------------------------------- |
| Pug                      | `pug/recruit/requirements/index.pug`                     |
| ページ用 CSS             | `css/recruit/requirements.css`                           |
| PC / SP 用モジュール CSS | `css/_modules/_recruit/_requirements/_pc.css`、`_sp.css` |
| ページ用画像ディレクトリ | `images/recruit/requirements/`                           |

- 親の階層（`recruit/`）も新規ページとして作る場合は、1階層のページとして別途作成する（`pug/recruit/index.pug`、`css/recruit.css`、`images/recruit/`）。
- Pug の `extends` / `include` は階層に合わせて `../` を増やす（例: `extends ../../utils/_base.pug`）。
- ページ用 CSS が1階層深くなるため、`@import` のパスは `../` から始める。

```css
/* css/recruit/requirements.css */
@import "../_config/_variables.css" layer(common);
@import "../_config/_media.css" layer(common);

@import "../_modules/_recruit/_requirements/_pc.css" layer(page);
@import "../_modules/_recruit/_requirements/_sp.css" layer(page);
```

- スタイルシートの読み込み: `link(rel="stylesheet", href=assetPath + 'css/recruit/requirements.css')`
- `canonicalURL`: `baseURL + '/recruit/requirements/'`

## 4. 作成後の確認

- `_develop` でビルド（`npm run build:pug`、`npm run build:css`）がエラーなく通ること。開発サーバーが起動中の場合は watch の出力を確認する。
- ブラウザで `/about/` を開き、ページが表示されること、横スクロールが出ていないことを確認する（Chrome DevTools MCP を接続している場合は PC / SP の両方）。

## 補足

- このプリセットでは JS は作成しない。
