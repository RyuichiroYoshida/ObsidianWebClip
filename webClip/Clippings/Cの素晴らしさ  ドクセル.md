---
title: C#の素晴らしさ | ドクセル
source: https://www.docswell.com/s/binnmti/53GMND-2024-10-09-113510#p1
author:
  - "[[Docswell]]"
published: 
created: 2025-05-06
description: ドクセルはスライドやPDFをかんたんに共有できるサイトです
tags:
  - CSharp
  - 思想系
  - Tech
read: false
---
106.2K Views

October 09, 24

スライド概要

シェア

[またはPlayer版](https://www.docswell.com/s/binnmti/#)

埋め込む [»CMSなどでJSが使えない場合](https://www.docswell.com/service/iframe)

ダウンロード

[ダウンロード(pdf - 525.3kB)](https://www.docswell.com/slide/53GMND/MR93D7VR/download)

## 関連スライド

##### 各ページのテキスト

[1.](https://www.docswell.com/s/binnmti/#p1)

C#の素晴らしさを 語る会 The meeting which talks the wonderfulness of C#

[2.](https://www.docswell.com/s/binnmti/#p2)

Myself 森理 麟(@moririring) 職業：ゲームプログラマ HP: moririringのHP Microsoft MVP for C# 2

- [https://twitter.com/moririring](https://www.google.com/url?q=https%3A%2F%2Ftwitter.com%2Fmoririring)
- [https://sites.google.com/site/moririringhp/](https://www.google.com/url?q=https%3A%2F%2Fsites.google.com%2Fsite%2Fmoririringhp%2F)

[4.](https://www.docswell.com/s/binnmti/#p4)

Room metro #21 大阪 10月26日（土）IIJ様 Paste! めとべやスペシャルサンクス Day！ 福井からのC#MVPが２人いらっ しゃります！ 是非参加してくださいー 同日英語勉強会もあります。。。 4

[5.](https://www.docswell.com/s/binnmti/#p5)

Paste! ExceptionalC++読書会vol.19大 阪特別篇 11月2日（土）大阪産業創造館 C++例外安全Day！ 東京からの豪華ゲスト επιστημηさん！ 是非参加してくださいー 5

[6.](https://www.docswell.com/s/binnmti/#p6)

プロローグ 6

[7.](https://www.docswell.com/s/binnmti/#p7)

1960年 Since 7

[8.](https://www.docswell.com/s/binnmti/#p8)

（プログラムの）歴史に名を 刻むであろう一人の天才がこ の世に誕生しました。 Birth 8

[9.](https://www.docswell.com/s/binnmti/#p9)

その男の名はアンダース・ヘ ルスバーグ。 His name is Anders Hejlsberg 9

[10.](https://www.docswell.com/s/binnmti/#p10)

言わずと知れたC#の父です。 He is father of C# 10

[11.](https://www.docswell.com/s/binnmti/#p11)

Only two type in human. この世には2種類の人間しか居 ません。 即ちC#を好きな人と… C#を知らない人。 11

[12.](https://www.docswell.com/s/binnmti/#p12)

本日の会の説明 12

[13.](https://www.docswell.com/s/binnmti/#p13)

It was not able to meet, if he was not. 彼が居なければ僕はC#に出会 うこともなく、今日ここにい る皆さんにも出会わなかった かも知れません。 そんな彼がプログラマにくれ たささやかなプレゼント。今 日はその素晴らしさを語る会 になります。 13

[14.](https://www.docswell.com/s/binnmti/#p14)

以前海外のセッションに出た 時、多くのセッションで参加 者が積極的に発言することが 非常に印象的でした。 しかも質問の仕方も上手く、 スピーカーも上手く対応しま す。 14

[15.](https://www.docswell.com/s/binnmti/#p15)

それ以来僕はコミュニケー ションがとれる発表が理想だ と思っています。 分からないことや聞いている うちに言いたいことが出来た ら是非積極的に発言してくだ さい！ 15

[16.](https://www.docswell.com/s/binnmti/#p16)

C#以外の他言語のコメンテータ の方にも参加して頂いています。 発表者からも気になったらどん どん聞いてください。 @pocketberserker F# @k\_maru JavaScript @irof Java @KazaKazaK C/Basic/asm @StoneGuitar777 C++ 16

[17.](https://www.docswell.com/s/binnmti/#p17)

発言者にはささやかなお土産 を用意していますー。 17

[18.](https://www.docswell.com/s/binnmti/#p18)

C#と Visual Studioと.NET Framework について 18

[19.](https://www.docswell.com/s/binnmti/#p19)

Anders’s work Turbo Pascalの原作者 Delphiのチーフアーキティク ト マイクロソフトのテクニカル フェロー 最近ではTypeScriptも設計 19

[21.](https://www.docswell.com/s/binnmti/#p21)

 C#が影響を受けた言語図 View of language C# was affected  引用:http://exploringdata.github.io/vis/programming-languages-influence-network/ 21

- [http://exploringdata.github.io/vis/programming-languages-influence-network/](https://www.google.com/url?q=http%3A%2F%2Fexploringdata.github.io%2Fvis%2Fprogramming-languages-influence-network%2F)

[22.](https://www.docswell.com/s/binnmti/#p22)

C# Visual Studio.NET framework C#はVisual StudioというIDEと 共に提供されており、C#で 作ったものは.NET Framework というフレームワーク上で動 きます。 22

[23.](https://www.docswell.com/s/binnmti/#p23)

Visual Studio Visual Studioとは痒い所に手が 届きまくる世界最強のIDE 編集、ビルド、デバッグまで 込みで開発可能 IDEを無料で提供している言語 はそう多くない。 ましてや同一会社が作ってい るのは珍しい 23

[24.](https://www.docswell.com/s/binnmti/#p24)

.NET .NET Frameworkとは？ 現在、.NETといえば.NET Frameworkの略語。 .NETとは元々Microsoft.NETと いうビジョン。  引用:「.NETとは何か？」http://www.atmarkit.co.jp/ait/articles/1105/30/news129.html 24

- [http://www.atmarkit.co.jp/ait/articles/1105/30/news129.html](https://www.google.com/url?q=http%3A%2F%2Fwww.atmarkit.co.jp%2Fait%2Farticles%2F1105%2F30%2Fnews129.html)

[25.](https://www.docswell.com/s/binnmti/#p25)

.NET Framework 25

[26.](https://www.docswell.com/s/binnmti/#p26)

CLI etc… 26

[27.](https://www.docswell.com/s/binnmti/#p27)

C+C+++C++++ 新たにプログラムを始める人 に言語を進めるなら僕はC#。 CやC++で難しかった概念を殆 ど簡易化しているので習得が 比較的容易。 C系を知っていれば基本だけな ら1日で作れる。 27

[28.](https://www.docswell.com/s/binnmti/#p28)

C++ ++ C#は活用の場が広い MSプラットフォームでは独壇場。 ASP.NETでWebもOK .NET MicroFrameworkで組み込み も XamarinでAndroid,iOSにも対応 MonoでMac,Linuxでも動作。 PlayStation MobileやUnityなど ゲームでも！ 28

[29.](https://www.docswell.com/s/binnmti/#p29)

C# Visual Studio.NET framework つまりC#と.NET Frameworkそ してVisual Studioは三位一体で 進化してきており、決して 別々に語ることは出来ない。 今日はC#だけでなくその3つ について素晴らしさを語る会 です。 29

[30.](https://www.docswell.com/s/binnmti/#p30)

年代 Visual Studio.NET Framework C# 2002年 2002 1.0 1.0 2003年 2003 1.1 1.1 2005年 2005 2.0 2.0 2006年 - 3.0 - 2008年 2008 3.5 3.0 2010年 2010 4.0 4.0 2012年 2012 4.5 5.0 ※C#自体は2000年登場 30

[31.](https://www.docswell.com/s/binnmti/#p31)

年代.NET Framework 2005年 2.0 2.0 64bitサポート,ジェ ジェネリクス, yield ネリック 2008年 3.5 LINQ,式ツリー 3.0 varキーワード,拡張 メソッド,ラムダ式, プロパティ,クエリ 式 2010年 4.0 マルチコアへの対 応 4.0 dynamic,オプショ ン引数 2012年 4.5 ストアアプリ用、 非同期 5.0 非同期処理 C# 31

[32.](https://www.docswell.com/s/binnmti/#p32)

\[beta\]
```
マルチコアや非同期をビック
リするぐらいシンプルに書け
る
 for (int i = 0; i <= 10; i++) { foo(); }

Simple!

↓
 Parallel.For(0, 10, i => { foo(); });

32
```

[33.](https://www.docswell.com/s/binnmti/#p33)

Roslyn Roslyn(ロズリン) 独自のコード解析ツールやリ ファクタリング ツール 対話的実行環境(REPL) で、1行 1行実行しながら書ける C#/VBへのDSL（domain specific language: 特定領域向け の独自言語）の組み込み。 33

[34.](https://www.docswell.com/s/binnmti/#p34)

C#と Visual Studioと.NET Framework の素晴らしさ 34

[35.](https://www.docswell.com/s/binnmti/#p35)

プロパティ Set関数,Get関数が以下で書け てしまう。 property  public string Name { get; set; } これだけで外部からはメン バー変数のように、内部から はメソッドのように使える 35

[36.](https://www.docswell.com/s/binnmti/#p36)

Path.Combine パス名の最後に\\があろうがな かろうが、成立する名前を返 してくれる Path.Combine  Path.Combine(@"C:\\work", "a.txt");  Path.Combine(@"C:\\work\\", "a.txt"); 36

[37.](https://www.docswell.com/s/binnmti/#p37)

結構長いこと知らずにこう書 いていた。  Path.Combine(@"C:\\work\\", "bbb\\\\a.txt"); ↓ 実はこうも書けちゃう。  Path.Combine(@"C:\\work\\", "bbb","a.txt"); 37

[38.](https://www.docswell.com/s/binnmti/#p38)

最後にVSで一番感動した機能 インデントされたプログラム を他のインデントにコピーし た時。 Paste! 38

[39.](https://www.docswell.com/s/binnmti/#p39)

ご清聴ありがとうございまし た Thank you for hearing my peppermint’s talk! 39