---
title: 30分で入門する Helm
source: https://blog.jp.square-enix.com/iteng-blog/posts/00044-helm-in-30min/
author:
  - "[[SQUARE ENIX CO.]]"
  - "[[LTD.]]"
published: 
created: 2025-05-06
description: Kubernetes の利用シーンは幅広い用途に広がり、長期計画でカスタムアプリケーションを開発してデプロイする以外にも、ぱっと cluster にアプリケーションを入れて使ってみるといったことも多く見られるようになりました。単純なアプリケーションでは kubectl apply で済むものも多いですが、じゃっかん複 …
tags:
  - Tech
  - Tools
  - Kubernetes
  - バックエンド
  - バックエンド設計
  - 学習系
read: false
---
Kubernetes の利用シーンは幅広い用途に広がり、長期計画でカスタムアプリケーションを開発してデプロイする以外にも、ぱっと cluster にアプリケーションを入れて使ってみるといったことも多く見られるようになりました。  
単純なアプリケーションでは `kubectl apply` で済むものも多いですが、じゃっかん複雑な構成のものや、また変数を使って動作を変更したいときなどでは Helm がよく使われています。

時々 Kubernetes について話しているときに、 `kubectl` はよく使うことになったが `helm` はまだ慣れていないんだよねという声も聞かれるようになりました。  
ここでは、そういったケースでの `kubectl` -> `helm` 間のギャップを埋めることを想定して、Helm の全体感がわかって日常的に使えるようになるまでを 30 分くらいでちゃちゃっとやってみましょう。

## おさらい: kubectl

kubectl は Kubernetes cluster (master) と会話するための cli です。cluster への権限を前提として、情報の取得や更新を行います。まずはこれができないと先に進めません。

こういう感じで使います。

```console
$ kubectl get pods -n default
```

console

今回は GKE や minikube などですでに Kubernetes cluster の準備があり、ここまでは理解している・できているものとして先に進めます。

## とにかく Helm を使ってみる

まず、手元で helm を使えるようにします。 [公式の install 手順](https://helm.sh/docs/intro/install/) をご参照ください。macOS では `brew` 、Windows では `choco` の方法が書かれています。 `kubectl` は `gcloud components` で入れることもできますが、 `helm` は見たところ無いようです。お手元の環境に合わせて install してください。

cluster に対して `kubectl` が実行できるようになって (通信と権限が設定されて) いれば、以下のようにして cluster に install されている chart を調べることができます。

```console
$ helm list

NAME    NAMESPACE       REVISION        UPDATED STATUS  CHART   APP VERSION
```

console

install されている chart が無い状態ではこのように、空の表が出力されます。  
chart というのは helm で install する単位です。詳細は以下で見ていきましょう。

### Helm でできること

Helm は、Kubernetes の package manager と言われています。それがどういうことなのか、試してみましょう。

#### Chart Repository から install する

[こちら](https://helm.sh/ja/docs/intro/quickstart/) によい例があります。  
以下は Kubernetes に mysql を入れてみる例です。

```console
$ helm repo update

$ helm install stable/mysql --generate-name
```

console

上記を実行した後、どのようなものができているか `kubectl get pods` などして見てみるとよいでしょう。  
削除するには以下のようにします。install 時に `--generate-name` option を使っているので、 `<install-name>` は自動で設定 (生成) されています。 `helm list` して名前を調べておきましょう。

上記は Helm を入れたときに最初から利用できるようになっている標準 repository から install する例ですが、repository を追加することもできます。わたしの手元ではすでにいくつか追加されており、以下のようになっていました。何を使っているかが伺えますね。初期状態ではひとつめの `stable` のみ存在していると思います。

```console
$ helm repo list

NAME                URL

stable              https://charts.helm.sh/stable

concourse           https://concourse-charts.storage.googleapis.com/

external-secrets    https://charts.external-secrets.io

fluent              https://fluent.github.io/helm-charts

external-dns        https://kubernetes-sigs.github.io/external-dns/
```

console

これは Docker でいう container repository や、Ubuntu の package 管理ツールである apt の package repository とおなじように、Helm の場合は chart がある場所を管理しています。  
install する chart によって、この repository の追加手順もあわせて説明されていることがあるので、それに沿って追加するといいでしょう。([Concourse の例](https://github.com/concourse/concourse-chart))

#### 自前の chart をつくる

ここまでは誰かがつくった chart を利用する手順でした。  
今度は自分で chart をつくり、Kubernetes に install する内容を管理する方法を見てみます。

[https://helm.sh/docs/chart\_template\_guide/getting\_started/](https://helm.sh/docs/chart_template_guide/getting_started/) (日本語版無さそう)

非常によい tutorial になっていますので、ぜひ説明を読みながら頭からやってください。  
chart を手元につくり、それを cluster に install します。中身は単純な ConfigMap だけなので、安全に単純な内容を確認できそうです。

上記 tutorial でもかんたんなものは使っていますが、template はもう少し見ておきたい、便利で強力な機能です。特に、 `helm` 実行時に外部から 変数値 (values) を与える機能は、おなじものを少し変えて deploy するのに非常に便利です。

[https://helm.sh/docs/chart\_template\_guide/values\_files/](https://helm.sh/docs/chart_template_guide/values_files/)

こちらのページを見ていただくとわかりますが、 `helm` 実行時に `--set` で直接指定、または template とは別に変数を定義したファイルを `-f` で指定することにより、変数値を渡すことができます。template 内では `{{ .Values.xxxx }}` のように変数を参照することで `helm` 実行時に値が展開されます。  
この template は Go template の仕組みが使われていて、非常に柔軟な機能を完結に記述できます。実行時にどのように展開されるかは、 `--dry-run` や `--debug` を使うことで確認できます。  
ぜひ、こちらのページの内容も実際にお手元で実行してみてください。

ここまでやると、たとえば node で実装した web app と MySQL 等の database を含む環境をひとつの chart にまとめ、名前や設定を調整しながら複数の環境として deploy するのがかんたんにできそうなイメージがついたのではないでしょうか。Ansible などでも似たようなことをしますが、よりまとまりよく・便利に管理できるように感じませんか？

## まとめ

Kubernetes は使っているけど、Helm に手を出すのはおっくう… というかた向けに、かんたんなガイドを書いてみました。あとはすこしずつ複雑度を増やしていくことで、大規模なアプリケーション管理もすぐにできるようになるでしょう。

#### この記事を書いた人

在宅適応力が高すぎることにじゃっかんの危機感を覚えつつあります。 `小麦粉` をこねて焼いたものを好んで食べます。高温・多湿な環境が苦手です。

興味をお持ちいただけたら、ぜひ採用情報ページもご覧下さい！