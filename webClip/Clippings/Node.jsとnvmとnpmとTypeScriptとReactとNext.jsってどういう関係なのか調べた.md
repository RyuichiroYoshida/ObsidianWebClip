---
title: Node.jsとnvmとnpmとTypeScriptとReactとNext.jsってどういう関係なのか調べた
source: https://zenn.dev/ikkik/articles/6b6d3415cf5324
author:
  - "[[Zenn]]"
published: 2023-05-20
created: 2025-05-06
description: 
tags:
  - Tech
  - アプリ
  - バックエンド
  - "#フロントエンド"
  - "#JS"
  - "#TS"
read:
---
50

5[tech](https://zenn.dev/tech-or-idea)

## はじめに

いままで弱小バックエンドエンジニアとしてJavaを使ってお仕事をしてきたのですが、フロントエンドの開発に携わる機会が訪れました。  
Reactで開発するということで、お勉強がてら簡単なWEBアプリを作ろうとしたら、様々な技術スタックのワードが出るわ出るわ・・・  
それらが何なのか調べたので、自分用に参考リンク集を残しておきます。

## 外観

![relation image](https://storage.googleapis.com/zenn-user-upload/a3bb1368321b-20230519.png)  
*Node.jsとTypeScriptとnvmとnpmとReactとNext.js*

## Node.jsについて

#### 概要

OS上でJavaScriptを動かすためのJavaScript実行環境のこと。  
JavaScriptエンジンとして、V8エンジンを使用している。

#### 参考：概要理解

#### 参考：Node.jsの歴史

#### こぼれ話：Deno

2020年にDenoという対抗馬が登場している模様。Node.jsのシェアを奪えるのかどうか。  
Node.jsの製作者が、Node.jsの悪い点を改良を目指して作ったとのこと。

### Google V8 JavaScriptエンジンとは

#### 概要

JavaScriptのコードを実行するためのJavaScriptエンジンのこと。C++で記述されている。  
Google社が開発するオープンソースであり、Node.jsで使用されている他、Chromiumベースのブラウザでも使用されている。

#### 参考

#### 対抗馬：他のJavaScriptエンジン

| JavaScriptエンジン | ブラウザ |
| --- | --- |
| V8エンジン | Chromium |
| JavaScriptCore | Safari |
| SpiderMonkey | Firefox |
| Chakra | IE |

## nvm(Node Version Manager)について

#### 概要

バージョンが異なる複数のNode.jsをインストール／管理ができるバージョン管理ツール。  
Node.jsのバージョンが異なるプロジェクトを並行開発するのに、都度Node.jsを切り替えるのは面倒だ。簡単に切り替えたい。というニーズから登場。

#### プロジェクト毎にバージョン定義する

プロジェクトのルートディレクトリに`.nvmrc` ファイルを置き、中身にNode.jsのバージョンを記載することで、そのプロジェクトのNode.jsバージョンを定義できる。

#### 参考

#### 対抗馬：他のバージョン管理ツール

Node.jsのバージョン管理ツールは他にもある。

> - n
> - nodeenv
> - nodebrew
> - fnm
> - asdf
> - Volta

引用： [https://qiita.com/tak8\_al/items/791e8d032bdc13a135de#検討ツール](https://qiita.com/tak8_al/items/791e8d032bdc13a135de#%E6%A4%9C%E8%A8%8E%E3%83%84%E3%83%BC%E3%83%AB)

## npm(Node Package Manager)について

#### 概要

JavaScriptパッケージをインストール／管理ができるパッケージ管理ツール。  
Node.jsにバンドルされているため、Node.jsをインストールすると併せてインストールされる。

#### package.jsonについて

プロジェクトを作成する場合、一番最初に `npm init` コマンドを実行して、プロジェクトのルートディレクトリにpackage.jsonを生成する。  
package.jsonには、そのプロジェクトで使用する（依存する）パッケージを記載する。  
その後、プロジェクトのルートディレクトリで `npm install` コマンドを実行することで、依存するパッケージを `node_modules` ディレクトリにダウンロードすることができる。

#### ローカルインストールとは

コマンド

```bash
npm install # package.jsonに記載されたパッケージを全てインストール
npm install [パッケージ名] # 指定したパッケージのインストール＆package.jsonへの追記
```

`node_modules` 配下へのパッケージのインストールを意味し、そのプロジェクトでのみ参照される。  
ローカルインストールされたパッケージのコマンドを実行する場合は、そのプロジェクトのルートディレクトリにて、 `npx [コマンド]` または`./node_modules/.bin/[コマンド]` とする必要がある。

#### グローバルインストールとは

コマンド

```bash
npm install -g [パッケージ名]
```

Node.js全体へのパッケージインストールを意味し、プロジェクトに関わらず使用可能となる。  
グローバルインストールされたパッケージのコマンドを実行する場合は、直接 `[コマンド]` を実行可能である。  
**注意：Node.js単位でのインストールとなるため、グローバルインストールしたとしてもバージョンが異なるNode.js間では共有されない。**

#### package-lock.jsonとは何なのか

`npm install` コマンドを実行すると、勝手にpackage-lock.jsonが作成される。これはインストールした全てのパッケージのバージョンを記録しておくものである。  
このファイルを共有することで、複数の開発環境で全てのパッケージバージョンを一致させることが可能となる。

#### パッケージはどこからダウンロードしてるのか

npm.Incが管理している [npmjs.com](https://www.npmjs.com/) というパッケージレジストリからダウンロードされる。

#### 参考

#### こぼれ話：略称の意味

npmは「Node Package Manager」の略ではないらしい...

#### 対抗馬：他のパッケージ管理ツール

JavaScriptのパッケージ管理ツールは他にもある。

- yarn
- pnpm

## TypeScriptについて

#### 概要

JavaScriptに機能を追加したプログラミング言語のこと。  
プログラム実行時にJavaScriptコードに変換されるため、JavaScriptエンジンが必要となる。

#### 特徴：静的型付け言語

静的型付けのJavaScriptがTypeScriptと言える。  
静的型付けの特徴として、変数や関数の引数に型を指定することで、故障を早期発見することができる。

#### 特徴：トランスパイル

tsc(TypeScript Compiler)により、TypeScriptは様々なJavaScriptのバージョンへトランスパイルすることができる。  
これによりブラウザや実行環境の互換性問題を回避できる。  
また、JSX（後述のReactセクション参照）をJavaScriptへトランスパイルすることもできる。

#### インストール方法

npm経由でインストールする。よくグローバルインストールで紹介されている。

コマンド

```bash
npm install -g typescript
```

#### 参考

## Reactについて

#### 概要

WebアプリケーションのUI作成に使用するJavaScriptライブラリ。  
Webアプリフレームワークではなくあくまでライブラリであるため、Webアプリを開発するためには、他のライブラリやフレームワークと組み合わせる必要がある。

#### JSX(JavaScript XML)とは

Reactの特徴として、XMLタグ要素(DOM要素)を、JavaScriptオブジェクト(仮想DOMと言う)で扱う点がある。  
この仮想DOMを、コードとして直感的に記述する方法をJSXと言う。

```jsx
const HelloWorld = () => {
  return <div>Hello, World!</div>; // JSX記法はXML要素をそのまま記載できる
};
```

JSXはReact独自の記法のため、JavaScriptエンジンがそのまま実行することはできない。  
そのため、JavaScriptとして処理できる形に変換する（＝トランスパイルする）必要がある。  
このトランスパイルは、以下の2通りの方法があるようだ。

- Babelというトランスパイラを用いて行う
- TypeScriptのコンパイラ(tsc)を用いて行う

#### SPA(Single Page Application)とは

単一のページで、Webアプリケーションを構成する設計構造のこと。  
JavascriptでHTMLの一部を差し替えて必要な部分だけを読み込むため、サーバーとの通信量を抑え、アプリケーションのパフォーマンス向上につながる。

#### 参考

#### 対抗馬：他のUI作成用JavaScriptライブラリ

- Angular
- Vue.js

### Babelについて

#### 概要

JavaScriptのトランスパイラ。新しいJavaScriptバージョンのコードを古いJavaScriptのバージョンに変換する。  
また、古いブラウザがカバーしていない機能を補完する（ポリフィルと言う）ことも行う。  
Babelを使うことで、ブラウザのサポートを待たなくとも、次世代の標準機能や構文を使ってJavaScriptを書くことができる。

#### 参考

### Webpackについて

#### 概要

モジュール（jsファイル）を 1 つ（もしくは少数）にまとめることができるバンドラ。  
まとめることで、ブラウザからのリクエスト数を減らし、ファイル転送の効率が向上する。  
開発したコードだけではなく、それらのモジュールが依存するパッケージも含めてバンドルすることができる。  
そのため、ブラウザではバンドルされたjsファイルを読み込むだけで動作させることが可能となる。

#### 参考

### ESModuleについて

#### 概要

モジュール（jsファイル）のExport/Importを可能とするJavaScriptの機能のこと。

ESModule

```js
// エクスポート側(hello.js)
export default function() {
  console.log("Hello, World!");
}
// インポート側(main.js)
import hello from './hello.js';
hello();  // Hello, World!
```

#### CommonJSとは

ESModuleと同様のニーズを解消するために、CommonJSというモジュール形式も存在する。  
こちらはNode.jsで使用されているらしいが、現在はESModuleに統合する流れとなっている模様。

CommonJS

```js
// エクスポート側(hello.js)
module.exports = function() {
  console.log("Hello, World!");
};
// インポート側(main.js)
var hello = require('./hello');
hello();  // Hello, World!
```

#### 参考

## Next.jsについて

#### 概要

React ベースのJavaScriptフレームワークで、Web アプリケーションの構築に使用される。  
サーバサイドレンダリング（SSR）や静的サイトジェネレーション（SSG）、ファイルベースルーティングなどの機能を提供する。

#### 参考

## 最後に

Reactを勉強しようと思ったら、JavaScriptの歴史が始まったなって思った。  
とりあえず、それぞれの役割が（概要レベルだけど）掴めたので、適当にコピペでcreate-react-appして遊んでた時よりは前進かな。

50

5