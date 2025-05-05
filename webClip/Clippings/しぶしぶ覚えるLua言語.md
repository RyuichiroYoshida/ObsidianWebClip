---
title: しぶしぶ覚えるLua言語
source: https://qiita.com/aike@github/items/2023bbeb21094af6795e
author:
  - "[[Qiita]]"
published: 2024-05-21
created: 2025-05-06
description: はじめにLua好きですか？ええ、そうですよね、好きでも嫌いでもないですよね。わかります。Luaが好きで好きでしょうがないという人はいません。ごめん、言い過ぎた。大抵は自分の使っているアプリのス…
tags:
  - Lua
  - 1
read:
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

## はじめに

Lua好きですか？  
ええ、そうですよね、好きでも嫌いでもないですよね。わかります。  
Luaが好きで好きでしょうがないという人はいません。ごめん、言い過ぎた。大抵は自分の使っているアプリのスクリプティング機能がLuaだからAPIを叩くために仕方なく使うという人が多いのではないでしょうか。

私のよく使うDTM分野のアプリでもReaperの [ReaScript](https://www.reaper.fm/sdk/reascript/reascript.php) 、Falconの [UVI Script](https://www.uvi.net/uviscript/) 、KONTAKTの [CREATOR TOOLS](https://www.native-instruments.com/jp/products/komplete/samplers/kontakt-6/building-in-kontakt/) 、HALionの [HALion Script](https://developer.steinberg.help/display/HSD/HALion+Script+Home) などLuaによるスクリプティングができるものがあります。 [Synthesizer V](https://resource.dreamtonics.com/scripting/ja/index.html) も音程や表現の自動調整のためにLuaが使えます。その他 [Roblox](https://developer.roblox.com/en-us/learn-roblox/coding-scripts) のようなゲームエンジン、画像・動画編集アプリなどでもLuaによるスクリプティング機能が搭載されることが増えてきました。

[Lua公式サイト](https://www.lua.org/) 、 [公式リファレンス](https://www.lua.org/manual/5.4/) の内容は充実しており、内容も最新版に合わせてメンテナンスされています。ただしソフトウェアエンジニア以外の非専門家が読むには少々難しい書き方になっています。  
Luaは日本語化されたドキュメントが比較的少ないです [^1] 。検索すれば日本語の解説はいくつかありますが、どこの馬の骨が書いたかもわからないQiitaの解説記事でプログラミング言語を学ぶなんてまったくナンセンスですよね。そういう学習意欲をそがれるような状況も問題だと思っています。

この記事は、Lua言語グルになりたいわけではない意識低めの人のための解説です。Lua言語を使い倒すのではなく、普段は別の言語を使っている人やプログラマー以外の人が、目的の機能を実装するために必要最低限の知識を最速で参照できるようにしています。特に他言語修得者がとまどいやすいポイントは詳しく解説しました。 [^2]

## Lua言語解説

## 1\. コメント

コメントの書き方はちょっと独特です。私もよく忘れてその都度調べます。

```lua
-- 1行コメント

--[[
　複数行コメント
--]]
```

## 2\. 文字列出力

Luaの文字列出力は一般にはprint()を使用します。

```lua
-- 改行あり出力
print("Hello World")

-- 改行なし出力
io.write("Hello World!")

-- 数値型変数の値を出力
x = 10
y = 20
print(x)
print(x, y)

-- 文字列連結して出力
s = "Hello"
print("string = " .. s)

-- 数値を文字列に連結して出力
print("value = " .. x)

-- 書式付き出力
print(string.format("%05d", x))
```

ただし、アプリの組み込み言語は標準出力コンソールが提供されているとは限らず、print()を使用しても何も表示されないこともあります。たとえばReaScriptの場合、文字列出力はprint()ではなくreaper.ShowConsoleMsg()を使用します。

```lua
reaper.ShowConsoleMsg("Hello World\n")
```

## 3\. インクリメント、デクリメント

++、--のようなインクリメント、デクリメント演算子はありません。x = x + 1 のように書きます。

## 4\. 制御文

### 4.1 ifによる分岐

if文の条件式にカッコは不要です。もちろんつけても構いません。  
条件式の後にthenをつけるのと、endでブロックを終了するのが特徴的です。

```lua
if x > 5 then
  print("x > 5")
end
```

elseifはelseとifの間に空白を開けません。

```lua
if x > 10 then
  print("x > 10")
elseif x > 5 then
  print("10 >= x > 5")
else
  print("5 >= x")
end
```

### 4.2 比較演算子

比較演算子は他の言語と同様に==、>=、<=、>、<を使いますが、不一致判定が~=というのは要注意です。

```lua
if x ~= 5 then
  print("x is not 5")
end
```

### 4.3 論理演算子

論理演算子は&&や||ではなく、andとorを使います。

```lua
if x > 5 and y > 5 then
  print("x > 5 and y > 5")
end
```

### 4.4 switch/selectによる複数分岐

switchやselectのような複数条件分岐はありません。前述のif ～ elseifを使います。

### 4.5 while/forによるループ

ループ構文としてwhileとforが用意されています。

```lua
i = 0
while i < 5 do
  print(i)
  i = i + 1
end
```

```lua
for i = 1, 5 do
  print(i)
end
```

repeat/untilもあります。whileがあればあえて使うケースはあまりないですが。

### 4.6 break/continue

ループを抜けるbreakはありますが、continueはありません。

## 5\. グローバル変数、ローカル変数

### 5.1 変数宣言

何もつけないで変数を使用すればグローバル変数、localをつけて使用すればローカル変数となります。  
関数宣言の中ではlocalをつけるようにすれば間違いありません。

```lua
x = 5

function foo()
  local x = 10
  print(x)
end
```

### 5.2 関数宣言外でのlocal

一番外側のメインコードチャンクの変数にlocalをつけるかどうかは流儀によります。localをつけても関数宣言内から見えてしまうのでグローバル変数とほとんど違いはありません。内部的にはグローバル環境を汚さないという違いはあるのですが、小規模のスクリプトでは違いを意識することはほぼないと思います。

```lua
x = 5
local y = 10

function foo()
  print(x)  -- 5が出力される
  print(y)  -- 10が出力される
end
```

## 6\. スタートアップ関数

アプリによりますが、onInit()のような最初に実行されるスタートアップ関数が用意されている場合があります。普通はメインコードチャンクを実行してからスタートアップ関数を実行するようになると思います。前述のとおりメインコードチャンクのローカル変数は関数から参照できます。  
以下はUVI Scriptの例です。

```lua
print("main code chunk")
local x = 5

function onInit()
  print("startup function")
  print(x)
end
```

出力例

```text
main code chunk
startup function
5
```

## 7\. コレクション

### 7.1 配列

この手の言語としては珍しく配列は１オリジンです。要注意です。

```lua
arr = { 5, 10, 15, 20 }
print(arr[1]) -- 5が出力される
```

配列の長さは#で取得できます。そのため配列全体をループする場合は以下のように書きます。

```lua
arr = { 5, 10, 15, 20 }
for i = 1, #arr do
  print(arr[i])
end
```

pairs()を使ってイテレーションすることもできます。インデックスと値の両方を取得できます。

```lua
arr = { 5, 10, 15, 20 }
for idx, val in pairs(arr) do
  print(idx, val)
end
```

### 7.2 連想配列

連想配列のイテレーションもpairs()を使います。keyとvalueの両方の要素を取得できます。

```lua
scores = { John = 98, Paul = 100, George = 85, Ringo = 80}
for name, score in pairs(scores) do
  print(name, score)
end
```

不要な要素がある場合は変数名を \_ とすることで、使用しないことを明示的に宣言することができます。

```lua
scores = { John = 98, Paul = 100, George = 85, Ringo = 80}
for _, score in pairs(scores) do
  print(score)
end
```

### 7.3 データ構造のループ

実際のLuaスクリプトでは入れ子になったアプリのデータ構造をたどることがよくあります。以下はdataの中にgroupが複数含まれ、groupの中にpartが複数含まれるようなデータ構造のイテレーション例です。

```lua
for _, group in pairs(data.groups) do
  print(group.name)
  for _, part in pairs(group.parts) do
    print(part.name)
  end
end
```

## 8\. ファイルアクセス

### 8.1 io.input/io.output

Luaのファイル読み書きはio.input/io.outputでできます。

```lua
io.input("input.txt")
line = io.read()
while line do
  print(line)
  line = io.read()
end
```

### 8.2 API経由のファイル入出力

しかしながら、実際に必要になるのはアプリ固有のデータファイルやメディアファイルの読み書きが多いはずです。そのためio.input/io.outputではなくアプリのAPIを通じたファイルアクセスが通常使われます。以下はReaScriptの例です。

```lua
-- open the project file using Reaper API
reaper.Main_openProject("c:/tmp/myproject.rpp")
```

```lua
-- insert the wav file into the selected track using Reaper API
reaper.InsertMedia("c:/tmp/kick.wav", 0)
```

### 8.3 フォルダ内のファイル一覧取得

残念なことに指定したフォルダ内にあるファイル名のリストを取得する標準的な方法は用意されていません。パイプを使ってlsコマンドを実行して結果を受け取るような方法しかないようです。

アプリによってはファイル一覧を取得するAPIが用意されている場合もあります。以下はCREATOR TOOLSの例です。

```lua
-- list filenames using CREATOR TOOLS filesystem API
for _, file in filesystem.directoryRecursive(path) do
  print(file)
end
```

## 9\. オブジェクト指向プログラミング

Luaでは、メソッドを持ったオブジェクトを定義できます。ただ、組み込みスクリプトで自作のオブジェクト定義が必要になるケースは少ないでしょう。一方でAPIがオブジェクトとメソッドで提供されるケースは多いので、呼び方は覚えておきましょう。

```lua
dog = {
  name = "pochi",
  say = function(self)
    print("bow wow")
  end
}

cat = {
  name = "tama",
  say = function(self)
    print("mew")
  end
}

print(dog.name)
print(cat.name)
dog:say()
cat:say()
```

コロン( dog:say() )とピリオド( dog.say() )は自分自身の参照を渡すか否かの違いがあり、どちらを使ってもよいケースもありますが、メソッドにはコロンを使うようにすると間違いが少ないです。

## 参考

以前書いた [DTMプログラミング言語探訪シリーズ](https://qiita.com/aike@github/items/77b3eccc6861de09513a) のLua言語ツールのリンクです。

- [NI KONTAKT CREATOR TOOLS](https://qiita.com/aike@github/items/7675d9db1864c1fed417)
- [UVI Falcon UVI Script](https://qiita.com/aike@github/items/71ef2275b9b8c8e71873)
- [Steinberg HALion Script](https://qiita.com/aike@github/items/2311bdac551f69db7067)

[0](https://qiita.com/aike@github/items/#comments)

[115](https://qiita.com/aike@github/items/2023bbeb21094af6795e/likers)

73

[^1]: [こちら](https://inzkyk.xyz/lua_5_4/) の非公式翻訳版リファレンスマニュアルは参考になります。「Lua言語」で検索しても上位に出てこないので気づきにくいかもしれません。

[^2]: 少し前に流行ったプログラミング言語基礎文法最速マスターの [Lua言語版](http://handasse.blogspot.com/2010/02/lua.html) も同じようなコンセプトでわかりやすいです。