---
title: 【C#】SpanとMemory
source: https://annulusgames.com/blog/span-and-memory/
author:
  - "[[Annulus Games]]"
published: 2024-03-21
created: 2025-05-06
description: 今回の記事はC#のSpanとMemoryについて。 現代のC#ではパフォー
tags:
  - CSharp
  - 設計
  - Tech
  - QA
read:
---
今回の記事はC#のSpan<T>とMemory<T>について。

現代のC#ではパフォーマンス向上のためにSpanが用いられる機会が非常に多くなっています。.NETでも多くのAPIがSpan<T>を受け入れるようになってきており、パフォーマンスに気を遣ってコードを書く場面ではもはやSpanの活用は必須と言えます。

また、C#ではSpanとは別に似たような型としてMemory<T>も存在しています。こちらは利用する上での制約がSpanと比べて少なく、Spanの代替として主にasyncメソッド内で用いられることが多いです。

そこで今回は、Span<T>やMemory<T>の利用方法や使い分け、またMemory<T>を適切に扱うための指針やIMemoryOwner<T>による所有権の管理についてまでをまとめていきたいと思います。基本的にはSpan<T>の利用のみで事足りますが、Memory<T>が必要になる場面も少なからず存在するので覚えておいて損はない知識でしょう。

記事の後半は所有者モデルによる安全なバッファ管理や、IMemoryOwner<T>による所有権の明示などのトピックを扱います。C#ではメモリ管理を明示的に行うことがほとんどないためやや高度な内容となりますが、Rustなど他の言語を使用する際にも生きる知識であるため興味のある方はぜひ読んでみてください。

## Span<T> / ReadOnlySpan<T>

まずは利用頻度の高いSpan<T> / ReadOnlySpan<T>から話を始めていきましょう。

Spanは配列や文字列などの **連続したデータにアクセスするための構造体** です。Spanを使用することで余計なアロケーションを避けつつ、高速な読み書きを可能にします。

```c#
var array = new int[8];

// 配列の一部をSpan<int>として取得
var span = array.AsSpan(2, 4);

// Spanを介してデータを書き込む
for (int i = 0; i < span.Length; i++)
{
    span[i] = 1;
}

// 元の配列が書き換えられている
foreach (var x in array)
{
    // 0, 0, 1, 1, 1, 1, 0, 0
    Console.WriteLine(x);
}
```

上のコードでは配列のindex\[2\]から4つ分だけの要素を指すSpanを作成し、それを介してデータを書き込んでいます。Span自体は構造体であるため、作成時にアロケーションは発生しません。

データを読み取り専用にしたい場合はReadOnlySpan<T>を使用します。 **Span<T>からReadOnlySpan<T>へは暗黙的に変換が可能です。**

```c#
// Spanを読み取り専用として受け取る(Span<int> -> ReadOnlySpan<int>の変換)
ReadOnlySpan<int> span = array.AsSpan(2, 4);
```

また、Spanは配列以外にも文字列(string)やスタック領域、アンマネージドなメモリ領域など、様々な場所を指すことが可能です。

以下は文字列の一部をReadOnlySpan<char>として取得するコード例です。Substringとは異なり文字列のコピーを作成しないため、余計なアロケーションが発生せず高速に読み取ることができます。

```c#
// 文字列の一部'ABCDE'をReadOnlySpan<char>として取得
// コピーを作成しないためアロケーションもなく高速
ReadOnlySpan<char> span = "ABCDEFGHIJKLMNOPQRSTUVWXYZ".AsSpan(0, 5);
```

### ArraySegment<T>との比較

従来のC#でも、配列の一部を読み書きするための型としてArraySegment<T>が存在しました。これとSpan<T>の違いを実装から見ていきましょう。

まずはArraySegment<T>から。こちらは概ね以下のような実装になっています。

```c#
public struct ArraySegment<T> : IList<T>, IReadOnlyList<T>
{
    private T[] _array;
    private int _offset;
    private int _count;
}
```

実装からもわかる通り、ArraySegment<T>はマネージドな配列のみを指すことが可能です。またArraySegment<T>は配列自体の参照を内部に保持し、それを介して配列の読み書きを行います。

対して、Span<T>の実装は概ね以下のようになります。(詳しくは後ほど解説しますが、Spanの実装は.NETのバージョンによって大きく異なります。以下は.NET 7以降とほぼ同等のものです。)

```c#
public readonly ref struct Span<T>
{
    internal readonly ref T _reference;
    private readonly int _length;
}
```

SpanはArraySegmentとは異なり、値の参照を直接保持します。そのため、配列に限らずあらゆるメモリ領域を参照することが可能です。またSpanにはランタイム側で特殊な最適化がかかるため、ArraySegmentと比較するとアクセスは遥かに高速です。

ArraySegmentは古い型であり用いられる場面は非常に少ないですが、Spanは既にあらゆる箇所で利用されるようになってきています。SpanとMemoryが中心となっている現在のC#において、わざわざArraySegmentを使用する機会はほとんどないと言って良いでしょう。

### ポインタとSpan<T>

Spanを使用する利点の一つとして、配列やポインタなどを一つの型で統一的に扱えるという点が挙げられます。

通常C#でポインタが用いられる機会はあまり多くありませんが、相互運用や高いパフォーマンスが求められる場面では往々にしてunsafeコードが使用されます。

```c#
// ポインタを引数に受け取るメソッド
public static unsafe void Foo(byte* ptr, int length)
{
    ...
}
```

ライブラリ内でunsafeが使用される分にはあまり問題はありませんが(ライブラリの作成者側で安全性を確保すれば済む話なので)、外部に公開するAPIにポインタが含まれる場合は呼び出し側にunsafeコンテクストを要求することになってしまいます。ライブラリならともかく、アプリケーション側でunsafeを使用することは安全性の観点から極力避けるべきです。

```c#
unsafe
{
    var array = new byte[10];
    fixed (byte* ptr = array)
    {
        // メソッドを呼び出す側もunsafeでなければならない
        Foo(ptr, array.Length);  
    }
}
```

そこでSpanを用いることにより、ポインタと配列を共通化しつつ安全に扱うことができます。以下のコードに示す通り、ポインタからSpanを作成したり、fixedでSpanからポインタを取得することが可能です。

```c#
// ポインタの代わりにSpanを引数に受け取る
public static unsafe void Foo(Span<byte> span)
{
    // Spanは配列と同様、fixedを用いてポインタを取得可能
    // ただしSpanは様々な最適化が施されているので、基本的にはそのままで十分(安全だし)
    fixed (byte* ptr = span)
    {
        ...
    }
}

var array = new byte[16];

// 配列を渡す
Foo(array.AsSpan(0, 8));

unsafe
{
    fixed (byte* ptr = array)
    {
        // ポインタからSpanを作成して渡す
        Foo(new Span<byte>(ptr, 8));  
    }
}
```

### Span<T>を用いたstackalloc

C#の配列は基本的にヒープ上に置かれますが、stackalloc式を用いることでスタック上に配列を確保できます。これは小さめのバッファをヒープアロケーションなしで確保する場合に便利ですが、従来のstackalloc式はunsafeブロック内でしか利用できない機能でした。

```c#
unsafe
{
    // 配列をスタック上に確保、ポインタを取得
    int* buffer = stackalloc int[10];
}
```

しかしSpanの登場後、stackallocで確保した配列をSpan<T>で受け取る機能が追加されました。これによりunsafeブロック内でなくともstackallocが使用可能になります。もちろん、ポインタとは異なりSpan<T>で受け取ったstackallocの配列は安全に使用できます。

```c#
// ポインタではなくSpan<T>で受け取る
Span<int> buffer = stackalloc int[10];

try
{
    // 境界外にアクセスした場合はちゃんと例外がスローされる
    buffer[-1] = 1;
}
catch (IndexOutOfRangeException ex)
{
    Console.WriteLine(ex);
}
```

またSpan<T>によるstackallocが追加されたC# 7.2では代入や条件演算子でのみstackallocを使用できましたが、C#8.0以降ではこの制限が撤廃されコード内のあらゆる箇所でstackallocが利用可能になっています。

```c#
// 条件演算子でstackalloc (これはC#7.2でも可能)
var buffer = size > 512 ? new byte[size] : stackalloc byte[size]

// Spanを引数にとる関数
void Foo(Span<byte> span)
{
   ...
}

// 引数内でstackalloc
Foo(stackalloc byte[1]);
```

### Span(ref構造体)の制約

ここまで見たきた通りSpanは非常に有用な型ですが、利用するにあたって幾つかの制約があります。

C#において、Span<T>は [ref構造体](https://learn.microsoft.com/ja-jp/dotnet/csharp/language-reference/builtin-types/ref-struct) として定義されています。

```c#
// refキーワードがついた構造体
public readonly ref struct Span<T>
{
    ...
}
```

refでマークされた構造体のインスタンスはスタック上に配置されることが保証されます。逆に言うと、 **ref構造体をヒープに配置することはできません。**

例えば、以下のようなコードはコンパイルエラーになります。クラスのフィールドはマネージドヒープに配置されるためです。

```c#
// Span(ref struct)をフィールドに保持することはできない
class FooClass
{
   // error CS8345: Field or auto-implemented property cannot be of type 'Span<int>' unless it is an instance member of a ref struct.
   public Span<int> span;
}
```

ただし例外として、ref構造体の中にはSpan等のref構造体をフィールドとして保持することが可能です。

```c#
// ref構造体はそれ自体がスタックに置かれることを保証されるため、内部にもref構造体を持てる
public ref struct FooStruct
{
    public Span<int> span;
}
```

また、他にもref構造体には **「インターフェースを実装できない」「イテレータやasyncメソッド内で使用できない」「ボックス化できない」** などの様々な制約がかかります。  
(foreachやusingに使用されるGetEnumeratorやDisposeはダックタイピングであるため、インターフェースを実装しなくても同名のメソッドを定義することでref structを渡すことができます。)

当然、ref構造体のこれらの制約はSpanにおいても同様です。そのため、これらの制約外でSpanのようなものを使用したい場合には、後述するMemory型を代替として使用することになります。

### \[余談\] Fast SpanとSlow Span

余談としてSpan<T>の内部実装にも触れておきましょう。

雑に説明すると、Spanは「参照」と「長さ」の2つを持つ構造体です。ただし、Spanは.NETのバージョンによってやや実装が異なり、その性能差からFast Span、Slow Spanなどと呼び分けられることがあります。

まずは.NET 7以降の実装です。.NET 7 / C#11以降ではrefフィールド(参照を保持するフィールド)を使用できます。そのため、Spanの実装にはこれが用いられています。

```c#
public readonly ref struct Span<T>
{
    internal readonly ref T _reference;
    private readonly int _length;
}
```

しかし、refフィールドはそれ以前のバージョンでは使用できません。そこで.NET Core 2.1ではランタイム側で特別な対応を行うByReference<T>型を導入することにより、これと同じ動作を実現しています。

```c#
public readonly ref struct Span<T>
{
    private readonly ByReference<T> _pointer;
    private readonly int _length;
}
```

これらがいわゆるFast Spanと呼ばれる実装です。

しかし、refフィールドもランタイム側の特殊対応もないそれ以前のバージョンでは上の実装は使用できません。そこで、性能を落としつつ旧来のバージョンに対応したSpan実装(Slow Span)も存在します。

```c#
public readonly ref struct Span<T>
{
    private readonly Pinnable<T> _pinnable;
    private readonly IntPtr _byteOffset;
    private readonly int _length;
}
```

こちらはmanagedなメモリをPinnable<T>という型で、unmanagedなメモリをIntPtrで保持することでSpan実装を実現しています。managed/unmanagedの分岐が必要になるほか、構造体サイズも大きくなっているためFast Spanと比べると若干性能は低下します。(それでもArraySegment<T>などよりは高速です)

## Memory<T> / ReadOnlyMemory<T>

続いてはMemory<T> / ReadOnlyMemory<T>について。

MemoryもSpan同様、何らかのメモリ領域を読み書きするために用いられる型です。ただし、Spanとは異なる点として、こちらはref構造体ではなく普通の構造体として定義されています。そのためクラスのフィールドにしたり、イテレータやasyncメソッドに渡したりなど、Spanよりも広い範囲で利用することが可能です。

```c#
public class FooClass
{
    // Spanとは異なりフィールドにできる
    public Memory<int> memory;
}

// asyncメソッドにも渡せる
public async Task BarAsync(Memory<int> memory)
{
    ...
}
```

Memoryも配列や文字列から簡単に作成することが可能なほか、読み取り専用にしたい場合はReadOnlyMemory<T>への暗黙的な変換が可能です。これもSpanと同じですね。

```c#
// 配列の一部をMemory<int>として取得
var array = new int[8];
var memory = array.AsMemory(2, 4);

// 文字列の一部をReadOnlyMemory<char>として取得
var str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
var strMemory = str.AsMemory(0, 5);
```

また、Memory<T>はSpan<T>に簡単に変換が可能です。Memory自体はインデクサなどを持たないため、読み書きの際にはSpanに変換して使用します。

```c#
// Spanプロパティで簡単に変換が可能
var span = memory.Span;
```

### Memory<T>の実装

Spanと合わせて、Memoryの実装についても触れておきましょう。Memory<T>型の実装は以下のようになっています。

```c#
public readonly struct Memory<T>
{
    private readonly object? _object;
    private readonly int _index;
    private readonly int _length;
}
```

MemoryはSpanとは異なり、オブジェクトの参照を内部で保持します。実装としてはArraySegment<T>に近いイメージですが、Memoryはobjectを保持するため配列以外を指すことも可能です。

ただし、Spanとは異なりMemoryはスタック領域を指すことができません。stackallocの際にはSpanかポインタを使用する必要があります。

また後ほど触れますが、Memoryを扱う場合、ガイドラインに沿って適切に運用しないと安全性を損なう可能性があります。そのため、 **可能な限りSpanを優先して使用する** ことが推奨されます。

## 所有権・消費・リース

先ほどMemory<T>の項目でも触れた通り、Memory<T>等のバッファは複数スレッドからアクセスされる可能性があるため、堅牢さを保つためにはバッファの有効期間を意識した運用が必須です。

これを適切に扱うため、所有権・消費・リースの3つの概念が用いられます。SpanやMemoryの話から少し逸れますが、安全なバッファ管理を行うために必要な概念であるため簡単に触れておきましょう。

### 所有権 (Ownership)

所有権は **バッファの管理を行う権利(義務)** のことです。C++のmove semanticsにも登場するほか、Rustでは言語設計の中心に組み込まれている概念でもあるため、これらの言語を使用している方には馴染み深い概念かもしれません。

所有権の文脈において、 **全てのバッファの所有者は必ず一つ** になります。所有者は最後にバッファを破棄する義務を持ちますが、これを他の相手に譲渡することも可能です。(所有権の移動)

そして所有権を譲渡した場合、譲渡した側はその時点でバッファを使用することができなくなります。例えばコンポーネントAがコンポーネントBにバッファの所有権を譲渡した場合、コンポーネントAはその時点でバッファを使用できなくなり、コンポーネントBがバッファの破棄までを担当することになります。

### 消費 (Consumption)

バッファは読み取りや書き取りに使用することが可能です。これを消費と呼び、消費する箇所を **コンシューマー** (消費者)と呼びます。

同時アクセスを避けるため、 **基本的にバッファが同時に持てるコンシューマーは一つ** です。ただし、なんらかの同期メカニズムが存在する場合はこの限りではありません。また、バッファのコンシューマーはバッファの所有者と必ずしも一致するわけではありません。

### リース (Lease)

リースは特定のコンポーネントが **バッファのコンシューマとなれる期間** を示す概念です。バッファを消費することができる期間を「バッファ上にリースを持つ」と表現します。

### コードによる例

概念の説明だけではわかりづらいので、これらを下のC#コードを例に説明していきます。

```c#
// ここでのBuffer型は何らかのバッファを表す

// バッファへ整数値を書き込む
void WriteToBuffer(int value, Buffer buffer)
{
    ...
}

// バッファの中身をコンソールに表示する
void DisplayBufferToConsole(Buffer buffer)
{
   ...
}

static void Main()
{
    // バッファを作成する
    var buffer = CreateBuffer();

    try
    {
        // コンソールに渡された値を読み取る
        int value = int.Parse(Console.ReadLine());

        WriteToBuffer(value, buffer);
        DisplayBufferToConsole(buffer);
    }
    finally
    {
        // バッファを破棄する
        buffer.Dispose();
    }
}
```

まずこのコードでは、初めにMainメソッド内でバッファを作成し所有しています。そのためMainメソッドはバッファの所有権を持ち、最終的にバッファを破棄する責任があります。(このコードでは処理の最後にfinallyでバッファを破棄しています。)

コード内にはWriteToBufferとDisplayBufferToConsoleの二つのメソッドが存在します。これらはバッファの書き込み/読み取りを行うためコンシューマーと見なせます。これらのメソッドはバッファを消費していますが、バッファを所有しているわけではありません。

また、メソッド呼び出しの開始からメソッドから返されるまでの間、WriteToBufferとDisplayBufferToConsoleのメソッドはバッファー上にリースを持ちます。これらのメソッドはこの間だけバッファを消費することができ、メソッドを抜けるとリースは解放されます。

## IMemoryOwner<T>を用いた所有権の管理

.NETでは、バッファの所有権を明示的に管理するためのインターフェースとして「IMemoryOwner<T>」が用意されています。

```c#
public interface IMemoryOwner<T> : IDisposable
{
    Memory<T> Memory { get; }
}
```

IMemoryOwner<T>自体はMemoryプロパティとDisposeメソッドのみを持つインターフェースです。Memoryプロパティからバッファを取得し、使い終わった後はDisposeで破棄します。これをコンポーネント間でやり取りすることにより、所有権の移動を明示することが可能になります。

これも実際のC#コードを通して見ていきましょう。

このコードでは最初にプールからバッファを取得しています。MemoryPool<T>は共有のメモリプールを扱うことが可能なクラスで、RentメソッドはIMemoryOwner<T>を返します。

今回の場合はMainメソッド内でIMemoryOwner<T>を受け取って参照を保持しているため、Mainメソッドがバッファの所有者であると考えることが可能です。

続いては読み取った文字列をバッファに書き込み、Consoleに表示する部分を見ていきます。ここではWriteToBufferとDisplayBufferToConsoleの二つのメソッドがMemory<T>(ReadOnlyMemory<T>)を受け取り、書き込み/読み取りを行うことでバッファを消費しています。そのためこれらはコンシューマーであると考えられるでしょう。

最後にバッファの所有者であるMainメソッドがIMemoryOwner<T>.Dispose()を呼び出してバッファを破棄します。今回は最後までMainメソッドがバッファの所有者でしたが、IMemoryOwner<T>を受け渡すことで所有権を譲渡することも可能です。

### 所有者なしのMemory<T>

先ほどのコードではIMemoryOwner<T>が存在したため、所有者を明示することができました。それでは、AsMemoryなどでMemory<T>を取得した場合には、誰が所有者となるのでしょうか。

```c#
static void Main()
{
    // 所有者が存在しない...?
    ReadOnlyMemory<char> buffer = str.AsMemory();
}
```

このような場合には、バッファのインスタンスを作成したメソッド(今回はMainメソッド)が暗黙的な所有者として扱われます。IMemoryOwner<T>を使用しないため、所有権を外部に譲渡することはできません。

(またはランタイムのガベージコレクションがバッファを所有し、全てのメソッドがコンシューマーであると見なすことも可能です。いずれにせよ、バッファの所有者は有効期間全体にわたって単一となります。)

## Memory<T>とSpan<T>のガイドライン

解説が長くなりましたが、再び話をSpanとMemoryに戻しましょう。

上のMemory<T>や所有権の話でも触れた通り、バッファは複数のコンポーネントに渡される可能性があるため、安全に扱うためにはガイドラインに沿った運用を行う必要があります。ここでは、Span<T>とMemory<T>の使い分けや、Memory<T>を安全に使用するためのガイドラインを一部紹介します。

ここで紹介するガイドラインはMicrosoftの以下の記事で提唱されているものです。今回触れた所有権などに関する話も載っているため、一度目を通してみると良いでしょう。

> Memory<T> と Span<T> の使用ガイドライン  
> [https://learn.microsoft.com/ja-jp/dotnet/standard/memory-and-spans/memory-t-usage-guidelines](https://learn.microsoft.com/ja-jp/dotnet/standard/memory-and-spans/memory-t-usage-guidelines#usage-guidelines)

### 1\. 同期APIの場合、可能な限りMemory<T>ではなくSpan<T>を使う

基本的にSpanはMemoryよりもパフォーマンスの面で優れています。また、Memory<T>からSpan<T>への変換は可能ですが、その逆は不可能です。他にもSpan<T>はスタックを指すことができるなど、汎用性の面においても優れています。

ref構造体の制約を持つため全てSpanというわけにはいきませんが、使用可能であればSpanを優先して使用しましょう。非同期処理でない場面では、ほとんどのケースにおいてSpanのみで十分です。

### 2\. 読み取り専用の場合はReadOnlySpan<T> / ReadOnlyMemory<T> を使う

記事の前半で解説した通り、Span<T>とMemory<T>には読み取り専用のReadOnlySpan<T>、ReadOnlyMemory<T>が存在します。バッファを読み取るだけの場合には、これらを使用を優先しましょう。

```c#
// 読み取り専用の場合はReadOnlyの方で受け取る
static void Foo(ReadOnlySpan<byte> buffer) { ... }
```

### 3\. メソッドがMemory<T>を受け取ってvoidを返す場合、メソッドの終了後にMemory<T>インスタンスを使用しない

この規則は少々わかりづらいので、具体的なコードを示しながら説明します。まずは、以下のメソッドを例に挙げて考えてみます。

```c#
// ReadOnlyMemory<char>を受け取るメソッド
static void Log(ReadOnlyMemory<char> message) { ... }
```

Logメソッドが同期処理である場合は、何ら問題なくバッファを消費できます。そもそもこのような場合にはReadOnlySpan<T>で十分でしょう。

```c#
static void Log(ReadOnlyMemory<char> message)
{
    // バッファを読み取りコンソールに表示する
    Console.WriteLine(message);
}
```

しかし、もしLogメソッドが以下のような実装を持っていたらどうでしょう。

```c#
// この実装はNG！！！
static void Log(ReadOnlyMemory<char> message)
{
    // Task.Runで別スレッドに処理を逃す
    Task.Run(() =>
    {
        var writer = File.AppendText(@".\input-numbers.dat");
        // ここでバッファを消費
        writer.WriteLine(message);
        writer.Flush();
    });
}
```

この実装が危険な理由は、先ほどの「リース」の概念から説明ができます。

Logメソッドはバッファを受け取って消費しますが、リースはLogメソッドの終了と同時に終了します。しかし、Task.RunはLogメソッドの終了後にバックグラウンドでバッファを使用するため、リースに違反しています。もしその間にメインスレッドからバッファを変更している間にLogメソッドがバッファを読み込んだ場合、データが破損する可能性があります。

これを避けるには、以下のような実装を行えばOKです。

```c#
// voidではなくTaskを返すように変更する
// 後は呼び出し側でawaitするなどで待機すれば良い
static Task Log(ReadOnlyMemory<char> message)
{
    return Task.Run(() =>
    {
        var writer = File.AppendText(@".\input-numbers.dat");
        writer.WriteLine(message);
        writer.Flush();
    });
}

// または以下のような方法でも可
static void Log(ReadOnlyMemory<char> message)
{
    // 防衛的にコピーしておくことでバッファへの同時アクセスを回避する
    var copy = message.ToString();
    Task.Run(() =>
    {
        var writer = File.AppendText(@".\input-numbers.dat");
        writer.WriteLine(copy);
        writer.Flush();
    });
}
```

### 4\. メソッドがMemory<T>を受け取ってTaskを返す場合、Taskの終了後にMemory<T>インスタンスを使用しない

これは先ほどのガイドラインの非同期版になります。

```c#
// Taskの終了後にmessageにアクセスしないので、この実装は問題ない
static Task Log(ReadOnlyMemory<char> message)
{
    return Task.Run(() =>
    {
        var writer = File.AppendText(@".\input-numbers.dat");
        writer.WriteLine(message);
        writer.Flush();
    });
}
```

Taskが完了、失敗、キャンセルなどで終了した場合は、それ以降にバッファにアクセスしてはいけません。これは戻り値がTask<T>、ValueTask、ValueTask<T>の場合も同様です。

### 5, 6. コンストラクタがMemory<T>を受け取る場合、またはフィールドやプロパティにMemory<T>を持つ場合、インスタンスはコンシューマーであるとみなされる

こちらも少々ややこしいですが、要するに以下のような場合です。

```c#
class OddValueExtractor
{
    // コンストラクタでReadOnlyMemory<int>を受け取る
    public OddValueExtractor(ReadOnlyMemory<int> input) { ... }

    public bool TryReadNextOddValue(out int value) { ... }
}

void PrintAllOddValues(ReadOnlyMemory<int> input)
{
    // インスタンスがコンシューマーと見なされる
    var extractor = new OddValueExtractor(input);

    // 直接渡してはいないが、TryReadNextOddValueもコンシューマーとみなされる
    while (extractor.TryReadNextOddValue(out int value))
    {
        Console.WriteLine(value);
    }
}
```

OddValueExtractorはコンストラクタでReadOnlyMemory<int>を受け取っています。そのため、コンストラクター自体がコンシューマーであり、インスタンスメソッドもコンシューマーであるとみなされます。

また、インスタンスがフィールドやプロパティでMemory<T>を保持する場合も、同様にそのバッファのコンシューマーであると見なすことが可能です。

### 7, 8. IMemoryOwner<T>の受け取りは所有権の受け取りとみなし、受け取った側は破棄か所有権譲渡のどちらかを行う

これは「IMemoryOwner<T>を用いた所有権の管理」の項目でも触れた内容になります。何らかの手段でIMemoryOwner<T>を受け取った所有者は、必ずDisposeでバッファを破棄するか、IMemoryOwner<T>を他のコンポーネントに受け渡すかのどちらかを行う必要があります。(両方ではない)

また、IMemoryOwner<T>を譲渡した場合、譲渡した側はそれ以降は一切IMemoryOwner<T>を使用してはいけません。バッファの所有者は常に一つでなければならないことに注意してください。

### 9, 10. P/Invokeをラップする場合、同期APIはSpan<T>、非同期APIはMemory<T>を使用する

これは最初のガイドラインとも一致する内容になります。P/InvokeをラップするためにSpanやMemoryを用いる場合には、同期ならSpan<T>、非同期APIはMemory<T>を使用します。

```c#
using System.Runtime.InteropServices;

[DllImport(...)]
static extern unsafe int ExportedMethod(byte* pbData, int cbData);

// 同期処理にはSpan<T>を用いる
public unsafe int ManagedWrapper(Span<byte> data)
{
    fixed (byte* ptr = data)
    {
        var result = ExportedMethod(ptr, data.Length);
        return result
    }
}

// 非同期処理にはMemory<T>を用いる
// (長くなるのでコード例は省略)
```

## まとめ

今回はSpan<T>とMemory<T>についての内容でしたが、いかがだったでしょうか。近年の.NETはパフォーマンスにも注力しているため、Span<T>やMemory<T>を用いた最適化に遭遇する場面は非常に多くなります。速度が要求される場面ではこれらをフルに活用していくことになるでしょう。

後半で触れた所有権などの内容は、C#において実際に活用されるケースはあまり多くありません。しかし.NETでもIMemoryOwner<T>を用いた所有モデルの運用が可能、ということは覚えておくと役に立つかもしれません。

Span<T>やMemory<T>を使いこなし、パフォーマンスの優れたコードを書いていきましょう。

[Hatena](https://annulusgames.com/#hatena "Hatena") [LinkedIn](https://annulusgames.com/#linkedin "LinkedIn") [Line](https://annulusgames.com/#line "Line") [Pinterest](https://annulusgames.com/#pinterest "Pinterest")

タグ: