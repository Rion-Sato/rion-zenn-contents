---
title: "Next.js（App Router）におけるSEO対策"
emoji: ""
type: "tech"
topics: [Next.js, SEO]
published: false
---
「『Next.jsはSEOに強い』って言われるけど、具体的にSEO対策ってどうやるの？」って思って調べて実装した備忘録です。

正直、3ヶ月前にフロントエンドを勉強し始めた私。

流れでNext.jsを勉強してきて、Next.jsを採用するだけでSEO対策いいの？って感じで調べてみると意外とやることあったので、参考にしたサイトをまとめて見ました。

# SEOって具体的に何すればいいの？

- SSG（任意）
- 構造化データ
- パンくずリスト（任意）
- サイトマップ
- メタデータ・OGP
- robots.txt

※それぞれの意味は、ググると出てくるので各自調べてください。

参考サイト

https://qiita.com/kitao6/items/02989a72b9ecd37785e8

https://zenn.dev/temasaguru/articles/641a10cd5af02a

# それぞれの具体的な実装方法

※今回、任意の項目に関しては扱いませんので、各自必要に応じてお調べください。

※大前提として、Next.jsの公式ドキュメント自体がわかりやすく作成されているので、今回の参考サイトには含めていません。

## 構造化データ

※これに関しては、[公式ドキュメント](https://nextjs.org/docs/app/building-your-application/optimizing/metadata#json-ld)が最適解だと思うので、以下のサイトは事例程度に思ってください。

https://inari-tech.net/posts/nextjs-jsonld

https://khsmty.com/article/nextjs_jsonld/

## サイトマップ

https://blog.kimizuka.org/entry/2023/06/27/132251

https://tatsumiyamamoto.com/articles/next13-sitemap

## メタデータ・OGP

https://zenn.dev/temasaguru/articles/641a10cd5af02a

※favicon.icoを作成するには、このサイトがおすすめです！

https://favicon-generator.mintsu-dev.com/

## robots.txt

https://blog.kimizuka.org/entry/2024/03/14/130751

