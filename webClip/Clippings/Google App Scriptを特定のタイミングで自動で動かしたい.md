---
title: Google App Scriptを特定のタイミングで自動で動かしたい
source: https://ex-ture.com/blog/2022/12/23/google-app-script%E3%82%92%E7%89%B9%E5%AE%9A%E3%81%AE%E3%82%BF%E3%82%A4%E3%83%9F%E3%83%B3%E3%82%B0%E3%81%A7%E8%87%AA%E5%8B%95%E3%81%A7%E5%8B%95%E3%81%8B%E3%81%97%E3%81%9F%E3%81%84/
author:
  - "[[エクスチュア株式会社ブログ]]"
published: 2022-12-23
created: 2025-05-06
description: こんにちは。エクスチュアの岩川です。さて、前回に引き続きGASです。段々とGASの基本的なことがわかってきたでしょうか。
tags:
  - GAS
  - Tools
read: false
---
- [ホーム](https://ex-ture.com/blog/)
- Google App Scriptを特定のタイミングで自動で動かしたい

![](https://ex-ture.com/blog/wp-content/uploads/2022/12/nixie_tube_clock.png)

こんにちは。エクスチュアの岩川です。

さて、前回に引き続きGASです。

段々とGASの基本的なことがわかってきたでしょうか。

今回はボタンをわざわざ押すのもなあ…という方向けに、『関数を特定の時間やタイミングで動かす方法』をお伝えしようと思います。

---

## ①トリガーを作成する

GASを開きましょう。今回は [前回で作成した関数](https://ex-ture.com/blog/2022/12/21/google-app-script%e3%81%a8google%e3%82%b9%e3%83%97%e3%83%ac%e3%83%83%e3%83%89%e3%82%b7%e3%83%bc%e3%83%88%e3%82%92%e9%80%a3%e6%90%ba%e3%81%95%e3%81%9b%e3%82%8b/) を引き続き使用していこうと思います。

時計アイコンをクリックします。

![](https://ex-ture.com/blog/wp-content/uploads/2022/12/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88-2022-12-23-19.05.27.png)

右下にある【トリガーを追加】をクリックします。

![](https://ex-ture.com/blog/wp-content/uploads/2022/12/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88-2022-12-23-19.06.14-1-1024x460.png)

## ②トリガーの種類を選択する

すると以下のようなウィンドウが出てきます。ひとつひとつ解説していきます。

![](https://ex-ture.com/blog/wp-content/uploads/2022/12/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88-2022-12-23-19.06.25.png)

## ◆実行する関数を選択

こちらは関数が複数ある場合、どの関数を動かしたいかを選択します。

## ◆実行するデプロイを選択

こちらはバージョンを選択できます。

コードの修正や追加を行った際にバージョン管理を行うことができ、そのバージョンを指定することができます。

すぐに使うことはないと思いますので今はHeadのままで大丈夫です。

## ◆イベントのソースを選択・イベントの種類を選択

ここからが重要です。

イベントのソースを選択したあと種類を選択していくのですが、こちらはソースごとに解説していきます。

・【スプレッドシートから】を選択

①起動時

スプレッドシートを開いた際に動くようになります。

例えばスプレッドシートを起動した際にフォーマットを整えるなどのコードに使えます。

②編集時

セルの値を変更したり、列や行の追加・削除をおこなった際に起動します。

追加や削除などによるフォーマットエラーが起こっていないかなどに使えます。

③変更時

編集時と似ていますが、こちらはセルの値の変更のみが対象です。

セルの値を操作したいだけなどだとこちらのほうが使いやすいです。

④フォーム送信時

こちらはGoogleフォームと連携した際にのみ動作するトリガーです。

Googleフォームで入力された値をスプレッドシートで集計・整理したい場合に便利です。

・【時間主導型】を選択

①特定の日時

YYYY-MM-DD HH:MMの形で日時を指定するとその時間に関数が動きます。

②●●ベースのタイマー

特定の期間おきに動作するようにします。

週・月ベースのタイマーは期間を選択した上で時刻も選択する必要があります。

・【カレンダーから】を選択

こちらは少し特殊で、calendarAppを利用することでカレンダーの編集時に値を取得することができます。

「カレンダーのオーナーのメールアドレス」にカレンダーIDを入力する必要があります。カレンダーと連携させたい場合に使用しましょう。

・エラー通知設定に関して

エラー通知設定はトリガーを動作させた際のエラーメッセージをどのタイミングで受け取るかを表しています。

## ③トリガーの設定

今回はすぐに結果が見れるように

【時間主導型】>【分ベース】>【5分おき】

の指定にします。

![](https://ex-ture.com/blog/wp-content/uploads/2022/12/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88-2022-12-23-19.52.33.png)

このまましばらくまってみましょう。

## ④結果

最初のスプレッドシートはこのようになっていました。

![](https://ex-ture.com/blog/wp-content/uploads/2022/12/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88-2022-12-23-19.50.52.png)

5分後、しっかり関数が動作していることがわかります。

![](https://ex-ture.com/blog/wp-content/uploads/2022/12/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88-2022-12-23-19.54.43.png)

なおこちらが実行しているかどうかは以下の赤枠部分をクリックすることでも一覧として出てきます。

![](https://ex-ture.com/blog/wp-content/uploads/2022/12/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88-2022-12-23-19.56.17-1024x345.png)

今まで関数を動かしてた方法（今回の場合は「時間主導型」と出ていますね）も表示されます。

エラーがある際に通知を見ていなくてもこちらで一覧として確認が可能です。

---

今回は指定した時間で実行してくれる方法についてお伝えしました。

このトリガーを利用することで更に様々な作業を効率よく行うことができます。

エクスチュアでは様々な企業様の基本的な悩みから高度な悩みまで丁寧にお手伝いいたします。

是非お気軽に [お問合せ](https://ex-ture.com/contact/) ください。

- 投稿者: [![](https://ex-ture.com/blog/wp-content/uploads/2022/12/computer_document_spreadsheet-200x200.png) Google App ScriptとGoogleスプレッドシートを連携させる](https://ex-ture.com/blog/2022/12/21/google-app-script%e3%81%a8google%e3%82%b9%e3%83%97%e3%83%ac%e3%83%83%e3%83%89%e3%82%b7%e3%83%bc%e3%83%88%e3%82%92%e9%80%a3%e6%90%ba%e3%81%95%e3%81%9b%e3%82%8b/ "Google App ScriptとGoogleスプレッドシートを連携させる")

 [![](https://ex-ture.com/blog/wp-content/uploads/2023/01/WWW-Cookie-Governance-v3-200x200.png) Cookieを数える -アメリカ、イギリス、オーストラリアの主要ウェブサイト300におけるサードパーティcookieの依存性に関するレポート](https://ex-ture.com/blog/2023/01/10/cookie%e3%82%92%e6%95%b0%e3%81%88%e3%82%8b-%e3%82%a2%e3%83%a1%e3%83%aa%e3%82%ab%e3%80%81%e3%82%a4%e3%82%ae%e3%83%aa%e3%82%b9%e3%80%81%e3%82%aa%e3%83%bc%e3%82%b9%e3%83%88%e3%83%a9%e3%83%aa%e3%82%a2/ "Cookieを数える -アメリカ、イギリス、オーストラリアの主要ウェブサイト300におけるサードパーティcookieの依存性に関するレポート")

1. [最速で理解したい人のためのIT用語集](https://ex-ture.com/blog/2018/08/28/mach-it-words/)

1. [![](https://ex-ture.com/blog/wp-content/uploads/2022/12/job_benriya_nandemoya-200x200.png)](https://ex-ture.com/blog/2022/12/16/google-app-script%e3%81%a7web%e3%82%a2%e3%83%97%e3%83%aa%e3%82%92%e4%bd%9c%e3%82%8b/)
	#### Google App ScriptでWebアプリを作る
	こんにちは。エクスチュアの岩川です。前回Google App…
2. [![](https://ex-ture.com/blog/wp-content/uploads/2022/12/copy_and_paste-200x200.png)](https://ex-ture.com/blog/2022/12/13/%e6%96%87%e5%ad%97%e5%88%97%e7%bd%ae%e6%8f%9b%e3%82%a2%e3%83%97%e3%83%aa%e3%82%92%e4%bd%9c%e6%88%90%e3%81%97%e3%81%be%e3%81%97%e3%81%9f/)
	#### 文字列置換アプリを作成しました
	こんにちは、エクスチュアの岩川です。突然ですがテキストツール…
3. [![](https://ex-ture.com/blog/wp-content/uploads/2022/12/computer_document_spreadsheet-200x200.png)](https://ex-ture.com/blog/2022/12/21/google-app-script%e3%81%a8google%e3%82%b9%e3%83%97%e3%83%ac%e3%83%83%e3%83%89%e3%82%b7%e3%83%bc%e3%83%88%e3%82%92%e9%80%a3%e6%90%ba%e3%81%95%e3%81%9b%e3%82%8b/)
	#### Google App ScriptとGoogleスプレッドシートを連携させる
	こんにちは。エクスチュアの岩川です。前回はGASを使ってWe…
4. [![](https://ex-ture.com/blog/wp-content/uploads/2023/07/ai_computer_sousa_robot-200x200.png)](https://ex-ture.com/blog/2023/07/27/%e3%80%90google-app-script%e3%80%91gas%e3%82%92%e5%88%a9%e7%94%a8%e3%81%97%e3%81%a6slack%e3%81%ab%e6%8a%95%e7%a8%bf%e3%81%99%e3%82%8bbot%e3%82%92%e4%bd%9c%e3%82%8b/)
	#### 【Google App Script】GASを利用してslackに投稿するbotを作る
	こんにちは、エクスチュアの岩川です。業務でSlackを使用さ…
5. [![](https://ex-ture.com/blog/wp-content/uploads/2023/01/kjhou_shifuku-200x200.png)](https://ex-ture.com/blog/2023/01/24/gas%e3%82%92%e5%88%a9%e7%94%a8%e3%81%97%e3%81%a6web%e3%82%b9%e3%82%af%e3%83%ac%e3%82%a4%e3%83%94%e3%83%b3%e3%82%b0%e3%82%92%e3%82%84%e3%81%a3%e3%81%a6%e3%81%bf%e3%82%88%e3%81%86/)
	#### GASを利用してWebスクレイピングをやってみよう
	Webスクレイピングとはスクレイピングとはデータを収集し、使…
6. [![](https://ex-ture.com/blog/wp-content/uploads/2023/02/figure_personal_space_people-200x200.png)](https://ex-ture.com/blog/2023/02/03/google%e3%82%b9%e3%83%97%e3%83%ac%e3%83%83%e3%83%89%e3%82%b7%e3%83%bc%e3%83%88%e3%81%ae%e3%83%87%e3%83%bc%e3%82%bf%e3%82%92gas%e3%81%a7%e6%95%b4%e7%90%86%e3%81%99%e3%82%8b%e3%80%90getrange%e7%b7%a8/)
	#### GoogleスプレッドシートのデータをGASで整理する【getRange編】
	こんにちは。エクスチュアの岩川です。今までGASの基本的な操…

### カテゴリ

- - 7
- - 9

### 最近の記事

1. [TROCCO入門](https://ex-ture.com/blog/2025/05/01/trocco_for_beginner/)
2. [コンポーザブルCDPにおけるSnowflakeのマルチモーダ…](https://ex-ture.com/blog/2025/04/21/composablecdp-snowflakellm/)
3. [boxMCPサーバーを使ってみた](https://ex-ture.com/blog/2025/04/21/boxmcp%e3%82%b5%e3%83%bc%e3%83%90%e3%83%bc%e3%82%92%e4%bd%bf%e3%81%a3%e3%81%a6%e3%81%bf%e3%81%9f/)
4. [#ai-datacloud勉強会でマルチモーダルに触れた日](https://ex-ture.com/blog/2025/04/17/ai-datacloud-multimodal/)
5. [Matillion ETLを安全に使いたい人へ送る、SSL対…](https://ex-ture.com/blog/2025/04/16/matillion-etl-ssl/)

1. [![](https://ex-ture.com/blog/wp-content/uploads/2019/05/%E3%82%AA%E3%83%97%E3%83%86%E3%82%A3%E3%83%9E%E3%82%A4%E3%82%BA%E7%94%BB%E5%83%8F.png)](https://ex-ture.com/blog/2019/05/16/%e3%82%aa%e3%83%97%e3%83%86%e3%82%a3%e3%83%9e%e3%82%a4%e3%82%ba%ef%bc%88optimize%ef%bc%89%e3%81%a3%e3%81%a6%e4%bd%95%ef%bc%9f/) [オプティマイズ（Optimize）って何？](https://ex-ture.com/blog/2019/05/16/%e3%82%aa%e3%83%97%e3%83%86%e3%82%a3%e3%83%9e%e3%82%a4%e3%82%ba%ef%bc%88optimize%ef%bc%89%e3%81%a3%e3%81%a6%e4%bd%95%ef%bc%9f/)
2. [![](https://ex-ture.com/blog/wp-content/uploads/2017/01/nihonchizu_area-500x300.png)](https://ex-ture.com/blog/2017/01/30/adobe-analytics-docodocojp-api/) [Adobe Analyticsと「どこどこJP」のAPIを連携する](https://ex-ture.com/blog/2017/01/30/adobe-analytics-docodocojp-api/)
3. [![](https://ex-ture.com/blog/wp-content/uploads/2019/12/Screen-Shot-2019-12-27-at-16.05.59-500x300.png)](https://ex-ture.com/blog/2019/12/27/tableau%e3%81%ae%e3%80%8cweb%e7%b7%a8%e9%9b%86%e3%80%8d%e6%a9%9f%e8%83%bd%e3%81%ab%e3%81%a4%e3%81%84%e3%81%a6%e7%90%86%e8%a7%a3%e3%81%99%e3%82%8b/) [Tableauの「WEB編集」機能について理解する](https://ex-ture.com/blog/2019/12/27/tableau%e3%81%ae%e3%80%8cweb%e7%b7%a8%e9%9b%86%e3%80%8d%e6%a9%9f%e8%83%bd%e3%81%ab%e3%81%a4%e3%81%84%e3%81%a6%e7%90%86%e8%a7%a3%e3%81%99%e3%82%8b/)
4. [![](https://ex-ture.com/blog/wp-content/uploads/2024/01/poster_ihan_rakugaki_man-400x300.png)](https://ex-ture.com/blog/2024/01/19/observepoint-cookie%e3%82%92%e6%9b%b8%e3%81%84%e3%81%9f%e3%83%a4%e3%83%84%e3%82%92%e8%a6%8b%e3%81%a4%e3%81%91%e3%82%8b/) [ObservePoint: Cookieを書いたヤツを見つける](https://ex-ture.com/blog/2024/01/19/observepoint-cookie%e3%82%92%e6%9b%b8%e3%81%84%e3%81%9f%e3%83%a4%e3%83%84%e3%82%92%e8%a6%8b%e3%81%a4%e3%81%91%e3%82%8b/)
5. [![](https://ex-ture.com/blog/wp-content/themes/mag_tcd036/img/common/no_image2.gif)](https://ex-ture.com/blog/2019/10/30/mouseflow%e3%81%ae%e3%83%97%e3%83%a9%e3%82%a4%e3%83%90%e3%82%b7%e3%83%bc%e8%a8%ad%e5%ae%9a%e3%81%af%e3%82%b7%e3%83%b3%e3%83%97%e3%83%ab%e3%81%a7%e7%9b%b4%e6%84%9f%e7%9a%84%e3%81%aavisual-privacy-tool/) [mouseflowのプライバシー設定はシンプルで直感的なVisual Priva…](https://ex-ture.com/blog/2019/10/30/mouseflow%e3%81%ae%e3%83%97%e3%83%a9%e3%82%a4%e3%83%90%e3%82%b7%e3%83%bc%e8%a8%ad%e5%ae%9a%e3%81%af%e3%82%b7%e3%83%b3%e3%83%97%e3%83%ab%e3%81%a7%e7%9b%b4%e6%84%9f%e7%9a%84%e3%81%aavisual-privacy-tool/)

- - 7
- - 9

[PAGE TOP](https://ex-ture.com/blog/2022/12/23/google-app-script%E3%82%92%E7%89%B9%E5%AE%9A%E3%81%AE%E3%82%BF%E3%82%A4%E3%83%9F%E3%83%B3%E3%82%B0%E3%81%A7%E8%87%AA%E5%8B%95%E3%81%A7%E5%8B%95%E3%81%8B%E3%81%97%E3%81%9F%E3%81%84/#header_top)