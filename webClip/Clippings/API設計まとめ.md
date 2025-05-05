---
title: API設計まとめ
source: https://qiita.com/KNR109/items/d3b6aa8803c62238d990
author:
  - "[[Qiita]]"
published: 2024-02-24
created: 2025-05-06
description: はじめに自分は2021年に新卒でWeb系の開発会社にフロントエンジニアとして入社し2022年で2年目になります。実務ではReact×TypeScriptを利用したフロント周りとNode.js(N…
tags:
  - バックエンド
  - Tech
  - バックエンド設計
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から1年以上が経過しています。

## はじめに

自分は2021年に新卒でWeb系の開発会社にフロントエンジニアとして入社し2022年で2年目になります。

実務ではReact×TypeScriptを利用したフロント周りとNode.js(Nest)やRailsを用いたバックエンド(API)の開発をしています。

その中で使っていたAPI設計について改めて学び直したのでまとめて行きます。

## この記事の対象者

- エンジニア初心者から中級者
- APIについて学びを深めたい人

## この記事の目標

- APIについて学ぶ
- 我流ではなく正しいAPI設計について学ぶ

## この記事でやらないこと

具体的にコードを用いたAPI設計の書き方の説明に関しては下記の記事で解説をしています。

## APIについて

### APIとは

APIは"Application Programming Interface"の略で、直訳すると「 **アプリケーションを使プログラミングを使ってつなぐ** 」という意味になります。

### Web APIとは

Web APIとは、 [Web API The Good Parts](https://www.amazon.co.jp/dp/4873116864/ref=as_sl_pc_as_ss_li_til?tag=sutabaopen-22&linkCode=w00&linkId=ebac746dfbed5ed11486209d48a1d8f3&creativeASIN=4873116864) において

> Web APIとはHTTPプロトコルを利用してネットワーク越しに呼び出すAPIです。  
> APIとは"Application Programming" Interfaceの略で、ソフトウェアコンポーネントと外部インターフェース、つまり機能はわかっているけども、その中身の動作はわからない(知らなくてよい)機能のカタマリを、外部から呼び出すための仕様のことを指します。

> 簡単にいうと、URIにアクセスすることで、サーバー側の情報を書き換えたり、サーバー側に置かれた情報を取得できるWebシステムです。

と説明されています。

Web APIを利用することで、Webサービス(Twitterやホットペッパー等)で提供している機能やデータを外からプログラムが読み取りやすい形で利用することができます。

具体的に [ホットペッパーが公開しているAPI](https://webservice.recruit.co.jp/doc/hotpepper/reference.html) を利用することで、飲食店のデータ取得を簡単に行うことができます。

## Webの仕組み

超簡単にまとめると下記のようになっています。

1. ユーザー(クライアント)はWebに対してHTTPリクエストを送る
2. リクエストを受けたWebサーバーはデータを返す

[![server.001.jpeg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2695521/e74c27e2-30fc-63e3-5846-d18dc9fe751f.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F2695521%2Fe74c27e2-30fc-63e3-5846-d18dc9fe751f.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=256caa394607131ce2898eed7e4ba119)

### HTTPリクエストについて

具体的にWebブラウザからWebサーバーに送っているHTTPリクエストは、

- リクエストライン
- ヘッダー
- ボディー

の3要素で構成されています。

#### リクエストライン

HTTPリクエストの中身の一つであるリクエストラインは下記のようになっています。

```text
GET /index.html HTTP/1.1
```

リクエストラインでは「 **何を、どうしたい** 」かが書かれており、構成は下記のようになっています。

- メソッド
- リクエストURI
- HTTPバージョン

メソッドは具体的にGET(データ取得)、POST(データ登録)、PUT(データ更新)、DELETE(データ削除)等があります。

#### リクエストヘッダー

リクエストヘッダーは、リクエストについての情報や属性(メタデータ)について記載されています。

#### リクエストボディ

実際にリクエストで送るデータが入ってくる。

### レスポンス

HTTPリクエストを送ったときにレスポンスは下記の構成になっています

- ステータスライン
- ヘッダー
- ボディー

#### ステータスライン

ステータスラインは下記の構成になっている。

- HTTPバージョン
- ステータスコード
- フレーズ

```text
HTTP/1.1 200 OK
```

よく見るステータスコードをまとめると下記のようになっています。

- 2xx: リクエストが成功した
- 4xx: リクエストに誤りがある
- 5xx: サーバー側のエラー

より詳しいステータスコードについての解説は下記の記事を参考にしてみてください。

#### ヘッダー

レスポンスヘッダーはメタ情報が含まれています。

```text
alt-svc: quic=":443"; ma=2592000; v="44,43,39,35"
cache-control: private, max-age=0
content-encoding: br
content-type: text/html; charset=UTF-8
date: Thu, 23 Aug 2018 15:37:28 GMT
expires: -1
server: gws
status: 200
x-frame-options: SAMEORIGIN
x-xss-protection: 1; mode=block
```

#### ボディー

具体的なサーバー側からの応答データが返却される。

## RESTについて

RESTとは、下記のように説明されています。

> RESTとは、分散システムにおいて複数のソフトウェアを連携させるのに適した設計原則の一つ。2000年にロイ・フィールディング（Roy Fielding）氏が提唱した。狭義には、それをWebシステムに適用したソフトウェアの設計様式を指し、一般にはこの意味で用いられることがほとんどである。

[IT用語辞典](https://e-words.jp/w/REST.html)

簡単にいうと、RESTとは「設計ルール」を意味します。

### RESTful

RESTfulとはRESTで求められている原則に従っていることを指します。

RESTful APIは、RESTの原則に基づいて設計されているAPIのことを表します。

### REST原則

RESTの原則は下記の6つがあります

- クライアント・サーバー制約
- ステートレス
- キャッシュ(クライアントサイド)
- 統一インターフェース
- レイヤーシステム
- コードオンデマンド

#### クライアント・サーバー制約

クライアントとサーバーは依存することなく独立し、お互い影響なく改良ができるような構造にするべきという考え方です。

つまり、クライアント(画面UI)とサーバー(データ)で関心を分離する。

[![スクリーンショット 2022-10-29 10.08.26.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2695521/875a8e8f-18f3-1328-817c-f631bdb97563.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F2695521%2F875a8e8f-18f3-1328-817c-f631bdb97563.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=8b729ab4710c894a2a0839e16611a368)

#### ステートレス

前の状態は保持せず、それぞれのリクエストが独立して成立していることを意味します。

つまり、

- サーバーに保持されているコンテキスト情報は使わない(サーバーセッションは使わない)
- 状態はクライアント上に保存される(リクエストに全て含める)

#### キャッシュ(クライアントサイド)

クライアントはレスポンスをキャッシュすることができる。

適切なキャッシュを行うことで、クライアントとサーバー間の通信を減らしユーザー体験の向上及び、リソースの向上を見込むことができる。

#### 統一インターフェース

同じリソースに対するAPIアクセスは、全て同じ形式で統一する(一意なURIでアクセス可能)。

またHTTPプロトコルのメソッド(GET,POST,PUT,DELETE等)を用いてのみ、データ操作を可能にする。

URIへの操作(GETやPOST)を統一することで、全体構成をシンプルにすることができる。

#### レイヤーシステム

複数の役割をもつサーバーを組み合わせることでAPIを実現するアーキテクチャーのことをレイヤーシステムと呼びます。

具体的には、リクエストを受けるレイヤー、応答を返すレイヤー、セキュリティー処理を行うレイヤー等に分けます。

レイヤーシステムは、各々の役割を独立させることで、再利用生を上げることができます。

#### コードオンデマンド

クライアントコードをダウンロードして実行することができます。

コードオンデマンドを組み込むことで、リリース済みのクライアントに対してサーバー主導で機能追加をすることができ、クライアント側に処理の実装を寄せるので、サーバーへの負荷を下げることができる。

RESTの原則をまとめます。

- クライアント・サーバー制約
- ステートレス
- キャッシュ(クライアントサイド)
- 統一インターフェース
- レイヤーシステム
- コードオンデマンド

## Web API設計

ここからは今回の本題であるWeb APIの設計について学んでいきます。

### URI設計

- 短くて入力がしやすい
- 省略しない
- 大文字と小文字が混ざってない(基本は小文字)
- 単語はハイフン繋ぎ
- 単語は複数形を使う
- エンコードが必要な文字を使わない
- 更新がしやすい
- ルールが統一されている

一つずつみて行きます

#### 短くて入力がしやすい

無駄な情報(重複情報)を省いて簡潔なURIにする。

**【BAD】**

```text
https://exmaple.com/service/api/search
```

**【GOOD】**

```text
https://exmaple.com/search
```

#### 省略しない

**【BAD】**

```text
https://exmaple.com/ja/u
```

**【GOOD】**

```text
https://exmaple.com/users
```

#### 大文字と小文字が混ざってない(基本は小文字)

基本的には小文字で統一する。大文字と小文字が混ざっていると間違えやすくなる。

**【BAD】**

```text
https://exmaple.com/Posts
```

**【GOOD】**

```text
https://exmaple.com/posts
```

#### 単語はハイフン繋ぎ

単語をつなぐ場合はハイフンで繋ぐようにする。

**【BAD】**

```text
https://exmaple.com/favorite_contents
```

**【GOOD】**

```text
https://exmaple.com/favorite-contents
```

※ そもそも単語を連結する必要がある場合はURI設計から見直す場合もある

```text
https://exmaple.com/favorite/contents
```

#### 単語は複数形を使う

**【BAD】**

```text
https://exmaple.com/post/1
```

**【GOOD】**

```text
https://exmaple.com/posts/1
```

#### エンコードが必要な文字を使わない

エンコードがあると可読性が下がり、URIから処理が推測できない

**【BAD】**

```text
https://exmaple.com/%TENFLOJFL$JSKF3352nb%scsd%
```

#### 更新がしやすい

**【BAD】**

```text
https://exmaple.com/alpha/contents/1
https://exmaple.com/beta/contents/1
```

**【GOOD】**

```text
https://exmaple.com/contents/1
```

#### ルールが統一されている

パラメタをパスパラメタで渡すのか、クエリパラメタで渡すのかなどルールを統一化させる。

```text
https://exmaple.com/beta/contents/1 # パスパラメタ
https://exmaple.com/alpha/contents/?id=1 # クエリパラメタ
```

URI設計をまとめると

- 短くて入力がしやすい
- 省略しない
- 大文字と小文字が混ざってない(基本は小文字)
- 単語はハイフン繋ぎ
- 単語は複数形を使う
- エンコードが必要な文字を使わない
- 更新がしやすい
- ルールが統一されている

### HTTPメソッド

先ほど紹介したURIはリソースを表し、HTTPメソッドはリソースに対しての操作を示します。

具体的に下記の場合は、 `/api/v1/posts/1` というリソースに対してデータを取得(GET)するようにリクエストを送っています。

```text
GET /api/v1/posts/1
```

HTTPメソッドで主要なものは下記になっています。

| メソッド名 | 処理内容 |
| --- | --- |
| GET | リソースの取得 |
| POST | リソースの新規登録 |
| PUT | 既存リソースの更新 |
| DLETE | リソースの削除 |

同じURIに対してメソッドを変えることで操作も変えることができます。

| 操作 | API |
| --- | --- |
| 記事一覧の取得 | GETメソッド/ http:///example.com/api/v1/posts |
| 記事単体の取得 | GETメソッド/ http:///example.com/api/v1/posts/1 |
| 記事の新規作成 | POSTメソッド/ http:///example.com/api/v1/posts |
| 記事の更新 | PUTメソッド/ http:///example.com/api/v1/posts/1 |
| 記事の削除 | DELETEメソッド/ http:///example.com/api/v1/posts/1 |

### クエリパラメーターとパスパラメーター

| 種類 | API |
| --- | --- |
| クエリパラメター | http:///example.com/api/v1/posts?id=1 |
| パスパラメター | http:///example.com/api/v1/posts/1 |

クエリパラメーターとパスパラメーターの使い分け

- 一意なリソースを表す場合は **パスパラメター** を利用
- 省略しても良い場合は **クエリパラメター**

具体的には検索する際のリクエストは一意にパスが定まらないのでクエリパラメタを利用する。

```text
http:///example.com/api/v1/users?name="suzuki"
```

### ステータスコード

ステータスコードによって、処理結果の概要を知ることができる。

具体的にエラーが出た際もスターテスコードを追うことでエラーの特定を行うことができます。

ステータスコードについてはこちらの記事を参考にしてみてください。

### レスポンスデータのフォーマット

主要なレスポンスデータのフォーマットは下記があります。

- XML
- JSON

#### XML

XMLの特徴は下記の通りです。

- テキスト形式
- タグで記述されている
- タグは入れ子
- タグに属性が付与されてる

**【XML形式】**

#### JSON

JSONの特徴は下記の通りです。

- テキスト形式
- JavaScriptを元にしたフォーマット
- XMLと比較するとデータ量が削減できる
- オブジェクトは入れ子にできる

**【JSON形式】**

```text
{
   user: {"id" : "1", "name" : "tanaka"}
  }
```

なおデータフォーマットを指定する際はリクエストヘッダーに記載することがお作法的に良しとされています。

【リクエストヘッダー】

```text
GET http://example.com/users
Host: exmaple.com
Acceot: application/json # JSON形式と指定した場合
```

### データ内部構造の設計

- エンベロープを使わない
- ネスト構造を減らす
- プロパティの命名規則を統一する
- 日付指定

#### エンベロープを使わない

JSONを返却するAPIの共通部部分であるエンベロープは使わないようにする。

下記のレスポンスに含まれている"header"部分に関してはリクエストヘッダーで受け渡される記載と共通しているので、不要。(情報が被っている)

**【エンベロープがある場合(BAD)】**

**【エンベロープを省いた場合場合(GOOD)】**

```json
{
    "books": [
        {
            "id": 1,
            "name": "はじめてのJavaScript",
            "price": 3000
        },
        {
            "id": 2,
            "name": "はじめてのAPI入門",
            "price": 1500
        }
    ]
}
```

#### ネスト構造を減らす

ネストがあるとレスポンス容量が増えてしまうので、無駄なネストを減らすようにレスポンスを設計する。

**【不要なネストがある場合(BAD)】**

```json
{
         "id": 2,
         "name": "はじめてのAPI入門",
         "info" : { 
             "author": "tanaka",
             "price": 1500
          }
     }
```

**【不要なネストを省いた場合(GOOD)】**

```json
{
         "id": 2,
         "name": "はじめてのAPI入門",
         "author": "tanaka",
         "price": 1500
     }
```

#### プロパティの命名規則を統一する

基本的にはスネークケースかキャメルケースを採用する。

| 名前 | 例 |
| --- | --- |
| スネークケース | snake\_case |
| キャメルケース | camelCase |
| パスカルケース | PascalCase |

#### 日付指定

日付はRFC3339(W3C-DTF)形式を使うようにする

```text
2022-10-31T18:00:00+09:00
```

### エラー

エラー詳細はレスポンスボディに追記する。

```json
{
  "code": "12345",
  "message": "認証エラーです"
}
```

またエラーの際はHTML形式でレスポンスをしないようにする。

フォーマットによってはクライアントサイドで処理ができない可能性もあるため。

## その他の設計

### 認証

#### JSON Web Token

JSON Web Token(JWT)は"ジョット"とよび、JSON形式のデータで署名による改ざんチェックを行います。

JWTを利用することで、認証結果をサーバー側で保持せずにクライアントで保持することができ、ステートレスな通信を行うことができます。

JWTの構成は下記のようになっています。

- ヘッダー
- ペイロード
- 署名

**【JWTのサンプル】**

```text
eyJhbGcGKOWJIUzI1NiI&3Hf5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjMPqfwibmFtZSIfksjDTW3nwoAOCWQ0IjoxNTE2MjM5MDIyfQ.
t42p4AHef69Tyyi880842bDTTOADINW93ncs7Zffs
```

.で結合されており、実際は下記の構成

```text
[ヘッダ].
[ペイロード].
[署名]
```

ヘッダーでは具体的に、署名で利用するアルゴリズムが定義されている。

**【ヘッダー】**

```js
{
  "alg": "HS256",
  "typ": "JWT"
}
```

ペイロードは保存したいデータの実態が入っている。

- subは同一Issuer内での識別子(ユーザーID等)
- iatはjwtが発行された日時を表す

**【ペイロード】**

```js
{
  "sub": "1234567890",
  "iat": 1516239022
}
```

署名はデータが改ざんされていないかを確認する。

#### Authorizationヘッダー

認証で利用するヘッダーとしてAuthorizationヘッダーがあります。

先ほど紹介したJWTを利用する場合にAuthorizationヘッダーにトークンを格納しリクエストを送ります。

下記のような形式で記述をします。

```text
Authorization: [type] [credentials]
```

`[type]` におる種類は下記です。

| 名前 | 説明 |
| --- | --- |
| Basic | ベーシック認証でIDとパスワードを平文で送る |
| Bearer | OAuth2.0でJWTはこれを使う |
| Digest | ダイジェスト認証でIDとパスワードをハッシュ化して送る |
| OAuth | OAuth1.0 |

また `[credentials]` には具体的にな認証情報を入れます。

有名な認証方法Auth0ではJWTトークンを用いてリクエストヘッダーに下記の情報を含めることで、サーバー側とデータのやり取りをすることができます。

```text
Authorization: Bearer eyxxxxxxxxx
```

### セキュリティー

下記の脆弱性対策について説明していきます。

- XSS
- CSRF
- HTTP
- JWT

#### XSS

XSSは悪意あるユーザーが正規のサイトに不正なスクリプトを埋め込み、正規ユーザーか情報を不正に抜き出すことができる。

対策としてはレスポンスヘッダーに下記を追加する

- `X-XSS-Protectio` n: 1でXSSフィルタリングが有効化
- `X-Frame-Options`: DENYでframeタグの呼び出しを拒否する
- `X-Content-Type-Options`: nosniffでIEの脆弱性を対応する

#### CSRF

本来拒否すべきアクセス元からのリクエストを処理してしまう。

対策としては下記がある

- アクセス許可していないリクエストを拒否する
	- X-API-Key
	- Authentication
- 悪意のある攻撃者が推測しずらいトークンを発行し照合処理行う
	- X-CSRF-TOKEN

#### HTTP

HTTPは通信経路が暗号化されていないので盗聴されやすい。

対策方法としてはHTTPSを利用する。

#### JWT

先ほど紹介したJWTはクライアント側で編集ができるので、サーバー側での検証が不十分だった場合、改ざんがされた情報を受け入れてしまう可能性がある。

対策方法としてはヘッダーのアルゴリズム(alg)にnoneを指定しない。

## 最後に

いかがだったでしょうか。今回はAPI設計についてまとめました。

他にもいろいろな記事を書いているのでぜひ読んでいただけたら嬉しいです。

## 参考文献

- [Web API: The Good Parts](https://www.amazon.co.jp/dp/4873116864/ref=as_sl_pc_as_ss_li_til?tag=sutabaopen-22&linkCode=w00&linkId=926b7446a25d760a1f3918a4ad49843e&creativeASIN=4873116864)
- [Webを支える技術](https://www.amazon.co.jp/dp/4774142042/ref=as_sl_pc_as_ss_li_til?tag=sutabaopen-22&linkCode=w00&linkId=c2a3cbb927c6aecd8f43b2f09f81edcd&creativeASIN=4774142042)

[4](https://qiita.com/KNR109/items/#comments)

[2031](https://qiita.com/KNR109/items/d3b6aa8803c62238d990/likers)

2141