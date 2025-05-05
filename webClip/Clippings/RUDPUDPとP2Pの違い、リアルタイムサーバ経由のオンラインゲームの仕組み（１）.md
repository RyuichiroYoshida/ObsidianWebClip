---
title: RUDP/UDPとP2Pの違い、リアルタイムサーバ経由のオンラインゲームの仕組み（１）
source: https://www.fs.com/jp/blog/difference-between-rudpudp-and-p2p-and-how-online-games-work-via-realtime-servers-1-13413.html
author:
  - "[[FS]]"
published: 
created: 2025-05-06
description: RUDPとは「ReliableUDP(User Datagram Protocol)」の略で、リアルタイム通信に用いられる通信プロトコルの１つです。UDPのような高速通信を重視する特徴を持ちながら、メッセージ再送による高品質・信頼性の確保を実現できます。従来の電信機から現在のオンライン通話に至るまで、実用された技術の革新・普及は目を瞠るほど飛躍的でした。その裏にはUDP、TCP、RUDP、P2Pなどといった通信プロトコルの活用があります。
tags:
  - Tech
  - 通信
  - バックエンド
read: false
---
[

スイッチング

](https://www.fs.com/jp/blog/)[

ネットワーキング

](https://www.fs.com/jp/blog/)[

光モジュール

](https://www.fs.com/jp/blog/)[InfiniBandネットワーク](https://www.fs.com/jp/blog/)

[](https://www.fs.com/jp/blog/)[

イーサネットネットワーク

](https://www.fs.com/jp/blog/)[

10/25/40/100Gモジュール

](https://www.fs.com/jp/blog/)[

1/2.5Gモジュール

](https://www.fs.com/jp/blog/)[

DAC/AOC/ACC/AECケーブル

](https://www.fs.com/jp/blog/)[

光伝送モジュール

](https://www.fs.com/jp/blog/)[

ワイヤレス & アクセスモジュール

](https://www.fs.com/jp/blog/)[

その他

](https://www.fs.com/jp/blog/)

[

光ケーブル

](https://www.fs.com/jp/blog/)[

ラック & エンクロージャー

](https://www.fs.com/jp/blog/)[

WDM & 光アクセス

](https://www.fs.com/jp/blog/)[

銅線システム

](https://www.fs.com/jp/blog/)[

光テスター & ツール

](https://www.fs.com/jp/blog/)

 お知らせ



[サインインして](https://www.fs.com/login.html?redirect=/jp/blog/difference-between-rudpudp-and-p2p-and-how-online-games-work-via-realtime-servers-1-13413.html) 、お知らせを再読み込みしてください。

カートに0つのアイテムが入っています



ショッピングカートに製品が入っていません。

[ログインして](https://www.fs.com/jp/login.html?redirect=/jp/blog/difference-between-rudpudp-and-p2p-and-how-online-games-work-via-realtime-servers-1-13413.html) 保存済みカートを一覧 [アカウントを作成](https://www.fs.com/jp/regist.html?redirect=/jp/blog/difference-between-rudpudp-and-p2p-and-how-online-games-work-via-realtime-servers-1-13413.html) して24時間サポート、簡単なオンライン見積もりとアカウント管理をご利用いただけます。

お客様にベストな体験をご提供するために弊社ウェブサイト上でCookieを使用いたします。「クッキーを受け入れる」をクリックすること、またはこのサイトを引き続き使用することにより、 [クッキーポリシー](https://www.fs.com/jp/policies/privacy_policy.html) に同意したものと見なされます。クッキーの使用を [拒否](https://www.fs.com/jp/blog/) することができます。









![consult icon](https://resource.fs.com/mall/generalImg/202412191100515nbxlz.svg) ![livechat icon](https://resource.fs.com/mall/generalImg/202411281533469ks844.svg)