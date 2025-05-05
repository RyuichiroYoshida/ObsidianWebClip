---
title: Terraformの状態（state）をローカルではなくリモートで管理する方法
source: https://atmarkit.itmedia.co.jp/ait/articles/2407/09/news006.html
author:
  - "[[＠IT]]"
published: 
created: 2025-05-06
description: インフラ自動化ツールの一つである「Terraform」について、これから学ぼうという方、使っていきたい方を対象に、Terraformの導入方法や基本的な使い方を紹介していく本連載。今回は、Terraformの状態（state）をローカルではなくリモートで管理する方法を紹介します。
tags:
  - Tech
  - クラウド
  - "#Terraform"
read: false
---
## Terraformの状態（state）をローカルではなくリモートで管理する方法：「AWS」×「Terraform」で学ぶクラウド時代のインフラ管理入門（8）

インフラ自動化ツールの一つである「Terraform」について、これから学ぼうという方、使っていきたい方を対象に、Terraformの導入方法や基本的な使い方を紹介していく本連載。今回は、Terraformの状態（state）をローカルではなくリモートで管理する方法を紹介します。

» 2024年07月09日 05時00分 公開

この連載には“電子ブックレットのまとめ”があります

アイティメディアID会員限定コンテンツのダウンロードが可能です。  
会員登録がお済みでない方も、下のダウンロードボタンをクリック！[ダウンロード（無料）](https://ids.itmedia.co.jp/dl/atmarkit_ebook132.pdf?bpc=6f41c14d871f36f29daa43002199b640f645df0d22def323391dccfc3479973f)

![](https://atmarkit.itmedia.co.jp/ait/articles/2503/27/eBook_132.jpg)

この記事は会員限定です。 会員登録（無料） すると全てご覧いただけます。

　インフラ自動化ツールの一つである「Terraform」について、Terraformの導入方法や基本的な使い方を紹介していく [本連載](https://atmarkit.itmedia.co.jp/ait/series/35484/) 。 [前回](https://atmarkit.itmedia.co.jp/ait/articles/2406/06/news005.html) は、既存のAWS（Amazon Web Services）リソースをTerraformの管理下に置く（インポートする）方法を紹介しました。

　Terraformは、管理しているクラウドリソースに関する情報を「状態（state）」というもので記録しています。これまでTerraformを実行した際にterraform.tfstateというファイルが作成されているのを見たことがある人もいるでしょう。stateの実態はこのファイルです。

　Terraformは、管理しているクラウドリソースの情報をstateに記録しておき、クラウドリソースの状態とstateを比較して、変更の要否を判断しています。つまり、このstateがなければクラウドリソースを正しく管理できなくなります。通常はローカルにファイルとしてstateが置かれますが、複数人で管理することが難しいですし、データの破損や紛失の心配もあります。

　今回はこのstateをリモートで管理する方法を紹介します。

## stateをリモートで管理するには

[続きを読む](https://id.itmedia.co.jp/isentry/contents?sc=0c1c43111448b131d65b3b380041de26f2edd6264ee1c371184f54d26ab53365&lc=7d7179c146d0d6af4ebd304ab799a718fe949a8dcd660cd6d12fb97915f9ab0a&ac=1a599d548ac1cb9a50f16ce3ba121520c8ab7e05d54e097bfa5b82cb5a328a0f&cr=90cfa6d666682f8b5dc3c798020e432fc294ef430deb069008d4f8bceeb02418&bc=1&return_url=https%3A%2F%2Fatmarkit.itmedia.co.jp%2Fait%2Farticles%2F2407%2F09%2Fnews006.html&pnp=1&encoding=shiftjis)

　Terraformで利用できるstateの格納先（格納方法）は、本記事の執筆時点で以下の10種類です（ [参考元](https://developer.hashicorp.com/terraform/language/settings/backends/configuration) ）。

- ローカル
- Azure Storage
- Consul KV Store
- Tencent Cloud Object Storage
- Google Cloud Storage
- HTTP (REST)
- Kubernetes secret
- Alibaba Cloud OSS
- PostgreSQL
- Amazon Simple Storage Service（Amazon S3）

　これらの10種類の中からstateの格納先を選択するわけですが、複数人で管理することが前提なら、ローカルは避けましょう。前述の通り、破損や紛失の心配もありますし、仮に「Git」などを使って共有していたとしても、常に最新状態を維持する必要があるためです。複数人でTerraformを実行する場合には、その実行タイミングによってはstateの内容でコンフリクトが生じる可能性もあります。

　ローカル以外のリモートで管理する設定を選ぶことで、Terraformによってクラウドリソースが変更されるたびに自動で最新のstateがリモート上に保存されるので、複数人でTerraformを実行する際のstateのコンフリクトは発生しづらくなります。

　今回は、Amazon S3にstateを格納する方法を解説します。

## Amazon S3を利用してリモートでstateを管理する

　Amazon S3上にstateを保存する場合、stateを保存するためのS3バケットを事前に作成する必要があります。AWSの管理コンソールからAmazon S3の管理画面を開き、新しくS3バケットを作成します。今回は例としてバケット名を「terraform-state.sios.jp」にしていますが、バケット名はグローバル名前空間で一意である必要がありますので、ここは好きな名前に置き換えてください。

[![S3バケットの作成例](https://image.itmedia.co.jp/ait/articles/2407/09/ait_240709_terraform8_1.jpg)](https://image.itmedia.co.jp/l/im/ait/articles/2407/09/l_ait_240709_terraform8_1.jpg) S3バケットの作成例

　S3バケットが作成できたら、次はTerraform側でこのS3バケットをstateの格納先に設定しましょう。 [第1回](https://atmarkit.itmedia.co.jp/ait/articles/2306/26/news006.html) で設定したTerraform コードであるterraformブロック内に、backendという新しい設定を追記します。

```
terraform {  required_providers {    aws = {      source = "hashicorp/aws"      version = "~> 4.0"    }    backend "s3" {      bucket = "terraform-state.sios.jp"      region = "ap-northeast-1"      key    = "example.tfstate"    }  }}
```

　設定できたら、terraform initコマンドを実行します。ローカルに保存していた既存のstateを、今回設定したAmazon S3上にコピーするかどうかを聞いてきますので、「yes」と答えましょう。すると、既存の情報を維持したままAmazon S3上でstateを管理できるようになります。

```sh
$ terraform init
Initializing the backend...
Do you want to copy existing state to the new backend?
  Pre-existing state was found while migrating the previous "local" backend to the
  newly configured "s3" backend. No existing state was found in the newly
  configured "s3" backend. Do you want to copy this state to the new "s3"
  backend? Enter "yes" to copy and "no" to start with an empty state.
  Enter a value: yes
Successfully configured the backend "s3"! Terraform will automatically
use this backend unless the backend configuration changes.
Initializing provider plugins...
- Reusing previous version of hashicorp/aws from the dependency lock file
- Using previously-installed hashicorp/aws v4.67.0
Terraform has been successfully initialized!
You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.
If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

　正常にstateをAmazon S3で管理できるようになったかどうかを確認してみます。AWSの管理コンソールから、今回作成したS3バケットの中身を見てみると、「example.tfstate」というファイルが作成されていることが確認できます。

[![S3バケットの内容](https://image.itmedia.co.jp/ait/articles/2407/09/ait_240709_terraform8_2.jpg)](https://image.itmedia.co.jp/l/im/ait/articles/2407/09/l_ait_240709_terraform8_2.jpg) S3バケットの内容

　これでstateをリモートで管理する状態になったはずです。試しに、ローカルに残っている「terraform.tfstate」ファイルを削除した上で、terraform planコマンドを実行してリソースの状況に変化がないかどうかを確認してみます。

```sh
$ rm -f terraform.tfstate
$ terraform plan
aws_instance.test: Refreshing state... [id=i-xxxxxxxxxxxxxxxxx]
No changes. Your infrastructure matches the configuration.
Terraform has compared your real infrastructure against your configuration and
found no differences, so no changes are needed.
```

　ローカルのstateファイルを削除しても、新しくインスタンスを作成するような動作にならないため、正しくリモートでstateを管理できていることが分かります。

　stateをリモート管理することで複数人での管理にも適したTerraform環境にできるため、チームでの運用時にも活用できます。ぜひ試してみてください。

[この連載を「連載記事アラート」に登録する](https://atmarkit.itmedia.co.jp/ait/articles/2407/09/) New

## 筆者紹介

サイオステクノロジー所属。OSS よろず相談室でサポート対応をしているテクニカルサポートエンジニア。Ansibleに出会ってから自動化に取りつかれ、自身の業務やプライベートであらゆるものの自動化に取り組む。プライベートではJavaでちょっとしたツールの開発を趣味にしている。

[サイオス OSS よろず相談室](https://sios.jp/lp/yorozu/)

[サイオステクノロジーエンジニアブログ](https://tech-lab.sios.jp/)

[サイオステクノロジーエンジニア YouTube チャンネル](https://www.youtube.com/channel/UCjIVEOLmZlBrgq7nrxVFuRw)

- [![Terraformで複数のAWS EC2インスタンスを作成、管理する方法](https://image.itmedia.co.jp/ait/articles/2309/08/news004.png) Terraformで複数のAWS EC2インスタンスを作成、管理する方法](https://atmarkit.itmedia.co.jp/ait/articles/2309/08/news004.html)
- [![「ダブルチェックを頑張る」でごまかさない、スクウェア・エニックスのサーバ設定漏れ防止策](https://image.itmedia.co.jp/ait/articles/2310/23/news011.jpg) 「ダブルチェックを頑張る」でごまかさない、スクウェア・エニックスのサーバ設定漏れ防止策](https://atmarkit.itmedia.co.jp/ait/articles/2310/23/news011.html)
- [![設定ミスの社外秘情報はググれます――サイバー攻撃者はどうやってクラウドを墜とすのか？](https://image.itmedia.co.jp/ait/articles/2404/05/news025.png) 設定ミスの社外秘情報はググれます――サイバー攻撃者はどうやってクラウドを墜とすのか？](https://atmarkit.itmedia.co.jp/ait/articles/2404/05/news025.html)

Special PR

[印刷／保存](https://id.itmedia.co.jp/isentry/contents?sc=0c1c43111448b131d65b3b380041de26f2edd6264ee1c371184f54d26ab53365&lc=7d7179c146d0d6af4ebd304ab799a718fe949a8dcd660cd6d12fb97915f9ab0a&return_url=https://ids.itmedia.co.jp/print/ait/articles/2407/09/news006.html&encoding=shift_jis&ac=e8cb9106baa7e37eb9feb877b9f0a27ddaf48b95ba02da49cbb3a8247ee7fec4&cr=e9fd42802bc22856808963077023568339063544b05e5a8646e62c02a898e0fd "この記事を印刷する")

[連載通知](https://id.itmedia.co.jp/isentry/contents?sc=0c1c43111448b131d65b3b380041de26f2edd6264ee1c371184f54d26ab53365&lc=7d7179c146d0d6af4ebd304ab799a718fe949a8dcd660cd6d12fb97915f9ab0a&return_url=https%3A%2F%2Fid.itmedia.co.jp%2Fapp%2Falert%2Fregist_setting%3Furl%3Dhttps%3A%2F%2Fatmarkit.itmedia.co.jp%2Fait%2Farticles%2F2407%2F09%2Fnews006.html%26type%3D2&encoding=shift_jis&ac=8b70865e13a2d61cf96c34172b9312018dffa6d8d496609bf9fd8314fd091e1a&cr=4634196a317325dfc00aa8fe3ea82f36ad62e80b103fcbdaf13b4d39fa261577 "「AWS」×「Terraform」で学ぶクラウド時代のインフラ管理入門")

スポンサーからのお知らせ PR

Special PR

**本日** **月間**

## 編集部からのお知らせ

[【Amazonギフトカード プレゼント】5月12日（月）開催【無料オンラインセミナー】『＠IT 運用管理セミナー 2025 春 システム運用管理の足腰を鍛える 2025年の壁を突破せよ』＠IT人気連載『IT訴訟 徹底解説』著者 細川義洋氏による【基調講演　裁判所調停委員が語る、運用・保守トラブルのリスクと課題】、テックカンファレンス「SRE NEXT」のFounder 北野勝久氏による【基調講演　SRE本出版からまもなく10年！ これまでに何が起こり、これから何が起こるのか。】などを配信](https://rd.itmedia.co.jp/8hnf#utm_source=ait&utm_content=rightcolumn_info)

あなたにおすすめの記事 PR