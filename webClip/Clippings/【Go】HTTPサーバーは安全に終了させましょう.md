---
title: 【Go】HTTPサーバーは安全に終了させましょう
source: https://zenn.dev/tksx1227/articles/5ab5b3c99336c3
author:
  - "[[Zenn]]"
published: 2024-06-21
created: 2025-05-06
description: 
tags:
  - Go
  - 1
  - バックエンド
read: false
---
146

2[tech](https://zenn.dev/tech-or-idea)

## はじめに

こんにちは。都内でソフトウェアエンジニアをしているtomoriです。

突然ですが、Go言語でHTTPサーバーを実装する際、サーバーの終了処理を適切に実装できている自信はありますか？

自分が開発に携わっているプロダクトでは、ほんの最近まで下記のような不適切な終了処理を行なっていました（話を簡単にするためにここでは `panic` を使っています）。

```
err := http.ListenAndServe(":8080", handler)
if err != nil {
    panic(err)
}
```

HTTPサーバー実装のサンプルとかでよく見るやつですね。

これだとアプリケーション側で、いわゆる Graceful Shutdown ができておらず、実行環境にて不具合を引き起こす恐れがあります。

というわけで、最近それを修正したのでアウトプットとして記事にします。

Go言語でHTTPサーバーを実装している方（特に初学者にこそ届いてほしい）の参考になれば嬉しいです。

## Graceful Shutdownとは

Graceful Shutdown とは、システムやアプリケーションが停止する際に、現在処理中のリクエストやトランザクションを適切に完了させ、データの一貫性や整合性を保ちながら安全に停止するプロセスを指します。

これにより、ユーザーへの影響を最小限に抑え、データの損失や破損を防ぐことができます。

具体的には、新しいリクエストの受け付けを停止し、現在のリクエストを完了させ、リソースを適切に解放する一連の手順を含みます。

## 安全に終了しないことによるユーザー影響

Graceful Shutdown を行わない場合、特に実行環境にアプリケーションをデプロイするときにほぼ必ず問題が生じるようになってしまいます。

例として、HTTPサーバーのコンテナが１台起動しており、そこでユーザーからのリクエストを都度処理している状況を考えます。

この時、 Graceful Shutdown を行わないアプリケーションを新たにデプロイすると、古いコンテナは新しいコンテナのデプロイが完了すると即座に停止されてしまいます。

この ”即座に停止” は通常アプリケーションの都合を待たずに行われます。

つまり、その時アクティブなコネクションがあった場合、それらも即座に破棄されてしまい、ユーザーに `4xx` や `5xx` のエラーが返るようになります。

デプロイの度に毎回一定数のリクエストが `4xx` や `5xx` エラーとして処理されてしまうのはシステムとしてよろしくない状態です。

## Graceful Shutdownの実現方法

Goの `net/http` には `Shutdown` というメソッドがあり、これを使用することで Graceful Shutdown を実現することができます。

`Shutdown` は以下の流れで安全にサーバーを終了させます。

1. 開いているすべてのリスナーを閉じる
2. アイドル状態のコネクションを閉じる
3. アクティブなコネクションがアイドル状態に戻るまで待機する
4. サーバーを終了する

> Shutdown gracefully shuts down the server without interrupting any active connections. Shutdown works by first closing all open listeners, then closing all idle connections, and then waiting indefinitely for connections to return to idle and then shut down.

HTTPサーバーを安全に終了させるために、サーバー終了時に `Shutdown` を呼び出すように修正します。

この `Shutdown` の呼び出し方に若干クセがあるので、その辺りを踏まえた実装方法を以降で解説していきます。

## 徐々に改善していく

サーバー終了処理のダメな実装例を順番に見ていきながら少しずつ改善していきます。

## 初期実装

冒頭に載せた実装に近いものになります。

とりあえずHTTPサーバーを起動してみる、という文脈でよくみる実装ですね。

```
func main() {
    addr := ":8080"
    handler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hello, playground"))
    })
    server := &http.Server{Addr: addr, Handler: handler}

    log.Printf("Server is running on %s", addr)
    if err := server.ListenAndServe(); err != nil {
        log.Fatalf("HTTP server ListenAndServe: %v", err)
    }

    log.Printf("Server is shut down")
}
```

`Shutdown` の呼び出しを追加しましょう。

## Shutdownを使用する

`defer server.Shutdown(context.Background())` として `Shutdown` を実行するようにしました。

```
func main() {
    addr := ":8080"
    handler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hello, playground"))
    })
    server := &http.Server{Addr: addr, Handler: handler}

    log.Printf("Server is running on %s", addr)
    err := server.ListenAndServe()
    defer server.Shutdown(context.Background())
    if err != nil {
        log.Fatalf("HTTP server ListenAndServe: %v", err)
    }

    log.Printf("Server is shut down")
}
```

一見良さそうに見えますが、これもダメな例です。

この実装だと `Ctrl+C` などの操作でサーバーを停止するとき、 `defer` どころか `if` を含むそれ以降の処理に到達することすらありません。

Goはデフォルトだと `SIGINT` や `SIGTERM` をランタイムパニックとして変換し、プログラムを終了する振る舞いをするように定められています。

> By default, a synchronous signal is converted into a run-time panic. A SIGHUP, SIGINT, or SIGTERM signal causes the program to exit.

そのため `SIGINT` などのシグナルを受け取った際、Goのデフォルトの挙動によって先ほどの実装の `server.ListenAndServe()` 以降の処理に到達することなく **即座にプロセスが終了** してしまいます。

これを防ぐために `SIGINT` や `SIGTERM` に対するシグナルハンドラに終了処理を記述しないといけません。

ではシグナルハンドラを追加しましょう。

---

余談ですが、実行環境によるアプリケーションの停止処理はシグナルによって行われます。

例えば、ECSだとタスクで起動しているコンテナは実行環境によって以下の流れで停止されると定められています。

1. 各コンテナの `PID=1` のプロセスに `SIGTERM` が送信される
2. `SIGTEMR` が送信されて30秒後に `SIGKILL` が送信される
3. コンテナのプロセスが停止する

> When a task is stopped, a `SIGTERM` signal is sent to each container’s entry process, usually PID 1. After a timeout has lapsed, the process will be sent a `SIGKILL` signal.

つまり、先ほどの実装だと `SIGTERM` を受け取った時点で即座にコンテナが終了してしまうことを意味します。

終了処理はシグナルの受信によって発火させる必要があるということですね。

## シグナルハンドラを追加する

新たに `goroutine` を作成し、そこで `SIGINT`, `SIGTERM` を受け取った際に `Shutdown` を実行するようにしました。

```
func main() {
    addr := ":8080"
    handler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hello, playground"))
    })
    server := &http.Server{Addr: addr, Handler: handler}

    go func() {
        c := make(chan os.Signal, 1)
        signal.Notify(c, os.Interrupt, syscall.SIGTERM) // SIGINT, SIGTERM を検知する
        <-c

        log.Printf("Server is shutting down...")
        if err := server.Shutdown(context.Background()); err != nil {
            log.Printf("HTTP server Shutdown: %v", err)
            return
        }
        log.Printf("Server is shut down")
    }()

    log.Printf("Server is running on %s", addr)
    if err := server.ListenAndServe(); err != nil {
        log.Fatalf("HTTP server ListenAndServe: %v", err)
    }
}
```

雰囲気良さそうですが、これでもまだシグナルを受け取った際に即座に終了してしまう実装になっています。

`Shutdown` はメソッドを呼び出した時点で `ListenAndServe` が即座に `ErrServerClosed` を返すという仕様になっています。

> When Shutdown is called, Serve, ListenAndServe, and ListenAndServeTLS immediately return ErrServerClosed. Make sure the program doesn't exit and waits instead for Shutdown to return.

そのため、上の実装では `Shutdown` を実行したものの、その処理の完了を待たずにプログラムが終了してしまいます。

これを防ぐために `Shutdown` の完了を待機させる処理を加える必要があります。

## Shutdownの完了を待機させる

`idleConnsClosed` と言うチャネルを作成し、 `Shutdown` の処理が完了するまで `main` が終了しないように制御するようにしました。

```
func main() {
    addr := ":8080"
    handler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hello, playground"))
    })
    server := &http.Server{Addr: addr, Handler: handler}
    idleConnsClosed := make(chan struct{})

    go func() {
        c := make(chan os.Signal, 1)
        signal.Notify(c, os.Interrupt, syscall.SIGTERM) // SIGINT, SIGTERM を検知する
        <-c

        log.Printf("Server is shutting down...")
        if err := server.Shutdown(context.Background()); err != nil {
            log.Printf("HTTP server Shutdown: %v", err)
            close(idleConnsClosed) // エラーが発生した場合は強制終了
            return
        }
        log.Printf("Server is shut down")
        close(idleConnsClosed) // Shutdown処理が完了したらチャネルを閉じる
    }()

    log.Printf("Server is running on %s", addr)
    if err := server.ListenAndServe(); err != nil {
        log.Fatalf("HTTP server ListenAndServe: %v", err)
    }

    <-idleConnsClosed // Shutdown処理が完了するまで待機する
}
```

これでシグナルを受け取った際に `Shutdown` を実行し、その処理が完了した後にプロセスを終了させる、と言う制御を実現することができました。

と思ったのですが、このコードを実際に実行してみるとサーバーの終了処理を待たずにプロセスが終了してしまいます。

これは `ListenAndServe` のエラーハンドリングが不適切な実装となっていることが原因です。

最後にそこを修正します。

## ListenAndServeのエラーハンドリングを正しく行う

`ListenAndServe` のエラーハンドリングを修正しました。

```
func main() {
    addr := ":8080"
    handler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hello, playground"))
    })
    server := &http.Server{Addr: addr, Handler: handler}
    idleConnsClosed := make(chan struct{})

    go func() {
        c := make(chan os.Signal, 1)
        signal.Notify(c, os.Interrupt, syscall.SIGTERM) // SIGINT, SIGTERM を検知する
        <-c

        log.Printf("Server is shutting down...")
        if err := server.Shutdown(context.Background()); err != nil {
            log.Printf("HTTP server Shutdown: %v", err)
            close(idleConnsClosed) // エラーが発生した場合は強制終了
            return
        }
        log.Printf("Server is shut down")
        close(idleConnsClosed) // Shutdown処理が完了したらチャネルを閉じる
    }()

    log.Printf("Server is running on %s", addr)
    if err := server.ListenAndServe(); !errors.Is(err, http.ErrServerClosed) {
        log.Fatalf("HTTP server ListenAndServe: %v", err)
    }

    <-idleConnsClosed // Shutdown処理が完了するまで待機する
}
```

修正箇所はこちらになります。

```
-    if err := server.ListenAndServe(); err != nil {
+    if err := server.ListenAndServe(); !errors.Is(err, http.ErrServerClosed) {
```

`ListenAndServe` は必ず non-nil エラーを返す実装となっています。

> ListenAndServe always returns a non-nil error. After Server.Shutdown or Server.Close, the returned error is ErrServerClosed.

そのため `err != nil` という条件だと、必ず条件を満たしてしまいエラーの際に期待していた分岐に進んでしまいます。

特に `Shutdown` や `Close` を実行した際には `ErrServerClosed` が返されることが明記されているため、 `ErrServerClosed` だった場合はエラーではなく、正常な挙動として処理を分岐させてあげます。

今回だと `err != nil` 時に `log.Fatal` を実行していたため、 `Shutdown` を待機せずにプロセスを終了してしまっていたので、そこを修正したという内容になります。

以上で修正はおしまいです。

最後に完成板を確認しましょう。

## 完成

こちらが全ての修正が完了した後の実装になります。

```
func main() {
    addr := ":8080"
    handler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hello, playground"))
    })
    server := &http.Server{Addr: addr, Handler: handler}
    idleConnsClosed := make(chan struct{})

    go func() {
        c := make(chan os.Signal, 1)
        signal.Notify(c, os.Interrupt, syscall.SIGTERM) // SIGINT, SIGTERM を検知する
        <-c

        ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
        defer cancel()

        log.Printf("Server is shutting down...")
        if err := server.Shutdown(ctx); err != nil {
            if errors.Is(err, context.DeadlineExceeded) { // タイムアウト時の処理を分ける
                log.Printf("HTTP server Shutdown: timeout")
            } else {
                log.Printf("HTTP server Shutdown: %v", err)
            }
            close(idleConnsClosed) // エラーが発生した場合は強制終了
            return
        }
        log.Printf("Server is shut down")
        close(idleConnsClosed) // Shutdown処理が完了したらチャネルを閉じる
    }()

    log.Printf("Server is running on %s", addr)
    if err := server.ListenAndServe(); !errors.Is(err, http.ErrServerClosed) {
        log.Fatalf("HTTP server ListenAndServe: %v", err)
    }

    <-idleConnsClosed // Shutdown処理が完了するまで待機する
}
```

１つ前の実装に加えて `Shutdown` 処理にタイムアウトを設けるようにしています。

```
-        log.Printf("Server is shutting down...")
-        if err := server.Shutdown(context.Background()); err != nil {
-            log.Printf("HTTP server Shutdown: %v", err)
-            close(idleConnsClosed)
-            return
-        }
+        ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
+        defer cancel()
+
+        log.Printf("Server is shutting down...")
+        if err := server.Shutdown(ctx); err != nil {
+            if errors.Is(err, context.DeadlineExceeded) {
+                log.Printf("HTTP server Shutdown: timeout")
+            } else {
+                log.Printf("HTTP server Shutdown: %v", err)
+            }
+            close(idleConnsClosed)
+            return
+        }
```

実行環境によっては `SIGTERM` を送信してから一定秒数後に `SIGKILL` を送信する、といった振る舞いをするものがあります。

`SIGKILL` による強制終了はできる限り避けたいので、あらかじめアプリケーション側でタイムアウトを設けて終了処理を行うようにしています。

これにより、タイムアウト時のログがあれば `SIGKILL` を送信するまでの間隔を調整したりすることができるようになります。

ご都合に合わせて追加してみてください。

## おわりに

ドキュメント内の `Shutdown` のサンプルコードを見れば「こういう書き方するんだなぁ」となんとなくわかりそうですが、なぜそうなっているのか、というところまでは理解できていなかったりするので、順を追って確認することで理解が深まるかと思います。

もし `Shutdown` を使用していない実装をしているのであれば、是非修正してみてください。

その際に少しでも本記事が参考になれば嬉しいです☺️

ではでは〜👋

## 参考記事

146

2

タグを追加