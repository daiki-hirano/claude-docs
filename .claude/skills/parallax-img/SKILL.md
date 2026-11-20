---
name: parallax-img
description: スクロールに合わせて画像を動かす画像パララックス（GSAP ScrollTrigger）を実装するときに使用する。「パララックス」「parallax」「スクロールに合わせて画像を動かす」と依頼されたとき、Figma やデザインの指示にパララックスがあるときは必ず使用する。Pug + PostCSS と React / Next の両方に対応する。
---

# 画像パララックス実装ルール

枠の中の画像を、スクロールに合わせて縦に動かす（画像パララックス）ときのルール。Pug + PostCSS のプロジェクトは 2〜6 章、React / Next のプロジェクトは 7 章に従う。

- 通常版のソースは `_develop/src/` 配下。以下のパスは `_develop/src/` からの相対パスで記載する。
- ファイルの配置・Pug・CSS・画像などの基本ルールは `frontend-coding` スキルに従う。
- 公式ドキュメント: https://gsap.com/docs/v3/Plugins/ScrollTrigger/

## 1. 実装前に確認すること

### プロジェクトの種類とパッケージ

- `package.json` を確認し、プロジェクトの種類を判定する。
  - `react` / `next` がある → React 版（7 章）
  - Pug + PostCSS → 通常版（2〜6 章）
- dependencies に `gsap` が入っているか確認する。**未インストールの場合は、インストールしてよいかユーザーに確認してから実行する**（勝手にインストールしない）。
  - 通常版: `npm install gsap`
  - React 版: `gsap` と `@gsap/react`（パッケージマネージャーはロックファイルで判定する）

### 仕組み

- 枠に `data-parallax-img` を付ける。枠は `overflow: hidden`。
- 中の img を CSS で枠より高くする（`height: 115%`、`object-fit: cover`）。
- GSAP の ScrollTrigger（`scrub`）で、img を「(img の高さ − 枠の高さ) だけ上にずらした位置」から `y: 0` へ動かす。スクロールより少し遅れて動いて見える。

### 動く量は CSS だけで決まる

- JS に px や % を書かず、`offsetHeight` の差を関数で求める。`invalidateOnRefresh: true` で、リサイズ時に再計算する。
- 動く量を変えたいときは、CSS の `height` の % を変えるだけ。動く量は「枠の高さ × (height の % − 100%)」。
  - `115%` → 枠の高さの 15%
  - `120%` → 枠の高さの 20%
- px で固定しない。ビルドで px が rem や vw に変換されるなど、画面幅で枠の高さが変わる環境では、固定の px だと隙間が見えることがある。

### ユーザーに指定がない場合に確認すること

プロンプトやデザインに指定がない項目は、`AskUserQuestion` で確認する（推測で決めない）。

- 動く量（`height` の %）。指定がなければ 115% から始める。
- SP でも動かすか（既定は PC / SP 共通）。
- `prefers-reduced-motion` で止めるか（プロジェクトの方針に合わせる）。

## 2. CSS（Pug + PostCSS のプロジェクト）

`css/_modules/_common/_parallax-img/_parallax-img.css` を作成し、`css/common.css` の `common` レイヤーで import する。

```css
/* css/common.css */
@import "./_modules/_common/_parallax-img/_parallax-img.css" layer(common);
```

```css
/* 画像パララックスの枠と img（js/utils/ui/parallax-img.ts）
   img は枠の 115% の高さにして、はみ出した分を JS でスクロールに合わせて縦に動かす */
@media (--pc) {
  [data-parallax-img] {
    overflow: hidden;

    img {
      display: block;
      width: 100%;
      height: 115%;
      max-width: none;
      object-fit: cover;
      will-change: transform;
    }
  }
}

@media (--sp) {
  [data-parallax-img] {
    overflow: hidden;

    img {
      display: block;
      width: 100%;
      height: 115%;
      max-width: none;
      object-fit: cover;
      will-change: transform;
    }
  }
}
```

- PC / SP で同じ値でも、`frontend-coding` のルールどおり `@media (--pc)` / `@media (--sp)` の両方に書く。動く量を PC / SP で変えたいときは、それぞれの `height` を変える。
- 枠の高さは、枠側のクラスで指定する（例: `aspect-ratio` や `height`）。

## 3. JavaScript（TypeScript）

### ファイル構成

複数ページで使うため `js/utils/ui/parallax-img.ts` に作成する。

```ts
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
gsap.registerPlugin(ScrollTrigger);

/**
 * 画像パララックス
 * - 枠（overflow: hidden）の data-parallax-img の中の img を動かす（img は CSS で枠より高くしてある）
 * - 動く量は CSS の高さの差から求める（リサイズ時は invalidateOnRefresh で再計算）
 * - 動かすのは transform（y）のみ。img に scale / translate は指定しない（別の要素に指定する）
 * - trigger は枠自身。属性に値があるときは、そのセレクターで closest() した祖先（複数の枠を同期できる）
 * - スライダーのクローンを含めるため、クローン作成後に生成する
 *
 * 【pug】
 * figure(data-parallax-img)
 *   +img(...)
 *
 * .wrap
 *   figure(data-parallax-img=".wrap")
 *   figure(data-parallax-img=".wrap")
 */
export default class ParallaxImg {
  constructor() {
    const frames = document.querySelectorAll<HTMLElement>(
      "[data-parallax-img]",
    );

    frames.forEach((frame) => {
      const img = frame.querySelector<HTMLElement>("img");
      if (!img) return;

      const selector = frame.dataset.parallaxImg;
      const trigger =
        (selector && frame.closest<HTMLElement>(selector)) || frame;

      gsap.fromTo(
        img,
        { y: () => -(img.offsetHeight - frame.offsetHeight) },
        {
          y: 0,
          ease: "none",
          scrollTrigger: {
            trigger,
            start: "top bottom",
            end: "bottom+=50% top",
            scrub: true,
            invalidateOnRefresh: true,
          },
        },
      );
    });
  }
}
```

### main.ts への登録

`js/main.ts` で import し、`Main` のコンストラクターで生成する。**ページ用 JS の初期化後に生成する**（スライダーなどで作られるクローンも対象にするため）。

```ts
import ParallaxImg from "./utils/ui/parallax-img";

// スライダーのクローンを含めるため、ページ用 JS の初期化後に生成する
getComponent().then(() => new ParallaxImg());
```

- 慣性スクロールのライブラリ（Lenis など）を使っている場合は、ScrollTrigger と連携させる（`lenis.on('scroll', ScrollTrigger.update)` など）。

### ScrollTrigger の設定

- `start: 'top bottom'`: trigger の上端が画面の下端に来たときに開始する（画面に入った瞬間）。
- `end: 'bottom+=50% top'`: trigger の下端から trigger の高さの 50% 下の位置が、画面の上端に来たときに終了する。枠が画面から出たあとも少し動き続けるため、スクロールより遅れて見える。
- `scrub: true`: スクロール量にそのまま連動させる。
- `start` / `end` はデザインに合わせて調整してよい（動く速さが変わる）。`end` を短くすると速く、長くすると遅く動く。

## 4. Pug での使い方

枠に `data-parallax-img` を付け、中に `+img` / `+pic` を入れる（`img` を直接書かない）。

```pug
figure.section__image(data-parallax-img)
  +img('images/xxx/photo.jpg', '画像の説明')
```

複数の枠を同じきっかけで動かす場合は、属性に祖先のセレクターを渡す。

```pug
.wrap
  figure.section__image(data-parallax-img=".wrap")
    +pic('images/xxx/photo.jpg', 'images/xxx/photo_sp.jpg', '画像の説明')
  figure.section__image(data-parallax-img=".wrap")
    +pic('images/xxx/photo2.jpg', 'images/xxx/photo2_sp.jpg', '')
```

- 枠の中に置く img は 1 枚（`picture` を使う場合も中の `img` が動く）。

## 5. 注意点とオプション

### 注意点

- **img に `scale` / `translate` / `rotate` などの transform 系を指定しない**。GSAP が transform にまとめて書き込むため壊れる。拡大などの演出は、`picture` や枠など別の要素にかける。
- 枠の `overflow: hidden` と、`clip-path` / `mask` による出現アニメーションを併用する場合は、枠と img で要素を分ければぶつからない（出現アニメーションは枠の外側の要素、または枠側に付ける）。
- スライダー（Splide の `loop` など）でクローンが作られる場合は、**クローン作成後に初期化する**。クローンの前に生成すると、クローン内の img が動かない。
- `height` を変えずに img だけを大きくしない。動く量が CSS の高さの差で決まるため、上下に隙間が出る。

### オプション: SP で止める

`gsap.matchMedia()` で条件を付ける。止めている間は、CSS の `height: 115%` のまま、`object-fit: cover` で枠に収まる。

```ts
const mm = gsap.matchMedia();

mm.add("(min-width: 768px)", () => {
  // frames.forEach(...)（上記の gsap.fromTo）
});
```

- 幅の条件は、`css/_config/_media.css` の `--pc` / `--sp` と合わせる（この例は SP が 767px 以下の場合）。

### オプション: `prefers-reduced-motion` で止める

プロジェクトの方針に合わせる。止める場合は `no-preference` のときだけ動かす。

```ts
const mm = gsap.matchMedia();

mm.add("(prefers-reduced-motion: no-preference)", () => {
  // frames.forEach(...)
});
```

- SP で止める条件と併用する場合は、`mm.add({ pc: '(min-width: 768px)', motion: '(prefers-reduced-motion: no-preference)' }, (ctx) => { if (!ctx.conditions?.pc || !ctx.conditions.motion) return; ... })` のように条件を組み合わせる。

## 6. 実装後の確認

- `_develop` で `npm run build:css` / `npm run build:js` がエラーなく通ること。
- Chrome DevTools MCP を接続している場合は、PC / SP の両方で以下を確認する（`frontend-coding` の「実装後の確認」も参照）。
  - スクロール中、枠の上下に隙間が見えないこと（枠が画面に入った直後と、出る直前の両方）
  - ウィンドウをリサイズしたあとも、隙間が出ず、動く量が正しいこと
  - クローンを持つスライダーの場合は、クローンのスライドの画像も動くこと
  - 同期させた枠が、同じタイミングで動くこと
  - 画像に `scale` などの transform を指定していないこと（指定している場合は別の要素に移す）
  - `prefers-reduced-motion` や SP で止める場合は、その挙動

## 7. React / Next のプロジェクト

参考: https://gsap.com/resources/React/

### インストールと CSS

- `gsap` と `@gsap/react` をインストールする（ユーザーに確認してから実行）。
- CSS は 2 章と同じ考え方で、枠は `overflow: hidden`、img は `height: 115%` / `object-fit: cover` にする。プロジェクトの CSS の方式（CSS Modules など）に合わせて書く。
- 動く量は CSS の高さの差で決める。JS に px や % を書かない。

```css
.parallax-img {
  overflow: hidden;
}

.parallax-img img {
  display: block;
  width: 100%;
  height: 115%;
  max-width: none;
  object-fit: cover;
  will-change: transform;
}
```

### コンポーネント

- Next.js（App Router）では `'use client'` を付ける。
- `useGSAP` を使うと、アンマウント時にアニメーションと ScrollTrigger が自動で `revert()` される。`scope` に枠の ref を渡す。

```tsx
"use client";

import { useRef, type ReactNode } from "react";
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
import { useGSAP } from "@gsap/react";

gsap.registerPlugin(ScrollTrigger, useGSAP);

type Props = {
  children: ReactNode; // 中に img（next/image など）を 1 枚入れる
  trigger?: string; // 同期させたい祖先のセレクター（省略時は枠自身）
  className?: string;
};

export default function ParallaxImg({ children, trigger, className }: Props) {
  const frameRef = useRef<HTMLDivElement>(null);

  useGSAP(
    () => {
      const frame = frameRef.current;
      const img = frame?.querySelector("img");
      if (!frame || !img) return;

      gsap.fromTo(
        img,
        { y: () => -(img.offsetHeight - frame.offsetHeight) },
        {
          y: 0,
          ease: "none",
          scrollTrigger: {
            trigger: (trigger && frame.closest(trigger)) || frame,
            start: "top bottom",
            end: "bottom+=50% top",
            scrub: true,
            invalidateOnRefresh: true,
          },
        },
      );
    },
    { scope: frameRef },
  );

  return (
    <div ref={frameRef} className={`parallax-img ${className ?? ""}`}>
      {children}
    </div>
  );
}
```

- `useGSAP` を使わず `useEffect` で書く場合は、`gsap.context(() => { ... }, frameRef)` を作り、クリーンアップで `ctx.revert()` を返す。

```tsx
useEffect(() => {
  const ctx = gsap.context(() => {
    // gsap.fromTo(...)
  }, frameRef);
  return () => ctx.revert();
}, []);
```

- img に `scale` などの transform 系を指定しない（5 章と同じ）。拡大などは枠の外側や別の要素にかける。
- SP で止める・`prefers-reduced-motion` で止める場合は、5 章と同じく `gsap.matchMedia()` を `useGSAP` の中で使う。
- 確認項目は 6 章と同じ。
