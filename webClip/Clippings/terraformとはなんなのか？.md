---
title: terraformとはなんなのか？
source: https://qiita.com/souhei-etou/items/7876767f041543321006
author:
  - "[[Qiita]]"
published: 2021-09-04
created: 2025-05-06
description: 最近terraformを使う機会があったのでメモがてら残しておこうと思います。対象者terraformに興味はあるが実際に触ったことがない人IaCを触ってみたい人大雑把にterraformの…
tags:
  - Tech
  - Terraform
  - バックエンド
  - クラウド
  - 学習系
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から3年以上が経過しています。

[@souhei-etou](https://qiita.com/souhei-etou)

投稿日

最近terraformを使う機会があったのでメモがてら残しておこうと思います。

## 対象者

- terraformに興味はあるが実際に触ったことがない人
- IaCを触ってみたい人
- 大雑把にterraformの概念だけ知っておきたい人

## terraformとは？

terraformはIacと呼ばれるツールの一種で、インフラの構成をソースコードとして管理できる機能です。  
注意点として、あくまで管理できるのはインフラ構成であって、サーバー内のユーザー設定やライブラリのインストールなどはできません。  
もし、サーバー内も管理したい場合は、imageでデプロイするか、ansibleなどを併用して管理していく必要が出てきます。

## terraformの実行順序

実際にファイルを書く前に、terraformが実際に動く順序だけ軽く記載します。terraformのCLIが入っている前提です。

- terraformファイルを書く
- terraform init で初期化する
	- 対象cloudへの認証処理が走る
	- 依存ファイルを取得してくる
	- リソースなどメタ情報が詰まったファイル(state ファイル)が生成される
- 実行結果を確認しながらファイルを修正
- 実際に実行してインフラなどを立ち上げる

明示的にインストールコマンドを叩かないなどの違いはありますが、感覚的には、npmやgoのget moduleなどに近いです。

## terraformの特性

terraformはjsonなどのような汎用的なファイル形式で管理するのではなく、独自の拡張子のtfファイルとプログラミング言語で処理を記述します。以下は公式で記載されているサンプルです。

```terraform
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.27"
    }
  }

  required_version = ">= 0.14.9"
}

provider "aws" {
  profile = "default"
  region  = "us-west-2"
}

resource "aws_instance" "app_server" {
  ami           = "ami-830c94e3"
  instance_type = "t2.micro"

  tags = {
    Name = "ExampleAppServerInstance"
  }
}
```

## terraform

一番最初に書かれている `terraform` と言うブロックはterraform自体の設定をするところです。依存するライブラリのバージョンを指定したり、 メタ情報が詰まったstateファイルを置く場所を指定できます。

## provider

`provider` はAWSなどのcloudと通信するための情報の指定です。  
リージョンや認証情報などがそうですね。  
今回はAWSのprofileを指定して認証情報を読みとって指定し、リージョンは明示的に指定しています。

他のproviderを見たい場合は以下から参照してください。  
[https://registry.terraform.io/browse/providers](https://registry.terraform.io/browse/providers)

## resource

`resource` はDBやサーバーなどの実際に構築するインフラの指定です。  
resource の後ろの文字列はそれぞれ、リソース名,変数名を指定します。  
リソース名でAWSのlamdaやdynamoなどのインフラを指定して、設定内容をresourceブロックの中に書いていきます。

変数名はresouceで指定したインフラの中身を参照したい時に使います。以下のような感じですね。

```terraform
resource "aws_instance" "app_server" {
  ami           = "ami-830c94e3"
  instance_type = "t2.micro"
  name ="test"

  tags = {
    Name = "ExampleAppServerInstance"
  }
}

resource "aws_db" "db" {
    # app_serverの情報を参照
    name = app_server.name
}
```

実際のawsのresourceなどは以下に書いてあるので、確認してみてください。  
[https://registry.terraform.io/providers/hashicorp/aws/latest/docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)

## 初期化と確認と実行

## 初期化

さてファイルが書けたら、terraformコマンドの  
`terraform init` を実行しましょう。これで初期化と認証が走ります。無事providerで指定したプラットフォームにアクセスし、依存ファイルを取得できたら初期化完了です。成功すると、ファイルが置いてあるディレクトリに  
`.terraform`  
`.terraform.lock.hcl`  
が生成されます。

## 確認

`terraform plan` を実行すると、以下のことをチェックしてくれます。

- 使用しているアカウントの権限のチェック
	- AWSでEC2インスタンを立ち上げる権限を持っているかなど
- 文法のチェック
- 最終的に生成されるリソースのチェック
	- DBのアカウントなど

## 実行

`terraform apply` を実行すると、実際にリソースを生成してくれます。  
正常終了した場合は、実際にプラットーフォームのコンソールで、指定したリソースが生成されているかを確認してみましょう。

## 最後に

インフラは秘伝のタレになりがちなので、コードで管理できれば色々嬉しいですね。  
それでは良いterraform ライフを

## 引用

[0](https://qiita.com/souhei-etou/items/#comments)

コメント一覧へ移動

[132](https://qiita.com/souhei-etou/items/7876767f041543321006/likers)

いいねしたユーザー一覧へ移動

91