---
title: VContainer入門(1) - IContainerBuilderとIObjectResolver
source: https://qiita.com/sakano/items/3a009019e279024fda19
author:
  - "[[Qiita]]"
published: 2021-06-05
created: 2025-05-06
description: この記事についてUnity用のDIコンテナであるVCointanerの使い方を解説します。DIコンテナを使うと複雑な依存関係を楽に管理できるようになります。DI(DependencyInject…
tags:
  - 1
  - Tools
  - Unity
  - 設計
read:
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から3年以上が経過しています。

## この記事について

Unity用のDIコンテナである [VCointaner](https://github.com/hadashiA/VContainer) の使い方を解説します。DIコンテナを使うと複雑な依存関係を楽に管理できるようになります。

DI(DependencyInjection)そのものについて詳しい説明はここではしません。

目次ページ: [VContainer入門](https://qiita.com/sakano/items/b91e01f7fc0a946090ac)

## VContainerの導入

VContainerはUPM(UnityPackageManager)を通してプロジェクトにインポートできます。

まずはUnityで新しいプロジェクトを作成し、プロジェクトのPackagesフォルダにある `manifest.json` を開きます。  
そしてdependencies内に以下の設定を追加します。

manifest.json

```json
"jp.hadashikick.vcontainer": "https://github.com/hadashiA/VContainer.git?path=VContainer/Assets/VContainer#1.8.0"
```

これでVContainerが使えるようになりました。

## VCointanerを試してみる

## 動作確認用のクラスを作成

まずはサンプルとして使うための適当な依存関係があるクラスを作っておきます。

Mock.cs

```csharp
using UnityEngine;
using VContainer;

// 先頭に[Logger]を付けてログ出力するクラス
public sealed class Logger
{
    public void Log(string message) => Debug.Log("[Logger] " + message);
}

// 足し算するだけのクラス
public sealed class Calculator
{
    public int Add(int a, int b) => a + b;
}

// LoggerとCalculatorに依存するクラス
public sealed class HogeClass
{
    private readonly Logger logger;
    private readonly Calculator calculator;

    [Inject]
    public HogeClass(Logger logger, Calculator calculator)
    {
        this.logger = logger;
        this.calculator = calculator;
    }

    public void LoggerTest()
    {
        logger.Log("LoggerTest");
    }

    public void CalculatorTest(int a, int b)
    {
        int result = calculator.Add(a, b);
        logger.Log($"{a} + {b} = {result}");
    }
}
```

`HogeClass` はコンストラクタで `Logger` と `Calculator` を受け取ります。 `[Inject]` 属性が付いているのはVCointanerがこのコンストラクタを使えるようにするためです。 [^1]

VContainerを利用しない場合、 `HogeClass` は以下のように使えます。

```csharp
var logger = new Logger();
var calculator = new Calculator();

var hoge = new HogeClass(logger, calculator);
hoge.LoggerTest(); // [Logger] LoggerTest と出力される
hoge.CalculatorTest(3, 5); // [Logger] 3 + 5 = 8 と出力される
```

VContainerはHogeClassにLoggerとCalculatorを渡して生成する部分を肩代わりしてくれます。次の項から実際に使ってみます。

## VContinerを使う流れ

VCointanerは以下の流れで使えます。

1. `IContainerBuilder` を生成する。
2. `IContainerBuilder` に使いたいクラスを登録する。
3. `IContainerBuilder` から `IObjectResolver` を生成する。
4. `IObjectResolver` を通して使いたいクラスを生成する。

実際にVContainerを使う際は `LifetimeScope` を使うことが多いです。 `LifetimeScope` は `IContainerBuilder` と `IObjectResolver` をいい感じに管理してくれるクラスです。

この `LifetimeScope` を通して使う場合でも内部の動作を知っていた方がわかりやすいので、まずは `IContainerBuilder` と `IObjectResolver` を直接使ってみます。

実際のコードは以下になります。このコンポーネントを適当なGameObjectにアタッチして実行すると動作確認できます。

コード中の1、2、3、4をそれぞれ説明していきます。

### 1\. IContainerBuilderを生成

`IContainerBuilder` の実体として `ContainerBuilder` クラスを使います。普通にnewで生成します。

### 2\. IContainerBuilderに使いたいクラスを登録 (Register)

`IContainerBuilder.Register` を使ってクラスを登録できます。型引数に登録したいクラス、引数にLifetimeを渡します。

Lifetimeは生成されたオブジェクトの生存期間を指定するものです。 [あとで説明](https://qiita.com/sakano/items/#lifetime%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6) するのでとりあえずLifetime.Singletonを渡しておいてください。

### 3\. IContainerBuilderからIObjectResolverを生成 (Build)

`IContainerBuilder.Build` で `IObjectResolver` を生成できます。 `IObjectResolver` はDisposeが必要なのでusingで囲っています。

### 4\. IObjectResolverで使いたいクラスを生成 (Resolve)

`IObjectResolver.Resolve` でBuildする前に登録しておいたクラスが生成できます。ここではHogeClassを生成しています。型引数に生成したいクラスを指定すればLoggerクラスやCalculatorクラスも生成できます。

## Register/Build/Resolveの内部動作

VContainerの内部動作も知っていた方が便利なので簡単に説明しておきます。

### Register

Registerした時点では登録されたクラスを `ContainerBuilder` の内部に保存しているだけです。

### Build

`ContainerBuilder` をBuildすると `IObjectResolver` が生成されます。  
この段階で、Registerされたクラス全てをリフレクションで解析して生成に必要な情報を集めます。 [^2]

HogeClassの場合はInject属性がついたコンストラクタを発見し、引数にLoggerとCalculatorが必要なことが記録されます。  
LoggerとCalculatorはInject属性がついたメンバがないのでただ生成すればいいことが記録されます。

### Resolve

Build時に記録した情報に基づいて指定されたクラスのインスタンスを返します。

ここで\_\_Resolve\_\_という単語は「依存関係を解決して生成されたオブジェクトを取り出すこと」を表します。

HogeClassの場合はLoggerとCalculatorが必要なので、まずはLoggerとCalculatorをResolveします。それからこのLoggerとCalculatorを使ってHogeClassのコンストラクタを呼び出してHogeClass自体を生成します。  
要するに `objectResolver.Resolve<HogeClass>()` の中では `new HogeClass(new Logger(), new Calculator());`と同様のことが実行されています。

依存先のクラス(ここではLoggerやCalculator)も再帰的にResolveされていくのが重要です。  
例えば、LoggerがコンストラクタでFileWriterというクラスを受け取らなければならなくなったとします。この場合でもFileWriterがRegisterされていれば自動的にFileWriterがResolveされてLoggerのコンストラクタに渡され、そのLoggerがまたHogeClassに渡されます。依存関係が何段階になっても同様にResolveされていきます。 [^3]

## Lifetimeについて

あとで説明すると言っていたLifetimeについて説明します。Lifetimeを変えるとResolve時の動作が少し変わります。

Lifetimeには `Lifetime.Singleton` ・ `Lifetime.Transient` ・ `Lifetime.Scoped` の3つがあります。

### Lifetime.Singleton

`Lifetime.Singleton` ではResolveで生成されたインスタンスがIObjectResolver内でキャッシュされます。つまり複数回Resolveを呼んでも同じインスタンスが返ってきます。

```csharp
using (IObjectResolver objectResolver = containerBuilder.Build())
{
    // hogeとhoge2は同じインスタンスになる
    HogeClass hoge = objectResolver.Resolve<HogeClass>();
    HogeClass hoge2 = objectResolver.Resolve<HogeClass>();
}
```

IObjectResolverごとにキャッシュされるためIObjectResolverが異なると別のインスタンスになります。  
次の例では2回BuildしてIObjectResolverを2つ生成しています。

```csharp
using (IObjectResolver objectResolver = containerBuilder.Build())
using (IObjectResolver objectResolver2 = containerBuilder.Build())
{
    // hogeとhoge2は違うインスタンスになる
    HogeClass hoge = objectResolver.Resolve<HogeClass>();
    HogeClass hoge2 = objectResolver2.Resolve<HogeClass>();
}
```

### Lifetime.Transient

`Lifetime.Transient` ではResolveするたびに別のインスタンスが生成されます。

HogeClassのRegisterで `Lifetime.Transient` を渡すように変えたのでhogeとhoge2は別のインスタンスになります。

LoggerとCalculatorは `Lifetime.Singleton` のままなので、hogeとhoge2のコンストラクタに渡されるLoggerとCalculatorはそれぞれ同じインスタンスになります。これも `Lifetime.Transient` にすれば別のインスタンスが渡されることになります。

### Lifetime.Scoped

`Lifetime.Scoped` は `Lifetime.Singleton` に似ていますがIObjectResolverが親子関係を持ったときの動作が異なります。親子関係と一緒に説明するのでここでは省略します。

### IDisposableの自動Dispose

IDisposableを実装したクラスをResolveするとLifetimeによっては自動的にDisposeされます。

- `Lifetime.Singleton` か `Lifetime.Scoped` の場合、IObjectResolverがDisposeされるとResolveで生成されたインスタンスも一緒にDisposeされます。
- `Lifetime.Transient` の場合はDisposeされません。IObjectResolverは作りっぱなしなのでDisposeする責任はインスタンスを渡された側になります。

## まとめ

Register -> Build -> Resolveの順に使うイメージを持っておいてください。

[次回](https://qiita.com/sakano/items/e38306a8204c531a40c1) はVContainerの本来の使い方であるLifetimeScopeを通した方法を説明します。

[1](https://qiita.com/sakano/items/#comments)

[40](https://qiita.com/sakano/items/3a009019e279024fda19/likers)

18

[^1]: コンストラクタが複数あるときにVContainerがどのコンストラクタを使うか識別するのに使われます。IL2CPPでストリッピングされないようにする効果もあります。省略しても動きますが必ず付ける方が安全です。

[^2]: VContainerのコード生成が有効になっているとBuildとResolveのリフレクションが省略されて実行速度が上がります。コード生成についてはいずれ説明します。

[^3]: 依存関係が循環すると例外が発生します。