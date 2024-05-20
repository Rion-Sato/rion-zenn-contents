---
title: "リッチテキストにTailwindCSSを当てる時の最適解"
emoji: ""
type: "tech"
topics: [TailwindCSS, リッチテキスト, microCMS]
published: true
---

microCMSのリッチテキストエディタを使用したブログ記事にTailwindCSSを当てる際、苦労した点を共有します。

最初は「TailwindCSS / Typographyプラグイン」を使用しましたが、この方法だと細かいところまで手が届かなかったので、その解決法を紹介します。

# 前提条件

- フレームワークにNext.js（App Router）を使用
- CSSフレームワークにTailwindCSSを使用

# 最適解

結論から言うと**リッチテキストには@layerでカスタムクラスを定義**することが最適解だと思いました。

以下で詳しく説明します。

## 1. リッチテキストをHMTL（JSX）に挿入する

以下のような、HTML（JSX）を想定します。

今回は、リッチテキストを`dangerouslySetInnerHTML`で挿入し、その`div`タグに`className="prose"`を設定しました。

```ts
import { createClient } from 'microcms-js-sdk';

// ブログの型定義
export type Blog = {
  title: string;
  description: string;

  // リッチテキストはcontentに格納されている
  content: string;
  thumbnail?: MicroCMSImage;
  tags?: Tag[];
  writer?: Writer;
};

// Initialize Client SDK.
export const client = createClient({
  serviceDomain: process.env.MICROCMS_SERVICE_DOMAIN,
  apiKey: process.env.MICROCMS_API_KEY,
});

// ブログの詳細を取得
export const getDetail = async (contentId: string, queries?: MicroCMSQueries) => {
  const detailData = await client
    .getListDetail<Blog>({
      endpoint: 'blog',
      contentId,
      queries,
    })
    .catch(notFound);

  return detailData;
};

// リッチテキストエディタを使用する画面レイアウト
export default const Page: FC = () => {
  const blog = await getDetail(contentId);

  return (
    <div 
      className="prose"
      dangerouslySetInnerHTML={{ __html: blog.content }}
    />
  )
}
```

## 2. `@layer`でカスタムクラス（スタイル）を定義する

上記でリッチテキスト部分に`prose`というクラス名を適応させたので、生のCSSのように`.prose`で定義していきます。

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer components {
  .prose {
    @apply text-base font-bold
  }
  .prose h1 {
    @apply text-xl
  }
  /* ... */
}
```

### @layer（base, components, utilities）とは？

公式ドキュメントがわかりやすいので、参考にしてください。

https://tailwindcss.com/docs/adding-custom-styles

### @applyとは？

簡単にいうと、生のCSSではなく、TailwindCSSの記法をそのまま使用するためのディレクティブです。

https://tailwindcss.com/docs/functions-and-directives

# カスタムクラスを定義するメリット

メリットとして以下の3つが考えられます。

1. リッチテキスト内の珍しいタグや独自のクラス名にもCSSを当てることができる
2. TailwindCSSの記法を使用できる
3. 別でCSSファイルを用意する必要がない

これが最適解だと思う理由です。

## 1. 珍しいタグや独自のクラス名にもCSSを当てられる

これは特に、microCMS + TailwindCSSで実装している人にとっては大きいメリットだと思います。

その理由がわかりやすいように、手軽にリッチテキストにCSSを当てられると有名な「Typographyプラグイン」を使用した場合と比較します。

### 想定するリッチテキスト（HTML）

実際にmicroCMSのリッチテキスト内で埋め込みを使用すると以下のようなコードが生成されます。

```
<div style="left: 0; width: 100%; height: 0; position: relative; padding-bottom: 56.25%;">
 <iframe src="https://www.youtube.com/embed/omKyoU5Ia04?rel=0" style="top: 0; left: 0; width: 100%; height: 100%; position: absolute; border: 0;" allowfullscreen scrolling="no" allow="accelerometer; 
 clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share;">
 </iframe>
</div>
```

### Typographyプラグインを使用した場合

Typographyプラグインでは、`iframe` タグ自体にCSSを当てられない設定になっています。

よって、上記のコードに**CSSを当てることができません**。

CSSを当てることができるタグ（Element modifiers）は以下の公式GitHubに記載されています。

https://github.com/tailwindlabs/tailwindcss-typography?tab=readme-ov-file#element-modifiers

### カスタムクラスを定義した場合

以下のコードのように、自分でCSSを当てたいタグも定義できるので、上記のコードに**CSSを当てることが可能**です。

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer components {
  .prose {
    @apply text-base font-bold
  }
  // リッチテキスト（proseクラス）内のiframeタグにcssを適応
  .prose iframe {
    @apply my-6
  }
  /* ... */
}
```

## 2. TailwindCSSの記法を使用できる

先ほど紹介した`@apply`ディレクトリを使用することによって、TailwindCSSの記法をそのまま利用できます。

これによって、TailwindCSSが用意してくれている既存のCSSを適応できるため、**爆速で開発**ができます。

## 3. 別でCSSファイルを用意する必要がない

これは、TailwindCSS自体の特徴でもありますが、カスタムクラスを定義することで、独自のクラス名にも対応しつつ、`module.css`などの別のCSSファイルを用意する必要もありません。

# 最後に

僕は、この解決に合計で1ヶ月近くかかったと思います。

なので、この記事が同じような悩みを抱える人に届けば幸いです。（そもそもTailwindCSSをしっかり理解している人はこんなことにはならないと思いますが）

また、この経験を通してTailwindCSSの可能性を実際に感じることができました。

一見、生のCSSより拡張性がないように見えますが、カスタマイズをすることによって拡張性・利便性の両方で生のCSSを凌駕できると感じました。

# 参考サイト

https://zenn.dev/nrikiji/articles/f4c72668277bd8

https://qiita.com/hirothugu326/items/f26a8ad850756f7271d2
