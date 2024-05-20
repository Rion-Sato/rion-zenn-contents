---
title: "Next.jsにGoogle Analytics 4を導入する"
emoji: ""
type: "tech"
topics: [Next.js, Google Analytics]
published: true
---
[個人ブログ](https://ribrary.uk)のランキング機能を実装するにあたって、Google Analytics 4（以下：GA4）を導入する必要があり、GA4の設定自体で躓きかけたので、手順をメモしておきます。

GA4設定自体で躓きかけたというのは、Google Analyticsにおけるアカウント設定で躓きかけたということです。

後ほど紹介するが、導入に関しては天下のNext.js様？（Vercel様？）が素晴らしいライブラリとドキュメントを作ってくれているので、クソほど簡単でした。

# GA4の設定手順

Googleの出してる以下の公式ドキュメント通りやれば簡単にできました。

https://support.google.com/analytics/answer/9304153?hl=ja

# GA4の導入

Next.js公式ドキュメントが一番わかりやすいので、参考にしてみてください。

https://nextjs.org/docs/app/building-your-application/optimizing/third-party-libraries

※ちなみに、上記の手法に気づいたのは、以下の記事で教訓を教えてくれる人がいたおかげなので、感謝感謝。

https://zenn.dev/socialplus/articles/922364f3752647

# 絶対注意してほしい点

それは、**@next/third-partiesを「canary」版でインストールする**ってことです！

これが唯一にして最大の注意点です。

Next.js（App Router）にGA4やGoogle Tag Managerを導入するときに、私が詰まって長時間無駄にしたので、ここは絶対に注意してほしいです。