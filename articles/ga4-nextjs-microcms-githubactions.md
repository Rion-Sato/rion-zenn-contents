---
title: "GA4 × Next.js（App Router）× microCMS × GitHub Actionsで人気記事を自動更新する"
emoji: ""
type: "tech"
topics: [Google Analytics, Next.js, microCMS, GitHub Actions]
published: true
---
このブログに人気記事一覧を表示できるようにしました。

以下で、流れを解説します。

# 前提条件

- コンテンツ管理にはmicroCMSを利用している
- アナリティクスにはGoogle Analytics 4を利用している
- ソースコードをGitHubで管理している
- フロントエンドはSSG or ISRしている

# 実装

実装に関しては以下の２つの記事がわかりやすいので、参考にしてみてください。

https://www.mythinkings.net/microcms-ga4-analytics

https://zenn.dev/frontendflat/articles/b195884617dcee

# 注意点

それは、「**上記の記事のコードをそのまま実装するだけでは、機能しない**」ということです。

特に、上記の記事の`.github/actions/ranking/index.js`ファイルの`dimensionFilter`という部分については、**あなたのサイトに合わせてコードを書く必要**があります。

その理由となる前提知識は以下のとおりです。

## 前提知識（dimensionFilterについて）

### 1. `fieldName: 'pagePath'`によって取得される値について

前提として、`dimensionFilter`の`filter`の`fieldName: 'pagePath'`が使用されているので、`response.rows.dimensionValues`には以下のように値が格納されます。

例えば、`https://www.example.com/store/contact-us?query_string=true`のpagePathの部分は`/store/contact-us`になり、これが`response.rows.dimensionValues`に格納されます。

参照：[https://developers.google.com/analytics/devguides/reporting/data/v1/api-schema?hl=ja](https://developers.google.com/analytics/devguides/reporting/data/v1/api-schema?hl=ja)

### 2. `stringFilter`をつけるとどうなるか

```js
dimensionFilter: {
  filter: {
    stringFilter: {
      value: "/store/",
      matchType: "CONTAINS",
    },
    fieldName: "pagePath",
  },
}
```

例えば、`https://www.example.com/store/contact-us?query_string=true`というURLを使用した場合、

上記のコードのようにコードを書くと、上記で解説したfieldName: 'pagePath'によって、/store/contact-usが取得され、

さらに、stringFilterによって、contact-usという部分がresponse.rows.dimensionValuesに格納されます。

valueやmatchTypeなどの引数について詳しく知りたい方は以下の公式ドキュメントを参考にしてください。

https://developers.google.com/analytics/devguides/reporting/data/v1/rest/v1beta/FilterExpression?hl=ja#StringFilter

**この前提知識をもとに、あなたのサイトに合わせて実装を行なってみてください！**

# 最後に

人気記事一覧の実装お疲れ様でした。

これで、少しは実用的なブログサイトに近づけたと思います。

つよつよエンジニアの人からしたら「こんなの楽勝だよ」って思われるかもしれませんが、僕は初心者なので苦労しました。

その苦労した点（注意点）について、今回は記事を書いてみました。

初学者の方々の役に立っていると嬉しいです。

僕自身もこの実装を通して、「公式ドキュメントを読む大切さ」と「真似するだけでなく自分で考えるプログラミングの面白さ」を知ることができ、少しは成長できたかなと思います。