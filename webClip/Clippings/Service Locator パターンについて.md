---
title: Service Locator パターンについて
source: https://qiita.com/tassi-yuzukko/items/a81a7b9aaa42198df689
author:
  - "[[Qiita]]"
published: 2019-05-29
created: 2025-05-06
description: 会社にて「Service Locator パターン」という単語を聞いて「？」となったので、戒めも含めて調べてみたメモ。いきなり結論結論から言うと、Service Locator パターンも De…
tags:
  - 1
  - 設計
read:
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から5年以上が経過しています。

会社にて「Service Locator パターン」という単語を聞いて「？」となったので、戒めも含めて調べてみたメモ。

## いきなり結論

結論から言うと、Service Locator パターンも Dependency Injection (いわゆる DI )と同じようにクラス間の密結合度を緩和するためのものと考えてよさそう。

というか、 [**こちら**](http://www.nuits.jp/entry/servicelocator-vs-dependencyinjection) に超参考になる記述があるので、これを備忘録として残しておく。

## 概要

DDDとかの Repository パターンで DI を使用することがあるが、それと同じような目的で使用できる。

## なんちゃってサンプルコード

ここでは、ドメイン層がインフラ層に依存しないように Repository パターンを使用する場合の、 DI 版と Service Locator 版をそれぞれ考えてみる。

### DI 使用したときの Repository パターン

[![DIによるRepositoryパターン.png](https://qiita-image-store.s3.amazonaws.com/0/132272/79f794af-2bc7-52fb-eace-71042b031171.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F132272%2F79f794af-2bc7-52fb-eace-71042b031171.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=327b7d8b42433a24593722e646a5c461)

アプリケーション層

```c#
// アプリケーション層は、インフラ層とドメイン層の両方に依存
namespace ApplicationLayer
{
    using DomainLayer;
    using InfrastractureLayer;

    class BookSearchService
    {
        public string[] SearchByTitle(string title)
        {
            // リポジトリの実体を生成
            var bookRepositoryImpl = new BookRepositoryImpl();

            // ドメインオブジェクトを生成
            var bookLibrary = new BookLibrary(bookRepositoryImpl);

            // 検索結果取得
            return bookLibrary.Search(title);
        }
    }
}
```

ドメイン層

```c#
// ドメイン層はどの層にも依存しない
namespace DomainLayer
{
    // Repositoryのインターフェイス
    public interface IBookRepository
    {
        string[] GetBooksByTitle(string title);
    }

    public class BookLibrary
    {
        IBookRepository bookRepository;

        // DIによりリポジトリの実装を注入してもらう
        public BookLibrary(IBookRepository bookRepository)
        {
            this.bookRepository = bookRepository;
        }

        public string[] Search(string title)
        {
            // ありえないロジックになってるけど、サンプルなので・・・
            return bookRepository.GetBooksByTitle(title);
        }
    }
}
```

インフラ層

```c#
// インフラ層はドメイン層のみに依存する
namespace InfrastractureLayer
{
    using DomainLayer;

    // ドメイン層で定義されているRepositoryインターフェイスの実装
    public class BookRepositoryImpl : IBookRepository
    {
        public string[] GetBooksByTitle(string title)
        {
            // 処理内容省略。例えばSQLとかが記述される。
            string[] books;

            ...

            return books;
        }
    }
}
```

### Service Locator を使用したときの Repository パターン

[![ServiceLocatorによるRepositoryパターン (1).png](https://qiita-image-store.s3.amazonaws.com/0/132272/cbf84657-8dce-08b9-8693-7c409f27f355.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F132272%2Fcbf84657-8dce-08b9-8693-7c409f27f355.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=27148b0bbdba35a48c1f47eba7edb842)

↑めんどくさくなったので、 Repository の基底インターフェイスおよび基底クラスを省略。  
たぶんクラス図見てもてんやわんやなので、ソース見たほうが早い。  
要は、依存関係をリスト化して別クラス（ ServiceLocator ）で保持している。

アプリケーション層

ドメイン層（DIの場合にIRepositoryを追加しているだけ）

```c#
// ドメイン層はどこにも依存しない
namespace DomainLayerSL
{
    // リポジトリの基底インターフェイス
    public interface IRepository { }

    // 本のリポジトリ
    public interface IBookRepository : IRepository
    {
        string[] GetBooksByTitle(string title);
    }

    public class BookLibrary
    {
        IBookRepository bookRepository;

        // DIによりリポジトリの実装を注入してもらう
        public BookLibrary(IBookRepository bookRepository)
        {
            this.bookRepository = bookRepository;
        }

        public string[] Search(string title)
        {
            // ありえないロジックになってるけど、サンプルなので・・・
            return bookRepository.GetBooksByTitle(title);
        }
    }
}
```

インフラ層（DIの場合にRepositoryImplを追加しているだけ）

```c#
// インフラ層はドメイン層のみに依存する
namespace InfrastractureLayerSL
{
    using DomainLayerSL;

    // リポジトリ実装の基底クラス
    public abstract class RepositoryImpl : IRepository { }

    // ドメイン層で定義されている本のリポジトリインターフェイスの実装
    public class BookRepositoryImpl : RepositoryImpl
    {
        public string[] GetBooksByTitle(string title)
        {
            // 処理内容省略。例えばSQLとかが記述される。
            string[] books;

            ...

            return books;
        }
    }
}
```

## 個人的な解釈

個人的には Service Locator パターンを以下のように理解していいかなと思っている。

### 目的

- DI と同じで、クラス間の依存関係を緩和するために用いる

### DI と異なる点

- Service Locator パターンでは、インターフェイスと実装の対をコレクションとして参照を保存しておき、必要なときに生成して取り出す

### DI と比べて優れる点

- 使用方法にもよるかもしれないが、グローバルにアクセスすることになるので、再利用性が高い

### DI と比べて劣る点

- 使用方法によるかもしれないが、グローバルにアクセスするので、依存関係が見えにくい
- DI の場合は依存注入するインターフェイスにのみ依存するが、Service Locator パターンでは Service Locator のコレクションそのものに依存するため、依存度が高い（でも Service Locator に依存してもそんなに問題ないか？）
- たぶんテストで Mock 用意する時には DI のほうが直感的でやりやすい

## まとめ

最初の結論にも書いたけど、目的としては DI を用いるのと同じで、クラス間を疎結合にするために使用すると考えてよさそう。  
今思いつく有効的な使い方としては、 Repository が複数あり、それを Repository の Service Locator としてコレクションにしておき、使い回せるようにするとか？まあ悪くはないか・・・  
とりあえず「 Service Locator パターンが～」という会話で「？」となるのは嫌なので、備忘録として残す。

## 参考にさせてもらったサイト

- [Service LocatorとDependency InjectionパターンとDI Container](http://www.nuits.jp/entry/servicelocator-vs-dependencyinjection)
	- こちらには、さらに DI コンテナについても記載があります。 Service Locator パターンや DI パターンを調べている方は、絶対にこっちを見たほうが良いです。

## 補足

調べてみると、 Service Locator パターンはアンチパターンとして挙げられている場合が結構あるみたい。  
似たような概念で DI コンテナがあるので、時間があったら今度はそっちをまとめてみたいと思う。

[3](https://qiita.com/tassi-yuzukko/items/#comments)

[110](https://qiita.com/tassi-yuzukko/items/a81a7b9aaa42198df689/likers)

93