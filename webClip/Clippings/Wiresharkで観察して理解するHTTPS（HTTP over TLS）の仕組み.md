---
title: Wiresharkで観察して理解するHTTPS（HTTP over TLS）の仕組み
source: https://tech-blog.rakus.co.jp/entry/20240701/wireshark
author:
  - "[[RAKUS Developers Blog | ラクス エンジニアブログ]]"
published: 2024-07-01
created: 2025-05-06
description: はじめに HTTPS(HTTP Over TLS)とは SSL/TLS HTTPSの流れ 実際に通信を観察 自己署名証明書の用意 サーバーの作成 WireSharkの準備 リクエストを送信して観察 まとめ
tags:
  - Tech
  - Tools
  - 通信
  - セキュリティ
read: false
---
![](https://cdn-ak.f.st-hatena.com/images/fotolife/t/tech-rakus/20240701/20240701125613.png)

## はじめに

エンジニア２年目のTKDSです！  
普段何気なく使ってるほとんどのWebサイトが対応している [HTTPS](https://d.hatena.ne.jp/keyword/HTTPS) 通信の仕組みについて調べてみました。  
本記事では、 [Wireshark](https://d.hatena.ne.jp/keyword/Wireshark) を用いて [HTTPS](https://d.hatena.ne.jp/keyword/HTTPS) の内部動作を解析し、どのようにしてデータが保護されているのかを具体的に解説します。  
記事の後半では、 [Wireshark](https://d.hatena.ne.jp/keyword/Wireshark) を使って実際の通信データを観察し、暗号化プロセスの詳細を確認してみます。

## HTTPS(HTTP Over TLS)とは

[HTTPS](https://d.hatena.ne.jp/keyword/HTTPS) (HTTP Over [TLS](https://d.hatena.ne.jp/keyword/TLS))は、HTTPの暗号化版で、ウェブサイトとブラウザ間の安全な通信を実現する [プロトコル](https://d.hatena.ne.jp/keyword/%A5%D7%A5%ED%A5%C8%A5%B3%A5%EB) です。  
[TLS](https://d.hatena.ne.jp/keyword/TLS) を使用して、HTTP通信を暗号化することで、データの機密性、データの整合性、通信先サーバーの信頼性を確認できます。

## SSL/TLS

[SSL](https://d.hatena.ne.jp/keyword/SSL) （Secure Sockets Layer）と [TLS](https://d.hatena.ne.jp/keyword/TLS) （Transport Layer Security）は、インターネット上でデータを暗号化して送受信するための [プロトコル](https://d.hatena.ne.jp/keyword/%A5%D7%A5%ED%A5%C8%A5%B3%A5%EB) です。  
これらの [プロトコル](https://d.hatena.ne.jp/keyword/%A5%D7%A5%ED%A5%C8%A5%B3%A5%EB) は、ウェブブラウザとウェブサーバー間の通信を保護し、データの盗聴や改ざんを防ぐために使用されます。

[SSL](https://d.hatena.ne.jp/keyword/SSL) は1990年代初頭に [Netscape](https://d.hatena.ne.jp/keyword/Netscape) 社によって開発されました。  
最初のバージョンである [SSL](https://d.hatena.ne.jp/keyword/SSL) 2.0は1995年にリリースされましたが、セキュリティ上の [脆弱性](https://d.hatena.ne.jp/keyword/%C0%C8%BC%E5%C0%AD) が発見され、1996年に [SSL](https://d.hatena.ne.jp/keyword/SSL) 3.0に置き換えられました。  
その後、1999年に [SSL](https://d.hatena.ne.jp/keyword/SSL) 3.0を基にした [TLS](https://d.hatena.ne.jp/keyword/TLS) 1.0が登場し、現在では [TLS](https://d.hatena.ne.jp/keyword/TLS) 1.3が最新バージョンとして使用されています。

## HTTPSの流れ

[HTTPS](https://d.hatena.ne.jp/keyword/HTTPS) は次の流れで通信を行います。

1. クライアントがサーバーに [HTTPS](https://d.hatena.ne.jp/keyword/HTTPS) 接続を要求
2. サーバーが [SSL](https://d.hatena.ne.jp/keyword/SSL) / [TLS](https://d.hatena.ne.jp/keyword/TLS) 証明書（公開鍵も含む）をクライアントに送信
3. クライアントが証明書を検証し、サーバーの身元を確認
4. クライアントがセッション鍵（対称鍵）を生成
5. クライアントがセッション鍵をサーバーの公開鍵で暗号化して送信
6. サーバーが [秘密鍵](https://d.hatena.ne.jp/keyword/%C8%EB%CC%A9%B8%B0) を使ってセッション鍵を復号、この時点で、クライアントとサーバーの両方が同じセッション鍵を共有、以降の通信はこのセッション鍵を使って暗号化
7. クライアントが暗号化されたHTTPリク [エス](https://d.hatena.ne.jp/keyword/%A5%A8%A5%B9) トを送信
8. サーバーがリク [エス](https://d.hatena.ne.jp/keyword/%A5%A8%A5%B9) トを復号して処理
9. サーバーが暗号化されたHTTPレスポンスを送信
10. クライアントがレスポンスを復号して表示

[HTTPS](https://d.hatena.ne.jp/keyword/HTTPS) の流れの図を以下に記載します。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/T/TKDS/20240629/20240629023258.png)

暗号鍵をクライアントとサーバー間の通信で使うことで安全に通信していることがわかりました。

## 実際に通信を観察

次に [HTTPS](https://d.hatena.ne.jp/keyword/HTTPS) を観察してみます。  
通信先のサーバーをGoで用意して、 [curl](https://d.hatena.ne.jp/keyword/curl) で通信します。

### 自己署名証明書の用意

今回使う証明書を用意します。

```
sudo apt-get update
sudo apt-get install openssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout server.key -out server.crt
```

### サーバーの作成

今回使用するhttpサーバーを用意します。  
ファイル名はmain.goを想定しています。  
起動は、 `go run main.go` で行ってください。

- HTTPサーバー
```go
package main

import (
    "fmt"
    "net/http"
)

func helloHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, World!")
}

func main() {
    http.HandleFunc("/", helloHandler)
    fmt.Println("Starting HTTP server on :8080")
    if err := http.ListenAndServe(":8080", nil); err != nil {
        fmt.Println("Error starting HTTP server:", err)
    }
}
```
- [HTTPS](https://d.hatena.ne.jp/keyword/HTTPS) サーバー
```go
package main

import (
    "fmt"
    "net/http"
)

func helloHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, World!")
}

func main() {
    http.HandleFunc("/", helloHandler)
    fmt.Println("Starting HTTPS server on :8443")
    if err := http.ListenAndServeTLS(":8443", "server.crt", "server.key", nil); err != nil {
        fmt.Println("Error starting HTTPS server:", err)
    }
}
```

### WireSharkの準備

通信のキャプチャは [WireShark](https://d.hatena.ne.jp/keyword/WireShark) を使います。  
以下の手順にしたがって準備してください。

- インストール
```
sudo apt-get update
sudo apt-get install wireshark
sudo dpkg-reconfigure wireshark-common
sudo usermod -aG wireshark $USER
newgrp wireshark
```
- 起動
```
sudo wireshark
```
- キャプチャするネットワークインターフェースの選択  
	今回は [localhost](https://d.hatena.ne.jp/keyword/localhost) なので、loを選択します。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/T/TKDS/20240629/20240629023954.png)

- キャプチャの開始 開始ボタンを押すとキャプチャが開始されます。  
	HTTPの場合、上部のフィルタに `http.request and tcp.port == 8080` を入力します。  
	[HTTPS](https://d.hatena.ne.jp/keyword/HTTPS) の場合、上部のフィルタに `tls and tcp.port == 8443` と入力します。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/T/TKDS/20240629/20240629024248.png)

### リクエストを送信して観察

- HTTP  
	http用のサーバーを起動してからリク [エス](https://d.hatena.ne.jp/keyword/%A5%A8%A5%B9) トを送る。  
	`curl http://localhost:8080`

通常のHTTP通信が行われているのが確認できます。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/T/TKDS/20240701/20240701000942.png)

フィルタのhttp指定を外してすべての通信をみても、 [https](https://d.hatena.ne.jp/keyword/https) ではないので暗号化は行われていないことがわかります。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/T/TKDS/20240701/20240701001124.png)

- [HTTPS](https://d.hatena.ne.jp/keyword/HTTPS)  
	[HTTPS](https://d.hatena.ne.jp/keyword/HTTPS) 用のサーバーを起動してからリク [エス](https://d.hatena.ne.jp/keyword/%A5%A8%A5%B9) トを送る。  
	`curl -k https://localhost:8443`

[TLS](https://d.hatena.ne.jp/keyword/TLS) ハンドシェイクが行われていることが確認できます。  
暗号化されているのでどのhttpメソッドを実行しているか、エンドポイントのパスはなにかなどは通信内容から読み取れなくなっています。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/T/TKDS/20240701/20240701001205.png)

[TLS](https://d.hatena.ne.jp/keyword/TLS) の流れを少しみてみましょう。

- Client Hello  
	[TLS](https://d.hatena.ne.jp/keyword/TLS) （Transport Layer Security）接続の開始時にクライアントからサーバーに送信される最初のメッセージです。  
	クライアントがサポートするセキュリティ設定をサーバーに通知し、サーバーとクライアントの間で使用する暗号化方法の [ネゴシエート](https://d.hatena.ne.jp/keyword/%A5%CD%A5%B4%A5%B7%A5%A8%A1%BC%A5%C8) を開始します。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/T/TKDS/20240629/20240629024941.png)

- Server Hello  
	Server Helloをクライアントに送信し、使用する [TLS](https://d.hatena.ne.jp/keyword/TLS) のバージョン、暗号スイート、圧縮方法を確定します。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/T/TKDS/20240629/20240629025058.png)

- Change Cipher Spec, Application Data  
	Change Cipher Specをクライアントに送信し、以降の通信が暗号化されることを通知

![](https://cdn-ak.f.st-hatena.com/images/fotolife/T/TKDS/20240629/20240629025130.png)

以上が [TLS](https://d.hatena.ne.jp/keyword/TLS) の流れでした。

## まとめ

ここまで読んでいただきありがとうございました！  
この記事ではHTTPと [HTTPS](https://d.hatena.ne.jp/keyword/HTTPS) の通信を観察し、通信内容を比較しました。  
ふとした疑問から調べはじめた内容でしたが、知識の復習やツールの使い方を思い出す役に立ちました 。  
普段意識せずに使っているものも、実際に調べてみることで、知識の定着につながりました。

---

[« JDKバージョンとBigDecimalの挙動について](https://tech-blog.rakus.co.jp/entry/20240705/bigdecimal) [モバイルエンジニアのためのGoogle I/O 20… »](https://tech-blog.rakus.co.jp/entry/20240628/mobiletechcafe)