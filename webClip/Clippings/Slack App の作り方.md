---
title: Slack App の作り方
source: https://zenn.dev/nyancat/articles/20211219-create-slack-app
author:
  - "[[Zenn]]"
published: 2021-12-20
created: 2025-05-06
description: 
tags:
  - 1
  - Slack
  - Tools
  - バックエンド
read: false
---
26[tech](https://zenn.dev/tech-or-idea)

Slack App / Slack Bot を作るのが初めてだったので、備忘録として作り方を詳しめに書きました。同じような方の参考になれば幸いです。

以下は [2021年10月7日に公開された Slack App Configuration 新デザイン](https://api.slack.com/changelog/2021-10-manifest-a-new-reality) の場合です。  
新デザインが気に入らない場合は Distribution > Revert to the old design から旧デザインに戻せます。  
![](https://res.cloudinary.com/zenn/image/fetch/s--fb7C_wdQ--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/50e178ffb88a487f4a2c5851.png%3Fsha%3D46b5a1885d2e78ee81c22481e60cccad34c83df0)

## Slack App 作成

[slack api > Your Apps](https://api.slack.com/apps) から `Create New App` をクリックします。  
特にこだわりがなければ `From scratch` を選択、最初から YAML や JSON で app manifest を入力する場合は `From an app manifest` を選択します。

![](https://res.cloudinary.com/zenn/image/fetch/s--E5rpwYzW--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/3a90ec1da39032793bee1d89.png%3Fsha%3D1415ec7f3f52330babb139f43d68e496a5b45a48)

次の画面で slack app 名と、この slack app を適用する slack workspace を指定します。  
workspace は後から変更できません。

![](https://res.cloudinary.com/zenn/image/fetch/s--Iy0SFlQa--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/7008ff09e345f1646bf6c4e1.png%3Fsha%3D46c12dc77f1780df2b7b94aa459463ecd09b9f63)

## Slack App に必要な権限を付与

Settings > App Manifest で [公式ドキュメント](https://api.slack.com/reference/manifests) を参考に、YAML または JSON で App に必要な権限を定義します。

![](https://res.cloudinary.com/zenn/image/fetch/s--zLNLXfFK--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/442b69a5b887db10462da7fd.png%3Fsha%3Dfa765effcef887cc90e37d670e1a76845d0d90b5)

下は user scopes に `channels:history`, bot scopes に `channels:history` と `chat:write` を設定する例です。  
bot scope を設定する場合は `bot_user` の記述も必要となります。

```yaml
features:
  bot_user:
    display_name: Test App
    always_online: false
oauth_config:
  scopes:
    user:
      - channels:history
    bot:
      - channels:history
      - chat:write
```

user scope と bot scope の違いですが

- user scope はユーザーに成り代わって処理をする権限
- bot scope はあくまでボットユーザーとして処理をする権限

となっているようです。

どの scope が必要かは用途に応じて選択します。どの [Web API methods](https://api.slack.com/methods) が用途に合うかを公式ドキュメントで調べて、その API method の `Required scopes` に書いてある [scope](https://api.slack.com/scopes) を逆引きして設定する、とすると上手く権限設定ができそうです。

例えばチャンネルにメッセージを投稿する `chat.postMessage` という method を実装で使うのであれば、Bot なら `chat:write`, User なら `chat:write` / `chat:write:user` / `chat:write:bot` を scope として追加すれば良いことが分かります。

![](https://res.cloudinary.com/zenn/image/fetch/s--VI1UjgZf--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/9d6cfc4468114f585b09e750.png%3Fsha%3De55663f9889b2c263a2178bf8401b68266b4e69c)

## Slack App を Workspace にインストール

Distribution > Install App から対象 Workspace に Slack App をインストールします。

![](https://res.cloudinary.com/zenn/image/fetch/s--8EsqDMFJ--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/01e5ef6a9fb489d3b65b1a65.png%3Fsha%3D236bb53277abfdbb3b57bbcbfafa974a5cf76ea1)

前段階で scopes として設定した権限に対するリクエストが表示されるので、許可するをクリックします。

![](https://res.cloudinary.com/zenn/image/fetch/s--jLyUut8v--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/baf1c02b4053333e9d9f524b.png%3Fsha%3D8735301656e98d54afa656715e4f6c02e8517cf8)

User OAuth Token (xoxp-) / Bot User OAuth Token (xoxb-) が表示されます。  
後で Distribution > Install App からも Token は確認可能です。

## Slack App を Slack Channel に追加

主に bot scope として権限を追加している場合に必要な手順です。

Slack を開き、Slack App の処理対象としたい Slack Channel の チャンネル詳細 > インテグレーション > app から作成した Slack App を追加します。  
この追加を忘れると bot user として Slack API リクエスト時には `not_in_channel` というエラーが返ってきます。

![](https://res.cloudinary.com/zenn/image/fetch/s--us_hKCPI--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/d90b8193a10a19fd6f43c134.png%3Fsha%3Dd2a6107c33479e58ae17c6e6062bb366747096f2)

## 実装

あとは発行した Token を用いて Slack API をリクエストするなどの実装を進めていきます。  
Slack 公式の各言語 SDK はこちらです。

## Slack API Tester

[Web API methods](https://api.slack.com/methods) ページには GUI から簡単に API をリクエストする Tester があり、実装前にエラー無く API をリクエストできるかテストすることが可能です。

![](https://res.cloudinary.com/zenn/image/fetch/s--f_omvA4B--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/44090f4d0ac443c8c62690ea.png%3Fsha%3D69c16c5a8642ed403d38014f645e9d482127f7d2)

## 参考

26