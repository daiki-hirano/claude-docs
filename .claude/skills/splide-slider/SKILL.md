---
name: splide-slider
description: ライブラリ Splide（@splidejs/splide / @splidejs/react-splide）でスライダー・カルーセルを実装するときに使用する。「スライダー」「カルーセル」「splide」と依頼されたとき、Figma のレイヤー名に slider / splide があるときは必ず使用する。Pug + PostCSS と React / Next の両方に対応する。
---

# Splide スライダー実装ルール

ライブラリ **Splide** を使ってスライダーを実装するときのルール。Pug + PostCSS のプロジェクトは 2〜5 章、React / Next のプロジェクトは 6 章に従う。

- 通常版のソースは `_develop/src/` 配下。以下のパスは `_develop/src/` からの相対パスで記載する。
- ファイルの配置・Pug・CSS・画像などの基本ルールは `frontend-coding` スキルに従う。
- 公式ドキュメント: https://ja.splidejs.com/documents/

## 1. 実装前に確認すること

### プロジェクトの種類とパッケージ

- `package.json` を確認し、プロジェクトの種類を判定する。
  - `react` / `next` がある → React 版（6 章）
  - Pug + PostCSS → 通常版（2〜5 章）
- `package.json` の dependencies に Splide が入っているか確認する。**未インストールの場合は、インストールしてよいかユーザーに確認してから実行する**（勝手にインストールしない）。
  - 通常版: `npm install @splidejs/splide`
  - React 版: `@splidejs/react-splide`（パッケージマネージャーはロックファイルで判定する）

### ユーザーに指定がない場合に確認すること

プロンプトに指定がない項目は、`AskUserQuestion` で確認する（推測で決めない）。

- 自動再生を ON にするか
- 自動再生の場合、`interval` の値をどうするか
- 自動再生の場合、Intersection 拡張機能（画面内にあるときだけ再生）を使用するか
  - 使い方: https://ja.splidejs.com/extensions/intersection/
- 再生速度（`speed`）の値をどうするか

### Figma MCP が接続されている場合

デザインを確認し、矢印・ページネーションの有無を判断する。

- デザインに**ない**場合: オプションでオフにする（`arrows: false` / `pagination: false`）。
- デザインに**ある**場合: 3 章の Splide のルールに沿って追加する。

## 2. 初期化用 CSS（Pug + PostCSS のプロジェクト）

- ユーザーから特に指定がなければ、公式の初期化用 CSS を取得して `css/splide-core.min.css` として配置する。すでに配置されている場合は再取得しない。
  - 取得元: https://github.com/Splidejs/splide/blob/master/dist/css/splide-core.min.css
  - 取得 URL（raw）: `https://raw.githubusercontent.com/Splidejs/splide/master/dist/css/splide-core.min.css`
- 初期化用 CSS は**そのスライダーを使うページ用 CSS** で import する。`page` より下位の `common` レイヤーに置き、ページ側で上書きできるようにする（実装例: `css/business.css`）。

```css
/* css/business.css */
@import "./_config/_variables.css" layer(common);
@import "./_config/_media.css" layer(common);

/* Splide の初期化用 CSS。page より下位のレイヤーに置き、ページ側で上書きできるようにする */
@import "./splide-core.min.css" layer(common);

@import "./_modules/_business/_pc.css" layer(page);
@import "./_modules/_business/_sp.css" layer(page);
```

- React / Next のプロジェクトでは `css/` に配置せず、**コンポーネント側で import** する（6 章）。

## 3. HTML 構造

参考: https://ja.splidejs.com/guides/structure/

### 基本の構造

```html
<section class="splide" aria-label="スライダーの説明">
  <div class="splide__track">
    <ul class="splide__list">
      <li class="splide__slide">...</li>
    </ul>
  </div>
</section>
```

- ルート（`.splide`）には `aria-label`（見出しが見える場合は `aria-labelledby`）を付ける。
- `track` と `list` には**任意の要素を追加できない**。`list` は `track` の、`slide` は `list` の子要素にする。
- `track` はルート直下でなくてもよい（ラッパーの `div` で囲んでもよい）。

### デザインに合わせて柔軟に組む

Splide のデフォルトの HTML 構造（矢印 → track → ページネーションを自動で挿入）がデザインと合わない場合は、`track` や、矢印・ページネーションの**プレースホルダ**を使って柔軟に組む。

- 矢印を track の下に置く、矢印でページネーションを挟む、などは track の外に要素を置いて作る。
- スライド内の画像の高さなどをコンテナに適用したい場合は `.splide__slide__container` を使う（スライド直下に置く）。

### 矢印

参考: https://ja.splidejs.com/guides/arrows/

- `.splide` 内の任意の位置に `.splide__arrows` を置き、その中に `button.splide__arrow.splide__arrow--prev` / `button.splide__arrow.splide__arrow--next` を書くと、Splide がプレースホルダとして使う（自動生成されない）。
- 矢印は `<button>` で作成する。`aria-label` / `aria-controls` は Splide が付与するため書かない。
- アイコン画像は `+img` / `+pic` を使い、`alt` は `''` にする（`aria-label` で意味が伝わるため）。
- ホバーが必要な場合は `.link__alpha` を付与する（`frontend-coding` のリンクのルールに従う）。

### ページネーション

参考: https://ja.splidejs.com/guides/pagination/

- 空の `ul.splide__pagination` を任意の位置に置くと、Splide がプレースホルダとして使い、その中に `li > button` を生成する。
- 見た目は `classes.page` で独自クラスを追加し、CSS で作る。現在のスライドのボタンには `is-active` が付く。
- ボタンの中身を変えたい場合は `pagination:mounted` イベントの `data.items`（`li` / `button` / `page`）で加工する。番号を表示するだけなら CSS カウンターを使う。

### `classes` オプション

`classes` で独自クラスを追加するときは、`splide__` で始まるクラス（Splide が参照している）は**削除せず併記**する。

```ts
classes: {
  arrows: 'splide__arrows your-arrows',
  arrow: 'splide__arrow your-arrow',
  prev: 'splide__arrow--prev your-prev',
  next: 'splide__arrow--next your-next',
  pagination: 'splide__pagination your-pagination',
  page: 'splide__pagination__page your-page',
},
```

### Pug の雛形

```pug
.xxx-slider.splide(aria-label="スライダーの説明")
  .splide__track
    ul.splide__list
      each item in slides
        li.splide__slide.xxx-slider__slide
          //- 内容
  //- 矢印・ページネーションは track の外に置く（デザインに合わせて位置・順序を決める）
  .xxx-slider__controller.splide__arrows
    button.xxx-slider__arrow.splide__arrow.splide__arrow--prev.link__alpha(type="button")
      +pic('images/xxx/slider_arrow.svg', 'images/xxx/slider_arrow_sp.svg', '')
    ul.xxx-slider__pagination.splide__pagination
    button.xxx-slider__arrow.splide__arrow.splide__arrow--next.link__alpha(type="button")
      +pic('images/xxx/slider_arrow.svg', 'images/xxx/slider_arrow_sp.svg', '')
```

実装例: `pug/business/index.pug`（製品実績スライダー）。

## 4. JavaScript（TypeScript）

### ファイル構成

`frontend-coding` の JavaScript のルールに従う。

- ページ固有の JS として `js/pages/<ページ名>/_modules/<名前>-slider.ts` を作成し、そのページの `index.ts` から import する。
- `js/main.ts` の `getComponent` に、そのページのパスの分岐（dynamic import）を追加する。
- スライダーの対象要素がなければ `return` する。

```ts
// js/pages/business/_modules/works-slider.ts
import Splide from "@splidejs/splide";

export default class WorksSlider {
  constructor() {
    const root = document.querySelector<HTMLElement>(".business-works__slider");
    if (!root) return;

    new Splide(root, {
      type: "loop",
      focus: "center",
      autoWidth: true,
      autoplay: false,
      speed: 2000,
      updateOnMove: true,
      classes: {
        page: "splide__pagination__page business-works__page",
      },
      i18n: {
        prev: "前のスライド",
        next: "次のスライド",
        first: "最初のスライドへ",
        last: "最後のスライドへ",
        slideX: "%s枚目のスライドへ",
        pageX: "%sページ目へ",
        carousel: "カルーセル",
        select: "表示するスライドを選択",
        slide: "スライド",
        slideLabel: "%s / %s",
      },
    }).mount();
  }
}
```

- `i18n` は日本語にする。
- 実装例: `js/pages/business/_modules/works-slider.ts`

### スライドの幅・間隔・状態の見た目

- **スライドの幅は CSS で決める**場合は `autoWidth: true` にして、`.splide__slide` に `width` を指定する（PC / SP それぞれ）。スライド間の間隔は CSS の `margin-right` で指定すると、Splide が最初のスライドの `margin-right` を gap として読み取る。
- 非アクティブなスライドの縮小・透明度などは、`scale` / `transform` を **`.splide__slide`（li）にかけず、内側の要素にかける**。Splide は li の実寸で位置を計算するため、li を変形すると位置がずれる。アクティブ時は `.splide__slide.is-active .xxx__card` のように指定する。
- CSS の `transition` の時間は、Splide の `speed` と揃えるか、デザインに合わせて決める。移動の開始時点でアクティブ表示を切り替えたい場合は `updateOnMove: true` にする。
- `type: 'loop'` ではクローンのスライドにも `is-active` が付く。JS や確認で本物のスライドを取得するときは `.splide__slide--clone` を除外する。
- 縦書きなどで長いテキストがスライドの高さを伸ばす場合は、テキスト側に `max-height` を指定して、スライドの高さが画像で決まるようにする。

### 枚数や状態を監視して特別な処理をする場合

スライドの枚数や状態（現在のインデックス・移動中かなど）を監視して何か特別なことをする場合は、**原則として Splide インスタンスのコンポーネントを使用する**。DOM を自前で数えたり、`MutationObserver` などで監視したりしない。

- 参考: https://ja.splidejs.com/components/
- 例（Slides / Controller コンポーネント）:
  - `splide.Components.Slides.getLength(true)`: スライドの総数（クローン除外）
  - `splide.Components.Slides.get(true)`: スライドの配列（クローン除外）
  - `splide.Components.Controller.getIndex()`: 現在のインデックス（`splide.index` と同じ）
  - `splide.Components.Controller.getEnd()`: 到達可能な最後のインデックス
  - `splide.Components.Controller.isBusy()`: 移動中かどうか
- 状態の変化は `splide.on('mounted' | 'move' | 'moved' | 'pagination:mounted', ...)` で受け取る。

```ts
const splide = new Splide(root, options);

splide.on("mounted move", () => {
  const length = splide.Components.Slides.getLength(true);
  const index = splide.index;
  // length / index を使った処理
});

splide.mount();
```

### プログレスバー

参考: https://ja.splidejs.com/tutorials/carousel-progress/

スライダーの進み具合をバーで表示する場合は、Splide のコンポーネントから幅（%）を計算する。DOM を数えない。

- マークアップ: `.splide` の中で、track の外に `.xxx__progress` > `.xxx__progress-bar` を置く。装飾なので `aria-hidden="true"` を付ける。
- JS: `splide.on('mounted move', ...)` で `Controller.getEnd() + 1` と `splide.index` から幅を計算する。`type: 'loop'` でも `getEnd()` は「総数 − 1」なので、そのまま使える。`on` は `mount()` より前に登録する。
- CSS: バーの `width` に `transition` を指定し、時間とイージングは Splide の `speed` / `easing` と揃える。

```pug
.xxx__slider.splide(aria-label="スライダーの説明")
  .splide__track
    ul.splide__list
      //- スライド
  .xxx__progress(aria-hidden="true")
    .xxx__progress-bar
```

```ts
const bar = root.querySelector<HTMLElement>(".xxx__progress-bar");
const splide = new Splide(root, options);

splide.on("mounted move", () => {
  if (!bar) return;
  const end = splide.Components.Controller.getEnd() + 1;
  bar.style.width = `${100 * Math.min((splide.index + 1) / end, 1)}%`;
});

splide.mount();
```

```css
.xxx__progress {
  height: 2px;
  background: linear-gradient(#fff, #fff) center / 100% 1px no-repeat;
}

.xxx__progress-bar {
  width: 0;
  height: 2px;
  background-color: #ffaec4;
  transition: width 2.4s; /* Splide の speed と揃える */
}
```

- 実装例: `js/pages/index/_modules/works-slider.ts`、`pug/index.pug`

### 自動再生と Intersection

自動再生を ON にする場合の例（`npm install @splidejs/splide-extension-intersection` が必要。未インストールならユーザーに確認してから実行する）。

```ts
import Splide from "@splidejs/splide";
import { Intersection } from "@splidejs/splide-extension-intersection";

new Splide(root, {
  type: "loop",
  autoplay: "pause", // 画面外では停止し、Intersection の inView で再生する
  interval: 3000,
  speed: 1000,
  intersection: {
    inView: { autoplay: true },
    outView: { autoplay: false },
  },
}).mount({ Intersection });
```

- Intersection を使わない場合は `autoplay: true` と `interval` / `speed` を指定する。
- 自動再生なしの場合は `autoplay: false`。

## 5. 実装後の確認

- `_develop` で `npm run build:css` / `npm run build:js` がエラーなく通ること。
- Chrome DevTools MCP を接続している場合は、PC / SP の両方で以下を確認する（`frontend-coding` の「実装後の確認」も参照）。
  - 横スクロールが出ていないこと
  - 矢印・ページネーションで移動できること
  - ループ（`type: 'loop'` の場合）すること
  - アクティブなスライドの表示が切り替わること
  - 自動再生・Intersection を使う場合は、その挙動

## 6. React / Next のプロジェクト

参考: https://ja.splidejs.com/integration/react-splide/

### インストールと CSS

- `@splidejs/react-splide` をインストールする（ユーザーに確認してから実行）。
- 初期化用 CSS は `css/` に配置せず、**コンポーネント側で import** する。

```tsx
import { Splide, SplideSlide } from "@splidejs/react-splide";
import "@splidejs/react-splide/css/core";
```

- Next.js（App Router）では、Splide を使うコンポーネントに `'use client'` を付ける。
- 矢印・ページネーションを track の外に置くなど、構造を柔軟に組む場合は `hasTrack={false}` と `SplideTrack` を使う。

```tsx
import { Splide, SplideTrack, SplideSlide } from "@splidejs/react-splide";

<Splide hasTrack={false} aria-label="スライダーの説明">
  <SplideTrack>
    <SplideSlide>...</SplideSlide>
  </SplideTrack>

  <div className="splide__arrows">
    <button className="splide__arrow splide__arrow--prev">Prev</button>
    <button className="splide__arrow splide__arrow--next">Next</button>
  </div>
</Splide>;
```

- 拡張機能（Intersection など）は `extensions` に渡す: `<Splide extensions={{ Intersection }}>`

### 型情報の補完（tsx のプロジェクト）

tsx のプロジェクトでは `@splidejs/react-splide` の型が補完されない不具合がある。`package.json` の `exports` に `types` を追加するパッチを当てて対処する。

追加する内容（`node_modules` 内の `@splidejs/react-splide/package.json` の `exports["."]`）:

```json
"exports": {
  ".": {
    "types": "./dist/types/index.d.ts",
    "require": "./dist/js/react-splide.cjs.js",
    ...
  }
}
```

**pnpm を使用している場合**

1. `pnpm patch @splidejs/react-splide` を実行する。
2. 出力されたディレクトリ（`node_modules/.pnpm_patches` 内の splide ディレクトリ）の `package.json` を開き、上記の `types` を追加して保存する。
3. 出力された `pnpm patch-commit <パス>` をそのまま実行して保存する。
4. `pnpm-lock.yaml`（`patchedDependencies`）と `patches/` が更新されていることを確認する。

**npm を使用している場合**

npm には patch の仕組みが標準でないため、`patch-package` を使用する。

1. `npm install -D patch-package postinstall-postinstall` を実行する。
2. `package.json` の `scripts` に `"postinstall": "patch-package"` を追加する。
3. `node_modules/@splidejs/react-splide/package.json` に上記の `types` を追加する。
4. `npx patch-package @splidejs/react-splide` を実行し、`patches/` にパッチを生成する。

### イベントハンドリング（重要）

React では、通常の JavaScript / TypeScript 版 `Splide` で行う **`splide.on('event', callback)` のようなインスタンスから直接メソッドを呼ぶ記述は、原則禁止**。

`Splide` のライフサイクルやイベントを処理する場合は、必ず `Splide` コンポーネントの **Props として用意されているイベントハンドラー**を使う。Props 名は `on` + イベント名（キャメルケース、コロンは削除）の形式（例: `mounted` → `onMounted`、`move` → `onMove`、`arrows:mounted` → `onArrowsMounted`）。ハンドラーの第 1 引数は常に Splide インスタンス。

```tsx
const HandleSplideEvents = () => {
  const handleMounted = (splide: any) => {
    console.log("Splide がマウントされました", splide);
  };

  const handleMove = (splide: any, newIndex: number, prevIndex: number) => {
    console.log(`移動中: ${prevIndex} -> ${newIndex}`);
  };

  return (
    <Splide
      onMounted={handleMounted}
      onMove={handleMove}
      options={{ perPage: 2 }}>
      <SplideSlide>Slide 1</SplideSlide>
      <SplideSlide>Slide 2</SplideSlide>
    </Splide>
  );
};
```

- スライドの枚数や状態を監視して特別な処理をする場合も、ハンドラーで受け取った Splide インスタンスのコンポーネント（`splide.Components.Slides.getLength(true)` など）を使う。
- マウント後にインスタンスが必要な場合は `ref.current.splide` から取得できる（マウント前は `undefined`）。ただし、イベントの購読には使わない。
