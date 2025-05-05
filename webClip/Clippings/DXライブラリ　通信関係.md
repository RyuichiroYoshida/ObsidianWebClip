---
title: DXライブラリ　通信関係
source: http://hisaming.blog.fc2.com/blog-entry-62.html
author: 
published: 
created: 2025-05-06
description: ConnectNetWork　他マシンに接続する   int ConnectNetWork(IPDATA IPData);LANやインターネット等で繋がっている他のマシンと通信状態を確立します。接続先のマシンはIPアドレスを使って指定します。接続に成功した場合はネットワークハンドルを、失敗した場合は-1を返します。CloseNetWork　接続を終了する   int CloseNetWork(int NetHandle);ConnectNetWork関数で確立した接続状態を終了し、データの送受信を終了します。Prepa...
tags:
  - Cpp
  - Tech
  - バックエンド
  - 通信
read: false
---
## Before Heaven

## 現在は、VB.NET VB を中心に書いてます。C# も随時 Up 中です。C++ etc は、停滞中です。。

### DXライブラリ　通信関係

**ConnectNetWork** 　他マシンに接続する  

```cpp
intConnectNetWork(IPDATA IPData);
```

LANやインターネット等で繋がっている他のマシンと通信状態を確立します。  
接続先のマシンはIPアドレスを使って指定します。  
接続に成功した場合はネットワークハンドルを、失敗した場合は-1を返します。  
  
**CloseNetWork** 　接続を終了する  

```cpp
intCloseNetWork(intNetHandle);
```

ConnectNetWork関数で確立した接続状態を終了し、データの送受信を終了します。  
  
**PreparationListenNetWork** 接続を受けられる状態にする  

```cpp
intPreparationListenNetWork(void);
```

自分のマシンが他マシンからのConnectNetWork関数によって接続を確立できる状態にします。  
  
**StopListenNetWork** 　接続を受けつけている状態を解除する  

```cpp
intStopListenNetWork(void);
```

PreparationListenNetWork関数によって接続受付状態になったマシンの受付状態を解除します。  
以降ConnectNetWork関数による通信接続要求には反応しなくなります。  
  
**NetWorkSend** 　データを送信する  

```cpp
intNetWorkSend(intNetHandle, void*Buffer, intLength);
```

ネットワークハンドルの示す接続先マシンにBufferの示すアドレスからLengthのバイト数分だけデータを送信します。  
  
**GetNetWorkDataLength** 　一時記憶バッファに溜まっているデータの量を得る  

```cpp
intGetNetWorkDataLength(intNetHandle);
```

相手マシンがNetWorkSend関数を使用して送信してきたデータは自機の受診データ一時記憶バッファに格納されます。  
この関数は、バッファに溜まっているデータの量を返します。  
  
**GetNetWorkSendDataLength** 　未送信のデータの量を得る  

```cpp
intGetNetWorkSendDataLength(intNetHandle);
```

NetWorkSend関数で送信したデータはすぐに送信されるわけではない場合があり、その場合はライブラリ内部のメモリ領域に一時的に保存されます。  
この関数は溜まっているデータ量を返します。  
  
**NetWorkRecv** 　一時記憶バッファに溜まっているデータを取得する  

```cpp
intNetWorkRecv(intNetHandle, void*Buffer, intLength);
```

自機の受信データ一時記憶バッファに格納されたデータを、Bufferが示すアドレスにLengthバイト数分だけコピーします。  
  
**NetWorkRecvToPeek** 　受信したデータを読み込む  

```cpp
intNetWorkRecvToPeek(intNetHandle, void*Buffer, intLength);
```

この関数は一時記憶バッファに溜まったデータを読み取ります。  
NetWorkRecv関数と違い、バッファから取得してもバッファのデータが失われません。  
  
**GetNewAcceptNetWork** 　新たな接続を示すネットワークハンドルを得る  

```cpp
intGetNewAcceptNetWork(void);
```

PreparationListenNetWork関数によって接続受付状態にした後、この関数によって新たに確立された接続のネットワークハンドルを得ます。  
確立された場合はネットワークハンドルを返し、確立されていない場合は-1を返します。  
  
**GetLostNetWork** 　破棄された接続を示すネットワークハンドルを得る  

```cpp
intGetLostNetWork(void);
```

通信エラーや、相手側のCloseNetWork関数使用などにより絶たれた接続を示すネットワークハンドルを得ます。  
  
**GetNetWorkAcceptState** 　接続状態を取得する  

```cpp
intGetNetWorkAcceptState(intNetHandle);
```

現在接続状態にあるのかどうかを調べるときに使用します。  
  
**GetNetWorkIP** 　接続先のIPアドレスを得る  

```cpp
intGetNetWorkIP(intNetHandle, IPDATA *IpBuf);
```

接続してきた、または接続したマシンのIPアドレスを得ます。  
  

関連記事

- [DXライブラリ　ジョイパッド入力関連関数](http://hisaming.blog.fc2.com/blog-entry-51.html)
- [DXライブラリ　マウス入力関連関数](http://hisaming.blog.fc2.com/blog-entry-52.html)
- [DXライブラリ　キーボード入力関連関数](http://hisaming.blog.fc2.com/blog-entry-53.html)
- [DXライブラリ　半角文字入力関連関数](http://hisaming.blog.fc2.com/blog-entry-54.html)
- [DXライブラリ　日本語入力関連関数](http://hisaming.blog.fc2.com/blog-entry-55.html)
- [DXライブラリ　ウェイト関係の関数](http://hisaming.blog.fc2.com/blog-entry-56.html)
- [DXライブラリ　時間関係の関数](http://hisaming.blog.fc2.com/blog-entry-57.html)
- [DXライブラリ　音利用関数](http://hisaming.blog.fc2.com/blog-entry-58.html)
- [DXライブラリ　音楽再生関数](http://hisaming.blog.fc2.com/blog-entry-59.html)
- [DXライブラリ　乱数取得関数](http://hisaming.blog.fc2.com/blog-entry-60.html)
- [DXライブラリ　ウィンドウモード・フルスクリーンモード](http://hisaming.blog.fc2.com/blog-entry-61.html)
- DXライブラリ　通信関係
- [DXライブラリ　ムービー関係](http://hisaming.blog.fc2.com/blog-entry-63.html)
- [DXライブラリ　ファイル読み込み関係](http://hisaming.blog.fc2.com/blog-entry-64.html)
- [DXライブラリ　マイナー関数](http://hisaming.blog.fc2.com/blog-entry-65.html)

テーマ:[プログラミング](https://blog.fc2.com/theme-42597-20.html "プログラミング") - ジャンル:[コンピュータ](https://blog.fc2.com/community-20.html "コンピュータ")

- [2017/01/25(水) 08:58:04](http://hisaming.blog.fc2.com/blog-entry-62.html) |
- [DXライブラリ](http://hisaming.blog.fc2.com/blog-category-9.html)
- | [コメント(0)](http://hisaming.blog.fc2.com/blog-entry-62.html#comment) |

### コメントの投稿

### 検索フォーム

### プロフィール

![Mayu](https://blog-imgs-104.fc2.com/h/i/s/hisaming/mayup2_1.jpg)

Author:Mayu  
  
Before Heaven  
  
使用しているメイン Pc は、訳あって、Windous 8.1 Pro です。  
  
システムは、  
  
プロセッサ：Intel(R) Core(TM)i7-4820K CPU @ 3.70GHz 3.70 GHz  
  
実装メモリ(RAM)：32.0GB  
  
システムの種類：64ビット オペレーティングシステム、X64ベース プロセッサ  
  
この環境で、Editor は、Visual Studio 2015 でコードを書いてます。

### 最新記事

- [C# 文字コードを取得する Asc 関数、文字コードから文字を取得する Chr 関数 (05/02)](http://hisaming.blog.fc2.com/blog-entry-87.html "C# 文字コードを取得する Asc 関数、文字コードから文字を取得する Chr 関数")
- [VB.NET C#　郵便番号から住所を検索するスピードを高速化する (04/13)](http://hisaming.blog.fc2.com/blog-entry-86.html "VB.NET C#　郵便番号から住所を検索するスピードを高速化する")
- [VB.NET VB　郵便番号から住所を検索するスピードをアップする　 (04/12)](http://hisaming.blog.fc2.com/blog-entry-85.html "VB.NET VB　郵便番号から住所を検索するスピードをアップする　")
- [C# 郵便番号から住所を検索表示する住所入力支援（郵便番号重複データ対応版） (04/09)](http://hisaming.blog.fc2.com/blog-entry-84.html "C#  郵便番号から住所を検索表示する住所入力支援（郵便番号重複データ対応版）")
- [日本郵便さんの KEN\_ALL.CSV ファイルを使用して郵便番号から住所を検索表示する C#版 (04/09)](http://hisaming.blog.fc2.com/blog-entry-83.html "日本郵便さんの KEN_ALL.CSV ファイルを使用して郵便番号から住所を検索表示する C#版")

### カテゴリ

[プログラミング (0)](http://hisaming.blog.fc2.com/blog-category-0.html "プログラミング")

[VB.NET (28)](http://hisaming.blog.fc2.com/blog-category-7.html "VB.NET")

┣ [VB.NET VB 文字列操作 (12)](http://hisaming.blog.fc2.com/blog-category-12.html "VB.NET VB 文字列操作")

┣ [VB.NET VB ネットワーク (4)](http://hisaming.blog.fc2.com/blog-category-8.html "VB.NET VB ネットワーク")

┗ [VB.NET VB 郵便番号・住所入力支援 (3)](http://hisaming.blog.fc2.com/blog-category-13.html "VB.NET VB 郵便番号・住所入力支援")

[C# (5)](http://hisaming.blog.fc2.com/blog-category-11.html "C#")

┣ [VB.NET C# 文字列操作 (1)](http://hisaming.blog.fc2.com/blog-category-15.html "VB.NET C# 文字列操作")

┗ [VB.NET C# 郵便番号・住所入力支援 (3)](http://hisaming.blog.fc2.com/blog-category-14.html "VB.NET C# 郵便番号・住所入力支援")

[C++ (24)](http://hisaming.blog.fc2.com/blog-category-1.html "C++")

┗ [DXライブラリ (23)](http://hisaming.blog.fc2.com/blog-category-9.html "DXライブラリ")

[Access (0)](http://hisaming.blog.fc2.com/blog-category-10.html "Access")

### 月別アーカイブ

- [2017/05 (1)](http://hisaming.blog.fc2.com/blog-date-201705.html "2017/05")
- [2017/04 (9)](http://hisaming.blog.fc2.com/blog-date-201704.html "2017/04")
- [2017/03 (10)](http://hisaming.blog.fc2.com/blog-date-201703.html "2017/03")
- [2017/02 (1)](http://hisaming.blog.fc2.com/blog-date-201702.html "2017/02")
- [2017/01 (23)](http://hisaming.blog.fc2.com/blog-date-201701.html "2017/01")
- [2016/12 (6)](http://hisaming.blog.fc2.com/blog-date-201612.html "2016/12")
- [2016/11 (3)](http://hisaming.blog.fc2.com/blog-date-201611.html "2016/11")
- [2016/05 (4)](http://hisaming.blog.fc2.com/blog-date-201605.html "2016/05")

### 最新コメント

### RSSリンクの表示

- [最近記事のRSS](http://hisaming.blog.fc2.com/?xml)
- [最新コメントのRSS](http://hisaming.blog.fc2.com/?xml&comment)

### リンク

- [管理画面](http://hisaming.blog.fc2.com/?admin)

[このブログをリンクに追加する](http://blog.fc2.com/?linkid=hisaming'\);)

### ブロとも申請フォーム

[この人とブロともになる](http://hisaming.blog.fc2.com/?mode=friends)

![QR](https://blog-imgs-79.fc2.com/h/i/s/hisaming/a493c39c2.jpg)