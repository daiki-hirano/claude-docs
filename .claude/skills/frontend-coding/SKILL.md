---
name: frontend-coding
description: このプロジェクト（Pug + PostCSS + TypeScript）でページ・コンポーネント・CSSをコーディングするときの基本ルール。Figma MCP からの実装、ページ追加、Pug/CSS の作成・修正、画像の追加、レイアウト実装を行うときは必ず使用する。
---

# フロントエンドコーディング基本ルール

ソースはすべて `_develop/src/` 配下。以下のパスは `_develop/src/` からの相対パスで記載する。

## 全体の方針

- **DRY 原則**（Don't Repeat Yourself）を守る。同じ内容・同じスタイルを複数箇所にコピーして書かない。
  - Pug: 同じ構造の繰り返しは `each` + データ（`_variable.pug`）や mixin にまとめる。
  - CSS: 同じスタイルが複数の要素・ページで使われる場合は、共通のコンポーネントとして切り出す（`common` レイヤー）。値は `css/_config/_variables.css` の変数を利用する。
  - 実装前に、既存の mixin・CSS・変数に使えるものがないか確認し、あれば再利用する。
  - 共通化のために無理な抽象化はしない。たまたま似ているだけで、役割や変更理由が別のものは分けてよい。

## レイアウト

- Figma MCP 経由で実装する場合は、flexbox ではなく **grid layout** で実装する。
  - ただし、コンテンツの固有幅に基づいて横並びにし、カラム落ちなどが想定される場合は flexbox を使用してよい。
- コンテンツ間の余白は **gap** で調整する。
- 1カラムのレイアウトは grid を指定し、横幅が溢れるバグを防ぐため `grid-template-columns: 100%` を指定する（**`1fr` ではなく `100%`**）。
- 画像とテキストの横並びは `grid-template-columns` を使用する。
  - 画像は固定幅、テキストなどのコンテンツは `auto` を指定する。
  - 画面縮小時のレイアウト崩れを防ぐため、画像は `minmax(0, <固定幅>)` で指定する。
  - コンテンツもデザイン上固定幅の場合は、`auto` ではなく固定幅を指定する。
- 子要素の位置揃えは、なるべく `place-content` / `place-items` を使用する。
  - `align-items` / `justify-content` を指定した方が保守性や可読性がよくなる場合は、そちらを使用してよい。
- **`position: absolute` は、装飾目的以外では極力使用しない**。
  - 横並びや、要素同士を重ねるレイアウトは grid で実装する。重ねる場合は、親に `grid-template-columns: 100%` / `grid-template-rows: 100%` を指定し、重ねる要素すべてに `grid-area: 1 / 1` と `position: relative` を指定する（画像の上にテキストやアイコンを乗せる場合など）。`position: relative` を指定しておくことで、後ろの要素が前の要素の上に乗り、`z-index` も効くようになる。
  - アイコンやテキストを並べる場合も、`grid-template-columns` と `column-gap` で並べる（例: 見出しの横に飾り文字を置く場合は、見出しを `grid-template-columns: auto auto` にして並べる）。
  - 少しズレる場合は、`translate` や `margin` で微調整してよい。
  - 背景に敷く大きな飾り文字、擬似要素（`::before` / `::after`）による装飾線・オーバーレイなど、レイアウトに影響させたくない装飾目的の場合は `position: absolute` を使用してよい。
- カードなどの横並びレイアウトで、子要素（タイトル・本文など）の高さを他のアイテムと揃えたい場合は **subgrid** を使用する。
  - 例: タイトルが 2 行のカードと 1 行のカードが混在していても、タイトルの高さ（その下の要素の開始位置）を揃える。
  - `height` / `min-height` で高さを固定して揃えない（文字量や画面幅で崩れるため）。
  - 親（リスト）に `grid-template-columns` を指定し、各アイテムは `grid-row: span <子要素の数>` と `grid-template-rows: subgrid` を指定して、親の行トラックを共有する。
  - subgrid の `gap` は親グリッドの値を引き継ぐ。アイテム内の余白を変えたい場合は、アイテム側で `row-gap` を指定して上書きする。

```css
.list {
  display: grid;
  grid-template-columns: 100%;
  gap: 24px;
}

.media {
  display: grid;
  grid-template-columns: minmax(0, 400px) auto;
  gap: 40px;
  place-items: center start;
}

/* 画像の上にコンテンツを重ねる: absolute ではなく同じグリッドエリアに置く */
.banner {
  display: grid;
  grid-template-columns: 100%;
  grid-template-rows: 100%;
}

.banner__image,
.banner__content {
  position: relative;
  grid-area: 1 / 1;
}

/* 横並びのカード: 画像 / タイトル / 本文 の 3 行を全アイテムで揃える */
.card-list {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  column-gap: 40px;
}

.card {
  display: grid;
  grid-row: span 3;
  grid-template-rows: subgrid;
  row-gap: 16px;
}
```

## CSS

### ファイル構成

- ページ用 CSS は `css/<ページ名>.css` として作成する（例: `css/index.css`、`css/about.css`）。
- PC / SP のスタイルは分離し、`css/_modules/_<ページ名>/_pc.css` と `css/_modules/_<ページ名>/_sp.css` に記述する（ページ名のディレクトリには先頭に `_` を付ける）。
  - 例: TOP ページ → `css/_modules/_index/_pc.css`、about ページ → `css/_modules/_about/_pc.css`
- ページ用 CSS では以下を**必ず**インポートし、続けて PC / SP のファイルをインポートする。

```css
/* css/about.css */
@import "./_config/_variables.css" layer(common);
@import "./_config/_media.css" layer(common);

@import "./_modules/_about/_pc.css" layer(page);
@import "./_modules/_about/_sp.css" layer(page);
```

### コンポーネントの CSS

`pug/_components/` 配下のコンポーネント（mixin）のスタイルは、**ページ用 CSS（`_pc.css` / `_sp.css`）には書かず**、コンポーネント専用の CSS ファイルに切り分ける。

| 種類             | CSS の配置先                                                                       | 読み込み                                                          |
| ---------------- | ---------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| サイト全体で使用 | `css/_modules/_common/_<コンポーネント名>/_<コンポーネント名>.css`                 | `css/common.css` に `layer(common)` で import                     |
| ページ固有       | `css/_modules/_<ページ名>/_components/_<コンポーネント名>/_<コンポーネント名>.css` | そのページの CSS（例: `css/index.css`）に `layer(page)` で import |

- 例: TOP 専用の背景文字コンポーネント（`pug/_components/index/text/_bg-text.pug`）→ `css/_modules/_index/_components/_bg-text/_bg-text.css`
- ページ固有コンポーネントの import は、そのページの `_pc.css` / `_sp.css` の**前**に書く（ページ側で上書きや微調整ができるように）。
- コンポーネントはコードが長くならないため、**PC / SP は分けず 1 ファイルにまとめる**。ファイル内に `@media (--pc)` と `@media (--sp)` を 1 つずつ書く。PC / SP で共通の値になるスタイルも `@media` の外には書かず、それぞれの `@media` に分けて記述する（「レスポンシブ」参照）。
- クラス名（ブロック名）は mixin 名と揃える（`+strengthCard` → `.strength-card`）。
- コンポーネントを使う側のページ固有の調整（位置の指定、`margin` など）は、ページ用 CSS に書く。コンポーネント自体の見た目は、ページ用 CSS に書かない。

```css
/* css/index.css */
@import "./_modules/_index/_mv-slider.css" layer(page);
@import "./_modules/_index/_components/_strength-card/_strength-card.css"
  layer(page);
@import "./_modules/_index/_pc.css" layer(page);
@import "./_modules/_index/_sp.css" layer(page);
```

```css
/* css/_modules/_index/_components/_strength-card/_strength-card.css */
@media (--pc) {
  .strength-card {
    padding: 30px;
  }
}

@media (--sp) {
  .strength-card {
    padding: vw(24);
  }
}
```

### カスケードレイヤー

`css/common.css` でカスケードレイヤー（`common, page, override`）を定義している。

- ページ固有の CSS → `page` レイヤー
- コンポーネントや共通コンテンツ（header、footer など） → `common` レイヤー

### レスポンシブ

- PC のスタイルは `@media (--pc)`、SP のスタイルは `@media (--sp)` で囲む。
- **`@media (--pc)` / `@media (--sp)` は 1 ファイルにつき 1 つだけ**定義し、そのファイル内のすべてのセクション・コンポーネントのスタイルをその中にまとめて記述する。セクションやコンポーネントごとに `@media` を何度も定義しない。
  - ホバー用の `@media (any-hover: hover)` など、用途の違う `@media` を内側にネストするのは可。
- **PC / SP で同じ値になるスタイルであっても、`@media` の外には書かず、必ず `@media (--pc)` と `@media (--sp)` のそれぞれに分けて記述する**（`display` や `color` など、PC / SP で共通の値も両方に書く）。
  - 値を変更するときに PC / SP それぞれで見直せるようにするため。DRY 原則の例外として、この重複は許容する。
  - アニメーション（`@keyframes` など）も、`@media` の外に書かず、PC / SP それぞれに記述する。
- PC / SP でコンテンツの表示・非表示を切り替える場合は、`css/_modules/_common_override.css` の **`.only-pc`**（PC のみ表示）/ **`.only-sp`**（SP のみ表示）を該当コンテンツに指定する。個別のページ CSS で `display: none` を書かない。
- SP では、Figma から得た px 値を `vw(数値)` で記述する（PostCSS 側で vw に変換される）。
  - 例: 18px → `vw(18)`

### カラー

- カラーは **カラーコードをそのまま入力**し、新たに CSS 変数化はしない。
- ただし、すでに同じカラーコードで変数化されているもの（`css/_config/_variables.css` など）がある場合は、その変数を使用する。

```css
/* 変数化されていない色 → カラーコードをそのまま記述 */
.about-intro__title {
  color: #0d3c72;
}

/* 同じカラーコードで変数化済み（--col_white: #fff）→ 変数を使用 */
.about-intro__label {
  color: var(--col_white);
}
```

### フォント

- サイト全体のフォントは `body` で以下を指定済み。通常はこの指定を継承するため、個別に `font-family` を書かない。

```css
font-family:
  "Hiragino Sans", "Hiragino Kaku Gothic ProN", "Hiragino Kaku Gothic Pro",
  "Noto Sans JP", "Yu Gothic", "Meiryo", sans-serif;
```

- デザインで上記以外のフォントを使用している場合は、Google Fonts に該当フォントがあるか確認し、ある場合はそのフォントの URL（`<link>`）を貼る。
  - サイト全体（複数ページ）で使う可能性が高いフォントは、`pug/utils/_base.pug` の Google Fonts の `link`（`family=` を `&` でつなげた 1 本の URL）にまとめて追記する。使用するウェイトも `wght@400;500;600;700` のように漏れなく含める。ページ側の `index.pug` などには書かない。
  - 特定ページでしか使わないフォントだけ、そのページの Pug の `block append stylesheet` に `link(rel="stylesheet", href="https://fonts.googleapis.com/...")` として記述する（`_base.pug` の `preconnect` が指定済み）。
- ただし、すでにフォントデータが何らかの形でインポートされている場合（Google Fonts の読み込み済み、`@font-face` での指定、ローカルのフォントファイルなど）は、Google Fonts の確認を省く。

### 記述ルール

- **BEM 記法**で記述する。
- **SCSS 構文**（ネスト）で記述し、Element / Modifier は `&__` / `&--` のようにネストして書く。
- Figma で `leading-trim: both;` と `text-edge: cap;` が使われている場合は、そのまま CSS に記述する（`npm_scripts/postcss-replace-figma-value.js` で自動変換される）。

```css
/* css/_modules/_about/_sp.css */
@media (--sp) {
  .about-intro {
    display: grid;
    grid-template-columns: 100%;
    gap: vw(24);

    &__title {
      font-size: vw(18);
      leading-trim: both;
      text-edge: cap;
    }
  }

  /* 同じファイル内の別セクションも、同じ @media (--sp) の中に続けて書く */
  .about-staff {
    display: grid;
    grid-template-columns: 100%;
    gap: vw(16);
  }
}
```

## Pug

### ページ

- ページ用の Pug は `index.pug` として作成する（例: `pug/about/index.pug`）。
- `pug/utils/_base.pug` を継承する。これがベースになるため、meta などに必要な変数を各ブロックで指定する。
  - `block meta`: `title`、`description`、`canonicalURL`
  - `block page`: `pageTitle`
  - `block append stylesheet`: ページ用 CSS の読み込み
  - `block ld`: パンくずの構造化データ（`+breadcrumbList`）
  - `block main`: ページ本文

```pug
//- pug/about/index.pug
extends ../utils/_base.pug

block meta
  -
    var title = "サイト名";
    var description = "ページの説明文";
    var canonicalURL = baseURL + '/about/';

block page
  - var pageTitle = 'ABOUT'

block append stylesheet
  link(rel="stylesheet", href=assetPath + 'css/about.css')

block ld
  +breadcrumbList([
    { url: '/about/', name: 'ABOUT' }
  ])

block main
```

### リンク

- リンク（`href`）は一旦すべて `#` を設定する。
- 原則として、リンクになる DOM（`a` など）には `css/_modules/_hover.css` の **`.link__alpha`** クラスを付与する（ホバー時の透明度アニメーション）。
  - コンポーネント（mixin）内のリンクにも付与する。
  - デザイン側でホバー表現が別途指定されている場合は、そちらを優先する。

```pug
a.about-intro__link.link__alpha(href="#") 詳しく見る
```

### 繰り返しコンテンツのデータ

カードレイアウトや同じスタイルのボックスが続く場合は、データをページの Pug と同階層の `_variable.pug` にオブジェクト形式で定義し、include して `each` で出力する。

```pug
//- pug/about/_variable.pug
-
  const sampleData = [
    { id: 0, name: 'hoge' },
    { id: 1, name: 'tanaka' },
  ];
```

```pug
//- pug/about/index.pug
include ./_variable.pug

block main
  ul.about-list
    each item in sampleData
      li.about-list__item= item.name
```

### サイト共通の変数

- コンポーネントを作成したら、**ページ全体（header / footer / 複数ページ）で使用する変数・データ**は `pug/utils/_variable.pug` に記述する。
  - 例: グローバルナビゲーションのリンク・テキスト、プライバシーポリシーのリンク、コピーライトなど。
  - 同じ内容を header・footer など複数のコンポーネントに直書きせず、`_variable.pug` を include して参照する。
- 特定ページでしか使わない繰り返しデータは、前述のとおりページと同階層の `_variable.pug` に記述する。

```pug
//- pug/utils/_variable.pug
-
  const globalNav = [
    { text: 'ホーム', href: '#' },
    { text: '会社を知る', href: '#' },
  ];
```

```pug
//- pug/assets/include/footer.pug
include ../../utils/_variable.pug

each item in globalNav
  a.link__alpha(href=item.href)= item.text
```

### header / footer（SSI）

- header / footer は SSI で読み込むため、`pug/assets/include/header.pug`、`pug/assets/include/footer.pug` として作成する（`assets/include/*.html` に出力される。読み込み側は `pug/utils/_base.pug` を参照）。
- `_base.pug` は継承せず、必要な mixin / 変数（`_mixin.pug`、`_variable.pug`、`_components/` 配下のコンポーネント）を include して使用する。

### コンポーネント（mixin）

コンポーネントは `pug/_components/` 配下に作成し、カテゴリーごとにディレクトリを分けて運用する。

| 用途             | 配置先                                 | 例                                       |
| ---------------- | -------------------------------------- | ---------------------------------------- |
| サイト全体で使用 | `_components/common/<カテゴリー>/`     | `_components/common/button/_button.pug`  |
| ページ固有       | `_components/<ページ名>/<カテゴリー>/` | `_components/about/list/_about-list.pug` |

#### Figma MCP 接続時の使い分け

- Figma のレイヤー名に **`(components)`** が付いているものは、コンポーネントとして扱う。
  - 該当レイヤーを新たにマークアップせず、`pug/_components/` 配下の既存コンポーネント（mixin）を呼び出して使用する。
  - 該当するコンポーネントが存在しない場合は、先に `_components/` 配下に作成してから使用する（配置先は上記の表に従う）。
  - コンポーネントの props で表現できない差異がある場合は、独自にマークアップせず、作業者に確認する。

#### 引数の渡し方

mixin の引数は、React の props のように**オブジェクト 1 つ**で渡す。引数を個別に並べない。

- mixin 側では引数名を `props` とし、冒頭で分割代入して使用する。
- 省略可能な値は分割代入のデフォルト値で指定する。
- 呼び出し側ではオブジェクトリテラルで渡す。

```pug
//- pug/_components/about/card/_about-card.pug
mixin aboutCard(props)
  -
    const { title, text = '', href = '#' } = props;
  a.about-card(href=href)
    p.about-card__title= title
    if text
      p.about-card__text= text
```

```pug
//- 呼び出し側
+aboutCard({ title: 'タイトル', text: '説明文' })

//- 繰り返しデータはそのまま渡せる
each item in sampleData
  +aboutCard(item)
```

## JavaScript（TypeScript）

ソースは `js/` 配下（`_develop/src/js/`）。

### ファイル構成

- ページのエントリーは `js/pages/<ページ名>/index.ts`（例: `js/pages/index/index.ts`）。`js/main.ts` の `getComponent` でパスに応じて動的 import される。
- **ページ固有の JS** は `js/pages/<ページ名>/_modules/<名前>.ts` に作成し、そのページの `index.ts` から `./_modules/<名前>` で import する。
  - 例: TOP の施工実績スライダー → `js/pages/index/_modules/works-slider.ts`
- 複数ページで使い回す JS だけを `js/utils/`（`ui/`、`logic/` など）に置く。ページ固有のものを `utils/` に置かない。

```ts
// js/pages/index/index.ts
import WorksSlider from "./_modules/works-slider";

export default class Index {
  constructor() {
    new WorksSlider();
  }
}
```

## 画像

### 配置

- そのページのみで使用する画像は、ページのディレクトリ名でフォルダを作成して管理する。
  - 例: about ページ → `images/about/`、TOP ページ → `images/index/`
- サイト全体で使用する画像は `images/common/` に置く。

### 命名規則

- SP 用の画像には末尾に **`_sp`** を付ける（例: `footer_bg_sp.jpg`、`mv_sp.jpg`）。
- PC 用の画像には何も付けない（例: `footer_bg.jpg`、`mv.jpg`）。

### 書き出し・変換

- 画像を切り出す場合は、高解像度ディスプレイを考慮して **2倍** で書き出す。
  - **原寸（1倍）や、Figma の塗りに入っている元画像（`get_design_context` が返す画像 URL や `download_assets` の `rawImages`）をそのまま使わない。**
  - Figma MCP の `download_assets` で、画像を表示しているレイヤーを `defaultScale: 2` と、下記「書き出し形式」に従った `defaultFormat` を指定して書き出し、その `export` を使う。
  - 保存したら、画像の縦横ピクセル数が**デザイン上の表示サイズの 2 倍**になっていることを確認する（例: PC で 800×560 で表示する画像は 1600×1120）。SP 用の画像（`_sp`）も同様に、SP の表示サイズの 2 倍にする。2 倍になっていない場合は、そのまま使わず原因を確認する。
  - 元画像が透過 PNG の場合、JPEG に変換すると端に白い 1px の余白が出ることがある。書き出した画像の端に余白がないことも確認する。
- **書き出し形式**は画像の内容で決める。`download_assets` の `defaultFormat` で指定する。
  - 背景が透過されているイラストなど、透過が必要な画像 → **`png`**
  - 風景・人物・製品写真など、透過が不要な画像（上記以外のすべて） → **`jpg`**
  - 透過が必要な画像を `jpg` にすると、透過部分が白く塗られるため使わない。
  - 判断に迷う場合（透過の有無が不明な場合など）は、作業者に確認する。
  - SVG（アイコン・図形・線）は `svg` で書き出す（`+img` で指定する）。
- `src/images/` 直下の画像は**元画像**として置いておく。WebP の生成と使用は `AGENTS.md` の「画像」に従う。
- Figma でレイヤーの「塗り」に画像が複数枚指定されているものを 1 枚の画像としてダウンロードする場合は、UI（デザイン）で表示されている画像 1 枚をダウンロードする。
  - 例: 塗りに image01、image02、image03 が組み込まれていたら、デザインで実際に表示されている塗り画像を元にダウンロードする。
  - ただし、フェードインのスライダーなど複数枚の画像を使う可能性もあるため、ユーザーからそう指示された場合はその指示に従う。
- 画像を表示しているレイヤーに角丸（border-radius）が指定されている場合は、スタイルを初期化した状態の画像をダウンロードし、角丸は CSS で再現する。
  - オーバーレイ（グラデーションや半透明の色の重ね）なども同様に、画像に焼き込んで 1 枚画像としてダウンロードせず、CSS で再現できるものはなるべく CSS で再現する。

### マークアップ

- 画像は、Pug では `images/` からのパスで `+img` / `+pic` に指定する（`AGENTS.md` の「画像」参照）。
  - `+pic('images/about/mv.jpg', 'images/about/mv_sp.jpg', alt)` のように PC / SP で画像を切り替える場合は、`_sp` を付けた画像をそれぞれ書き出す。
  - Pug のデータ（`_variable.pug` など）に画像パスを持たせる場合も `images/` から指定し、mixin に渡す。
- SVG も `pug/utils/_mixin.pug` の `+img` で `images/` から指定する。
- 画像はなるべく `figure` タグで囲み、`aspect-ratio` を指定する。Figma に値がない場合は画像の縦横サイズを取得して指定する。
- `alt` は、アイコンなど意味のない画像以外は必ず入力する。意味のない画像は `''` を明示的に渡す。

```pug
figure.about-intro__image
  +img('images/about/intro.jpg', '代表メッセージの写真')

figure.about-intro__visual
  +pic('images/about/mv.jpg', 'images/about/mv_sp.jpg', 'メインビジュアル')
```

```css
.about-intro__image {
  aspect-ratio: 400 / 300;
}
```

## アクセシビリティ

- A11y 対策として、WCAG の**レベル A** の基準を満たすようにコーディングする。
- アコーディオンやハンバーガーメニューなど、開閉して表示・非表示を切り替えるコンポーネントでは `aria-hidden` を使用する。
  - 閉じている間は非表示にする領域に `aria-hidden="true"` を指定し、開いたら `aria-hidden="false"` に切り替える。

```pug
.accordion__body(aria-hidden="true")
  p 開閉するコンテンツ
```

## 実装後の確認

Chrome DevTools MCP を接続している場合は、実装後に PC / SP の両方で以下を確認する。

- コンテンツ幅を超えて横スクロールバーが表示されていないか
- 表示崩れが起きていないか

ローカルサーバー（BrowserSync）のポートはデフォルト 3000 だが、起動時に変わる可能性があるため、起動ログに表示された URL を使用する。
