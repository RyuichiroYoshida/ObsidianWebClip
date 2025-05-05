---
title: GitHub ActionsでUnityのお手軽CI
source: https://qiita.com/kura_cvr/items/872cae5620a7be8a60ad
author:
  - "[[Qiita]]"
published: 2023-12-13
created: 2025-05-06
description: こんにちは。カバー株式会社エンジニアのKです。弊チームでは、UnityプロジェクトのビルドにGitHub Actionsを利用しています。GitHub　Actionsは、GitHubにアップロード…
tags:
  - CI-CD
  - 1
  - Unity
  - Tools
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から1年以上が経過しています。

こんにちは。カバー株式会社エンジニアのKです。

弊チームでは、UnityプロジェクトのビルドにGitHub Actionsを利用しています。GitHub　Actionsは、GitHubにアップロードしたプロジェクトを、設定に応じて自動でテスト・ビルド・デプロイなどのワークフローを実行してくれる機能です。

CIサービスは他にも色々ありますが、GitHub ActionsはGitHubで管理しているプロジェクトに少し設定ファイルを足すだけでCIを回せるので便利です。GitHubの公式サービスなので、プッシュやプルリクとの連動が簡単に行える点も魅力です。また、GitHubが用意したクラウド上のマシンや自前で用意したPCなど、ビルドマシンを柔軟に設定できるのも助かります。

GitHub Actionsについてはネット上に紹介している記事がたくさんあるので、この記事では弊チームの活用事例と設定を中心に紹介します。

## セルフホステッドランナー

GitHub Actionsには、自前で用意したPCでランナーを実行することで、そのPC内でワークフローを実行することができる機能があります。これをセルフホステッドランナーと言います。一方で、GitHubが自動でクラウド上のリソースを用意してビルドする機能もあります。こちらはGitHubホステッドランナーと言います。

弊チームでは、セルフホステッドランナーを使っています。できればGitHubホステッドランナーも使ってみたかったのですが、プロジェクトが大きすぎてGitHubホステッドランナーの標準プラン内に収まらないのと、Unity用に用意されているGameCIというプラグインが対応していないユースケースがあったため、結局自前のPCを使うことになりました。

用意したマシンをセルフホステッドランナーとして設定するには、以下の手順に従います。手順に従ってコマンドを打ち込むだけなので非常に簡単です。

## Unityプロジェクトにビルドメソッドを追加

プロジェクトに以下のようなビルドメソッドを用意しておきます。置き場所はEditorフォルダの中にします。

## ワークフローの例

ビルドしたいリポジトリの.github/workflowsに以下のymlを配置します。

build\_app.yml

```yaml
name: Build App windows

on:
  workflow_dispatch: { }
defaults:
  run:
    shell: bash

jobs:
  buildForWindowsBasedPlatforms:
    name: Build for ${{ matrix.targetPlatform }}
    runs-on: [ self-hosted, Windows ]
    strategy:
      fail-fast: false
      matrix:
        targetPlatform:
          - StandaloneWindows64 # Build a Windows 64-bit standalone.
        buildMethod:
          - BuildApp.Build
        unityPath:
          - '"C:\Program Files\Unity\Hub\Editor\2020.3.33f1\Editor\Unity.exe"' #使用しているUnityのバージョンのパス
    steps:
      - uses: actions/checkout@v4
        with:
          lfs: true
          clean: false
          submodules: recursive

      - name: Run Build
        run: |
          ${{ matrix.unityPath }} -batchmode -quit -projectPath . -buildTarget ${{matrix.targetPlatform}} -executeMethod ${{ matrix.buildMethod }} -logFile -

      - name: Upload App
        uses: actions/upload-artifact@v3
        with:
          name: App_Develop
          path: Builds/App/
          retention-days: 7
```

配置すると、GitHubのActionsのタブにワークフローが表示されるようになります。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3627517/aa213732-431c-8bb1-10d1-08edcb4153d4.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3627517%2Faa213732-431c-8bb1-10d1-08edcb4153d4.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=42aa41690ac876b9f62c5d9bf30ad3f0)

画面左側にある「Build App windows」をクリックすると以下のような画面が表示されます。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3627517/d8e587f3-c5c8-c442-741f-6ec048f19e91.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3627517%2Fd8e587f3-c5c8-c442-741f-6ec048f19e91.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a3a9014fd6545da95a43a176bcdeca2a)

右側にある「Run workflow」をクリックし、緑色の「Run workflow」ボタンをクリックすると、ビルドが開始されます。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3627517/b9e7fd4f-f0df-fb75-cbed-3e3ac34f1c11.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3627517%2Fb9e7fd4f-f0df-fb75-cbed-3e3ac34f1c11.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=81d14e3da9a43deabf1399ac0c309aad)

ビルド中は、GitHub Actionsの画面上で進捗を確認できます。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3627517/09195520-d330-1454-bf78-52359e22156d.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3627517%2F09195520-d330-1454-bf78-52359e22156d.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=18028f54e1d1de93cf0a90bf6e67eb80)

ビルドが完了すると、「Summary」画面にビルド成果物のダウンロードリンクが表示されます！

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3627517/5b4978db-081c-99e6-9bde-b4555a49c0ee.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3627517%2F5b4978db-081c-99e6-9bde-b4555a49c0ee.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=9d9fe6049f6370682cb0a6233b9f6060)

## ワークフローの実行タイミング

上記の例では、ワークフローの実行タイミングを以下のように設定しました。

```text
name: Build App windows

on:
  workflow_dispatch: { }
```

この「workflow\_dispatch」は、Actionsの管理画面から手動実行するオプションです。他に、プルリクエストの作成時、特定のブランチへのプッシュ時など、GitHubのアクションに合わせてワークフローを実行することもできます。詳しくはこちら↓

## まとめ

GitHub Actionsのセルフホステッドランナーを使えば、余分なPCにUnityとランナーをインストールするだけで、すぐに簡単なCIワークフローを構築することが可能です。お手軽なので、小規模なチームの初めてのCIとしておすすめです。

ありがとうございました。

[0](https://qiita.com/kura_cvr/items/#comments)

コメント一覧へ移動

[21](https://qiita.com/kura_cvr/items/872cae5620a7be8a60ad/likers)

いいねしたユーザー一覧へ移動

10