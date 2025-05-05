---
title: Web開発において複数人で同じ環境を用意しよう(Dockerの基本)
source: https://qiita.com/JavaLangRuntimeException/items/6f6d03bd6d1e69c9e67e
author:
  - "[[Qiita]]"
published: 2025-05-05
created: 2025-05-06
description: この記事ではDockerの基本を記述する．Dockerを用いるとWeb開発において非常に便利である．開発するときあるある私個人の主観ですが，経験談でこのようなことがあった．複数人で開発するとき…
tags:
  - 1
  - Tools
  - "#Docker"
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事ではDockerの基本を記述する．Dockerを用いるとWeb開発において非常に便利である．  
[![スクリーンショット 2024-06-10 16.15.07.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3757442/0402523d-6641-e82e-86fe-e3454ee0f7f3.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3757442%2F0402523d-6641-e82e-86fe-e3454ee0f7f3.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=1c7cbedc32cfc9b32a6b8ecdf2c9ab5e)

## 開発するときあるある

私個人の主観ですが，経験談でこのようなことがあった．

## 複数人で開発するとき...こんなことありませんか？

[![スクリーンショット 2024-06-10 15.55.08.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3757442/6ef4d839-e888-630c-03f0-fd139943db04.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3757442%2F6ef4d839-e888-630c-03f0-fd139943db04.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=c5db91170a8a05fecfab26196ff1e578)  
そもそも同じPCで開発すればいいが...そんなわけにもいかないだろう．ならば違うPCでも同じような環境で動かせばいいだろう．それでも同じPCでもさまざまなアプリを開発しているから，他のパッケージ，フレームワーク，ライブラリが邪魔しないだろうか？

## Web開発においてこんなことありませんか？

[![スクリーンショット 2024-06-10 16.10.36.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3757442/6d609b76-8ce1-181a-c4ef-ffb266f5b9ce.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3757442%2F6d609b76-8ce1-181a-c4ef-ffb266f5b9ce.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=7dda453fbfe0a92509823495a4f316ce)  
個人開発では作成するソフトウェアのパッケージ，フレームワーク，ライブラリのバージョンが今までインストールされていたバージョンと一致しないことがあるだろう．

[![スクリーンショット 2024-06-10 16.12.54.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3757442/8dde981c-f233-4f78-e71c-068aa3dabf84.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3757442%2F8dde981c-f233-4f78-e71c-068aa3dabf84.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=8423abeef266c9287e52814ee16a0b5a)  
共同開発ではチームメンバーでパッケージ，フレームワーク，ライブラリのバージョンを合わせたり，各々で個別にインストールすることが大変だろう．

## Dockerの概要

アプリケーションをコンテナという隔離された環境にパッケージ化するためのオープンソースのプラットフォーム．  
開発，デプロイ，実行が簡単にできるようになる．

## Dockerを使うメリット

**環境一貫性とポータビリティ**  
Docker コンテナはアプリケーションとその依存関係を一つのパッケージにカプセル化する．これにより，開発，テスト，本番環境間での「動作するはずが動かない」という問題を解決する．また，Docker コンテナはどのプラットフォーム（Linux, Windows, macOSなど）でも実行できる．

**開発の効率化**  
Dockerを使用することで，必要な環境をすぐに用意できる．また、再利用，配布をすれば，開発プロセスの効率化につながる．Dockerはマイクロサービスアーキテクチャの採用を助ける．各マイクロサービスを個別のコンテナとして分離・運用することが容易になる．

**インフラストラクチャの最適化**  
Docker コンテナは軽量で，起動が速く，少ないリソースで多くのコンテナを同時に実行できる．これにより，ハードウェアリソースをより効率的に使用できる．Docker コンテナは簡単にスケールアップおよびスケールダウンが可能．クラウド環境やオーケストレーションツール（Kubernetesなど）と組み合わせることで、自動スケーリングやロードバランシングが容易になる．

## Dockerを使う上での用語説明

[![スクリーンショット 2024-06-10 16.23.16.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3757442/c8427e99-d68c-3743-a52d-f7aa745259f0.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3757442%2Fc8427e99-d68c-3743-a52d-f7aa745259f0.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=79e040ba1e75f5ae2faf079d5945d3d8)

## Docker Container(コンテナ)

[![スクリーンショット 2024-06-10 16.24.20.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3757442/6ea8eb38-4144-8107-ff76-9f395e3e0d50.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3757442%2F6ea8eb38-4144-8107-ff76-9f395e3e0d50.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=e9a264d02c2bc781976e2045fbc70248)  
Docker Containerとは，アプリケーションとその実行に必要なすべての依存関係を含む軽量な，スタンドアローンの実行可能パッケージ．Docker コンテナは，Docker エンジンがインストールされた任意のシステム上で動作するように設計されており，これを **コンテナ仮想化技術** という

**コンテナ仮想化技術と仮想環境の違い**  
従来の仮想マシン（VM）よりも効率的にシステムリソースを利用する．VMがハードウェアレベルの仮想化を提供するのに対し，コンテナはオペレーティングシステムレベルの仮想化を提供する．これにより，コンテナは軽量で，起動が速く，少ないリソースで多くのコンテナを実行できる．

> 詳しくは以下の記事で説明している．  
> [https://qiita.com/tarakokko3233/items/aa732d049d16e50f5d07](https://qiita.com/tarakokko3233/items/aa732d049d16e50f5d07)

つまり，分かりやすく言うと以下の画像のようになる．  
[![スクリーンショット 2024-06-10 16.32.47.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3757442/14f561ce-fb58-8030-bbc0-cb6f30e2b5c5.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3757442%2F14f561ce-fb58-8030-bbc0-cb6f30e2b5c5.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=f9e3fefdf564cd86da3375e0f7dc5be4)

ある入れ物(コンテナ)の中に  
**OS**  
**依存関係**  
**ミドルウェア**  
**アプリケーション**  
が入っており，これでアプリケーション単位での依存関係(パッケージ，フレームワーク，ライブラリ)の管理が可能になる．

これを他の人でも共有して開発できれば簡単に共同開発できる

## Docker Image(イメージ)

[![スクリーンショット 2024-06-10 16.40.35.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3757442/177208f6-94dc-b20c-f15c-51128ed8ebf0.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3757442%2F177208f6-94dc-b20c-f15c-51128ed8ebf0.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a709fffaf6efc4423ddb8159aea77586)

Docker イメージは，Docker コンテナを実行するための静的なテンプレート．これは，コンテナを実行する際に必要なすべてのファイル，コード，ライブラリ，環境設定，および依存関係が含まれている．これを実行すると，コンテナが形成される．  
[![スクリーンショット 2024-06-10 16.45.30.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3757442/25b4a6c9-3f91-94f7-4d22-494a5ff06ab9.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3757442%2F25b4a6c9-3f91-94f7-4d22-494a5ff06ab9.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=9777e599dc7d410bbaaeef10aa557c1d)

Dockerイメージを生成するために `Dockerfile` を作成する．これはDocker イメージの設計図となる．

### Dockerfileの作成例

まず雛形を提示する

```dockerfile
# ベースイメージの指定
FROM python:3.8

# 環境変数の設定
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# 作業ディレクトリの設定
WORKDIR /code

# 依存関係ファイルのコピー
COPY requirements.txt /code/

# 依存関係のインストール
RUN pip install -r requirements.txt

# アプリケーションのソースコードをコピー
COPY . /code/

# アプリケーションの起動コマンド
CMD ["python", "app.py"]
```

**FROM**  
どのイメージを基にするかを定義する．実行するためのプログラミング言語，パッケージなどのバージョンと合わせて記述する．

**ENV**  
環境変数を設定する

**WORKDIR**  
コンテナ内の作業ディレクトリを設定．名前はなんでもいいがこの例では `/code` を使っている．このディレクトリにさまざまなファイルを追加していく．

**COPY**  
`requirements.txt` を先ほど作成した `/code/` ディレクトリに追加する．

> pythonでは依存関係のリストをrequirements.txtに格納する．  
> また，`. /code/` でカレントディレクトリにあるすべてのソースコードを `/code/` ディレクトリに追加する．

**RUN**  
必要なパッケージやソフトウェアをインストールする．この例ではrequirements.txtにリストされたPythonパッケージをインストールしている．

> pythonでは `pip nstall -r requirements.txt` で `requirements.txt` に書かれた依存関係のリストをまとめてインストールできる．

**EXPOSE**  
コンテナがリッスンするポートを指定できる．

**CMD**  
コンテナが起動時に実行するデフォルトのコマンドを設定する．

> pythonでは `python app.py` で実行する

このようにして作成したDockerfileをビルドすることでDocker イメージができる．

## Docker Volume(ボリューム)

[![スクリーンショット 2024-06-10 16.58.41.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3757442/e316436e-3c65-e2e0-9197-01ff53cb03d2.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3757442%2Fe316436e-3c65-e2e0-9197-01ff53cb03d2.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=aba1a65a1c9dadbead90c1814e29eef3)  
データを保存し永続化するための場所．コンテナは通常，削除されるとその内部で作成されたファイルも失われる．しかし，ボリュームを使用すると，これらのデータをコンテナのライフサイクルとは独立して保存できる．

## 複数のコンテナを組み合わせるには

複数のコンテナを組み合わせて使用することは非常に一般的であり，Dockerの主要な利用シナリオの一つである．特に，マイクロサービスアーキテクチャの実装や，異なるサービスが連携して動作するアプリケーションを開発する場合に有効である．例えばアプリケーションが二つ以上あるときや，データベースと組み合わせたい時に使われる．  
複数のコンテナやボリュームなどを組み合わせるならば `docker-compose.yml` を作成する．

dockerイメージが1つしか定義していない場合もぜひ `docker-compose.yml` を作成してほしい． `services` に定義するコンテナを1つにすればいい．

docker-compose.yml

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
    depends_on:
      - db
  db:
    image: postgres:12
    environment:
      POSTGRES_DB: exampledb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:

networks:
  app-network:
    driver: bridge
```

Dockerfileを明示的に指定しなくてもよい，通常は `build` で指定したディレクトリのルートにある `Dockerfile` を探すが， `Dockerfile` 以外のファイル名を使用する場合は以下のようにあるサービス内に指定する必要がある．

```docker
dockerfile: Dockerfile.custom
```

## docker-compose.ymlの中身の説明

**services**  
管理する全てのサービス（コンテナ）の定義を始めることを示す．

### appサービスについて

**app**  
これはコンテナ名．なんでもいい．

**build:.**  
build キーはDockerfileのあるディレクトリのパスを指定する．ここでは、カレントディレクトリ（.）が指定されている．

**ports: - "5000:5000"**  
ポートの指定．

**volumes: -.:/code**  
はホストのディレクトリとコンテナ内のディレクトリをマッピングするために使用する．ここでは，カレントディレクトリがコンテナの /code ディレクトリにマップされている．

**depends\_on: - db**  
`app` が `db` に依存していることを示す．これにより， `db` が先に起動する．

### dbサービスについて

**db**  
これはコンテナ名．なんでもいい．

**image**  
使用するDockerイメージを指定する．これはデータベース用にDockerfileを作成してその中に書いても良いが， `docker-compose.yml` の中に記述しても良い．

**environment**  
データベースの名前，ユーザー名，パスワードといった環境変数を設定．

**volumes: - db-data:/var/lib/postgresql/data**  
データベースのデータをホストの db-data ボリュームに保存する．

### ボリューム設定

**volumes: db-data:**  
データベースのデータを永続化するためのボリュームを定義．

### ネットワーク設定

**networks: app-network: driver: bridge**  
Composeファイルで定義されるネットワークの設定．ここでは app-network がブリッジドライバを使用して定義されている．これにより， `app` と `db` が同じネットワーク内で通信することが可能になる．

これによりコンテナを連携させられる

## Docker上で立ち上げたDBサーバーとアプリを通信する

簡単に以下の記事でまとめたので参考にしてほしい．

## Dockerでよく使うコマンド集

docker composeの実行方法などdockerでよく使うコマンド集をリストアップしたので参考にしてほしい．本記事で作成したdockerイメージを実行するためには以下の記事から．

[DockerDesktop](https://www.docker.com/get-started/) アプリで大体のことはできるが，コマンドの方が早く実行できる場合がある．

[0](https://qiita.com/JavaLangRuntimeException/items/#comments)

[34](https://qiita.com/JavaLangRuntimeException/items/6f6d03bd6d1e69c9e67e/likers)

37