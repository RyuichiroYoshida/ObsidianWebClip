---
title: C#で簡単なWebAPIサービスを作成する
source: https://qiita.com/YoshijiGates/items/093387ee22e0df60404e
author:
  - "[[Qiita]]"
published: 2023-02-20
created: 2025-05-06
description: 1. はじめにOpenAPIver3に対応したAPIを開発したいC# ASP.NETCore WebAPIを使用したいSwaggerで動作検証をしたい2. 開発環境Visual Studi…
tags:
  - CSharp
  - Tech
  - アプリ
  - バックエンド
  - 通信
  - 学習系
read:
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から1年以上が経過しています。

## 1\. はじめに

- OpenAPIver3に対応したAPIを開発したい
- C# ASP.NETCore WebAPIを使用したい
- Swaggerで動作検証をしたい

## 2\. 開発環境

- Visual Studio 2022
- .NET6 (ASP.NET Core)
- Windows11

## 3\. Visual Studioで新規プロジェクトを作成する

- WebAPIを呼び出すと本の情報を5件分取得できる簡単なサンプルを作成する
- 「ASP.NET Core WebAPI」 を選択する
- ASP.NETサービスがインストールされていない場合は別途Visual Struio インストーラーからインストールする  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2852081/8641aebf-c416-1d87-001b-5913abced664.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F2852081%2F8641aebf-c416-1d87-001b-5913abced664.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=d3e2f4dffe001de29b39d2a2e530b468)
- ソリューション名/プロジェクト名は「WebApiServer」を設定する（任意）  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2852081/9496bf44-3846-5550-1d7e-7bd8df60cbbc.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F2852081%2F9496bf44-3846-5550-1d7e-7bd8df60cbbc.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=18d54bce3e447ac348db8135a66a4af5)
- フレームワークは「.NET 6.0」を選択する  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2852081/e2b4016e-61d2-d1f5-0660-93a153f300d3.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F2852081%2Fe2b4016e-61d2-d1f5-0660-93a153f300d3.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a2235e3bf7554d0797695b18f5022c86)

## 4\. コントローラーを追加する

- 新しい項目を追加で、「APIコントローラー -空」 を追加する  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2852081/99688b6d-8a9c-0d54-7328-1ca5fdc9202b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F2852081%2F99688b6d-8a9c-0d54-7328-1ca5fdc9202b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=43e78878b67d3f4c55c6f21704e12d9d)

BooksController.cs

```csharp
using Microsoft.AspNetCore.Mvc;

namespace WebAPI.Controllers
{
    [Produces("application/json")]
    [Route("api/Books")]
    public class BooksController : ControllerBase
    {
        private List<Models.Book> _lst;
        public BooksController() { 
            _lst = new List<Models.Book>();
            _lst.Add(new Models.Book { id = 1, title = "title1", price = 100, page = 10 });
            _lst.Add(new Models.Book { id = 2, title = "title2", price = 200, page = 20 });
            _lst.Add(new Models.Book { id = 3, title = "title3", price = 300, page = 30 });
            _lst.Add(new Models.Book { id = 4, title = "title4", price = 400, page = 40 });
            _lst.Add(new Models.Book { id = 5, title = "title5", price = 500, page = 50 });
        }

        // GET: api/Books
        [HttpGet]
        public IEnumerable<Models.Book> Get()
        {
            return _lst;
        }

        // GET: api/Books/5
        [HttpGet("{id}", Name = "Get")]
        public Models.Book Get(int id)
        {
            return _lst.FirstOrDefault(x =>x.id == id);
        }

        // POST: api/Books
        [HttpPost]
        public string Post([FromBody] Models.Book value)
        {
            return string.Format("post title:{0}", value.title);
        }

        // Put: api/Books/5
        [HttpPut("{id}")]
        public void Put(int id, [FromBody] string value)
        {
            // do nothing
        }

        // DELETE: api/ApiWithActions/5
        [HttpDelete("{id}")]
        public void Delete(int id)
        {
            // do nothing
        }

    }
}
```

## 5\. モデルを追加する

- 新しい項目を追加で、「クラス」 を追加する

Book.cs

```csharp
namespace Models
{
    public class Book
    {
        public int id { get; set; }
        public string title { get; set; }
        public int price { get; set; }
        public int page { get; set; }
    }
}
```

## 6\. サンプルのWebAPIを起動する

- 実行ボタンをクリックすると、Webブラウザ(Swagger)が立ち上がる  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2852081/045df923-b53b-c2a8-e6a0-730e1742c36e.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F2852081%2F045df923-b53b-c2a8-e6a0-730e1742c36e.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=582027c39ec4f52fc1c832e674873a51)
- GETメソッドの「Try it」>「Execute」ボタンをクリックすると、WebAPIが実行できる  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2852081/2521479d-4152-a458-c4c5-46e1821596db.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F2852081%2F2521479d-4152-a458-c4c5-46e1821596db.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=b55ef4219526da5f736d2ac7248cc03c)
- Responseに、WebAPIの呼び出し結果が表示される  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2852081/dae6a85c-1ff0-479f-2ed2-44fbbf553b18.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F2852081%2Fdae6a85c-1ff0-479f-2ed2-44fbbf553b18.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=7aed4a81a9c38221f7c1d28ea34cd57e)

## 7\. ソースコード

## 8\. 参考文献

[0](https://qiita.com/YoshijiGates/items/#comments)

[9](https://qiita.com/YoshijiGates/items/093387ee22e0df60404e/likers)

8