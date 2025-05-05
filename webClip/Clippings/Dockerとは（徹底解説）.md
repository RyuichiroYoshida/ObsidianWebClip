---
title: Dockerとは（徹底解説）
source: https://kinsta.com/jp/knowledgebase/what-is-docker/
author:
  - "[[Kinsta®]]"
published: 
created: 2025-05-06
description: Dockerは、サンドボックスでアプリケーションを開発するためのオープンソースプラットフォームです。この記事では、Dockerの概要と使用方法についてご紹介します。
tags:
  - Docker
  - バックエンド
  - 1
read: false
---
最終更新日

2023年08月31日

![Dockerとは（徹底解説）](https://kinsta.com/jp/wp-content/uploads/sites/6/2022/10/what-is-docker-1024x512.jpeg)

アプリケーションの開発には、 [複雑なデータベース](https://kinsta.com/jp/docs/wordpress-hosting/database-management/wordpress-database-access/) 、プログラミング言語、フレームワーク、依存関係などの管理が必要になることも。しかし、複数のオペレーティングシステム（OS）間での作業では、互換性の問題が発生する可能性があり、また加えた変更がワークフローに悪影響を及ぼすこともあります。

そんな問題を解消するためにお勧めしたいのが、Dockerです。Dockerを使えば、コンテナ化された環境でアプリケーションを構築・管理することができます。多くの複雑な設定作業を簡略化し、簡単かつ効率的に開発が行えるソフトウェアです。

この記事では、Dockerについて、そしてその仕組みについてご説明します。また、主な使用事例と使用方法も見ていきましょう。

## Dockerとは

[Docker](https://www.docker.com/) とは、サンドボックス（完全隔離された空間）でアプリケーションを開発できるオープンソースプラットフォームです。その軽量な仮想環境は、「コンテナ」と呼ばれます。

![Dockerのウェブサイト](https://kinsta.com/wp-content/uploads/2022/10/Docker-Website.png)

Dockerのウェブサイト

コンテナは [1979年に誕生](https://blog.aquasec.com/a-brief-history-of-containers-from-1970s-chroot-to-docker-2016) しましたが、Dockerの登場が、コンテナをより身近なものにしました。Dockerを使えば、開発者はアプリケーションを [ローカル環境または本番サーバーで構築、テスト、デプロイ](https://kinsta.com/jp/ebooks/wordpress/wordpress-local-development/) することができます。

Dockerは、2014年のDocker 1.0リリース以降、本番環境での実行を正式に認めるかたちで、個人・企業向けのサービスを提供してきました。現在では、Netflix、Target、Adobeなどの大手企業を含む1,300万以上の利用者を抱えています。

![Dockerを利用する代表的な企業](https://kinsta.com/wp-content/uploads/2022/10/Docker-Customers.png)

Dockerを利用する代表的な企業

利用者は年々増加しており、 [Datadog](https://www.datadoghq.com/docker-adoption/) によると、全体の約25％もの企業がアプリケーションの監視にDockerを採用し始めたとのこと。2015年以降は、毎年3～5%の勢いで利用者が増えています。

![Dockerを利用する企業は増え続けている（出典: <a href=](https://kinsta.com/wp-content/uploads/2022/10/Docker-Adoption-Behavior.png)

Dockerを利用する企業は年々増え続けている（ 出典: Datadog ）

このように、Dockerはアプリケーションを開発・デプロイするためのプラットフォームとして人気を博しています。次に、Dockerの機能について掘り下げていきましょう。

## Dockerと仮想マシンの比較

Dockerでは、 [ソフトウェア開発](https://kinsta.com/jp/blog/cms-software/) に利用可能なアプリケーションを標準化された単位にパッケージ化することができます。この単位（コンテナ）には、アプリケーションのコードと依存関係が含まれているため、どのようなコンピューティング環境でも簡単に実行できます。

Dockerの登場以前は、アプリケーションの実行に仮想マシン（VM）使用するのが一般的でした。VMは物理的なコンピュータをエミュレートし、1台のサーバーを複数のサーバーとして使用することができます。しかし、この方法にはいくつか欠点があります。

各VMには、OSとアプリケーションの完全なコピーに加え、必要なバイナリやライブラリも格納されるため、コンピュータ上で数十GB近くの容量を占めてしまうことがあります。また、ゲストOS用にハードウェアを仮想化すると、かなりのオーバーヘッドが発生することもあります。

ハードウェアを仮想化する代わりに、OSを仮想化するのがコンテナです。Dockerコンテナは、コードと依存関係の両方を格納することができるアプリレイヤーにある仮想空間で、同じマシン上で、複数のコンテナを個別に実行することができます。

![Dockerと仮想マシンの比較（出典: <a href=](https://kinsta.com/wp-content/uploads/2022/10/Container-VM-Comparison.png)

Dockerと仮想マシンの比較（ 出典: ResearchGate ）

その結果、通常、 [容量を抑える](https://kinsta.com/jp/docs/billing/wordpress-hosting-plans/overages/#disk-space-addon) ことができます。また、VMやOSを増やすことなく、多くのアプリケーションを格納することが可能です。

## Dockerの仕組み

ある場所から別の場所に貨物を輸送することを想像してみてください。コンテナを使えば、隔離された環境で特定の物を一箇所にまとめて保管することができ、船や列車、飛行機での輸送も簡単です。

Dockerは、これとよく似ています。一言で言えば、 [ソフトウェアの開発・デプロイ](https://kinsta.com/jp/wordpress-hosting/staging/) 方法を標準化してくれる技術です。

Dockerはコンテナで動作し、コンテナには、 [Python](https://kinsta.com/jp/blog/python-commands/) 、Node、依存関係などの再利用可能なコンポーネントを格納できます。また、互換性の問題を気にすることなく、コンテナをどこにでもデプロイすることができるようになります。

Dockerの仕組みはやや複雑なため、まずはすべての主要コンポーネントについてご説明していきます。鍵となる機能を押さえておけば、アプリケーション開発を効率化することができるはず。

### Docker Engine

Docker Engineは、Dockerでアプリケーションを構築し、コンテナ化するためのクライアントサーバー型アプリケーション。コンテナベースのアプリケーションの実行に関わるすべての作業をサポートしてくれます。

![Docker Engineの仕組み（出典: <a href=](https://kinsta.com/wp-content/uploads/2022/10/Docker-Diagram.png)

Docker Engineの仕組み（ 出典: Docker ）

Docker Engineの主な構成要素は以下の通りです。

- **Dockerデーモン** ─Docker Image、コンテナ、ネットワーク、ボリュームの管理およびDocker APIリクエストの処理を行う
- **Docker Engine** **REST API** ─デーモンとの対話を行うDockerの独自API
- **Docker CLI** ─デーモンと通信するためのコマンドラインインターフェース

Docker Engineを使用すると、あらゆるインフラストラクチャ上でコンテナ化されたアプリケーションを実行することができます。このセットアップが、同社の業界トップの [コンテナランタイム](https://www.docker.com/products/container-runtime/) の秘密です。

### Dockerイメージ

Dockerイメージは、アプリケーションの実行に必要なソースコード、依存関係、ツールを含んだパッケージで、コンテナ作成時に指示を出す読み取り専用のテンプレートです。

コンテナの実行時に必要なスナップショットが組み込まれた青写真といったところです。

複数のレイヤーが重なった形になっており、例えば、ウェブサーバーのイメージを作る場合、最初に [LinuxのUbuntu](https://kinsta.com/knowledgebase/check-ubuntu-version/) を入れて、次にApacheと [PHPのコード](https://kinsta.com/jp/blog/php-testing-tools/) をその上に重ねることができます。

Dockerイメージを作る際は、最も可変性の高いレイヤーを上にもってくるのが得策です。変更が必要になっても、イメージ全体を再構築する必要がなくなります。

### Dockerコンテナ

先に触れたように、コンテナはDockerの肝となる構成要素。簡単に言うと、Dockerコンテナは、システムの他の部分に影響を与えることなく、アプリケーションを実行できる隔離された領域です。あるアプリケーションから別のアプリケーションに簡単に移行できる様、すべてのコードと依存関係がパッケージ化されます。

Dockerコンテナのメリットは以下の通り。

- **標準化されている** ─コンテナは何十年も前から存在していましたが、業界標準を確立したのはDocker。移動が容易で使いやすいのが特徴。
- **軽量** ─OSのカーネルを共有するため、アプリケーションごとに別のOSを用意する必要がない。アプリを効率的に実行でき、サーバーやライセンスの費用を削減可能。
- **安全性が高い** ─VMとは異なり、アプリケーションがそれぞれ分離されるため、 [安全性が高い](https://kinsta.com/blog/website-security-check/) 。デフォルトで隔離される仕様。

総合的に見て、DockerコンテナはVMよりも優れています。どちらも似た方法でリソースを分離して割り当てますが、一般的にコンテナの方がよりポータブルで効率的、なおかつ安全です。

### Docker Compose

[Docker Compose](https://docs.docker.com/compose/) は、複数のコンテナを1つのサービスとして実行するために設計されたツール。アプリに [Nginx](https://kinsta.com/jp/knowledgebase/what-is-nginx/) と [MySQL](https://kinsta.com/jp/knowledgebase/what-is-mysql/) の両方が必要な場合、Docker Composeを使用して両方のコンテナを起動する1つのファイルを作成すれば、個別に起動する必要がなくなります。

Docker Composeを使用する主なステップは以下の通りです。

1. 再現できるように、アプリケーションの環境を定義（Dockerfileを作成）
2. 分離された環境で実行できるように、アプリケーションの各サービスを **docker-compose.yml** ファイルで定義
3. Docker Composeコマンドを使用して、アプリケーションを起動・実行

基本的には、複数のコンテナを分離して実行するためのものですが、必要に応じて相互作用するように設定することも可能です。

データベースやキャッシュ、 [ウェブAPI](https://kinsta.com/jp/blog/performance-api/) など、アプリケーションのサービス依存関係を作成・設定する際に有用です。

### Dockerfile

Dockerfileは言うなれば、Dockerイメージを構築するための指示が記載されたテキスト文書です。このファイルを読み込むことで、自動的にイメージが構築されます。

`docker build` コマンドを使って、Dockerfileとコンテキスト（指定されたパスまたは [URL](https://kinsta.com/jp/knowledgebase/what-is-a-url/) にあるファイルのまとまり）からイメージを作成します。

まずは、以下のコマンドを実行してください。

```
docker build
```

これで、コンテキスト全体がDockerデーモンに送信されます。ファイルシステム内のDockerfileを指定するには、以下のコマンドを使用します。

```
docker build -f /path/to/a/Dockerfile
```

構築できたら、イメージを保存するリポジトリとタグを指定しましょう。

```
docker build -t shykes/myapp
```

その後、DockerデーモンがDockerfileの検証を行い、構文に問題があればエラーが返されます。

### Docker Desktop

Mac、Linux、Windows環境でDockerを使用するには、 [Docker Desktop](https://www.docker.com/products/docker-desktop/) をインストールしてみてください。使いやすいシンプルなインターフェースで、コンテナ、アプリケーション、イメージを管理することができます。

![Docker Desktop](https://kinsta.com/wp-content/uploads/2022/10/Docker-Desktop.png)

Docker Desktop

Docker Desktopを使えば、必須作業の実行にコマンドラインが不要になるため、 [開発ワークフロー](https://kinsta.com/jp/blog/wordpress-workflow/) を効率化することができます。

![Docker Desktopの管理画面](https://kinsta.com/wp-content/uploads/2022/10/Docker-Desktop-Dashboard.png)

Docker Desktopの管理画面

また、いわゆるアプリストアのような、 **Extensions Marketplace** も組み込まれており、開発者向けのサードパーティツールを簡単に利用できます（ [アプリケーションのデバッグ](https://kinsta.com/jp/blog/application-performance-monitoring/) 、テスト、セキュリティ強化など）。

![Docker Extensions](https://kinsta.com/wp-content/uploads/2022/10/Extensions-Marketplace.png)

Docker Extensions

個人または中小企業の場合、無料で利用できますが、大企業の場合は、 [サブスクリプション](https://www.docker.com/pricing/) （月額5ドル〜）が必要です。

### Docker Hub

[Docker Hub](https://hub.docker.com/) は、コンテナイメージを検索・共有できるプラットフォーム。コミュニティ開発者、オープンソースプロジェクト、独立系ソフトウェアベンダー（ISV）提供のリソースにアクセスできる、世界最大のコンテナイメージのリポジトリです。

![Docker Hub](https://kinsta.com/wp-content/uploads/2022/10/Docker-Hub.png)

Docker Hub

Docker Hubには、以下のような機能があります。

- コンテナイメージのプッシュ/プル用のリポジトリ
- プライベートリポジトリにアクセスできるチームや組織の作成
- Docker公式イメージ
- Docker認証済みイメージ
- [GitHubやBitbucket](https://kinsta.com/blog/bitbucket-vs-github/) からコンテナイメージを作成して、Docker Hubに反映可能
- Webhooksでアクションをトリガー可能

Docker Hubを使うには、リポジトリの作成が必要です。名前を付けて、「Visibility（公開設定）」を選択してください。

![Docker Hubのリポジトリを作成](https://kinsta.com/wp-content/uploads/2022/10/Docker-Hub-Repository.png)

Docker Hubのリポジトリを作成

Docker Desktopをダウンロードすれば、Docker Hubからコンテナイメージのプルとプッシュができるようになります。プッシュした内容は、作成したリポジトリの最新タグの下に表示されます。

## Dockerの使用事例

Dockerは、主に [DevOps](https://kinsta.com/blog/devops-engineer/) と開発者向けで、アプリケーションをポータブルで軽量なコンテナとして作成、編集、デプロイすることができるソフトウェアです。このセットアップは、すべての依存関係を1つの単位にパッケージ化して、事実上すべてのOS上で実行することができます。

一般的な使用事例には、以下のようなものが挙げられます。

1. ローカル環境で記述したコードをDockerコンテナを使ってチームと共有
2. アプリケーションをテスト環境に反映して、自動/手動テストを実行
3. （バグが見つかり開発環境でトラブルシューティングを行う際）修正をテスト環境で確認して再デプロイ
4. バグの修正後、更新されたイメージを本番環境に反映

このワークフローを採用すると、 [自分でインストール](https://kinsta.com/jp/docs/wordpress-hosting/wordpress-getting-started/manually-installing-wordpress/) を行うことなく、新しいソフトウェアをテストすることができます。例えば、 [MySQLサーバーのセットアップ](https://kinsta.com/knowledgebase/mysql-community-server/) は面倒な作業。しかし、Docker CLIを使用すれば、この作業をたった1つのコマンドで実行可能です。

また、Dockerには独自のCLIがあり、初心者はこれを使ってコマンドラインの操作方法を習得できます。Linux環境でDockerをセットアップすれば、 [Linuxのコマンド](https://kinsta.com/blog/linux-commands/) を使うことができるため、システム管理作業をより効率的に行うことが可能です。

ローカル、 [オフラインでWordPressサイト](https://kinsta.com/jp/blog/build-wordpress-site-offline/) を開発する場合は、Docker＋ [DevKinsta](https://kinsta.com/jp/devkinsta/) をぜひ活用してみてください。Kinstaの開発ツールはDockerベースであり、サイトを個々のコンテナで作成・管理することが可能です。

![コンテナ化されたWordPressサイト](https://kinsta.com/wp-content/uploads/2022/10/DevKinsta-Sites.png)

コンテナ化されたWordPressサイト

他のローカル [開発ツール](https://kinsta.com/jp/blog/web-development-tools/) と比較して、DevKinstaは少ないリソースで、本番環境に近いパフォーマンスを維持することができます。Dockerを採用したDevKinstaでは、WordPressサイトを迅速かつ安全に開発できるだけでなく、 [テストメールの送信](https://kinsta.com/jp/knowledgebase/send-test-email/) などのその他の管理作業も簡単に行うことができます。

## Dockerのメリットとデメリット

Dockerの主な構成要素を押さえたところで、次にメリットとデメリットも見ていきましょう。Dockerの使用を検討する際の判断材料になるはずです。

### メリット

開発作業の中には、退屈で反復的な作業も。Dockerコンテナを活用すれば、 [cronジョブ](https://kinsta.com/jp/knowledgebase/wordpress-cron-job/) で、そういった作業の自動化をスケジュールすることができます。作業負荷が大幅に軽減でき、時間の節約につながります。

Dockerは移植性にも優れています。同じプロジェクトに関わるチームメンバー同士が異なるサーバー、マシン、OSを使用していても、開発を行うことができ、プラットフォームの非互換性に起因する問題を避けることができます。

また、仮想マシンと比較すると、似たような役割を果たしながら、容量が小さく軽量です。

さらに、Dockerには大きな利用者コミュニティがあります。コミュニティのイベントに参加することで、他のDockerユーザーと対面したり、オンラインでつながったりすることができるのも魅力です。

![Dockerコミュニティ](https://kinsta.com/wp-content/uploads/2022/10/Docker-Community.png)

Dockerコミュニティ

また、Dockerの広範な [フォーラム](https://forums.docker.com/c/community/59) も忘れてはなりません。特に初心者であれば、役立つアドバイスやヒントを得られるでしょう。

### デメリット

先に述べた通り、DockerはVMよりも効率的ですが、物理サーバー上でアプリケーションを実行する方が、通常はるかに高速です。

また、操作方法の習得の難易度がやや高いことも挙げられます。GUI（グラフィカル・ユーザー・インターフェース）でアプリケーションを実行するようには設計されていないため、コマンドラインを学ばなければならず、初心者にはハードルが上がります。

そして、Dockerコンテナは、サーバー側のOS上で実行されるため、コンテナに悪意のあるソフトウェアが紛れ込むと、サーバーマシンが危険にさらされる恐れがあります。

## Dockerを使い始めるには

[Dockerを使い始める](https://docs.docker.com/get-started/) には、まずメインサイトにアクセスして、使用しているOSに対応したバージョンをダウンロードします。

Macをお使いの場合は、IntelチップまたはAppleチップのどちらかに対応したDockerバージョンを選択してください。また、WindowsとLinuxユーザー向けのバージョンもあります。

ダウンロードしたファイルを開くと、Docker DesktopがPCにインストールされます。次に、 [Docker Hub](https://hub.docker.com/) を開いて、アカウントを作成しましょう。

![Dockerでアカウントを作成](https://kinsta.com/wp-content/uploads/2022/10/Create-Docker-Account.png)

Dockerでアカウントを作成

アカウントの作成後、Docker Desktopを開いてサインインします。コマンドラインまたはbashウィンドウを開いて、以下のコマンドを実行してください。

```
docker run -d -p 80:80 docker/getting-started
```

すると、Docker Desktop管理画面に新規コンテナが表示されます（コンテナ名はアカウント用にランダムに生成されます）。

![Dockerコンテナを作成](https://kinsta.com/wp-content/uploads/2022/10/Docker-Container.png)

Dockerコンテナを作成

また、Dockerイメージも作成されます。「 **Images** 」ページに移動すると、 **docker/getting-started** イメージが使われていることがわかります。

![Dockerイメージ](https://kinsta.com/wp-content/uploads/2022/10/Docker-Image.png)

Dockerイメージ

これで、Dockerを使ってローカル開発を行う準備が完了です。

## まとめ

複数のアプリケーションを管理する場合、Dockerを使用すれば、各アプリを個別に効率よく管理できます。また、コンテナで動作するため、 [プログラミング言語](https://kinsta.com/blog/best-programming-language-to-learn/) やライブラリ、フレームワークが干渉し合うことなく、開発を行えます。

懸念点として、Dockerは仮想マシン（VM）よりも効率的ですが、物理サーバー上で作業する方が高速な場合もあります。また、コマンドラインインターフェースを採用しており、全くの初心者には向いていないかもしれません。

[WordPress開発者](https://kinsta.com/blog/wordpress-developer-salary/) であれば、時にはローカルのテスト環境を即座に作成しなければいけないことも。 [DevKinsta](https://kinsta.com/jp/devkinsta/) を使えば、DockerベースのWordPressサイトを構築することができ、テーマやプラグインの開発がスムーズに行えます。

## 関連記事

## 【3つのシナリオ別】「ERR\_OSSL\_EVP\_UNSUPPORTED」エラーを解決するには

開発プロジェクトのセキュリティ強化にOpenSSLを使用している場合、「ERR\_OSSL\_EVP\_UNSUPPORTED」エラーに遭遇することがあります。この記事で、このエラーの…

読むのにかかる時間

1分で読めます

更新日

2023年09月01日

投稿タイプ

知識ベース

トピック

Node.js

トピック

React

トピック

Vue.js

## 「NET::ERR\_Certificate\_Transparency\_Required」エラーの解決方法

「NET::ERR\_Certificate\_Transparency\_Required」エラーが発生する原因はいくつか考えられます。この記事では、このエラーの原因と解決方法を詳しくご紹介し…

読むのにかかる時間

2分で読めます

更新日

2023年09月05日

投稿タイプ

知識ベース

トピック

Chromeエラー

トピック

ウェブサイトのエラー

## WordPressサイトにCloudflareを導入する方法

今回の記事では、お使いのWordPressサイトにCloudflareを導入（プラグインをインストール）して、最初の設定を行う方法、そして、Cloudflare提供の公式WordPr…

読むのにかかる時間

1分で読めます

更新日

2025年02月19日

投稿タイプ

知識ベース

トピック

Cloudflare