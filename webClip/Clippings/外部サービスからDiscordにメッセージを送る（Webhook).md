---
title: 外部サービスからDiscordにメッセージを送る（Webhook)
source: https://zenn.dev/lambta/articles/5edbda4ccb1ec6
author:
  - "[[Zenn]]"
published: 2023-08-08
created: 2025-05-06
description: 
tags:
  - CI-CD
  - Discord
  - JS
  - 1
  - Tools
  - TS
  - バックエンド
read: false
---
17

1[tech](https://zenn.dev/tech-or-idea)

## はじめに

以前からやろうと思っていたのですが、スラッシュコマンドからDiscord関連の開発を初めた経験から、手順が煩雑なんだろうなと思い込んでいたんですが、簡単でした。。。

ただ、管理画面で紹介されている資料かえって分かりににくいので、記事にするほどのものでもないかと思いつつ、丁寧めに記事にすることにしました。  
[Webhooksへの序章](https://support.discord.com/hc/ja/articles/228383668)  
[Webhook Resource](https://discord.com/developers/docs/resources/webhook)

## Webhook作成

- 「サーバー設定」を選択

![](https://storage.googleapis.com/zenn-user-upload/5d3b3be6c488-20230808.png)

- 「連携サービス」を選択  
	![](https://storage.googleapis.com/zenn-user-upload/9f3cf3a2d605-20230808.png)
- 「ウェブフック」を選択  
	![](https://storage.googleapis.com/zenn-user-upload/f0d2e9feabe2-20230808.png)
- 「新しいウェブフック」を選択  
	![](https://storage.googleapis.com/zenn-user-upload/b58f11ae8809-20230808.png)
- 名前と投稿先のチェンネルを選んだら「ウェブフックURLをコピー」をクリック  
	![](https://storage.googleapis.com/zenn-user-upload/d8fab6186a61-20230808.png)

> [https://discord.com/api/webhooks/0000000000000000000/xxxx-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx](https://discord.com/api/webhooks/0000000000000000000/xxxx-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx)

これはもちろん実在しないURLですが、こんな形式で発行されています。

## まずはシェルからcurlで試す

curlなんて返って難しいわという人は読み飛ばしてくださ。

```
#!/bin/bash

# Webhook URL
WEBHOOK_URL="YOUR_WEBHOOK_URL"

# メッセージコンテンツ
MESSAGE_CONTENT="Hello, this is a test message sent via webhook!"

# リクエストボディ
REQUEST_BODY="{\"content\":\"$MESSAGE_CONTENT\"}"

# POSTリクエストを送信
curl -X POST -H "Content-Type: application/json" -d "$REQUEST_BODY" "$WEBHOOK_URL"
```

実行するとこのように届きます。

![Execute Webhook](https://storage.googleapis.com/zenn-user-upload/9761b1e54fd0-20230808.png)

## パラメーター

メッセージを送る以外にできることはたくさんありますが、  
node-fetchのサンプルコードを兼ねて２パターン、簡単なものを掲載します。

詳しくは以下を読んでください。  
私も正直一部しか試したことがないですが。。。

[Execute Webhook](https://discord.com/developers/docs/resources/webhook#execute-webhook)

### 名前とアバター画像の指定

### embed

requestBodyのみ書き換えます。

```
const requestBody = {
  content: "Hello!!",
  embeds: [
    {
      type: "link",
      title:
        "LangChain JSを参考にしつつ、Supabaseで作成したベクトルデータベースの検索を実装してみる",
      description: "zennに書いた記事です。",
      url: "https://zenn.dev/lambta/articles/c55941310e6687",
    },
  ],
};
```

![](https://storage.googleapis.com/zenn-user-upload/ad3896e11a1c-20230808.png)

## まとめ

何か自分用に便利なツールを思いついた時、スラッシュコマンドを追加しようと思うと億劫になるのですが、この方法だとかなりお手軽にできますね。

これでアイデアを形にする頻度が増えそうです。

17

1

### Discussion

![](https://static.zenn.studio/images/drawing/discussion.png)

ログインするとコメントできます