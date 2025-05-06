---
title: Slack Appの作り方を丁寧に残す【BotとEvent APIの設定編】
source: https://zenn.dev/mokomoka/articles/6d281d27aa344e
author:
  - "[[Zenn]]"
published: 2021-02-03
created: 2025-05-06
description: 
tags:
  - Slack
  - Tools
  - バックエンド
read: false
---
128

2[tech](https://zenn.dev/tech-or-idea)

とある事情でSlack Appを久しぶりに作ろうとしたら、APIやトークンの仕様が変わっていて混乱したため、自分のための備忘録を兼ねて工程を残すことにしました。  
最終的に作るAppの実装は別記事の内容になりますが、今回はその前段階として、主に **Slack Appに関する設定をSlackのサイト上で行う流れ** について残しておきます。この部分はAppの内容にあまり依存しないため、少しでも多くの方に役立てば幸いです。  
※スクリーンショット多いので重かったらごめんなさい

Botの作成（主にメッセージ送信などを行う）と、Event APIの設定（トリガーとなるメッセージ受信などを行う）の2つに分けて説明します。

## 1\. Botの作成

## 1.1 Appの作成

ブラウザでSlackワークスペースにサインインした状態で、自分が作成したSlack Appを見ることができるページ（ [https://api.slack.com/apps](https://api.slack.com/apps) ）にアクセスします。  
右上の「Create New App」をクリックします。  
![Your Apps](https://storage.googleapis.com/zenn-user-upload/mco7g158chn2w4r7uxfsys78uraf)

すると、「App Name」（Appの管理画面や、ワークスペース内でAppを選択したときに表示される方の名前）と「Development Slack Workspace」（どのワークスペースでAppを使用するか）を入力/選択するウィンドウが表示されるので、適当に入力します。  
![Create a Slack App](https://storage.googleapis.com/zenn-user-upload/kp2toun5nu1bwwbfw7lbctfzx9gi)  
![Create a Slack App（入力後）](https://storage.googleapis.com/zenn-user-upload/x0ttot0coh92s5ztfrwm1r1r30xt)  
プルダウンの中に選びたいワークスペースが表示されない場合は、おそらくサインインできていない状態です。  
Appのリストの下にある「Sign in to another workspace.」リンクをクリックし、サインインしましょう。

## 1.2 Botに必要な権限設定

1.1でAppを作成した直後は、「Basic Information」というページが表示されます。  
![Createした直後のBasic Information](https://storage.googleapis.com/zenn-user-upload/3a6cctnrqplkyuphv47221zx31fb)

少しスクロールすると、「Permissions」というリンクがあるので、それをクリックします。  
![Permissionsへのリンク](https://storage.googleapis.com/zenn-user-upload/49b0dm3dk6orh0kaxvhji9o2hfhz)

「OAuth & Permissions」というページが表示されるので、中程までスクロールして、「Scopes」を見つけます。  
「Add an OAuth Scope」をクリックし、必要な権限のScopeを追加します。  
![Scopes](https://storage.googleapis.com/zenn-user-upload/5ce62ayb8g59rmbrwbamtku1y30w)

今回の例では、

1. チャンネルのメッセージを読む → channels:history
2. メッセージを投稿する → chat:write
3. メッセージにリアクションする → reactions:write
4. チャンネルに参加する → channnels:join

といった4つのScopeを追加しました。  
![Scopes（選択後）](https://storage.googleapis.com/zenn-user-upload/fnnkgkjbsbtfi52c4t8ho5c739q0)

## 1.3 Botの設定など

1.2でクリックしたPermissionsリンクの隣に、「Bots」というリンクがあるので、それをクリックします。  
![Botsへのリンク](https://storage.googleapis.com/zenn-user-upload/vy4tw9zqg59x47471i770oqbjxby)

「App Home」というページが表示されるので、「Your App's Presence in Slack」内の「Edit」をクリックします。  
![App Home](https://storage.googleapis.com/zenn-user-upload/67h9fhpy19v1r9uw1ndvlb43xzks)

すると、「Display Name (Bot Name)」（Slack上で表示される名前、メッセージ投稿などで一般的に見かける名前）と「Default username」（ID的なもの）を入力するウィンドウが表示されるので、適当に入力します。  
ちなみに私は、Default usernameを使う場面に遭遇したことがないので、正直なにに使うのかよく分かっていません……（一般ユーザならカスタムレスポンスでメンション送るときに使いますが）  
![Add App Display Name](https://storage.googleapis.com/zenn-user-upload/ueroip1kkuttjn1smkz50nzwocl0)

その後、同じページの上部にスクロールを戻し、「Install to Workspace」をクリックしてBotをワークスペースに導入します。  
![Install to Workspace](https://storage.googleapis.com/zenn-user-upload/ltuyabjv4zddvvw2mqf1e9nujxiz)

次のページで、Botがアクセス可能な情報や実行できる内容が、先程設定したScope通りかどうか確認してから、「許可する」をクリックしましょう。  
![Botがワークスペースにアクセスする権限をリクエストしている画面](https://storage.googleapis.com/zenn-user-upload/319oodylkankoozi9wst40sgtkl7)

その後OAuth & Permissionsページに表示される「Bot User OAuth Access Token」は、Bot開発時に使いますので、必要になったらコピペしましょう。  
![Bot Token](https://storage.googleapis.com/zenn-user-upload/qk3fa2f0o9llmn1rff3b1or6xw4h)

## 1.4 Botの開発

1.3までで、Slack側で準備する部分は終了ですので、あとはWeb APIのリファレンス（ [https://api.slack.com/web](https://api.slack.com/web) ）を参照して開発していきます。

例えば、メッセージを投稿したい場合（1.2のScopeでいう `chat:write` ）は、上記リンク先のリストの中から、 `chat.postMessage` メソッドのリンク先を確認すれば、どのエンドポイントを使えばいいのか、どの情報を必須で送らないといけないのかなどを知ることができます。  
botにどのScopeが必要なのかも記載してあるため、逆にこちらのリファレンスで使用するメソッドを決めてから、1.2の権限設定に戻っても良いと思います。

（2023/5/19追記：開発の詳細は [別の記事](https://zenn.dev/mokomoka/articles/924db0e5e356a7) に記載しています。）

## 2\. Event APIの設定

## 2.1 Appの設定

（Event API以外のAPI使用でも、Appの設定は必要だと思いますが……）  
Basic Informationページをスクロールすると、Slack上で確認できるAppの情報を編集できる箇所があります。  
「App name」（Appの管理画面や、ワークスペース内でAppを選択したときに表示される方の名前）、「Short description」（Appの説明文）、「App icon」（Appのアイコン画像）、「Background color」（App nameとShort descriptionの背景色）を適当に設定しましょう。  
設定後は「Save Changes」で保存するのを忘れないようにしてください。  
![Display Information](https://storage.googleapis.com/zenn-user-upload/iwpc6ejp4c9p8pkvz61cuzj494k0)

Basic Informationページの「App Credentials」はApp開発時に使いますので、必要になったらコピペしましょう。  
![App Credentials](https://storage.googleapis.com/zenn-user-upload/i02ucf462g40qr8hcyg2w4y6r76f)

## 2.2 Event API使用のための準備

AppはEvent APIのドキュメント（ [https://api.slack.com/apis/connections/events-api](https://api.slack.com/apis/connections/events-api) ）を参照して開発していきますが、先に準備すべきことがあります。

上記ドキュメントページの [Request URL Configuration & Verification](https://api.slack.com/apis/connections/events-api#the-events-api__subscribing-to-event-types__events-api-request-urls__request-url-configuration--verification) に記載されている通り、初回はURL Verification（サーバの存在確認用）のリクエストがきますので、それに対応するコードを書く必要があります。  
ドキュメントによると、以下のようなPOSTリクエストが届きます。（ドキュメントから引用）

```json
{
    "token": "Jhj5dZrVaK7ZwHHjRyZWjbDl",
    "challenge": "3eZbrw1aBm2rZgRNFdxV2595E9CY3gmdALWMmHkvFXO7tYXAYM8P",
    "type": "url_verification"
}
```

つまり、

1. `token` が2.1で述べたApp Credentials内のVerification Tokenに書かれているものと一致しているか確認する
2. `type` が `url_verification` と一致しているか確認する
3. `challenge` のテキストをそのまま返す

の3段階が必要になります。

例えば、GASではこのように書けると思います。

```javascript
function doPost(e) {
  // プロジェクトのプロパティ>スクリプトのプロパティから情報取得
  // AppのVerification Tokenが入っている前提
  const prop = PropertiesService.getScriptProperties();
  
  // Events APIからのPOSTを取得
  // 参考→https://api.slack.com/events-api
  const json = JSON.parse(e.postData.getDataAsString());
  
  // Events APIからのPOSTであることを確認
  if (prop.getProperty("verification_token") != json.token) {
    throw new Error("invalid token.");
  }
  
  // Events APIを使用する初回、URL Verificationのための記述
  if (json.type == "url_verification") {
    return ContentService.createTextOutput(json.challenge);
  }
}
```

リクエストに対処できるようにしたら、そのリクエストを受け取るURLをSlack App側に登録します。  
Basic Informationページに「Event Subscriptions」というリンクがあるので、それをクリックします。  
![Basic InformationのEvent Subscriptions](https://storage.googleapis.com/zenn-user-upload/q463foejqr51uqcv9r60wftp0o65)

Event Subscriptionsページが表示されるので、「Enable Events」のトグルを「On」にします。  
![Event Subscriptions](https://storage.googleapis.com/zenn-user-upload/p7mr4j2uigpd6idqp97mv6iqf1ml)

そして、Request URLにリクエストを受け取るURLを入力します。  
例えば、GASの場合は「ウェブアプリケーションとしてデプロイ（導入）」した際のURLを入力します。  
![Request URL入力時のEvent Subscriptions](https://storage.googleapis.com/zenn-user-upload/1xx8fsllmrnevbw0swnat0h93y0s)  
入力後はすぐにリクエストが送られるためそのまま待ちます。  
「Verified」が表示されれば認証が通ったことになります。

## 2.3 受け取るイベントを設定

2.2のEvent Subscriptionsの下に、「Subscribe to bot events」というセクションがあり、そこで受け取るイベントを設定できます。  
「Add Bot User Event」をクリックすることでイベントを追加します。（追加前にスクリーンショットを撮り忘れてしまったため、画像は追加後の状態です）

![Subscribe to bot events](https://storage.googleapis.com/zenn-user-upload/q4vnomhb10mq7g2auvijimugc46i)

今回の例では、チャンネルに投稿されたメッセージによって、1.3で述べたようなリアクションや投稿を行いたいため、 `message.channels` を指定しました。  
この際、「Required Scope」列に書かれているScopeが、1.3で設定したScopeに含まれているか確認してください。含まれていないとおそらく正常にイベントを受け取れません。  
設定後は「Save Changes」で保存するのを忘れないようにしてください。

## 2.4 Appの開発

事前準備/設定は以上ですので、あとはリファレンスを基に開発していくのみです。

[先程も紹介した、Event APIのドキュメント](https://api.slack.com/apis/connections/events-api) からReference -> Events API -> Subscription typesとたどると、イベントのリストが表示されます。（ [https://api.slack.com/events](https://api.slack.com/events) ）  
例えば2.3と同様の例で言うと、 `message.channels` のところをクリックして見れば、どのような情報が取得できるかがわかります。

（2023/5/19追記：こちらも開発の詳細は [別記事](https://zenn.dev/mokomoka/articles/924db0e5e356a7) に記載しています。）

## 使い方

上記手順で作成したAppは、「アプリを追加する」ことで使用できます。  
「アプリを追加する」は、作ったばかりのチャンネルでは分かりやすい位置に表示されています。  
![アプリを追加する1](https://storage.googleapis.com/zenn-user-upload/9xhowysr4tl9ih4t0lvh84o885th)

また、チャンネルの「詳細」からたどる事もできます。  
![アプリを追加する2](https://storage.googleapis.com/zenn-user-upload/wrq0jclwlkv8e8bpof67nf8t5l54)

どちらにせよアプリ一覧にたどり着くことができますので、こちらで「追加」をクリックしましょう。  
![アプリを追加する3](https://storage.googleapis.com/zenn-user-upload/xpcjwi1upozwb4de6oleagfr0yrn)

## 参考

[SlackのtokenとAPI、botの種類をまとめた](https://qiita.com/tkit/items/2536ea6971754f9a75d1)  
[\[GAS\]Slackのbotを作る方法を、お節介なほど丁寧に説明](https://qiita.com/sew_sou19/items/723feb39ea4aa7becfab)

以上です。  
分かりづらいと思ったところは適宜加筆修正していきます。  
私も特別詳しいわけではありませんが、質問や困ったことがあればお気軽にどうぞ！

（2023/5/19追記：開発の詳細＆続編はこちらです。GASで書いたコードのリポジトリへのリンクと、内容の解説があります。）

128

2