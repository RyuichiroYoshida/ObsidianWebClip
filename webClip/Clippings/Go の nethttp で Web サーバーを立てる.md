---
title: Go の net/http で Web サーバーを立てる
source: https://qiita.com/BitterBamboo/items/6119f7986a04c5a0ac57
author:
  - "[[Qiita]]"
published: 2022-08-23
created: 2025-05-06
description: 基本のコードマルチプレクサとして DefaultServeMux を使用し、http.HandlerFunc() でハンドラを付与していきます。package mainimport (	"fm…
tags:
  - Go
  - 1
  - バックエンド
  - 通信
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から1年以上が経過しています。

## 基本のコード

マルチプレクサとして `DefaultServeMux` を使用し、 `http.HandlerFunc()` でハンドラを付与していきます。

main.go

```go
package main

import (
    "fmt"
    "net/http"
)

func hoge(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "hoge")
}

func fuga(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "fuga")
}

func main() {
    server := http.Server{
        Addr:    ":8080",
        Handler: nil,
    }

    http.HandleFunc("/hoge", hoge)
    http.HandleFunc("/fuga", fuga)

    server.ListenAndServe()
}
```

## 解説

### 構造体 http.Server で Web サーバーを立ち上げる

構造体 `http.Server` の値に、Web サーバーの設定を記述することができます。  
そして `ListenAndServe()` メソッドを呼び出し、Web サーバーを立ち上げます。

main.go

```go
package main

import (
    "net/http"
)

func main() {
    server := http.Server{
        Addr:    ":8080",
        Handler: nil,
    }
    server.ListenAndServe()
}
```

ハンドラを登録していないので、どこをアクセスしても 404 になります。

*(厳密には `Handler: nil` のとき `DefaultServeMux` がハンドラとして使用されますが、 `DefaultServeMux` はマルチプレクサなので「どのパスにどのようなハンドラを結びつけるか」を記述しなくてはなりません。)*

### 単一のハンドラを登録する

構造体 `http.Server` のフィールド `Handler` は、 `http.Handler` 型を期待します。  
この `http.Handler` はインターフェースです。

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

したがって、 `ServeHTTP(http.ResponseWriter, *http.Request` のシグネチャを持つメソッドにより `http.Handler` を実装できます。

main.go

```go
package main

import (
    "fmt"
    "net/http"
)

type HelloHandler struct{}

// *HelloHandler がインターフェース http.Handler を実装
func (h *HelloHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "Hello, world!")
}

func main() {
    // HelloHandler 型の変数を宣言
    handler := HelloHandler{}

    server := http.Server{
        Addr:    ":8080",
        Handler: &handler,
    }
    server.ListenAndServe()
}
```

こうしてハンドラを登録できたので、localhost:8080 の任意のパスにアクセスすると、\`Hello, world!" と表示されるようになります。

### http.Handle() で複数のハンドラを登録する

構造体 `http.Server` のフィールド `Handler` が `nil` のとき、ハンドラとして `DefaultServeMux` が使用されます。

以下のように、 `DefaultServeMux` は 構造体 `ServeMux` 型のポインタです。

```go
type ServeMux struct {
    // 省略
}

var defaultServeMux ServeMux

var DefaultServeMux = &defaultServeMux
```

そして、 `*ServeMux` は インターフェース `http.Handler` を実装しています。

```go
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request) {
    // 省略
}
```

すなわち、 `DefaultServeMux` は `http.Handler` 型です。

この `DefaultServeMux` はマルチプレクサとして機能します。  
関数 `http.Handle()` を用いて `DefaultServeMux` に複数のハンドラを付与していきます。

main.go

```go
package main

import (
    "fmt"
    "net/http"
)

type HogeHandler struct{}
type FugaHandler struct{}

func (h *HogeHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "hoge")
}

func (h *FugaHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "fuga")
}

func main() {
    hoge := HogeHandler{}
    fuga := FugaHandler{}

    server := http.Server{
        Addr:    ":8080",
        Handler: nil, // DefaultServeMux を使用
    }

    // DefaultServeMux にハンドラを付与
    http.Handle("/hoge", &hoge)
    http.Handle("/fuga", &fuga)

    server.ListenAndServe()
}
```

localhost:8080/hoge にアクセスすると "hoge"、 localhost:8080/fuga にアクセスすると "fuga" と表示されます。

このとき、関数 `http.Handle()` は `DefaultServeMux` のメソッド `Handle()` を呼び出しています。

```go
func Handle(pattern string, handler Handler) { DefaultServeMux.Handle(pattern, handler) }
```

### http.HandleFunc() で複数のハンドラを登録する

関数 `http.HandleFunc()` を使用すると、もっと簡単にハンドラを登録できます。  
関数 `http.HandleFunc()` は第二引数に `HandlerFunc` 型の関数を期待します。

main.go

```go
package main

import (
    "fmt"
    "net/http"
)

// HandlerFunc 型の関数を定義
func hoge(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "hoge")
}

// HandlerFunc 型の関数を定義
func fuga(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "fuga")
}

func main() {
    server := http.Server{
        Addr:    ":8080",
        Handler: nil, // DefaultServeMux を使用
    }

    // HandlerFunc を Handler に変換
    // Handler を DefaultServeMux に付与
    http.HandleFunc("/hoge", hoge)
    http.HandleFunc("/fuga", fuga)

    server.ListenAndServe()
}
```

関数 `http.HandleFunc()` は `DefaultServeMux` のメソッド `HandleFunc()` を呼び出しています。

```go
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
    DefaultServeMux.HandleFunc(pattern, handler)
}
```

そして、 `DefaultServeMux` のメソッド `HandleFunc()` は、 `DefaultServeMux` のメソッド `Handle()` を呼び出しています。

```go
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
    if handler == nil {
        panic("http: nil handler")
    }
    mux.Handle(pattern, HandlerFunc(handler))
}
```

このとき、 `HandlerFunc(handler)` は `http.Handler` を実装しています。  
ここで `HandlerFunc` 型の定義を確認してみましょう。

```go
type HandlerFunc func(ResponseWriter, *Request)

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```

`HandlerFunc` 型にはメソッド `ServeHTTP(http.ResponseWriter, *Request)` が定義されています。  
したがって `HandlerFunc` 型で関数をキャストすると、キャストした結果は `http.Handler` を実装することになります。

## ServeMux によるルーティング

パスの末尾に `/` がない場合、完全一致でルーティングされます。  
パスの末尾に `/` がある場合、以下のようなルーティングとなります。

main.go

```go
package main

import (
    "fmt"
    "net/http"
)

func index(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "index")
}

func hoge(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "hoge")
}

func hogefuga(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "hogefuga")
}

func main() {
    server := http.Server{
        Addr:    ":8080",
        Handler: nil,
    }

    http.HandleFunc("/", index)
    http.HandleFunc("/hoge/", hoge)
    http.HandleFunc("/hoge/fuga/", hogefuga)

    server.ListenAndServe()
}
```

| 入力パス | 結果 |
| --- | --- |
| / | index |
| /random | index |
| /hoge | hoge |
| /hoge/fuga | hogefuga |
| /hoge/123/fuga | hoge |
| /hoge/123/fuga/456 | hoge |
| /hoge/fuga/456 | hogefuga |

また、 `/users/123/posts/456` のような URL からパスパラメータを取り出す機能はないので、自前で処理を書くか、他のライブラリやフレームワークを使用する必要があります。

## 参考

- [https://pkg.go.dev/net/http](https://pkg.go.dev/net/http)
- [Goプログラミング実践入門](https://www.amazon.co.jp/Go%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E5%AE%9F%E8%B7%B5%E5%85%A5%E9%96%80-%E6%A8%99%E6%BA%96%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA%E3%81%A7%E3%82%BC%E3%83%AD%E3%81%8B%E3%82%89Web%E3%82%A2%E3%83%97%E3%83%AA%E3%82%92%E4%BD%9C%E3%82%8B-impress-gear%E3%82%B7%E3%83%AA%E3%83%BC%E3%82%BA-Sheong-Chang-ebook/dp/B06XKPNVWV)

## 次回

[0](https://qiita.com/BitterBamboo/items/#comments)

[5](https://qiita.com/BitterBamboo/items/6119f7986a04c5a0ac57/likers)

5