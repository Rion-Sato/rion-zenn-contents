---
title: "手軽にWebサイトにgoogleマップを使うなら埋め込みが最適"
emoji: ""
type: "tech"
topics: [Google Map]
published: true
---
Next.jsフレームワークを使用したWebサイトにGoogle Mapを使いたいと思った時に、どうしたかの備忘録。

最初は、Google MapのAPIを使おうと思ったけど、Google Cloudの設定とか面倒で、別の方法を模索した結果、「埋め込み」という選択肢を見つけたので共有する。

API不要で、貼り付けるだけだから超楽だった。

# 「埋め込み」をする際に参考にしたサイト

Next.jsでも貼り付けるだけで普通に動いたから、めちゃくちゃ簡単。

https://ay-net.jp/homepage/201/

# 特殊な場合はAPIを使用すべき

大抵の場合は、「埋め込み」で十分だと思う。

なぜなら、普通は1つの場所だけを示すことが多いからだ。

しかし、何かしらの理由で複数箇所をピン留めして示す場合などはAPIを使用して、カスタマイズする必要がある。

一応、Next.jsフレームワークでAPIを使用する時に参考になりそうなサイトを貼っておきます。

https://zenn.dev/kou_kawa/articles/11-next-ts-googlemap