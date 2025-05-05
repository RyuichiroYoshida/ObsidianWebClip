---
title: 運用開始から4か月経ってから振り返るAWS&GitHub&Terraformを使ったUnity CI環境
source: https://synamon.hatenablog.com/entry/2022/09/22/153523
author:
  - "[[Activ8 Tech Blog]]"
published: 2022-09-22
created: 2025-05-06
description: エンジニアの岡村です。 このテックブログでは以前、AWS EC2でUnity環境を立ち上げ、GitHub ActionsのCI環境を構築する記事シリーズを投稿しました。 synamon.hatenablog.com synamon.hatenablog.com synamon.hatenablog.com synamon.hatenablog.com 今回、それの運用が始まってから4か月程度が経過したので、あれから変化があったことを簡単に今回の記事に纏めました。 今の状態 運用開始からこれまでにあった主な出来事 帯域の問題の解決、及びビルド時間の高速化 github/cache-actionの…
tags:
  - CI-CD
  - 1
  - Terraform
  - クラウド
  - Unity
  - IaC
read: false
---
エンジニアの岡村です。

このテックブログでは以前、AWS EC2でUnity環境を立ち上げ、GitHub ActionsのCI環境を構築する記事シリーズを投稿しました。

[synamon.hatenablog.com](https://synamon.hatenablog.com/entry/2022/03/29/190000)

[synamon.hatenablog.com](https://synamon.hatenablog.com/entry/2022/04/13/174034)

[synamon.hatenablog.com](https://synamon.hatenablog.com/entry/2022/04/20/190000)

[synamon.hatenablog.com](https://synamon.hatenablog.com/entry/2022/05/02/180000)

今回、それの運用が始まってから4か月程度が経過したので、あれから変化があったことを簡単に今回の記事に纏めました。

## 今の状態

現状の全体像はこんな感じです。EC2ビルドマシンがTerraformで構築したLambdaによって自動でスケールアウト、スケールインする以外にはあまり凝った事はしていません。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/S/Sokuhatiku/20220920/20220920122909.png)

今回はこの4か月で追加、強化した、以下の5か所にスポットを当てて紹介していきます。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/S/Sokuhatiku/20220922/20220922144241.png)

## 運用開始からこれまでにあった主な出来事

## 帯域の問題の解決、及びビルド時間の高速化

運用が始まってまず起こったのが、帯域の問題でした。

GitHubのLFS機能には容量および帯域に無料枠があり、無料枠の上限を超えた場合は課金が必要になります。

Unityのプロジェクトには画像のような要領の大きいバイナリアセットが多く含まれている為、CIでUnityプロジェクトを扱うと毎回プロジェクトのクローン時にダウンロードすることになり、帯域を大幅に圧迫してしまいます。

弊社のプロジェクトでもLFS含めて合計で3GB程度のリポジトリ（Unityで生成されるキャッシュを含めると10GB超）のため、あっという間に以下の画像のように、二週間程度で購入したデータパックを使い切るような状態になってしまいました。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/S/Sokuhatiku/20220914/20220914163742.png)

### github/cache-actionの導入

LFS帯域問題の解決としてgithub公式の提供しているのキャッシュ機能を使うようにしました。

[docs.github.com](https://docs.github.com/ja/actions/using-workflows/caching-dependencies-to-speed-up-workflows)

このアクションを使うとgithubが提供しているストレージにキャッシュを保存し、後のCI実行時に再利用することができます。これはself-hosted runnerでも利用可能で、しかもキャッシュ用のストレージはLFSと違って帯域で課金されません。

今回、特にダウンロード量の多い部分として以下の二か所をキャッシュ化しました。

- `Library/PackageCache/`
- `.git/lfs`

それぞれのキャッシュ化をした結果、帯域は大幅に改善し、GitHubの帯域がリセットされるまでの30日間動かしても50GBを切る様になりました。 ![](https://cdn-ak.f.st-hatena.com/images/fotolife/S/Sokuhatiku/20220920/20220920101922.png)

ちなみに、LFSのキャッシュは500MB、UPMパッケージのキャッシュは1GBあります。UPMパッケージ内でも大容量ファイルの管理にLFSが使われている為、サイズとしてはUPMパッケージのキャッシュの方が大きくなっています。

GitHub公式が提供しているキャッシュの容量は2022年9月14日現在において、最大10GBとなっています。この容量はLFSストレージとは異なり、課金して増やすことは出来ません。これはUnityプロジェクトとしては心許ない数値です。今の所は納まってはいますが、プロジェクトが進み大規模化していくとはみ出す恐れが十分にありそうです。

キャッシュサイズが大きいせいか、キャッシュダウンロードが途中で止まってしまうケースを何度か確認しています。そうなると6時間経っても終わらないので、 `continue-on-error` を有効にした上でタイムアウトを設定すると良いと思います。（ベストプラクティスとして、GitHubActionsの全てのステップにタイムアウトを設定すべきというのはあります。）

```yaml
steps:
      - uses: actions/checkout@v3
        with:
          lfs: false
      - name: Get LFS file catalog
        run: git lfs ls-files -l | cut -d' ' -f1 | sort | uniq > .lfs-files
        shell: bash
      - name: Cache LFS files
        uses: actions/cache@v3
        with:
          path: .git/lfs
          key: lfs-files-${{ hashFiles('.lfs-files') }}
        timeout-minutes: 10
        continue-on-error: true
      - name: LFS pull
        run: git lfs pull
```

公式のキャッシュは簡単な代わりに前述のような不安定さや制約があるので、今後折を見てs3にキャッシュするようなactionに移行するようにしたいと考えています。

### Unity Accelerator

また、もう一つのキャッシュの仕組みとして、Unity Acceleratorを導入しました。こちらはUnityが起動したときの、アセットインポート処理を高速化する仕組みです。こちらはLFSとは関係なく、帯域問題の解決には使えませんが、Unityの起動を高速化してくれる恩恵があります。

Unity Acceleratorはライセンスサーバーと同じVPC上に立て、VPC Peeringでランナーから繋がる様にしました。

34分10秒掛かっていたワークフローが24分50秒になり、10分弱の改善につながりました。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/S/Sokuhatiku/20220920/20220920112146.png) → ![](https://cdn-ak.f.st-hatena.com/images/fotolife/S/Sokuhatiku/20220920/20220920112228.png)

立てること自体は特に問題なくできましたが、エンドポイント設定がユーザー単位のレジストリに保存される仕組みになっており、EC2 Image Builderによるビルド時はビルド用のユーザーが使われるので、 `New-Item` でレジストリを追加できませんでした。その結果今のところはGitHub Actionsのワークフロー内のUnity実行時の引数で直接設定していますが、ビルド環境に強く依存している情報なのでなんとかワークフロー外に出せるようにしたいと考えています。

### マシンスペックの強化

インスタンスタイプを `t3-xlarge` から `c6i.2xlarge` に変更しました。カタログスペック上は単純に2倍の強化で、処理時間もほぼ半分くらいまで短くなりました。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/S/Sokuhatiku/20220920/20220920111750.png) → ![](https://cdn-ak.f.st-hatena.com/images/fotolife/S/Sokuhatiku/20220920/20220920111850.png)

この作業はterraformの設定を変更するだけなので非常に楽でした。

## Mac環境の用意

これは元々から視野に入っていたのですが、iOSやMacプラットフォーム向けへのビルドを行うためにMacのUnity実行環境が必要でした。

結論から言うと、Mac環境は社内にMac miniを設置して、手動でセットアップしてSelf-Hosted Runnerとして動かしています。最初は当然AWSのMacインスタンスを検討したのですが、試算したところ、ひと月動かすだけでM1 Mac miniを買えてしまう値段である事が分かったため、オンプレで使う事にしました。

ライセンスサーバーに関しては社内IPからのアクセスを許可し、AWS上で稼働するマシンと共有しています。ただし、Mac環境用のAcceleratorに関しては社内に立てたものを繋ぐようにしています。

当然、前述のようなUnityバージョン変更があると手作業でのインストールが必要になるのですが、Windowsで事足りる部分はWindowsを積極的に使うようにし、Macは必要な作業でのみ利用する事で物理マシン数を減らしてなんとかやっています。

## Unityのアップデート

CIパイプラインの構築直後はUnity2021.2.11f1を使っていたのですが、CIが動き出した直後に2021の安定板(2021.3.0f1)が出たため、CI環境もアップデート作業を行いました。この辺りはUnity Cloud Buildなら勝手に環境が用意されていてくれるのですが、Self-Hosted Runnerではそうもいきません。一応、インストール作業はEC2 Image Builderを使い半IaC化してあったので、手を動かす量自体はそれ程多くありませんでした。

- Image Builderのコンポーネントの新しいUnityバージョン用を用意する
- 新しいマシンイメージをビルド
- 完成したマシンイメージをTerraformから実行するように変更

完全に自動化されてはいないのでAMIのビルドとデプロイに関しては人の手が必要でしたが、ビルドはImageBuilderでバージョンを書き換えるだけ、デプロイはterraformのAMI名を書き換えてapplyするだけなので手間はかなり少ないと思います。

ただし、異なるバージョンのUnityを同時に使えるような作りには出来ていません。これに関しては複数ラインで同じCI環境を共有するような状況になったら対策が必要そうです。

## Windowsマシンの数の追加

開発が進み、ターゲットプラットフォームが増えるなどパターン数が増えてきたため、ライセンスを追加購入してEC2上のマシンの最大数を増やしました。

こちらはUnity Licensing Serverのガイド通りに、Unityのダッシュボード上に登録したライセンサーバーにシートを追加してから、新しいライセンスファイル（zip）をダウンロードし、それをライセンスサーバーに読み込ませます。その後、Terraformを弄って起動最大数を増やすと完了です。

この作業はフローティングライセンス&スケール可能な仕組みにしておいた事でかなり楽ができました。

## Firebase App Distributionへのデプロイ

ビルドしたアプリのテストをスムーズに行うため、成果物をFirebase App Distributionへデプロイする処理をActionsに追記しました。

ビルドを行うワークフロー呼び出し、成果物を一度artifactとしてアップロードし、後続のジョブ(runs-on: ubuntu-latest)でダウンロードしてからfirebase app distributionへアップロードしています。

ubuntu-latestには元からnpmが入っている為、簡単にfirebase-toolsのインストール、及びアップロードが出来ました。この辺りは巨人の肩に乗れている感じがして実に良いですね。

```yaml
firebase-deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Install Firebase
        run: npm install -g firebase-tools
      - name: Download Android artifact
        uses: actions/download-artifact@v3
        with:
          name: Artifact-${{ github.sha }}-${{ github.run_id }}-Android
      - name: Upload for Android
        env:
          FIREBASE_TOKEN: ${{ secrets.FIREBASE_TOKEN }}
          COMMIT_MESSAGE: ${{ github.event.head_commit.message }}
        run: >-
          firebase appdistribution:distribute Build.apk
          --app ****
          --groups ****
          --release-notes "$COMMIT_MESSAGE"
          --token "$FIREBASE_TOKEN"
```

ただし、社内でのテストのタイミングにデプロイが被って混乱が起こりかけたので、開発用と言えど、デプロイは手動、もしくは業務時間外に動くようなトリガーにした方が良さそうです。

## お金の話

開発規模によっても左右されるのと、ボカしてもいるので参考程度ですが、今の所CI全体で¥100,000強/月くらい掛かっています。

| 項目 | 費用(月額) | 詳細 |
| --- | --- | --- |
| AWS | $400 | ビルドマシン(EC2)、キャッシュサーバー(EC2)、成果物バックアップ(S3)など |
| GitHub | $170 | Spending Limit及びLFSデータパック |
| Unity | ¥34,000 | Unity Pro Build Server \* 5 |

CIにどの程度費用が掛かっているのかの情報はあまり世に出てこないので感覚は分からないのですが、現時点ではあまり凝った事をやっていないので、まだ安い方なのではないかと思います。

もしこの記事を読んでいる方にCI環境を弄っている方がいましたら、是非、ざっくりとで良いので費用感を教えてください……！

最近は円安のせいか色々値上がりして辛いです……上のUnityの値段などは2022年10月に行われる値上がり前の値を使っています。

## その他、今後やろうとしていること

- EC2インスタンスの起動の高速化
- ビルドの開始、終了、ダウンロードリンクが一覧できるようなslack通知
- AWS S3上にバックアップしている成果物にAWSコンソールを触らずアクセス出来る手段
- コーディング規約を強制統一する為の自動コードフォーマッティング
- 機能開発しているプラットフォームとは別のプラットフォーム上でのエラーを素早く通知する仕組み
- リリース用デプロイワークフローのコード化

やりたいことはまだまだたくさんあります。UnityのCIはまだ手探りの部分も多く、どうしても現状機能開発の方を優先しがちな為出来ていない事も多いのですが、アプリ本体やアセバンの各プラットフォーム自動ビルドなどは既に結構役に立ってくれています。開発体験の向上の為にCIは欠かせないものだと考えているので、まだまだ改善を続けていきたいと考えています。

[« 【Golang】Ginのログをカスタマイズしてリ…](https://synamon.hatenablog.com/entry/2022/09/28/180048) [UnityのAddressablesのRemoteコンテンツ運… »](https://synamon.hatenablog.com/entry/unity_addressables_catalog)