---
title: "Next.js（App Router）+ microCMSで埋め込みが初回レスポンス時以外は正常に表示されない"
emoji: ""
type: "tech"
topics: [Next.js, microCMS, JavaScript, React, 初心者]
published: false
---

Next.js（App Router）製のブログでmicroCMSの埋め込みリンク（iframeタグ）を使おうとして、沼にハマった時の解決策です。

この解決に僕は１週間もかかってしまい、皆さんにはそんな思いをしてほしくないので、共有します。

あと、最後にちょっといい話もあるので、ぜひ読んでみてください。


# 問題

microCMSの埋め込みリンク（iframeタグ）が初回レスポンス時以外は正常に表示されないこと

# 原因

原因は２つ考えられます。

- `//cdn.iframe.ly/embed.js`が読み込まれていない
- scriptタグがレンダレングごとに実行されていない

## `//cdn.iframe.ly/embed.js`が読み込まれていない

そもそもmicroCMSの埋め込み（iframeタグ）に関するjsファイルが読み込まれていないと動作しません。

なので、`script`タグなどで`//cdn.iframe.ly/embed.js`を読み込む必要があります。

しかし、ReactやNext.jsといったフレームワークを使用している場合は、これでも正常に動作しませんので、次の原因を疑ってみてください。

## scriptタグがレンダレングごとに実行されていない

`//cdn.iframe.ly/embed.js`を`script`タグに書き込んだとしても、ReactやNext.jsといったフレームワークを使用している場合は正常に動作しません。

それは、ReactやNext.jsなどが仮想DOMを使用しているため、デフォルトでは「動的に`script`タグが生成され、外部スクリプトが読み込まれる」ことがないのが理由です。

なので、`useEffect()`などを使用して、動的に`script`タグが生成し、外部スクリプトが読み込む必要があります。

# 解決策

解決策は以下のように実装するだけです。

```ts:page.tsx
'use client';

import { useEffect ) from 'react';

export default const Page: FC = () => {
　　　　 useEffect(()=>{
     // scriptを読み込み
     const script = document.createElement('script');
     script.src = "//cdn.iframe.ly/embed.js";
     document.body.appendChild(script);
     // アンマウント時に一応scriptタグを消しておく
     return () => {
       document.body.removeChild(script);
     }
  }, [])

// iframeタグの入ったJSX
  return (
     <iframe
      ...
     />
  )
}
```

これで完了です。

# ちょっといい話

いい話をする前に、「この実装に１週間もかかったの？」と思われた方がいるかもしれませんが、そうです。

僕のReactの勉強不足だったことは否めません。

ですので、あまり助けになっていないかもしれません。

ただ、この経験を通して、技術的な部分以外で大きな学びがありました。

それは、**エンジニアコミュニティは意外と優しい**ってことと**どんどん行動しよう**ってことです。

なぜ急にこんなことを言い出しかというと、今回の解決策のヒントはmicroCMSのCXOである[**柴田和祈**](https://twitter.com/shibe97)さんにXのDMで教えていただいたからです。

その際は、柴田さんありがとうございました！（この記事を読んでくださっていたら）

あと、ブログサイトのデザインパクってすいません🙇（あまりにも魅力的だったので）

話を戻して、なぜこの話をしたかというと、かなり有名な企業のCXOの方でさえ、XのDMで聞いてみる（自分から行動する）ことさえすれば答えてくれるっていうことを伝えたかったからです。

この経験から、上記のことを学ぶことができました。

この学びが、僕自身も含め、この記事を見てくださった皆さんが共に成長するための手助けになれば幸いです😊

# 参考サイト

https://zenn.dev/catnose99/articles/329d7d61968efb