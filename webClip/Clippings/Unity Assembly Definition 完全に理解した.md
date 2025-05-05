---
title: Unity Assembly Definition 完全に理解した
source: https://qiita.com/toRisouP/items/d206af3029c7d80326ed
author:
  - "[[Qiita]]"
published: 2019-12-20
created: 2025-05-06
description: はじめにこの記事におけるソースコードはすべて Public Domain です。また本記事において利用しているUnityバージョンは2019.2です。Assembly Definitionとは…
tags:
  - CSharp
  - Tech
  - Unity
  - 設計
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から5年以上が経過しています。

## はじめに

この記事におけるソースコードはすべて `Public Domain` です。  
また本記事において利用しているUnityバージョンは `2019.2` です。

## Assembly Definitionとは何か

[Assembly Definition](https://docs.unity3d.com/Manual/ScriptCompilationAssemblyDefinitionFiles.html) という機能をご存知でしょうか。  
これは「C#のビルドファイル（アセンブリ）を分割して出力する」ことができる機能であり、Unity 2017.3で追加されました。

[![WhatIsADF.gif](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F47146%2F91b6419a-138e-da2b-30ba-907f48f6d566.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=64a11021d536a2dddd2914aec8c1daea)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F47146%2F91b6419a-138e-da2b-30ba-907f48f6d566.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=64a11021d536a2dddd2914aec8c1daea)

特に **ライブラリやC#アセットを公開することがある人は絶対に覚えておくべき** 機能です。  
「 `Assembly Definition` よくわからん」という状態でもプログラムは一応作成できますが、プロならば絶対に覚えておくべき機能です。

なお、 `Assembly Definition` 自体はUnityの機能の名前です。  
実際にこの機能を有効化するためにユーザが定義するファイルのことを `Assembly Definition Files` と呼びます。  
（長いので略して `adf` と呼びます）。

## Assembly Definitionを利用した場合のUnityの挙動

`Assembly Definition` を利用した場合、アセンブリが分割されてコンパイル・ビルドされることになります。  
そのため `Assembly Definition` を利用するときと利用しないときとでは、コーディングの工程が大きく変化します。

そのため `Assembly Definition` を利用する場合はその挙動をしっかり把握する必要があります。

## 1\. csprojが分解される

`adf` を定義した場合、その定義ごとに別れて `csproj` ファイルが生成されることになります。

このとき、 `adf` を定義していない従来のアセンブリはすべて `Assembly-CSharp` に収められます。

[![csproj.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47146/58101e77-5234-fcbf-bc39-f695066b951f.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F47146%2F58101e77-5234-fcbf-bc39-f695066b951f.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=d320e5c8b6450ac711fbc208fcd97a36)

## 2\. 差分ビルドされる

ソースコードに修正があった場合、該当するアセンブリのみがコンパイルされます。  
そのため全体でコンパイル時間が短くなります。

## 3\. 参照関係を明示的に設定する必要がある

`Assembly Definition` を有効化するとアセンブリが分割されます。  
そのため「どのアセンブリが、他のどのアセンブリに依存しているか」を手動で設定する必要があります。

[![ReferenceGif.gif](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F47146%2F110e5ec9-b043-8252-2687-71a863ed5a99.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=67abc800a3e33933d1d8736bbdf67040)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F47146%2F110e5ec9-b043-8252-2687-71a863ed5a99.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=67abc800a3e33933d1d8736bbdf67040)

## 4\. Assembly-CSharpの扱いが特殊になる

`Assembly-CSharp` は `adf` を **定義していないスクリプト** がすべて押し込められる、いわゆる「全部入り」のアセンブリです。最初はすべてのスクリプトがこのアセンブリに入った状態となります。  
つまり、 `adf` を定義するということは、「 `Assembly-CSharp` アセンブリからモジュールを抜き出して分割していく」と同義になります。

さて、この `Assembly-CSharp` ですが、参照関係が特殊となっています。

- 他の `adf` で分割されたアセンブリへは、 **`Assembly-CSharp` からすべて参照可能**
- 逆に他のアセンブリから、 **`Assembly-CSharp` へは絶対にアクセスできない**

[![AssemblyCSharp.gif](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F47146%2F5d435eff-8943-ddbb-f3fc-7826d13f0302.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a100ac43f4b0e955ec352de30aa9f65f)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F47146%2F5d435eff-8943-ddbb-f3fc-7826d13f0302.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a100ac43f4b0e955ec352de30aa9f65f)

`Assembly Definition` を利用しているときはこの性質が問題となることがあるため、この挙動はしっかりと把握しておきましょう。

## Assembly Definition Filesの定義方法

## Assembly Definition Filesの作成

`adf` は `Project View` において、定義したいディレクトリ上で右クリックメニューから作成することができます。

[![ModuleA.gif](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F47146%2F25986699-dae7-4a8d-eedf-096821b8a1b9.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=4d17e4db1309aa937e75adc16afd7313)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F47146%2F25986699-dae7-4a8d-eedf-096821b8a1b9.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=4d17e4db1309aa937e75adc16afd7313)  
`adf` が配置された時点で **そのディレクトリ以下** の `csproj` ファイルが分割され、アセンブリも別に生成されます。

[![Solution.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47146/a9638402-11a1-e99f-5099-6d88efe5a9db.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F47146%2Fa9638402-11a1-e99f-5099-6d88efe5a9db.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=71689c0201eda94432915272c8525977)  
Banana.csprojが作成された。

[![ScriptAssemblies.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47146/f4852d75-5ee0-e840-f573-e3bec37ff344.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F47146%2Ff4852d75-5ee0-e840-f573-e3bec37ff344.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=80073469b45f766c42761c2e072f71fe)  
`/Library/ScriptAssemblies` を覗くと `Banana.dll` に分割されている。

## Assembly Definition Filesの設定

単に `dll` に分割するだけなら、 `adf` を作成するだけで完了です。

より突っ込んだ設定、たとえば次の設定が行いたい場合は `Inspector View` にて設定することができます。

- このアセンブリ内でのみ `unsafe` コードの利用を許可したい
- 他の `adf` によって定義されたアセンブリに依存したい
- 特定のプラットフォームでのみ有効化したい
- Editor拡張/Testコードだと設定したい

[![ADFSetting.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47146/3352bc71-e2f1-575c-e580-a3b43912d472.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F47146%2F3352bc71-e2f1-575c-e580-a3b43912d472.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=2bab63a5d6d9659989b488845650223a)

#### General

- **Allow 'unsafe' Code**: このアセンブリ内でunsafeコードを許可するか
- **Auto Referenced**: 本体側のアセンブリ(`Assembly-CSharp`)から参照するか
- **Override References**: すでにコンパイル済みの `dll` を参照に追加するか

この「 **Auto Referenced** 」にチェックが入っている場合、毎回コンパイル対象になります。  
`Assembly-CSharp` から直接参照しないものはオフにしておくとよいです。

#### Define Constraints

文字列を設定します。ここに設定された文字列が `Player Settings` の `Scripting Define Symbols` に定義されていた場合のみビルドされます。

[![RIPENED.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47146/7e328c30-244c-a875-540b-5367a8a1c6ab.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F47146%2F7e328c30-244c-a875-540b-5367a8a1c6ab.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=0ccf26bd57b5f517c997127019e8a0e5)  
（ `RIPENED_BANANA` が定義されていた場合のみこのアセンブリは有効となる。ちなみに意味は"完熟バナナ"）

#### Assembly Definition References

他の `adf` によって定義されたアセンブリを参照する場合に設定します。  
**超重要** 。

[![Reference.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47146/c43c504b-ef5c-3430-fdab-506c5e91766b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F47146%2Fc43c504b-ef5c-3430-fdab-506c5e91766b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=70b6a208aae0fa2e48d328641513fb8c)

たとえば `Juice Mixer` は `Banana` と `Milk` に依存している場合は、 `Assembly Definition References` でそれぞれに参照を追加しなければいけません。

#### Platforms

どのプラットフォームで有効化するか設定します。  
`/Editor` に `adf` を定義した場合は `Editor` にチェックを入れる必要があります。  
（ `Editor Test` の場合も同様）

[![EditorBuild.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47146/60079e12-ef65-3818-3890-ed5e00a54a97.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F47146%2F60079e12-ef65-3818-3890-ed5e00a54a97.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a93a59030ab0c043a23f3f68102cd337)

（エディタ拡張などは `Editor` にチェックをいれてアセンブリを別ける必要がある）

## Assembly Definition Filesのディレクトリ構造

`adf` はディレクトリをネストして定義しても問題はありません。  
その場合も「ディレクトリ単位」でアセンブリが分割されビルドされます。

[![directories.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47146/6c2e6794-949f-866a-81b9-07dc2c0cf0e8.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F47146%2F6c2e6794-949f-866a-81b9-07dc2c0cf0e8.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=4402fe5b4194579785a8aef323bc662a)  
[![dlls.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47146/c2e5425b-78b1-d7bc-0c4b-d1eccd6d1018.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F47146%2Fc2e5425b-78b1-d7bc-0c4b-d1eccd6d1018.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a39f7582cb85a38d655e0f79437f4f1d)

## Assembly Definitionを利用するメリット・デメリット

`Assembly Definition` は基本的には定義した方がメリットは大きいのですが、一応デメリットも存在します。  
メリット・デメリットを把握した上で実際に利用するかどうかを決めるとよいでしょう。

#### メリット

- **差分コンパイルになるためEditor上でのコンパイル時間が短くなる** (`Auto Referenced` を適切にOFFにしておく必要あり)
- **`internal` や `private protected` などのアクセスレベルが活用できる**
- **`unsafe` コードを有効化する範囲をアセンブリ単位で限定できる**
- **モジュールの依存関係を強制できる**
- **ライブラリ同士の干渉を最小限に抑えてプロジェクトに追加できる**
- **対象プラットフォームごとにアセンブリレベルで別けてビルドができる**

#### デメリット

- `Asset Bundle` と併用した場合に挙動がややこしいことになる
- アセンブリをまたいで `partial` クラスが定義できない
- `static` を使っていたときに分割しにくい
- たまに謎のコンパイルエラーが出て動かなくなる

## Assembly Definitionを利用するべきシチュエーション

**次のようなシチュエーションでは必ずAssembly Definitionを利用するべきです。**

### ライブラリやアセットを公開する場合

ライブラリや、スクリプトを含んだアセットを公開する場合は `adf` を定義してアセンブリを分割するべきです。  
**むしろやってくださいおねがいします。**

理由としては「 `adf` が定義されていないライブラリは `Assembly-CSharp` アセンブリに入ってしまう」からです。  
これは自身のプロジェクトが `Assembly Definition` を活用していた場合に非常に面倒な問題になります。

場合を分けて説明します。

---

##### セーフ：Assembly Definitionを一切利用していない場合

[![Library1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47146/30820740-0f0b-5e7b-9ce1-c4734e92d343.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F47146%2F30820740-0f0b-5e7b-9ce1-c4734e92d343.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=ff5313200ed3293a4faa7cc7c97e39bb)

自身のプロジェクトも、導入した外部ライブラリも、どれも `adf` を定義されていない場合です。  
この場合は問題ありません。

---

##### セーフ：全員がAssembly Definitionを利用している場合

[![Library3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47146/16e13bb9-7076-777b-06ab-c7302920b2e8.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F47146%2F16e13bb9-7076-777b-06ab-c7302920b2e8.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=03090943f33df7e9e0b3364ad71fec01)

自身のプロジェクトと導入した外部ライブラリ、その両方が `adf` を定義していた場合です。  
この場合はアセンブリ同士の参照を正しく設定すれば問題ありません。

---

##### アウト：自身のプロジェクトのみadfを定義していた場合

[![Library2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47146/5380429d-4f4d-7e72-7eab-825185f3fc30.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F47146%2F5380429d-4f4d-7e72-7eab-825185f3fc30.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=9edda50eabcf3ef9ade7581e7f1441dd)

**この場合はアウトです。**  
自身のプロジェクトは `adf` を定義しているが、導入したライブラリには定義されていない場合。  
この場合はライブラリが `Assembly-CSharp` に入り込んでしまうため、自身のモジュールからアクセスができないためエラーになってしまいます。

こうなると自身でライブラリに `adf` を定義してアセンブリを分割する必要がでてきます。  
そしてライブラリ側の構造が歪で `adf` をキレイに定義できない、定義しても `static` や `partial` を乱用していてエラーが出まくる…、みたいな **地獄** な状況になる場合もあります。

なのでもし、これからライブラリやモジュールを作って公開するつもりがある人は **必ずAssembly Definitionを意識した作りにしてください。**  
マジでお願いします。マジで。

---

### アーキテクチャに沿った開発を行う場合

クリーンアーキテクチャなど、レイヤ間での依存関係をしっかり定義したい場合は `Assembly Definition` がかなり有用です。  
`adf` でモジュール間の依存を定義することで、「 `Domain` から外への参照を禁止する」といったことがプロジェクト設定で制御することが可能になります。

## 合わせて覚えておくべき技

`Assembly Definition` を利用する場合、覚えておくと開発効率の向上につながるテクニックがいくつかあります。

## Editor上でコンパイルエラーが出た場合の対策

`Assembly Definition` を使っていると、コードに変更がなくてもコンパイルエラーが出てしまうことがあります。 **本当によく起きます。**

これはエディタがキャッシュしているアセンブリがおかしくなってしまったことが原因の場合が多いです。  
そのため次の手順を行うと復旧することがあります。

1. `\Library\ScriptAssemblies` を削除してみる
2. （↑で直らないなら）Unity Editorを再起動する
3. （↑で直らないなら）該当するAssetのReimportを行う
4. （↑で直らないなら） `\Library` を消してUnity Editorを再起動する

## アセンブリ単位でのアクセスレベルを利用する

C#のアクセス修飾子にはアセンブリ単位でアクセスを制御するものがあります。  
外部公開するライブラリを作るときなどに便利に使えるためお勧めです。

詳しくは [岩永さんのサイト](https://ufcpp.net/study/csharp/oo_conceal.html) を見てもらうとよいのですが、一応解説します。

### アクセスレベル

C#のアクセス修飾子には次のものがあります。

| レベル | 説明 | 備考 |
| --- | --- | --- |
| public | どこからでもアクセス可能 | ガバガバ |
| protected | クラス内とその派生クラスからのみアクセス可能 | 継承するときに使う |
| **internal** | 同一アセンブリのクラス内からのみアクセス可能 | アセンブリ内からはpublic、外からはprivateに相当 |
| **protected internal** | 同一アセンブリのクラス内 OR 派生クラスからアクセス可能 |  |
| **private protected** | 同一アセンブリのクラス内 AND 派生クラスからアクセス可能 | C# 7.2以降 |
| private | クラス内からのみアクセス可能 | 一番厳しい |

とくに `internal` 、 `protected internal` 、 `private protected` は `Assembly Definition` を利用している場合のみに利用できるアクセスレベルです。

これらを活用することでライブラリの実装が少し楽になったりします。

### 利用例1:管理用のメソッドの定義

`internal` を利用することで、「同一アセンブリ内からのみ呼び出せるメソッド」を定義することができます。  
これはライブラリやフレームワークを作成する場合に、オブジェクトの管理メソッドなどを定義するときに活用することができます。

```cs
/// <summary>
/// 敵
/// </summary>
public class Enemy
{
    /// <summary>
    /// ライブラリ外から呼びだせるのはこっち
    /// </summary>
    public void Destroy()
    {
        /*
         * なんか実装
         * いろいろ処理が走ったり走らなかったり
         */
    }

    /// <summary>
    /// ライブラリ内部でリソース管理用に呼び出すのはこっち
    /// ライブラリ内からのみコール可能
    /// </summary>
    internal void ForceDestroy()
    {
        /*
         * なんか実装
         * Force()より単純化されていたり、複雑だったり…
         */
    }
}
```

### 利用例2:ファクトリの強制

クラスや構造体のコンストラクタのアクセスレベルを `internal` にすることで、ファクトリ経由したインスタンス化をライブラリ外に強制することができます。

User

```cs
/// <summary>
/// User
/// </summary>
public class User
{
    /// <summary>
    /// Userの識別子
    /// </summary>
    public int Id { get; }

    /// <summary>
    /// コンストラクタをinternalで定義する
    /// </summary>
    internal User(int id)
    {
        Id = id;
    }
}
```

UserFactory

```cs
public class UserFactory
{
    /// <summary>
    /// UserIdを発行してもらうのに使う
    /// </summary>
    private readonly UserClient _userClient;

    public UserFactory(UserClient userClient)
    {
        _userClient = userClient;
    }

    /// <summary>
    /// Userのファクトリメソッド
    /// </summary>
    public async Task<User> CreateUserAsync()
    {
        // Idをサーバから発行してもらって使う、など
        var newId = await _userClient.CreateUserIdAsync();
        return new User(newId);
    }
}
```

別のアセンブリから呼び出した場合

```cs
using UserAssembly;
using UnityEngine;

public class Sample : MonoBehaviour
{
    // Inject from DI Container
    [Inject] private UserFactory _userFactory;

    public async void Start()
    {
        /*
         * 「Cannot access internal constructor 'User' here」
         * とエラーが出てインスタンス化できない
         */
        
        // var user = new User(123);

        
        /*
         * ファクトリ経由ならインスタンス化できる
         */
        var user = await _userFactory.CreateUserAsync();
    }
}
```

## internalアクセスを外部アセンブリに公開する

たとえばライブラリを作っているときにこのテストコードを用意した場合、ライブラリ本体とテストコードは別のアセンブリに別れてしまいます。  
そうなってくると、 `internal` などのアクセスレベルにテストコードからはアクセスができない、といった状況になってしまいます。

この問題を解決する方法としては `AssemblyInfo.cs` を定義すればOKです。

これについてはすでに別の記事で解説しているため、そちらを参考にしてください。

- [【Unity】 Assembly Definition Files を使った上でinternalにアクセスする方法](https://qiita.com/toRisouP/items/156eb91dfe7e4a848dc3)

## まとめ

- `Assembly Definition` はプログラマなら必須で覚えておくべき機能
- ライブラリやアセットを公開する場合は必ず `adf` を定義しよう
- `internal` などのアクセスレベルを活用しよう

[0](https://qiita.com/toRisouP/items/#comments)

[345](https://qiita.com/toRisouP/items/d206af3029c7d80326ed/likers)

262