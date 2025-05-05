---
title: 【Unity】GitHub Actions × Game CIでWebGLのCI/CD環境構築(デプロイ先はNetlify)
source: https://qiita.com/OKsaiyowa/items/ac86f1a220652309890e
author:
  - "[[Qiita]]"
published: 2023-12-05
created: 2025-05-06
description: はじめにUnityのWebGLビルドを経験された方はご存じだと思いますが、プロジェクトサイズによってはそれなりの時間がかかります。ビルド中は手元のPCに負荷がかかってしまったり、毎回のデプロ…
tags:
  - CI-CD
  - Tech
  - Tools
  - Unity
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から1年以上が経過しています。

## はじめに

UnityのWebGLビルドを経験された方はご存じだと思いますが、  
プロジェクトサイズによってはそれなりの時間がかかります。

ビルド中は手元のPCに負荷がかかってしまったり、  
毎回のデプロイ作業が面倒だったりと、効率化の余地がある部分です。

そこでUnityのWebGLビルドに焦点を当てて、CI/CDの構築を目指しました。  
また、基本利用は無料サービスのものを選定し、  
お金をかけずに最低限必要そうなものを揃えることができたのでメモします。

## GitHub Actions

GitHub Actionsを利用することでPush、Issue作成、PR作成など  
**GitHubプラットフォームのイベントをトリガーとしてワークフローを起動** 可能となります。

今回は深堀しませんが、テストの自動実行なども  
トリガーとなるイベントのタイミングで呼び出すことができます。

GitHub Actionsの `ワークフローを起動する機能` を使って、  
`ビルド→デプロイまでのフローを自動化する` というのが今回の試みです。

GitHub Actionsは無料アカウントでも利用することができます。  
実行上限が決まっており、勝手に課金されることはないので安心して利用可能です。

【参考リンク】： [About billing for GitHub Actions](https://docs.github.com/ja/billing/managing-billing-for-github-actions/about-billing-for-github-actions#about-spending-limits)

## GameCI

先ほど `ビルド→デプロイまでのフローを自動化する` と言いましたが、  
この部分はGame CIの力を借ります。

Game CIには `Unityビルド用のワークフロー` がまとめられています。  
この `Unityビルド用のワークフロー` をGitHub Actionsから呼び出すことで  
`ビルド→デプロイまでのフローを自動化する` ことが可能となります。

Game CIは **無料** で利用可能です。聖人君子のような思想で開発/運営されています。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/244071/5a70c1ab-80f3-68ee-cd36-400320791374.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F244071%2F5a70c1ab-80f3-68ee-cd36-400320791374.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=66159ab6c7c3cd37f2206bab109a8f45)

【引用元】： [Game CI](https://game.ci/)

## GitHub Actions × Game CI

では、実際にGitHub ActionsとGame CIを動かしてみます。

まずは最小構成として  
`PRがmasterにマージされたタイミングで自動ビルドするサンプル`  
を作ってみます。

導入までのフローに関しては下記記事が詳しいです。  
【参考リンク】： [GameCI で Unity の CI 環境を GitHub Actions で構築する](https://zenn.dev/nikaera/articles/unity-gameci-github-actions)

諸々の設定が完了したらymlファイルをAdd fileからCreate new fileを選択し  
`.github/workflows` 配下に追加します。  
[![タイトルなし.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/244071/65d43ad7-e0ac-1fa3-f6be-7391df9563d3.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F244071%2F65d43ad7-e0ac-1fa3-f6be-7391df9563d3.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=ca11dfd68bce39d194311c56a52aceef)

.github/workflows/webgl\_build.yml

```yml
name: Build

on:
  # masterへのPRマージをトリガーとしてワークフローを開始する
  pull_request:
    branches:
      - master
    types: [closed]

  # 手動実行デバッグ用
  workflow_dispatch: {}

jobs:
  build:
    name: Build my project
    runs-on: ubuntu-latest
    steps:
      # Checkout
      - name: Checkout repository
        uses: actions/checkout@v3

      # Build
      - name: Build project
        uses: game-ci/unity-builder@v2
        env:
          UNITY_LICENSE: ${{ secrets.UNITY_LICENSE }}
        with:
          targetPlatform: WebGL
          unityVersion: 2022.1.13f1 # ここは各自プロジェクトに合わせて設定
```

これだけで自動ビルドが完成です。  
リポジトリのActionsから成否を確認可能です。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/244071/cee27cb2-2181-f1f3-aa9d-7d7ac86d15e8.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F244071%2Fcee27cb2-2181-f1f3-aa9d-7d7ac86d15e8.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=bb20431188436569526086896ba62575)

---

### 独自のビルドパイプラインを走らせる

通常のUnityのビルドフローだと、 `Build Settings` からBuildを行います。  
先ほどの最小構成の例は `Build Settings` からBuildするフローと同じになります。

一方で、独自のビルドパイプラインを構築した場合は少し方法が異なります。  
以下のような最小構成のビルドパイプラインを実行する想定で考えます。

上記のDevBuildメソッドを呼び出す仕組みに変えたワークフローが以下です。  
`buildMethod: CustomBuild.DevBuild` を追加しただけです。

.github/workflows/webgl\_build.yml

```yml
# Build
  - name: Build project
    uses: game-ci/unity-builder@v2
    env:
      UNITY_LICENSE: ${{ secrets.UNITY_LICENSE }}
    with:
      targetPlatform: WebGL
      unityVersion: 2022.1.13f1 # ここは各自プロジェクトに合わせて設定
      # Dev Buildのメソッド呼び出し
      buildMethod: CustomBuild.DevBuild
```

ただ、これだと1つ問題があります。  
ビルドがこけたときに、ActionのステータスがErrorになりません。  
これでは、失敗していても見かけ上は成功したように見えてしまいます。

そこで、ビルドパイプライン側のコードを以下のように変更します。

```cs
using UnityEditor;
using System.IO;
using System.Linq;
using UnityEditor.Build.Reporting;

/// <summary>
/// 独自のビルドパイプラインを実行するEditor拡張
/// </summary>
public class CustomBuild : EditorWindow
{
    [MenuItem("MyTool/build/dev")]
    private static void DevBuild()
    {
        DoCustomBuildPipeLine();
    }
    
    /// <summary>
    /// CICDのワークフローから実行する時はこちらを呼び出す
    /// </summary>
    private static void DevBuildForCICD()
    {
        DoCustomBuildPipeLine(true);
    }

    /// <summary>
    /// 独自ビルドパイプライン実行
    /// </summary>
    private static void DoCustomBuildPipeLine(bool isCICD = false)
    {
        //保存先のパス取得
        var path = EditorUtility.SaveFolderPanel("Choose Location", "", "");

        //パスが入っていれば選択されたとみなす（キャンセルされたら入ってこない）
        if (string.IsNullOrEmpty(path))
        {
            return;
        }

        //ビルドパイプライン設定
        var buildPlayerOptions = new BuildPlayerOptions
        {
            scenes = EditorBuildSettings.scenes.Where(s => s.enabled).Select(s => s.path).ToArray(),
            locationPathName = Path.Combine(path, "Custom Build"),
            target = BuildTarget.WebGL,
            options = BuildOptions.Development
        };

        //Build
        var report = BuildPipeline.BuildPlayer(buildPlayerOptions);
        var summary = report.summary;

        if (isCICD)
        {
            //成否に応じてUnityEditorの終了プロセスを決定する
            EditorApplication.Exit(summary.result == BuildResult.Succeeded ? 0 : 1);
        }
    }
}
```

`EditorApplication.Exit` に `0` を渡すと正常終了、 `1` を渡すと異常終了とみなされます。

あとはymlファイルを `DevBuildForCICD` を呼び出す形で以下のように記述すれば、  
独自ビルドパイプラインを呼び出したうえでビルドエラーもしっかりと拾うことができます。

.github/workflows/webgl\_build.yml

```yml
name: Build

on:
  # masterへのPRマージをトリガーとしてワークフローを開始する
  pull_request:
    branches:
      - master
    types: [closed]

  # 手動実行デバッグ用
  workflow_dispatch: {}

jobs:
  build:
    name: Build my project
    runs-on: ubuntu-latest
    steps:
      # Checkout
      - name: Checkout repository
        uses: actions/checkout@v3

      # Build
      - name: Build project
        uses: game-ci/unity-builder@v2
        env:
          UNITY_LICENSE: ${{ secrets.UNITY_LICENSE }}
        with:
          targetPlatform: WebGL
          unityVersion: 2022.1.13f1 # ここは各自プロジェクトに合わせて設定
          # Dev Buildのメソッド呼び出し
          buildMethod: CustomBuild.DevBuildForCICD
```

---

### Netlifyへのデプロイ

ビルドの自動化は実現できたので、デプロイも自動化していきます。  
今回手軽に試せるデプロイ先としてNetlifyを選定しました。

GitHub Pagesも候補に挙がりましたが、PrivateリポジトリをGitHub Pagesで公開するためには  
Proアカウントを契約する必要があるので見送りました。

> GitHub Pagesは、GitHub Free及びOrganizationのGitHub Freeのパブリックリポジトリ、GitHub Pro、GitHub Team、GitHub Enterprise Cloud、GitHub Enterprise Serverのパブリック及びプライベートリポジトリで利用できます。

【引用元】： [GitHub Pages について](https://docs.github.com/ja/pages/getting-started-with-github-pages/about-github-pages)

まずはNetlifyにデプロイするためのサイトIDやアクセストークンを  
該当リポジトリのSecretsに設定しておく必要があります。

[![タイトルなし.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/244071/90af8cbe-bfc3-6e08-bdb3-e0229e97100e.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F244071%2F90af8cbe-bfc3-6e08-bdb3-e0229e97100e.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=e1619f3514590b8c4254efa11ae2801a)

ワークフローの使い方含め、以下リンクに詳細があります。  
【参考リンク】： [Netlify Deploy](https://github.com/marketplace/actions/netlify-deploy)

デプロイのフローを追加したymlファイルは以下です。

.github/workflows/webgl\_build.yml

```yml
name: Build and deploy

on:
  # masterへのPRマージをトリガーとしてワークフローを開始する
  pull_request:
    branches:
      - master
    types: [closed]

  # 手動実行デバッグ用
  workflow_dispatch: {}
  
env:
  target : Dev_Build

jobs:
  build:
    name: Build my project
    runs-on: ubuntu-latest
    steps:
      # Checkout
      - name: Checkout repository
        uses: actions/checkout@v3

      # Build
      - name: Build project
        uses: game-ci/unity-builder@v2
        env:
          UNITY_LICENSE: ${{ secrets.UNITY_LICENSE }}
        with:
          targetPlatform: WebGL
          unityVersion: 2022.1.13f1 # ここは各自プロジェクトに合わせて設定
          
      # Builder で出力した WebGL ビルドをアーティファクトでダウンロード可能にする
      - name: Upload the WebGL Build
        uses: actions/upload-artifact@v3
        with:
          name: ${{ env.target }}
          path: ${{ env.target }}
          
  # NetlifyへのDeploy処理
  deploy:
    name: Deploy
    needs: build
    runs-on: ubuntu-latest

    steps:
     # Builder で出力した WebGL ビルドをダウンロード
     - name: Download artifact
       uses: actions/download-artifact@v3
       with:
         name: ${{ env.target }}
         path: ${{ env.target }}

     # ダウンロードしたビルドファイルをNetlifyへDeploy
     - name: Deploy to Netlify
       uses: netlify/actions/cli@master
       env:
         NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
         NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID_FOR_DEV }}
       with:
         args: deploy --dir=${{ env.target }} --prod
```

特筆すべきこととして、アーティファクトの取り回しが挙げられます。  
アーティファクトとは **ワークフローの成果物** を指します。  
ログやテスト結果などもアーティファクトとして利用可能です。

【参考リンク】： [Storing workflow data as artifacts](https://docs.github.com/ja/actions/using-workflows/storing-workflow-data-as-artifacts)

今回は `ビルド結果のフォルダ一式` をアーティファクトとして利用します。  
デプロイまでのワークフローの流れは以下です。

**1.ビルドする  
2.ビルド結果のフォルダ一式を任意の場所にアップロードする  
3.ビルド結果のフォルダ一式をダウンロードする  
4.ダウンロードしたビルド結果のフォルダ一式をNetlifyにデプロイする**

一度アップロードしているのが少し違和感がありますが、  
ビルド結果のフォルダ一式はGame CIのビルドが終了するとCleanUpされてしまうので、  
このような手法でないと成功しないのだと思います。  
(これ確証はないので間違っていたらコメントください)

アーティファクトの保存期間はデフォルトでは90日間となっており、  
リポジトリの設定で変更が可能です。ymlファイルへの記述で個々に変更することも可能です。

保存期間5日の例

```yml
- name: Upload Artifact
    uses: actions/upload-artifact@v3
    with:
      name: my-artifact
      path: my_file.txt
      retention-days: 5
```

【参考リンク】： [Configuring the retention period for GitHub Actions artifacts and logs in your organization](https://docs.github.com/en/organizations/managing-organization-settings/configuring-the-retention-period-for-github-actions-artifacts-and-logs-in-your-organization)  
【参考リンク】： [Retention Period](https://github.com/actions/upload-artifact#retention-period)

---

### Slackへの通知

ここまでのワークフローでも十分に効率化ができましたが、  
欲を言えば結果の通知機能がほしいです。

そこで今回は `デプロイまでのワークフローの結果の成否` に応じて  
Slackへ通知を飛ばす処理を追加してみました。

以下のリポジトリのワークフローを利用します。  
設定についても記載があり、 [Incoming WebHooks](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks) の設定を行うだけなので非常に便利です。  
【参考リンク】： [Slack Notify - GitHub Action](https://github.com/rtCamp/action-slack-notify)

.github/workflows/webgl\_build.yml

```yml
name: CICD workflow

on:
  # masterへのPRマージをトリガーとしてワークフローを開始する
  pull_request:
    branches:
      - master
    types: [closed]

  # 手動実行デバッグ用
  workflow_dispatch: {}
  
env:
  target : Dev_Build

jobs:
  build:
    name: Build my project
    runs-on: ubuntu-latest
    steps:
      # Checkout
      - name: Checkout repository
        uses: actions/checkout@v3

      # Build
      - name: Build project
        uses: game-ci/unity-builder@v2
        env:
          UNITY_LICENSE: ${{ secrets.UNITY_LICENSE }}
        with:
          targetPlatform: WebGL
          unityVersion: 2022.1.13f1 # ここは各自プロジェクトに合わせて設定
          
      # Builder で出力した WebGL ビルドをアーティファクトでダウンロード可能にする
      - name: Upload the WebGL Build
        uses: actions/upload-artifact@v3
        with:
          name: ${{ env.target }}
          path: ${{ env.target }}
          
  # NetlifyへのDeploy処理
  deploy:
    name: Deploy
    needs: build
    runs-on: ubuntu-latest

    steps:
     # Builder で出力した WebGL ビルドをダウンロード
     - name: Download artifact
       uses: actions/download-artifact@v3
       with:
         name: ${{ env.target }}
         path: ${{ env.target }}

     # ダウンロードしたビルドファイルをNetlifyへDeploy
     - name: Deploy to Netlify
       uses: netlify/actions/cli@master
       env:
         NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
         NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID_FOR_DEV }}
       with:
         args: deploy --dir=${{ env.target }} --prod
         
  # Slackへの成功通知処理 
  NotifySucceed:
    if: ${{ success() }}
    name: Notify succeed
    # ほかのjobの結果を待つ
    needs: [build, deploy]
    runs-on: ubuntu-latest
    
    steps:
      - name: Nofity build and deploy succeed
        uses: rtCamp/action-slack-notify@v2
        env:
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK_URL }}
          SLACK_USERNAME: GitHub Actions
          SLACK_TITLE: Workflow succeeded
          SLACK_MESSAGE: 'https://hogehoge-kento.netlify.app/'
           
  # Slackへの失敗通知処理 
  NotifyFailure:
    if: ${{ failure() }}
    name: Notify failure
    # ほかのjobの結果を待つ
    needs: [build, deploy]
    runs-on: ubuntu-latest
    
    steps:
      - name: Nofity build or deploy failure
        uses: rtCamp/action-slack-notify@v2
        env:
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK_URL }}
          SLACK_USERNAME: GitHub Actions
          SLACK_TITLE: Workflow failed
          SLACK_COLOR: danger
          SLACK_MESSAGE: 'Run number : #${{ github.run_number }}'
```

`needs: [ジョブ名]` でほかのjobの処理の完了を待つことができます。  
今回はビルドとデプロイのjobの完了を待ち、  
その成否に応じてSlackに通知を送るワークフローをわけています。

メッセージにデプロイ先のリンクを張り付けておくと  
そのままSlackからデプロイされたビルドの確認が行えるので便利です。

---

### キャッシュ

ビルドの箇所でLibraryフォルダをキャッシュしていました。

しかし、キャッシュのkeyが同じであれば、キャッシュの箇所に変更があっても、  
そのkeyのキャッシュはすでに存在しているとみなされ、  
新たにキャッシュが作られることは無い仕様となっていました。

この仕様だと、Libraryフォルダの中身が大きく変わるレベルの変更があった場合、  
キャッシュが更新されず、余分なビルド時間がかかってしまいます。

そこで、今回は日付とkeyを紐づけることで、  
一定期間ごとにキャッシュが更新される仕組みを導入しました。

該当箇所だけを抜粋したものが以下です。  
月を跨いだタイミングでキャッシュが更新される例となっています。

月跨ぎのタイミングでキャッシュを更新する例

```yml
# 年月日を取得　キャッシュのkeyに利用
  - name: Set current datetime as env variable
    env:
      TZ: 'Asia/Tokyo' # タイムゾーン指定
    run: echo "CURRENT_DATETIME=$(date +'%Y-%m')" >> $GITHUB_ENV
    
  # キャッシュ keyを月跨ぎで更新することで定期的にキャッシュ自体の更新を行う
  - uses: actions/cache@v3
    with:
      path: Library
      key: "Library_${{ env.CURRENT_DATETIME }}"
```

プロジェクトのアップデート頻度に応じて、  
キャッシュの更新頻度を日/月/年と調整可能です。

この辺は他にも良い方法がありそうなので、  
プロジェクトの要望に応じて変更するのが望ましいかと思われます。

---

### デバッグ

ビルドパイプラインの修正などを行う際、  
任意のブランチに対してワークフローを走らせたいという要求がありました。

そこで、ブランチ名を指定してそのブランチにチェックアウトした状態で  
ワークフローを走らせるデバッグ機能を用意しました。(下記画像参照)  
[![タイトルなし.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/244071/125f6422-515a-65b5-1bc6-c7e81f439f26.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F244071%2F125f6422-515a-65b5-1bc6-c7e81f439f26.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=7c67ecefe4904c391dd2a29f3109f450)

該当箇所だけを抜粋したものが以下です。

```yml
on:
  # Debug用にActionsメニューからGUIでActionを走らせることを許容する
  workflow_dispatch: 
    # 入力欄の定義
    inputs: 
      ref:
        description: "ref"
        required: false
        default: ""
        
jobs:
  # ビルド処理
  build:
    name: Run the WebGL build
    runs-on: ubuntu-latest
    steps:
      # 作業ディレクトリにUnity プロジェクトの中身をダウンロードしてくる
      - name: Check out my unity project.
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - run: git checkout ${{ github.event.inputs.ref }}
        if: ${{ github.event.inputs.ref != '' }} # ref が指定されていたら git checkout
```

【参考リンク】： [GitHub Actionsで特定のコミットをcheckoutしてワークフローを実行する](https://zenn.dev/satococoa/articles/e026c0689e5678)

## ワークフロー全文

最後に、出来上がったワークフローを載せて終わりとします。

```yml
name: CICD workflow

on:
  # masterへのPRマージをトリガーとしてワークフローを開始する
  pull_request:
    branches:
      - master
    types: [closed]

  # Debug用にActionsメニューからGUIでActionを走らせることを許容する
  workflow_dispatch: 
    # 入力欄の定義
    inputs: 
      ref:
        description: "ref"
        required: false
        default: ""
  
env:
  target : Dev_Build

jobs:
  build:
    name: Build my project
    runs-on: ubuntu-latest
    steps:
      # 作業ディレクトリにUnity プロジェクトの中身をダウンロードしてくる
      - name: Check out my unity project.
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - run: git checkout ${{ github.event.inputs.ref }}
        if: ${{ github.event.inputs.ref != '' }} # ref が指定されていたら git checkout

      # Build
      - name: Build project
        uses: game-ci/unity-builder@v2
        env:
          UNITY_LICENSE: ${{ secrets.UNITY_LICENSE }}
        with:
          targetPlatform: WebGL
          unityVersion: 2022.1.13f1 # ここは各自プロジェクトに合わせて設定
          # 独自のビルドメソッドを呼び出したい場合はここで行う。
          # buildMethod: CustomBuild.DevBuild
          
      # Builder で出力した WebGL ビルドをアーティファクトでダウンロード可能にする
      - name: Upload the WebGL Build
        uses: actions/upload-artifact@v3
        with:
          name: ${{ env.target }}
          path: ${{ env.target }}
          
  # NetlifyへのDeploy処理
  deploy:
    name: Deploy
    needs: build
    runs-on: ubuntu-latest

    steps:
     # Builder で出力した WebGL ビルドをダウンロード
     - name: Download artifact
       uses: actions/download-artifact@v3
       with:
         name: ${{ env.target }}
         path: ${{ env.target }}

     # ダウンロードしたビルドファイルをNetlifyへDeploy
     - name: Deploy to Netlify
       uses: netlify/actions/cli@master
       env:
         NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
         NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID_FOR_DEV }}
       with:
         args: deploy --dir=${{ env.target }} --prod
         
  # Slackへの成功通知処理 
  NotifySucceed:
    if: ${{ success() }}
    name: Notify succeed
    # ほかのjobの結果を待つ
    needs: [build, deploy]
    runs-on: ubuntu-latest
    
    steps:
      - name: Nofity build and deploy succeed
        uses: rtCamp/action-slack-notify@v2
        env:
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK_URL }}
          SLACK_USERNAME: GitHub Actions
          SLACK_TITLE: Workflow succeeded
          SLACK_MESSAGE: 'https://hogehoge-kento.netlify.app/'
           
  # Slackへの失敗通知処理 
  NotifyFailure:
    if: ${{ failure() }}
    name: Notify failure
    # ほかのjobの結果を待つ
    needs: [build, deploy]
    runs-on: ubuntu-latest
    
    steps:
      - name: Nofity build or deploy failure
        uses: rtCamp/action-slack-notify@v2
        env:
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK_URL }}
          SLACK_USERNAME: GitHub Actions
          SLACK_TITLE: Workflow failed
          SLACK_COLOR: danger
          SLACK_MESSAGE: 'Run number : #${{ github.run_number }}'
```

## 参考リンク

- [GithubActionsで環境変数を理解する。](https://zenn.dev/hashito/articles/aef4de448f341b)
- [Effectively Manage GitHub Actions Artifacts to Deploy Releases](https://adamtheautomator.com/github-actions-artifacts-2/)
- [Slack が提供する GitHub Action "slack-send" を使って GitHub から Slack に通知する](https://qiita.com/seratch/items/28d09eacada09134c96c)
- [pushしたら自動でUnityビルドが走る環境を手に入れる](https://blog.kyubuns.dev/entry/2021/07/04/212005)
- [\[GitHubActions\] 自動でUnityPackageをアップロードしてリリースする](https://zenn.dev/kameffee/articles/c49499f82ce81c)
- [GitHub Actions ワークフローにおけるジョブ制御](https://developer.mamezou-tech.com/blogs/2022/02/20/job-control-in-github-actions/)
- [exit と　exit 1 の違い](https://teratail.com/questions/152186)
- [GitHub Actions覚え書き](https://qiita.com/progrhyme/items/56c24b3731deffcd4481)
- [GitHub Actionsで現在日時を取得する](https://qiita.com/itouuuuuuuuu/items/ebf0d4c306b54747374f)

[2](https://qiita.com/OKsaiyowa/items/#comments)

[18](https://qiita.com/OKsaiyowa/items/ac86f1a220652309890e/likers)

14