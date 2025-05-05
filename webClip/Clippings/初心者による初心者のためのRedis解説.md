---
title: 初心者による初心者のためのRedis解説
source: https://qiita.com/keinko/items/60c844bcf329bd3f4af8
author:
  - "[[Qiita]]"
published: 2021-06-30
created: 2025-05-06
description: "#はじめにインターン先でRedisについてまとめるよう言われました。そのとき作ったパワーポイントを元にした文章をQiitaにも投稿しておきたいと思います。対象者はRedis初心者、サーバ運用の…"
tags:
  - 1
  - バックエンド
  - インフラ
  - "#Redis"
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から3年以上が経過しています。

## はじめに

インターン先でRedisについてまとめるよう言われました。  
そのとき作ったパワーポイントを元にした文章をQiitaにも投稿しておきたいと思います。  
対象者はRedis初心者、サーバ運用の初心者等々です。  
自分も初心者なので、あまり期待しないでください。

## Redisとは(導入部)

最初に結論っぽいことをまとめます。  
Redisとは、

- 無料で使えるデータベース管理システムの一つ
- 高速にデータを処理することができる、という特徴がある
- データベースの種類としては"NO SQL"というものに分類される

です。全くわからなくても大丈夫です。これから丁寧に説明していきます。

ちなみにロゴはこんな感じ。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/208979/1408dba3-5728-f45e-3b38-793e02a2130b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F208979%2F1408dba3-5728-f45e-3b38-793e02a2130b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=c7adc7601ee01c7312b6166a4726022a)  
いくつかの単純な図形が描かれたブロックが複数積み上がっています。  
このデザインだってデザイナーが一生懸命考えて作ったものなので、なにか大きな意図があるはず…！

## 前提知識

誰だって最初からIT技術に詳しい人はいません。  
自分も初心者ですが、自分よりも初心者の人のために今回出てくる単語について解説していきます。

## データベースとは

「データベースサーバ」の略です。  
**データを保存する機能** と **そのデータを好きなように(取得 / 作成 / 書き換え / 削除)できる機能** の二つを合わせもつサーバのことです。

### (補足)サーバってなんやねん

レバーを引いたら、ビールが出てくるのがビールサーバ。  
ボタンを押したら、好きなドリンクが出てくるのがドリンクサーバ。  
何か入力したら、○○してくれるのが○○サーバ。  
あのデータちょうだいって言ったら、そのデータを送ってくれるのがデータベースサーバ。  
データ管理しといてって言ってデータ渡したら、そのデータを管理してくれるのもデータベースサーバ。  
他にも世の中にはいろんなサーバがありますが、全てこの文章に置き換えられると思います。

### SQLって言われるもの

研修とかで絶対に耳にしそうな言葉"SQL"。  
これは古くから利用されている、データベースを管理するシステムの総称です。  
また、そのシステムを動かすためのプログラミング言語のことを指します。  
データベースと言えばSQL、みたいな。  
このSQLはだいたい、RDBMS(Relational database management system)に分類されます。  
RDBMSは、データ管理においてユーザーに **安全性** と **汎用性** を提供します。

- 安全性を実現する例

【問題】  
Aさんが銀行のATMからBさんの口座に10000円送金しようとした。  
しかしながら、データを送信する途中で停電が起こりデータベースの電源が落ちてしまった。  
Aさんの口座の残高からは10000円消え、Bさんの口座は一銭も増えていない。  
Aさんのお金やいづこに…。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/208979/215df6ad-05a6-3e5d-04ca-746184a3de0b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F208979%2F215df6ad-05a6-3e5d-04ca-746184a3de0b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a1173410b6f173d61b8b446811006634)  
このような問題を解決するために、 *トランザクション処理* というものがRDBMSにはあります。  
これは、任意の処理に対して、ユーザが「これは大事な処理！」  
という設定を行うことができ、もしもその処理を行なっている途中に何かしらの障害が起きた場合には、その大事な処理を行う前のデータに巻き戻してくれる機能です。  
これで、Aさんのデータが消えてしまうことはありません。  
これは安全性を実現する機能の一例と言ってもいいのではないでしょうか。

- 汎用性を実現する例

Cさんはあるコンビニチェーンのマーケティングの部署に所属する社員さんです。  
「20代女性が夏によく買うもの」を調べて今後の売り上げを伸ばしたいと考えています。  
データベースに保存されている購買データからどのように分析していけばいいでしょうか。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/208979/8c7dd339-7f28-2a41-47e4-ef2de29ec63d.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F208979%2F8c7dd339-7f28-2a41-47e4-ef2de29ec63d.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=0e5b93bc5751ad1b92ed55ee5286fabd)  
これに関しての答えはCさんが考えろって話ですが、RDBMSは複雑な絞り込み検索機能を提供しています。  
これによってどんな対象に対する分析もできそうですね。

## RDBMSとRedisの比較

Redisについて解説する前に、まずは両者の仕組みをイメージしやすいような形で図式化します。

## RDBMSの仕組みのイメージ

左の男がみなさん、そして机、本棚、司書さんをまとめてデータベースとします。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/208979/6801056b-73f7-7fff-57cb-d9eaf175e8d4.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F208979%2F6801056b-73f7-7fff-57cb-d9eaf175e8d4.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=176f1a698217e3018a9f0719caf5624c)  
皆さんは好きな本を司書さんに持ってくるように頼み、机に出してもらって本の中身を見ることができます。  
このとき、複数の本を持ってきてもらって複数見ることも可能です。  
「1995年にベストセラーに選ばれた本全てと、それらの本の著者の別年代に出版された本も全てもってきて」なんていう無茶振りにも司書さんは応えてくれます。  
ただ、司書さんはしっかり確実に仕事を行うタイプで仕事のスピードは速いとは言えません。  
本を見終わったあとは本棚に戻してもらい、司書さんが安全に管理してくれます。

### RDBMSの長所・短所

＜長所＞

- 安全にデータを管理できる
- 検索機能が豊富

＜短所＞

- 処理が遅い

## Redisの仕組みのイメージ

基本的には本棚は存在せず、本を机に置きっ放しで管理するのがRedisです。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/208979/ac7c2c40-b03d-e2ad-4f7e-c341e17f5189.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F208979%2Fac7c2c40-b03d-e2ad-4f7e-c341e17f5189.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=7b94debeff0f612bcd610b27b63b79ea)  
一応、それぞれのデータは少しはまとまって整理されています。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/208979/29a4c282-f002-6675-ac26-5c03b3f75062.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F208979%2F29a4c282-f002-6675-ac26-5c03b3f75062.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=ffe77048089b6ea039bae0f4753e7ba7)  
例えば、「シリーズものの本はシリーズ順にまとめて紐で縛ってある」とか「とりあえず順番とかいう概念はないけど、関連性のありそうなものを袋にまとめておく」などです。

読みたい本を読むときは、探したい本がその机のどこにあるかは精通している机マスターにお願いしてとってもらいます。  
机マスターはとても俊敏に行動するタイプの人です。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/208979/83666307-b68d-0ec0-e4fa-e092d34d9d44.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F208979%2F83666307-b68d-0ec0-e4fa-e092d34d9d44.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=326f76db8e8d7b30289b1cb52b19ddfd)  
本を置くスペースがなくなってきたら、別の机を用意し、別の机マスターを雇って管理します。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/208979/d2eb41f6-aa0a-a515-17fd-b580c4db0c8b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F208979%2Fd2eb41f6-aa0a-a515-17fd-b580c4db0c8b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=920396f46693b31b3d14e33c2563feef)

やろうとすれば一つの大きなカバンで管理することも可能です。  
しかし、読みたい本を高速で持ってきてもらうのがこの管理システムの本質なので、カバンで管理することはあまりないと考えてもいいでしょう。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/208979/f60ab5a2-8cde-225a-939b-9897d38a9c63.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F208979%2Ff60ab5a2-8cde-225a-939b-9897d38a9c63.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=6fea6a19d04f122df69ff1c285821af7)

### Redisの長所・短所

＜長所＞

- 処理が速い

＜短所＞

- 単純なデータしか扱えない
- 複雑な検索は不可能
- 管理は安全でない

## なぜRedisは速いのか

今述べたイメージを、より現実のコンピュータの世界の話に近づけて「なぜRedisが速いのか」に関して言及します。  
コンピュータの世界では、このように一時的に保存、およびデータを参照する机のようなものを **メモリ** といいます。  
一方、本棚のような保存するための場所が **ハードディスクドライブ** です。  
(他にも種類はありますが、ここでは簡単のためそう呼ぶこととします。)  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/208979/a9b7b70c-66fd-b3eb-1e8f-42e81a61001a.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F208979%2Fa9b7b70c-66fd-b3eb-1e8f-42e81a61001a.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=dd6790d27a27a6279ff1f351bcda70d8)

メモリの上にあるものは、コンピュータの電源が落ちてしまえば消えてしまいます。  
したがって、データベースは常に電源がついているものですが、万が一電源が切れてしまえばRedisのデータは消えてしまうこととなります。

### なぜRDBMSは遅いのか

理由は3つ考えられます。あくまでもRedisと比べて、です。

- メモリとハードディスクの間を行き来する必要があるため
- 複雑な検索の場合、そもそも処理に時間がかかる
- 司書さんの仕事が丁寧だから

3つめについて補足するとRDBMSは安全性を実現させるために、データが消えていないか、データは重複していないか等の確認や、バックアップの作成を司書さんは行なってくれます。このため、一つ一つの処理が重くなります。

### インメモリデータベース

Redisのようにメモリの上でデータを管理するデータベースをインメモリデータベースといいます。  
特に計測はしたことはありませんが、ハードディスクとメモリでは圧倒的にメモリのほうが速いと考えてください。  
そもそもハードディスクのものをメモリに持ってきて処理を行うので当然と言えば当然なんですけれども。

### つまり

なぜ速いか= (データ&処理が単純) × (メモリ上で処理を行う)

## Redisについて

ここではより具体的にRedisについて見ていくこととします。

## Redisの仕組み

少し内容が難しくなるので飛ばしてもらっても構いません。

Redisは複数のコマンドが与えられても、一つ一つ順番に処理を行います。  
このような処理を **シングルスレッド** と言います。  
このへんの図は [大谷様の資料](https://www.slideshare.net/yujiotani16/redis-76504393) を使わせて頂きます。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/208979/6498d42c-2cef-b5ce-e9d7-3fd393e3066c.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F208979%2F6498d42c-2cef-b5ce-e9d7-3fd393e3066c.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=9be024e7887fdb6f1910feb1ddbac028)  
引用元： [Redisの特徴と活用方法について](https://www.slideshare.net/yujiotani16/redis-76504393)

たとえ同時にコマンドを実行した場合でも、Redisは順番に処理を行なっていきます。

### Redis cluster(Redisデータ分類機)

先ほどの図においてRedisと書かれた青い四角の部分が、以下の図の青い点線部分にあたります。  
これから説明していくので図に関して疑問に思っても先に進んでください。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/208979/6c6b9723-5091-eb03-4445-a1509136def8.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F208979%2F6c6b9723-5091-eb03-4445-a1509136def8.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=6d176194fbe0e0830f927f1347b806e0)  
引用元： [Redisの特徴と活用方法について](https://www.slideshare.net/yujiotani16/redis-76504393)

一つのメモリがもつ容量は少ないため、上図のように複数台のサーバを用意して容量を増やしていきます。  
複数台でデータを分散させ、データを保持することを **シャーディング** と呼び、シャーディングを管理するのが **Redis cluster** です。  
サーバとデータの関係は一対多対応となっています。  
「AというサーバのもつaっていうデータはA以外のサーバB,C,...いずれにも属さない」ということです。

### slotについて

Redis clusterの実際の処理としては、まず、データそれぞれにslotと呼ばれるidのようなものを設定し、それぞれ管理するデータベースを分けます。  
例えば、「slot0番〜slot5460番はnode1っていうサーバ」、「slot5461番〜slot10922番はnode2っていうサーバ」、「slot10923番〜slot16383番はnode3っていうサーバ」というふうに。  
ユーザーは、ある適当なデータベースサーバに「slot〇〇番のデータはありますか」という命令を送信し、ない場合には、データベース同士が通信を行い該当のslot番号のデータを見つけ、そのデータをユーザーに返します。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/208979/65f32f8e-3a27-9484-eff0-6e6dc26bd94e.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F208979%2F65f32f8e-3a27-9484-eff0-6e6dc26bd94e.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=abc523f4fdaca4fad0072df197294007)  
これがRedisの大まかな仕組みです。

### 実際の構築

ここで、実際に運用する場合を妄想します。  
データがどんどん増えてきたらサーバを増設することによって対応することができます。  
Redisにはそれぞれのサーバの担当slotを割り直す機能があります。  
これを **リシャーディング** と言います。

しかしながら、いちいちデータが増えてきたら新しいコンピュータを購入してRedisのセッティングをしてデータベースサーバとして運用するのは面倒です。  
お金を払えば、そういうことをいい感じに代わりにやってくれるのがAWSのサービスの一つである **Amazon ElastiCache** です。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/208979/7a54b256-ba67-883f-647f-1733aaa95bc3.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F208979%2F7a54b256-ba67-883f-647f-1733aaa95bc3.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=d3c238221660212163b6afb4f75b1401)  
もし自分で構築することになったらElastiCacheのドキュメントを見てください。

## 型の種類

Redisには5つのデータの型が存在します。  
これはNO SQLにしては豊富なほうだそうです。(詳しくない)  
[北村様の資料](https://www.slideshare.net/ssusera66c10/webredis) を参考にさせて頂きました。

### String型

Redisにおいて最も単純なデータ構造です。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/208979/77c11f9f-f566-8d41-aaaf-34437de02969.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F208979%2F77c11f9f-f566-8d41-aaaf-34437de02969.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=b70979b54eb752a156430add98e6a098)  
Rubyでいうhash、Pythonでいうdictionary型に近いと言えます。  
`["Aさんの年齢", "42歳"], ["Bさんの年齢", "24歳"], ...`的な。  
ここでいう `"Aさんの年齢"` が上図のKeyにあたり、 `"42歳"` がvalueにあたります。  
「String型」と言っても、valueに数値を入れれば簡単な四則演算は可能となっています。

### List型

String型が配列構造になったものです。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/208979/109750bb-875b-bcc7-4328-e01920a6fbaf.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F208979%2F109750bb-875b-bcc7-4328-e01920a6fbaf.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=1746f5cb67e07ce4b24cacceced6767a)  
最大約40億要素まで保持できるそうです。

### Set型

List型からインデックスがなくなったものです。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/208979/10d0fb27-574b-8646-12f1-598a64d13b03.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F208979%2F10d0fb27-574b-8646-12f1-598a64d13b03.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=65b98868c2c91de42a524af948817c9a)  
最初に説明したイメージ図でいうビニール袋にまとめるやつに近いです。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/208979/1192a67b-e2a1-50e7-b358-99b925cffeae.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F208979%2F1192a67b-e2a1-50e7-b358-99b925cffeae.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=d9a6d7ad3c5dc5e4ccaaaedca344985e)  
Valueに重複はありません。  
Listよりも高速に処理を行えます。  
無作為にデータを抽出できる、という意味では機械学習などに使えそうだな、と思いました。

### Sorted set型

Set型にスコアという概念がついたものです。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/208979/b9a53a3b-e244-eb2d-6df1-509533f76338.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F208979%2Fb9a53a3b-e244-eb2d-6df1-509533f76338.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=048b3bdc798027cfd725076c96138e57)  
こちらもValueに重複はありません。  
スコア順に並び替えたり、スコアでの閾値処理が可能となっています。  
ゲームのリアルタイム・ランキング表示などに使えそうです。

### Hash型

Setに文字列で指定できるfieldという概念がついたものです。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/208979/7866f602-c1cb-bd29-8b02-d924f6541485.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F208979%2F7866f602-c1cb-bd29-8b02-d924f6541485.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=8faaa1af1e06c4f7d3a7126a8ff9243d)  
RedisのString型がRubyのhashなら、RedisのHash型はRubyでいうhashの中にさらにhashがある構造のようなデータ型です。

## データの有効期限設定機能

少し話は変わりますが、Redisにはデータを自動的に削除する機能が存在します。  
これはRedisがメモリ上にデータを保存していく性質上、メモリの容量を削減するためのものです。  
有効期限を何秒か後に設定する方法と有効期限を日付で指定する方法があります。

```text
# "keyname"のkeyのvalueを60秒後に消去
EXPIRE keyname 60

# 上の処理をタイムスタンプ表現の日付で指定
EXPIREAT keyname 802537200
```

## バックアップ作成機能

インメモリデータベースなので基本的にはメモリの上でデータを管理しますが、バックアップを作成しメモリ外に保存することもできます。  
保存の方法は2種類存在します。  
同然ですが、これらを有効化すると処理速度が低下してしまいます。

### 秒数指定・回数指定で保存

保存のタイミングを、「"seconds"秒ごとに、または"changes"回変更されたとき`.rdb` ファイルとして保存する」というように指定する方法です。  
`redis.conf` ファイルを変更することによって保存設定を反映できます。

redis.conf

```
# 60秒ごとに、または10000回変更がされたときに.rdbファイルとして保存
save 60 10000
```

`.rdb` ファイルは次回Redis起動時に自動的に立ち上がるようになっています。

### 更新ごとに保存(AOF方式)

データベースの更新ごとにバックアップを作成するもの。

redis.conf

```
# AOF方式の保存を有効化
appendonly yes

# 常に更新されたら保存
appendfsync always
```

## まとめ

かなり長くなりましたが、Redisについて今回調べた内容をまとめると以下のようになります。

- (RDBMSよりも、)かなり高速に処理を行える。
- 基本的には消えてもいいデータを保存する。
- (他のNoSQLと比べて、)データの型が豊富に用意されている。
- データの有効制限を簡単に設定できる。
- (インメモリDBだけれども、)データのバックアップをメモリ外にとることができる。

間違っているところ、補足すべきところがあればご教授よろしくお願いします。

## 参考

Redisの特徴と活用方法について  
[https://www.slideshare.net/yujiotani16/redis-76504393](https://www.slideshare.net/yujiotani16/redis-76504393)

Webエンジニアのためのはじめてのredis  
[https://www.slideshare.net/ssusera66c10/webredis](https://www.slideshare.net/ssusera66c10/webredis)

RedisとElastiCacheを分かりやすくまとめてみた  
[https://qiita.com/gold-kou/items/966d9a0332f4e110c4f8](https://qiita.com/gold-kou/items/966d9a0332f4e110c4f8)

## 補足

インターン先で発表したら、いろいろな補足、およびダメ出しを受けてより理解が深まりました。  
特にデータ取得以外にも、データ書き込み等のことについても言及すべきでした。

[3](https://qiita.com/keinko/items/#comments)

[396](https://qiita.com/keinko/items/60c844bcf329bd3f4af8/likers)

272