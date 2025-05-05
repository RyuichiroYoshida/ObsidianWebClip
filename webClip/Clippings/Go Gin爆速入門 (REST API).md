---
title: Go Gin爆速入門 (REST API)
source: https://qiita.com/ozora/items/0597e52b3f9c1759e292
author:
  - "[[Qiita]]"
published: 2021-05-29
created: 2025-05-06
description: "##　はじめにGoで圧倒的人気を誇るWebフレームワークのGinを使ってREST APIを爆速で構築するための入門です。コードはginのREADMEドキュメントを元にしています。https:/…"
tags:
  - Go
  - 1
  - Tools
  - バックエンド
  - 学習系
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から3年以上が経過しています。

## はじめに

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/407956/098fbaef-c096-599e-c163-aced79706c75.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F407956%2F098fbaef-c096-599e-c163-aced79706c75.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=0da9ecd486f0afba5e898909314ab28a)

Goで圧倒的人気を誇るWebフレームワークのGinを使ってREST APIを爆速で構築するための入門です。  
コードはginのREADMEドキュメントを元にしています。

### Ginの導入方法

```bash
mkdir test-app && cd test-app
go mod init test-app
go get -u github.com/gin-gonic/gin
```

## Quick Start

`/pin` というエンドポイントにアクセスすると `pon` と表示する簡単な例

main.go

```go
package main

import "github.com/gin-gonic/gin"

func main() {
    r := gin.Default()
    r.GET("/ping", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "message": "pong",
        })
    })
    r.Run()
}
```

`go run main.go` した後に

```bash
curl localhost:8080/ping
{"message":"pong"}
```

コードの説明を簡単にしていくと、

### gin.Default

gin.Defaultは\*gin.EngineをReturnする関数で、GenではこのEngineを使ってエンドポイントの追加やミドルウェアの登録を行うGinのコア的なやつ。  
gin.Engineでは以下のようなメッソドを用いてエンドポイントの登録を行うことになります。

- gin.Engine.GET
- gin.Engine.POST
- gin.Engine.PUT
- gin.Engine.DELETE

### gin.Context

GinではContextを用いてリクエストのパラメータやデータにアクセスしたりフォームからPUTされたパラメータにアクセスしたりすることができます。  
Contextのコードの中身はこんな感じです。  
Contextの理解は非常に重要なので元の確認しておくと理解が深まるでしょう。

context.go

```go
type Context struct {
    writermem responseWriter
    Request   *http.Request
    Writer    ResponseWriter

    Params   Params
    handlers HandlersChain
    index    int8
    fullPath string

    engine *Engine

    // Keys is a key/value pair exclusively for the context of each request.
    Keys map[string]interface{}

    // Errors is a list of errors attached to all the handlers/middlewares who used this context.
    Errors errorMsgs

    // Accepted defines a list of manually accepted formats for content negotiation.
    Accepted []string

    // queryCache use url.ParseQuery cached the param query result from c.Request.URL.Query()
    queryCache url.Values

    // formCache use url.ParseQuery cached PostForm contains the parsed form data from POST, PATCH,
    // or PUT body parameters.
    formCache url.Values
}
```

## API

以下の例ではEngineを使ってそれぞれのエンドポイントを表示しています。  
どのメッソドでも第一引数にエンドポイント、第二引数にControllerと呼ばれる関数を指定します。

main.go

```go
func main() {
    router := gin.Default()

    router.GET("/someGet", getting)
    router.POST("/somePost", posting)
    router.PUT("/somePut", putting)
    router.DELETE("/someDelete", deleting)
    router.PATCH("/somePatch", patching)
    router.HEAD("/someHead", head)
    router.OPTIONS("/someOptions", options)

    router.Run()
}
```

### Controller

Controllerの実態は引数にContextをとる関数で、役割としてはmain.goから振られたリクエストをにハンドルし、レスポンスを返します。

例えば、上の例でいうとこんな感じで書くことができます。

controller/controller.go

```go
func getting(c *gin.Context){
    c.JSONP(http.StatusOK, gin.H{
        "message": "ok",
        "data": SOMETHING,
    })
}

func posting(c *gin.Context) {
    /*
       DB操作など
    */
    if err != nil{
        c.String(http.StatusInternalServerError, "Server Error")
        return
    }
    c.JSON(http.StatusCreated, gin.H{
        "status": "ok",
    })
}
```

### パラメータを含んだパス

`:param` や `*param` を使ってパスにパラメータを指定できる。  
例えば、 `/user/:name` と書くと、 `/user/tom` はマッチするが、 `/user/tom/send` とか `/user/tom/` とかはエラーになってします。  
`/user/:name/*action` と書くと、 `/user/tom/send` や `/user/tom/` もマッチします。

main.go

```go
func main() {
    router := gin.Default()

    // 200 -> /user/john, 301 -> /user/john/, 404 -> /user/john/get
    router.GET("/user/:name", func(c *gin.Context) {
        name := c.Param("name")
        c.String(http.StatusOK, "Hello %s", name)
    })

    // 200 -> /user/john/get, /user/john/get/ok
    router.GET("/user/:name/*action", func(c *gin.Context) {
        name := c.Param("name")
        action := c.Param("action")
        message := name + " is " + action
        c.String(http.StatusOK, message)
    })

    router.POST("/user/:name/*action", func(c *gin.Context) {
        // /user/john/get/ok も /user/:name/*actionと等価になる
        fmt.Println(c.FullPath() == "/user/:name/*action")
    })

    router.Run(":8080")
}
```

### Querystring parameters

以下の例では `welcome?firstname=Jane&lastname=Doe` のようなエンドポイントを想定しています。  
`Context.DefaultQuery(parameter, defaultValue)` で該当するクエリがなかったときにデフォルト値を設定します。

main.go

```go
func main() {
    router := gin.Default()

    // welcome?firstname=Jane&lastname=Doe
    router.GET("/welcome", func(c *gin.Context) {
        firstname := c.DefaultQuery("firstname", "Guest")
        lastname := c.Query("lastname")

        c.String(http.StatusOK, "Hello %s %s", firstname, lastname)
    })
    router.Run(":8080")
}
```

### Only Bind Query String

この例だと、Personの各パラメータに対するクエリを自動でバインドしてくれます。

main.go

```go
type Person struct {
    Name    string \`form:"name"\`
    Address string \`form:"address"\`
}

func main() {
    route := gin.Default()
    route.Any("/testing", startPage)
    route.Run(":8085")
}

func startPage(c *gin.Context) {
    var person Person
    if c.ShouldBindQuery(&person) == nil {
        log.Println("====== Only Bind By Query String ======")
        log.Println(person.Name)
        log.Println(person.Address)
    }
    c.String(200, "Success")
}
```

実行例

```sh
$ curl "localhost:8085/testing?name=test&address=tokyo"

2020/08/16 21:04:59 ====== Only Bind By Query String ======
2020/08/16 21:04:59 test
2020/08/16 21:04:59 tokyo
[GIN] 2020/08/16 - 21:04:59 | 200 |      73.068µs |       127.0.0.1 | GET      "/testing?name=test&address=tokyo"
```

### Multipart/Urlencoded Form

これはapplication/x-www-form-urlencoded, multipart/form-dataの時にformを参照する方法のようです。

main.go

```go
func main() {
    router := gin.Default()

    router.POST("/form_post", func(c *gin.Context) {
        message := c.PostForm("message")
        nick := c.DefaultPostForm("nick", "anonymous")

        c.JSON(200, gin.H{
            "status":  "posted",
            "message": message,
            "nick":    nick,
        })
    })
    router.Run(":8080")
}
```

curlでテストすると正しく動いていることが確認できます。

```sh
$ curl -X POST localhost:8080/form_post -d 'message=urlencoded'
{"message":"urlencoded","nick":"anonymous","status":"posted"}

$ curl -X POST localhost:8080/form_post -F 'message=multipart'
{"message":"multipart","nick":"anonymous","status":"posted"}
```

### Grouping routes

Groupingすることでエンドポイント毎にHTTPメソッドを設定できます。

main.go

```go
func main() {
    router := gin.Default()

    v1 := router.Group("/v1")
    v1.GET("/get", func(c *gin.Context) {
        c.String(200, "/v1/GET")
    })

    v2 := router.Group("/v2")
    {
        v2.GET("/get", func(c *gin.Context) {
            c.String(200, "/v2/GET")
        })
    }

    router.Run(":8080")
}
```

### XML, JSON, YAML, ProroBufレンダリング

gin.Hで返さずに、構造体を定義してレスポンスを返すこともできます。

main.go

```go
func main() {
    r := gin.Default()

    r.GET("/someJSON", func(c *gin.Context) {
        c.JSON(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
    })

    r.GET("/moreJSON", func(c *gin.Context) {
        var msg struct {
            Name    string \`json:"user"\`
            Message string
            Number  int
        }
        msg.Name = "Lena"
        msg.Message = "hey"
        msg.Number = 123
        c.JSON(http.StatusOK, msg)
    })

    r.Run(":8080")
}
```

## Middleware

Ginでは各種ミドルウェアを適宜実装してあげる必要があります。  
以下の例では最初にミドルウェアなしでEngineを初期化しています。

### カスタムミドルウェア(log)

この例ではログをファイルに保存するようにミドルウェアを書き換えています。

main.go

```go
package main

import (
    "io"
    "os"

    "github.com/gin-gonic/gin"
)

func main() {
    // gin.DisableConsoleColor()

    f, _ := os.Create("gin.log")

    // gin.DefaultWriter = io.MultiWriter(f)
    gin.DefaultWriter = io.MultiWriter(f, os.Stdout)

    router := gin.Default()
    router.GET("/ping", func(c *gin.Context) {
        c.String(200, "pong")
    })

    router.Run(":8080")
}
```

また、以下のように出力されるログのフォーマットをカスタマイズすることも可能です。

main.go

```go
router := gin.New()
    router.Use(gin.LoggerWithFormatter(func(param gin.LogFormatterParams) string {
        return fmt.Sprintf("%s - [%s] \"%s %s %s %d %s \"%s\" %s\"\n",
            param.ClientIP,
            param.TimeStamp.Format(time.RFC1123),
            param.Method,
            param.Path,
            param.Request.Proto,
            param.StatusCode,
            param.Latency,
            param.Request.UserAgent(),
            param.ErrorMessage,
        )
    }))
```

デフォルトだとこうなのが

```sh
[GIN] 2020/08/16 - 18:48:10 | 200 |      83.465µs |       127.0.0.1 | GET      "/ping"
[GIN] 2020/08/16 - 18:48:12 | 404 |         860ns |       127.0.0.1 | GET      "/ping/pong"
```

デフォルトだとこうなのが

```sh
127.0.0.1 - [Sun, 16 Aug 2020 18:50:16 JST] "GET /ping HTTP/1.1 200 160.899µs "curl/7.54.0" "
127.0.0.1 - [Sun, 16 Aug 2020 18:50:19 JST] "GET /ping/pong HTTP/1.1 404 587ns "curl/7.54.0" "
```

こうなります。

## Graceful Shutdown

API WebサーバーをGracefulにshutdownする方法として以前はendlessなどの3rd partyを使うテクニックがあったようですが、標準パッケージにShutdown()が追加されたので、次のようなテクニックが生まれたようです。

kill signalを受け取るchannelとserverを動かすchannelを別にした後にShutdown()を動作させて、Graceful Shutdownを実現するみたいですね。

main.go

```go
func main() {
    router := gin.Default()
    router.GET("/", func(c *gin.Context) {
        time.Sleep(5 * time.Second)
        c.String(http.StatusOK, "Welcome Gin Server")
    })

    srv := &http.Server{
        Addr:    ":8080",
        Handler: router,
    }

    // Initializing the server in a goroutine so that
    // it won't block the graceful shutdown handling below
    go func() {
        if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Fatalf("listen: %s\n", err)
        }
    }()

    // Wait for interrupt signal to gracefully shutdown the server with
    // a timeout of 5 seconds.
    quit := make(chan os.Signal)
    // kill (no param) default send syscall.SIGTERM
    // kill -2 is syscall.SIGINT
    // kill -9 is syscall.SIGKILL but can't be catch, so don't need add it
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit
    log.Println("Shutting down server...")

    // The context is used to inform the server it has 5 seconds to finish
    // the request it is currently handling
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    if err := srv.Shutdown(ctx); err != nil {
        log.Fatal("Server forced to shutdown:", err)
    }

    log.Println("Server exiting")
}
```

[0](https://qiita.com/ozora/items/#comments)

[101](https://qiita.com/ozora/items/0597e52b3f9c1759e292/likers)

76