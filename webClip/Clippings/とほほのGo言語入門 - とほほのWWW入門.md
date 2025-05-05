---
title: とほほのGo言語入門 - とほほのWWW入門
source: https://www.tohoho-web.com/ex/golang.html#print
author: 
published: 
created: 2025-05-06
description: 
tags:
  - 1
  - "#Go"
read: false
---
## とほほのGo言語入門

目次

- [概要](https://www.tohoho-web.com/ex/#overview)
	- [Go言語とは](https://www.tohoho-web.com/ex/#about)
	- [バージョン](https://www.tohoho-web.com/ex/#versions)
	- [インストール](https://www.tohoho-web.com/ex/#install)
	- [Hello world](https://www.tohoho-web.com/ex/#hello-world)
	- [Print・Println・Printf](https://www.tohoho-web.com/ex/#print)
	- [変数(var)](https://www.tohoho-web.com/ex/#variables)
	- [定数(const)](https://www.tohoho-web.com/ex/#const)
	- [コメント](https://www.tohoho-web.com/ex/#comments)
	- [行末のセミコロン](https://www.tohoho-web.com/ex/#semicolons)
	- [キーワード](https://www.tohoho-web.com/ex/#keywords)
- [演算子](https://www.tohoho-web.com/ex/#operators)
- [型(type)](https://www.tohoho-web.com/ex/#types)
	- [型変換](https://www.tohoho-web.com/ex/#type-convert)
	- [リテラル・値](https://www.tohoho-web.com/ex/#literals)
	- [エスケープシーケンス](https://www.tohoho-web.com/ex/#escape-sequence)
	- [配列(array)](https://www.tohoho-web.com/ex/#array)
	- [スライス(slice)](https://www.tohoho-web.com/ex/#slice)
	- [マップ(map)](https://www.tohoho-web.com/ex/#map)
- [制御構文](https://www.tohoho-web.com/ex/#control-syntax)
	- [If文(if)](https://www.tohoho-web.com/ex/#if)
	- [Switch文(switch)](https://www.tohoho-web.com/ex/#switch)
	- [For文(for)](https://www.tohoho-web.com/ex/#for)
	- [Goto文(goto)](https://www.tohoho-web.com/ex/#goto)
- [関数(func)](https://www.tohoho-web.com/ex/#functions)
- [構造体(struct)](https://www.tohoho-web.com/ex/#structures)
- [インタフェース(interface)](https://www.tohoho-web.com/ex/#interfaces)
- [interface {}型](https://www.tohoho-web.com/ex/#interface-any)
- [ポインタ(pointer)](https://www.tohoho-web.com/ex/#pointer)
- [領域確保(new)](https://www.tohoho-web.com/ex/#new)
- [遅延実行(defer)](https://www.tohoho-web.com/ex/#defer)
- [インポート(import)](https://www.tohoho-web.com/ex/#import)
- [モジュール(module)](https://www.tohoho-web.com/ex/#module)
- [パッケージ(package)](https://www.tohoho-web.com/ex/#package)
- [ワークスペース(workspace)](https://www.tohoho-web.com/ex/#workspace)
- [ゴルーチン(Goroutine)](https://www.tohoho-web.com/ex/#goroutines)
- [リンク](https://www.tohoho-web.com/ex/#links)

## 概要

### Go言語とは

- Google が開発したプログラミング言語です。「 **Go言語** 」や「 **Golang** 」と表記されます。
- UNIX、B言語(C言語の元)、UTF-8の開発者ケン・トンプソンや、UNIX、Plan 9、UTF-8の開発者ロブ・パイクによって設計されました。
- 静的型付け、メモリ安全性、ガベージコレクションを備えるコンパイル言語です。
- シンプル、高速、メモリ効率が良い、メモリ破壊が無い、並行処理が得意などの特徴を備えています。
- メモリ破壊が無く、並行処理を得意とする、進化したC言語という側面があります。
- Linux、Mac OS X、Windows、Android、iOS で動作します。

### バージョン

おおよそ半年に一度バージョンアップを行っているようです。このページは Go 1.14 をターゲットに記述しています。

- [Go 1.18](https://go.dev/blog/go1.18) 2022年3月15日  
	速度改善、ジェネリクス、ファジングテストツール、ワークスペースモード、any型、~演算子、comparable識別子など。
- [Go 1.17](https://go.dev/blog/go1.17) 2021年8月16日  
	Windows/arm64 サポート、など。
- [Go 1.16](https://go.dev/blog/go1.16) 2021年2月16日
- [Go 1.15](https://go.dev/blog/go1.15) 2020年8月11日
- [Go 1.14](https://go.dev/blog/go1.14) 2020年2月25日
- [Go 1.13](https://go.dev/blog/go1.13) 2019年9月3日
- [Go 1.12](https://go.dev/blog/go1.12) 2019年2月25日
- [Go 1.11](https://go.dev/blog/go1.11) 2018年8月24日
- [Go 1.10](https://go.dev/blog/go1.10) 2018年2月16日
- [Go 1.9](https://go.dev/blog/go1.9) 2018年8月24日
- [Go 1.8](https://go.dev/blog/go1.8) 2017年2月16日
- [Go 1.7](https://go.dev/blog/go1.7) 2016年8月15日
- [Go 1.6](https://go.dev/blog/go1.6) 2016年2月17日
- [Go 1.5](https://go.dev/blog/go1.5) 2015年8月19日
- [Go 1.4](https://go.dev/blog/go1.4) 2014年12月10日
- [Go 1.3](https://go.dev/blog/go1.3) 2014年6月18日
- [Go 1.2](https://go.dev/doc/go1.2) 2013年12月1日
- [Go 1.1](https://go.dev/doc/go1.1) 2013年5月13日
- [Go 1.0](https://go.dev/doc/go1) 2012年3月28日

### インストール

ちょっとだけ試してみるには [The Go Playground](https://play.golang.org/) も使用できますが、ローカル環境にインストールするには下記の通り。

```c
# CentOS 7
# yum install -y epel-release
# yum install -y golang
# go version
go version go1.18.9 linux/amd64

# CentOS/RockyLinux/Alma Linux 8
# dnf install -y golang
# go version
go version go1.18.9 linux/amd64

# Ubuntu 20.04 LTS
$ sudo apt-get install -y golang
$ go version
go version go1.13.8 linux/amd64

# Ubuntu 22.04 LTS
$ sudo apt-get install -y golang
# go version
go version go1.18.1 linux/amd64

# Linux(最新版) from https://golang.org/dl/
# 1.20をインストールする例
# wget https://go.dev/dl/go1.20.2.linux-amd64.tar.gz
# tar -C /usr/local -xzf go1.20.2.linux-amd64.tar.gz
# echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
# source ~/.bashrc
# go version
go version go1.20.2 linux/amd64
```

### Hello world

プログラムの拡張子は.go とします。プログラムは main パッケージの main 関数から実行されます。

```c
package main        // mainパッケージであることを宣言

import "fmt"        // fmtモジュールをインポート

func main() {        // 最初に実行されるmain()関数を定義
    fmt.Println("hello, world")
}
```

直接実行するには **go run** コマンドを使用します。

```c
$ go run sample.go
Hello, world!
```

コンパイルするには **go build** コマンドを使用します。実行形式のファイル sample が作成されます。

```c
$ go build sample.go
$ ./sample
Hello, world!
```

ソースを標準のコーディングスタイルに整形するには **gofmt** を使用します。標準スタイルでは、インデントはスペースではなくタブ文字を使用します。

```c
$ gofmt sample.go
```

### Print・Println・Printf

**fmt.Print()** は引数を文字列として出力します。 **fmt.Println()** は引数の間にスペースを入れ、最後に改行文字 "\\n" を出力します。 **fmt.Printf()** は %d (数値) や %s (文字列) などのフォーマットを指定して引数を出力することができます。

```c
num := 123
str := "ABC"

fmt.Print("num=", num, " str=", str, "\n")    // 改行無し、空白無し、フォーマット無し
fmt.Println("num =", num, "str =", str)    // 改行有り、空白有り、フォーマット無し
fmt.Printf("num=%d str=%s\n", num, str)    // 改行無し、空白無し、フォーマット有り
```

Printf() のフォーマットには下記を使用できます。%4d とすると 4桁整数、%04d とすると 0埋め4桁整数で出力します。

**%v** (デフォルト形式)、 **%#v** (Go言語表記)、 **%t** (真偽値)、 **%d** (整数)、 **%s** (文字列)、 **%c** (文字)、 **%f** (小数)、 **%F** (小数)、 **%e** (浮動小数点数e)、 **%E** (浮動小数点数E)、 **%g** (%f/%e自動選択)、 **%b** (2進数)、 **%o** (8進数)、 **%O** (0o付き8進数)、 **%x** (16進数)、 **%X** (16進数大文字)、 **%U** (Unicode)、 **%p** (ポインタ)、 **%q** ("..."付き文字列)、 **%T** (型表示)、 **%%** (パーセント)

### 変数(var)

**var** は 「var 変数名 型名」の形式で変数を定義します。

```c
var a1 int
```

初期値を指定することができます。初期値を省略すると 0 や空文字列 "" などで初期化されます。

```c
var a1 int = 123
```

初期値により型名が明白な場合は型名を省略することもできます。

```c
var a2 = 123
```

初期値を指定する場合は、**:=** を用いると **var** も省略することができます。

```c
a3 := 123
```

下記の様にまとめて定義することもできます。

```c
var (
    a1 int = 123
    a2 int = 456
)
```

演算子 **\=** は、右辺の値を左辺の変数に代入します。

```c
a1 = 456
```

複数の値を同時に代入することもできます。

```c
name, age = "Yamada", 26
```

### 定数(const)

**const** は定数を定義します。定数の場合は型名は省略可能で、多くの場合指定しません。

```c
const foo = 100
```

下記の様にまとめて定義することもできます。

```c
const (
    foo = 100
    baa = 200
)
```

**//...** 形式、または、 **/\*... \*/** 形式のコメントを書くことができます。コメントはコンパイルの際に無視されます。

```c
// ここはコメントです
/* ここも
   コメントです */
```

### 行末のセミコロン

行末にはセミコロン(**;**)を書きますが、ほとんどの場合省略することができます。セミコロンを書くことで複数の文を1行に記述することができます。

```c
num = 123; str = "ABC";
```

### キーワード

```c
break        default      func         interface    select
case         defer        go           map          struct
chan         else         goto         package      switch
const        fallthrough  if           range        type
continue     for          import       return       var
```

## 演算子

```c
x + y        加算 (文字列の連結にも利用)
x - y        減算
x * y        乗算
x / y        除算
x % y        除算の余り
x & y        論理積(AND)
x | y        論理和(OR)
x ^ y        排他的論理和(XOR)
x &^ y        x AND (NOT y)
x << y        yビット左にシフト
x >> y        yビット右にシフト
x = y        xにyを代入
x := y        xにyを代入(初期化の使用可能)
x++        x = x + 1 と同義
x--        x = x - 1 と同義
x += y        x = x + y と同義
x -= y        x = x - y と同義
x *= y        x = x * y と同義
x /= y        x = x / y と同義
x %= y        x = x % y と同義
x &= y        x = x & y と同義
x |= y        x = x | y と同義
x ^= y        x = x ^ y と同義
x &^= y        x = x &^ y と同義
x <<= y        x = x << y と同義
x >>= y        x = x >> y と同義
x && y        xかつy(AND)
x || y        xまたはy(OR)
!x        xがtrueの場合false/falseの場合true(NOT)
x == y        xとyが等しければ
x != y        xとyが等しくなければ
x < y        yがxより大きければ
x <= y        yがx以上であれば
x > y        yがxより小さければ
x >= y        yがx以下であれば
ch <- x        chチャネルにxを送信
x =<- ch    chチャネルからxに受信
```

## 型(type)

```c
bool                真偽値(true or false)
int8/int16/int32/int64        nビット整数
uint8/uint16/uint32/uint64    nビット非負整数
float32/float64        nビット浮動小数点数
complex64/complex128        nビット虚数
byte                1バイトデータ(uint8と同義)
rune                1文字(int32と同義)
uint                uint32 または uint64
int                int32 または int64
uintptr                ポインタを表現するのに十分な非負整数
string                文字列
```

下記の様に型に別名をつけることができます。型の異なる値を代入することはできません。

```c
type UtcTime string        // string型の別名 UtcTime を定義
type JstTime string        // string型の別名 JstTime を定義
var t1 UtcTime = "00:00:00"
var t2 JstTime = "09:00:00"
t1 = t2                // 型が異なるので代入エラー
```

下記の様にまとめて定義することもできます。

```c
type (
    UtcTime string
    JstTime string
)
```

### 型変換

**型名()** で型変換を行うことができます。

```c
var a1 uint16 = 1234
var a2 uint32 = uint32(a1)
```

### リテラル・値

```c
nil        無しを示す特別な値
true        真偽値の真
false        真偽値の偽
1234        整数
1_234        整数(カンマ区切りの代わりに_を使用。_は無視される)
0777        8進数
0o755        8進数(0Oも可)
0x89ab        16進数(0Xも可)
0b1111        2進数(0Bも可)
123.4        小数
1.23e4        浮動小数点数(1.23E4も可)
1.23i        虚数
"ABC"        文字列
'A'        文字(rune)
```

### エスケープシーケンス

文字列や文字の中では下記のエスケープシーケンスを使用することができます。

```c
\a        ベル(U+0007)
\b        バックスペース(U+0008)
\t        タブ(U+0009)
\n        改行(U+000A)
\v        垂直タブ(U+000B)
\f        フォームフィード(U+000C)
\r        キャリッジリターン(U+000D)
\"        ダブルクォート(U+0022)
\'        シングルクォート(U+0027)
\\        バックスラッシュ(U+005C)
\x42        ASCII文字(U+0000～U+00FF)
\u30A2        Unicode(U+0000～U+FFFF)
\U0001F604    Unicode(U+0000～U+10FFFF)
```

### 配列(array)

コンパイル時に個数が決まっていて変更不可のものを **配列** と言います。型名の前に **\[**個数**\]** をつけて宣言します。配列のインデックスは 0 からはじまります。途中で個数を変更することはできませんが、メモリ効率や性能の点で優れています。

```c
a1 := [3]string{}
a1[0] = "Red"
a1[1] = "Green"
a1[2] = "Blue"
fmt.Println(a1[0], a1[1], a1[2])
```

初期化時に値を設定することもできます。

```c
a1 := [3]string{"Red", "Green", "Blue"}
```

初期化によって個数が決まる場合は、個数を **...** と省略することができます。

```c
a1 := [...]string{"Red", "Green", "Blue"}
```

### スライス(slice)

メモリ効率や速度は若干落ちますが、個数を変更可能なものを **スライス** と呼びます。型名の前に **\[\]** をつけて宣言します。スライスには **append()** を用いて要素を追加します。

```c
a1 := []string{}            // スライス。個数不定
a1 = append(a1, "Red")
a1 = append(a1, "Green")
a1 = append(a1, "Blue")
fmt.Println(a1[0], a1[1], a1[2])
```

**len()** は配列やスライスの長さ(length)、 **cap()** は容量(capacity)を求めます。長さは実際に使用されている数、容量はメモリ上に確保されている数です。容量を超えると、倍の容量のメモリが別に確保され、既存データがコピーされます。

```c
a := []int{}
for i := 0; i < 10; i++ {
    a = append(a, i)
    fmt.Println(len(a), cap(a))
}
```

スライスの場合、 **make(**スライス型, 初期個数, 初期容量**)** を用いたメモリの確保ができます。初期容量を省略した場合は初期個数と同じ容量が確保されます。容量をあらかじめ確保しておくことで、容量超過時の再確保を減らして速度を速めることができます。

```c
bufa := make([]byte, 0, 1024)
```

### マップ(map)

「 **map\[**キーの型**\]** 値の型」 を用いて連想配列のようなマップを使用することができます。

```c
// マップを定義する
a1 := map[string]int{
    "x": 100,
    "y": 200,            // 改行する場合はカンマ必須
}

// マップを参照する
fmt.Println(a1["x"])

// マップに要素を加える
a1["z"] = 300

// マップの要素を削除する
delete(a1, "z")

// マップの長さを求める
fmt.Println(len(a1))

// マップに要素が存在するかどうかを調べる
_, ok := a1["z"]
if ok {
    fmt.Println("Exist")
} else {
    fmt.Println("Not exist")
}

// マップをループ処理する
for key, value := range a1 {
    fmt.Printf("%s=%d\n", key, value)
}
```

## 制御構文

### If文(if)

「 **if** 条件 **{** 処理 **}** 」 は条件が真の時のみ処理を実行します。条件を (...) で囲む必要はありません。{... } の中括弧は必須です。

```c
if x > y {
    fmt.Println("x is greater then y")
}
```

**else** や **else if** を使用することができます。

```c
if x > y {
    fmt.Println("Big")
} else if x < y {
    fmt.Println("Small")
} else {
    fmt.Println("Equal")
}
```

### Switch文(switch)

「 **switch** 式 **{**... **}** 」は、式の値によって処理を振り分けます。

```c
switch mode {
case "running":
    fmt.Println("実行中")
case "stop":
    fmt.Println("停止中")
default:
    fmt.Println("不明")
}
```

「 **switch** **{**... **}** 」では、 **case** 分で条件を記述することもできます。

```c
switch {
case x > y:
    fmt.Println("Big")
case x < y:
    fmt.Println("Small")
default:
    fmt.Println("Equal")
}
```

他の言語にあるような **break** 文は必要ありません。逆に、次の条件の処理も実行するには **fallthrough** を用います。下記の例は dayOfWeek が "Sat" または "Sun" であれば "Horiday" を返します。

```c
switch dayOfWeek {
case "sat":
    fallthrough
case "sun":
    fmt.Println("Horiday")
default:
    fmt.Println("Weekday")
}
```

### For文(for)

Go言語には **while** 文が無く、繰り返し処理はすべて **for** を用います。下記の例は x が y よりも小さい間、処理を繰り返します。

```c
for x < y {
    x++
}
```

「 **for** 開始処理**;** 条件**;** 後処理 **{** 処理 **}** 」は、最初に開始処理を行い、条件が真の間、処理と後処理を繰り返し実行します。

```c
for i := 0; i < 10; i++ {
    fmt.Println(i)
}
```

条件を省略すると無限ループになります。 **continue** は次のループを繰り返します。 **break** はループを抜けます。

```c
n := 0
for {
    n++
    if n > 10 {
        break
    } else if n % 2 == 1 {
        continue
    } else {
        fmt.Println(n)
    }
}
```

配列やスライスなどイテラブルなものに対しては **range** を用いてループ処理することができます。

```c
colors := [...]string{"Red", "Green", "Blue"}

for i, color := range colors {
    fmt.Printf("%d: %s\n", i, color)
}
```

### Goto文(goto)

**goto** 文は指定したラベルにジャンプします。Go言語には **try** **catch** **raise** のような例外処理構文はサポートされていませんので、似たようなことをやるとすると下記の様になります。

```c
package main

import (
    "fmt"
    "errors"
)

func main() {
    funcA()
}

func funcA() (string, error) {
    var err error
    filename := ""
    data := ""

    filename, err = GetFileName()
    if err != nil {
        fmt.Println(err)
        goto Done
    }

    data, err = ReadFile(filename)
    if err != nil {
        fmt.Println(err)
        goto Done
    }

    fmt.Println(data)

Done:
    return data, err
}

func GetFileName() (string, error) {
    return "sample.txt", nil
}

func ReadFile(filename string) (string, error) {
    return "Hello world!", errors.New("Can't read file")
}
```

## 関数(func)

**func** は関数を定義します。 **return** には関数の戻り値を指定します。

```c
func add(x int, y int) int {
    return x + y
}

func main() {
    fmt.Println(add(5, 3))        // => 8
}
```

複数の値を返却することもできます。複数の場合は型名は (...) で囲む必要があります。

```c
func addMinus(x int, y int) (int, int) {
    return x + y, x - y
}

func main() {
    add, min := addMinus(8, 5)
    fmt.Println(add, min)
}
```

複数の値を返却する関数などで、不要な戻り値がある場合は、ブランク変数 **\_** を使用することができます。

```c
// funcA() は x と y を返却するが、x は無視して y のみ受け取る
_, y := funcA()
```

**...** を用いることで可変引数を実現することができます。

```c
func funcA(a int, b ... int) {
    fmt.Printf("a=%d\n", a)
    for i, num := range b {
        fmt.Printf("b[%d]=%d\n", i, num)
    }
}

func main() {
    funcA(1, 2, 3, 4, 5)
}
```

## 構造体(struct)

Go言語では、クラス(class)の代わりに **構造体(struct)** を使用します。構造体にはメンバ変数のみを定義し、クラスメソッドに相当する関数は関数名の前に **(**thisに相当する変数 **\*** 構造体名**)** をつけて定義します。

```c
type Person struct {
    name string
    age int
}

func (p *Person) SetPerson(name string, age int) {
    p.name = name
    p.age = age
}

func (p *Person) GetPerson() (string, int) {
    return p.name, p.age
}

func main() {
    var p1 Person
    p1.SetPerson("Yamada", 26)
    name, age := p1.GetPerson()
    fmt.Printf("%s(%d)\n", name, age)
}
```

構造体のメンバの内、大文字で始まるものはパッケージ外からアクセス可能、小文字で始まるものはパッケージ外からアクセス不可となります。

```c
type Person struct {
    Name string    // 外部からアクセス可能
    Age int        // 外部からアクセス可能
    status int        // 外部からアクセス不可
}
```

構造体を使用する際は、下記の様にパラメータを初期化することができます。

```c
a1 := Person{"Yamada", 26}            // 順序通りに初期化
a2 := Person{name: "Tanaka", age: 32}        // 名前で初期化
```

## インタフェース(interface)

**インタフェース(interface)** はポリモーフィズムを実装するための機能です。下記の例では構造体 Person も、構造体 Book も ToString() というメソッドと PrintOut() というメソッドを実装しています。

```c
type Person struct {
    name string
}
func (p Person) ToString() string {
    return p.name
}
func (p Person) PrintOut() {
    fmt.Println(p.ToString())
}

type Book struct {
    title string
}
func (b Book) ToString() string {
    return b.title
}
func (b Book) PrintOut() {
    fmt.Println(b.ToString())
}

func main() {
    a1 := Person {name: "山田太郎"}
    a2 := Book {title: "吾輩は猫である"}
    a1.PrintOut()
    a2.PrintOut()
}
```

ToString() は中身が異なるので仕方ないですが、PrintOut() は中身も同じなので、インタフェースを用いてひとつの関数にまとめることができます。Printable というインタフェースは、ToString() というメソッドをサポートしている構造体であれば、自動的に適用することが可能となります。

```c
package main

import "fmt"

type Printable interface {
    ToString() string
}
func PrintOut(p Printable) {
    fmt.Println(p.ToString())
}

type Person struct {
    name string
}
func (p Person) ToString() string {
    return p.name
}

type Book struct {
    title string
}
func (b Book) ToString() string {
    return b.title
}

func main() {
    a1 := Person {name: "山田太郎"}
    a2 := Book {title: "吾輩は猫である"}
    PrintOut(a1)
    PrintOut(a2)
}
```

## interface {}型

関数の無いインタフェース **interface {}** は、 **any** 型のように使用することができます。下記の関数はどんな型の引数でも受け取ることができます。

```c
func funcA(a interface {}) {
    ...
}
```

**.(**型名**)** は、 **interface{}** 型の変数を他の型に変換します。

```c
func funcA(a interface{}) {
    fmt.Printf("%d\n", a.(int))
}
```

型変換の第二戻り値は、型変換可能かどうかを返します。オブジェクトが interface {... } で定義したインタフェースを実装しているかどうかを調べることができます。

```c
func PrintOut(a interface{}) {
    // aをPrintableインタフェースを実装したオブジェクトに変換してみる
    q, ok := a.(Printable)
    if ok {
        // 変換できたらそのインタフェースを呼び出す
        fmt.Println(q.ToString())
    } else {
        fmt.Println("Not printable.")
    }
}
```

**switch** 変数**.(type) {**... **}** によって、型を判断することもできます。

```c
func funcA(a interface{}) {
    switch a.(type) {
    case bool:
        fmt.Printf("%t\n", a)
    case int:
        fmt.Printf("%d\n", a)
    case string:
        fmt.Printf("%s\n", a)
    }
}
```

**interface {}** は **any** の様に使えるという特徴を生かし、任意の型の値を持つマップを定義することもできます。

```c
p1 := map[string]interface{} {
    "name": "Yamada",
    "age": 26,
}
```

下記の様にすれば階層構造を持つ Python の **dict** もどきを定義することができます。p1\["address"\]\["tel"\] のように参照できないのが残念ですが。

```c
type any interface{}
type dict map[string]any

p1 := dict {
    "name": "Yamada",
    "age": 26,
    "address": dict {
        "zip": "123-4567",
        "tel": "012-3456-7890",
    },
}
name := p1["name"]
tel := p1["address"].(dict)["tel"]    // anyをdictに変換してから参照
```

## ポインタ(pointer)

ポインタとは、変数が格納されているメモリのアドレスです。C言語と同様に、オブジェクトのポインタを参照するには **&** を、ポインタの中身を参照するには **\*** を用います。

```c
var a1 int        // int型変数a1を定義
var p1 *int;        // intへのポインタ変数p1を定義

p1 = &a1        // p1にa1のポインタを設定
*p1 = 123        // ポインタの中身(つまりa1)に123を代入
fmt.Println(a1)    // => 123
```

変数の値を渡すことを「値渡し」、変数のポインタを渡すことを「参照渡し」と呼びます。値渡しでは値のコピーしか渡していないので元の変数を変更することはできませんが、ポインタ渡しであれば関数の中で変数の値を変更することが可能となります。

```c
func main() {
    var a1 int = 123
    var a2 int = 123
    fn(a1, &a2)        // a1は値渡し、a2は参照渡し
    fmt.Println(a1, a2)    // => 123 456
}

func fn(b1 int, b2 *int) {
    b1 = 456
    *b2 = 456
}
```

演算子 **.** は、構造体のメンバ変数でも、ポインタが指し示す構造体のメンバ辺でもアクセスすることができます。

```c
a1 := Person{"Tanaka", 26}    // 構造体Personのオブジェクトa1を確保して初期化
p1 := &a1            // 構造体a1へのポインタをp1に格納
fmt.Println(a1.name)        // メンバ変数には左記のようにアクセス
fmt.Println((*p1).name)        // ポインタpの中身(後続体)のメンバ変数には左記のようにアクセス
fmt.Println(p1.name)        // ただし、Go言語ではこれを、左記のようにも記述できる
```

## 領域確保(new)

**new()** を用いて領域を動的に確保し、その領域へのポインタを得ることができます。確保した領域は参照されなくなった後にでガベージコレクションにより自動的に開放されます。

```c
type Book struct {
    title string
}

func main() {
    bookList := []*Book{}

    for i := 0; i < 10; i++ {
        book := new(Book)
        book.title = fmt.Sprintf("Title#%d", i)
        bookList = append(bookList, book)
    }
    for _, book := range bookList {
        fmt.Println(book.title)
    }
}
```

## 遅延実行(defer)

「 **defer** 処理」は、関数から戻る直前に処理を遅延実行します。リソースを忘れずに解放する際によく用いられます。

```c
func funcA() {
    fp, err := os.Open("sample.txt")
    if err != nil {
        return
    }
    defer fp.Close()

    for {
        ...
    }
```

## インポート(import)

**import** はパッケージをインポートします。

```c
import "fmt"
```

下記の様に複数のパッケージをインポートすることもできます。

```c
import (
    "os"
    "fmt"
)
```

## モジュール(module)

モジュール環境で開発を行うには下記の様にします。 **go mod init** を実行するとカレントディレクトリに **go.mod** ファイルが作成されます。

```c
$ mkdir hello
$ cd hello
$ go mod init hello
$ vi main.go
```

プログラムを作成します。

```c
package main

import "fmt"

func main() {
    fmt.Println("Hello world!")
}
```

プログラムを実行します。

```c
$ go run .
Hello world!
```

公開されたモジュールを利用するには下記の様に取り込みます。モジュールは 環境変数 **GOPATH** ディレクトリ（省略時は $HOME/go）に保存されます。

```c
# go get golang.org/x/example
go: downloading golang.org/x/example v0.0.0-20220412213650-2e68773dfca0
go: added golang.org/x/example v0.0.0-20220412213650-2e68773dfca0
```

プログラムからこれを利用します。

```c
package main

import "fmt"
import "golang.org/x/example/stringutil"

func main() {
    fmt.Println(stringutil.Reverse("Hello world!"))
}
```

実行します。

```c
$ go run .
!dlrow olleH
```

インポート時にパッケージの別名をつけることができます。これによりパッケージ名の重複の問題を回避することが可能です。

```c
import (
    "fmt"
    gstr "golang.org/x/example/stringutil"
)
```

## パッケージ(package)

自作パッケージの例を下記に示します。

```c
$HOME
 └ go
   └ src
     ├ sample.go
     └ local
       └ mypkg
         └ mypkg.go
```

mypkg.go ファイルを次の内容で作成します。 **package** でパッケージ名を宣言します。

```c
package mypkg

import "fmt"

func FuncA() {            // 大文字で始まるものは自動的にエクスポートされる
    fmt.Println("FuncA()")
}

func funcB() {            // 小文字で始まるものはエクスポートされない
    fmt.Println("funcB()")
}
```

sample.go ファイルを次の内容で作成します。大文字で始まる FuncA() は公開されているので使用できますが、小文字で始まる funcB() は非公開なので使用することができません。

```c
package main

import "local/mypkg"

mypkg.FuncA()        // 呼び出せる
mypkt.funcB()        // Error
```

## ワークスペース(workspace)

Go 1.18 からはワークスペース機能がサポートされています。複数のパッケージをワークスペースで管理します。

```c
# ワークスペースを作成する
$ mkdir workspace
$ cd workspace
$ go work init

# myappモジュールを作成する
$ mkdir myapp
$ cd myapp
$ go mod init example.com/myapp
# myapp.go を作成(後述)
$ cd ..
$ go work use ./myapp

# mypkgモジュールを作成する
$ mkdir mypkg
$ cd mypkg
$ go mod init example.com/mypkg
# mypkg.go を作成(後述)
$ cd ..
$ go work use ./mypkg

# 実行する
$ go run example.com/myapp
Hello world!
```

myapp.go

```c
package main

import "fmt"
import "example.com/mypkg"

func main() {
    fmt.Println(mypkg.Hello())
}
```

mypkg.go

```c
package mypkg

func Hello() string {
    return "Hello world!"
}
```

## ゴルーチン(Goroutine)

ゴルーチン(goroutine)はGo言語における並行処理を実現するもので、スレッドよりも高速に並行処理を実現することができます。下記の例では、メインの処理を実行しながら、並行して funcA() ゴルーチンを **go** により実行しています。

```c
func funcA() {
    for i := 0; i < 10; i++ {
        fmt.Print("A")
        time.Sleep(10 * time.Millisecond)
    }
}

func main() {
    go funcA()
    for i := 0; i < 10; i++ {
        fmt.Print("M")
        time.Sleep(20 * time.Millisecond)
    }
}
```

下記の例ではチャネルを用いてゴルーチンの終了を待ち合わせる例です。 **chan** はチャネルを生成します。 **<-** はチャネルにメッセージを送受信します。

```c
func funcA(chA chan <- string) {
    time.Sleep(3 * time.Second)
    chA <- "Finished"        // チャネルにメッセージを送信する
}

func main() {
    chA := make(chan string)    // チャネルを作成する
    defer close(chA)        // 使い終わったらクローズする
    go funcA(chA)        // チャネルをゴルーチンに渡す
    msg := <- chA        // チャネルからメッセージを受信する
    fmt.Println(msg)
}
```

次の例では **select** を用いて、ゴルーチン funcA() と funcB() 双方の待ち合わせを行います。

```c
func funcA(chA chan <- string) {
    time.Sleep(1 * time.Second)
    chA <- "funcA Finished"
}

func funcB(chB chan <- string) {
    time.Sleep(2 * time.Second)
    chB <- "funcB Finished"
}

func main() {
    chA := make(chan string)
    chB := make(chan string)
    defer close(chA)
    defer close(chB)
    finflagA := false
    finflagB := false
    go funcA(chA)
    go funcB(chB)
    for {
        select {
        case msg := <- chA:
            finflagA = true
            fmt.Println(msg)
        case msg := <- chB:
            finflagB = true
            fmt.Println(msg)
        }
        if finflagA && finflagB {
            break
        }
    }
}
```

## リンク

- [本家サイト (英語)](https://golang.org/)

---