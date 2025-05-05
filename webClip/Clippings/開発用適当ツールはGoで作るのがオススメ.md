---
title: 開発用適当ツールはGoで作るのがオススメ
source: https://qiita.com/ssc-ksaitou/items/6c66669f1672806ac9bb
author:
  - "[[Qiita]]"
published: 2025-03-24
created: 2025-05-06
description: 開発用適当ツールとは？開発していると、たまに何かしらプロジェクト内で開発者用や運用者用にテストデータを作成したり、DBやAPIに繋いでCSVやExcelを出したりする名もなきツールが大量に必要にな…
tags:
  - Go
  - Tech
  - Tools
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

## 開発用適当ツールとは？

開発していると、たまに何かしらプロジェクト内で開発者用や運用者用にテストデータを作成したり、DBやAPIに繋いでCSVやExcelを出したりする **名もなきツールが大量に必要になってきますよね？** 配布して他の人にも使ってもらったりしたくなりますよね？

これが開発用適当ツール [^1] です。

そういった **開発用適当ツールをGoで作ってみたら案外体験が悪くなかった** のでシェアしたいと思います。

## どうやって開発用適当ツールを作るか？

既存プロジェクトにそのままGoのプロジェクトレイアウトを重ねていきます。

具体的には以下のような感じです。

```text
project-root-directory/
  プロジェクトの既存ファイル
  go.mod
  go.sum
  cmd/
    ツール1/
      main.go
    ツール2/
      main.go
  internal/ (もしくは分かりやすいディレクトリ)
    (main.go に入らなかった共通処理などはここに)
```

基本的には [公式のGoプロジェクトレイアウト](https://go.dev/doc/modules/layout) と同じものを既存プロジェクトにそのまま乗せてしまいます。（あるいはツール用に独自プロジェクトを切ります）

実際に始めるにあたり、以下のコマンドでプロジェクトを初期化すればプロジェクトファイル (`go.mod`) を自動作成できます。

Goプロジェクト初期化コマンド

```sh
$ go mod init my-super-awesome-tools
```

あとは `cmd/ツール1/main.go` などに以下のようなコードを書いて、参照するパッケージが増えたら必要に応じて `go mod tidy` （後述） を実行していけばよいです。

cmd/ツール1/main.go

```go
package main

import (
    "fmt"
)

func main() {
    fmt.Println("私の特別なツール！")  
}
```

実行する際は以下で一発で動かせます。

自作ツールを動かしてみる

```sh
$ go run cmd/ツール1/main.go
私の特別なツール！
```

## Goベースの開発用適当ツール作成の体験が良かった点

### go run path/to/main.go でツールがすぐに動く

ツールを動かす人にGoをインストールしてもらっていれば、以下のコマンドだけでツールがOSを問わずすぐに動きます。

```sh
$ go run path/to/main.go
```

通常、Pythonなどで開発ツールを作った場合、動かす前に依存しているライブラリのインストールなどが必要になりますが、Goの場合は **`go run` コマンドだけで依存性のダウンロードからコンパイル、動作まで全部やってくれます。**

これが非常に大きいです。ちょっとしたツールを使うためだけに、利用者ごとに環境特有の環境整備・環境分離ツールの習熟・依存性のインストール管理は大変だと思います。特に作成から時間を置いたツールが今動かなくなる率の高いこと高いこと……。

しかもGoだと後方互換性を大事にしているので、動作させるためのGoについては何も考えず最新版を入れてくれればいい上、最近は [Go Toolchains](https://go.dev/doc/toolchain) (前方互換性) の仕組みで自動的にローカルより上のバージョンのGoが必要な場合は勝手にインストールした上で起動してくれます。すごい。

### Goならクロスコンパイルできる

Goなら、最悪の場合、利用者にGoをインストールしてもらわなくても、 **各種環境のバイナリを自由に出力して配布することができます。**

```sh
# 特にWindowsやmacOSを持っていなくても自由に実行ファイルを作成できる！！

# Windows (AMD64)
$ GOOS=windows GOARCH=amd64 go build -o main-windows-amd64.exe main.go
# Linux (AMD64)
$ GOOS=linux GOARCH=amd64 go build -o main-linux-amd64 main.go
# macOS (Apple Silicon)
$ GOOS=darwin GOARCH=arm64 go build -o main-darwin-amd64 main.go
```

出力されたバイナリは **シングルバイナリとして動作し、動作時に外部のライブラリ(`*.dll`, `*.so` など)を必要としない** のも嬉しいところです。また、通常の処理を書く限りはプログラムが環境依存することもありません。

また、CI/CDで開発用適当ツールを使いたい場合、GoであればCI/CD用にツールをDockerイメージにするのもそこまで苦労しません。イメージの中身はほぼGoのシングルバイナリだけなので、サイズが小さいのも嬉しいところです。

ちなみにツールのリポジトリがアクセス可能な場所にある場合か、プライベートでも [アクセス手段](https://cloud.google.com/artifact-registry/docs/go/authentication?hl=ja) をきちんと設定してある場合は、インストールをスキップしてツールをいきなり実行することも可能です。

```bash
# いきなり実行！
$ go run github.com/swaggo/swag/cmd/swag@latest

NAME:
   swag - Automatically generate RESTful API documentation with Swagger 2.0 for Go.

USAGE:
   swag [global options] command [command options] [arguments...]

...
```

### サンプルコードがすぐに動く 〜 依存性管理が楽

たとえばGo言語におけるYAMLの入出力方法が分からない場合、ネットで検索するかAIに質問すると思います。そうすると以下のようなコードがサジェストされ……

main.go

```go
package main

import (
        "fmt"
        "io/ioutil"
        "log"
        "gopkg.in/yaml.v3" // 標準ではないライブラリを紹介された！
)
```

だいたい言語にバンドルされていない標準ではないライブラリを紹介されることと思います。（上記では `gopkg.in/yaml.v3` ）

こういった場合でもGoでは `go mod tidy` のコマンドを実行するだけで勝手にソースコードに書いてある未知のパッケージをインストール＆依存性の記録までやってくれます。 [^2]

```sh
# gopkg.in/yaml.v3 をインストール & go.mod/go.sum に記録してくれる
$ go mod tidy
```

上記の例だと未知のパッケージは1個だけでしたが、大抵の場合は複数あるので `go mod tidy` は非常に助かります。

### ビルドが驚くほど速い

Goのビルドは驚くほど速いです。（※ 個人の感想です）

そのおかげで `go run path/to/main.go` コマンドは **スクリプト感覚で実行できます。**

### 動作速度もそこそこ速い

RustのようにC/C++に匹敵するぐらい爆速 (blazingly fast) という感じではありませんが、Goは基本的には現在使われている言語の中でそこそこ速い方だと思います。少なくともスクリプト言語には負けません。

### 退屈な言語を目指しているので未来のメンテが怖くない

Goは **退屈を良し** としている珍しい (?) 言語です。

> **Boring is good**. Boring is stable. Boring means being able to focus on your work, not on what’s different about Go. This post is about the important work we shipped in Go 1.21 to keep Go boring.  
> [https://go.dev/blog/compat](https://go.dev/blog/compat)

Goにおいては、言語のバージョンアップは後方互換性が重視されますし、言語自体もチャネルやgoroutineなどいくつかの面白いものを除けば馴染のある構造体・スライス（リスト）・プリミティブな抽象や制御構造をベースに、クラスベースオブジェクト指向でもなく、例外や継続といった複雑な制御構造もなく、エラー処理といえばひたすら `error` を下から上にあげるだけ、という退屈な言語です。

よって、コードも平易に書くほかなく、（書く時はやや苦労しますが）後々メンテが大変になることも少ないと思われます。 [^3]

## Go以外だと開発用適当ツールにどの言語を選べばいいのか？

似たような用途で使えそうな言語と私なりの偏見と感想を書いておきます。

- Python
	- 最近のUnix環境ならほぼ入っているのが強い
	- 標準パッケージで色々な機能をサポートしているのも強い
	- 反面、Python自体のバージョン管理＆依存性管理が大変
		- **（2025/03/24） これについては [uvを使うと良さそうという記事](https://qiita.com/ssc-ksaitou/items/9da75058489ebe8c2009) を書きました**
		- そもそもそれらツールが豊富すぎてどれを使えばいいのかわからない
		- Python依存関係の本当の大変さはプロジェクトを放置した後にやってくる
- Perl
	- ワンライナーでワンポイントだけで使いたい言語だが大きなものは作りたくない
	- Perl Monger自体が絶滅危惧種
- Groovy (with Gradle)
	- `build.gradle` の単一ファイルでスクリプト定義も依存性管理も全て完了するのは良い
		- Gradle由来のタスクランチャー機能・ファイルセレクタなどが標準で備わっている
		- [Grape](https://docs.groovy-lang.org/latest/html/documentation/grape.html) を一生懸命使うよりGradleを使ったほうが便利だと思う
	- Javaの管理が面倒。特にJDKが上がると動かなくなる
	- 立ち上がりが遅い
- Node.js / Bun
	- JSプロジェクト内で使うならアリ
		- or 依存性なしの本当に小さなfetchをするスクリプトとかを書くならアリ
	- ネイティブコンパイルが絡むライブラリ(gyp)があると面倒
- Deno
	- 1ファイルで完全に依存性まで指定できるので適当ツール作成という面ではポテンシャルがある
		- 対応パッケージが少ないのが難点か
	- デフォルトのセキュリティが強いのが面倒かも
- Swift
	- ちょっと調べてみたが、案外いいかもしれない
		- 1ファイルだけで依存性指定も全て完了する
	- Apple端末以外への可搬性がどうかは分からない

**（2025/03/24 追記）Pythonについてはuvを使うと良さそうです。以下に記事を書きましたので参考にしてください。**

## まとめ

開発用適当ツールという用途をベースにGo言語の良さを紹介してきました。私も実際に開発プロジェクトでの各所ツールにGoを使っています。

開発用適当ツールを作ろうと思った際に、Goを候補にしてみるのもいいと思います。

[5](https://qiita.com/ssc-ksaitou/items/#comments)

[469](https://qiita.com/ssc-ksaitou/items/6c66669f1672806ac9bb/likers)

373

[^1]: 一部の人は治具と呼んでいるかもしれません

[^2]: GoにはJSでいう `package.json` / `package-lock.json` に相当する `go.mod` / `go.sum` があり、一旦インストールされた依存性はどこでも再現することができます。

[^3]: とはいえ、最近はGitHub CopilotなどのAI入力が退屈な入力を割と代替してくれるので、Goのような退屈な言語にはいい時代になりました