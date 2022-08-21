---
title: Grafana CloudからのアラートをLINEで受けとる
---

Raspberry Piから温湿度計データをGrafana Cloudに送っているのだが、ある一定の室温を超えた段階でアラートを受け取りたかったので、LINEへ通知するように設定してみたメモ。

## LINE Notifyからアクセストークンを発行する

1. LINE Notifyの公式サイト: [https://notify-bot.line.me/my/](https://notify-bot.line.me/my/) へログインする。

2. 「トークンを発行する」ボタンを押下する。

![](https://imgur.com/cVfDzMu.png)

3. 「トークン名」を適当に入力し、通知を送信するルームを選択。「発行する」ボタンを押下する。

![](https://imgur.com/LovP3B8.png)

4. 発行されたトークンをコピーしておく。

![](https://imgur.com/JsHC3cN.png)

## Grafana Cloud側にアラート送信の設定を行う

1. 「Contact points」タブの「New contact point」ボタンを押下する。

![Imgur](https://i.imgur.com/QYYYJCV.png)

2. Contact pointの設定を行い、「Save contact point」ボタンを押下する。

※ 「Test」ボタンで通知送信のテストができるので、Token入力後にテストしておくとよい。

- Name: "LINE"など適当な値を設定
- Contact point type: "LINE"を選択
- Token: LINE Notifyから取得したトークンを設定

![Imgur](https://i.imgur.com/dznRxze.png)

3. Notification policiesタブ等から、通知の送信先を2で設定したContact pointに変更する。