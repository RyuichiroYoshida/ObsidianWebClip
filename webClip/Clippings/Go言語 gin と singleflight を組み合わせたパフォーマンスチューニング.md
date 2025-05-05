---
title: "[Go言語] gin と singleflight を組み合わせたパフォーマンスチューニング"
source: https://tech.techtouch.jp/entry/go-gin-middleware-and-singleflight
author:
  - "[[Techtouch Developers Blog]]"
published: 2024-08-28
created: 2025-05-06
description: パフォーマンス最適化の文脈で登場することの多い singleflight と、鵜ウェブフレームワーク gin を組み合わせた実装例を紹介します。
tags:
  - Go
  - Tech
  - QA
  - バックエンド
read: false
---
![](https://cdn-ak.f.st-hatena.com/images/fotolife/i/izzii/20241118/20241118113437.png)

こんにちは。SRE 兼 テックブログ編集長の [izzii](https://x.com/ahneahneahne) です。毎年お盆休みに連泊で登山するのですが今年は台風の影響で行けませんでした。悲しい！

さて、本記事ではパフォーマンス最適化の文脈で登場することの多い [singleflight](https://pkg.go.dev/golang.org/x/sync/singleflight) と、ウェブフレームワーク [gin](https://pkg.go.dev/github.com/gin-gonic/gin) を組み合わせた実装例を紹介します。要素技術の概要とモチベーションに触れつつ、試行錯誤についてもお話しします。実装に関しては結局妥協が必要であり正解の形がないので GitHub で公開という形ではなく、ブログを通して自分の考えた２つの実装を紹介することにしました。この記事は Go に関する基礎知識を必要とします。

## singleflight とは

複数のスレッド（go routine）で同一の処理が同時になされるような場合、一つのスレッドで処理を実行して残りのスレッドで結果を受け取る形にすることで、処理にかかるリソース（e.g. DB）利用の効率化を図る仕組みです。本記事では同一の GET 系のウェブリクエストのリソース利用効率化のために利用しています。

singleflight よりもキャッシュの方がシンプルにパフォーマンス改善には効きますが、キャッシュの有効期限が切れた瞬間やそもそもキャッシュが効きづらい状況では singleflight が活躍します。

キャッシュの有効期限が切れた瞬間にリソースが高負荷に襲われる事象は、 [キャッシュスタンピード](https://en.wikipedia.org/wiki/Cache_stampede) と名前がつくくらいには一般的に起こりうる問題です。 [テックタッチも MAU 600万人の SaaS に成長してきた](https://prtimes.jp/main/html/rd/p/000000220.000048939.html) ため他人事ではありません。

キャッシュと singleflight は競合しないので、キャッシュ有効時はキャッシュがレスポンスし、キャッシュが無効な瞬間には singleflight がレスポンスする、ことでなるべくリソースへの負荷を下げる実装が可能になります。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/i/izzii/20240827/20240827112750.png)

[golang.org/x/sync/singleflight](https://pkg.go.dev/golang.org/x/sync/singleflight) というパッケージにライブラリが実装されています。

ちなみに下図はテックタッチのバックエンドアプリケーションに対する負荷試験時の DB のアクティブセッション数の時系列です。上下それぞれが singleflight なしとありのグラフとなります。縦軸のスケールが違うことには注意ですが、キャッシュがない状態で急激なトラフィックを与えても DB に対する負荷の上昇が抑えられることが見て取れます。具体的な負荷に関しては詳細を省きます。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/i/izzii/20240827/20240827112846.png)

## gin とは

[pkg.go.dev](https://pkg.go.dev/github.com/gin-gonic/gin)

gin は、Go 言語における最も著名なウェブフレームワークの一つです。概要的な説明は他の多くの記事と重複する可能性があるので避けます。 ここでは singleflight と組み合わせる上での２つの重要な概念に絞って説明します。

### Middleware

gin を使う場合、柔軟な middleware の仕組みを活用して認証認可やキャッシュといった、共通機能を middleware に集約することになるでしょう。例えば「singleflight middleware」のようなものを作ると、シンプルなシンタックスで様々なパスに対して singleflight の特性をアタッチできます。

```go
// Users API に一括で singleflight を設定している例

sfm := singleflightMiddleware.New()

users := api.Group("/users")
users.Use(sfm)
users.GET("/", Users.List)
users.GET("/:uuid", Users.Find)
```

### Context

gin は一つの HTTP リクエスト/レスポンスが一つの gin.Context に対応しています。コード上では gin.Context からリクエストの情報を取得し、レスポンスの情報を設定することで HTTP リクエストの処理を実装できます。HTTP ストリーム（※HTTP/2.0 以降の表現ですが適宜読み替えてください。）への書き込みのタイミングなどが抽象化されていて便利な反面、 [go ネイティブの context](https://pkg.go.dev/context) よりも柔軟性はありません。context は 枝分かれさせることでマルチスレッドアプリケーションでの状態管理が自由度高く記述できますが、gin.Context は枝分かれが制限されており、１つのリクエストで１つの gin.Context を強要されます。これによって下図のように HTTP ストリームと独立した gin.Context に処理を任せて HTTP ストリームに紐づいた gin.Context に結果を返すといった、原理的に副作用を分離するような singleflight の実装は難しいです。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/i/izzii/20240827/20240827113021.png)

また gin.Context は HTTP レスポンスのヘッダー設定（NOT HTTP ストリームへの書き込み）がプリミティブな map で実装されていることも singleflight 実装の上で制約となります。HTTP レスポンスヘッダーの設定を非同期に扱おうとすると concurrent write で panic が起きる可能性があります。gin.Context の状態の更新であったり、HTTP レスポンスのヘッダーの HTTP ストリームへの書き込み自体はスレッドセーフに作れられている/ハイジャック可能なので不思議です。

```go
// gin-gonic/gin/context.go
//
// Context にヘッダーを設定するメソッド
// 設定イコール HTTP ストリームへの書き込みではない
// このメソッドで mutex して欲しい
func (c *Context) Header(key, value string) {
    if value == "" {
        c.Writer.Header().Del(key)
        return
    }
    // この Writer は ResponseWriter
    c.Writer.Header().Set(key, value)
}

===

// net/http/server.go
//
// Header() は Header 型を返す
type ResponseWriter interface {
  // ...
    Header() Header
 
==

// net/http/header.go
//
// map でした
type Header map[string][]string

==

// gin-gonic/gin/context.go
//
// ちなみに Context に状態を設定するメソッドは mutex でスレッドセーフだったりする
// header もスレッドセーフにすればいいのに
func (c *Context) Set(key string, value any) {
    c.mu.Lock()
    defer c.mu.Unlock()
    if c.Keys == nil {
        c.Keys = make(map[string]any)
    }

    c.Keys[key] = value
}
```

これらの制約上 singleflight を gin の middleware として実装するのは妥協を伴いますが、メンテナンス性の観点で middleware として singleflight を実装するのが良いと判断しました。

## ２つの実装例

なぜ２つを紹介するかというと上記の gin.Context の制約上２つの実装に別々の妥協を伴うためです。実際にテックタッチのバックエンド実装に合う妥協点を探るために、２つの実装で動作確認をしています。

### 1\. 同期待ち方式

![](https://cdn-ak.f.st-hatena.com/images/fotolife/i/izzii/20240827/20240827113047.png)

上の画像のように一つの gin.Context に処理を任せて、同一のリクエストは同期的に待機させておいて結果を共有する形です。Do() 関数に一番最初に到達した gin.Context が処理を実行し、残りの gin.Context は Do() が実行するまで待たされるわけです。これは後述の「非同期待ち方式」に比べ gin.Context のキャンセルを検知した処理が書けないので、例えば代表 gin.Context の実行時間が長い場合に多くの goroutine がメモリ上に溜まり続ける可能性があります。

```go
import (
    "bytes"
    "context"
    "errors"
    "net/http"
    "slices"

    "golang.org/x/sync/singleflight"

    "github.com/gin-gonic/gin"
)

var sfg singleflight.Group

const SingleFlightContextKey = "singleflight"

/*
SingleFlightMiddleware は同じ通信をひとつの代表 context に集約して結果を返す仕組みです。
注意1：このミドルウェアではチャンク方式のレスポンスの利用は想定していません。
注意2：このミドルウェアよりも奥で c.Set() したものを非代表 context に反映させません。
これは c.Set() されたオブジェクトのリソース競合を防ぐための制約です。
注意3：このミドルウェアは panic を拾った場合、非代表 context にも引き継がせます。
*/
type SingleFlightMiddleware struct {
    keyFunc     func(*gin.Context) string
    allowedMethods []string // default: ["GET", "OPTIONS", "HEAD"]
}

type singleFlightResponse struct {
    Status int
    Header http.Header
    Data   []byte
    Errors []string
}

type sfWriter struct {
    gin.ResponseWriter
    body *bytes.Buffer
}

func (w *sfWriter) Write(b []byte) (int, error) {
    w.body.Write(b)
    return w.ResponseWriter.Write(b)
}

func (w *sfWriter) WriteString(s string) (int, error) {
    w.body.WriteString(s)
    return w.ResponseWriter.WriteString(s)
}

func (m *SingleFlightMiddleware) Handle(c *gin.Context) {
    // 一度 SingleFlight を通過した context や メソッドが許可されない場合は処理しない
    _, exists := c.Get(SingleFlightContextKey)
    if !slices.Contains(m.allowedMethods, c.Request.Method) || exists {
        c.Next()
        return
    }

    sfkey := m.keyFunc(c)
    c.Set(SingleFlightContextKey, "undelegated")

    // sfkey が共通の場合その後の処理を １つの context に代表させる
    v, _, _ := sfg.Do(sfkey, func() (r interface{}, err error) {
        // 代表 context であることを表す
        c.Set(SingleFlightContextKey, "delegated")
     
        // 代表 context がキャンセルされても後段の処理はキャンセルされない。
        // 書き込み内容を保存。
        w := &sfWriter{c.Writer, bytes.NewBufferString("")}
        ctx := context.WithoutCancel(c.Request.Context())
        c.Request = c.Request.WithContext(ctx)
        c.Writer = w
     
        c.Next()
        // c.Set() された値はスレッドセーフでないので引き継がない実装とした。
        // ケースバイケース
        r = singleFlightResponse{
            Header: c.Writer.Header().Clone(),
            Status: c.Writer.Status(),
            Data:   w.body.Bytes(),
            Errors: c.Errors.Errors(),
        }
        return
    })

    sfr := v.(singleFlightResponse)
    syncContext(c, &sfr)
 
    c.Abort()
}

func syncContext(c *gin.Context, sfr *singleFlightResponse) {
    if c.Writer.Written() {
        return
    }
    for k, vs := range sfr.Header {
        for _, v := range vs {
            c.Writer.Header().Add(k, v)
        }
    }
    c.Writer.WriteHeader(sfr.Status)
    if _, err := c.Writer.Write(sfr.Data); err != nil {
        c.AbortWithStatusJSON(http.StatusInternalServerError, "")
    }
    for _, e := range sfr.Errors {
        c.Error(errors.New(e))
    }
}
```

### 2\. 非同期待ち方式

![](https://cdn-ak.f.st-hatena.com/images/fotolife/i/izzii/20240827/20240827113119.png)

上の画像のように一つの gin.Context に処理を任せて、同一のリクエストは非同期的に待機させておいて結果を共有する形です。DoChan() 関数に一番最初に到達した gin.Context が処理を実行し、残りの gin.Context はチャネルを待ち受けます。この実装の良いところは待機 gin.Context のキャンセルが発火した際に gin.Context を回収できるところです。「同期待ち方式」と異なり、代表 gin.Context の実行に時間がかかる場合でも待機 gin.Context がキャンセル可能なので、singleflight が原因で余計に goroutine がメモリに溜まることはないでしょう。

しかし gin.Context の Header 設定がスレッドセーフでないことが、singleflight middleware よりも手前側の実装によっては潜在的なリスクになり得ます。

```go
import (
    "bytes"
    "context"
    "errors"
    "net/http"
    "slices"

    "golang.org/x/sync/singleflight"

    "github.com/gin-gonic/gin"
)

var sfg singleflight.Group

const SingleFlightContextKey = "singleflight"

/*
SingleFlightMiddleware は同じ通信をひとつの代表 context に集約して結果を返す仕組みです。
注意1：このミドルウェアではチャンク方式のレスポンスの利用は想定していません。
注意2：このミドルウェアよりも奥で c.Set() したものを非代表 context に反映させません。
これは c.Set() されたオブジェクトのリソース競合を防ぐための制約です。
注意3：このミドルウェアは panic を拾った場合、非代表 context にも引き継がせます。
*/
type SingleFlightMiddleware struct {
    keyFunc     func(*gin.Context) string
    allowedMethods []string // default: ["GET", "OPTIONS", "HEAD"]
}

type singleFlightResponse struct {
    Status int
    Header http.Header
    Data   []byte
    Errors []string
}

type sfWriter struct {
    gin.ResponseWriter
    body *bytes.Buffer
}

func (w *sfWriter) Write(b []byte) (int, error) {
    w.body.Write(b)
    return w.ResponseWriter.Write(b)
}

func (w *sfWriter) WriteString(s string) (int, error) {
    w.body.WriteString(s)
    return w.ResponseWriter.WriteString(s)
}

func (m *SingleFlightMiddleware) Handle(c *gin.Context) {
    // 一度 SingleFlight を通過した context や メソッドが許可されない場合は処理しない
    _, exists := c.Get(SingleFlightContextKey)
    if !slices.Contains(m.allowedMethods, c.Request.Method) || exists {
        c.Next()
        return
    }

    sfkey := m.keyFunc(c)
    c.Set(SingleFlightContextKey, "undelegated")

    // sfkey が共通の場合その後の処理を １つの context に代表させる
    ch := sfg.DoChan(sfkey, func() (r interface{}, err error) {
        defer func() {
            if rec := recover(); rec != nil {
                err = fmt.Errorf("panic in delegated singleflight go routine: %v", rec)
            }
        }()
        c.Set(SingleFlightContextKey, "delegated")
     
        // 代表 context がキャンセルされても後段の処理はキャンセルされない。
        // 書き込み内容を保存。
        w := &sfWriter{c.Writer, bytes.NewBufferString("")}
        ctx := context.WithoutCancel(c.Request.Context())
        c.Request = c.Request.WithContext(ctx)
        c.Writer = w
     
        c.Next()
        // c.Set() された値はスレッドセーフでないので引き継がない実装とした。
        // ケースバイケース
        r = singleFlightResponse{
            Header: c.Writer.Header().Clone(),
            Status: c.Writer.Status(),
            Data:   w.body.Bytes(),
            Errors: c.Errors.Errors(),
        }
        return
    })

    // context がキャンセルされた場合は処理を中断することで goroutine のリークを防ぐ
    // 例えば DB のレスポンスが遅い場合でも、クライアントからキャンセルされた go routine は代表 context を除いて解放される。
    // 代表 context は c.Next() の処理が終わらない場合 leak するがこの middleware の責任ではない。
    for {
        select {
        case <-c.Request.Context().Done():
            // 代表 context でなければ httpStatusCanceled を返して終了
            // 代表 context で Context().Done() は nil では？と思うかもしれませんが、
            // 非同期的に c.Request.ctx を入れ替えているのでタイミングによっては nil ではないです。
            // for で select を囲うのも上記の理由からです。
            if c.GetString(SingleFlightContextKey) == "undelegated" {
                c.AbortWithError(httpStatusCanceled, fmt.Errorf("context canceled"))
                return
            }
        case r := <-ch:
            if r.Err != nil {
                panic(r.Err)
            }
            sfr := r.Val.(singleFlightResponse)
            syncContext(c, &sfr)
            c.Abort()
            return
        }
    }
}

func syncContext(c *gin.Context, sfr *singleFlightResponse) {
    if c.Writer.Written() {
        return
    }
    for k, vs := range sfr.Header {
        for _, v := range vs {
            c.Writer.Header().Add(k, v)
        }
    }
    c.Writer.WriteHeader(sfr.Status)
    if _, err := c.Writer.Write(sfr.Data); err != nil {
        c.AbortWithStatusJSON(http.StatusInternalServerError, "")
    }
    for _, e := range sfr.Errors {
        c.Error(errors.New(e))
    }
}
```

## まとめ

パリッと正解な実装を出せない妥協だらけの実装が心苦しいですが、メンテナンス性の観点では gin middleware に実装するのは良い選択だと思います。アプリケーションによっては DB アクセスのためのレイヤーに実装するのも良いかもしれません。僕の実装を参考にしていただけるとすごく嬉しいですが、もっといい実装がある気もするのでもしアイデアあれば教えていただきたいです！

月一のブログ掲載のために最近やった仕事から捻り出したネタでしたが、最後までお付き合いいただけていたら心から感謝です。

[« AWS ECS で実現するBlue/Green Deployment…](https://tech.techtouch.jp/entry/production-ready-ecs-bg-sample) [最高効率でテストをするためにQaseを選ん… »](https://tech.techtouch.jp/entry/reason-for-choosing-qase)