---
title: Terraformのコードを読み取ってクラウド費用を推定してくれる「Infracost」にJetBrains拡張機能版が登場
source: https://gigazine.net/news/20240817-jetbrains-plugin-infracost/
author:
  - "[[GIGAZINE]]"
published: 2024-08-17
created: 2025-05-06
description: コードでインフラストラクチャーを管理する「Terraform」を使用している場合に、クラウドサービスの月間の推定費用を算出してくれるツール「Infracost」がJetBrainsのマーケットプレイスに登場しました。
tags:
  - Tech
  - Terraform
  - Tools
  - IaC
read: false
---
[ソフトウェア](https://gigazine.net/news/C4/)

[![](https://i.gzn.jp/img/2024/08/17/jetbrains-plugin-infracost/00_m.png)](https://i.gzn.jp/img/2024/08/17/jetbrains-plugin-infracost/00.png)

  
コードでインフラストラクチャーを管理する「 **[Terraform](https://ja.wikipedia.org/wiki/Terraform)** 」を使用している場合に、クラウドサービスの月間の推定費用を算出してくれるツール「 **Infracost** 」がJetBrainsのマーケットプレイスに登場しました。  
  
**Infracost Plugin for JetBrains IDEs | JetBrains Marketplace  
[https://plugins.jetbrains.com/plugin/24761-infracost](https://plugins.jetbrains.com/plugin/24761-infracost)**  

[![](https://i.gzn.jp/img/2024/08/17/jetbrains-plugin-infracost/snap6838_m.png)](https://plugins.jetbrains.com/plugin/24761-infracost)

  
どのように動作するのかについては下記のムービーで確認できます。  
  
**[Infracost JetBrains plugin - YouTube](https://www.youtube.com/watch?v=kgfkdmUNzEo)**  

![](https://img.youtube.com/vi/kgfkdmUNzEo/maxresdefault.jpg)

  
まず、こんな感じでTerraformのコードが書かれているとします。  

[![](https://i.gzn.jp/img/2024/08/17/jetbrains-plugin-infracost/snap6839_m.png)](https://i.gzn.jp/img/2024/08/17/jetbrains-plugin-infracost/snap6839.png)

  
設定の「Plugins」を開き、「infra」と検索するとヒットする「Infracost」を選択して「Install」をクリック。これでInfracostをインストールできました。  

[![](https://i.gzn.jp/img/2024/08/17/jetbrains-plugin-infracost/snap6841_m.png)](https://i.gzn.jp/img/2024/08/17/jetbrains-plugin-infracost/snap6841.png)

  
プラグインの再生マークのボタンを押すとTerraformのコードを読み取ってクラウドの月間推定費用を算出してくれます。  

[![](https://i.gzn.jp/img/2024/08/17/jetbrains-plugin-infracost/snap6842_m.png)](https://i.gzn.jp/img/2024/08/17/jetbrains-plugin-infracost/snap6842.png)

  
それぞれどのインスタンスがどれくらいの費用になりそうなのかを詳しく確認することが可能です。  

  
Terraformのコード側でも費用が表示される模様。  

  
コードを変更すると自動で費用が再計算されます。  

  
また、開発環境・本番環境など複数の環境用の設定ファイルを作成することで環境ごとの費用も計算できるとのこと。  

  
そのほか、 **[CI/CDへ統合](https://www.infracost.io/docs/integrations/cicd/)** することでプルリクエストを送るたびに費用を計算するといった使い方も紹介されています。「プラグインをリリースしたばかりなのでフィードバックやバグがあれば **[GitHubのリポジトリ](https://github.com/infracost/infracost)** で報告してほしい」とのことです。

この記事のタイトルとURLをコピーする

**・関連記事**  
**[複数のクラウドサービスに渡ってあらゆる側面からコストを計測可能にするツール「Dashdive」 - GIGAZINE](https://gigazine.net/news/20240213-dashdive)**  
  
**[Amazonのクラウド「AWS」がGoogleなど競合他社に乗り換える際のデータ転送料を無料化 - GIGAZINE](https://gigazine.net/news/20240306-aws-free-data-transfer)**  
  
**[なぜ現代の半導体工場を建設するには何兆円もの費用がかかってしまうのか - GIGAZINE](https://gigazine.net/news/20240507-20-billion-semiconductor-fabrication)**  
  
**[「2025～2026年にはAIモデルの学習費用が1兆円を超えて人類に脅威をもたらすAIが登場する」とAI企業・AnthropicのCEOが予言 - GIGAZINE](https://gigazine.net/news/20240416-anthropic-ceo-ai-training-cost)**  
  
**[AWSを使わないだけでサーバー代を500億円以上節約できた実例 - GIGAZINE](https://gigazine.net/news/20230318-cloud-cost)**

**・関連コンテンツ**

- [![](https://i.gzn.jp/img/2018/08/23/nodejs-web-google-app-engine-gcp/00.png)](https://gigazine.net/news/20180823-nodejs-web-google-app-engine-gcp/)
	[GoogleクラウドのPaaS「App Engine」を使ってオートスケールするWebサーバーをNode.jsで書いてみた](https://gigazine.net/news/20180823-nodejs-web-google-app-engine-gcp/)
- [![](https://i.gzn.jp/img/2023/08/25/meta-code-llama/00_m.png)](https://gigazine.net/news/20230825-meta-code-llama/)
	[Metaが商用利用可能なコーディング支援AI「Code Llama」をリリース、Llama 2と同じライセンスで無料公開へ](https://gigazine.net/news/20230825-meta-code-llama/)
- [![](https://i.gzn.jp/img/2006/05/15/gmap/200605_gmap.jpg)](https://gigazine.net/news/20060515_google_maps_access/)
	[リアルタイムに訪問者をGoogle Mapsにマッピング](https://gigazine.net/news/20060515_google_maps_access/)
- [![](https://i.gzn.jp/img/2008/01/17/sun_buys_mysql/Sun_014_m.jpg)](https://gigazine.net/news/20080117_sun_buys_mysql/)
	[MySQLがSUNに10億ドルで買収されました](https://gigazine.net/news/20080117_sun_buys_mysql/)
- [![](https://i.gzn.jp/img/2008/03/24/vertrigoserv/vs00.png)](https://gigazine.net/news/20080324_vertrigoserv/)
	[Apache/PHP/MySQLなどをWindowsに一発でインストールできる「VertrigoServ」](https://gigazine.net/news/20080324_vertrigoserv/)
- [![](https://i.gzn.jp/img/2019/05/24/github-sponsors/00_m.png)](https://gigazine.net/news/20190524-github-sponsors/)
	[GitHubでクラウドファンディングができる「GitHub Sponsors」がスタート](https://gigazine.net/news/20190524-github-sponsors/)
- [![](https://i.gzn.jp/img/2008/07/25/templateyes/top_m.jpg)](https://gigazine.net/news/20080725_templateyes/)
	[プロっぽくて編集しやすいフリーのウェブサイト用テンプレート配布サイト「TemplateYes」](https://gigazine.net/news/20080725_templateyes/)
- [![](https://i.gzn.jp/img/2021/04/05/github-actions-abused-to-cryptocurrency-mining/00.jpg)](https://gigazine.net/news/20210405-github-actions-abused-to-cryptocurrency-mining/)
	[GitHubのサーバーを仮想通貨マイニングに利用する攻撃の存在が判明、GitHub Actionsを悪用](https://gigazine.net/news/20210405-github-actions-abused-to-cryptocurrency-mining/)

in [ソフトウェア](https://gigazine.net/news/C4/), [動画](https://gigazine.net/news/C9/), Posted by log1d\_ts

You can read the machine translated English article **[JetBrains extension version of Infracost…](https://gigazine.net/gsc_news/en/20240817-jetbrains-plugin-infracost)**.