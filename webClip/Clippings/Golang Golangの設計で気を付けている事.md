---
title: "[Golang] Golangの設計で気を付けている事"
source: https://qiita.com/kishibashi3/items/a244c4e4b42684bcd801
author:
  - "[[Qiita]]"
published: 2020-05-11
created: 2025-05-06
description: Golangの設計の際に個人的に気を付けている事をメモしておきます。比較的大きめなWeb Applicationがターゲットです。小さなツールだとまた違うかもしれません。他にも「私はこんなことに…
tags:
  - Go
  - 1
  - 設計
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から3年以上が経過しています。

Golangの設計の際に個人的に気を付けている事をメモしておきます。

比較的大きめなWeb Applicationがターゲットです。小さなツールだとまた違うかもしれません。  
他にも「私はこんなことに気を付けています！」的な情報歓迎します！

Golangの設計指針というと、Clean Architectureが独り歩きしていて、具体的な指針がない印象です。  
そういうモノがもっとあるといいな、と個人的に思います。

## 1\. 構造に関するもの

## 1-1. Clean Architecture

もちろんこれ。Golangの標準構成の指針です。パッケージ構成、インターフェースの使い方、パッケージ間の依存関係など、必要な事が書かれていますね。

## 1-2. インターフェースによる依存

Golangモジュール間参照は必ずインターフェースを使いましょう。  
Clean Architecture的にも必須なのはもちろんですが、単体テストでモックに差し替える事ができなくなります！

## 1-3. DI

モジュール間参照は必ずモジュールのコンストラクタで渡せるようにしましょう。  
そして、main.go（というよりロジックの外側）で依存性を注入しましょう。

## 1-4. 関数か、クロージャか、モジュールか

まず関数、次にクロージャを検討しましょう。  
インターフェースを用いたモジュールのDIよりも、関数typeを用いたDIのほうがシンプルです。

何らかの処理（メソッド）を超えて状態を持ち続ける必要があり、かつ処理（メソッド）のセットがある場合にはモジュール（struct）にしましょう。

## 1-5. パッケージ間の双方向依存を避ける

Clean Architecture的にもですが、cyclic import問題を発生させないためにも、パッケージ間依存の方向性は必ず一方向になるようにあらかじめ依存方向を定義しましょう。

## 1-6. 環境依存性

local環境、dev環境、stage環境、prod環境など、環境ごとに動作を替えたい事があると思います。  
基本的には環境変数によって動作を変えますが、そうではなくコード自身を切り替えたい場合はbuild tagを使いましょう。

## 2\. モジュール設計

## 2-1. Interface

インターフェースに登場する値（enum定義やFunctional Optionなど）は、必ずインターフェース側に書きましょう。実装側に書くとcyclic importになりますよ！

> インターフェースは実装側ではなく「使う側のもの」だという事をお忘れなく！　

## 2-2. デフォルト値

Golangではstringやintなどにnullは存在しません。""や0がnull扱いになります。そのためデフォルト値の扱いにしばしば困ります。  
Functional Optionを使用しましょう。

## 2-3. 継承とオブジェクト指向

Golangはオブジェクト指向言語でないと言われますが、埋め込み型によって継承は可能です。

> 型を継承しないだけで、機能は継承します。

これを行ったとき、通常継承についてのアンチパターンで言われている事と同じリスクが発生します。  
埋め込み型の取り扱いには十分注意しましょう。

オブジェクト指向を知らないなら、埋め込み型を使うべきではありません。

## 2-4. 関数化

Golangは、関数を値として扱うのがとても楽な言語だと思います。積極的に関数インターフェースをtype宣言しましょう！

```go
type OptionalFunction func(*Option)

func hoge(a int, options ...OptionalFunction) err {
}
```

## 2-5. enum

golangには安全なenumは言語レベルでは用意されていません。  
enumクラスを作る事ができないではありませんが、type aliasを作成してconst定義するでいいと思います。

```go
type ErrorLevel int
const (
   Critical ErrorLevel = 0
   Error ErrorLevel = 1
   Warning ErrorLevel = 2
   Info ErrorLevel = 3   
)
// 個人的にはあまりiotaが好きじゃない。べた書きしたい。
```

const定義されていない値を作る事は簡単にできますので、必ず例外処理を書きましょう。

```go
switch (level) {
   case Critical:
      //pass
   case Error:
      //pass
   default:
      return nil, errors.New("illegal error level")
}
```

## 2-6. 型安全性

Golangは型安全な言語です。型安全性を活用することを心がけて設計しましょう。  
具体的には、down castはよほどの必要性がない限り避けましょう。

## 2-7. interface{}型

型安全性と逆の事を言っているようですが、interface{}型を活用するのも重要です。  
型を忘れていいときは積極的に忘れましょう。

型を忘れていいときとは、「忘れても思い出す必要がないとき」です。

## 2-8. context

golangの設計にあたって、contextの扱いは重要です。

- contextに付与するvalueは、contextと生存期間が一致するモノでなければなりません。
- contextは、時間的な範囲を表します。structの属性にすべきではありません。必ずメソッドの第一引数で受け渡しましょう。

## 3\. コーディング

## 3-1. 変数

### 変数を減らす

不要な変数は宣言しない。変数を可能な限り減らしましょう。

### constにする

golangにはjavascriptのようなconst/letがあるわけではありませんが、可能な限りlet的な変数を排除し、変数はconst的に扱いましょう。

## 3-2. productionコードにtestにしか使わないコードを紛れ込ませない

productionで不要なコードをif文で切り替えるのはやめてください。  
DIできるようにしておき、環境変数やbuild tagによって切り替えてください。

## 3-3. 明示的に書く

```py
if len(str(arg)) > 0:
```

これはpythonの例なのですが、これ一目見て何をやろうとしているか分かります？

コードは文章です。  
やろうとしている事をそのまま（直接的・明示的に）表現するコードを書きましょう。  
明示的でないコードは、他人が読んだとき、何がしたいのか読み取るのに無駄な時間が必要になりますし、バグが混入する可能性が高いと思います。

## 3-4. 中途半端な抽象化をしない

```go
func doSomething(arg string) error {
    if arg == "a" {
       // doSomething_a
    }
    if arg == "b" {
       // doSomething_b
    }
    if arg == "c" {
       // doSomething_c
    }
    if arg == "a" {
       // doSomething_a_2nd
    }
    if arg == "b" {
       // doSomething_b_2nd
    }
    if arg == "c" {
       // doSomething_c_2nd
    }
}
```

こんな関数を時々見ます。なぜ

```go
func doSomething_a() error {
   // doSomething_a
   // doSomething_a_2nd
}
func doSomething_b() error {
   // doSomething_b
   // doSomething_b_2nd
}
func doSomething_c() error {
   // doSomething_c
   // doSomething_c_2nd
}
```

こうしないんですか？  
doSomethingという関数は、a/b/cを抽象化したかったのはわかります。しかし、関数の中の処理が全く抽象化されていません。  
こういう場合はif文を使わずに書ける部分に絞って関数に切り出しましょう。

if文が多いコードは、「中途半端な抽象化」の病が発症しているサインだと思います。

## 3-5. collection

### setがない

Golangには、array、slice、mapしか存在しません。最も困るのがsetです。  
setを諦めると、もれなくfor文の2重ループ地獄が待っています。

setが必要なときは `map[?]struct{}` を使いましょう。  
ただ、list長が短いなら、2重ループでもそれほど遅くはなりません。あえてSetを使用しないという選択もあると思います。

### 無駄な拡張をしない

arrayやmapには「容量」があることを忘れずに。これを忘れると無駄な拡張が頻発します。あらかじめ必要な容量が分かっているなら、あらかじめ容量分確保しましょう。

```go
items := make([]int, 0, 10)
```

これで、容量10確保済みのsliceを作る事ができますよ。mapも同様。

## 3-6. deferを使おう

pythonのwithのようなコンテキストを表現するには、Golangではdeferを用います。

```go
defer s.Close()
```

- deferが実行されるのは「関数の終わり」だと言う事を忘れずに。

## 3-7. json

MarshalJSON、UnmarshalJSONメソッドの実装について学ぶことは、Golang設計において必須事項です。  
とはいえ、MarshalJSON/UnmarshalJSONを凝りすぎるのはまた問題だと思います。やりすぎないように。

### jsonの入力

Golangのネストの深いjsonを扱うのはとても難しく、普通に書くとdown castだらけのコードになります。

> 「腕力」と言ったりしますね。

これはとても読みにくいと思います。

Golangの内部では型安全にデータを扱う事を心がけ、必ず必要な情報はstructに格納しましょう。  
また、nestの深いjsonの奥のほうだけ必要な場合は、jsonpathのような技術を利用する方法もあります。

### jsonの出力

structをjsonに変換する事は全く難しくありませんが、出力するjson全てstruct定義が必要かと言うとそんなことはありません。  
もし厳密な型定義が不要なら、template変換によってjsonテキストを生成する事を検討しましょう。

### CloneやCopy

同じstruct間のcloneや、異なるstruct間のcopyが必要になる場合、jsonを介したcopyを検討しましょう。  
属性をひとつづつ移送するコードはバグの温床です。

将来的に属性が追加されたとき、移送漏れが発生しやすいため、可能な限り属性移送は避けましょう。  
例えばこんな感じ。

```go
func Copy(src interface{}, dest interface{}, allowUnknownFields bool) error {

    data, err := json.Marshal(src)
    if err != nil {
        return errors.WithStack(err)
    }
    dec := json.NewDecoder(bytes.NewReader(data))
    if !allowUnknownFields {
        dec.DisallowUnknownFields()
    }
    if err = dec.Decode(&dest); err != nil {
        return errors.WithStack(err)
    }
    return nil
}
```

allowUnknownFields引数がfalseのとき、コピー先に移送したい属性が不足しているとエラーになります。  
これにより、移送漏れを防ぐ事ができます。

まあ、どうしても性能を気にするなら、モデル側にコピーコンストラクタを用意しておくという方法はあるかもしれません。

### MarshalJSON/UnmarshalJSON

例えば time.Timeのような既存の型のjson表現を自由に変更することが可能です。

```go
type MyTime time.Time
func (t *MyTime) MarshalJSON() ([]byte, error) {
}
func (t *MyTime) UnmarshalJSON([]byte) error {
}
```

を実装することで、値としてはtime.Timeを持ちながらjson表現をカスタマイズできます。

## 3-8. 非同期

goroutineやchannelを学ぶことは、golang設計では必須事項です。  
特にstreamingなI/O処理を書くためには、goroutine、io.Pipeを使った処理を書く必要があります。

### Lockではなくchannelを

非同期処理を書く時、他の言語ではLockが必要になりますが、golangでは、Lockを使用せずにchannelを使用しましょう。  
具体的には、Lockして共有メモリを複数スレッドで読み書きするのではなく、channelを用いてスレッド間通信を検討しましょう。

## 3-9. コメント

Golangでは、pydocやjavadocのような詳細なコメントを書く文化があまりないようです。  
基本的には、コメントがなくても理解できるコードを書くよう心がけましょう。

もちろん、必要とあらばどんどんコメントを書きましょう！  
特に他人から利用される共有ライブラリ的なコードには、README的なコメントと、使い方が分かるテストをしっかり書きましょう。

## 3-10. テスト

モジュールや関数ごとにユニットテストを書きましょう。  
gomock、mockeryなど、プロジェクトごとに利用するmockライブラリを決定しましょう。

テストは、単にそのコードが正常に動いているかどうか確認するものであるだけでなく、そのコードを利用する方法を確認するためのものでもあります。  
つまり、テストもコメントの一部です。

### メッセージを使ったテストと状態を使ったテスト

テストには大きく二つの方法があります。一つはメッセージを使ったテストで、もう一つは状態を使ったテストです。  
メッセージを使ったテストでは、mockのrequestとresponseの内容を使ってテストします。  
状態を使ったテストでは、実際にRDBのようなストレージを用意して、「結果的にどういう状態になったか」を確認することでテストします。

基本的に単体テストでは可能な限りメッセージを使ってテストを書くべきだと思います。  
ただ、どうしても状態を使ったテストを書く必要があるケースもあります。

RDBなど、queryが複雑な場合、メッセージを使ったテストではなかなか正しさの証明ができないケースがあります。

状態を使ったテストは、最小限の範囲に絞って行いましょう。  
そして、それ以外の場所ではメッセージを使ったテストを書くのがいいと思います。

### monkey patch

mockのDIができない場合、monkey patchを利用する必要があります。ただ、monkey patchには動作が怪しい事が多いので、極力使用を限定的にしたほうが良いと思います。

---

## 4\. （追加）

## 4-1. 例外

golangの例外は値です。try catch機構など例外を扱うための特別な機構を持たない変わった言語ですね。

### early returnする

例外が発生したら「即座にreturnする」のがgolangの作法のようです。

### panicはプロセスを落としたいときだけ使う

panic recoverによってtry catch的な事ができないわけではないですが、そういう使い方は推奨されてないと思いますよ！  
Graceful Shutdownやpanic原因のロギングのため、main.goでrecoverすることをお忘れなく！

### errorsパッケージではなく、pkg/errorsパッケージを使用する

pkg/errorsは、stacktrace情報を持っている例外オブジェクトを作ってくれます。  
通常のerrorsパッケージでは単純に情報量が足りません。  
かならずpkg/errorsを使いましょう。

### cause句がない

他の言語だと大抵ある、例外のrethrow/reraiseのためのcause機構がgolangでは用意されていません！  
rethrowの際にcauseをロギングするだけでも良いかとは思いますが、カスタム例外を作る事を検討したほうが良いのではないかと感じます。

### 例外を握りつぶさない

他の言語と同じ。例外を握りつぶすくらいならpanicしましょう。

---

まずはここまで（編集中）。

Golang特有でないものがいろいろ混じってますが、外したほうがいいんでしょうかね・・・

[0](https://qiita.com/kishibashi3/items/#comments)

[52](https://qiita.com/kishibashi3/items/a244c4e4b42684bcd801/likers)

40