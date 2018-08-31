---
slug: rss-feed-to-google-chat-webhook-using-firebase-functions-and-superfeedr
date: 2018-08-25T16:16:29.749Z
title: 'RSS Feed to Google Chat Webhook using Cloud Functions for Firebase and Superfeedr'
link: https://github.com/PaulKinlan/superfeedr-to-chat
tags: [links,rss,firebase,superfeedr]
---
社内では、Slack のような Google チャットを使用してチーム間のコミュニケーションを取っています。RSSフィードを介してアクセスできる多くのコンテンツを作成し、[誰でも見られるチームフィード](http://devwebfeed.appspot.com)もあります。[WebHooks](https://developers.google.com/hangouts/chat/how-tos/webhooks)を使うと、とても簡単に投稿だけをする単純なボットをつくることができることが最近になってわかりました。そこで思いついたのが、RSS フィードを Webhook に送り、チームチャットに直接投稿できる簡単な仕組みを作成できるということです。

このアイデアは最終的にかなりシンプルなものでした。以下が全コードです。これには、Firebase Functions を使用しました。Superfeedr その他の、Faas（Functions as a service）サイトでも簡単にできるのではないかと思います。[Superfeedr](https://superfeedr.com/) は PubSubHubbub (PuSH)（現在の WebSub）を聞くことができるサービスで、PuSH が設定されていない RSS フィードもポーリングします。フィードが見つかると、新しく見つかったフィードデータを XML または JSON 形式で構成された URL（今回の場合、Firebase の Cloud Functions）に ping を実行します。やらなければいけないのは、データをパースしてやろうと思っていることをするだけです。


```javascript
const functions = require('firebase-functions');
const express = require('express');
const cors = require('cors');
const fetch = require('node-fetch');
const app = express();

// Automatically allow cross-origin requests
app.use(cors({ origin: true }));

app.post('/', (req, res) => {
  const { webhook_url } = req.query;
  const { body } = req;
  if (body.items === undefined || body.items.length === 0) {
    res.send('');
    return;
  }

  const item = body.items[0];
  const actor = (item.actor && item.actor.displayName) ? item.actor.displayName : body.title;

  fetch(webhook_url, {
    method: 'POST',
    headers: {
      "Content-Type": "application/json; charset=utf-8",
    },
    body: JSON.stringify({
      "text": `*${actor}* published <${item.permalinkUrl}|${item.title}>. Please consider <https://twitter.com/intent/tweet?url=${encodeURIComponent(body.items[0].permalinkUrl)}&text=${encodeURIComponent(body.items[0].title)}|Sharing it>.`
    })  
  }).then(() => {
    return res.send('ok');
  }).catch(() => {
    return res.send('error')
  });
})
// Expose Express API as a single Cloud Function:
exports.publish = functions.https.onRequest(app);
```


[完全な記事を読む]（https://github.com/PaulKinlan/superfeedr-to-chat）。

セットアップがあまりにも簡単なことに驚き、そしてうれしく思いました。
