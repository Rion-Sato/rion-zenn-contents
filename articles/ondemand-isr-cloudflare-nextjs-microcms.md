---
title: "【Cloudflare編】Next.js（App Router）+ microCMS でSSGすると、記事が更新されない"
emoji: ""
type: "tech"
topics: [Next.js, Cloudflare, microCMS, On-Demand ISR, SSG]
published: false
---
前回の[【Vercel編】Next.js（App Router）+ microCMS でSSGすると、記事が更新されない](https://zenn.dev/rion_freelance/articles/ondemand-isr-vercel-nextjs-microcms)の続編です。

必ず、【**Vercel編**】を読んでからこの記事を読んでください🙇（解決しないので）

**Cloudflare**をホスティングサービスに使ってる人はぜひ読んでみてください。

# 原因

原因は２つあると考えられます。（仮説）

1. **Cloudflare PagesがOn-demand Revalidationに対応していない**
2. **キャッシュシステムに問題がある**

## **Cloudflare PagesがOn-demand Revalidationに対応していない**

Cloudflare Pagesでは、`@cloudflare/next-on-pages`を使用してビルドするわけですが、この仕組み上、revalidatePathやrevalidateTagなどのOn-demand Revalidationが働かないようです。

詳細については、以下のサイトが参考になるので、気になる方は読んでみてください。

https://github.com/cloudflare/next-on-pages/issues/292

## **キャッシュシステムに問題がある**

まず、Cloudflare Pagesでは**デフォルト**のキャッシュのストレージオプションとして、「**Cache API**」というものが使用されいます。

`@cloudflare/next-on-pages` + On-demand Revalidationを試したことのある人ならわかると思いますが、このCache APIでOn-demand Revalidationの設定をすると、サイトへのアクセスによって様々なデータが取得・表示されてしまいます。

具体的には、microCMSなどで記事の更新を行なっても、更新前の記事一覧が表示されたりするということです。

つまり、このキャッシュシステムに問題がある可能性があります。

※Cloudflare Pagesが提供するキャッシュのためのストレージオプションについては、以下の公式ドキュメントを参考にしてください。

https://github.com/cloudflare/next-on-pages/blob/main/packages/next-on-pages/docs/caching.md

# 解決策

結論から言うと、「**Workers KVにキャッシュ先を変更する**」ことです。

**Workers KV**とは、先ほど説明したCache APIと並ぶ、Cloudflare Pagesが提供するキャッシュのためのストレージオプションの1つです。

先述したように、原因（仮説）が２つ考えられましたが、１つ目の「Cloudflare PagesがOn-demand Revalidationに対応していない」ことに関しては、Cloudflareを作る側ではないので解決できません。

よって、２つ目の「キャッシュシステムに問題がある」ことについての考えた結果、この結論に至りました。

## なぜ、キャッシュ先を変えると解決するの？

簡単にいうと、「**最新のデータが絶対に取得できるようになるから**」です。

詳しい理由は、以下の通りです。

※長いので、解決策の具体的な実装方法だけ知りたい人は次の**実装**まで飛ばしてください。

### Cache APIとWorkers KVの動作の違い

1. Cache APIの場合
   - Cache APIは、リクエストのURLをキーとしてキャッシュを管理します。
   - revalidateTagを使用してキャッシュを更新する際、Cache APIはキャッシュのパージや更新を非同期的に行います。
   - キャッシュの更新リクエストが発生しても、すでにキャッシュされているデータは即座には無効化されません。
   - キャッシュの更新が完了するまでの間、古いキャッシュデータが返される可能性があります。
   - そのため、アクセスごとに異なるデータ（古いキャッシュデータと新しいデータ）が取得される可能性があります。

2. Workers KVの場合
   - Workers KVは、明示的に指定したキーに対応する値を保存および取得します。
   - revalidateTagを使用してキャッシュを更新する際、Workers KVはキーに対応する値を即座に更新します。
   - キャッシュの更新リクエストが発生すると、Workers KVは即座に新しいデータを保存し、古いデータを上書きします。
   - 次のアクセスからは、常に更新された最新のデータが取得されます。
   - Workers KVではキャッシュの更新が同期的に行われるため、アクセス間でデータの不整合が発生しません。

### まとめ

- Cache APIは、キャッシュの更新を非同期的に行うため、更新が完了するまでの間は古いキャッシュデータが返される可能性があります。これにより、アクセスごとに異なるデータが取得される可能性があります。
- 一方、Workers KVは、キーに対応する値を即座に更新するため、キャッシュの更新が同期的に行われます。更新リクエストが発生すると即座に新しいデータが保存され、次のアクセスからは常に最新のデータが取得されます。

### 参考サイト

https://zenn.dev/naporin24690/scraps/00e8ee1d262caf

# 実装

## 1. KVの名前空間を作成する

1. **Cloudflare メインページ ＞ Workers & Pages ＞ KV ＞ 名前空間を作成する** の順に移動
2. 以下の画像のように、「名前空間の名前」の部分に、任意の空間名を入力（適当で大丈夫です）
3. 「**追加**」をクリック（完了）

![KVの名前空間を作成](https://storage.googleapis.com/zenn-user-upload/428ad6b627b1-20240519.png)

## 2. Cloudflare Pagesとの紐付け

1. **Workers & Pages ＞ あなたのWebサイト ＞ 設定 ＞ Functions ＞ KV 名前空間のバインディング** の順に移動
2. 「**バインディングを編集**」をクリック
3. 「変数名」を`__NEXT_ON_PAGES__KV_SUSPENSE_CACHE`に設定（**絶対！**）
4. 「KV 名前空間」を先ほど作成した名前空間を設定
5. 「**保存**」をクリック（完了）

![Cloudflare Pagesとの紐付け](https://storage.googleapis.com/zenn-user-upload/75843d28e043-20240519.png)

## 3. 再デプロイ

### 以上で実装完了です！

# 最後に

長い長い実装、お疲れ様でした。

これで、当初の要件を満たして理想通りに動いていることを願ってます。

# 参考サイト

https://community.cloudflare.com/t/nextjs-serving-stale-old-fetch-cache-across-network-how-to-clear-cache/646169

