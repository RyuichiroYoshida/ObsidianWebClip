---
title: Unit Testを始めよう①~DI・モック・スタブ・Modelのテスト~
source: https://qiita.com/hinakko/items/8a34ef105087d831580b
author:
  - "[[Qiita]]"
published: 2024-09-21
created: 2025-05-06
description: はじめにテストコードを学び始めたばかりの時、テストコードを書くの難しい、何をテストしたら良いかわからないと思っていました。現役のiOSエンジニアの方々にアドバイスをいただきながら学習を進める中でテ…
tags:
  - Tech
  - QA
  - 設計
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

## はじめに

テストコードを学び始めたばかりの時、テストコードを書くの難しい、何をテストしたら良いかわからないと思っていました。現役のiOSエンジニアの方々にアドバイスをいただきながら学習を進める中でテストコードのありがたみが少し理解できたので、 **MVVMの設計で作成したAPI通信のサンプルアプリのコードを用いてUnit Testの始め方** を紹介します。  
GitHub: [https://github.com/hinakkograshi/QiitaViewer](https://github.com/hinakkograshi/QiitaViewer)

## 用語の説明

### Dependency Injection(DI・依存性注入)

依存している部分を外から注入する！「あるオブジェクトが別のオブジェクトに依存している場合、その依存関係を外部から注入することによってオブジェクト間の結合度を低くし、柔軟性を持たせる」  
今回の例では外部に依存しているURLSessionモックオブジェクトを外から注入することで、サーバー環境に関わらずテストを実行することが可能になります。

### モック

ダミー・偽物を作成

### スタブ

特定の処理を差し替え、任意の値を返すようにする

## テストのありがたみ

⚫︎複雑なロジックがちゃんと動くか確かめることができる  
⚫︎モックに置き換えてあげることで、サーバー環境に関わらずテストが可能  
今回の例ではAPI通信は外部への依存するので、サーバー環境が悪いとテストできなくなってしまう。それを防ぐためにモックを作成する。

## 今回のテストの方針

必要な箇所のみをDI  
テストのためにモックは捏造する  
最低限外部に依存する入出力をスタブする

## Modelのテスト

Qiita APIを使用したサンプルコードを例にArticleAPIClientのテストの導入方法を説明します。

### テスト導入前

ArticleAPIClient.swift

```swift
struct ArticleAPIClient {
    func fetchArticles(query: String) async throws -> [Article] {
        let components = Self.createArticleURLComponents(query: query)
        let articles = try await fetchData(components: components, responseType: [Article].self)
        return articles
    }

    private func fetchData<T: Decodable>(components: URLComponents, responseType: T.Type) async throws -> T {
        guard let url = components.url else { throw ArticleAPIClientError.invalidURL  }
        let urlRequest = URLRequest(url: url)
        let urlResponse: URLResponse
        let data: Data
        do {
        // ⭐️URLSession外部に依存(API通信)
            (data, urlResponse) = try await URLSession.shared.data(for: urlRequest)
        } catch {
            throw ArticleAPIClientError.networkError
        }
        guard let httpStatus = urlResponse as? HTTPURLResponse else { throw ArticleAPIClientError.unexpectedResponse}
        switch httpStatus.statusCode {
        case 100 ... 199:
            throw ArticleAPIClientError.networkError
        case 200 ... 299:
            let articleArray = try JSONDecoder().decode(T.self, from: data)
            return articleArray
        case 300 ... 399:
            throw ArticleAPIClientError.networkError
        case 400 ... 499:
            throw ArticleAPIClientError.networkError
        case 500 ... 599:
            throw ArticleAPIClientError.serverError
        default:
            throw ArticleAPIClientError.networkError
        }
    }

    private static func createArticleURLComponents(query: String) -> (URLComponents) {
        var components = URLComponents()
        components.scheme = "https"
        components.host = "qiita.com"
        components.path = "/api/v2/items"
        let queryItems = [
            URLQueryItem(name: "page", value: "1"),
            URLQueryItem(name: "per_page", value: "20"),
            URLQueryItem(name: "query", value: query)
        ]
        components.queryItems = queryItems
        return components
    }
}
```

テストの方針として上記にも書いたように外部に依存しており必要な箇所URLSessionをDIする。  
エラーハンドリングが複雑なので、statusCodeに対して、意図するエラーの種類が投げられているかをテストする。(例えばstatusCodeが500だったらArticleAPIClientError.serverErrorが投げられているか？)

### テスト導入後

**DI(依存性を注入)するためにURLSessionProtocolを作成**

URLSessionProtocol.swift

```swift
protocol URLSessionProtocol {
    func data(for request: URLRequest) async throws -> (Data, Int)
}
```

**テストのためにモック(偽物)MockURLSessionを作成**

MockURLSession.swift

```swift
struct MockURLSession: URLSessionProtocol {
    let statusCode: Int
    let jsonString: String

    func data(for request: URLRequest) async throws -> (Data, Int) {
        let data = jsonString.data(using: .utf8)!
        return (data, statusCode)
    }
}
```

**本物のRealURLSessionを作成**

作成したURLSessionProtocolを使用し、ArticleAPIClientに依存性を注入する。

ArticleAPIClient.swift

```swift
struct ArticleAPIClient {
// ⭐️依存性注入
    let urlSession: any URLSessionProtocol
    // メンバーワイズイニシャライザ
    //init(urlSession: any URLSessionProtocol) {
        //self.urlSession = urlSession
    //}
    func fetchArticles(query: String) async throws -> [Article] {
        let components = Self.createArticleURLComponents(query: query)
        let articles = try await fetchData(components: components, responseType: [Article].self)
        return articles
    }
    
    private func fetchData<T: Decodable>(components: URLComponents, responseType: T.Type) async throws -> T {
        guard let url = components.url else { throw ArticleAPIClientError.invalidURL  }
        let urlRequest = URLRequest(url: url)
        let data: Data
        let statusCode: Int
        do {
            (data, statusCode) = try await urlSession.data(for: urlRequest)
        } catch {
            throw ArticleAPIClientError.networkError
        }    
        switch statusCode {
        case 100 ... 199:
            throw ArticleAPIClientError.networkError
        case 200 ... 299:
            let articleArray = try JSONDecoder().decode(T.self, from: data)
            return articleArray
        case 300 ... 399:
            throw ArticleAPIClientError.networkError
        case 400 ... 499:
            throw ArticleAPIClientError.networkError
        case 500 ... 599:
            throw ArticleAPIClientError.serverError
        default:
            throw ArticleAPIClientError.networkError
        }
    }
    
    private static func createArticleURLComponents(query: String) -> (URLComponents) {
        var components = URLComponents()
        components.scheme = "https"
        components.host = "qiita.com"
        components.path = "/api/v2/items"
        let queryItems = [
            URLQueryItem(name: "page", value: "1"),
            URLQueryItem(name: "per_page", value: "20"),
            URLQueryItem(name: "query", value: query)
        ]
        components.queryItems = queryItems
        return components
    }
}
```

今回はArticleAPIClientを構造体で定義したので `init()` で初期化の処理は不要。  
依存性注入してあげることでArticleAPIClientの呼び出しもとで、本物のRealURLSessionを使うのか・偽物(モック)のMockURLSessionを使うのかを選択することができるようになりました！  
QiitaSearchViewModelでは本物のRealURLSessionを使用することで、DIする前と変わらない挙動になります。

QiitaSearchViewModel.swift

```swift
func fetchAllArticles(query: String) async throws {
        do {
            loadingState = .loading
            articles = try await ArticleAPIClient(urlSession: RealURLSession()).fetchArticles(query: query)
            loadingState = .success
        } catch let error as ArticleAPIClientError {
        ......................
```

準備が整ったのでテストコードを書いていきましょう！  
テストのターゲットが追加されていない場合は、File > New > Target > Unit Testing Bundleから追加しましょう！

MockURLSessionTest.swift

```swift
import XCTest
// テストするターゲットを指定
@testable import QiitaViewer

final class MockURLSessionTest: XCTestCase {

    private let jsonString = """
                [
                {
                    "id": "idTest1",
                    "likes_count": 1,
                    "title": "Test Article",
                    "user": {
                        "name": "hinakko",
                        "profile_image_url": "https://placehold.jp/60x60.png"
                    }
                }
                ]
                """

　　// 正常時 StatusCode: 200
    func test_正常にURLSessionとDecodeが行われて記事が1つ返ってくる() async {
        let mockArticleAPIClient = ArticleAPIClient(urlSession: MockURLSession(statusCode: 200, jsonString: jsonString))
        do {
            let articles = try await mockArticleAPIClient.fetchArticles(query: "Test")
            XCTAssertEqual(articles.count, 1)
        } catch {
            XCTFail("Error: 記事が取得できませんでした")
        }
    }

    // 異常時 StatusCode: 400 networkError
    func test_400番の異常の時にnetworkErrorが返ってくる() async {
        let mockArticleAPIClient = ArticleAPIClient(urlSession: MockURLSession(statusCode: 400, jsonString: jsonString))
        do {
            _ = try await mockArticleAPIClient.fetchArticles(query: "Test")
            XCTFail("異常時であるはずが失敗しなかった")
        } catch let error as ArticleAPIClientError {
            XCTAssertEqual(error, ArticleAPIClientError.networkError)
        } catch {
            XCTFail("例外的なエラーが発生しました: \(error)")
        }
    }

    // 異常時 StatusCode: 500 serverError
    func test_500番の異常の時にserverErrorが返ってくる() async {
        let mockArticleAPIClient = ArticleAPIClient(urlSession: MockURLSession(statusCode: 500, jsonString: jsonString))
        do {
            _ = try await mockArticleAPIClient.fetchArticles(query: "Test")
            XCTFail("異常時であるはずが失敗しなかった")
        } catch let error as ArticleAPIClientError {
            XCTAssertEqual(error, ArticleAPIClientError.serverError)
        } catch {
            XCTFail("例外的なエラーが発生しました: \(error)")
        }
    }
}
```

上記のように `jsonString` はMockURLSessionでデータを用意するのではなく、テストケース側で用意してあげる必要がある。モックの中で暗黙的にfixture（テスト用のデータ）を用意してしまうと、テストケースを読んだときに、どういう状態をテストしているかわかりづらくて、モックの実装を見に行く必要が出てきてしまうから。  
**実装とデータの分離というのを常に意識する**

**今回は以下の3つのケースのテストを行います。**  
わかりやすいようにメソッドを日本語で命名しています。  
**正常時 StatusCode: 200**

```swift
func test_正常にURLSessionとDecodeが行われて記事が1つ返ってくる() async {
        let mockArticleAPIClient = ArticleAPIClient(urlSession: MockURLSession(statusCode: 200, jsonString: jsonString))
        do {
            let articles = try await mockArticleAPIClient.fetchArticles(query: "Test")
            XCTAssertEqual(articles.count, 1)
        } catch {
            XCTFail("Error: 記事が取得できませんでした")
        }
    }
```

StatusCode: 200(成功)時は `XCTAssertEqual` でarticles配列のカウントが1に等しいかテストする。 StatusCode: 200にも関わらずエラーが発生したら `XCTFail` でテストを失敗させる。

**異常時 StatusCode: 400 networkError**

```swift
func test_400番の異常の時にnetworkErrorが返ってくる() async {
        let mockArticleAPIClient = ArticleAPIClient(urlSession: MockURLSession(statusCode: 400, jsonString: jsonString))
        do {
            _ = try await mockArticleAPIClient.fetchArticles(query: "Test")
            XCTFail("異常時であるはずが失敗しなかった")
        } catch let error as ArticleAPIClientError {
            XCTAssertEqual(error, ArticleAPIClientError.networkError)
        } catch {
            XCTFail("例外的なエラーが発生しました: \(error)")
        }
    }
```

StatusCode: 400(クライアントエラー)時はエラーが発生した時 `XCTAssertEqual` でerrorがArticleAPIClientError.networkErrorに等しいかテストする。StatusCode: 400にも関わらずエラーが発生しなかったら `XCTFail` でテストを失敗させる。

**異常時 StatusCode: 500 serverError**

```swift
func test_500番の異常の時にserverErrorが返ってくる() async {
        let mockArticleAPIClient = ArticleAPIClient(urlSession: MockURLSession(statusCode: 500, jsonString: jsonString))
        do {
            _ = try await mockArticleAPIClient.fetchArticles(query: "Test")
            XCTFail("異常時であるはずが失敗しなかった")
        } catch let error as ArticleAPIClientError {
            XCTAssertEqual(error, ArticleAPIClientError.serverError)
        } catch {
            XCTFail("例外的なエラーが発生しました: \(error)")
        }
    }
```

StatusCode: 500(サーバーエラー)時はエラーが発生した時 `XCTAssertEqual` でerrorがArticleAPIClientError.serverErrorに等しいかテストする。StatusCode: 500にも関わらずエラーが発生しなかったら `XCTFail` でテストを失敗させる。

## おわりに

テストコードを書く目的を考えて、テストコードを書いていきたいと思いました。これから色々なテストコードを書いていきたいと思います。今後ViewModelのテストコードの記事も書きたいと思います。  
この記事でアドバイス等ありましたら、コメント・PRいただけると嬉しいです。

## 参考文献

神山.swiftに集まったiOSエンジニアの方々

[0](https://qiita.com/hinakko/items/#comments)

[16](https://qiita.com/hinakko/items/8a34ef105087d831580b/likers)

16