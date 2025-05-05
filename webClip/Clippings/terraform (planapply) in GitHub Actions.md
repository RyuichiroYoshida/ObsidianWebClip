---
title: terraform (plan|apply) in GitHub Actions
source: https://knowledge.sakura.ad.jp/38553/
author:
  - "[[さくらのナレッジ]]"
published: 2024-07-25
created: 2025-05-06
description: この記事は、2024年6月24日(月)に行われた社内IaC勉強会における発表を、さくナレ編集部にて記事化したものです。 目次はじめにterraform (plan|apply)を実行する際のポイントtfstateの管理t […]
tags:
  - CI-CD
  - Tech
  - Terraform
  - クラウド
  - バックエンド
  - IaC
read: false
---
![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/07/t.png)

> この記事は、2024年6月24日(月)に行われた社内IaC勉強会における発表を、さくナレ編集部にて記事化したものです。

## はじめに

さくらインターネット SRE室の久保です。

今日は「terraform (plan|apply) in GitHub Actions」というタイトルで発表させていただきます。

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/07/20240624-kubo-1-2-680x272.jpg)

今日発表する内容は、画像で表すと上図のようになります。誰かがPull Requestを送ると、それをもとにGitHub Actionsを動かし、Terraformのplanやapplyを動かして、自動的にTerraform管理下にあるリソースを更新してくれる、そういう仕組みを作ったという話です。

## terraform (plan|apply)を実行する際のポイント

Terraformのplanとapplyを実行する際のポイントとして、まず各種秘匿情報、具体的にはAPIキーなどが必要になるので、実行結果をチーム内で共有してレビューするのが結構面倒です。何らかの方法でAPIキーを共有して使うにしても、あるいは各自で個別にAPIキーを管理するにしても、やはり煩雑でとても面倒な作業になります。

それから、Terraformにはtfstateというファイルがあり、これを使ってTerraformがリソースの状態を管理しています。このtfstateは一意でなければなりません。

## tfstateの管理

tfstateは秘匿情報を含むことがあるので、リポジトリにはコミットしないようにします。ではリビジョン管理はどうするかというと、オブジェクトストレージがバージョニングという機能を提供しているので、これを利用しましょうということになります。

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/07/20240624-kubo-fig2-680x146.jpg)

あとはこれはGitHubの話になりますが、Branch Protection Ruleで保護するというのも重要です。複数のPull Requestをまとめてマージしたりすると、そのときにTerraformのapplyが同時に走ってしまって、その結果、tfstateが壊れることが起こり得ます。これを防ぐ方法はいろいろありますが、一番簡単に防ぐ方法はBranch Protection Ruleの「Require status checks to pass before merging」と「Require branches to be up to date before merging」にチェックを入れておくことです(上図参照)。特定のブランチをマージするときに、mainブランチの変更に追従していない場合はマージできないようになるので、複数のPull Requestをまとめてマージするのを防ぐことができます。TerraformのplanとapplyをGitHub Actionsでやるときは、これのチェックは絶対に入れましょう。

## tfstateをさくらのオブジェクトストレージに格納する

tfstateをさくらのオブジェクトストレージに入れる場合の設定ですが、まずbackend.tfは以下のような感じで設定を記述します。

```
terraform {
 backend "s3" {
  bucket  = "(バケット名)"
  key   = "(オブジェクトキー)"
  region  = "(リージョン)"
  endpoints = {
   s3 = "https://s3.isk01.sakurastorage.jp"
  }

  skip_credentials_validation = true
  skip_requesting_account_id = true
  skip_region_validation   = true
  skip_metadata_api_check   = true
  skip_s3_checksum      = true
 }
}
```

バージョニングを有効にするのは、AWS CLIで以下のコマンドを実行します。

```
aws --endpoint-url="https://s3.isk01.sakurastorage.jp" s3api put-bucket-versioning --bucket (バケット名) –versioning-configuration Status=Enabled
```

## SRE室における実例

実際にSRE室でTerraformのplanとapplyをGitHub Actionsで実行している例としてはマニュアルサーバがあります。今年の2月か3月ぐらいにマニュアルサーバをリプレースしたのですが、そのときにTerraformを導入しまして、ここでGitHub Actionsでplanとapplyの実行を自動化しています。それから、モニタリングサービスのMackerelの設定も、できるところは基本的に全部Terraformで管理しています。また、DNSの設定やGitHubのorganizationメンバーやチームもTerraformで管理できないか検証を始めました。

## tfcmt

[tfcmt](https://github.com/suzuki-shunsuke/tfcmt) というのは、Terraformの実行結果を整形して出力するためのユーティリティです。これを使うと、その出力結果をGitHubのコメントとして出力することができるようになります。これはメルカリが開発してOSSとして公開している [tfnotify](https://github.com/mercari/tfnotify) という同種のツールからforkしたもので、tfnotifyの機能を絞って、代わりにいくつかの機能を強化したようなものになっています。

### tfcmtの使い方

tfcmtの使い方は下図のような感じで、tfcmtのコマンドに、出力フォーマットなどを定義したYAMLファイルを-configで設定して、それから--の後ろにterraformのplanないしapplyのコマンドを引数で渡すという形になっています。

```
# plan
tfcmt -config .tfcmt.yml plan -- terraform plan
# apply
tfcmt -config .tfcmt.yml apply -- terraform apply
```

### tfcmtの設定

tfcmtの設定については、まず.tfcmt.ymlの上の部分を抜粋して示します。

```
repo_owner: (オーナー名)
repo_name: (リポジトリ)
ghe_base_url: https://(GHESのホスト名)
ghe_graphql_endpoint: https://(GHESのホスト名)/api/graphql
```

ここではGitHub Enterprise Server(GHES)用の設定やリポジトリ関連の設定を行っています。リポジトリのオーナー、リポジトリ名、それから、さくらではGitHub Enterprise Serverを利用しているので、そのためのURLの設定も行っています。

続いて後半部分がこちらです。planとapplyの実行結果の出力フォーマットを指定しています。

```
terraform:
  plan:
    template: |
      {{template "plan_title" .}} <sup>{{if .Link}}[CI link]({{.Link}}){{end}}</sup>
      {{ .Message }}
      {{if .Result}}
      <pre><code>{{ .Result }}
      </pre></code>
      {{end}}
      <details><summary>Details (Click me)</summary>
      <pre><code>{{ .CombinedOutput }}
      </pre></code></details>
  apply:
    template: |
      {{ .Title }}
      {{ .Message }}
      {{if .Result}}
      <pre><code>{{ .Result }}
      </pre></code>
      {{end}}
      <details><summary>Details (Click me)</summary>
      <pre><code>{{ .CombinedOutput }}
      </pre></code></details>
```

### tfcmtの実行例

ではtfcmtの実行例を紹介します。

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/07/20240624-kubo-fig5.png)

Pull Requestを送ると、上図のようにterraform planが走り、tfcmtがそれを受け取って、GitHubコメントとしてその結果を出してくれます。実行結果には「Details(Click me)」という項目が表示されており、これをクリックするとterraform planの詳細が出るようになっています。一方、applyの方はマージしたときに動くようになっていて、これも「Details(Click me)」をクリックすると詳細が出るようになっています。

## IaC via Terraformでtfcmt(やtfnotify)を使う利点

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/07/20240624-kubo-19-2-680x383.jpg)

Terraformのplanとapplyをtfcmtやtfnotifyで実行することのメリットとしては、Terraformを利用した構成管理を通常の開発と同じにできるというのがあります。Pull Requestを作ってそれをコードレビューしてapproveした後、マージしてデプロイする、これがあの一連の流れの中ですべてできるというのが大きなメリットです。あと単純に楽というのもあります。実際に使っていてすごく楽で、上司からもそういう声をもらっています。

## まとめ

GitHub Actionsを使ってTerraformのplanやapplyを動かし、Terraform管理下にあるリソースを自動的に更新する仕組みを作った話をしました。このようにしてTerraformのplanやapplyは手動ではなくCI/CDで自動化して走らせようというのが本記事の主旨になります。発表は以上です。ありがとうございました。