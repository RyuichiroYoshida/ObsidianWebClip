---
title: APIエンドポイントとは？
source: https://www.cloudflare.com/ja-jp/learning/security/api/what-is-api-endpoint/
author: 
published: 
created: 2025-05-06
description: APIエンドポイントはAPI接続の端点で、API呼び出しを受信するところです。APIエンドポイント認証について学びましょう。
tags:
  - 1
  - バックエンド
  - 通信
read: false
---
Preview Mode

[Documentation](https://staging.mrk.cfdata.org/mrk/redwood-blade-repository/)

APIエンドポイントはAPI接続の端点で、API呼び出しを受信するところです。

記事のリンクをコピーする

## APIエンドポイントとは？

[アプリケーションプログラミングインターフェイス（API）](https://www.cloudflare.com/learning/security/api/what-is-an-api/) は、アプリケーションが別のアプリケーションからのサービスをリクエストする手段です。APIがあれば、開発者はアプリケーションの既存機能を再構築する必要がありません。APIエンドポイントは、リクエスト（ [API呼び出し](https://www.cloudflare.com/learning/security/api/what-is-api-call/) ）が実行される場所です。

アリスとボブが電話で会話しているとします。アリスの言葉がボブに伝わり、ボブの言葉がアリスに伝わります。アリスは自分の言葉を、会話の「エンドポイント」であるボブに向けます。

*アリス：「もしもし、ボブ？」 ----------> ボブ*

API統合も会話のようなものです。APIクライアントがAPIサーバーに対して、「もしもし」の代わりに「あるデータが必要なんだけど」というようなことを言います（API呼び出し）。すると、APIサーバーのエンドポイントが「これがそのデータだよ、どうぞ」と答えます（API応答）。APIエンドポイントは、アリスやボブのような物理的主体ではありません。ハードウェアではなく、ソフトウェアの中に存在しています。

#### APIサーバーとAPIクライアント

APIは、単一または複数のサーバー（データを保存し、ソフトウェアプログラムを実行する専用のコンピューター）でホストされています。各サーバーは、データ、コンテンツ、ソフトウェアの機能を、インターネットを介して他のデバイスへ「提供」します。APIエンドポイントは通常、サーバーでホストされています。

API接続のもう一方の端がAPIクライアント、つまりAPIからのサービスをリクエストする主体です。API呼び出しはたいてい自動化されていますが、APIクライアントをAPI「ユーザー」と呼ぶ人もいます。

## APIクライアントはサーバーのエンドポイントをどうやって知るのか

APIを使うにはドキュメントが必要です。ドキュメントには、APIで受け付けるリクエストの種類、APIで提供可能なもの、応答のフォーマット、エンドポイントなどが記されています。開発者は、APIドキュメントのレビューを行い、アプリケーションを構築する際にその情報を組み込むことができます。

例として、CloudflareのAPIドキュメント（エンドポイントを含む）をこちらでご覧ください。 [https://api.cloudflare.com/](https://api.cloudflare.com/)

## APIはURLをどう使うか

ユニフォームリソースロケーター（URL）はWeb上で、Webページ見つけるなど複数の目的で使われます。たとえば、このWebページのアメリカ英語版のURLはhttps://www.cloudflare.com/learning/security/api/what-is-api-endpoint/です。ユーザーがそのURLをブラウザーに入力すると、ブラウザーはこのWebページがどこにあるかを知り、読み込むことができます。

URLにはAPIエンドポイントも表示されています。アリスとボブが電話で話す時、アリスはボブの電話番号を使ってボブに電話します。APIエンドポイントのURLは、APIを呼び出すための電話番号のようなものです。

APIサーバーは、単一または複数のAPIエンドポイントをホストすることができます。つまり、それらのエンドポイントURLに向けられた呼び出しを、サーバーが受け付けて処理するのです。APIクライアントにもURLが必要です。APIサーバーが返事をどこに送ればいいかを知るためです。ボブとアリスが電話で話すのに双方の電話番号が必要なのと同じです。開発者はアプリケーションを構築する際にこのURLを設定します。

URLには必ず、そこへ到達するための [アプリケーション層](https://www.cloudflare.com/learning/ddos/what-is-layer-7/) プロトコル（ [HTTP](https://www.cloudflare.com/learning/ddos/glossary/hypertext-transfer-protocol-http/) など）が含まれます。Web APIはたいていHTTPを使っていますので、APIエンドポイントのURLにhttpが含まれています。

## APIエンドポイントとAPIクライアントの認証方式は？

適正に設計されたAPIは、呼び出し元を問わずAPI呼び出しを受け付けることはしません。そんなことをすれば、攻撃者からAPIサーバーへ悪意のあるデータが送られかねません。また、APIの利用にはコストがかかるのが通常ですので、APIサーバーは、料金を支払う顧客からのAPI呼び出しかどうかをチェックしなければなりません。

これらの理由から、APIサーバーは、呼び出し元のAPIクライアントが既知で信頼できることを確認しなければならないのです。この確認は [認証](https://www.cloudflare.com/learning/access-management/what-is-authentication/) によって行われます。

認証はアイデンティティを検証するプロセスです。システムに対して人間のユーザーを認証する際に [いくつかの方法](https://www.cloudflare.com/learning/access-management/what-is-multi-factor-authentication/) があるように、APIエンドポイントの認証実行にも主に4つのやり方があります。

1. **APIキー** ： APIクライアントに [キー](https://www.cloudflare.com/learning/ssl/what-is-a-cryptographic-key/) （当該クライアントとAPIサービスだけが知る固有の文字列）が割り当てられます。APIクライアントがサーバーエンドポイントにAPI呼び出しを送信する際にこのキーを使い、サーバーが呼び出し元を識別するという仕組みです。
2. **Basic認証（ユーザー名とパスワード）** ：キーを使うやり方と同様に、APIクライアントがAPIサービスにユーザー名とパスワードを設定しておき、API呼び出しの際にそれらの資格情報を入力します。
3. **OAuthトークン** ：APIサーバーは、クライアントの認証を求める代わりに、 [OAuth](https://www.cloudflare.com/learning/access-management/what-is-oauth/) プロトコルを用いて、信頼できる認証サーバーから認証トークンを取得することもできます。
4. **相互TLS** ： [TLS](https://www.cloudflare.com/learning/ssl/transport-layer-security-tls/) は、Webページ読み込みの際にクライアントとサーバーの間に認証済み接続を作成するプロトコルです。API統合の両側の認証にも使えます。

多くの場合、 [相互TLS](https://www.cloudflare.com/learning/access-management/what-is-mutual-tls/) が最も効果的な認証方式です。理由の一つは、クライアントだけでなくエンドポイントとクライアントの双方向認証であるため、お互いが正当なソースからデータを受け取っていることを確認できる点です。また、エンドポイント間で決して共有されないプライベートキーを使用するため、トランジット中に傍受されることもありません。APIキー、パスワード、トークンはすべて、複製されたり盗まれたりする可能性があります。

Cloudflare API Shieldは、相互TLSでAPIエンドポイントとクライアントの認証を行うため、双方を攻撃から保護することができます。 [API Shieldは、その他のAPIセキュリティ機能（](https://www.cloudflare.com/learning/security/api/what-is-api-security/) [レート制限](https://www.cloudflare.com/learning/bots/what-is-rate-limiting/) 、 [データ損失防止（DLP）](https://www.cloudflare.com/learning/access-management/what-is-dlp/) [など）もいくつか提供しています。API Shieldの詳細はこちら。](https://www.cloudflare.com/products/api-gateway/)