---
title: C++のHTTPサーバ、HTTPクライアントライブラリ一覧
source: https://kagasu.hatenablog.com/entry/2021/04/04/124307
author:
  - "[[備忘録]]"
published: 2021-04-04
created: 2025-05-06
description: Ⅰ. はじめに タイトルの通り「C++のHTTPサーバ、HTTPクライアントライブラリ一覧」です。 Ⅱ. 一覧 cpprestsdk HTTP Server/Client C++でHTTP(S)でGET/POSTする（cpprestsdk の使い方） cpprestsdk をビルドしてstatic linkする cpp-httplib HTTP Server/Client ヘッダファイルのみ Android NDKで問題なく利用できた Drogon HTTP Server(Web Framework) Oat++ HTTP Server(Web Framework)/Client https:…
tags:
  - Cpp
  - 1
  - 通信
  - バックエンド
read: false
---
### Ⅰ. はじめに

タイトルの通り「C++のHTTPサーバ、HTTPクライアントライブラリ一覧」です。

### Ⅱ. 一覧

##### cpprestsdk

- HTTP Server/Client
- [C++でHTTP(S)でGET/POSTする（cpprestsdk の使い方）](https://kagasu.hatenablog.com/entry/2017/10/07/190551)
- [cpprestsdk をビルドしてstatic linkする](https://kagasu.hatenablog.com/entry/2017/10/07/190551)

##### cpp-httplib

- HTTP Server/Client
- ヘッダファイルのみ
- Android NDKで問題なく利用できた

##### Drogon

- HTTP Server(Web Framework)

##### Oat++

- HTTP Server(Web Framework)/Client
- [https://github.com/oatpp/oatpp](https://github.com/oatpp/oatpp)

##### mongoose

- HTTP Server/Client, WebSocket Server/Client, MQTT Server/Client, Socks5 Server
- イベント駆動型 非ブロッキング
- 2ファイルのみ（ヘッダとソース）
- 幅広い採用実績がある（OSS, 商用, 国際宇宙ステーション）
- Android NDKで問題なく利用できた

##### civetweb

- HTTP Server
- 2ファイルのみ（ヘッダとソース）
- mongooseのfork
- CGI, Luaをサポート

##### httpserver.h

- HTTP Server

##### libhv

- HTTP Server/Client, TCP/UDP Server/Client, RUDP, SSL/TLS, WebSocket, MQTT

### 参考

- [https://github.com/nothings/single\_file\_libs](https://github.com/nothings/single_file_libs)

[« AndroidStudioを利用してAndroidで実行可…](https://kagasu.hatenablog.com/entry/2021/04/04/181813) [Chrome拡張機能でProxy設定情報を取得する… »](https://kagasu.hatenablog.com/entry/2021/04/01/140120)