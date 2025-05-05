---
title: WebSocket 入門
source: https://zenn.dev/nameless_sn/articles/websocket_tutorial
author:
  - "[[Zenn]]"
published: 2023-10-09
created: 2025-05-06
description: 
tags:
  - Tech
  - 通信
  - バックエンド
read: false
---
226

6[tech](https://zenn.dev/tech-or-idea)

## はじめに

今回の記事ではWebSocketを解説する。

## 対象とする読者

- WebSocketについてわからないひと

## WebSocketとは？

WebSocketは双方向のHTTPプロトコルで、クライアントとサーバの通信で成立する。HTTPとは異なり、 `ws://` あるいは `wss://` から始まる。WebSocketはHTTPとは違って、クライアントとサーバ間の接続はどちらか一方が切断されると終了する。WebSocketが動く仕組みはHTTPのそれとは異なり、ステータスコード `101` がプロトコルの切り替えを示す。

## WebSocketが動く仕組み

Webページは多くの場合、HTTP接続を介してインターネット上で転送される。このプロトコルは、Webサイトがブラウザに表示されるようにデータを転送するために使われる。これを実現するために、クライアントはアクションを行うたびにサーバーにリクエストを行う。HTTPを使ってWebサイトにアクセスする場合、クライアントはまずサーバーにリクエストを送信する。その後、サーバーは要求された接続を送信することで応答する。

WebSocketはHTTPとは違って、Webサイトをリアルタイムで呼び出すことができるようになった。クライアントはWebSocketプロトコルの通信を送信してサーバーとの接続を確立しなければならない。この通信は、データ伝送に必要な識別情報をすべて提供する。

通信した後もチャンネルは開いたままなので、ほぼ常時通信ができる。つまり、クライアントがデータを要求しなくても、サーバーが勝手にデータを配信できるのだ。つまり、サーバーが新しいデータを受信した場合、クライアントに特別なリクエストを要求することなく、そのデータをクライアントに配信する。

下の図(Mermaid)は、WebSocket上の通信を簡潔に図式化したものである。

WebSocketは接続の確立、情報の交換や接続の切断をクライアント、サーバの双方向で行うHTTPプロトコルだ。

WebSocketで通信を始める時、クライアントはHTTPと同様にリクエストを送信する。

クライアントが以下のリクエストを送信すると、

```bash
GET /chatService HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Origin: http://example.com
Sec-WebSocket-Protocol: soap, wamp
Sec-WebSocket-Version: 13
```

サーバはこのように返す。

```bash
HTTP/1.1 101 Switching Protocols
Upgrade: WebSocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: wamp
```

## WebSocketとHTTPの違い

| **WebSocket** | **HTTP** |
| --- | --- |
| 双方向通信プロトコル。クライアントからサーバへ、かつサーバからクライアントへデータを送信できる。 | TCPプロトコル上で動作する単方向プロトコル。HTTPリクエストメソッドで接続できる。 |
| リアルタイムで更新が必要なアプリケーションで活用されている(特にチャットアプリ) | シンプルなRESTfulアプリケーションに使われる |
| 頻繁にデータを更新できるので、HTTPよりも高速。 | リクエストやレスポンスに従って動作するので、WebSocketよりも遅い。 |

## WebSocketの活用事例‐Socket.io

![](https://storage.googleapis.com/zenn-user-upload/8907aeb9ee94-20231005.png)

Socket.ioは、サーバー(Node.js)とクライアント(ブラウザ、Node.js、または他のプログラミング言語)間の双方向通信を確立するための技術だ。WebSocketの技術を応用したライブラリである。すべての環境やブラウザでWebSocketが使えるわけではないので、Socket.ioは「 **HTTPロングポーリング** 」を使う。これは、クライアントがサーバーに「新しい情報はありますか？」と定期的に問い合わせることで、新しい情報があればそれを受け取るということになる。

Engine.IOはサーバーとクライアント間の基本的な接続を担当し、WebSocketなどの通信を管理する。接続が開始する際に何らかの通信が行われ、情報のやりとりが開始する。

## 実装例

本章では、Socket.ioを用いて簡単なチャットアプリを作る方法を解説する。

## (1) モジュールのインストール

```bash
npm install express socket.io
```

## (2) アプリケーションの作成

以下の手順で実行する。

1. `public` という名前のディレクトリをルートに作る。
2. その中に `index.html` と `script.js` を作成する。
3. `app.js` でサーバを立ち上げる処理を書く。

まずは、 `index.html` でチャットアプリの簡単な画面をつくる。

public/index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Socket.io Chat</title>
</head>
<body>
    <input id="message" placeholder="Type a message">
    <button onclick="sendMessage()">Send</button>
    <ul id="messages"></ul>
    <script src="/socket.io/socket.io.js"></script>
    <script src="/script.js"></script>
</body>
</html>
```

次に、 `script.js` でチャットアプリ上で実行する処理を書く。

public/script.js

```js
const socket = io();

socket.on('chat message', function(msg) {
    const li = document.createElement('li');
    li.textContent = msg;
    document.getElementById('messages').appendChild(li);
});

function sendMessage() {
    const message = document.getElementById('message').value;
    socket.emit('chat message', message);
    document.getElementById('message').value = '';
}
```

最後に、 `app.js` でサーバを立ち上げる処理を書く。

app.js

```js
const express = require('express');
const http = require('http');
const socketIo = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = socketIo(server);

// このように書くことで\`public/script.js\`にアクセスできる
app.use(express.static('public'));

io.on('connection', (socket) => {
  socket.on('chat message', (msg) => {
    io.emit('chat message', msg);
  });
});

const PORT = 3000;

server.listen(PORT, () => {
  console.log(\`Server is running on http://localhost:${PORT}\`);
});
```

## (3) 実行

以下のコードを実行する。

```bash
node app.js
```

ブラウザ上で `http://localhost:3000` にアクセスすると簡単なチャットアプリケーションが表示される。

## 参考サイト

[GitHubで編集を提案](https://github.com/shota-nukumizu/zenn-book1/blob/main/articles/websocket_tutorial.md)

226

6