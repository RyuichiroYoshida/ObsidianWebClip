---
title: Go Template の基本的な動作を見てみよう
source: https://qiita.com/_otakakot_/items/3ca4eba2576bc5bb1f30
author:
  - "[[Qiita]]"
published: 2023-03-20
created: 2025-05-06
description: はじめにGoにはテンプレートエンジンの機能としてtext/templateおよびhtml/templateがあります。https://pkg.go.dev/text/templatehttps…
tags:
  - Go
  - 1
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から1年以上が経過しています。

## はじめに

Goには [テンプレートエンジン](https://ja.wikipedia.org/wiki/%E3%83%86%E3%83%B3%E3%83%97%E3%83%AC%E3%83%BC%E3%83%88%E3%82%A8%E3%83%B3%E3%82%B8%E3%83%B3) の機能として `text/template` および `html/template` があります。

HTMLを出力する場合はコード・インジェクションに対応している `html/template` を用いる必要があります。

`{{` `}}` で囲まれた「アクション」（データ評価または制御構造）と`.`を用いたデータ参照を組み合わせてテンプレートを作成します。  
Go Template は HELM の [テンプレート](https://helm.sh/ja/docs/howto/charts_tips_and_tricks/) にも採用されておりGopher以外の方でも触れる機会があるかと思います。

今回の記事では、基本的な動作となるテンプレートと入力を複数組み合わせてどのような出力となるのかを確認してみます。

## Go Template の使い方

テンプレート作成・出力の基本的な流れは以下のようになります。

1. [Template](https://pkg.go.dev/text/template#Template) を [New](https://pkg.go.dev/text/template#New) により用意
	```go
	tmpl, err := template.New("my-template").Parse("Hello, {{.}}!")
	```
2. [Execute](https://pkg.go.dev/text/template#Template.Execute) によりテンプレートを埋め込み文字列を作成する流れとなります。
	```go
	err := tmpl.Execute(os.Stdout, "World")
	```
3. 出力
	```shell
	Hello, World!
	```

公式サイトの [ドキュメント](https://pkg.go.dev/text/template) では `Inventory` という構造体を定義しテンプレート側で `{{.Count}}` および `{{.Material}}` にて値を参照しています。

```go
package main

import (
    "os"
    "text/template"
)

func main() {
    type Inventory struct {
        Material string
        Count    uint
    }

    sweaters := Inventory{"wool", 17}
    
    tmpl, err := template.New("test").Parse("{{.Count}} items are made of {{.Material}}")
    if err != nil {
        panic(err)
    }

    err = tmpl.Execute(os.Stdout, sweaters)
    if err != nil {
        panic(err)
    }
}
```

実行結果

```shell
17 items are made of wool
```

## {{.}} で何が表示される？

シンプルな挙動を確認するためにテンプレート文字 `{{.}}` を用意しさまざまな値を設定して出力を確認します。  
以下のコードを基本として変数 `val` に様々な値を設定して出力を確認します。

```go
package main

import (
    "os"
    "text/template"
)

func main() {
    val := 1 //　ここの値（型）を変えてみる

    text := "{{.}}"

    tmp, err := template.New("template").Parse(text)
    if err != nil {
        panic(err)
    }

    if err := tmp.Execute(os.Stdout, val); err != nil {
        panic(err)
    }
}
```

## 数値型 int

```go
// input
val := 1

// output
// 1
```

## 浮動小数点 float

```go
// input
val := 1.1

// output
// 1.1
```

## 論理値型 bool

```go
// input
val := true

// output
// true
```

## 文字列型 string

```go
// input
val := "value"

// output
// value
```

## rune

```go
// input
val := 'a'

// output
// 97
```

※ `UTF-8` 文字コードで文字列をエンコードした結果

## byte

```go
// input
val := []byte("value")

// output
// [118 97 108 117 101]
```

## 配列

```go
// input
val := []string{"a", "b", "c"}

// output
// [a b c]
```

## 構造体

```go
// input
val := struct {
    X string
    Y int
}{
    X: "value",
    Y: 1,
}

// output
// {value 1}
```

## map

```go
// input
val := map[string]string{
    "key": "value",
}

// output
// map[key:value]
```

## 特定の要素にアクセス

配列や構造体、mapによる入力はテンプレート側で特定の要素にアクセスするように記述することができます。

## 配列

以下のコードを基本として入力は変えずに `Template` としてアクセスする要素を変更してみます。  
配列にアクセスするには `index` という [Functions](https://pkg.go.dev/text/template#hdr-Functions) を用いる必要があります。  
`.[0]` のようなアクセスはできないようです。

```go
package main

import (
    "os"
    "text/template"
)

func main() {
    val := []string{"a", "b", "c"}

    temp := "{{ index . 0 }}"

    tmp, err := template.New("template").Parse(temp)
    if err != nil {
        panic(err)
    }

    if err := tmp.Execute(os.Stdout, val); err != nil {
        panic(err)
    }
}
```

```go
// temp
temp := "{{ index . 0 }}"

// output
// a
```

```go
// temp
temp := "{{ index . 1 }}"

// output
// b
```

```go
// temp
temp := "{{ index . 2 }}"

// output
// c
```

```go
// temp
temp := "{{ index . 3 }}"

// output
// panic: template: template:1:3: executing "template" at <index . 3>: error calling index: reflect: slice index out of range
```

与えられた配列の外側にアクセスしようとするとエラーが発生します。

## map

以下のコードを基本として `Template` としてアクセスする要素を変更してみます。

```go
package main

import (
    "os"
    "text/template"
)

func main() {
    val := map[string]string{
        "a": "A",
        "b": "B",
        "c": "C",
    }

    temp := "{{ .a }}"

    tmp, err := template.New("template").Parse(temp)
    if err != nil {
        panic(err)
    }

    if err := tmp.Execute(os.Stdout, val); err != nil {
        panic(err)
    }
}
```

```go
// temp
temp := "{{ .a }}"

// output
// A
```

```go
// temp
temp := "{{ .b }}"

// output
// B
```

```go
// temp
temp := "{{ .c }}"

// output
// C
```

```go
// temp
temp := "{{ .d }}"

// output
// <no value>
```

配列のときと異なりエラーが発生するのではなく `<no value>` と出力されました。

## 構造体

以下のコードを基本として `Template` としてアクセスする要素を変更してみます。

```go
package main

import (
    "os"
    "text/template"
)

func main() {
    val := struct {
        X string
        Y int
    }{
        X: "value",
        Y: 1,
    }

    temp := "{{ .X }}"

    tmp, err := template.New("template").Parse(temp)
    if err != nil {
        panic(err)
    }

    if err := tmp.Execute(os.Stdout, val); err != nil {
        panic(err)
    }
}
```

```go
// temp
temp := "{{ .X }}"

// output
// value
```

```go
// temp
temp := "{{ .Y }}"

// output
// 1
```

```go
// temp
temp := "{{ .Z }}"

// output
// panic: template: template:1:3: executing "template" at <.Z>: can't evaluate field Z in type struct { X string; Y int }
```

配列のときと同様、存在しないキーにアクセスを試みるとエラーが発生します。

## おわりに

基本型である数値は文字列はそのまま出力されることが確認できました。  
また、配列や構造体、mapでは型情報も一緒に出力されることが確認できました。  
配列や構造体、mapにおいてはそれぞれの要素に対してアクセスする方法も確認しました。  
基本的な動作となるテンプレートと入力を複数組み合わせてみましたが、少しだけ Go Template に対する壁がなくなったと思います。  
Go Template を使いこなすためには制御構文や演算子を使いこなす必要があるので手を動かしてみてそちらも記事を書こうと思います。

[0](https://qiita.com/_otakakot_/items/#comments)

[7](https://qiita.com/_otakakot_/items/3ca4eba2576bc5bb1f30/likers)

3