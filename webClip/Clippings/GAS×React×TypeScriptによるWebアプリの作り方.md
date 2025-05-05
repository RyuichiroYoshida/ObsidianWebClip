---
title: GAS×React×TypeScriptによるWebアプリの作り方
source: https://qiita.com/__SATO__/items/89151d4a4cd7cd80f96b
author:
  - "[[Qiita]]"
published: 2024-11-24
created: 2025-05-06
description: はじめにGASでWebアプリを作るにはフロントエンド部分を HTMLファイル にまとめる必要があります。ただし、これは飽くまで最終的にHTMLにまとまっていればいいということなので、ローカル環境…
tags:
  - GAS
  - 1
  - バックエンド
  - フロントエンド
  - TS
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

[@\_\_SATO\_\_](https://qiita.com/__SATO__)

投稿日

## はじめに

GASでWebアプリを作るにはフロントエンド部分を HTMLファイル にまとめる必要があります。

ただし、これは飽くまで **最終的にHTMLにまとまっていればいい** ということなので、ローカル環境では好きな方法でWebアプリを作ることができます。

この記事では例としてVSCodeでTypeScriptによるReactアプリを作り、Viteというプラグインで１つのHTMLファイルにまとめてから clasp でGASのプロジェクトにアップロードしてみます。

## 事前準備

## Node.jsのインストール

TypeScriptやReact、Vite、claspなどを使うには Node.js をインストールする必要があります。

Node.jsをインストールするには公式サイトからインストーラを入手して実行しましょう。

- Node.js 公式サイト： [https://nodejs.org/en/](https://nodejs.org/en/)

## VSCodeでの開発

## 新規プロジェクトを作成する

任意のディレクトリにGASでデプロイするWebアプリを作るためのプロジェクト用のディレクトリを作成して移動してください。

```sh
mkdir react-web-app
cd react-web-app
```

## Viteプロジェクトの作成

### Viteのインストール

ViteはReactなどで作成したWebアプリを様々な形式でまとめるためのプラグインです。

今回はReactアプリを1ファイルにまとめるため、ViteSingleFile というプラグインをインストールします。

```sh
npm install vite-plugin-singlefile --save-dev
```

また、入力補完が使えるようついでにGASの型情報もインストールしておきましょう。

```sh
npm i @types/google-apps-script --save-dev
```

### Viteプロジェクトの初期化

Viteのインストールが完了したら次のコマンドでカレントディレクトリにViteプロジェクトを作成します。

```sh
npm create vite@latest .
```

するとカレントディレクトリの既存ファイルの取り扱いや使用するフレームワークの種類などを問われるので、それぞれ以下のように選択してください。

- 既存のファイルの取り扱い： Ignore files and continue
- フレームワーク：React
- バリエーションの選択：TypeScript

### 出力ディレクトリの設定

ViteでまとめたHTMLファイルの出力先を設定するため、vite.config.ts というファイルを次のように編集してください。

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { viteSingleFile} from 'vite-plugin-singlefile';

// https://vite.dev/config/
export default defineConfig({
  plugins: [react(), viteSingleFile()],
  build: {
    outDir: "appsscript/client",
  }
});
```

## GASプロジェクトとの接続

Viteプロジェクトが作成できたらGASプロジェクトとソースを共有するための作業スペースを作っていきます。

まずは clasp でGoogleアカウントにログインしてください。

続いて、GASプロジェクトにアップロードするファイルを格納するディレクトリを作ります。

```sh
mkdir appsscript
cd appsscript
```

ここで、新規のGASプロジェクトを作成する場合は次のコマンドを実行してください。

```sh
clasp create
```

あるいは、GASプロジェクトが既に存在する場合は次のコマンドでクローンしてください。

```sh
clasp clone スクリプトID
```

GASプロジェクトの新規作成またはクローンが完了したら.clasp.json を次のように編集してください。

```json
{
    "scriptId":"スクリプトID",
    "rootDir":"./",
    "parentId":"バインドされているGoogleWorkspaceアプリのID"
}
```

※ parentId：スクリプトプロジェクトがスプレッドシートなどのGoogleWorkspaceアプリのファイルにバインドされている場合に記載します。

また、ローカル環境からWebアプリをデプロイしたい場合は appsscript.json を次のように編集してください。

```json
{
  "timeZone": "Asia/Tokyo",
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8",
  "dependencies": {},
  "webapp": {
    "executeAs": "USER_DEPLOYING",
    "access": "MYSELF"
  }
}
```

デプロイの設定について

- executeAs：  
	doGet や doPost をどのアカウントで実行するかを設定する項目で次の選択肢があります。
	- USER\_DEPLOYING：Webアプリをデプロイしたアカウント
	- USER\_ACCESSING：Webアプリにアクセスしているアカウント
- access：  
	Webアプリにアクセスできるユーザの範囲を設定する項目で次の選択肢があります。
	- MYSELF：自分だけ
	- ANYONE：Googleアカウントにログインしている全員
	- ANYONE\_ANONYMOUS：全員
	- DOMAIN：同一ドメインのユーザだけ（※無料アカウントでは選択不可）

## サーバサイドの実装

ViteとGASのプロジェクト設定ができたらいよいよ実装です。

サーバサイドの処理は appsscript/server ディレクトリ に code.js というファイルを作り次のように定義しましょう。

```javascript
function doGet() {
  return HtmlService.createHtmlOutputFromFile("client/index")
    .addMetaTag("viewport", "width=device-width, initial-scale=1")
    // .setFaviconUrl("ファビコンURL") // ※ 任意
    .setTitle("GAS×React×TypeScript App");
}
```

## クライアントサイドの実装

Viteは srcディレクトリ の内容を vite.config.ts で設定した出力先に書き出すので、フロントエンド部分は普段の Reactアプリ の開発要領で実装できます。

ここでは App.tsx を次のように編集してみましょう。（コピペで大丈夫です！）

```typescript
import { useState } from 'react'
import reactLogo from './assets/react.svg'
import './App.css'

function App() {
  const [count, setCount] = useState(0)

  return (
    <>
      <div>
        <a href="https://workspace.google.com/intl/ja/products/apps-script/" target="_blank">
          <img 
            src="https://storage.googleapis.com/assets_workspace/uploads/7uffzv9dk4sn-3MG3mFcK1L7h5YD0Cgq36v-07b7abd2909e1389a54962bd6c3c99ed-AppsScript-title-logo.svg" 
            className="logo" alt="Google Apps Script logo"
          />
        </a>
        <a href="https://react.dev" target="_blank">
          <img src={reactLogo} className="logo react" alt="React logo" />
        </a>
        <a href="https://www.typescriptlang.org/ja/" target="_blank">
          <img 
            src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/4c/Typescript_logo_2020.svg/512px-Typescript_logo_2020.svg.png"
            className="logo react" alt="React logo" 
          />
        </a>
      </div>
      <h1>GAS × React × TypeScript</h1>
      <div className="card">
        <button onClick={() => setCount((count) => count + 1)}>
          count is {count}
        </button>
        <p>
          Edit <code>src/App.tsx</code> and save to test HMR
        </p>
      </div>
    </>
  )
}

export default App
```

### publicディレクトリのクリア

デフォルトのViteプロジェクトはpublicディレクトリにViteのロゴを配置します。

このロゴはアプリのビルド時に出力先ディレクトリへコピーされてしまいます。

この挙動を避けるため、ビルド前にpublicディレクトリは空にしておきましょう。

## Webアプリのデプロイ

Webアプリの実装が完了したらプロジェクトルートに移動してビルドコマンドを実行します。

```sh
npm run build
```

ビルドが完了したらappsscriptディレクトリに移動してソースコードをGASプロジェクトにプッシュします。

```sh
clasp push -f
```

プッシュが完了したらデプロイを実行しましょう。

```sh
clasp deploy
```

## 動作確認

デプロイが完了したらスクリプトエディタを開いて「デプロイ」→「デプロイの管理」からWebアプリのURLを確認してアクセスしてみましょう。

以下のような画面が表示されれば成功です！

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3939665/247cc180-6b47-cfd7-2e83-f581eaddd214.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3939665%2F247cc180-6b47-cfd7-2e83-f581eaddd214.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=9a32fa35942bbb85dbea9c7bb7234cc9)

[0](https://qiita.com/__SATO__/items/#comments)

[98](https://qiita.com/__SATO__/items/89151d4a4cd7cd80f96b/likers)

116