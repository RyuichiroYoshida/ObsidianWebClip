---
title: Vercel に Go サーバーを無料でデプロイできるんです！
source: https://zenn.dev/otakakot/articles/9e9269a87aafeb
author:
  - "[[Zenn]]"
published: 2024-07-06
created: 2025-05-06
description: 
tags:
  - Go
  - Tech
  - バックエンド
  - クラウド
read: false
---
192

2[tech](https://zenn.dev/tech-or-idea)

## はじめに

個人開発で使っていたのですが Gopher の集いにて Vercel で Go の開発ができることを話したら意外と知られていなかったのでご紹介します。  
（貧乏エンジニアリングだと言って似たような記事を以前お見かけしていた場合、それはきっと私の残像が書いたものです。）

## Vercel とは

いわゆる PaaS ( Platform as a Service ) です。  
Vercel 社が [Next.js](https://nextjs.org/) を開発していることもあり、静的サイトや SPA ホスティングを簡単にできるサービスの印象が強いですが、 Functions や Postgres などを備えているのでサーバーの構築もできちゃいます。  
Next.js のイメージから TypeScript と思われますが Go もデプロイできます。

## 準備 & version

Vercel アカウントを作成します。

無料で使いたいので `Hobby` 版を選択します。

各種操作を行うために [Vercel CLI](https://vercel.com/docs/cli) をインストールします。

node --version

```shell
v20.15.0
```

いくつかインストールの方法はありますが `npm` にてインストールします。

```shell
npm i -g vercel
```

vercel -v

```shell
Vercel CLI 34.3.0
34.3.0
```

プロジェクトを開始するために Vercel にログインしておきます。

開発のために Go をインストールしておきます。

go version

```shell
go version go1.22.5 darwin/arm64
```

## Functions のデプロイ

公式ドキュメントの通りに進めれば簡単にデプロイまで辿りつきます。

```shell
mkdir <project>
cd <project>
go mod init <project>
```

```shell
mkdir api
touch api/index.go
```

index.go

```
package api

import (
    "fmt"
    "net/http"
)

func Handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "<h1>Hello from Go!</h1>")
}
```

```shell
touch vercel.json
```

vercel.json

```json
{
  "build": {
    "env": {
      "GO_BUILD_FLAGS": "-ldflags '-s -w'"
    }
  }
}
```

ここまで準備ができたら CLI を使ってデプロイします。

```shell
vercel --prod
Vercel CLI 34.3.0
? Set up and deploy “~/<project>”? yes
? Which scope do you want to deploy to? <account>
? Link to existing project? no
? What’s your project’s name? <project>
? In which directory is your code located? ./
Local settings detected in vercel.json:
No framework detected. Default Project Settings:
- Build Command: \`npm run vercel-build\` or \`npm run build\`
- Development Command: None
- Install Command: \`yarn install\`, \`pnpm install\`, \`npm install\`, or \`bun install\`
- Output Directory: \`public\` if it exists, or \`.\`
? Want to modify these settings? no
🔗  Linked to <account>/<project> (created .vercel and added it to .gitignore)
🔍  Inspect: https://vercel.com/<account>/<project>/<random> [2s]
✅  Production: https://<project>.vercel.app [2s]
```

完全に好みの問題ですが、私は GitHub との連携はせず常にローカルから明示的にデプロイコマンドを実行しています。とくにこだわりがあるというわけでもないです。

```shell
curl https://<project>.vercel.app/api
<h1>Hello from Go!</h1>
```

こんな感じで簡単に Go サーバーが構築できちゃいます！

## Postgres と接続

`Hobby` プランだとアカウントあたり１つの Postgres が作成できます。

Postgres を作成するとプロジェクトから下記画像のように `Connect` ができるようになります。

![](https://storage.googleapis.com/zenn-user-upload/75add721662e-20240706.png)

`Connect` を行うと環境変数に各種値が設定されます。これらの値を Go 側から読み込んで Postgres に接続するようにします。

![](https://storage.googleapis.com/zenn-user-upload/80dafbd7af54-20240706.png)

実装すると以下のようになります。

index.go

```
package handler

import (
    "context"
    "fmt"
    "net/http"
    "os"

    "github.com/jackc/pgx/v5/pgxpool"
)

func Handler(w http.ResponseWriter, r *http.Request) {
    pool, err := GetPool(r.Context(), GetDSN())
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)

        return
    }

    defer pool.Close()

    fmt.Fprintf(w, "<h1>Hello from Go!</h1>")
}

func GetDSN() string {
    return "postgres://default:" + os.Getenv("POSTGRES_PASSWORD") + "@" + os.Getenv("POSTGRES_HOST") + ":5432/" + os.Getenv("POSTGRES_DATABASE") + "?sslmode=require"
}

func GetPool(
    ctx context.Context,
    dsn string,
) (*pgxpool.Pool, error) {
    conn, err := pgxpool.ParseConfig(dsn)
    if err != nil {
        return nil, err
    }

    pool, err := pgxpool.NewWithConfig(ctx, conn)
    if err != nil {
        return nil, err
    }

    if err := pool.Ping(ctx); err != nil {
        return nil, err
    }

    return pool, nil
}
```

ドライバーとして [`pgx`](https://github.com/jackc/pgx) を選択していますが Postgres なので [`pq`](https://github.com/lib/pq) でも接続できるはずです。

## 注意点

`go mod vendor` をするとデプロイに失敗します。（めんどくさいので原因追求していないです。）

```shell
Error: Command failed: go build -ldflags -s -w -o /tmp/1a06585b/bootstrap /vercel/path0/main__vc__go__.go
```

## お片付け

```shell
vercel project rm <project-name>
```

## おわりに

今回は軽くご紹介するくらいで終わりにします。  
ルーティングどうやるんだとかパスパラメータ使えるんだっけ？とかほかにも紹介できることがあるので気が向いたら記事書きます。

あと、Storage として [`Blob`](https://vercel.com/docs/storage/vercel-blob), [`KV`](https://vercel.com/docs/storage/vercel-kv), [`Edg Config`](https://vercel.com/docs/storage/edge-config) があるので触っておきます。（いまのところ使う予定がないので未検証）

貧乏エンジニアリング（いかに無料で開発をするか） の醍醐味はいかに自分に縛りを設けるかです。  
みなさん Go で開発をしたい場合はぜひ [Cloudflare](https://www.cloudflare.com/) × [workers](https://github.com/syumai/workers) を使いましょう！w

今回実装したコードは以下に置いておきます。

192

2

この記事に贈られたバッジ

![Thank you](https://static.zenn.studio/images/badges/paid/badge-frame-5.svg)