---
title: SlackAPIを使うまでのメモ
source: https://qiita.com/kotaronov27/items/21338b193e6cb3947fe9
author:
  - "[[Qiita]]"
published: 2020-02-20
created: 2025-05-06
description: はじめにSlackAPIを初めて使う方向けの、SlackAPIの権限設定からデバッグまでのメモです。おおまかな流れは、１．Token（特定のSlackチャンネルにアクセスするキー情報）の発行…
tags:
  - 1
  - "#Slack"
  - Tools
  - バックエンド
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から5年以上が経過しています。

[@kotaronov27 (Yagi Kotaro)](https://qiita.com/kotaronov27)

最終更新日 投稿日 2019年04月25日

## はじめに

SlackAPIを初めて使う方向けの、  
SlackAPIの権限設定からデバッグまでのメモです。

おおまかな流れは、  
１．Token（特定のSlackチャンネルにアクセスするキー情報）の発行  
２．APIデバッグ用のページで実際に使用できるかのテスト  
３．実装

以上。

## １．Tokenの発行方法

[https://api.slack.com/apps](https://api.slack.com/apps)  
１）ログイン（ワークスペースの選択）  
２）Tokenの目的設定  
３）ワークスペースのインストール

たったこれだけ。

Slack API 推奨Tokenについて　 [@ykhirao](https://qiita.com/ykhirao "ykhirao") さん  
[https://qiita.com/ykhirao/items/3b19ee6a1458cfb4ba21](https://qiita.com/ykhirao/items/3b19ee6a1458cfb4ba21)  
でとてもわかり易くまとめられています。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/316594/0ac86012-7ecc-e8a9-5272-ed80d1aaba05.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F316594%2F0ac86012-7ecc-e8a9-5272-ed80d1aaba05.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=55e4d747ad0015426fea9713b0c27feb)

## ２．Tokenのスコープ

わからないところはサクッと（使うことがあれば追記していきます。）

### Incoming Webhooks

メッセージ受信

### Interactive Components

メッセージ送信

### Slash Commands

コマンド実行

### OAuth & Permissions

APIの許可権限設定  
迷ったらこれ  
例）Access information about user’s public channels  
　　channels:read  
　　ユーザーの公開チャンネルに関する情報にアクセスする  
　　チャンネル：読み込み権限

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/316594/ddb55ca2-237a-4c6c-e380-bea2cb7e5da6.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F316594%2Fddb55ca2-237a-4c6c-e380-bea2cb7e5da6.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=d4053def5c157d47cb6cdf76debbaaa5)

### Event Subscriptions

通知機能

### Bot Users

BOTユーザ作成

### User ID Translation

グローバルIDをローカルユーザIDに変換する。

## ３．SlackAPIの種類について

## ４．テスト方法

入力例  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/316594/9bad7ab7-debe-cbf8-e26e-f0fd016e0ba0.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F316594%2F9bad7ab7-debe-cbf8-e26e-f0fd016e0ba0.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=e848b63169dce79bad8cf2946c8a4a4f)

### 失敗

{  
"ok":"false",  
"error":"not\_allowed\_token\_type"  
}

```text
Tokenのスコープ設定を見直して、
OAuth & Permissionsに権限を追加。
変更するとトークンを再インストール（関連付け）する必要があるので実行する。

### 成功（Jsonファイルが取得できる）
>\`\`\`
{
    "ok":"true",
    "channels":[
        {
            "id":"XXXXXXXXX"
            "name":"test"
            (省略)
        }
    ]
}
```

あとはAPIをスクリプトで実行すればよいだけ。

## トラブルシューティング

１）missing\_scope（権限が足りないよ）

\[19-04-25 17:41:20:665 JST\] {needed=chat:write:user, provided=identify,channels:read,files:read,files:write:user, ok=false, error=missing\_scope}

```text
>needed:
chat:write:user, 
provided=identify,
channels:read,
files:read,
files:write:user

この権限が必要です
chat:write:user, 
provided=identify,
channels:read,
files:read,
files:write:user

>ok=false

だめでした

>error=missing_scope

権限が足りないよ

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/316594/49375ad5-3092-f1e9-799b-5de7a9e47b79.png)

さくっと追加して。成功。
```

[0](https://qiita.com/kotaronov27/items/#comments)

コメント一覧へ移動

[33](https://qiita.com/kotaronov27/items/21338b193e6cb3947fe9/likers)

いいねしたユーザー一覧へ移動

26