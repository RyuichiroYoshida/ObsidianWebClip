---
title: 【C＋＋】C＋＋ REST SDKを使用したhttp通信してみた
source: https://logicalbeat.jp/blog/6231/
author:
  - "[[株式会社ロジカルビート]]"
published: 
created: 2025-05-06
description: こんにちは！情熱開発部の金澤です！ 年も明けて寒さが厳しくなってきましたが、皆様いかがお過ごしでしょ [...]<p><a class="btn btn-secondary understrap-re
tags:
  - 1
  - "#Cpp"
  - 通信
  - バックエンド
read: false
---
こんにちは！情熱開発部の金澤です！

年も明けて寒さが厳しくなってきましたが、皆様いかがお過ごしでしょうか。

私は冬だからか今回初めての技術ブログだからなのか分かりませんが震えています。  
というわけで今回は、『C++ REST SDKを使用したhttp通信』についてご紹介していきたいと思います！

## C++ REST SDKとは

C++ REST SDKとは「ネイティブ コードをクラウド環境に移行できるようにするためにマイクロソフトが踏み出した最初の 1 歩」とMicrosoftが提供しているオープンソースのライブラリです。

JSON対応や非同期通信はもちろん、Microsoftが提供していることもあり何よりインストールが非常に簡単で、VisualStudioからであればNuGetにて直ぐにインストールすることが出来ます。

また、C++との一貫性が高いのも魅力的ですね。

## http通信してみた

それでは早速cpprestsdkでhttp通信をしてみようと思います。

テストには [httpbin](https://httpbin.org/) というサイトを使用させていただきます。

こちらのサイトもhttp通信のテストにはもってこいのサイトになっており、シンプルなHTTPリクエストとレスポンスを返してくれるサービスです。

それではサンプルプロジェクトを作ってみましょう。

![](https://logicalbeat.jp/wp01/wp-content/uploads/2020/08/animal_chara_computer_penguin-300x278.png)

今回参考にさせて頂いたサイトは以下になります。

[Httpクライアントチュートリアル](https://github.com/microsoft/cpprestsdk/wiki/Getting-Started-Tutorial)

[C++でHTTP(S)でGET/POSTする（cpprestsdk の使い方）](https://kagasu.hatenablog.com/entry/2017/10/07/190551)

[C++ REST SDK (Casablanca) でSlack Web APIを触ってみた](http://yamada-shi.hatenablog.com/entry/2017/12/22/000000)

まずはGETのソースがこちら。

```c++
#include <cpprest/http_client.h>
#include <cpprest/filestream.h>

using namespace utility;                    // 文字列変換などの一般的なユーティリティ
using namespace web;                        // URIのような共通の機能
using namespace web::http;                  // 共通のHTTP機能
using namespace web::http::client;          // HTTP クライアントの機能
using namespace concurrency::streams;       // 非同期ストリーム

pplx::task<void> GetTest()
{
    return pplx::create_task([] 
    {
        // クライアントの設定
        http_client client(L"https://httpbin.org/get");

        // リクエスト送信
        return client.request(methods::GET);

    })
        
    .then(    {
        // ステータスコード判定
        if (response.status_code() == status_codes::OK)
        {
            // レスポンスボディを表示
            auto body = response.extract_string();
            std::wcout << body.get().c_str() << std::endl;
        }
    });
}

int main(void)
{
    try
    {
        GetTest().wait();
    }
    catch (const std::exception& e)
    {
        printf("Error exception:%s\n", e.what());
    }
    return 0;
}
```

cpprestsdkのTaskクラスとhttp\_clientクラスを使用してGETリクエストを送信しています。

Taskクラスでリクエストを送信し、タスクが完了するとthen関数が呼び出され、returnされたレスポンスがhttp\_responseクラスに入るという仕組みになっています。

main関数では呼び出す際には例外がスローされた時のことを考えてTryCatchを使用して待機させます。

あとはこれを実行し”args”や”headers”等の情報が返ってくれば成功です！

次にPOSTのソースはこちら

```c++
#include <cpprest/http_client.h>
#include <cpprest/filestream.h>

using namespace utility;                    // 文字列変換などの一般的なユーティリティ
using namespace web;                        // URIのような共通の機能
using namespace web::http;                  // 共通のHTTP機能
using namespace web::http::client;          // HTTP クライアントの機能
using namespace concurrency::streams;       // 非同期ストリーム

pplx::task<void> PostTest()
{
    return pplx::create_task([] 
    {
        // クライアントの設定
        http_client client(L"https://httpbin.org/post");
        
        // 送信データの作成
        json::value postData;
        postData[L"message"] = json::value::string(L"Hello http");

        // リクエスト送信
        return client.request(methods::POST, L"", postData.serialize(), L"application/json");
    })
    
    .then(    {
        // ステータスコード判定
        if (response.status_code() == status_codes::OK)
        {
            // レスポンスボディを表示
            auto body = response.extract_string();
            std::wcout << body.get().c_str() << std::endl;
        }
    });
}

int main(void)
{
    try
    {
        PostTest().wait();
    }
    catch (const std::exception& e)
    {
        printf("Error exception:%s\n", e.what());
    }
    return 0;
}
```

今回は送信するデータにJSONを使用することにしました。

JSONデータ”message”として”Hello http”という文字列を送信します。

大まかな流れはGETの時と変わりませんね！

タスクでrequestを送信し、then関数で受け取る流れになっています。

ただし、requestの中身が少し変わります。

GETの際はmethodの指定だけで良かったのですが、POSTの場合は送信するデータとデータのMIMEタイプも必要になってきます。

今回は空白にして使用していませんが、クライアントURLにクエリ等を追加することもできるようです。

こちらを実行するとGETの時の様に”args”や”headers”等の情報が返ってきますが、その中の”json”に送信した”message”：”Hello http”があれば成功です！

## 最後に

実は、今回このブログを書くまではあまりhttp通信について馴染みがありませんでした。通信というだけで難しいものだとも勝手に思っていました。ですが実際に調べてコードを書いてみると、存外とっつきやすく感じました。やはり実際に触れてみないと分からないことが沢山ありますね。これを機に何か一つアプリを作ってみるのも良いかもしれません！  
それでは、最後までご清覧いただきありがとうございました。

---

##### 【免責事項】

本サイトでの情報を利用することによる損害等に対し、  
株式会社ロジカルビートは一切の責任を負いません。

[一覧へ戻る](https://logicalbeat.jp/blog)