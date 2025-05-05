---
title: Go による WebSocket 通信
source: https://zenn.dev/empenguin/articles/bcf95c19451020
author:
  - "[[Zenn]]"
published: 2021-04-08
created: 2025-05-06
description: 
tags:
  - Go
  - Tech
  - バックエンド
  - 通信
read: false
---
65

1[tech](https://zenn.dev/tech-or-idea)

## はじめに

- 最近 [Real World HTTP 第2版](https://www.oreilly.co.jp/books/9784873119038/) のプロトコルのアップグレードの項を読んだのですが、現状プロトコルのアップグレードは WebSocket 専用であるとのことなので、復習も兼ねて Go による WebSocket 通信を行うサーバを実装してみました。

## WebSocket について

- WebSocket とはクライアントとサーバ間でオーバヘッドの小さい双方向の通信を行うためのプロトコルで、ユースケースとしてはチャットアプリが挙げられます。仕様に関しては [RFC 6455, The WebSocket Protocol](https://tools.ietf.org/html/rfc6455) に具体的な説明がされています。非公式な文書ですが、 [日本語訳された文書](https://triple-underscore.github.io/RFC6455-ja.html) も存在します。
- WebSocket による双方向の通信を行うためには、先ほども述べたとおりはじめに HTTP のプロトコルアップグレードの機能を使用します。具体的にはクライアントは以下のようなリクエストをサーバに対して行います。

```
GET /ws HTTP/1.1
Host: server.localhost
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Origin: http://localhost
Sec-WebSocket-Version: 13
```

- `Upgrade` 、 `Connection` ヘッダを付与することにより、使用するプロトコルを HTTP から WebSocket にアップグレードする旨をサーバに対して知らせています。 `Sec-WebSocket-Key` ヘッダは特定のクライアントとのコネクションかどうかを判定するために使用され、これによりなりすましを防止することができます。 `Sec-WebSocket-Version` ヘッダに関しては現在のバージョンである13を設定します。
- クライアントがリクエストを実行したのち、サーバからは以下のようなレスポンスが返されます。

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

- `Sec-WebSocket-Accept` ヘッダの値は `Sec-WebSocket-Key` ヘッダの値に対応したものが設定されています。 この後は WebSocket を使用してクライアント・サーバ間の双方向通信を行うことができます。

## 実装

- 今回はサーバ側の実装を Go、クライアント側の実装を HTML + JavaScript で行いました。ファイル構成は以下です。

```shell
$ tree
.
├── main.go
├── public
│   ├── index.html
│   └── main.js
..
```

## サーバ側の実装

- まずはサーバ側の実装を以下に示します。今回 Go のバージョンは `1.15.5` 、Web Framework に [Echo](https://echo.labstack.com/) を使用しています。バージョンは `v4.2.1` です。

main.go

```
package main

import (
    "fmt"
    "github.com/labstack/echo/v4"
    "github.com/labstack/echo/v4/middleware"
    "golang.org/x/net/websocket"
)

func handleWebSocket(c echo.Context) error {
    websocket.Handler(func(ws *websocket.Conn) {
        defer ws.Close()

        // 初回のメッセージを送信
        err := websocket.Message.Send(ws, "Server: Hello, Client!")
        if err != nil {
            c.Logger().Error(err)
        }

        for {
            // Client からのメッセージを読み込む
            msg := ""
            err = websocket.Message.Receive(ws, &msg)
            if err != nil {
                c.Logger().Error(err)
            }

            // Client からのメッセージを元に返すメッセージを作成し送信する
            err := websocket.Message.Send(ws, fmt.Sprintf("Server: \"%s\" received!", msg))
            if err != nil {
                c.Logger().Error(err)
            }
        }
    }).ServeHTTP(c.Response(), c.Request())
    return nil
}

func main() {
    e := echo.New()
    e.Use(middleware.Logger())
    e.Static("/", "public")
    e.GET("/ws", handleWebSocket)
    e.Logger.Fatal(e.Start(":8080"))
}
```

## クライアント側の実装

- 次にクライアント側の実装を示します。まずは HTML のコードです。

index.html

```html
<!doctype html>
<html lang="en">

<head>
    <meta charset="utf-8">
    <title>WebSocket</title>
    <script src="main.js"></script>
</head>

<body>
    <p id="output"></p>
    <input type="text" id="input"></p>
    <p><input type="submit" class="btn" value="送信"></p>
</body>

</html>
```

- 次に JavaScript のコードを示します。

main.js

```js
document.addEventListener('DOMContentLoaded', () => {
    let loc = window.location;
    let uri = 'ws:';
    if (loc.protocol === 'https:') {
        uri = 'wss:';
    }
    uri += '//' + loc.host;
    uri += loc.pathname + 'ws';

    const ws = new WebSocket(uri)
    ws.onopen = function() {
        console.log('Connected')
    }

    ws.onmessage = function(evt) {
        let out = document.getElementById('output');
        out.innerHTML += evt.data + '<br>';
    }

    const btn = document.querySelector('.btn')
    btn.addEventListener('click', () => {
        ws.send(document.getElementById('input').value)
    })
});
```

## 確認

- 実際にサーバを起動して動作確認をしました。 `http://localhost:8080` にブラウザでアクセスし、テキストボックスに文字を入力して「送信」ボタンを押下したところ、サーバ側から入力した文字列を含むメッセージが返却されることを確認しました。動作確認を行った時の画像を以下に示します。

![](https://storage.googleapis.com/zenn-user-upload/pja9ksmiz62ynex2hq3om8oq0q5f)

65

1