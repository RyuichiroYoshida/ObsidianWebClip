---
title: スケーラブルなUnityのCI環境の実現｜カバー株式会社　公式note
source: https://note.cover-corp.com/n/n09132dda6669
author:
  - "[[カバー株式会社　公式note]]"
published: 2024-04-26
created: 2025-05-06
description: こんそめ～🐻💿 カバー株式会社CTO室エンジニアのAです。今回は、タレントの皆さんが普段の配信で使用している「ホロライブアプリ」の開発における、CI環境の改善の取り組みについてご紹介します。  はじめに  CI（Continuous Integration）とは、システム開発における、複数人の開発者が書いたコードを頻繁に統合し、自動化されたテストを行うことでコードの品質を保つ手法です。  CI とは何ですか? - 継続的インテグレーションの説明 - AWSaws.amazon.com  CIを導入することで、短期間でのバグの発見、統合作業の分散による生産性の
tags:
  - CI-CD
  - 1
  - Tools
  - クラウド
  - Unity
read: false
---
![見出し画像](https://assets.st-note.com/production/uploads/images/138330194/rectangle_large_type_2_91181c56e3dc4a3fb373373fbe45f20f.png?width=1200)

## スケーラブルなUnityのCI環境の実現

[カバー株式会社　公式note](https://note.cover-corp.com/)

こんそめ～🐻💿  
カバー株式会社CTO室エンジニアのAです。今回は、タレントの皆さんが普段の配信で使用している「ホロライブアプリ」の開発における、CI環境の改善の取り組みについてご紹介します。

## はじめに

CI（Continuous Integration）とは、システム開発における、複数人の開発者が書いたコードを頻繁に統合し、自動化されたテストを行うことでコードの品質を保つ手法です。

[**CI とは何ですか? - 継続的インテグレーションの説明 - AWS** *aws.amazon.com*](https://aws.amazon.com/jp/devops/continuous-integration/)

CIを導入することで、短期間でのバグの発見、統合作業の分散による生産性の向上、常にリリース可能にコードの品質を保つことによる開発サイクルの高速化などの利点が見込まれます。

ホロライブアプリはUnityで開発しており、具体的なCIプロセスとして、コードのテストとアプリケーションのビルドがそれに当たります。CIに関連するコンセプトとして [CD（Continuous Delivery）](https://aws.amazon.com/jp/devops/continuous-delivery/) というものもあり、これらをまとめてCI/CDプロセスとしてとらえて扱うこともあります。

さて、いままでのホロライブアプリは、CI/CDパイプラインとして [GitHub Actions](https://docs.github.com/ja/actions/learn-github-actions/understanding-github-actions) を利用し、WindowsとMacそれぞれ1台ずつのオンプレマシンを [Self-hosted Runner](https://docs.github.com/ja/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners) として登録した環境で開発を行っていました。Self-hosted Runnerを利用している理由としては、Unityのテスト・ビルドを行うために、あらかじめUnityがインストールされた環境をRunnerとする必要があるためです。

GitHub Actionsの利用については、弊社のKさんがこちらの記事で詳しく紹介しております。

[**GitHub ActionsでUnityのお手軽CI - Qiita** *こんにちは。カバー株式会社エンジニアのKです。弊チームでは、UnityプロジェクトのビルドにGitHub Actionsを* *qiita.com*](https://qiita.com/kura_cvr/items/872cae5620a7be8a60ad)

こちらも併せてご参照ください。

ところが、ホロライブアプリの開発規模が拡大するにつれ、直接操作して環境を整備する必要があるオンプレマシンの管理のしにくさや、テスト・ビルドが渋滞してしまい快適な開発ができないなどの問題点が浮上してきました。

![画像](https://assets.st-note.com/img/1713938629252-5aLtrVY4aI.png?width=1200)

これに対して、AWS EC2の仮想サーバーをスケーラブルな（必要な時に必要な台数だけ起動できる）ビルドサーバーとして用いるシステムを構築しました。

今回のCI環境構築にあたり、こちらの記事シリーズを大いに参考にさせていただきました。

[**運用開始から4か月経ってから振り返るAWS&GitHub&Terraformを使ったUnity CI環境 - Activ8 Tech Blog** *エンジニアの岡村です。 このテックブログでは以前、AWS EC2でUnity環境を立ち上げ、GitHub Actionsの* *synamon.hatenablog.com*](https://synamon.hatenablog.com/entry/2022/09/22/153523)

この場を借りてお礼申し上げます。

## terraform-aws-github-runnerの利用

インスタンスを管理するシステムの構築には、以下のOSSを利用させていただきました。

このリポジトリは、GitHub Actionsの実行に応じてEC2インスタンスを起動・削除するAWSインフラを構築するTerraformモジュールと、その中で動くLambdaのソースコード等を提供します。詳細なドキュメントが用意されているため、その通りに設定すると下図のようなAWSインフラが簡単に構築できます。

![画像](https://assets.st-note.com/img/1713938662802-J51dDSG2Ih.png?width=1200)

（画像は [https://philips-labs.github.io/terraform-aws-github-runner/#detailed-design](https://philips-labs.github.io/terraform-aws-github-runner/#detailed-design) より引用）

構築したインフラは次のようなフローを実現します。

- Actions Workflowが実行されると、WebhookがLambdaに通知を送る。
- 受け取った通知から必要なEC2インスタンスを起動する。
- GitHub Appが起動したインスタンスをSelf-hosted Runnerとして登録する。
- インスタンスは指定の時間おきに使用状況を確認され、使用されていないインスタンスはGitHub AppによりRunnerの登録を解除され、削除される。

また、通常のSelf-hosted Runnerと同様に、Runnerごとにラベルを付与することで、Actions WorkflowのラベルとマッチしたRunnerのインスタンスを起動することが可能です。さらに、起動するインスタンスはカスタムAMIを含むAMIを指定できるため、複数の環境をRunnerインスタンスとして利用し、ラベルで使い分けることが可能です。

## EC2インスタンスについて

こちらの項の内容は2023年の [弊社アドベントカレンダー](https://qiita.com/advent-calendar/2023/cover) への投稿記事でも取り上げさせていただきました。

[**PackerでUnityが動作するUbuntu環境AMIをビルドする - Qiita** *こんにちは、カバー株式会社エンジニアのAです。本記事では表題の通り、Unityが実行可能なUbuntu環境のAMIをビルド* *qiita.com*](https://qiita.com/ra-cover/items/c97949bebbbcabf8e6d9)

本記事では上記記事の内容をかいつまんでご紹介します。詳しくは上記記事をご参照ください。

ホロライブアプリにはiOS版とWindows版があり、それらを1つのUnityプロジェクトで開発しています。  
iOS版のビルドはアプリを配信するフローの都合によりオンプレ環境で実行しているため、EC2インスタンスで実行したいActions Workflowは

- iOS版テスト
- Windows版テスト
- Windows版ビルド

の３つになります。このうち、Windows版ビルドはビルド結果をインストーラーにパッケージする都合により、Windows環境で実行する必要がありました。

terraform-aws-github-runnerでは、起動するインスタンスの種類をon-demand（通常のEC2インスタンス）とspotから選択できます。EC2 Spot InstanceはAWSの余剰リソースを安価に使えるオプションです。

[**Amazon EC2 スポットインスタンス | AWS** *Amazon EC2 スポットインスタンスは AWS クラウドでの未使用の計算容量で、オンデマンド料金と比べて、お客様に大* *aws.amazon.com*](https://aws.amazon.com/jp/ec2/spot/)

安価に使えることのトレードオフとして、spotインスタンスはEC2が容量を必要としたときに中断される可能性があります。Linuxインスタンスはon-demandから70% offくらいの料金で利用でき、Windowsと比較して安価に利用できることを確認したため、Linuxで実行可能なiOS/Windows版テストはLinux環境で実行することにしました。Windowsインスタンスについては、spotの場合、開発段階でインスタンスが停止されることが数回あり、またビルドはテストと比較して時間もかかりあまり中断されたくなかったため、on-demandを利用することにしました。  
以上、実行環境と実行するWorkflowをまとめると次のようになります。

- Linux環境spotインスタンス
	- iOS版テストの実行
	- Windows版テストの実行
- Windows環境on-demandインスタンス
	- Windows版ビルドの実行

インスタンスタイプはすべての環境でm6a-xlargeを利用しています。

さて、本件ではインスタンス起動時にUnityがインストールされている必要があります。これについては、Unityが動作するWindows/Linux環境の [AMI](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/AMIs.html) を作成し、インスタンス起動時にそれぞれのWorkflowに合致したAMIを使用するように設定することで実現しています。AMIは、terraform-aws-github-runnerが提供するサンプルをもとに、 [Packer](https://www.packer.io/) で記述しビルドしました。

  

![画像](https://assets.st-note.com/img/1713938696644-3lZwQSU09P.png?width=1200)

## ライセンスサーバの導入

Unity Editorの実行にはマシンごとにライセンスが必要ですが、本件のようなスケーラブルに使い捨てるインスタンスのそれぞれに通常のライセンスを付与することはできません。そのため本件では、UnityがCI/CD用ライセンスとして提供している [Unity Build Server](https://unity.com/ja/products/unity-build-server) のFloating Licenseを利用しています。

[**Unity Licensing Server** *docs.unity.com*](https://docs.unity.com/licensing/en-us/manual)

このライセンスは構築したライセンスサーバーに購入した数が保持され、このライセンスを利用するように設定したクライアントは、Unity Editorを実行する際にサーバーに接続してライセンスを受け取り、終了したら返却することで動作します。そのため、購入したライセンス数がビルドサーバーの同時実行可能数になります。ホロライブアプリでは3ライセンスを購入し利用しています。また、このライセンスは通常のライセンスと比較して安価に利用できますが、Unity EditorのGUI操作はできません。

ライセンスサーバーはビルドサーバーと異なるVPCのEC2上に構築し、Linuxのt2.microインスタンス上で常時起動しています。またビルドサーバーとはPeering接続により相互通信可能にしました。

  

![画像](https://assets.st-note.com/img/1713938704775-CvR6TWlefx.png?width=1200)

クライアント（本件ではUnityを実行するEC2ビルドサーバーインスタンス）はライセンスサーバーの接続設定を記入したconfigファイルを指定のディレクトリに置くだけで、勝手にライセンスを取得するよう動作してくれます。しかし本件のLinux環境では、都合によりUnityを通常とは異なる/opt下にインストールしたため、うまく動作してくれませんでした。そのためWorkflowにおいて、Unity Editorでテスト・ビルドを実行する前にCLIからライセンスを取得しています。 以下はWorkflowファイルのサンプルになります。

```ruby
...
- name: Test Unity Licensing Client # 事前にライセンスを取得
  run: |
     /opt/Unity/Hub/Editor/2020.3.33f1/Editor/Data/Resources/Licensing/Client/Unity.Licensing.Client --acquire-floating

- name: Run Test # テスト実行
  run: |
    xvfb-run --auto-servernum --server-args='-screen 0 640x480x24' \
    "/opt/Unity/Hub/Editor/2020.3.33f1/Editor/Unity" -batchmode -projectPath . -buildTarget StandaloneWindows64 -executeMethod BuildApp.RunEditorTests -logFile -
    if [ $? -ne 0 ]; then
      echo "Test Failed!"
      exit 1
    fi
...
```

## Github App tokenによる認証

ビルドサーバーがGitHubからソースコードを取得する際、GitHub Acrionsは通常では実行元リポジトリのアクセス権限しか持たず、プライベートリポジトリのSubmoduleがある場合はActions Workflowにそれらへのアクセス権限を付与しなければなりません。これに対して本件では、GitHub Appを用いた認証を利用しています。

[**GitHub Actions ワークフローで GitHub App を使用して認証済み API 要求を作成する - GitHub Docs** *GitHub App からのインストール アクセス トークンを使って、GitHub Actions ワークフローで認証済み* *docs.github.com*](https://docs.github.com/ja/apps/creating-github-apps/authenticating-with-a-github-app/making-authenticated-api-requests-with-a-github-app-in-a-github-actions-workflow)

これはまずSubmoduleへのアクセス権を付与したGitHub Appを作り、Workflow内でそのGitHub Appによりアクセストークンを生成し、それを利用することでプライベートSubmoduleにアクセス可能になります。

  

![画像](https://assets.st-note.com/img/1713938717065-Nrhcot5Qc1.png?width=1200)

トークンの生成には以下の公式のActionsを利用しています。

以下はWorkflowファイルのサンプルになります。

```ruby
...
- name: Generate App Token
  uses: actions/create-github-app-token@v1
  id: app_token
  with:
    app-id: ${{ secrets.APP_ID }}
    private-key: ${{ secrets.APP_PRIVATE_KEY }}
    owner: ${{ github.repository_owner }}
    repositories: "ThisRepo,Submodule1,Submodule2" # 実行元リポジトリも含める必要がある
    skip-token-revoke: true # トークンは1時間で失効し、Workflowが1時間以上かかる場合、失効したトークンをrevokeしようとしてエラーが出るため

- name: Checkout
  uses: actions/checkout@v4
  with:
    lfs: true
    clean: true
    persist-credentials: false
    submodules: recursive # すべてのSubmoduleも含めて再帰的にチェックアウトする
    token: ${{ steps.app_token.outputs.token }} # 上で生成したトークンを参照
...
```

## キャッシュシステムの導入

ここまでで、Unityが実行可能なビルドサーバーをActionsの実行に応じてスケーラブルに立ち上げ、コードの変更を反映してUnityのテスト・ビルドを実行するシステムが構築できました。

さて、ここで立ち上がったビルドサーバーは一度もUnityプロジェクトを起動していないまっさらな状態で、Unityでの開発経験のある方ならわかると思いますが、プロジェクトの規模が大きいほどUnity Editorを実行する際に走るアセットのインポートが膨大になり、時間がかかってしまいます。そこで、インポートの結果生成されるLibraryディレクトリをキャッシュとして利用する仕組みを導入しました。

GitHub Actionsでキャッシュを利用したい場合、GitHub公式が提供しているキャッシュ機能を利用することが可能です。しかしこちらは保存可能容量がリポジトリごとに10GBまでの制限があり、Unityの膨大なキャッシュを保存するとすぐに容量を使い果たしてしまいます。そこで、本件ではキャッシュをAWS S3に保存しています。

![画像](https://assets.st-note.com/img/1713938802557-wg1qnxb5EL.png?width=1200)

キャッシュをS3から取得・保存するために、こちらのサードパーティのActionsを導入しました。

こちらは公式のキャッシュ機能と同様の記述方法で利用することが出来ます。  
キャッシュファイルの命名は「{ブランチ名}-{commit hash}.tgz」とし、commitごとに保存することにしました。  
取得時は

- 同じブランチの最新のキャッシュを取得
- なければdevelopブランチの最新のキャッシュを取得

するよう設定しています。  
以下はWorkflowファイルのサンプルになります。

```ruby
...
# S3にキャッシュを保存する際に/が含まれているとエラーになるため、/を_に置換する
- name: Modify ref name 
  run: |
    modified_ref_name="${GITHUB_REF#refs/heads/}"
    modified_ref_name="${modified_ref_name//\//_}"
    echo "MODIFIED_REF_NAME=${modified_ref_name}" >> $GITHUB_ENV

# プルリクから発火したときは、github.head_refからブランチ名を取得する
- if: ${{ github.event_name == 'pull_request' }}
  run: |
    modified_ref_name=${{ github.head_ref }}
    modified_ref_name="${modified_ref_name//\//_}"
    echo "MODIFIED_REF_NAME=${modified_ref_name}" >> $GITHUB_ENV

- name: Restore cache
  uses: whywaita/actions-cache-s3@v2
  with:
    aws-s3-bucket: cache-bucket
    aws-access-key-id: ${{ secrets.CACHE_USER_ACCESS_KEY }}
    aws-secret-access-key: ${{ secrets.CACHE_USER_SECRET_KEY }}
    aws-region: ap-northeast-1
    aws-s3-bucket-endpoint: false
    aws-s3-force-path-style: true
    key: actions-cache/linux/${{ env.MODIFIED_REF_NAME }}-${{ github.sha }}.tgz
    path: |
      Library/
    restore-keys: |
      actions-cache/linux/${{ env.MODIFIED_REF_NAME }}
      actions-cache/linux/develop
...
```

  
ブランチ名を取得してキャッシュファイルに命名するために、Actions Workflowの [環境変数](https://docs.github.com/ja/actions/learn-github-actions/variables) や [コンテキスト](https://docs.github.com/ja/actions/learn-github-actions/contexts) を利用しています。

キャッシュを保存するS3バケットには [ライフサイクル](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/userguide/object-lifecycle-mgmt.html) を設定することで、無限にキャッシュが蓄積されないようにしています。  
以下はTerraformコードのサンプルになります。

```swift
...
# lifecycle設定
resource "aws_s3_bucket_lifecycle_configuration" "cache_bucket_versioning_config" {
  bucket = aws_s3_bucket.cache_bucket.id
  rule {
    id = "test-cache-expiration-rule"
    filter {
      prefix = "actions-cache/Linux/"
    }
    expiration {
      days = 20
    }
    status = "Enabled"
  }
  rule {
    id = "build-cache-expiration-rule"
    filter {
      prefix = "actions-cache/Windows/"
    }
    expiration {
      days = 60
    }
    status = "Enabled"
  }
}
...
```

実行頻度の高いテストのキャッシュは20日間、ビルドのキャッシュは60日間保持し、期間を過ぎたものは完全に削除しています。

## 運用コスト

本システムは2023年末から運用を始め、必要に応じて改修しつつ4か月を迎えます。  
本システムで特に料金のかかる部分としては

- AWS
	- ビルドサーバーEC2インスタンス料金
	- ライセンスサーバーEC2インスタンス料金
- Unity floating license料金

となります。

ビルドサーバーについて、2024年1月の稼働時間は

- Linuxインスタンス：約23時間
- Windowsインスタンス：約23時間

でした。  
インスタンス利用料金について

- m6a.xlarge, Linux, spot: 約0.085 USD/h
- m6a.xlarge, Windows, on-demand：0.4072 USD/h

であるため、ビルドサーバーのインスタンス費用は合計だいたい11.3 USDでした。  
ライセンスサーバー（Linux, t2.micro: 0.0152 USD/h）は常時稼働しているため、月額11.3 USDとなり、EBS料金や通信料金など合わせたEC2のひと月の利用料金は30 USD前後でした。キャッシュS3については、保存量は高々300~400GB程度なので、AWS全体でかかるコストとしては月々30~40 USDとなりました。

Unity floationg license料金については一般に公表されていないため、詳細な金額をご紹介することは控えさせていただきます。

## おわりに

今回はホロライブアプリのCI環境についてご紹介しました。  
本件で構築したCI環境はTerraformとPackerによりほとんどの部分をIaC化したため、整備性が格段に向上し、また社内の他プロジェクトへの共有も容易になりました。このような環境整備は目に見える新機能などと違い、タレントやファンの皆様の目に触れる機会のないものですが、日々の開発をスムーズにし、今後のホロライブアプリの進化を実現する上では、無くてはならないものです。  
今後もホロライブアプリをより発展させ、タレントの皆様がより使いやすく、ファンの皆さまにより楽しいコンテンツをご提供できるツールになるよう開発してまいります。

  

スケーラブルなUnityのCI環境の実現｜カバー株式会社　公式note