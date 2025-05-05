---
title: UnityとRustでTCP通信をしてみる
source: https://qiita.com/ibukani/items/557a8cb87c3bcc507200
author:
  - "[[Qiita]]"
published: 2023-12-15
created: 2025-05-06
description: はじめにハッカソンでUnityとPhotonを使ってゲームを作ったのですが、その際にPhoton周りでうまく動作しないとうことが多く起こっていました。それがきっかけで、Unityのようなゲームエ…
tags:
  - Rust
  - Tech
  - 通信
  - Unity
  - バックエンド
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から1年以上が経過しています。

## はじめに

ハッカソンでUnityとPhotonを使ってゲームを作ったのですが、その際にPhoton周りでうまく動作しないとうことが多く起こっていました。

それがきっかけで、Unityのようなゲームエンジンを使いゲームを作る際に必要な通信技術を学んでみようという思いがあり、UnityとRustを使ってTCP通信を行ってみました！

## なぜTCP通信なのか？

FPSや格闘ゲームのように人と戦う系のゲームは速度を必要としており、UDP通信を使っていることが多いようです。  
しかし、UDP通信やゲームに関する通信技術のことを調べてもあまり詳しく情報が出てきませんし、有名なゲームエンジンは通信系のライブラリが既にあり、そちらを使ったほうが早いです。  
大抵のライブラリが実装しているのはゲーム内のオブジェクトの同期などのゲームを行うための通信を行うライブラリです。

通信速度を必要とせず、正確性のほうが必要なものは？と考えた結果、思いついたのがプレイヤーの戦績のような情報でした。そのため、今回はTCP通信で情報を送信しデータベースに保存できたら、いつか使えないだろうか？と思ったのでTCP通信にしました。

## 実装

今回はUnityがクライント、Rustがサーバーとして作っていきます。  
クライアントが接続後 `ping` という文字列を送り、それを受信したサーバーが `pong` を返すようなプログラムを作ります。

## 使用ライブラリ

### Rust

- Tokio

## クライアント (Unity側)

Client.cs

```c#
using System;
using System.Net.Sockets;
using System.Text;
using UnityEngine;

public class Client : MonoBehaviour
{
    private string address = "127.0.0.1";
    private int port = 2056;

    private TcpClient tcpClient;
    private NetworkStream networkStream;
    
    // プレイ開始時に呼び出し
    private void Start()
    {
        try
        {
            // サーバーに接続
            tcpClient = new TcpClient(address, port);
            networkStream = tcpClient.GetStream();

            Debug.Log("接続に成功しました");
        }   
        catch (SocketException)
        {
            Debug.Log("接続に失敗しました");
        }
    }

    // ボタンを押したときに呼び出す関数
    public void OnClick()
    {
        try
        {
            // サーバーへの送信処理
            var writeBuffer = Encoding.UTF8.GetBytes("ping");
            networkStream.Write(writeBuffer, 0, writeBuffer.Length);

            Debug.Log("Pingの送信に成功しました");

            // サーバーからの受信処理
            var readBuffer = new byte[1024];
            var bytesRead = networkStream.Read(readBuffer, 0, readBuffer.Length);

            var message = Encoding.UTF8.GetString(readBuffer, 0, bytesRead);
            Debug.Log("受信を確認しました　内容:" + message);
        }
        catch (Exception e) 
        {
            Debug.LogError("送受信に失敗しました" + e);
        }
    }

    // オブジェクトが破棄されたときに呼び出す
    private void OnDestroy()
    {
        tcpClient.Dispose();
        networkStream.Dispose();

        Debug.Log("切断しました");
    }
}
```

## サーバー (Rust側)

Cargo.toml

```toml
[package]
name = "unity_tcp_server"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
tokio = { version = "1.16.1", features = ["full"] }
```

main.rs

```rust
use tokio::{
    io::{AsyncReadExt, AsyncWriteExt},
    net::TcpListener,
};

#[tokio::main]
async fn main() {
    let listener = TcpListener::bind("127.0.0.1:2056").await.unwrap();

    println!("サーバー起動しました");

    loop {
        // 接続待ち
        let (mut stream, _) = listener.accept().await.unwrap();
        println!("接続を確認しました");

        tokio::spawn(async move {
            loop {
                // 受信の処理
                let mut read_buffer = [0; 1024];
                let bytes_read = stream.read(&mut read_buffer).await.expect("");

                // 切断の確認
                if bytes_read == 0 {
                    println!("切断を確認しました");
                    return;
                }

                // 受信されたメッセージの読み取り
                let message = String::from_utf8_lossy(&read_buffer[..bytes_read]);
                println!("受信を確認しました 内容:{}", message);

                // 送信の処理
                let write_buffer = "Pong".as_bytes();
                match stream.write_all(write_buffer).await {
                    Ok(_) => println!("Pongの送信に成功しました"),
                    Err(_) => println!("送信に失敗しました"),
                }
            }
        });
    }
}
```

## 実行結果

### クライアント (Unity側)

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3399220/9682697c-f3cc-ae2e-9d2b-5dc1a40f37c0.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3399220%2F9682697c-f3cc-ae2e-9d2b-5dc1a40f37c0.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=f1409fc02ec46ecee01c833fd89b2db7)  
送信と受信、どちらとも成功しています。

### サーバー (Rust側)

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3399220/bd415232-940a-3883-9e4b-6b995ade68bb.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3399220%2Fbd415232-940a-3883-9e4b-6b995ade68bb.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=5fafa3b0e3f7c8fd6f1c3a1d3ea088d8)  
Unity側から送られてきた情報がちゃんと読み取れていますね

## まとめ

初めてTcpListnerやTcpClinetを実装したのですがうまく動いてくれてよかったです。  
今回作ったものは、あくまで簡単な通信を行う実験として作ったため、もし今後プレイヤーの戦績のような情報をやり取りするためには、TCP通信やプロトコルスタックなどの知識を深める必要がありそうです。

## 参考文献

[0](https://qiita.com/ibukani/items/#comments)

[4](https://qiita.com/ibukani/items/557a8cb87c3bcc507200/likers)

4