---
title: GoroutineとChannel
source: https://zenn.dev/mikankitten/articles/6344d71f4f4920
author:
  - "[[Zenn]]"
published: 2021-07-17
created: 2025-05-06
description: 
tags:
  - Go
  - 1
read: false
---
64

1[tech](https://zenn.dev/tech-or-idea)

## 最初に

`ゴルーチン(goroutine)` 使ってますか？ あまり使わない、使い所がわからない、使い方がわからないなど様々な意見がありそうです。今回はGoの最大の？特徴でもある `ゴルーチン(goroutine)` と `チャネル(channel)` を紹介してみます。  
Goの並行処理(`goroutine`)を使う場面でよく見かける `channel` という単語を見かけると思います。  
Go言語の中で `goroutine` を利用する以外の場面ではあまり使われないですが `goroutine` の機能を利用する上では重要な役割を果たしている機能です。

## Goroutineとは

まず、 `ゴルーチン(goroutine)` って何なのか。  
`ゴルーチン` はGoプログラムの最も基本的な構成単位です。なぜならGoのプログラムで main関数で実行される処理が `ゴルーチン(goroutine)` だからです。それを `メインゴルーチン` と呼びます。 `Goroutine` は Goランタイムによって管理される `軽量な並行処理スレッド（コルーチン）` です。通常の `コルーチン）(co-routine)` とは異なり開発者が処理の操作・制御を行う事はできません。というより制御する必要がないという方がいいかもしれません。  
スレッド数やメモリアクセスの管理など複雑な作業はランタイムが管理するため、開発者側は実装に注力できるということになります。  
Goroutineについては、「 [Go言語による並行処理](https://www.oreilly.co.jp/books/9784873118468/) 」を読まれると理解が深まると思います。

## 実装する

`goroutine` の定義方法は簡単です。  
つまり並行処理スレッドの生成コストは小さく、かつ容易に実装できるようにGo言語がそれを提供しています。

```
// 関数の前に go キーワードを追加する
go f(x,y,z)

// 即時関数でも定義できます
go func(x,y,z int){
    return x + y + z
}(p1,p2,p3)
```

ゴルーチンの実装例

```
// 
func output(s string, wg *sync.WaitGroup) {
    // WaitGroupを終了させるコード
    defer wg.Done()

    for i := 0; i < 5; i++ {
        time.Sleep(100 * time.Millisecond)
        fmt.Printf("%d回目の %s\n", i, s)
    }
}

// 通常の関数
func outputNormal(s string) {
    for i := 0; i < 5; i++ {
        time.Sleep(100 * time.Millisecond)
        fmt.Printf("通常：%d回目の %s\n", i, s)
    }
}

func main() {
    // goroutineより、main関数が終了するより時間がかかるため
    // 処理まちをさせるために、syncパッケージのWaitGroupを利用します
    var wg sync.WaitGroup
    wg.Add(1)

    // goroutineを実際にはこんな感じで実装します
    go output("Hello", &wg)

    // 普通の処理
    outputNormal("World")

    // メイン関数の処理がsay関数より早く終了するために
    // WaitGroupでgoroutineの待ち合わせをします
    wg.Wait()
}
```

output:

```
通常：0回目の World
GR: 0回目の Hello
GR: 1回目の Hello
通常：1回目の World
通常：2回目の World
GR: 2回目の Hello
GR: 3回目の Hello
通常：3回目の World
通常：4回目の World
GR: 4回目の Hello
```

通常処理と `goroutine` での処理が平行に処理されているような出力になりました。

[Go Playgroundで確認する](https://play.golang.org/p/xAmB5mQzW64)

## 処理の概要図

メイン処理から分離・分岐し、平行で実行されて再びメイン処理に合流するイメージです。

![](https://storage.googleapis.com/zenn-user-upload/b02848dbd1c8065162838942.png)

## Channelとは

`Channel` は「チャネル」と呼びます。「チャンネル」でもいいのかな 🤔

[Go by Example: Channels](https://gobyexample.com/channels) には↓のように書かれています。

> Channels are the pipes that connect concurrent goroutines. You can send values into channels from one goroutine and receive those values into another goroutine.  
> (チャネルは、同時実行するゴルーチンを接続するパイプです。あるゴルーチンからチャネルに値を送信し、それらの値を別のゴルーチンに受け取ることができます。)

## Channelの概要図

同時実行されているgoroutine間でデータの送受信ができる機能です。

![](https://storage.googleapis.com/zenn-user-upload/89c98d5849223a8f9ca504e9.png)

## Channelの実装

シンタックス:

```
// 通信データがintの場合
var ch chan int
```

チャネルを `make` 関数で定義する場合

シンタックス:

```
// 通信データがintの場合
ch := make(chan int)

// Hogeというstructの場合
ch := make(chan Hoge)
```

`var` で作成されるチャネルは `nil` になりますが、 `make` で作成されたチャネルは `nil` にはなりません。

```
package main

import (
    "fmt"
)

func main() {
    // varキーワードで作成する場合
    var mych chan int
    // 値はnil
    fmt.Println("mychの値: ", mych)
    fmt.Printf("チャネル：mychのデータ型: %T ", mych)

    // makeで作成する場合
    mychMk := make(chan int)
    // 値はアドレスが入ります
    fmt.Println("mychMkの値: ", mychMk)
    fmt.Printf("チャネル：mychMkのデータ型: %T ", mychMk)
}
```

output:

```
mychの値:  <nil>
チャネル：mychのデータ型: chan int 
mychMkの値:  0xc000062060
チャネル：mychMkのデータ型: chan int
```

[Go Playgroundで確認する](https://play.golang.org/p/XYcpGaxOiFd)

### データの送受信

`<-` 演算子を使って送受信を定義します。  
`<-` はデータがはいるチャネルの方向を示して、送信は `chan<-` 、受信は `<-chan` と表します。

```
// チャンネル:ch にデータ:vを送信する
ch <- v

// チャンネル:ch からデータを受信してvに代入する
v := <-ch
```

![](https://storage.googleapis.com/zenn-user-upload/2dfabec0e094816fdbe4bf37.png)

デフォルトでは、反対側の準備ができるまで送受信をブロックします。  
この機能によって、明示的なロックや条件を変数で設定することなくゴルーチンを同期できます。  
送信された値を受信する準備ができている受信（<-chan）が設定されている場合にのみ、送信（chan <-）を受け入れます。

## Channelの種類

チャネルにはバッファードチャネル(`Buffered Channel`)とアンバッファードチャネル(`Unbuffered Channel`)の２種類あります。デフォルトはアンバッファードチャネルです。バッファードチャネル(`Buffered Channel`)は、それに対応する受信（<-chan）がなくても、限られた数の値を受け入れます。

### バッファードチャネル(Buffered Channel)

バッファーチャンネルも `make` 関数で定義しますが、サイズを設定する点が異なります。

```
// 通信データがintの場合、データサイズも指定します
ch := make(chan int, 3)
```

### バッファードチャネルの実装

```
func main() {
    // intを送受信するチャネルを定義します。データサイズは2とします。
    ch := make(chan int, 2)
    // 1つ目のデータ
    ch <- 1
    // 2つ目のデータ
    ch <- 10

    fmt.Println("chのデータ数は",len(ch))
    // チャネルの送受信を止めます
    close(ch)

    // チャネルの中身を出力する
    for c := range ch {
        fmt.Println(c)
    }
}
```

コンソール出力結果

```bash
chのデータ数は 2
1
10
```

#### データサイズ以上のデータを入れようとすると

```
func main() {
    // intを送受信するチャネルを定義します。データサイズは2とします。
    ch := make(chan int, 2)
    // チャネルにデータサイズ以上を入れようとすると
    ch <- 1
    fmt.Println("chのデータ数は",len(ch))
    ch <- 10
    fmt.Println("chのデータ数は",len(ch))
    ch <- 100
    fmt.Println("chのデータ数は",len(ch))
}
```

コンソール出力結果は、オーバフローしてしまうのでエラーとなります。ご注意ください。

```bash
chのデータ数は 1
chのデータ数は 2
fatal error: all goroutines are asleep - deadlock!
```

## データ送受信をとめる

`close()` 関数を使いチャネルを非アクティブにできます。クローズしたチャネルはデータの送受信ができなくなります。

シンタックス:

```
close(chan)
```

## クローズの確認

シンタックス：

```
response, ok = <-myChan
```

## channelを使って実装する

最初に実装した `goroutine` のサンプルで `sync` パッケージの `waitGroup` を使った実装をしましたが、今回は `channel` を使った実装のサンプルをつくってみます。

```
package main

import (
    "fmt"
)

// 受け取った文字列を出力する関数
func say(strs ...string) {
    // stringを受け取るchannel
    ch := make(chan string)
    n := 0
    for _, str := range strs {
        n++
        // goroutineでchannelに文字を入れる処理
        go func(s string) {
            ch <- s
        }(str)
    }

    for i := 0; i < n; i++ {
        // 出力する
        fmt.Println(<-ch)
    }
    // 送受信をやめる
    close(ch)
}

func main() {
    say("hello", "goroutine", "world", "with", "channel")
}
```

コンソール出力結果は、処理がされた順に出力されます。（その都度結果は変わります）

```bash
channel
goroutine
hello
world
with
```

[Go Playgroundで確認する](https://play.golang.org/p/1EonAQJ0Gl8)

## Channel vs WaitGroup

では、 `channel` と `sync.WaitGroup` が出てきましたが、どういったケースがそれぞれ適しているかを解説します。

**`sync.WaitGroup` が適しているのは**

- `goroutine` から返る結果には興味がないケース
- シンプルに処理を止めておくという明確な実装のみでいいケース

**`channel` が適しているのは**

- `goroutine` から返る結果が重要なケース
- ただ止めるだけではなく、その結果にこだわるケース

という観点で使い分けの目安になると思います。

## 最後に

今回は `goroutine` と `channel` という特徴的な機能について少し深堀りしてみました。  
実際に使えるチャンスが有る場合は積極的に使っていきたいなと思います。

## 参考リンク

- [Go言語による並行処理](https://www.oreilly.co.jp/books/9784873118468/)
- [Go by Example: Channels](https://gobyexample.com/channels)
- [A tour of Go: Channels](https://tour.golang.org/concurrency/2)

64

1