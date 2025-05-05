---
title: Go でマルチスレッドプログラミングする際に最低限知っておくべきこと
source: https://blog.cybozu.io/entry/2024/08/13/111500
author:
  - "[[Cybozu Inside Out | サイボウズエンジニアのブログ]]"
published: 2024-08-13
created: 2025-05-06
description: この記事は、CYBOZU SUMMER BLOG FES '24 (クラウド基盤 Stage) DAY 10 の記事です。 こんにちは。クラウド基盤本部の野島です。Go は goroutine やチャネルなどの仕組みが備わっており、簡単にマルチスレッドなプログラムを書くことができる言語だと言われています。しかし、マルチスレッドプログラミングには独特の罠があり、何も知らない人が雰囲気でコードを書くとわかりにくいバグを仕込んでしまうリスクが非常に高いです。 この記事では、マルチスレッドプログラミングに詳しくない人に向けて、そのような罠を避けるための方法を紹介します。この記事は Go の基本的な使い…
tags:
  - Go
  - Tech
  - バックエンド
  - バックエンド設計
  - 通信
read: false
---
この記事は、 [CYBOZU SUMMER BLOG FES '24](https://cybozu.github.io/summer-blog-fes-2024/) (クラウド基盤 Stage) DAY 10 の記事です。

こんにちは。クラウド基盤本部の野島です。Go は goroutine やチャネルなどの仕組みが備わっており、簡単にマルチスレッドなプログラムを書くことができる言語だと言われています。しかし、マルチスレッドプログラミングには独特の罠があり、何も知らない人が雰囲気でコードを書くとわかりにくいバグを仕込んでしまうリスクが非常に高いです。

この記事では、マルチスレッドプログラミングに詳しくない人に向けて、そのような罠を避けるための方法を紹介します。この記事は Go の基本的な使い方を知っていることを前提としています。

## 這い寄るデータ競合の恐怖

まずは以下のようなプログラムを考えてみましょう。これは複雑な計算を行って結果を返すような HTTP サーバーのコードです。

```go
// 複雑な計算を行って結果を返す API
func doExpensiveCalculation(w http.ResponseWriter, r *http.Request) {
    // クエリパラメータとして与えられた x を取得する
    x, err := strconv.Atoi(r.URL.Query().Get("x"))
    if err != nil {
        http.Error(w, "invalid argument", http.StatusBadRequest)
        return
    }

    answer := x * x // 複雑で時間のかかる計算をしていると思ってください
    fmt.Fprintf(w, "The answer is %v\n", answer)
}

func main() {
    log.Fatal(http.ListenAndServe(":8080", http.HandlerFunc(doExpensiveCalculation)))
}
```

ここまでは問題のないプログラムです。ある日、API のパフォーマンスに不満を持ったプログラマが、計算結果をキャッシュして高速に結果を返そうと、以下のような改修を行ったとします。

```go
// 結果をキャッシュしておくための map
var cache = map[int]int{}

func doExpensiveCalculation(w http.ResponseWriter, r *http.Request) {
    // クエリパラメータとして与えられた x を取得する
    x, err := strconv.Atoi(r.URL.Query().Get("x"))
    if err != nil {
        http.Error(w, "invalid argument", http.StatusBadRequest)
        return
    }

    // 結果がキャッシュされていればそれを返す
    if cached, ok := cache[x]; ok {
        fmt.Fprintf(w, "The answer is %v (cached)\n", cached)
        return
    }

    answer := x * x   // 複雑で時間のかかる計算をしていると思ってください
    cache[x] = answer // 計算結果をキャッシュする
    fmt.Fprintf(w, "The answer is %v\n", answer)
}

func main() {
    log.Fatal(http.ListenAndServe(":8080", http.HandlerFunc(doExpensiveCalculation)))
}
```

残念ながら、この改修によりデータ競合(data race)が混入してしまいました。何が起こるか実際に検証してみましょう。まずは並列にサーバーにアクセスするテストを用意します。

```go
func TestDataRace(t *testing.T) {
    // HTTPサーバーを起動する
    server := httptest.NewServer(http.HandlerFunc(doExpensiveCalculation))
    defer server.Close()

    var wg sync.WaitGroup
    // 1000個のリクエストを並行して送信する
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            _, err := http.Get(server.URL + fmt.Sprintf("?x=%d", i%30))
            if err != nil {
                t.Log(err)
            }
        }()
    }
    wg.Wait() // すべてのリクエストが完了するまで待つ
}
```

これを実行すると、私の環境では以下のようにクラッシュしてしまいました (タイミングによるので、環境によってはもしかしたら再現しないかもしれません)。

```go
❱ go test
(中略)
goroutine 1536 [runnable]:
net/http.(*Server).Serve.gowrap3()
        /home/nojima/.asdf/installs/golang/1.22.6/go/src/net/http/server.go:3290
runtime.goexit({})
        /home/nojima/.asdf/installs/golang/1.22.6/go/src/runtime/asm_amd64.s:1695 +0x1
created by net/http.(*Server).Serve in goroutine 20
        /home/nojima/.asdf/installs/golang/1.22.6/go/src/net/http/server.go:3290 +0x4b4
exit status 2
FAIL    ex1     0.352s
```

Go コンパイラにはデータ競合を検出する機能があり、 `-race` オプションを付けてコンパイルすると有効にできます。これを利用すると何が起こったのかわかりやすくなります。実際に `-race` を付けてテストを実行してみると、以下のようにデータ競合が検出されました。

```go
❱ go test -race
==================
WARNING: DATA RACE
Read at 0x00c000126ba0 by goroutine 155:
  (中略)
  ex1.doExpensiveCalculation()
      /home/nojima/tmp/go-memory-model/ex1/main.go:19 +0xb1
  (中略)

Previous write at 0x00c000126ba0 by goroutine 143:
  (中略)
  ex1.doExpensiveCalculation()
      /home/nojima/tmp/go-memory-model/ex1/main.go:25 +0x12e
  (中略)
==================
--- FAIL: TestDataRace (0.55s)
    testing.go:1398: race detected during execution of test
FAIL
exit status 1
FAIL    ex1     0.562s
```

この出力によると、 `cached, ok := cache[x]` の部分と `cache[x] = answer` の部分でデータ競合が発生したということがわかります。たしかに、 `cache` がクラッシュを引き起こしたようです。

このように、データ競合が発生すると、プログラムがクラッシュしたり、中途半端なデータが読み取られたりすることがあります。このようなバグはテストで検出することが難しく、たいていの場合、本番環境において再現性のないバグとして検出されることになります。実際、筆者は一億リクエストに一回の確率でサーバーがクラッシュするバグに悩まされた経験があります。

データ競合を起こす正確な条件を知るためには Go のメモリモデルを理解する必要があります。しかし、日常業務においてそこまでの知識が必要になることは稀です。業務においては、そんな境界ギリギリを攻めるようなプログラムを書くよりも、安全側に倒したプログラムを書くほうが多くの場合望ましいからです。

よって、この記事ではメモリモデルには立ち入らず、代わりに典型的なパターンを具体例をあげて説明していくことにします。経験上、込み入ったマルチスレッドプログラミングをするのでなければ、この記事であげた例だけ知っていればあまり困らないのではないかと思っています。

## 原則

マルチスレッドなプログラムを書くとき、まずは以下のように考えてください:

> **原則として、メモリ上の同じ位置に複数の goroutine からアクセスしてはならない**

メモリ上の同じ位置というのは例えば以下のようなものです:

- 同じ変数
- 同じ配列の同じインデックス
- 同じ struct の同じフィールド

例として、次のプログラムを考えてみましょう。

```go
func main() {
    done := false // 完了フラグ

    go func() { // サブの goroutine を起動する
        time.Sleep(1 * time.Second) // 何かの処理
        done = true // 完了フラグを立てる
    }()

    // 完了するまで待つ
    for !done {
        time.Sleep(100 * time.Millisecond)
    }
}
```

サブの goroutine を立てて計算を行い、メインの goroutine でそれを待つという単純なプログラムですが、バグがあります。

`done` という変数に注目してください。 `done` はサブの goroutine から write されており、かつメインの goroutine から read されています。よって「メモリ上の同じ位置に複数の goroutine からアクセスしてはならない」に違反しています。実際、 `go run -race` で実行してみるとデータ競合が検出されます [^1] 。

しかし、この原則に従うと goroutine 間で全くデータを共有できないということになってしまいます。それでは困りますね。実際には、ある条件を満たせば同じデータにアクセスすることができます。そこで、そのような条件を満たすパターンのうち、代表的なものを3つ紹介していきます。

## 大丈夫なパターン1: 共有されるデータが immutable であるとき

実は、データ競合は同じメモリ位置に並列なアクセスがあり、かつ **少なくとも一方が write である場合** にのみ発生します。言い換えると、すべてのアクセスが read であるときは安全であるということです。immutable なデータはまさにその条件を満たすため、それへのアクセスは常に安全であると考えてよいです。

例として次のコードを見てみましょう。 `upstreamURL` が10個の goroutine から並列にアクセスされています。しかし、それらのアクセスはすべて read であり、誰も `upstreamURL` に write していません。よってこれは安全です。

```go
func main() {
    upstreamURL := "http://localhost:8080/hello"

    // 10並列で http.Get する
    for i := 0; i < 10; i++ {
        go func() {
            _, err := http.Get(upstreamURL) // ← upstreamURL を参照している
            log.Print(err)
        }()
    }
}
```

マルチスレッドプログラミングに限らず、immutable なデータ構造はバグを減らす上で非常に役に立ちます。immutable にできるものは積極的に immutable にしていくのがよいでしょう。

## 大丈夫なパターン2: Mutex で保護しているとき

immutable なデータは何もしなくても安全でしたが、mutable なデータは明示的な保護が必要です。やり方はいくつかありますが、最も典型的なのは `sync.Mutex` を使うことです。Mutex は排他制御を行うための仕組みです。Mutex を知らない人は [Tour of Go の該当のセクション](https://go.dev/tour/concurrency/9) などで学ぶとよいでしょう。

以下のコードは冒頭であげた例に Mutex を追加したものです。 `cache` に関するデータ競合を解消するために、 `cache` への read/write を Mutex の Lock/Unlock で囲っています。

```go
var (
    // cache を保護するための mutex
    mutex sync.Mutex
    // 結果をキャッシュしておくための map
    cache = map[int]int{}
)

func doExpensiveCalculation(w http.ResponseWriter, r *http.Request) {
    // クエリパラメータとして与えられた x を取得する
    x, err := strconv.Atoi(r.URL.Query().Get("x"))
    if err != nil {
        http.Error(w, "invalid argument", http.StatusBadRequest)
        return
    }

    mutex.Lock()
    defer mutex.Unlock()

    // 結果がキャッシュされていればそれを返す
    if cached, ok := cache[x]; ok {
        fmt.Fprintf(w, "The answer is %v (cached)\n", cached)
        return
    }

    answer := x * x   // 複雑で時間のかかる計算をしていると思ってください
    cache[x] = answer // 計算結果をキャッシュする
    fmt.Fprintf(w, "The answer is %v\n", answer)
}
```

`mutex.Lock()` から `mutex.Unlock()` (defer で実行されるので関数の最後)の間には、同時にひとつの goroutine しか入ることができません。これにより、 `cache` に read と write が同時には行われなくなり、データ競合は解消されました。実際、この修正により `go test -race` してもデータ競合は検出されなくなりました。

ひとつ注意すべき点として、Mutex を使うときは **read と write 両方** を排他しないといけません。write だけを排他すれば十分と思っている人がいますが、それは誤解です。おそらく read 同士はデータ競合しないという条件を拡大解釈してしまっているのだと思いますが、read と write は競合するので write 同士の競合だけを防いでも不十分です。

例えば、上の例で次のように `cache` への write のみを Mutex で排他してもデータ競合は防げません。

```go
func doExpensiveCalculation(w http.ResponseWriter, r *http.Request) {
    /* 中略 */

    // 結果がキャッシュされていればそれを返す
    if cached, ok := cache[x]; ok {
        fmt.Fprintf(w, "The answer is %v (cached)\n", cached)
        return
    }

    answer := x * x // 複雑で時間のかかる計算をしていると思ってください

    mutex.Lock() // cache への write だけを保護 (← 間違い)
    cache[x] = answer // 計算結果をキャッシュする
    mutex.Unlock()

    fmt.Fprintf(w, "The answer is %v\n", answer)
}
```

read と write が競合するというのがイメージしにくい人は、Go の気持ちになって `cache[x] = answer` が実際には何をしているのかを考えてみるとよいでしょう。マップの内部は複雑なデータ構造になっており、それに値を書き込むためには内部のポインタを書き換えたり、メモリを確保したりと、いろいろな作業が必要です。それを行っている最中に別の goroutine が同じマップを読み取ると、中途半端な状態のデータ構造を読み取ってしまうかもしれません。

また、map だけではなく `int` や `bool` のような単純なデータであっても CPU によるキャッシュやコンパイラによる最適化などによって read と write が競合することがあります。したがって、原則としてメモリ上の同じ位置に複数の goroutine からアクセスしてはならないということを忘れないようにしましょう。

## 大丈夫なパターン3: スレッドセーフなデータの場合

Go の標準ライブラリには複数の goroutine から安全に読み書きできるデータ構造があります。例えば `atomic.Bool` や `atomic.Int64` のようなアトミック型や、 `sync.Map` のようなデータ構造などが挙げられます。また、チャネル(`chan T`)もそのようなデータ構造であると考えることができます。

例えば `atomic.Bool` を使うと、上で出てきた goroutine の終了をループで待つ例は以下のように書き直すことができます。こうするとデータ競合は発生しなくなります。

```go
func main() {
    var done atomic.Bool // atomic.Bool のゼロ値は false

    go func() { // サブの goroutine を起動する
        time.Sleep(1 * time.Second) // 何かの処理
        done.Store(true)            // 完了フラグを立てる
    }()

    // 完了するまで待つ
    for !done.Load() {
        time.Sleep(100 * time.Millisecond)
    }
}
```

※ goroutine の終了を待つ場合、チャネルの close を待つように書くほうが簡潔ですし効率もよいです。この例はあくまで説明のためのコードであると考えてください。

しかし、このようなデータ構造を使う場合、十分な注意が必要です。データ構造自体がスレッドセーフでも、データ構造の外でデータ競合を発生させてしまうことがよくあるからです。以下の例を見てください。これは `sync.Map` を使って API の統計情報を記録しているコードです。

```go
// API ごとに統計情報を記録するための map
// 複数の goroutine からアクセスされるため sync.Map を使っている
var statsMap sync.Map // map[APIName]*Stats

type Stats struct {
    Success uint // 成功回数
    Failure uint // 失敗回数
}

// doSomething は並列に実行されることを想定した関数
func doSomething() {
    // なんらかの処理を行っていると思ってください

    statsAny, _ := statsMap.LoadOrStore("doSomething", &Stats{})
    stats := statsAny.(*Stats)
    stats.Success++ // 成功カウントを増やす (← これは大丈夫か？)
}
```

残念ながらデータ競合が発生しています。 `sync.Map` が使われているので、 `statsMap` に対する読み書きは安全です。しかし、 `statsMap` から取得されたのは **ポインタ** です。このポインタを使って複数の goroutine が write を行うとデータ競合が発生してしまいます。今回の場合、回数を正しくカウントできていないといった現象が発生することでしょう。つまり、 `sync.Map` を使ってもデータを取得したあとに起こるデータ競合は防げないわけです。

この例の場合、 `stats.Success` や `stats.Failure` への read/write をスレッドセーフにしなければなりません。例えば `atomic.Uint64` を使うと以下のように書けます。こうすることでデータ競合を解消でき、回数を正しくカウントできるようになります。

```go
// API ごとに統計情報を記録するための map
// 複数の goroutine からアクセスされるため sync.Map を使っている
var statsMap sync.Map // map[APIName]*Stats

type Stats struct {
    Success atomic.Uint64 // 成功回数
    Failure atomic.Uint64 // 失敗回数
}

// doSomething は並列に実行されることを想定した関数
func doSomething() {
    // なんらかの処理を行っていると思ってください

    statsAny, _ := statsMap.LoadOrStore("doSomething", &Stats{})
    stats := statsAny.(*Stats)
    stats.Success.Add(1) // 成功カウントを増やす
}
```

もちろん、 `atomic` を使う代わりに `stats` への読み書きを Mutex で排他するという方法でもスレッドセーフにできます。

ここでは `sync.Map` にポインタを格納する例を使って危険なパターンを説明しましたが、危険なのは `sync.Map` に限った話ではありません。例えば、チャネルでポインタをやり取りした場合でも同じ現象が発生します。よって、チャネルを使うときも

- チャネルを通じてポインタをやり取りするのは避ける
- それが避けられない場合、ポインタの指す先を immutable にする
- それすら無理な場合、Mutex などを使ってアクセスを排他する

のようにデータ競合を防ぐことを意識しておきましょう。

## おわりに

Go に慣れた人であっても、データ競合は不意に起こしてしまうものです。この記事が少しでもそのような問題を防ぐ一助となれば幸いです。

---

### あわせて読みたい

- [Go で新しいサービスを実装する際に意識したポイント](https://blog.cybozu.io/entry/2025/04/14/100000)
- [23新卒エンジニアがチーム開発研修で学んだこと](https://blog.cybozu.io/entry/2023/09/15/080000)
- [サイボウズサマーインターン2021 報告 〜 Kubernetes基盤開発コース](https://blog.cybozu.io/entry/2021/09/10/170132)
- [Goの静的解析ツールをgolintからstaticcheckに移行した話](https://blog.cybozu.io/entry/2021/02/26/081013)
- [分散システムの耐障害性テストの取り組み](https://blog.cybozu.io/entry/2018/09/06/080000)

[« Storybook をフル活用してテストを実装し…](https://blog.cybozu.io/entry/2024/08/13/140000) [E2Eテストの部分実行によるテスト時間短縮 »](https://blog.cybozu.io/entry/2024/08/13/110000)

[^1]: 筆者の環境では、このプログラムを実行してもクラッシュしたり無限ループしたりはしませんでした。データ競合が起こっているから必ずクラッシュするというわけではないのです。しかし、コンパイラを更新して最適化の方法が変化したり、実行する CPU が変わってメモリアクセスが変化したりすると、問題が発生する可能性は十分にあります。実際、我々の本番環境でも gcc のバージョンの更新によって重大な障害を引き起こすデータ競合が顕在化したことがあります。あらゆる意味での再現性のなさがデータ競合の怖さです。