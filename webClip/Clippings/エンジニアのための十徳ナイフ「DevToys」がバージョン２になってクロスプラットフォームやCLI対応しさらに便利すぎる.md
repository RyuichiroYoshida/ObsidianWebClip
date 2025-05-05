---
title: エンジニアのための十徳ナイフ「DevToys」がバージョン２になってクロスプラットフォームやCLI対応しさらに便利すぎる
source: https://qiita.com/danishi/items/a75ab904da66cecc71b4
author:
  - "[[Qiita]]"
published: 2024-06-18
created: 2025-05-06
description: はじめに以前紹介させていただき、2022年Qiitaのいいねランキング18位、ストックランキング20位を記録したこちらの記事の続編です！https://qiita.com/danishi/ite…
tags:
  - Tools
  - 1
read:
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

## はじめに

以前紹介させていただき、 [2022年Qiitaのいいねランキング18位、ストックランキング20位](https://qiita.com/Qiita/items/75a34af032d898a86679) を記録したこちらの記事の続編です！

DevToysはリリース後しばらく定期的なバージョンアップが続けられていましたが、去年の7月からぱったりとアップデートが止まっている状態でした。

リポジトリや作者のXを見るとバージョン２の開発を行っているようで、今か今かと待ち続けていましたが数日前リリース予告のポストを見つけて、今日ついにされました！

ということで早速紹介していきます！

## DevToysとは

DevToysは「開発者のためのスイスアーミーナイフ」の紹介文の通り、開発時によく使うツールを十徳ナイフのようにまとめたアプリとなっています。

JSONの整形とかエンコードデコードetc...  
プログラミングや保守運用の調査でやりがちな作業をいちいち変換サイトを探したり、エディター拡張機能のショートカットを探したりせずとも、これ一つですぐにできます！

オフラインで軽快に動作するため、張り付ける文字列の流出に気を使わなくてよいところも便利なポイントです。

## DevToys 2.0の新機能

目玉の一つが **クロスプラットフォーム** になったことです！  
以前はWindows専用アプリだったのでMacやLinuxユーザーは使うことができなかったのですが、どのOSでも使えるようになったのは大きな変更です。

他にも、拡張機能を追加できるようになったり、CLI版も用意されています。

ツール群や基本的な機能はバージョン1から引き継がれているので、機能の紹介は [こちら](https://qiita.com/danishi/items/2de6a4ab028d27a8f4ab) の記事を確認ください。  
ただし、バージョン2で追加されたツールもあるのでそれらは後程紹介します。

## インストール方法

ダウンロードページから自身のOS、CPU環境に合うものを選んでインストールしてください。

※M1 Macでは [こちら](https://github.com/DevToys-app/DevToys/issues/1183#issuecomment-2158706255) の対応を行わないと動かないことがあるようです。

## 各OSでの見た目

### Windows

[![](https://qiita-user-contents.imgix.net/https%3A%2F%2Fpbs.twimg.com%2Fmedia%2FGPy-a5SaYAAA2Aw%3Fformat%3Djpg%26name%3Dlarge?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=2d9d322e8d674835e94e86e66e3c2218)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fpbs.twimg.com%2Fmedia%2FGPy-a5SaYAAA2Aw%3Fformat%3Djpg%26name%3Dlarge?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=2d9d322e8d674835e94e86e66e3c2218)

### Mac

[![](https://qiita-user-contents.imgix.net/https%3A%2F%2Fpbs.twimg.com%2Fmedia%2FGPy-a5SasAAtH10%3Fformat%3Djpg%26name%3Dlarge?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=bcc60a038bf54e19276abf352fea2701)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fpbs.twimg.com%2Fmedia%2FGPy-a5SasAAtH10%3Fformat%3Djpg%26name%3Dlarge?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=bcc60a038bf54e19276abf352fea2701)

### Linux

[![](https://qiita-user-contents.imgix.net/https%3A%2F%2Fpbs.twimg.com%2Fmedia%2FGPy-a5WaYAAQ2Gm%3Fformat%3Djpg%26name%3Dlarge?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a27e09a2e192a450e7c4aaabcdfc1738)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fpbs.twimg.com%2Fmedia%2FGPy-a5WaYAAQ2Gm%3Fformat%3Djpg%26name%3Dlarge?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a27e09a2e192a450e7c4aaabcdfc1738)

## 追加ツール紹介

### JSONの表形式ビューとCSV変換

JSONを表形式ビューにして、結果をCSVやTSVでクリップボードコピーできる。  
ネストされたJSONもある程度解釈できる。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96507/9fc20723-bf6c-7268-0bad-897a497b027f.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F96507%2F9fc20723-bf6c-7268-0bad-897a497b027f.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=6d58cf8f111550fd2eea07351b92f93e)

### QR コードエンコーダー / デコーダー

文字列からQRコードを生成。  
逆のQRコードの内容を読み取って文字列の生成もできる。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96507/82426b8a-5107-2a22-b15d-520edb85fbe9.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F96507%2F82426b8a-5107-2a22-b15d-520edb85fbe9.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=280d87b86276d639d1717da797c115af)

### リストの比較

文字列リストの集合演算ができる。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96507/780b126a-03a3-f5d9-7397-dedcd599e450.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F96507%2F780b126a-03a3-f5d9-7397-dedcd599e450.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=18ea2eb0f4ab092e1753544d03bfae7d)

## CLI版

CLI版ではコマンドラインから各種ツールが使えます。  
下記はヘルプを出してみたところです。

```text
>devtoys -h
Description:
  DevToys

Usage:
  devtoys [command] [options]

Options:
  --version       Show version information
  -?, -h, --help  Show help and usage information

Commands:
  lc, listcompare           2つのリストを比較
  esc, escape               パース処理に使われるメタ文字をエスケープまたは解除します
  textutilities, txt        テキストの分析、変換および情報の表示
  xmltester, xsd            XSD を用いた XML データの検証
  jpt, jsonpathtester       JSONPath を用いたテスト
  imageconverter, imgconv   画像フォーマットをロスレスで変換
  cbs, colorblindsimulator  画像またはスクリーンショットの色覚異常シミュレーター
  guid, uuid                バージョン 1 およびバージョン 4 の UUID (GUID) の生成
  password, pwd             パスワード用のランダムな文字列の生成
  li, loremipsum            ダミー (Lorem Ipsum または同様の) テキストを生成します
  checksum, hash            テキストまたはバイナリデータからハッシュ値を生成、検証
  xmlf, xmlformatter        XML データを整形または縮小します
  sqlf, sqlformatter        SQL クエリを整形します
  jsonf, jsonformatter      JSON データを整形または縮小します
  url                       URL エンティティに該当するすべての文字をエンコードまたはデコードします
  qrcode                    QR コードの読み取り、テキストデータから QR コードの表示および PNG / SVG の生成
  html                      HTML エンティティ該当する文字をエンコードまたはデコードします
  gzip                      GZip と Base64 を組み合わせた圧縮 / 展開ツール
  cert, certificate         証明書のデータをデコードして内容を確認します
  b64, base64               テキストデータまたは Base64 データを相互に変換します
  b64i, base64img           画像データまたは Base64 データを相互に変換します
  nb, numberbase            ある基数から別の基数に数値を変換
  jsontotable               JSON 配列を表形式で表示し、CSV、TSVに変換
  jsontoyaml                JSON と YAML を相互に変換します
  date                      Unix 時間や UTC、対人可読形式などへ日時を変換
  cron, cronparser          Cron 式を解析してスケジュールを表示します
```

試しにパスワード生成のコマンドを使ってみました。

```text
>devtoys pwd -h
Description:
  パスワード用のランダムな文字列の生成

Usage:
  devtoys password [options]

Options:
  -l, --length <length>      パスワードの文字数 [default: 30]
  -u, --uppercase            大文字を使用 (ABCDEFGHIJKLMNOPQRSTUVWXYZ) [default: True]
  -m, --lowercase            a から z を使用 [default: True]
  -d, --digits               数字 0123456789 を使用 [default: True]
  -s, --special              特殊文字 !"#$%&')*+,-.:;=>?@]^_}~ を使用 [default: True]
  -e, --excluded <excluded>  生成時に除外する文字 []
  -?, -h, --help             Show help and usage information

>devtoys pwd -l 8 -mds
8rE^5s20
```

CI/CDやスクリプトと組み合わせて便利に使ったり、クライアントPCよりCLIしか利用できないLinuxサーバーに入れることで真価を発揮しそうだと思いました！

## 拡張機能

拡張機能はDevToysには同梱されていないサードパーティのツールを追加するための機能です。

自分で作成したものや [NuGet](https://www.nuget.org/packages?q=Tags%3A%22devtoys-app%22) から取得した拡張機能をDevToysにインストールして使うことができます。

PNG圧縮ツールが公開されていたのでインストールしてみます。

パッケージをダウンロードして、「拡張機能をインストール」からダウンロードファイルを選択して取り込みます。

アプリを閉じて再度開くと新たなツールが追加されています。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96507/4de3726e-f105-50b5-68af-42bb7c68cc43.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F96507%2F4de3726e-f105-50b5-68af-42bb7c68cc43.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a83daed6d146f88171407ee76a21cc38)

本家の対応を待たなくても新しいツールがどんどん追加できるようになっていきそうなアップデートですね。

## その他

リリースにあたっての作者のブログもエモいので一見の価値ありです。

---

個人的に業務PC端末がMacに移ったのもあり、使えなくて歯がゆい思いをしていたので今回のアップデートがとても嬉しいです！

引き続きアップデートがあればこの記事を更新していきます。

[0](https://qiita.com/danishi/items/#comments)

[328](https://qiita.com/danishi/items/a75ab904da66cecc71b4/likers)

307