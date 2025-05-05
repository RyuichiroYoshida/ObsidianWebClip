---
title: dotnet-formatでC#ソースコードを整形する
source: https://qiita.com/skitoy4321/items/f88fe188bc0f85351f3c
author:
  - "[[Qiita]]"
published: 2019-04-11
created: 2025-05-06
description: はじめにroslynのリポジトリを見ていたら、dotnet-formatなるツールがあって、ちょうど良さそうなツールだったので紹介してみる。なぜ必要か昨今IDEの発達により、ソースコードのかな…
tags:
  - CSharp
  - Tech
  - Tools
  - QA
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から5年以上が経過しています。

## はじめに

roslynのリポジトリを見ていたら、 [dotnet-format](https://github.com/dotnet/roslyn/tree/master/src/Tools/dotnet-format) なるツールがあって、ちょうど良さそうなツールだったので紹介してみる。

## なぜ必要か

昨今IDEの発達により、ソースコードのかなりの部分が自動的に整形されるようになり、あまり自分のコーディングスタイルというものを意識しない人も増えてきたと思う。  
しかし、それでも自分でも無意識のうちに持っているコーディングスタイルというものはどうしても存在する。

一人で開発しているうちは好きにしたらいいと思うが、これが複数人での開発となると、各々のコーディングスタイルの違いから、本来の修正以上の差分が発生してノイズになってしまう場合がある。  
特にIDEの設定が各々異なるものを持っていた場合、自動的に整形されて余計な差分だらけになってしまうという悲劇はある(あった)。

対策として、 [EditorConfig](https://editorconfig.org/) を設定するという手があるが、現状対応していないエディタというのもあるにはある。  
また、都度都度エディタに任せて変換するよりも、コミット時に、確認の意味も含めて一括で変換したい場合もあるだろう。

そこで、そういう時は今回紹介するdotnet-formatが役に立つんじゃないかと思う。

## インストール

## 事前準備

dotnet-formatは [.NET Core グローバルツール](https://docs.microsoft.com/ja-jp/dotnet/core/tools/global-tools) として作られている。  
また、内部的にmsbuildも使用しているため、 [dotnet-sdk 2.1以降](https://dotnet.microsoft.com/download) が必要となる。

## インストールの実行

READMEにも書いてあるが、コマンドプロンプトで下記コマンドを実行する。

### 安定版、またはプレリリースバージョンを入れる場合

`dotnet tool install -g dotnet-format --version [パッケージバージョン]` と入力すれば良い。  
使えるバージョンは、 [dotnet-formatのページを参照](https://www.nuget.org/packages/dotnet-format/)

### 開発版を入れる場合

`dotnet tool install -g dotnet-format --version [NuGetバージョン] --add-source https://dotnet.myget.org/F/roslyn/api/v3/index.json`  
実行後、 `dotnet format --help` でヘルプが出力されればOK。されない場合、PATHが通っているか確認すること(Winの場合、 `%USERPROFILE%\.dotnet\tools` 、LinuxとMacの場合は `$HOME/.dotnet/tools`)。  
バージョン番号は、実際に [roslynのmygetサイト](https://dotnet.myget.org/feed/roslyn/package/nuget/dotnet-format) を見て確認すること。  
この方法でインストールした場合、最新のものを使用可能だが、代わりに予期しない不具合や、仕様変更等があるかもしれないので注意すること。

## 使い方

## EditorConfigの作成

基本的に無ければデフォルトの設定で動作するが、そのままだと `unset` で何もしない設定がかなり多いため、予め作成しておいた方がいいだろう。  
`.editorconfig` は基本的にslnファイルと同じ場所に置くのが良いと思う。基本的な書き方は [EditorConfig本サイト](https://editorconfig.org/) や、それこそ"EditorConfig"で検索すればかなりの数出てくると思うので、ここで説明はしない。

C#独自の設定項目としては、 [MS公式サイト](https://docs.microsoft.com/ja-jp/visualstudio/ide/editorconfig-code-style-settings-reference?view=vs-2017#formatting-conventions) が参考になる。ただし、\*\*自分が試した限りでは使えるのは書式規則だけで、言語コード スタイル等は使えなかったので注意してほしい。\*\*この辺り、使えた人がいたら情報求む。  
後、\*\*charsetとend\_of\_lineも挙動が怪しかった(効いてなさそう)\*\*ので、多少注意が必要だろう。  
この辺りはdotnet-formatの問題というよりは、ソースを見る限りはroslynの `Workspace.TryApplyChanges` 辺りの問題と思われる。

### 使用可能な設定

dotnet-formatで使用可能な設定の一覧は、下記URLに記載してある。  
[https://github.com/dotnet/format/wiki/Supported-.editorconfig-options](https://github.com/dotnet/format/wiki/Supported-.editorconfig-options)

## フォーマットの実行

フォーマットの実行は、 `dotnet format -w [ソリューションファイル又はプロジェクトファイル]` で良い。  
実行すると、ソリューション配下またはプロジェクト配下のC#ソースとVBソース全てに整形が適用される。  
**F#ソースやXML等その他のファイルには適用されないことに注意。**

### 対象ファイルの絞込

デフォルトではプロジェクト配下全てのファイルを対象にするが、3.0.4からは `--files` オプションが有効になり、これによって対象ファイルを明示的に指定できるようになった。  
複数指定する場合は、`,`で区切る。  
ただし、今の所対象外ファイルの指定や、ワイルドカード等は使えない模様。

## 終りに

dotnet-formatは、フォーマットしかしない。この点、gofmt等と比べると不足を感じるかもしれないが、それでも単機能ツールとしては十分だと思う。  
複数人プロジェクトで開発する時、PRレビュー等で書式関連の余計なノイズを減らしたいという場合は、検討してみるのもいいのではないかと思う。

[3](https://qiita.com/skitoy4321/items/#comments)

コメント一覧へ移動

[23](https://qiita.com/skitoy4321/items/f88fe188bc0f85351f3c/likers)

いいねしたユーザー一覧へ移動

18