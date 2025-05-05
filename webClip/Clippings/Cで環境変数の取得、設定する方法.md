---
title: C#で環境変数の取得、設定する方法
source: https://developers-trash.com/archives/1080#google_vignette
author:
  - "[[開発者のごみ箱]]"
published: 2022-09-08
created: 2025-05-06
description: 実行環境.Net6.0Visual Studio Version 17.3.0 Preview 5.0プログラム内での環境変数の取得と設定方法以下で環境変数の取得と設定ができます。public static void Main(string...
tags:
  - 1
  - CSharp
  - セキュリティ
read:
---
本ブログはアフィリエイト広告を利用しています。

## 実行環境

- .Net6.0
- Visual Studio Version 17.3.0 Preview 5.0

## プログラム内での環境変数の取得と設定方法

以下で環境変数の取得と設定ができます。

```
public static void Main(string[] args)
{
    // 環境変数一覧を取得
    var envs = Environment.GetEnvironmentVariables();
    foreach (DictionaryEntry env in envs)
    {
        Console.WriteLine($"{env.Key} - {env.Value}");
    }

    // 特定の環境変数を取得
    var proxy = Environment.GetEnvironmentVariable("http_proxy");
    Console.WriteLine(proxy == null);   // 未設定のためnull

    // 環境変数を設定
    Environment.SetEnvironmentVariable("http_proxy", "1.2.3.4:8080");
    proxy = Environment.GetEnvironmentVariable("http_proxy");
    Console.WriteLine(proxy);           // 1.2.3.4:8080が取得できる
}
```

## Visual Studioでデバッグ実行する時に環境変数を設定する

実行時にのみ適用したい場合はデバッグプロパティに環境変数を設定することも可能です。

![debug_property](https://i0.wp.com/developers-trash.com/wp-content/uploads/2022/09/debug_property1.png?resize=293%2C81&ssl=1)

debug\_property

![debug_property](https://i0.wp.com/developers-trash.com/wp-content/uploads/2022/09/debug_property2.png?resize=784%2C513&ssl=1)

debug\_property

以上です。")

![](https://developers-trash.com/wp-content/uploads/2022/09/bug_gokiburi-120x68.png)

[

ゴキブリにべチバーは効くのか(5月から9月まで効果を試した)

](https://developers-trash.com/archives/1070 "ゴキブリにべチバーは効くのか(5月から9月まで効果を試した)")[![](https://developers-trash.com/wp-content/uploads/2022/09/searchtool-120x68.png)

Aterm Wi-Fiルータ(Aterm WG1900HP2)のIPアドレスがわからない時に調べる方法

](https://developers-trash.com/archives/1090 "Aterm Wi-Fiルータ(Aterm WG1900HP2)のIPアドレスがわからない時に調べる方法")

[ホーム](https://developers-trash.com/)

- [ホーム](https://developers-trash.com/)
- 
- [トップ](https://developers-trash.com/archives/#)
-

タイトルとURLをコピーしました