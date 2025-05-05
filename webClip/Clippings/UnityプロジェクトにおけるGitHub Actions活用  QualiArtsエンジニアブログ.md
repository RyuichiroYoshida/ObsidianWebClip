---
title: UnityプロジェクトにおけるGitHub Actions活用 | QualiArtsエンジニアブログ
source: https://technote.qualiarts.jp/article/52/
author: 
published: 2023-03-31
created: 2025-05-06
description: Unityプロジェクトにおいて現状どのようにGitHub Actionsを活用しているかを紹介します
tags:
  - CI-CD
  - Tech
  - Tools
  - Unity
read: false
---
## はじめに

株式会社QualiArtsの田村です。 Unityエンジニアとしていくつかのゲーム開発に携わったのち、現在は開発推進室という主にゲーム開発のための基盤を開発する組織に所属しています。

さて、QualiArtsのUnityプロジェクトでは、CIツールとして長らくJenkinsを活用しています。 Jenkinsでどのようなことをしているかについては、本ブログの以前の記事「 [Unity開発の現場でJenkinsがしていることの紹介](https://technote.qualiarts.jp/article/48) 」で書かれていますのでぜひ参照してみてください。 その記事の中で、別のCIツールとしてGitHub Actionsについても検証しているということが書かれていますが、実際に現段階でいくつかの処理についてはGitHub Actionsでの運用が行われています。 そこで本記事では、実際のUnityプロジェクトにおいてGitHub Actionsをどのように活用しているかを紹介します。

## GitHub Actionsとは

GitHub Actionsとは、GitHub上でのワークフローの自動化を行うためのGitHub公式のサービスです。 ワークフローとは、開発プロジェクトにおける何らかの一連のタスクのことで、例えばコードの自動テスト、デプロイ、リリースなどが含まれます。

GitHub Actionsの特徴として以下のような点が挙げられます。

- ワークフローの処理内容はYAMLで記述され、GitHubのリポジトリ内で管理される
- プッシュやプルリクエストなど、GitHub上のイベントをトリガーとしたワークフローの実行が可能
- 公式・非公式含め、GitHub上で提供されている多数のアクションを自分のワークフローから利用可能
- ワークフローの実行環境として、GitHubが提供するクラウド環境の他に、自分で用意したビルド環境を選択可能

GitHub ActionsはGitHub公式のサービスなので、GitHubとの親和性が特に高いと感じます。 またGitHub Actionsは比較的新しいサービスで、開発も活発に行われているため、今後もより便利になっていくだろうという期待もしています。

## Jenkinsの問題点とGitHub Actionsの選択

JenkinsはCIツールとして広く利用され、実際QualiArts内でも長く利用されていますが、社内ではいくつか問題が上げられてきました。

大きな問題の1つは、ジョブ（GitHub Actionsでいうワークフロー）の設定が大変という点です。 Jenkinsの設定はGUIで行われるため、気軽に設定可能という利点があるものの、コードで管理するのが難しく、履歴を追うのが難しかったり再利用性に乏しかったりします。 また、設定の変更をレビューするようなフローも作りづらいため、その設定について作った人しか知らないという属人化が発生しがちです。

もう1つ大きな問題は、管理が大変という点です。 Jenkinsはセキュリティの観点から継続的にアップデートが必要ですが、多数のプラグインがインストールされたJenkins環境では、プラグインの依存バージョンの問題などからアップデートは簡単ではありません。

このような問題を解消できるかもしれない策として、GitHub Actionsの利用を進めています。 特にワークフローの設定に関しては、YAML形式でGitHub上で管理されるのに加えて、再利用も可能なため、Jenkinsにおける問題を解消できそうだと手応えを感じています。

## 実際にGitHub Actionsで動いているワークフロー

QualiArtsのUnityプロジェクトで実際に動いているワークフローには以下のようなものがあります。

- プルリクエストの自動テスト
- 定期的なコードのCleanup
- Unity Pacakgeの社内用のレジストリへのpublish（参考： [QualiArtsのUnity開発を支える基盤の紹介](https://creator.game.cyberagent.co.jp/?p=7615) ）
- アプリバージョンごとに作成されるバージョンブランチ間の自動マージ
- バージョンブランチのタグへの変換

特にプルリクエストの自動テストは、実装の品質を支える大事な仕組みになっています。 まだJenkinsから移行できていない主なジョブとしては、アプリビルドとアセットバンドルビルドがあります。 これらは処理も複雑で移行のハードルが高いですが、より良い在り方を引き続き模索していければと考えています。

## 実行環境

GitHub Actionsの実行環境を独自に用意するための仕組みはself-hosted runnerと呼ばれます。 QualiArtsでは、大きく分けて2種類のself-hosted runnerを利用しています。

1つ目は、CyberAgentグループ内で運用されている [myshoes](https://developers.cyberagent.co.jp/blog/archives/35729/) のrunnerです。 こちらは、UnityプロジェクトであってもUnityを起動する必要がないワークフロー(例えば、PullRequestにLabelをつけたり、タグへの変換等)で利用しています。

もう1つは、QualiArts独自で運用しているrunnerです。 従来からJenkinsを運用していた社内のマシン上に構築しており、Unityの実行が必要なワークフローで利用しています。 この構築にあたっては、 [runnerのdonwload urlを取得するapi](https://docs.github.com/ja/rest/actions/self-hosted-runners?apiVersion=2022-11-28#list-runner-applications-for-an-organization) や [runnerの登録用tokenを発行するapi](https://docs.github.com/ja/rest/actions/self-hosted-runners?apiVersion=2022-11-28#create-a-registration-token-for-an-organization) 等が公式から提供されているので、ある程度の自動化が可能です。

## Composite Actionによる処理の共通化

ワークフローを作っていると、複数のワークフローで同一の処理を行いたいことがよくあります。 そのようなものを共通化して再利用可能にすると、新しくワークフローを作るときに非常に楽になりますしメンテナンス性も向上します。

これを実現する方法として、Composite Actionという仕組みが用意されています。 Composite Actionを定義するYAMLファイルは以下のようなものになります。

```md
name: "Composite Action Test"

description: "Composite Actionのテストです"

inputs:

  message:

    description: “メッセージ"

    required: true

runs:

  using: "composite"

  steps:

      - run: echo メッセージはこちら「${{ inputs.message }}」

        shell: bash
```

`runs.using` が `"composite"` になっており、これがComposite Actionであることを示しています。 この例ではinputsで入力を受け付けているだけですが、outputsを使えば処理結果を出力してComposite Actionを利用した側で利用することもできます。 また、Composite Actionが利用可能になった当初はstepsの中にifやusesは使えなかったようですが、現在は使えるようになっています。 そのため、入力に応じて処理を変えることもできますし、Composite Actionの中からさらに別のComposite Actionを利用するということもできます。

### Composite Actionの利用

Composite Actionを利用するにはusesを使うのですが、その指定方法は、

- ローカルパスでの指定
- リポジトリでの指定

の2パターンとなります。

ローカルパスでの指定は、

```md
- uses: ./.github/actions/my-action
```

というように、Composite Actionのあるフォルダのパスを指定することになります。 これはワークフローを実行するリポジトリ内に必ずComposite Actionを含めないといけないということに思えますが、実はワークフロー実行時に動的に追加されたComposite Actionも実行することができます。 すなわち、Composite Actionは別のリポジトリで管理し、それをワークフロー内でcheckoutして利用するようなことができます。 これにより、複数リポジトリで共通のComposite Actionを利用できるようになります。

またリポジトリでの指定は、

```md
- uses: QualiArts/qua-actions@main
```

というように、 `Organization名/リポジトリ名@ブランチ名等` で指定します。 またあまり知られていないかもしれませんが、リポジトリでの指定でもパス（サブディレクトリ）を指定することが可能です。

```md
- uses: QualiArts/qua-actions/my-action@main
```

このリポジトリでの指定は、最近までpublicのリポジトリに対してしか利用できませんでした。 この制約は、企業で開発したComposite Actionなど安直に公開したくないシチュエーションでは致命的でした。 しかし、2022年12月に同じOrganizationであればprivateのリポジトリに対しても利用できるようになりました（ [公式のアナウンス](https://github.blog/changelog/2022-12-14-github-actions-sharing-actions-and-reusable-workflows-from-private-repositories-is-now-ga/) ）。 この機能追加により、リポジトリでの指定によるComposite Actionの利用が一気に現実的になりました。

現在QualiArtsでは、ローカルパスでの指定とリポジトリでの指定が混在していますが、今後は基本的にはリポジトリでの指定に移行していくことなると思います。

## Unity Libraryフォルダのキャッシュ

プルリクテストなどのワークフローでは、UnityプロジェクトをUnityで開く処理を行っています。

一般的に、実際に開発されているプロジェクトは巨大なため、プロジェクトを開くだけでもかなりの時間がかかります。 プロジェクトを一度開くと、Libraryという名前のフォルダが作成され、そこにUnityプロジェクトに関するキャッシュが保存されます。 以降、プロジェクトを開くときは、このLibraryフォルダのおかげで処理時間をかなり短縮させることができます。

一方、同じself-hosted runnerで実行されるワークフローは、ワークフローが異なっても同一リポジトリであれば同じワークスペースが共有されます。 そのため、特定のワークフローが前回の結果を利用するのが難しく、ワークスペースはクリーンな状態を想定する（そうでなければ自分できれいにする）のが原則になります。 結果として、GitHub ActionsではLibraryフォルダの恩恵を受けるのが難しい状態になります。 しかし、特にプルリクテストはできれば短時間で終わってほしいため、何とかしてLibraryフォルタを活用したいところです。

そこで、Libraryフォルダをビルドターゲットなど特定条件ごとにself-hosted runnerのマシン内の別の場所に保存し、シンボリックリンクを貼ることで使い回すような仕組みを構築しました。 この仕組みにより、GitHub ActionsでもLibraryフォルダの活用を可能にし、ワークフローの高速化を実現することに成功しました。

## おわりに

本記事では、QualiArtsのUnityプロジェクトにおいてGitHub Actionsをどのように活用しているかについて紹介しました。 JenkinsからGitHub Actionsへ移行することで、ワークフローの定義をGitで管理できるようになったり、 Composite Actionにより再利用性が向上したりという改善を達成できました。 引き続きGitHub Actionsを使ってより良いCI環境を構築していければと思います。