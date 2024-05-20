---
title: "【Vercel編】Next.js（App Router）+ microCMS でSSGすると、記事が更新されない"
emoji: ""
type: "tech"
topics: [Next.js, Vercel, microCMS, On-Demand ISR, SSG]
published: true
---
Next.js（App Router）+ microCMS + SSG で[ブログ](https://ribrary.uk)を作った際に出くわした『記事が更新されない！』問題の解決策の備忘録です。

僕もWeb開発を始めて4ヶ月弱の初心者なので、同じ悩みを抱えてる初心者の人の助けになれば幸いです。

# 原因

それは、**App RouterでSSGを実装すると、キャッシュが更新されない**ことにあります。

その理由は、以下で図解します。

## SSGにおけるPages RouterとApp Routerの違い

- **Pages Router（SSG）**

以前のNext.jsのルーティング方法であるPages Routerの時は、下図のようにできた。

これが理想ですよね。

![Pages Router](https://storage.googleapis.com/zenn-user-upload/5d3b9e713b1c-20240518.png)

- **App Router（SSG）**

しかし、App Routerでは、**キャッシュがデプロイにまたがって永続化されるため、再デプロイしても、コンテンツは更新されない**のです。（下図）

![App Router](https://storage.googleapis.com/zenn-user-upload/e7afc47bfc69-20240518.png)

※公式ドキュメントからの引用↓
> Next.js has a built-in Data Cache that persists the result of data fetches across incoming server requests and deployments. This is possible because Next.js extends the native fetch API to allow each request on the server to set its own persistent caching semantics.

# 要件定義

解決策を提示する前に、**実現したいこと**をおおまかに整理しておきましょう。

- **microCMSでデータ更新時のみビルドする**（だから SSG + WebHooks にしたんですよね？）
- **上記のタイミングで再デプロイした時に、キャッシュも更新**（＝サイトも更新）

※厳密にいうと、今回の解決策は、ホスティングサービスによるビルドに関係なく、Next.js自体のキャッシュを更新するWebHooksを使用しているので、再ビルドしなくても記事が更新されます。

# 解決策

結論から言うと、**On-demand Revalidationを使用してキャッシュの更新を行う**ことです。

詳細については以下で詳しく説明します。

**ただし**、ここからが長いので、「そんなのいいから、早く実装方法を教えてくれよ」って人は**実装方法**の部分まで飛ばしてください。

でも、ここからがプログラミングの面白いところですよ！（初心者の僕が言うのはなんですが）

なぜなら、プログラミングは原因・構造から考えて解決できるからです。

今回でいえば、以下の思考プロセスをたどりました。（実際は調べるだけでなく、ローカル環境で実装して試したりもしましたが）

## 思考プロセス

1. 「キャッシュが更新されない」（原因）
2. 「microCMSの更新に伴って、キャッシュを更新する（or削除＋再生成する）方法ないの？」（調べる）
   ※ここの調べ方は頑張りどころかも
3. 「どうやらNext.jsのキャッシュには『再検証』できる機能があるらしい（これを『Revalidation』という）」（糸口発見？）
4. 「Revalidationってどうやって実現するの？」（調べる）
5. 「SSGじゃなくてISRっていうレンダリング方法で実装できる」（解決したか？）
6. 「でも、ISRって『時間指定』でのビルドしかできないのか？（revalidateっていう引数で指定）」（要件に合わないな〜）
   ※ほとんどの記事だとrevalidateについて書かれてるから
7. 「こっちは、microCMSでデータ更新時のみビルドして欲しいんだけどな」（また調べる）
8. 「On-demand Revalidationっていう方法があるぞ！（特定のタイミングで再検証する方法）」（解決策発見！）

※上記の中でわからなそうな単語などについては以下に記しておきました。

---

### Revalidation（キャッシュされたデータの再検証）とは？

簡単に言うと、僕らが実現したかった**キャッシュの更新**です。（下図）

![Revalidation](https://storage.googleapis.com/zenn-user-upload/a2db2205ddbe-20240518.png)

出典：https://zenn.dev/ynot/articles/dc27182e5cc263

### 実は、microCMSの公式記事で紹介されている方法でも、今回の問題は解決できない？？

microCMSの記事でも**時間指定のISR**が紹介されていますが、これだと「microCMSでデータ更新時のみキャッシュを更新する」っていう要件を満たせません。（一時的な解決策にはなります）

https://blog.microcms.io/nextjs13-microcms-rsc/

---

## 教訓

**Next.jsやmicroCMSの公式ドキュメントやGitHubを読むことが解決策を見つける１番の近道！**

初心者だからGoogleなどを使って調べがち（僕もそうです）です。

しかし、公式ドキュメントやGitHubは一見、「**難しそう**」に見えても、わかりやすく書いてあったりするので読んでみることをお勧めします。

これが「急がば回れ」ってやつなのかな？

# 実装方法

**以下の記事をそのまま実装**してください。

※microCMSのダッシュボード側の設定（WebHooksの設定）は割愛されてるので以下に「補足」を掲載しておきました。

https://zenn.dev/ynot/articles/dc27182e5cc263

## ※補足（WebHooksの設定）

### 1. 「カスタム通知」でWebHooksを設定

![WebHooks設定](https://storage.googleapis.com/zenn-user-upload/37e17bb3049b-20240518.png)

### 2. 基本設定

- 「識別名」を設定（適当で大丈夫です）
- 「URL」にmicroCMSのWebhookの受け皿となるRoute Handlerを実装したapiのURLを設定
  **※ちなみにここが間違ってると記事が更新されないので重要！**
  ※上記の記事では`const tag = request.nextUrl.searchParams.get("tag")`で取ってきたtagプロパティを`revalidateTag(tag)`で使用しているので、これも考慮した上でURLを設定してください。

![基本設定](https://storage.googleapis.com/zenn-user-upload/2e27ce9fe0b1-20240518.png)

### 3. カスタムリクエストヘッダーの設定

- 上記の記事だと、Keyとして`X-WEBHOOK-API-KEY`を使用している。
- Valueは、各個人で設定してください。

![カスタムリクエストヘッダー](https://storage.googleapis.com/zenn-user-upload/13af4635c82d-20240518.png)

### 4. WebHooksを発火するタイミングを設定

### これで完了です！

---

### ※実装参考記事の「2. 基本設定」のRoute Handlerのapiを実装するファイルとrequestのURLに関して

以下の公式ドキュメントが参考になるので、読んでみてください。

https://nextjs.org/docs/app/building-your-application/routing/route-handlers

https://nextjs.org/docs/app/api-reference/functions/next-request

---

# これでも解決しない人がいる？？

**Vercel**をホスティングに使ってる人は、これで最初に立てた要件を満たしたサイトが出来上がったと思います。

実は、**Vercel以外でホスティングしている人**は、これだけでは**解決しない**と思います。（ホスティングサービスによると思いますが）

なので、続編として[【Cloudflare編】](https://zenn.dev/rion_freelance/articles/ondemand-isr-cloudflare-nextjs-microcms)（僕がCloudflareでホスティングしているので）も書くので困った人は見てみてください。

# 終わりに（この苦労は報われる）

実は、この問題を解決できるとご褒美があるんですよ。

それは、**データ転送量を節約できる**ってことです。

microCMSには、プランによってですが月に使えるデータ転送量が決まっています。

だから、GET APIの回数を減らせれば、データ転送量を節約できます！

ISRをrevalidateで時間指定にすると、指定時間ごとにビルドされ、その都度APIが呼ばれるので回数が多くなってしまいます。

ただ、On-demand Revalidation（今回だとrevalidateTagを使用）にすると、データ更新の時以外ビルドされず、その時のみAPIが呼ばれるので回数も減らせます。（Pages Router（SSG）のWebHooksを使用した再デプロイ（ビルド）と同じ仕様）

これを「キャッシュ戦略」とも言うそうで、企業が記事にするくらいのことを実現できたってことです。（素晴らしい👏）

https://tech.uzabase.com/entry/app-router-micro-cms-media

# 参考サイト

https://zenn.dev/ynot/articles/dc27182e5cc263

https://qiita.com/shunuke-y/items/5da0201b588771307e60

https://blog.microcms.io/on-demand-isr/

https://nextjs.org/docs/app/api-reference/functions/revalidateTag

https://document.microcms.io/manual/webhook-setting

https://tech.uzabase.com/entry/app-router-micro-cms-media

https://blog.microcms.io/nextjs13-microcms-rsc/