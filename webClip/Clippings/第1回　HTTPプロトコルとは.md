---
title: 第1回　HTTPプロトコルとは
source: https://atmarkit.itmedia.co.jp/ait/articles/1703/29/news045.html
author:
  - "[[＠IT]]"
published: 
created: 2025-05-06
description: HTTPプロトコルの概要を知るための超入門連載（全3回）。今回は、HTTPプロトコルの概要と規格について説明する。
tags:
  - Tech
  - バックエンド
  - 通信
read: false
---
## 第1回　HTTPプロトコルとは：超入門HTTPプロトコル

HTTPプロトコルの概要を知るための超入門連載（全3回）。今回は、HTTPプロトコルの概要と規格について説明する。

» 2017年03月29日 05時00分 公開

\[， デジタルアドバンテージ\]

本入門連載では、システム管理者やシステムエンジニアの方々を主な対象として、IT業界でよく使われる技術や概念、サービスなどの解説をコンパクトにまとめておく。

- 第1回「HTTPプロトコルとは」（本記事）
- 第2回「 [HTTPプロトコルの詳細](https://atmarkit.itmedia.co.jp/ait/articles/1703/30/news042.html) 」
- 第3回「 [HTTPプロトコルの補足](https://atmarkit.itmedia.co.jp/ait/articles/1703/31/news045.html) 」（最終回）

---

## Webの基本プロトコル、HTTPとは？

　Webブラウザを使ってWebサイト（Webサーバ）にアクセスする場合、そこでは「 **HTTP（Hypertext Transfer Protocol）** 」というネットワークプロトコルが利用されている。Webブラウザのアドレスバーに「 **http:**//www.atmarkit.co.jp/」などと入力した場合の、「 **http:**」の部分のことである。これはHTTPというプロトコル **＊1** を使って、サイトwww.atmarkit.co.jpにアクセスせよ、という指令である。

**＊1** 以下では「HTTPプロトコル」とする。HTTPのPはProtocolの略なので、HTTPプロトコルと呼ぶとPが重複することになるが、TCPやIPなどと同様に、十分普及したHTTPという1つの単語と考えてこう呼ぶことにする。

[![HTTPのプロトコルを見る](https://image.itmedia.co.jp/ait/articles/1703/29/wi-httpfig03.png)](https://image.itmedia.co.jp/l/im/ait/articles/1703/29/l_wi-httpfig03.png) **HTTPのプロトコルを見る**  
HTTPはWebサーバからデータを取得するための基本プロトコルである。Webブラウザの開発者ツールを使うとHTTPプロトコルの通信内容をモニタできる。これはGoogle Chromeのデベロッパーツールの例（［Ctrl］＋［Shift］＋［I］で起動できる）。  
**（1）** レンダリングされたWebページの例。  
**（2）** HTTPのGETコマンドの詳細。

　Webサーバというと、Webブラウザを使ってアクセスするものと考えるかもしれないが、現在ではさまざまなサービスでWebサーバやHTTPプロトコルが使われている。WebページやWebサイトをホスティングするだけでなく、ファイル共有や、APIを介したデータ提供、リモートのシステム操作、スマホやWebブラウザ上で動作するアプリのバックエンド、ユーザー認証やID管理、暗号化された通信の提供など、その利用シーンは非常に広がっている。

　アプリケーションや用途ごとに今まで数多くのネットワークプロトコルが開発され利用されてきたが、最近ではWebサーバ（とHTTPプロトコル）をベースとしてシステムを構築し、その上で独自のサービスや機能を提供することが多い。セキュリティや管理上の都合などのため、HTTP以外のプロトコルはファイアウォールなどでブロックされることが多いし、HTTPだけでほぼどんなサービスでも提供できるようになってきたからでもある。

　昨今ではセキュリティを強化するため、HTTPに代わってHTTPS（HTTP over SSL/TLS、常時SSLなどとも呼ばれる）の導入も要請されるようになった。だがHTTPSはセキュアな通信路上でHTTPを利用できるようにしたものであり、中核のプロトコルについてはHTTPと大きな違いはない。

　利用シーンが急速に拡大しているHTTP／HTTPSであるが、その基本は開発された当初からほとんど変わっていない。通信速度が上がったり、ブラウザなどで利用できる機能が増えたりしたように見えるが、変わったのはHTTPで送受信される通信データの内容であって、HTTPプロトコルそのものには互換性を損なうような大きな変更はない（改善点は多数あるが）。

　本連載では、ますます重要性が増すHTTPとHTTPSプロトコルについて、その基本を解説する。まずはHTTPについて、全3回でまとめる。HTTPSについては、その後に別連載で取り上げる。。

## HTTPプロトコルの仕組み

　HTTPプロトコルの仕組みは非常にシンプルである。Webサーバ（HTTPのサーバ）とWebブラウザ（HTTPのクライアント）の2つが、TCPの80番ポートを使って、サーバ／クライアント形式で通信する。最初にサーバ側がTCPの80番ポートで待ち受け（リッスン）していて、クライアント側はその80番ポートに対してTCPの接続を行うのだ。

　TCPの接続が完了すると、今度はクライアント側からHTTPのファイル取得要求（GET要求）を1つ送信する。すると、それに対してサーバ側から応答を1つ返す。

![WebサーバとWebブラウザ間でコマンドをやりとりする図](https://image.itmedia.co.jp/ait/articles/1703/29/wi-httpfig01.png) **WebサーバとWebブラウザ間でコマンドをやりとりする図**

　例えば、あるサイトをWebブラウザで開くと（例：www.atmarkit.co.jpなど）、1回のHTTPプロトコルのやりとりで、まずトップページの要素（例えばindex.htmlファイルの内容）がクライアント側へ渡される。

　Webページを構成する「部品」が多数ある場合、それらも全てこの手順を繰り返して取得する。現在のWebサイトでは、画像ファイルやCSS、JavaScript、フォントなど、多くのデータが別ファイルに記述されているだろう。最初に取得したHTMLファイルから必要なファイルをたどって、それらも順次全て取得する。それを基にしてWebページの描画処理（レンダリング）を始める。

### ●マルチコネクションによる通信効率の向上

　ところで、これらのデータを本当に順番に1つずつ取得していたのでは非常に時間がかかるだろう。現在のWebサイトでは、1つのWebページを表示するのに必要な要素（HTMLやCSS、画像、スクリプト他）が100とか200以上ということも少なくない。

　そこで一般的なWebブラウザでは、WebサーバとWebブラウザ間で複数のHTTP通信路（TCPコネクション）を開設して、幾つかのデータを同時に取得して通信時間を短縮している。例えばInternet Explorerでは、ネットワークの種類によって、同時接続数を2～8コネクション程度の間で変えている（現在主流のHTTP/1.1では動的にコネクション数の最大値を調整している）。

　同時接続数を多くすればするほどデータの取得は速くなるかもしれないが、その分他のWebブラウザや他の通信アプリケーション、他のユーザーのネットワーク帯域などを圧迫することになるので、増やせばよいというものでもない。

![TCPのマルチコネクションによるデータ取得の高速化](https://image.itmedia.co.jp/ait/articles/1703/29/wi-httpfig02.png) **TCPのマルチコネクションによるデータ取得の高速化**

　なおWebブラウザのレンダリング処理においては、全てのデータがそろわなくても、画像や描画要素のサイズが判明した時点で先にレンダリングを開始することができる。これにより見かけ上の応答性が向上するが、これはHTTPプロトコルとは関係ないので、ここでは触れない。

### ●TCPコネクションのクローズ

　最初に開設したTCPの通信路は、データの取得が完了しても、基本的には開いたままで、次の要求を受け付けるようにする。当初のHTTP規格では、データを1回やりとりするごとにコネクションをクローズしていた。だがTCPコネクションのオープンやクローズ処理は、実はかなり「重い」処理であり、このような使い方はシステムの負荷を無用に高めるため避けるべきことである。

　現在のHTTP規格（HTTP/1.1以降）では、基本的にはコネクションはずっとオープンしたままでデータのやりとりを行う。データのやりとりが終わってもTCPのコネクションをクローズしない。

　昨今は無限スクロールしながらデータを表示するようなWebのアプリケーションが多いため（地図やSNSのタイムライン表示、インタラクティブなコンテンツなど）、この方が望ましいだろう。

　そして明示的にCloseするように指示された場合には（例えばサイトからサインアウトする場合など）、コネクションを終了するようになっている。この場合、同じサイトにもう一度接続すると、別のTCPコネクションが開設されることになる。

### ●ステートレスでシンプルなHTTPプロトコル

　HTTPは基本的にはステートレスな（“状態”を持たない）、シンプルなプロトコルである。Webサーバに対して要求をする前に、通信路をセットアップするなどの準備は不要だし、データを取得した後もクローズ処理などは不要である。それぞれのGET要求はお互い独立しているため、同時並行処理すれば比較的容易に高速化できる。この単純さが、Webの基幹プロトコルとして長く使われている理由の1つと言える。

## HTTPプロトコルの歴史

　HTTPプロトコルでは、サーバからクライアントに対してデータを送る場合はGET／HEADメソッドを使う。逆にサーバ側にデータを送信するにはPOSTメソッドを使うか、GETメソッドのパラメータとして渡す。これ以外にも幾つかメソッドがあるがほとんど使われない。

　HTTPプロトコルは [RFC](https://www.atmarkit.co.jp/icd/root/15/5786915.html) で規格化されており、現在のところは以下のバージョンが存在する。

<table><thead><tr><th>バージョン</th><th>内容</th></tr></thead><tbody><tr><th>HTTP/0.9</th><td>Tim Berners-Leeが開発した、最初のWebシステムで使われていたプロトコル。Webサーバからデータを取得するGETメソッドしかなく、結果の状態を表すステータスコードもなかった。HTTP/1.0を定義したときに、それ以前の規格がHTTP/0.9としてまとめられた</td></tr><tr><th>HTTP/1.0</th><td><a href="https://tools.ietf.org/html/rfc1945">RFC 1945</a> として定義された、最初の実用的なHTTP規格（1996年制定）。ヘッダ情報のみを取得するHEADメソッド、データを送信するPOSTメソッドなどが追加された他、メソッドに対する補助的なパラメータなどを表すヘッダフィールドなどが整備された。またさまざまな種類のデータを表すMIMEタイプもサポートされた</td></tr><tr><th>HTTP/1.1</th><td><a href="https://tools.ietf.org/html/rfc7230">RFC 7230</a> ／ <a href="https://tools.ietf.org/html/rfc7231">7231</a> ／ <a href="https://tools.ietf.org/html/rfc7232">7232</a> ／ <a href="https://tools.ietf.org/html/rfc7233">7233</a> ／ <a href="https://tools.ietf.org/html/rfc7234">7234</a> ／ <a href="https://tools.ietf.org/html/rfc7235">7235</a> で定義された、HTTP/1.0の改訂版（2014年最終改訂）。同一IPで複数のホスト（ホスト名）をサポートするHost:ヘッダフィールドの追加や、PUT／DELETE／OPTIONS／CONNECTなどのメソッドの追加、効率を向上させるパイプライン機能の追加など</td></tr><tr><th>HTTP/2</th><td>大規模、高速ネットワークに向けて最適化されたHTTP/1.1の改良規格（2015年制定）。HTTP/1.1に対して完全な上位互換性を持つ</td></tr></tbody><tfoot><tr><td colspan="2"><strong>HTTPのバージョン</strong></td></tr></tfoot></table>

　現在の主流はHTTP/1.1であるが、今後はより高速／大規模ネットワークに対応したHTTP/2の普及も見込まれている（第3回で説明する）。

## HTTPプロトコルの通信の例

　最後にHTTPプロトコルの通信例を示しておく。TelnetクライアントでWebサーバ（例えばserver1の80番ポート）に接続して「GET / HTTP/1.0」などのコマンド文字列を入力すると、プロトコルの流れを実際に見ることができる。

　通信内容の詳細については次回解説するが、基本的な通信の流れは分かるだろう。GETコマンドを送信すると、それに対する応答がサーバ側から返ってくるだけである。コマンドも応答も全てテキスト形式なので、（通信効率は悪いが）人間には理解しやすい。

**※HTTP/1.1要求の送信例**  
GET / HTTP/1.1 **……サイトのトップページデータの取得**  
HOST: www.microsoft.com **……HTTP/1.1を使う場合はhost:ヘッダを指定すること**  
  
**※HTTP/1.1応答の例**  
HTTP/1.1 200 OK **……1行目はHTTP応答**  
Server: Apache **……2行目以下はHTTP応答ヘッダフィールド**  
ETag: "6082151bd56ea922e1357f5896a90d0a:1425454794"  
Last-Modified: Wed, 04 Mar 2015 07:39:54 GMT  
Accept-Ranges: bytes  
Content-Length: 1020  
Content-Type: text/html  
Date: Mon, 27 Mar 2017 07:07:14 GMT  
Connection: keep-alive  
**……空行（ヘッダと本文の境界）**  
<html> **……以下、HTTP応答本文。HTML形式のテキストが返ってきている**  
<head>  
<title>Microsoft Corporation</title>  
<meta http-equiv="X-UA-Compatible" content="IE=EmulateIE7"></meta>  
<meta http-equiv="Content-Type" content="text/html; charset=utf-8"></meta>  
<meta name="SearchTitle" content="Microsoft.com" scheme=""></meta>  
**……（以下省略）……**

---

　今回はHTTPプロトコルの概要についてまとめた。次回はHTTPプロトコルの詳細について解説する。

[この連載を「連載記事アラート」に登録する](https://atmarkit.itmedia.co.jp/ait/articles/1703/29/) New

Special PR

この記事に関連する製品／サービスを比較（キーマンズネット）

- [まずネットワークの性質を十分に見極めよう！『ネットワーク管理』製品比較](http://www.keyman.or.jp/manage/nwmgt/product?cx_source=kn-pdb201704)
- [信頼性や可用性に対する取り組みは？『ネットワークスイッチ』製品比較](http://www.keyman.or.jp/nwdevice/switch/product?cx_source=kn-pdb201704)
- [構築したいネットワーク要件で大きく変わる『ルーター』の選び方](http://www.keyman.or.jp/nwdevice/router/product?cx_source=kn-pdb201704)
- [既存のネットワーク構成とマッチする？『WAN高速化』製品の選び方](http://www.keyman.or.jp/nwdevice/wansu/product?cx_source=kn-pdb201704)
- [L4負荷分散とL7負荷分散どちらを重視？『ADC／ロードバランサ』製品一覧](http://www.keyman.or.jp/nwdevice/adc/product?cx_source=kn-pdb201704)

[印刷／保存](https://id.itmedia.co.jp/isentry/contents?sc=0c1c43111448b131d65b3b380041de26f2edd6264ee1c371184f54d26ab53365&lc=7d7179c146d0d6af4ebd304ab799a718fe949a8dcd660cd6d12fb97915f9ab0a&return_url=https://ids.itmedia.co.jp/print/ait/articles/1703/29/news045.html&encoding=shift_jis&ac=e8cb9106baa7e37eb9feb877b9f0a27ddaf48b95ba02da49cbb3a8247ee7fec4&cr=e9fd42802bc22856808963077023568339063544b05e5a8646e62c02a898e0fd "この記事を印刷する")

[連載通知](https://id.itmedia.co.jp/isentry/contents?sc=0c1c43111448b131d65b3b380041de26f2edd6264ee1c371184f54d26ab53365&lc=7d7179c146d0d6af4ebd304ab799a718fe949a8dcd660cd6d12fb97915f9ab0a&return_url=https%3A%2F%2Fid.itmedia.co.jp%2Fapp%2Falert%2Fregist_setting%3Furl%3Dhttps%3A%2F%2Fatmarkit.itmedia.co.jp%2Fait%2Farticles%2F1703%2F29%2Fnews045.html%26type%3D2&encoding=shift_jis&ac=8b70865e13a2d61cf96c34172b9312018dffa6d8d496609bf9fd8314fd091e1a&cr=4634196a317325dfc00aa8fe3ea82f36ad62e80b103fcbdaf13b4d39fa261577 "超入門HTTPプロトコル")

スポンサーからのお知らせ PR

Special PR

**本日** **月間**

## 編集部からのお知らせ

[【Amazonギフトカード プレゼント】5月12日（月）開催【無料オンラインセミナー】『＠IT 運用管理セミナー 2025 春 システム運用管理の足腰を鍛える 2025年の壁を突破せよ』＠IT人気連載『IT訴訟 徹底解説』著者 細川義洋氏による【基調講演　裁判所調停委員が語る、運用・保守トラブルのリスクと課題】、テックカンファレンス「SRE NEXT」のFounder 北野勝久氏による【基調講演　SRE本出版からまもなく10年！ これまでに何が起こり、これから何が起こるのか。】などを配信](https://rd.itmedia.co.jp/8hnf#utm_source=ait&utm_content=rightcolumn_info)

あなたにおすすめの記事 PR