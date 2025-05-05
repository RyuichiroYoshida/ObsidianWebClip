---
title: 5分で分かるTerraform（Infrastructure as Code） | LAC WATCH
source: https://www.lac.co.jp/lacwatch/service/20200903_002270.html
author:
  - "[[株式会社ラック]]"
published: 2023-05-26
created: 2025-05-06
description: HashiCorp社が提供するTerraformは、マルチクラウド上のコンピュータやネットワークの構築を自動化する、エンジニアにとても人気のあるツールです。その「成り立ち」と「何を目指しているのか」についてご紹介します。
tags:
  - Tech
  - Terraform
  - クラウド
  - バックエンド
read: false
---
さらに詳しく知るにはこちら

[Terraform（テラフォーム）](https://www.lac.co.jp/solution_product/terraform.html)

HashiCorp社が提供するTerraformは、マルチクラウド上のコンピュータやネットワークの構築を自動化する、エンジニアにとても人気のあるツールです。

Terraformをご存じなかった方にも、その「成り立ち」と「何を目指しているのか」についてご理解いただけるよう、今回は「5分で分かるTerraform（Infrastructure as Code <sup>※1</sup> ）」と題した記事を日本語訳してお届けします。著者のMehdi ZedはモントリオールでDevOpsバックエンド開発に携わっており、DevOpsについてSNSで積極的に発信しています。その発信内容は非常に分かりやすく、今回の記事についても本人の了解を得て日本語化しました。

さらに詳しく知るにはこちら

[Terraform（テラフォーム）](https://www.lac.co.jp/solution_product/terraform.html)

※1 インフラの構築・運用に関わる作業をコード化、自動化するアプローチ。設定変更作業をコードで管理することで手作業による間違いを防ぎ、設定変更を履歴として管理できる。また、設定変更作業を自動化することで、大量のサーバーに迅速に設定変更を反映できる。

元記事

- [Understand Terraform (infra-as-code) in 5 minutes - Je suis un dev](https://www.jesuisundev.com/en/understand-terraform-infra-as-code-in-5-minutes/)

---

Terraformは、今の私にとって「ささやかな、ごちそう」です。今回はこのTerraformについてご紹介したいと思います。ご存じのとおり、ここ数年のDevOpsムーブメントの広がりに伴い、Infrastructure as Codeは今や誰もが取り組むべきものになってきました。そんな中でもTerraformはもっとも「クール」なソリューションです。

## かつては

2006年の初めに、Amazonは少し知られたサービススイートである [Amazon Web Service（AWS）](https://aws.amazon.com/jp/) を発表しました。AWSについては皆さんご存じかと思います。

同じ年の8月25日、AWSは [EC2](https://aws.amazon.com/jp/ec2/) のベータ版を米国でリリースしました。これにより開発者は初めて、クラウド上にサーバーを数分で起動できるようになりました。1年後、このサービスは世界中で一般公開されます。EC2のリリースはその後の全てを変えてしまうような出来事でした。

この革命以前は、アプリケーション用のサーバーを用意するのはかなり大変だったということを理解しておく必要があります。クラウドが登場する前は、社内でインフラサービスを用意するよう誰かに依頼しなくてはいけませんでした。オンプレミスのサーバーを手動で構築する必要があり、そのサーバーを構築している間はずっと待たなくてはなりません。

ところが現在では、数回のクリックでほとんど瞬時にサーバーを構築、変更、破棄することができます。これはこれで非常に良いことですが、これだけではサーバーを完全にコントロールするには不十分です。

それではコードを使って、クラウド上のインフラ環境を動的に構築し、管理できるとすればどうでしょうか。Amazonはこのアイデアを非常に気に入って、2010年5月15日にCloudFormationを発表します。

サービスとしてのインフラ（IaaS：Infrastructure as a Service）が生まれ、多くの人々に歓迎されました。そして現在最も話題になっているIaC（Infrastructure as Code）ツールTerraformがこの記事のテーマです。

## Terraformとは

Terraformは、IaCを実現するツールです。Terraformはオープンソースであり、HashiCorpによってGo言語で開発されました。具体的にはTerraformではインフラの構成をコードで宣言します。構造化された構成ファイルでは、手動で操作することなくインフラ構成を自動で管理できます。インフラの初期プロビジョニング、更新、破棄、いずれもTerraformではコードにより宣言し、実行します。

IaCの世界では、インフラ構成を実現する方法がいくつかあります。Terraformは、いわゆる宣言型の方法を採用しており、インフラの望ましい状態を構成ファイルで宣言します。Terraformは、インフラの望ましい状態を達成するために最小限のことだけを行います。

例：あなたの管理する稼働中のインフラでは、すでにWebサーバーとデータベースが実行されています。コード内の構成はこれに完全に対応しています。次に2つのデータベースが必要になる構成を追加します。コミットするとTerraformは追加のデータベースを作成します。それ以外には何もしません。これによりコードで宣言された望ましい状態に達しました。あなたはこれを一見して「美しい」と感じるでしょう。

コードを見るだけでインフラの構成内容を理解できるということは素晴らしいことです。コードはGitでバージョン管理されます！つまり、あなたのインフラ環境もバージョン管理されるということです。

コードでインフラを構築する場合、本番環境を作る前にいくつかの環境でインフラ構築をテストできます。開発環境で同じコードをプッシュすると、インフラの変更を安全にテストできます。 同僚とインフラの状態をコードレビューすることも可能です。コードを数行変更するだけで、すばやく繰り返し重要な変更をすることができます。つまり、DevOpsで開発を進めるにあたり、最適な手段といえます。

しかし実際にTerraformはどのように機能しているのでしょうか？Terraformはインフラ構成の一部を、どのように構築、変更、破棄を管理しているのでしょうか？

## どのように機能しているのか？

Terraformは構成ファイルで宣言した状態を取得し、変更内容を宛先のプロバイダーにプッシュします。AWS上にサーバーを構築する場面をイメージしてください。サーバーの構成を宣言し、それをプッシュすることでAWSアカウント上にサーバーが現れます。

Terraformは [Go言語で書かれたAWS SDK](https://docs.aws.amazon.com/sdk-for-go/api/) を使用しています。このSDKを介して、サーバーをクラウド上に直接構築しました。ですが、SDKコードに煩わされたり、背後で何が起こっているのかを本当に理解したりする必要はありません。Terraformがそのような面倒なことを吸収してくれますので、コードだけチェックすればOKです。これにより、インフラの更新フローが非常にシンプルかつ高速になります。

![Terraform使用してクラウド上にサーバーを構築する構成図](https://i.imgur.com/6zpK45F.jpg)

Terraform使用してクラウド上にサーバーを構築する構成図

そして重要なのは、Terraformが1つのプロバイダー（AWS）やクラウド一般に限定されないことです。ほとんどすべてのタイプのインフラをTerraformのリソースとして表すことができます。そう考えるとTerraformの素晴らしさが伝わると思います。

ローカルマシンの [Dockerコンテナ](https://registry.terraform.io/providers/kreuzwerker/docker/latest/docs) から [DigitalOcean](https://registry.terraform.io/providers/digitalocean/digitalocean/latest/docs) のクラウドアカウントに至るまで、Terraformによってインフラをどこにでも再現できます。この記事では、利用可能プロバイダーの全リストを提供するつもりはありませんが、たとえば [こちら](https://www.terraform.io/docs/providers/index.html) を参照ください。そしてこの利用可能プロバイダーが多数あるという事実はTerraformの有用性を高めます。

さて、理論はこれで十分ですので、次は実際に手を動かしてやってみましょう。

## コードを見る

それでは、設定から始めてみましょう。ここでは、Linuxディストリビューションを使用しているものと想定して話を進めます。そうでない場合は [こちら](https://learn.hashicorp.com/tutorials/terraform/install-cli) をご覧ください。

\# dependencies  
sudo apt-get install wget unzip  
  
\# download last version  
wget  
https://releases.hashicorp.com/terraform/0.12.20/terraform\_0.12.20\_linux\_amd64.zip  
  
\# unzip in bin folder  
sudo unzip./terraform\_0.12.20\_linux\_amd64.zip -d /usr/local/bin/  
  
\# check its working  
terraform -v

次に最初の構成ファイルを作成します。まず簡単なところから始めましょう。AWS上にEC2サーバーとPostgresデータベースを作成します。管理者のクレデンシャルを発行できるAWSアカウントも持っているものと想定しています。最初のステップでは、プロバイダーを定義するmain.tfファイルを作成します。

### main.tf

\# definition provider  
provider "aws" {  
  version = "~> 2.0"  
  region = "${var.provider\_region}"  
  access\_key = "${var.secret\_access\_key}"  
  secret\_key = "${var.secret\_key}"  
}

ここでは、AWSを利用すること、及びバージョンとリージョンを指定します。AWSにアクセスするにはクレデンシャル情報を指定します。指定する値は変数を介して参照されます。これらの変数はすべて、この後に見るファイルに格納されます。

これでAWSの利用するバージョン、リージョンを宣言したので、次は構築するものを宣言します。いつものように、どこから始めればよいかわからないときは、ドキュメントを調べましょう。UbuntuとPostgresデータベースを宣言するresource.tfファイルを作成してみましょう。

### resource.tf

resource "aws\_instance" "web" {  
  ami = "${data.aws\_ami.ubuntu.id}"  
  instance\_type = "${var.server\_instance\_type}"  
  
  tags = {  
    Name = "${var.server\_tag\_name}"  
  }  
}  
  
data "aws\_ami" "ubuntu" {  
  most\_recent = true  
  
  filter {  
    name = "name"  
    values = \["ubuntu/images/hvm-ssd/ubuntu-trusty-14.04-amd64-server-\*"\]  
  }  
  
  owners = \["099720109477"\]  
}  
  
resource "aws\_db\_instance" "default" {  
  allocated\_storage = "${var.db\_allocated\_storage}"  
  storage\_type = "${var.db\_storage\_type}"  
  engine = "${var.db\_engine}"  
  engine\_version = "${var.db\_engine\_version}"  
  instance\_class = "${var.db\_instance\_class}"  
  name = "${var.db\_database\_name}"  
  username = "${var.db\_database\_username}"  
  password = "${var.secret\_db\_database\_password}"  
  final\_snapshot\_identifier = "${var.db\_snapshot\_identifier}"  
}

必要なすべてを定義したので、これに対応する変数を入力します。専用のvariables.tfファイルを作成します。シークレットは本来ファイル内で平文であることは望ましくありませんが <sup>※2</sup> 、今回はデモを簡単に見せるために、このようにします。

※2 訳注：Terraformで使うシークレットを安全に管理するには、Vaultと組み合わせることを推奨したい。  
LAC WATCH： [Vaultの動的シークレットでTerraformのセキュリティレベルを向上させるには](https://www.lac.co.jp/lacwatch/service/20200217_002083.html)

\# provider variables  
variable "provider\_region" {  
 description = "Provider region"  
 default = "us-east-1"  
}  
  
variable "secret\_access\_key" {  
 description = "Provider access key"  
 default = "YOUR-ACCESS-KEY"  
}  
  
variable "secret\_key" {  
 description = "Provider secret key"  
 default = "YOUR-SECRET-KEY"  
}  
  
\# instance web variables  
variable "server\_instance\_type" {  
  description = "Server instance type"  
  default = "t2.micro"  
}  
  
variable "server\_tag\_name" {  
  description = "Server tag name"  
  default = "JeSuisUnDev"  
}  
  
\# instance base de données RDS postgres variables  
variable "db\_allocated\_storage" {  
  description = "Allocated storage"  
  default = 20  
}  
variable "db\_storage\_type" {  
  description = "Storage type"  
  default = "gp2"  
}  
  
variable "db\_engine" {  
  description = "Storage engine"  
  default = "postgres"  
}  
  
variable "db\_engine\_version" {  
  description = "Storage engine version"  
  default = "11.5"  
}  
  
variable "db\_instance\_class" {  
  description = "Storage instance class"  
  default = "db.t2.micro"  
}  
  
variable "db\_database\_name" {  
  description = "Storage database name"  
  default = "postgres"  
}  
  
variable "db\_database\_username" {  
  description = "Storage database name"  
  default = "postgres"  
}  
  
variable "secret\_db\_database\_password" {  
  description = "Storage database secret password"  
  default = "postgres"  
}  
  
variable "db\_snapshot\_identifier" {  
  description = "Storage database snapshot identifier"  
  default = "postgres"  
}

これで設定終了です！それでは実際に動かしてみましょう。まずTerraformを初期化、ファイルを検証し、アクションを計画（plan）して、最後に変更を適用（apply）します。Terraform CLI（Terraformが用意するコマンドライン）を使用してやってみましょう！

はじめに、Terraform初期化コマンドを入力してみましょう。これは先程ファイルを作成したフォルダで行う必要があります。

\> terraform init  
  
Initializing the backend...  
  
Initializing provider plugins...  
\- Checking for available provider plugins...  
\- Downloading plugin for provider "aws" (hashicorp/aws) 2.48.0...  
  
Terraform has been successfully initialized!

構成ファイルの記述がすべて正常であることを検証します。

\> terraform validate  
  
Success! The configuration is valid.

次の段階ではPlanを行います。今回は新しいインフラを構築するため、作成の操作のみ実行されます。

\> terraform plan  
  
An execution plan has been generated and is shown below.  
Resource actions are indicated with the following symbols:  
  + create  
  
Terraform will perform the following actions:  
  
  # aws\_db\_instance.default will be created  
  # aws\_instance.web will be created  
  
Plan: 2 to add, 0 to change, 0 to destroy.

そして今、まさにIaCの魔法がかかる瞬間です。Terraformで行った計画（plan）を実行（apply）します。これにより実際にインフラをクラウドに構築します。

\> terraform apply  
  
aws\_instance.web: Creating...  
aws\_db\_instance.default: Creating...  
aws\_instance.web: Still creating... \[10s elapsed\]  
aws\_instance.web: Still creating... \[30s elapsed\]  
aws\_instance.web: Creation complete after 43s \[id=i-0f285230749e69a67\]  
aws\_db\_instance.default: Still creating... \[50s elapsed\]  
aws\_db\_instance.default: Still creating... \[1m50s elapsed\]  
aws\_db\_instance.default: Still creating... \[2m50s elapsed\]  
aws\_db\_instance.default: Still creating... \[3m50s elapsed\]  
aws\_db\_instance.default: Still creating... \[4m0s elapsed\]  
aws\_db\_instance.default: Creation complete after 4m8s \[id=terraform-20200208064624729700000001\]  
  
Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

コードでインフラが出来上がりました。美しいですよね？さて構築が完了したので、terraform showコマンドでステータスを確認してみましょう。そして最も重要なのは、インフラを新規構築するのと同じくらい簡単にインフラを破棄できることです。

\> terraform destroy  
  
An execution plan has been generated and is shown below.  
Resource actions are indicated with the following symbols:  
  - destroy  
  
aws\_instance.web: Destroying... \[id=i-0f285230749e69a67\]  
aws\_db\_instance.default: Destroying... \[id=terraform-20200208064624729700000001\]  
aws\_instance.web: Destruction complete after 30s  
aws\_db\_instance.default: Destruction complete after 50s  
  
Destroy complete! Resources: 0 added, 0 changed, 2 destroyed.

これでインフラ全体をコードで完全に制御できます。そして、すべてをCI/CDパイプラインに入れると、何もあなたを止めることはできません。

## 最後に

もっと詳しく知りたい方は [Terraformのドキュメント](https://www.terraform.io/) をご確認ください。とても分かりやすく作られているので、参考になると思います。また、 [HashiCorpが作成したチュートリアル](https://learn.hashicorp.com/terraform) もあり、初心者にはとても役立ちます。こちらもぜひ、楽しみながら試してみてください！

![お問合せ](https://www.lac.co.jp/common/img/ico_chat01.svg)

Terraformに関するお問い合わせ

[お問い合わせフォーム](https://cp.lac.co.jp/lachp/hashicorp_contact)

この記事をシェアする

- [![noteで書く](https://www.lac.co.jp/common/img/ico_share_note01.png)](https://note.com/intent/post?url=https://www.lac.co.jp/lacwatch/service/20200903_002270.html)

メールマガジン

サイバーセキュリティや  
ラックに関する情報を  
お届けします。

[登録する](https://cp.lac.co.jp/subscription/lacmag)

関連記事

よく読まれている記事

- [![Microsoft Purviewではじめる情報漏えい対策](https://www.lac.co.jp/lacwatch/assets_c/2025/03/thumb_lacwatch_004314-thumb-400xauto-17689.png)
	2025年3月11日　|　サービス・製品
	Microsoftソリューション推進チーム
	](https://www.lac.co.jp/lacwatch/service/20250311_004314.html)
- [![社会を支える仕組みを作る、「システム開発」の仕事紹介ハンドブックを作りました](https://www.lac.co.jp/lacwatch/assets_c/2025/03/thumb_lacwatch_004324-thumb-400xauto-17705.jpg)
	2025年3月19日　|　お知らせ
	高橋 怜子
	](https://www.lac.co.jp/lacwatch/announce/20250319_004324.html)
- [![Webアプリケーションの脆弱性と、設計段階で効果的なセキュリティ対策とは](https://www.lac.co.jp/lacwatch/assets_c/2025/03/thumb_lacwatch_004334-thumb-400xauto-17730.png)
	2025年3月27日　|　サービス・製品
	齋藤 実成
	](https://www.lac.co.jp/lacwatch/service/20250327_004334.html)

タグ

- [アーキテクト](https://www.lac.co.jp/lacwatch/architect/)
- [アジャイル開発](https://www.lac.co.jp/lacwatch/agile/)
- [アプリ開発](https://www.lac.co.jp/lacwatch/app_development/)
- [インシデントレスポンス](https://www.lac.co.jp/lacwatch/incident_response/)
- [イベントレポート](https://www.lac.co.jp/lacwatch/event_report/)
- [カスタマーストーリー](https://www.lac.co.jp/lacwatch/customer_story/)
- [カルチャー](https://www.lac.co.jp/lacwatch/culture/)
- [官民学・業界連携](https://www.lac.co.jp/lacwatch/cooperation/)
- [企業市民活動](https://www.lac.co.jp/lacwatch/citizenship/)
- [クラウド](https://www.lac.co.jp/lacwatch/cloud/)
- [クラウドインテグレーション](https://www.lac.co.jp/lacwatch/cloud_integration/)
- [クラブ活動](https://www.lac.co.jp/lacwatch/club/)
- [コーポレート](https://www.lac.co.jp/lacwatch/corporate/)
- [広報・マーケティング](https://www.lac.co.jp/lacwatch/marketing/)
- [攻撃者グループ](https://www.lac.co.jp/lacwatch/attack_group/)
- [もっと見る +](https://www.lac.co.jp/lacwatch/service/#)
- [子育て、生活](https://www.lac.co.jp/lacwatch/life/)
- [サイバー救急センター](https://www.lac.co.jp/lacwatch/cyber119/)
- [サイバー救急センターレポート](https://www.lac.co.jp/lacwatch/cyber119_report/)
- [サイバー攻撃](https://www.lac.co.jp/lacwatch/cyberattack/)
- [サイバー犯罪](https://www.lac.co.jp/lacwatch/cybercrime/)
- [サイバー・グリッド・ジャパン](https://www.lac.co.jp/lacwatch/cyber_grid_japan/)
- [サプライチェーンリスク](https://www.lac.co.jp/lacwatch/supply_chain_risk/)
- [システム開発](https://www.lac.co.jp/lacwatch/system/)
- [趣味](https://www.lac.co.jp/lacwatch/hobby/)
- [障がい者採用](https://www.lac.co.jp/lacwatch/challenge/)
- [初心者向け](https://www.lac.co.jp/lacwatch/beginner/)
- [白浜シンポジウム](https://www.lac.co.jp/lacwatch/shirahama/)
- [情シス向け](https://www.lac.co.jp/lacwatch/informationsystem/)
- [情報モラル](https://www.lac.co.jp/lacwatch/moral/)
- [情報漏えい対策](https://www.lac.co.jp/lacwatch/information_leak_prevention/)
- [人材開発・教育](https://www.lac.co.jp/lacwatch/education/)
- [診断30周年](https://www.lac.co.jp/lacwatch/security_diagnosis_30th/)
- [スレットインテリジェンス](https://www.lac.co.jp/lacwatch/threat_intelligence/)
- [すごうで](https://www.lac.co.jp/lacwatch/sugoude/)
- [セキュリティ](https://www.lac.co.jp/lacwatch/security/)
- [セキュリティ診断](https://www.lac.co.jp/lacwatch/security_diagnosis/)
- [セキュリティ診断レポート](https://www.lac.co.jp/lacwatch/security_diagnostic_report/)
- [脆弱性](https://www.lac.co.jp/lacwatch/vulnerability/)
- [脆弱性管理](https://www.lac.co.jp/lacwatch/vulnerability_management/)
- [ゼロトラスト](https://www.lac.co.jp/lacwatch/zerotrust/)
- [対談](https://www.lac.co.jp/lacwatch/talk/)
- [テレワーク](https://www.lac.co.jp/lacwatch/telework/)
- [データベース](https://www.lac.co.jp/lacwatch/database/)
- [デジタルアイデンティティ](https://www.lac.co.jp/lacwatch/digital_identity/)
- [働き方改革](https://www.lac.co.jp/lacwatch/workstyle/)
- [標的型攻撃](https://www.lac.co.jp/lacwatch/targeted_attack/)
- [プラス・セキュリティ人材](https://www.lac.co.jp/lacwatch/plus_security/)
- [モバイルアプリ](https://www.lac.co.jp/lacwatch/mobileapp/)
- [ライター紹介](https://www.lac.co.jp/lacwatch/writer/)
- [ラックセキュリティアカデミー](https://www.lac.co.jp/lacwatch/academy/)
- [ランサムウェア](https://www.lac.co.jp/lacwatch/ransomware/)
- [リモートデスクトップ](https://www.lac.co.jp/lacwatch/remote_desktop/)
- [1on1](https://www.lac.co.jp/lacwatch/1on1/)
- [AI](https://www.lac.co.jp/lacwatch/ai/)
- [ASM](https://www.lac.co.jp/lacwatch/asm/)
- [CIS Controls](https://www.lac.co.jp/lacwatch/cis_controls/)
- [CODE BLUE](https://www.lac.co.jp/lacwatch/code_blue/)
- [CTF](https://www.lac.co.jp/lacwatch/ctf/)
- [CYBER GRID JOURNAL](https://www.lac.co.jp/lacwatch/cyber_grid_journal/)
- [CYBER GRID VIEW](https://www.lac.co.jp/lacwatch/cyber_grid_view/)
- [DevSecOps](https://www.lac.co.jp/lacwatch/devsecops/)
- [DX](https://www.lac.co.jp/lacwatch/dx/)
- [EC](https://www.lac.co.jp/lacwatch/ec/)
- [EDR](https://www.lac.co.jp/lacwatch/edr/)
- [FalconNest](https://www.lac.co.jp/lacwatch/falconnest/)
- [IoT](https://www.lac.co.jp/lacwatch/iot/)
- [IR](https://www.lac.co.jp/lacwatch/ir/)
- [JSOC](https://www.lac.co.jp/lacwatch/jsoc/)
- [JSOC INSIGHT](https://www.lac.co.jp/lacwatch/jsoc_insight/)
- [LAC Security Insight](https://www.lac.co.jp/lacwatch/lsi/)
- [OWASP](https://www.lac.co.jp/lacwatch/owasp/)
- [SASE](https://www.lac.co.jp/lacwatch/sase/)
- [Tech Crawling](https://www.lac.co.jp/lacwatch/tech_crawling/)
- [XDR](https://www.lac.co.jp/lacwatch/xdr/)

![](https://acq-3pas.admatrix.jp/if/5/01/bf4b8990e5cfcb70d3ed0d66e5fd1948.fs?cb=9119906&rf=https%3A%2F%2Fwww.lac.co.jp%2Flacwatch%2Fservice%2F20200903_002270.html&prf=https%3A%2F%2Fwww.google.com%2Fsearch%3Fq%3DTerraform%26rlz%3D1CDGOYI_enJP866JP867%26oq%3Dterr%26gs_lcrp%3DEgZjaHJvbWUqDggCEEUYJxg7GIAEGIoFMgYIABBFGDkyBggBEEUYPTIOCAIQRRgnGDsYgAQYigUyDQgDEAAYgwEYsQMYgAQyCggEEAAYsQMYgAQyBwgFEAAYgAQyCggGEAAYsQMYgAQyDQgHEAAYgwEYsQMYgAQyCggIEAAYsQMYgAQyDQgJEAAYgwEYsQMYgATSAQgxODUyajBqN6gCGbACAeIDBBgBIF8%26hl%3Dja%26sourceid%3Dchrome-mobile%26ie%3DUTF-8&i=11U8Me7M)