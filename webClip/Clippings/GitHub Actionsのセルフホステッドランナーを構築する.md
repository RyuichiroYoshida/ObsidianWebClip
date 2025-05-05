---
title: GitHub Actionsのセルフホステッドランナーを構築する
source: https://qiita.com/h_tyokinuhata/items/7a9297f75d0513572f4a
author:
  - "[[Qiita]]"
published: 2023-04-22
created: 2025-05-06
description: "使用技術GitHub ActionsGitHubが提供しているCI/CDGitHubにシームレスに統合されており、GitHubとの親和性が高いGitHub: https://github.c…"
tags:
  - CI-CD
  - 1
  - Tools
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から1年以上が経過しています。

## 使用技術

## GitHub Actions

- GitHubが提供しているCI/CD
- GitHubにシームレスに統合されており、GitHubとの親和性が高い
- GitHub: [https://github.com/actions/runner](https://github.com/actions/runner)
- ドキュメント: [GitHub Actionsのドキュメント](https://docs.github.com/ja/actions)
- GitHub Actionsには以下の２種類の提供方式が存在する

## GitHubホステッドランナー

- GitHubがSaaSとしてCI/CDの実行環境を提供する方式
- ジョブ毎にVMを作成することで、ジョブ内での変更が他のジョブへ影響を与えない仕組み
- Publicなリポジトリ or Privateなリポジトリで１ヶ月あたり2,000分以下の実行時間であれば無償
- 作成されるVMのスペックやOSの種類に制限がある
	- 参考: [サポートされているランナーとハードウェアリソース](https://docs.github.com/ja/actions/using-github-hosted-runners/about-github-hosted-runners#supported-runners-and-hardware-resources)
- ドキュメント: [GitHub ホステッド ランナーの概要](https://docs.github.com/ja/actions/using-github-hosted-runners/about-github-hosted-runners)

## セルフホステッドランナー

- 自身でサーバを用意してセットアップする方式
- 以下のような条件に当てはまる場合に選定されることが多い
	- GitHub Enterpriseを使用する場合
	- GitHubホステッドランナーでは社内のセキュリティ要件を満たせない場合
	- CI/CDの実行に高いスペックのサーバが必要
	- ニッチなOS上でCI/CDを実行したい場合
	- CI/CDの実行にかかる費用を抑えたい場合
- ドキュメント: [セルフホステッド ランナーの概要](https://docs.github.com/ja/actions/hosting-your-own-runners/about-self-hosted-runners)

## セルフホステッドランナーの構築

## 必要な情報の確認

- GitHub上に適当なリポジトリを作成する
- Settings > Actions > Runners に移動
- 「New self-hosted runner」を押下

[![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/78249/78c5e3c9-b613-dcb4-5865-d2b27ac4a5eb.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F78249%2F78c5e3c9-b613-dcb4-5865-d2b27ac4a5eb.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=fa2876c9813869c865ede4e4acde2ad4)

- 今回セルフホステッドランナーを構築するサーバのアーキテクチャはx64で、OSはUbuntu Server 22.04 LTS
- そのため、「Runner image」は `Linux` 、「Architecture」は `x64` を選択する

[![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/78249/732f4a55-a673-3db4-7dcb-94d47353fae6.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F78249%2F732f4a55-a673-3db4-7dcb-94d47353fae6.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=454c2560d7286e462f4e59bc4656469f)

- 「Download」からダウンロード先を控えておく

[![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/78249/46654d9f-5af6-82af-0933-1c187fc9ab1f.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F78249%2F46654d9f-5af6-82af-0933-1c187fc9ab1f.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=1a985ad9c0e999dfb799a97df0ffe8db)

- 「Configure」から `--url` と `--token` を控えておく

[![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/78249/730a1059-1e86-790b-4f5f-56d0375741b9.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F78249%2F730a1059-1e86-790b-4f5f-56d0375741b9.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=6429cf4d1d1d98b02b965fa1c9fc197d)

## ランナーのダウンロード

```bash
# rootユーザにスイッチ
$ sudo su -

# ダウンロード先となるディレクトリの作成
$ mkdir /opt/actions-runner && cd /opt/actions-runner

# 先ほど控えたダウンロード先からダウンロード
$ curl -o actions-runner-linux-x64-2.303.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.303.0/actions-runner-linux-x64-2.303.0.tar.gz

# ハッシュ値の検証
$ echo "e4a9fb7269c1a156eb5d5369232d0cd62e06bec2fd2b321600e85ac914a9cc73  actions-runner-linux-x64-2.303.0.tar.gz" | shasum -a 256 -c
actions-runner-linux-x64-2.303.0.tar.gz: OK

# tarballの解凍
$ tar xzf ./actions-runner-linux-x64-2.303.0.tar.gz
```

## ランナーの設定

```bash
# runnerユーザの作成
# ランナーの実行ユーザとして使用する
$ useradd runner -m

# パスワード無しでsudoコマンドを実行できるようにしておく
$ echo "runner ALL=(ALL:ALL) NOPASSWD: ALL" > /etc/sudoers.d/runner

# カレントディレクトリ以下のファイルの所有者とグループを再帰的にrunnerに変更
$ chown runner:runner . -R

# ランナーの設定
# --urlと--tokenには先ほど控えておいたものを指定する
# --unattendedはインタラクティブモードを無効化するオプション
# --replaceは既存のランナーを上書きして登録するオプション
# --nameはランナーの名称を指定するオプション
$ sudo -u runner ./config.sh --url https://github.com/tyokinuhata/XXXXXXXX --token XXXXXXXX --unattended --replace --name foo-runner

# ランナーのインストール
# runnerユーザとしてランナーが実行されるようにしておく
$ ./svc.sh install runner

# ランナーの実行
$ ./svc.sh start
```

## ランナーの確認

- サーバ上で確認する場合

```bash
# ランナーのステータス確認
$ ./svc.sh status
```

- GitHub上で確認する場合は Settings > Actions > Runners を開く

[![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/78249/35760672-8a25-4790-4531-e75acdc965fc.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F78249%2F35760672-8a25-4790-4531-e75acdc965fc.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=49b2d23a3b6a4034e1139781d6b63bb9)

- ランナーが正常に実行できており、ネットワークの疎通も取れている場合、「Status」は `Idle` になっている
- ランナーが正常に実行できていないか、ネットワークの疎通に問題がある場合、「Status」は `Offline` になっている
- ランナーがワークフローを実行中の場合、「Status」は `Active` になる

## ワークフローの実行

- 作成したリポジトリの`.github/workflows/` 以下に適当なワークフローを定義する
- 以下のサンプルは `git push` 時に `hello, world!`をコンソール上に出力する設定

.github/workflows/sample.yml

```yaml
name: Sample
on:
  - push
jobs:
  echo:
    runs-on: ubuntu-latest
    steps:
      - name: Hello world
        run: echo hello, world!
```

- その後、GitHub上に `git push` するとワークフローが実行される
- 実行過程やステータスは Actions > <ワークフロー名> > <該当コミット> > Jobs > <ジョブ名> から確認できる

[![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/78249/5bf16df8-251d-b99d-d6f3-f5c63eddb68a.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F78249%2F5bf16df8-251d-b99d-d6f3-f5c63eddb68a.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=948c0a17342465af5e052b80ff68cbd0)

## ログの確認

```bash
# サービス名の確認
$ cat /opt/actions-runner/.service
actions.runner.tyokinuhata-XXXXXXXX.foo-runner.service

# ログの確認
$ sudo journalctl -u actions.runner.tyokinuhata-XXXXXXXX.foo-runner.service
```

## ヘルプの確認

```bash
# config.shのその他のオプションは以下のコマンドで確認できる
$ sudo -u runner ./config.sh --help

# svc.shのその他のサブコマンドは以下のコマンドで確認できる
$ ./svc.sh --help
```

## config.shに変更を加えたい場合

```bash
# ランナーのアンインストール
$ ./svc.sh uninstall

# 設定の削除
# トークンの入力を求められるため、 Settings > Actions > Runners に移動し、「New self-hosted runner」を押下後、「Configure」からトークンを取得する
$ sudo -u runner ./config.sh remove

# ランナーの設定
# 今回は--nameをfoo-runnerからbar-runnerに変更した
$ sudo -u runner ./config.sh --url https://github.com/tyokinuhata/XXXXXXXX --token XXXXXXXX --unattended --replace --name bar-runner

# ランナーのインストール
$ ./svc.sh install runner

# ランナーの実行
$ ./svc.sh start

# サービス名が変更された
$ cat /opt/actions-runner/.service
actions.runner.tyokinuhata-XXXXXXXX.bar-runner.service
```

- GitHub上でも Settings > Actions > Runners を開くと、ランナーの名前が `bar-runner` に変更されたことが分かる

[![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/78249/786dd9c3-d181-7fc5-2d08-ce38a0e92c5f.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F78249%2F786dd9c3-d181-7fc5-2d08-ce38a0e92c5f.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=1111181dcfdeb9bfff88d3af9c0d9548)

[0](https://qiita.com/h_tyokinuhata/items/#comments)

[48](https://qiita.com/h_tyokinuhata/items/7a9297f75d0513572f4a/likers)

22