---
title: Golang Web API開発入門ガイド
source: https://qiita.com/y_murotani/items/1c1a93fba22035b45d6a
author:
  - "[[Qiita]]"
published: 2024-10-17
created: 2025-05-06
description: はじめにこの記事では、Go言語のGin Web FrameworkとBun ORMを用いてWeb API作成する際に、その一助となる記事になることを目指しています。Go言語のはじめの一歩としては…
tags:
  - Tech
  - Go
  - バックエンド
  - 通信
  - 学習系
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

## はじめに

この記事では、Go言語の [Gin Web Framework](https://gin-gonic.com/ja/) と [Bun ORM](https://bun.uptrace.dev/) を用いてWeb API作成する際に、その一助となる記事になることを目指しています。  
Go言語のはじめの一歩としては [A Tour of Go](https://go-tour-jp.appspot.com/list) や [Progate](https://prog-8.com/courses/go) などがあります。こちらを一度行った上でこの記事を読むとより良いかと思います。  
また、一部分のみの挙動を確認したい場合は、 [paiza.io](https://paiza.io/) や [The Go Playground](https://go.dev/play/) を活用すると気軽に確認が可能です。

## 動作環境

この記事の動作環境は以下のとおりとなっています。

```環境情報
OS: Mac OS Ventura 13.3.1(a)
homebrew: 4.0.18
Go: 1.20.4
    Gin: 1.9.0
    Bun: 1.1.12
Docker 23.0.5
Docker compose: 2.17.3
golangci-lint: 1.52.2
gci: 0.10.0
```

## 目次

- [環境構築](https://qiita.com/y_murotani/items/##%E7%92%B0%E5%A2%83%E6%A7%8B%E7%AF%89)
	- [Go言語のインストールとセットアップ](https://qiita.com/y_murotani/items/###Go%E8%A8%80%E8%AA%9E%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%A8%E3%82%BB%E3%83%83%E3%83%88%E3%82%A2%E3%83%83%E3%83%97)
	- [Hello World](https://qiita.com/y_murotani/items/###HelloWorld)
- [Dockerを使った開発環境](https://qiita.com/y_murotani/items/##Docker%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%9F%E9%96%8B%E7%99%BA%E7%92%B0%E5%A2%83)
	- [Dockerfileとdocker-compose.yaml](https://qiita.com/y_murotani/items/###Dockerfile%E3%81%A8docker-compose.yaml)
- [Gin Web Framework](https://qiita.com/y_murotani/items/##GinWebFramework)
	- [ヘルスチェックエンドポイントの実装](https://qiita.com/y_murotani/items/###%E3%83%98%E3%83%AB%E3%82%B9%E3%83%81%E3%82%A7%E3%83%83%E3%82%AF%E3%82%A8%E3%83%B3%E3%83%89%E3%83%9D%E3%82%A4%E3%83%B3%E3%83%88%E3%81%AE%E5%AE%9F%E8%A3%85)
- [Bun Lightweight Golang ORM](https://qiita.com/y_murotani/items/###BunLightweightGolangORM)
	- [Bun ORMを使用してデータベースのマイグレーション](https://qiita.com/y_murotani/items/###BunORM%E3%82%92%E4%BD%BF%E7%94%A8%E3%81%97%E3%81%A6%E3%83%87%E3%83%BC%E3%82%BF%E3%83%99%E3%83%BC%E3%82%B9%E3%81%AE%E3%83%9E%E3%82%A4%E3%82%B0%E3%83%AC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3)
- [モデルの作成方法、データベースへのアクセス](https://qiita.com/y_murotani/items/##%E3%83%A2%E3%83%87%E3%83%AB%E3%81%AE%E4%BD%9C%E6%88%90%E6%96%B9%E6%B3%95%E3%80%81%E3%83%87%E3%83%BC%E3%82%BF%E3%83%99%E3%83%BC%E3%82%B9%E3%81%B8%E3%81%AE%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9)
- [golangci-lintを導入](https://qiita.com/y_murotani/items/##golangci-lint%E3%82%92%E5%B0%8E%E5%85%A5)
- [おわりに](https://qiita.com/y_murotani/items/##%E3%81%8A%E3%82%8F%E3%82%8A%E3%81%AB)

## 環境構築

### Go言語のインストールとセットアップ

Go言語は [公式サイト](https://go.dev/) よりダウンロードが可能です。そちらよりダウンロードしていただき、インストールを行うと良いでしょう。また、homebrewを使われている方はそちらでもインストールが可能です。その場合はこちらのコマンドになります。

```sh
$ brew install go
```

homebrewを用いてインストールを行なった場合、PATHが通っていないため、自身でPATHの設定が必要となります。筆者の環境の場合、shellはzshを用いているため、.zshrcに以下の行を追加しています。他のshellを使っている場合は同様の内容を所定のファイルに記述してください。

.zshrc

```text
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```

以上でGo言語のインストールは完了となります。Go言語がインストールできているか確認してみてください。

```sh
$ go version
> go version go1.20.4 darwin/arm64

$ ECHO $GOPATH
> /Users/hoge_user/go
```

### Hello World

まず、Hello Worldを行うためのディレクトリを作成し、モジュールの初期化を行います。

```sh
$ mkdir hoge
$ cd hoge
$ go mod init hoge
> go: creating new go.mod: module hoge
$ ls
> go.mod
```

続いて、 `hello.go` を作成します。

```sh
$ touch hello.go
$ ls
> go.mod hello.go
```

hello.go

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello World")
}
```

実行する際には、 `go run` コマンドを使用します。

```sh
go run hello.go
> Hello World
```

また、Go言語はビルドすることでバイナリファイルを生成することができます。その際、オプションにて指定しない場合は初期化したモジュールの名前になります。

```sh
$ go build
$ ls
> go.mod hello.go hoge*

$ ./hoge
> Hello World
```

以上がHello Worldまでの手順となります。

## Dockerを使った開発環境

### Dockerfileとcompose.yaml

Dockerを使っての開発が主流になっているかと思います。Go言語の開発環境をDockerで構築することで、比較的容易に開発環境を共有することが可能となります。また、今回はRDBMSにMySQLを採用し開発を行なっていきます。docker composeを用いることでGoが動作するコンテナとMySQLが動作するコンテナの２つを管理していきます。

今回のDocker　imageには容量は大きいですがDebianベースのイメージを用います。alpineなどでも問題なく開発は可能なので、用途に合わせて選択してください。今回のDockerfileの例は次のようになります。

```dockerfile
FROM golang:1.20.4 AS build

CMD ["/bin/bash", "-b"]

ENV GOPATH=/go

RUN apt-get update && apt-get upgrade -y\
    && apt-get autoremove -y\
    && apt-get clean\
    && rm -rf /var/lib/apt/lists/*

WORKDIR /go/src

COPY ./src/go.mod /go/src/go.mod
COPY ./src/go.sum /go/src/go.sum
RUN go mod download
```

今回の環境では `go mod init api` にて初期化を行なっています。また、`.env` ファイルを用いて環境変数の設定を行なっています。`.env` ファイルは `compose.yaml` と同一のディレクトリに格納しておくことをお勧めします。

.env

```shell
DB_NAME=hoge
DB_USER=user
DB_USER_PASSWORD=password
DB_HOST=127.0.0.1
DB_ADDRESS=db
DB_COLLATION=utf8mb4_general_ci
DB_TZ=Asia/Tokyo
MYSQL_ROOT_PASSWORD=rootpassword
```

compose.yaml

```yaml
version: "3"
services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - ./src:/go/src
    ports:
      - "${APP_PORT:-8080}:8080"
    depends_on:
      - db
    env_file:
      - ./.env
    command: go run main.go

  db:
    image: mysql:8.0
    volumes:
      - data-volume:/var/lib/mysql
      - ./db:/docker-entrypoint-initdb.d:ro
    ports:
      - "${DB_PORT:-3306}:3306"
    environment:
      MYSQL_ROOT_PASSWORD: "${MYSQL_ROOT_PASSWORD}"
      TZ: "${DB_TZ}"
      DB_USER_NAME: "${DB_USER}"
      DB_PW: "${DB_USER_PASSWORD}"
      DB_HOST: "${DB_HOST}"
      DB_NAME: "${DB_NAME}"
```

./dbディレクトリに `init.sql` ファイルを配置しています。このファイルをdocker-emptrypoint-initdb.dにバインドすることでMySQLコンテナをビルドした際に実行されます。ユーザーを作成することでMySQLにアクセスする際、root以外のユーザーでアクセスすることができます。init.sqlの内容は次のようになります。

init.sql

```plsql
CREATE USER 'user'@'%' IDENTIFIED BY 'password';
GRANT ALTER, CREATE, DELETE, DROP, INSERT, REFERENCES, SELECT, UPDATE ON hoge.*  TO 'user'@'%';
FLUSH PRIVILEGES;

CREATE DATABASE IF NOT EXISTS hoge;
```

`docker compose up` コマンドを用いてコンテナを立ち上げます。正しくコンテナが立ち上がった場合、MySQLのコンテナは動作し続けますが、Goのコンテナは終了します。Goのコンテナも動作し続けるために `tty: true` の設定を入れる場合もありますが、後述するGinFWを開発する上では必要ないため今回は設定せずに進みます。

また、この記事ではsrcディレクトリ以下に実装を行なっていきます。最終的なディレクトリ構成は次のようになります。

```text
src
  ├── app
  │   ├── http
  │   │   └── controllers
  │   ├── models
  │   ├── repositories
  │   └── router.go
  ├── config
  │   └── env.go
  ├── go.mod
  ├── go.sum
  └── main.go
```

## Gin Web Framework

Ginのドキュメントは日本語に対応しているため、 [公式ドキュメント](https://gin-gonic.com/ja/) を読みながら開発をしていくと良いかと思います。次の引用文も公式ドキュメントより引用しています。

> Ginとは何か？
> 
> Gin は、Golang で書かれた Web フレームワークです。
> 
> martini に似た API を持ちながら、非常に優れたパフォーマンスを発揮し、最大で40倍高速であることが特徴です。
> 
> 性能と優れた生産性が必要なら、きっと Gin が好きになれるでしょう。

### ヘルスチェックエンドポイントの実装

[クイックスタート](https://gin-gonic.com/ja/docs/quickstart/) のページに書かれている内容を踏襲すれば容易に実装することができます。

> まず、example.go を作成します。  
> 後述のコードが、example.go のファイルにあるとします。
> 
> ```sh
> $ touch example.go
> ```
> 
> 次に、下記のコードを example.go に書きます。
> 
> example.go
> 
> ```go
> package main
> 
> import "github.com/gin-gonic/gin"
> 
> func main() {
>   r := gin.Default()
>   r.GET("/ping", func(c *gin.Context) {
>       c.JSON(200, gin.H{
>           "message": "pong",
>       })
>   })
>   r.Run() // 0.0.0.0:8080 でサーバーを立てます。
> }
> ```
> 
> そして go run example.go でコードを実行します。
> 
> example.go を実行し、ブラウザで 0.0.0.0:8080/ping にアクセスする
> 
> ```sh
> $ go run example.go
> ```

作成したエンドポイントをcurlを使って叩いてみます。今回はGETメソッドで実装しているため、オプションは必要ありません。

```sh
$ curl http://localhost:8080/ping
> {"message":"pong"}%
```

この内容を踏まえてmain.goを実装していきます。main.goの肥大化を避けるため、ルートの設定をrouter.goで行うようにします。router.goはsrc/app/に配置します。その場合、次のようになります。

main.go

```go
package main

import (
    "log"

    "api/app"
)

func main() {
    router := app.SetRouter()
    if err := router.Run(":8080"); err != nil {
        log.Fatal(err)
    }
}
```

router.go

```go
package app

import "github.com/gin-gonic/gin"

func SetRouter() *gin.Engine {
    r := gin.Default()
    r.GET("/health", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "message": "Health check OK",
        })
    })
    return r
}
```

.Run()メソッドは文字列を引数に取り、明示的にポートを指定することが可能です。上の例ではドキュメント同様に8080を指定していますが、他のポートにしたい場合は `router.Run(":8888")` のように記述することでポート番号の指定が可能となっています。

ルートの設定を行うファイルを切り分けることで、ルートのグループ化において大きな有用性があると考えています。例えば、 `~/hoge/fuga` 、 `~/hoge/piyo/read` 、 `~/hoge/piyo/write` のように、共通なルートがある場合を考えます。もっとも単純な場合、次のように書くことができます。

router.go

```go
package app

import "github.com/gin-gonic/gin"

func SetRouter() *gin.Engine {
    r := gin.Default()
    r.GET("/hoge/fuga")
    r.GET("/hoge/piyo/read")
    r.GET("/hoge/piyo/write")
    return r
}
```

エンドポイントの数が多くない場合はこちらでも十分だと思います。しかし多くの場合は、コメントなどで書く順番などのルールを書くことになるかと思います。そこで、Ginではルートのグループ化を行うことでエンドポイントのネストを行うことが可能です。グループ化を行った場合は次のように書くことができます。

router.go

```go
package app

import "github.com/gin-gonic/gin"

func SetRouter() *gin.Engine {
    r := gin.Default()
    hoge := r.Group("/hoge")
    {
        hoge.GET("/fuga")
        hoge.GET("/piyo/*action")
    }
    return r
}
```

このように書くことで、エンドポイントのネストがわかりやすくなると思います。また、他にルートを設定していない場合、 `~/hoge/piyo` にリダイレクトしてくれるため、別途 `~/hoge/piyo` を記述する必要がない点もポイントです。

## Bun Lightweight Golang ORM

Bun Lightweight Golang ORMはSQLite、PostgreSQL、MySQL、MSSQLで動作するORMです。マルチテナントやカスタムタイプに対応しているため、uuidなどを定義することも十分に可能になっています。

この記事で用いるデータベースは次のようになっています。

[![ER](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/522394/4e91dac0-3604-3ed7-e064-732711899e08.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F522394%2F4e91dac0-3604-3ed7-e064-732711899e08.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=4cef4b355f3e4c8599bca95508483e5b)

### Bun ORMを使用してデータベースのマイグレーション

Bunを用いてマイグレーションを行う場合、２つのSQLファイルを用意する必要があります。１つはテーブルを作成するSQLファイル、もう一つはテーブルを削除するSQLファイルです。前者は `yyyyMMddHHmmss_${hoge}.up.sql` 、後者は `yyyyMMddHHmmss_${hoge}.down.sql` となります。SQLファイルはmigrationを行うGoファイル以下のディレクトリに置くことが望ましいです。  
この記事の場合、以下のようになります。

```sql
CREATE TABLE \`stores\` (
  \`id\` integer PRIMARY KEY AUTO_INCREMENT,
  \`name\` varchar(100) NOT NULL,
  \`phone\` varchar(20) UNIQUE NOT NULL,
  \`address\` varchar(255) UNIQUE,
  \`email\` varchar(255) UNIQUE,
  \`created_at\` datetime NOT NULL,
  \`updated_at\` datetime NOT NULL
);

--bun:split

CREATE TABLE \`books\` (
  \`store_id\` integer NOT NULL,
  \`id\` integer PRIMARY KEY AUTO_INCREMENT,
  \`title\` text NOT NULL,
  \`created_at\` datetime NOT NULL,
  \`updated_at\` datetime NOT NULL
);

--bun:split

CREATE TABLE \`authors\` (
  \`id\` integer PRIMARY KEY AUTO_INCREMENT,
  \`book_id\` integer NOT NULL,
  \`created_at\` datetime NOT NULL,
  \`updated_at\` datetime NOT NULL
);

--bun:split

ALTER TABLE \`books\` ADD FOREIGN KEY (\`store_id\`) REFERENCES \`stores\` (\`id\`);

--bun:split

ALTER TABLE \`books\` ADD FOREIGN KEY (\`id\`) REFERENCES \`authors\` (\`book_id\`);
```

SQLファイルでは、 `--bun:split` でテーブル間を分割する必要があります。他のSQLを使っている場合も同様のファイルを用いることでマイグレーションを行うことができます。また、命名ルールを遵守していれば、複数のSQLファイルを用意しても構いません。

Bunを用いてマイグレーションを行いますが、Bunは `database/sql` を前提として動作しています。また、MySQLの場合はそれ以外に `go-sql-driver/mysql` を用いることが前提となっています。こちらを用いてMySQLとGoのコンテナの接続に関して詳しくは説明しませんが、`.env` ファイルで設定した環境変数を読み込む `env.go` とデータベースとの接続を行う `database.go` の例を示します。

env.go

```go
package config

type DatabaseEnv struct {
    Name      string \`env:"DB_NAME"\`
    User      string \`env:"DB_USER"\`
    Password  string \`env:"DB_USER_PASSWORD"\`
    Address   string \`env:"DB_ADDRESS"\`
    Collation string \`env:"DB_COLLATION"\`
}
```

database.go

```go
package database

import (
    "database/sql"
    "fmt"
    "os"
    "time"

    "github.com/caarlos0/env"
    "github.com/go-sql-driver/mysql"
    "github.com/uptrace/bun"
    "github.com/uptrace/bun/dialect/mysqldialect"

    "api/config"
)

func Connect() (*bun.DB, error) {
    cfg := config.DatabaseEnv{}
    if err := env.Parse(&cfg); err != nil {
        return nil, err
    }
    jst, err := time.LoadLocation(os.Getenv("DB_TZ"))
    if err != nil {
        return nil, err
    }
    c := mysql.Config{
        User:                 cfg.User,
        Passwd:               cfg.Password,
        Net:                  "tcp",
        Addr:                 cfg.Address,
        DBName:               cfg.Name,
        Collation:            cfg.Collation,
        Loc:                  jst,
        ParseTime:            true,
        AllowNativePasswords: true,
    }
    sqldb, err := sql.Open("mysql", c.FormatDSN())
    if err != nil {
        return nil, err
    }
    if err = sqldb.Ping(); err != nil {
        return nil, err
    }
    db := bun.NewDB(sqldb, mysqldialect.New())
    return db, nil
}
```

Bunを用いたDatabaseとの接続方法の箇所のみを抜き出したものが次になります。

```go
sqldb, err := sql.Open("mysql", c.FormatDSN())
if err != nil {
    return nil, err
}
if err = sqldb.Ping(); err != nil {
    return nil, err
}
db := bun.NewDB(sqldb, mysqldialect.New())
```

`go-sql-driver/mysql` で接続を確保し、pingを飛ばして接続を確認しています。その後、Bun.NewDB()でラップすることでBunでの接続を作っている形になります。こちらで確保した接続を用いてマイグレーションを行なっていきます。

まずは、bunのmigrationに用いるインスタンスを生成する必要があります。 `bun/migrate` のmigrate.Migrations()メソッドを使い、インスタンスを作成します。その後DiscoverCaller()メソッドを用いてSQLファイルの検索を行います。その後、Migratorインスタンスを生成し、データベースにmigrationテーブルを作成します。その後にSQLファイルを使用してマイグレーションを行います。実際のコードでは次のようになります。

migration.go

```go
package database

import (
    "context"
    "fmt"

    "github.com/uptrace/bun/migrate"

    "api/database"
)

func Migration() {
    var migrations = migrate.NewMigrations()
    db, err := database.Connect()
    if err != nil {
        panic(err)
    }
    defer db.Close()
    if err := migrations.DiscoverCaller(); err != nil {
        panic(err)
    }
        c := context.Background()
        migrator := migrate.NewMigrator(db, migrations)
    if err := migrator.Init(c); err != nil {
        panic(err)
    }
    group, err := migrator.Migrate(c)
    if err != nil {
        panic(err)
    }
    if group.IsZero() {
        fmt.Printf("there are no new migrations to run (database is up to date)\n")
        panic(err)
    }
}
```

## モデルの作成方法、データベースへのアクセス

Bunでデータベースを操作する場合モデルという概念を用います。Goの場合はstructを定義しBunタグを設定することでモデルを表しています。詳しくは [公式ドキュメント](https://bun.uptrace.dev/guide/models.html) を参照するのが良いですが、例としてStoresテーブルとBooksテーブルを示します。

shop.go

```go
package models

import (
    "time"

    "github.com/uptrace/bun"
)

type Store struct {
    bun.BaseModel \`bun:"table:stores"\`

    ID        int       \`bun:"id,pk"\`
    Name      string    \`bun:"name,notnull"\`
    Phone     string    \`bun:"phone,notnull,unique"\`
    Address   string    \`bun:"address,unique"\`
    Email     string    \`bun:"email,unique"\`
    CreatedAt time.Time \`bun:",nullzero,default:current_timestamp"\`
    UpdatedAt time.Time \`bun:",nullzero,default:current_timestamp"\`

    Books []*Book \`bun:"rel:has-many,join:id=store_id"\`
}
```

book.go

```go
package models

import (
    "time"

    "github.com/uptrace/bun"
)

type Book struct {
    bun.BaseModel \`bun:"table:books"\`

    Store   *Store \`bun:"rel:belongs-to,join:store_id=id"\`
    StoreID int    \`bun:"store_id,notnull"\`

    ID        int       \`bun:"id,pk"\`
    title     string    \`bun:"title,notnull"\`
    CreatedAt time.Time \`bun:",nullzero,default:current_timestamp"\`
    UpdatedAt time.Time \`bun:",nullzero,default:current_timestamp"\`
}
```

このように書くことができます。StoresテーブルとBooksテーブルは１対多の関係にあるため、Store structにhas-many、Book structにbelongs-toを書くことになります。また、タグ内の""で囲われた箇所はカンマ(,)の後ろにスペースを入れたくなりますが、カンマを入れると正しく動作しないため、間を空けずに記述する必要があります。

次に、この作成したモデルを用いてデータをデータベースから取り出します。1件のみ取り出す場合は作成したモデルの `＊struct` が必要になります。複数件の場合は `[]*struct` を用いて取り出します。以下にデータを取り出す際の例を示します。

select.go

```go
db, err := database.Connect()
if err != nil {
    return nil, err
}
defer db.Close()
stores := make([]*Store, 0)
if err := db.NewSelect().Relation("Book").Scan(context.Background()); err != nil {
    panic(err)
}
```

取り出すデータの絞り込みに必要なWhereなどもあるため、必要に応じて使っていく必要があります。

## golangci-lintを導入

Go言語は標準のformatterが非常に優秀なため、個人開発で行う場合はlinterなどを導入しなくても大きく問題になることは少ないと思います。しかし、複数人で開発を行なっていく場合、importの順番などを統一しておくことでより良い開発環境を整備することが可能になります。そこで導入を推すのが [golangci-lint](https://golangci-lint.run/) になります。gciやgofmtなどを個別に管理する必要がないため、有用です。また、Github Actionsで動作する [golangci-lint-action](https://github.com/golangci/golangci-lint-action) も準備されているため、pull\_requestの際に動作するように設定することで、コードの細かな揺れの抑止につながります。

golangci-lintはデフォルトではgciなどは有効化されていないため、`.golangci.yml` ファイルに設定を記述することで必要なlinterを有効化する必要があります。次はその一例になります。

.golangci.yml

```yaml
run:
  timeout: 5m
  skip-dirs:
    - "github.com/go-sql-driver/mysql"
  skip-dirs-use-default: false
  go: '1.20'

linters:
  enable:
    - bodyclose
    - containedctx
    - cyclop
    - dogsled
    - dupl
    - errname
    - errorlint
    - exportloopref
    - forcetypeassert
    - gci
    - gocognit
    - goconst
    - gofmt
    - gosec
    - lll
    - nilerr
    - nilnil
    - rowserrcheck
    - sqlclosecheck
    - tagliatelle
    - unconvert
    - whitespace

linters-settings:
  gci:
    sections:
      - standard
      - default
      - prefix(api)
  lll:
    tab-width: 0
    line-length: 120
```

Github Actionsで動作させるためにはworkflowを設定する必要があります。こちらの設定は公式のREADMEに記載されている内容を踏襲していただければ良いかと思います。 `golangci-lint.yaml` に設定を行う例になります。

golangci-lint.yaml

```yaml
name: golangci

on: [pull_request]

jobs:
  #略
  lint:
  needs: setup
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    - name: golangci-lint
      uses: golangci/golangci-lint-action@v3
      with:
        version: latest
        working-directory: ./src
        args: --config=.golangci.yml
```

CIを導入することで、レビューを行う際などにその一助となります。型や構文のエラー、適切なエラーハンドリングなどは気にする必要性が減少するため、コードの中身や動作に集中することができます。

## おわりに

GinやBunを使ってのAPI開発の基本的な方法は以上になります。この記事でも紹介しきれていない内容もあるので、公式のドキュメントなども見た上で開発していくと良いかと思います。

誤った情報などありましたら、ご指摘いただきますと幸いです。

[0](https://qiita.com/y_murotani/items/#comments)

[29](https://qiita.com/y_murotani/items/1c1a93fba22035b45d6a/likers)

26