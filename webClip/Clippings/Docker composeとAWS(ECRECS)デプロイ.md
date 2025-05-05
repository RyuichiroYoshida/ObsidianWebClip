---
title: Docker composeとAWS(ECR/ECS)デプロイ
source: https://zenn.dev/soreiyu/articles/29fe1cacc071bd
author:
  - "[[Zenn]]"
published: 2020-12-15
created: 2025-05-06
description: 
tags:
  - Docker
  - 1
  - クラウド
  - バックエンド
  - フロントエンド
read: false
---
41[tech](https://zenn.dev/tech-or-idea)

## React・DockerとAWS

みなさんDockerは使っているだろうか。  
私は恥ずかしながら業務では使用していない。

だが、今回(2020/09)非常に興味深い `docker compose` とECS統合という物が実装された。

[Docker Compose for Amazon ECS Now Available](https://www.docker.com/blog/docker-compose-for-amazon-ecs-now-available/)

平たくいうと、dockerとAWSの連携コマンドなのだが  
※ `docker compose` は `docker-compose` コマンドのdocker cli版である。  
今後はdoker cliを使用してね。という事だそうだ。

この魔法のようなコマンドはdockerのコマンドを使うことで自動でAWSにデプロイしてしまう。  
という凄まじい物だった。

私としても備忘録にするため、筆を取った次第だ。

## 前提条件

実際にDockerを使用している皆様にはもう十分だろう。  
随時参考リンクがあるので見てくれると嬉しい。  
( ※もし、詳しく聞きたいというコメントしてくれれば対応するかもしれないね:) )

- AWSアカウントの作成とCLIのインストール
	- [AWS](https://aws.amazon.com/jp/cli/)
	- [AWS CLI のインストール](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-chap-install.html)
	- [AWS CLIでECRにログインする時はget-loginではなくget-login-passwordを使おう](https://qiita.com/hayao_k/items/3e4c822425b7b72e7fd0)
- Dockerインストール
	- [Docker](https://www.docker.com/)
	- Dockerfile、docker-compose.ymlの作成
	- Dockerイメージのビルド
- コンテナの起動とReactの実行
- docker Compose でデプロイ

## AWSアカウントの作成

AWSアカウントについては今更述べる必要は薄いとは思う。  
やりたい事はIAM Userの作成なのだが、以下に私が昔参考にしたブログを載せておく

- [\[AWS 1\]AWSアカウント取得後にする大切なこと！先ずIAMへ行け！](https://qiita.com/HOKARI_Yutaka/items/df34941c4db0fcde9e8d)

※上記の情報は古いが、ここで重要なのはどんなワードを元に検索するか？だ。

## Dockerインストール

Dockerには様々なチュートリアルが存在している。  
後で詳しく説明するがlinuxベースの仮想環境である。  
と言えば少しは荷が軽くなったような気がする。

以下のサイトが参考になるかと思う。  
[Docker入門：Docker概要,基本操作,マウント,Dockerfile,マルチステージビルド](https://qiita.com/shiro01/items/04ca672a93384b463701)

### Dockerfile、docker-compose.ymlの作成

ここは重要なフェーズの一つだ。  
`docker compose` コマンドの名が示す通り  
`docker-compose` コマンドの亜種だ(同じコマンドではないので注意だ。)  
なので複数コンテナが使う事を想定されている。

ここでcomposeをいまいち分かってない人は以下のブログを参考にすると良いだろう。  
[Docker Composeとは](https://knowledge.sakura.ad.jp/16862/)

さて、今回はapi用のDockerと、front用のDockerを用意する

Dockerfile.api

```dockerfile
# ベースイメージの作成 
FROM node:14.15.1-buster
# コンテナ内で作業するディレクトリを指定
WORKDIR /usr/src/app
# package.jsonとyarn.lockを/usr/src/appにコピー
COPY ["./api/package.json", "./api/yarn.lock", "./"]
RUN yarn install
# ファイルを全部作業用ディレクトリにコピー
COPY ./api .
# ポートのエクスポート
EXPOSE 9999

CMD yarn start
```

Dockerfile.front

```dockerfile
# ベースイメージの作成 
FROM node:14.15.1-buster
# コンテナ内で作業するディレクトリを指定
WORKDIR /usr/src/app
# package.jsonとyarn.lockを/usr/src/appにコピー
COPY ["./front/package.json", "./front/yarn.lock", "./"]
RUN yarn install
# ファイルを全部作業用ディレクトリにコピー
COPY ./front .
# ポートのエクスポート
EXPOSE 3000

CMD yarn start
```

以下の2つのDockerを纏める `docker-compose.yml` を用意する。

- Dockerfile.api
- Dockerfile.front

docker-compose.yml

```yml
version: "3"

services:
  frontend:
    image: soreiyu/front
    build:
      context: .
      dockerfile: Dockerfile.front
    command: "yarn start"
    ports:
      - "3000:3000"
    volumes:
      - ./front:/usr/src/app
    tty: true

  api:
    image: soreiyu/api
    build:
      context: .
      dockerfile: Dockerfile.api
    command: "yarn start"
    ports:
      - "9999:9999"
    volumes:
      - ./api:/usr/src/app
```

上記は以下を参考にしているため、以下のサイトを参考に、まずは普通にコンテナ起動をしてみよう。  
※あとでdocker contextというものを変更するが、ローカル環境はdefaltでないと動かないので気を付けよう

- [Docker でフロントエンドとAPIを開発してみた](https://www.to-r.net/media/docer-cra/)
- [DockerでReactの開発環境を作る](https://qiita.com/tanaka-tt/items/49628cd423e490120eeb)
- [Reactの開発環境をDockerで構築してみた](https://blog.web.nifty.com/engineer/2714)

DockerのNodeをどれにするか？

- [dockerのnode公式イメージ](https://hub.docker.com/_/node)

Dockerのその他

- [Docker利用時のソースコードの変更反映](https://qiita.com/tearoom6/items/326895f0a35774d70729)  
	\- [「hadolint」にシバかれながら美しいDockerfileを書き上げる](https://nekopunch.hatenablog.com/entry/2018/10/08/213513)
- [Docker — docker コンテナの中で vim が使えない場合](https://qiita.com/YumaInaura/items/3432cc3f8a8553e05a6e)

## コンテナの起動とReactの実行

ここまで出来ていれば以下のコマンドで一発だ。

`docker-compose up`

### 付録 Dockerでbabelのwatch機能が利かない

私がDockerでreactの開発をしているときに  
自動でトランスパイルされない自体に陥った。

以下`.env` ファイルを作成する事により  
watch機能を有効にしたことで解決した。

.env

```
CHOKIDAR_USEPOLLING=true
```

[babelのwatch機能がdockerコンテナ上で動かない](https://qiita.com/nishiurahiroki/items/52ff8cf144049b7834b0)

## docker Compose でデプロイ

さて、以下のようなファイル構成になったと思う。

```
dockerProject
  ├─api                     ← api用プロジェクト
  ├─front                   ← front用プロジェクト(react)
  │  └─.env                 ← react用のenvファイル
  ├─Dockerfile.api          ← api用のDockerfile
  ├─Dockerfile.front        ← front用のDockerfile
  └─docker-compose.yml      ← compose用のymlファイル
```

いよいよAWSにデプロイする。  
今回使用するAWSサービスは以下の二つだ

- [ECR(Amazon Elastic Container Registry)](https://aws.amazon.com/jp/ecr/)
	- AWS版のDockerHUBのような物だ。Dockerファイルのホスティングサービスだ
- [ECS(Elastic Container Service)](https://aws.amazon.com/jp/ecs/)
	- 完全マネージド型のコンテナオーケストレーションサービスだ。

大まかな流れとしてはECRにpushしたものをECSで公開するという流れだ  
(※別段ECRで管理する必要はないが、AWSの認証の関係でECR,ECSの連携の方が楽なため、この方式にしている)

以下のコマンドを使用し、まずはコマンドラインのawsのログインをしよう。

`aws ecr get-login-password | docker login --username AWS --password-stdin https://<aws_account_id>.dkr.ecr.<region>.amazonaws.com`

参考資料  
[AWS CLIでECRにログインする時はget-loginではなくget-login-passwordを使おう](https://qiita.com/hayao_k/items/3e4c822425b7b72e7fd0)

### docker-compose コマンドでECRにpush

さて、次にECRに今回作成したdocker imageをpushしたい。  
その為にまずはAWSでレポジトリを作成しよう

![](https://storage.googleapis.com/zenn-user-upload/a3sntcaqtjxv77aebrmap1vdvxnm)

ここでプライベートレポジトリにしないと、コマンドで使用できないため、プライベートにしよう。

![](https://storage.googleapis.com/zenn-user-upload/a665ml585hoahs12s91jyq1o5wsh)

次にymlファイルのpush先を変更しなければならない。  
これは、AWSのプッシュコマンドを表示というところをクリックすれば細かい詳細がみれるので  
そこから使用すればよいだろう。

docker-compose.yml

```yml
version: "3"

services:
  frontend:
    # image: soreiyu/front
    image: 000000000000.dkr.ecr.ap-northeast-1.amazonaws.com/soreiyu/front
    build:
      context: .
      dockerfile: Dockerfile.front
    command: "yarn start"
    ports:
      - "3000:3000"
    volumes:
      - ./front:/usr/src/app
    tty: true

  api:
    # image: soreiyu/api
    image: 000000000000.dkr.ecr.ap-northeast-1.amazonaws.com/soreiyu/api
    build:
      context: .
      dockerfile: Dockerfile.api
    command: "yarn start"
    ports:
      - "9999:9999"
    volumes:
      - ./api:/usr/src/app
```

dockerをbulidしtag名を付けて以下のコマンドで実際にpush出来るかためそう。  
tag名に関しては、細かい詳細で確認しよう

`docker-compose push`

※私がここで良く失敗するのは  
docker context を `default` 以外にしてしまっている時だ。

さて実際にECRレポジトリにpushされたか確認しよう

`aws ecr describe-repositories`

これで以下のような表示がされれば無事push出来ている  
(※パブリックだとこのコマンドで表示されない。)

```
{
    "repositories": [
        {
            "registryId": "012345678910",
            "repositoryName": "ubuntu",
            "repositoryArn": "arn:aws:ecr:us-west-2:012345678910:repository/ubuntu"
        },
        {
            "registryId": "012345678910",
            "repositoryName": "test",
            "repositoryArn": "arn:aws:ecr:us-west-2:012345678910:repository/test"
        }
    ]
}
```

## 魔法のdocker compose

さて、ここまできてようやく下地が整った。  
今回追加された `docker compose` コマンドを使用しよう

以下の通りにAWS コンテキストの生成を行う。  
[ECS での Docker コンテナーのデプロイ](https://matsuand.github.io/docs.docker.jp.onthefly/engine/context/ecs-integration/)

次にAWSコンテキストを以下コマンドで使用しよう。

`docker context use myecscontext`

今なんのcontextを使用しているかは以下コマンドで確認できる。

`docker context ls`

そして次に重要なのはcompose ymlファイルの編集だ。  
compose コマンドは対応していないルールがあるので、そこだけコメントアウトする必要がある。  
(※消しても良いが、コメントを外したりする機会があるかと思うので私はこうしている。)

docker-compose.yml

```yml
version: "3"

services:
  frontend:
    # image: soreiyu/front
    image: 000000000000.dkr.ecr.ap-northeast-1.amazonaws.com/soreiyu/front
    # build:
    #   context: .
    #   dockerfile: Dockerfile.front
    command: "yarn start"
    ports:
      - "3000:3000"
    # volumes:
    #   - ./front:/usr/src/app
    # tty: true

  api:
    # image: soreiyu/api
    image: 000000000000.dkr.ecr.ap-northeast-1.amazonaws.com/soreiyu/api:latest
    # build:
    #   context: .
    #   dockerfile: Dockerfile.api
    command: "yarn start"
    ports:
      - "9999:9999"
    # volumes:
    #   - ./api:/usr/src/app
```

さて、以下コマンドを祈りながら発行しよう。

`docker compose up`

うまく出来ていればAWSで公開されている。

では最後に一括完全消去して終わりにしよう。

`docker compose down`

ここまで読んでくれてありがとう。  
良いDockerライフを！;)

### エラーになったときの対処

クラスターが作成されない → route53のサービスが中途半端に削除されているなど

41