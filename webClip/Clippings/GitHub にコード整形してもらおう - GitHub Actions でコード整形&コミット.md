---
title: GitHub にコード整形してもらおう - GitHub Actions でコード整形&コミット
source: https://qiita.com/Ouvill/items/7b6df0e9b981093b330f
author:
  - "[[Qiita]]"
published: 2020-03-01
created: 2025-05-06
description: コード整形していますか？複数人数でコードを書いていたりするとエディターの設定によってインデントの幅が違ったり、改行の位置が違ったりして困ってしまうことってありませんか？GitHub Action…
tags:
  - CI-CD
  - QA
  - 1
  - Tools
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から5年以上が経過しています。

コード整形していますか？

複数人数でコードを書いていたりするとエディターの設定によってインデントの幅が違ったり、改行の位置が違ったりして困ってしまうことってありませんか？

GitHub Actions を利用すると、コードをプッシュしたときに自動的にコードを整形するワークフローが構築できます。

Git Commit -> Push -> GitHub Actions でコード整形 -> 整形コミットを GitHub にコミット&プッシュ

本記事は GitHub Actions を用いた自動コード整形環境の構築方法を紹介します。

## 今回の紹介ツール

- Prettier ( JavaScript 整形ツール )
- GitHub Actions
- stefanzweifel/git-auto-commit-action

## Prettier

[Prettier](https://prettier.io/) とは JavaScript や HTML, CSS を自動整形してくれるツールです。

自分は JavaScript ( TypeScript ) をよく記述しているので、今回は Prettier を利用してコード整形する方法を紹介します。

Prettier のインストール

```text
npm install -D prettier
// or
yarn add -D prettier
```

prettierの設定ファイル `.prettierrc.js` の作成(オプション)

```js
'use strict';

module.exports = {
  bracketSpacing: false,
  singleQuote: true,
  jsxBracketSameLine: true,
  trailingComma: 'es5',
  printWidth: 80
};
```

以下のコマンドで、Pretteir でコード整形できます。

```text
npx prettier --write "src/**/*.{ts,tsx,js,jsx}"
//or
yarn prettier --write "src/**/*.{ts,tsx,js,jsx}"
```

## GitHub Actions

[GitHub Actions](https://github.com/features/actions) とは GitHub が提供している CI ツールです。CircleCI や Travis CI などのように Git の Push やプルリクのアクションに応じて、あらかじめ設定しておいたタスクを実行してくれるものです。

GitHub でコード管理 && CI 実行が完結するので管理が楽になります。

`.github/workflow/******.yml` に保存されたコードを GitHub Actions は実行してくれます。

## GitHub Actions でコード変更を行ったときにコミットする

GitHub Actions でコードの内容を変更したとき、変更内容をコミットするために以下の Action を利用します。

[stefanzweifel/git-auto-commit-action](https://github.com/stefanzweifel/git-auto-commit-action)

使いかたはこんな感じです。

```yml
steps:
     - uses: stefanzweifel/git-auto-commit-action@v3.0.0
        with:
          commit_message: Apply Prettier Change
```

GitHub Actions 内で Pretteir でコード整形をしたあとに、この Action を実行すれば、整形済みコードをコミットできるというわけです。

## 自動的にコードを整形

サンプルコードを二つ載せます

- master ブランチが更新されたら、自動的に pretteir でコードを整形
- Pull Request をもらったときに PR ブランチを整形

### master ブランチが更新されたら、自動的に pretteir でコードを整形

Master ブランチがコミットされると自動で整形コミットを行います。  
プルリクエストが取り込まれたあとに、コード整形が行われるので GitHub Flow や Git Flow　などといった直接 master に push しない運用だと、push 時にコンフリクトしにくいです。

`.github/workflows/prettier.yml`

```yml
name: Pretter

on:
  push:
    branches:
      - master

env:
  FILE_PATTERN: "{src,__tests__}/**/*.{ts,tsx,js,jsx}"

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v1
        with:
          node-version: 12.x
      - name: Run Prettier
        run: |
          yarn install
          yarn prettier --write ${FILE_PATTERN}
      - uses: stefanzweifel/git-auto-commit-action@v3.0.0
        with:
          commit_message: Apply Prettier Change
```

### Pull Request をもらったときに PR ブランチを整形

プルリクエストブランチに対して、整形を行います。プルリクエストがきたときに整形を行うので master ブランチには整形済みのコードしかマージされません。

ただ注意点は、プルリクエストをだした後に整形コミットを GitHub 側で追加するので、プルリクエストに追加コミットをプッシュするとき、 `git push -f` をするか、もしくは、 `git pull` してから `git push` する必要があります。

`.github/workflows/prettier.yml`

```yml
name: Pretter

on:
  pull_request:

env:
  FILE_PATTERN: "{src,__tests__}/**/*.{ts,tsx,js,jsx}"

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          ref: ${{ github.head_ref }}
      - uses: actions/setup-node@v1
        with:
          node-version: 12.x
      - name: Run Prettier
        run: |
          yarn install
          yarn prettier --write ${FILE_PATTERN}
      - uses: stefanzweifel/git-auto-commit-action@v3.0.0
        with:
          commit_message: Apply Prettier Change
          ref: ${{ github.head_ref }}
```

### まとめ

コード整形とコード修正がまざってしまうと、肝心のコード変更点がどこなのかわかりにくくなってしまいがちです。  
GitHub Actions を利用してコード整形を自動化してしまうことで、コード整形忘れをなくしましょう。

[0](https://qiita.com/Ouvill/items/#comments)

[13](https://qiita.com/Ouvill/items/7b6df0e9b981093b330f/likers)

16