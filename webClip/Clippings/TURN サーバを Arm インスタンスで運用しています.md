---
title: TURN サーバを Arm インスタンスで運用しています
source: https://zenn.dev/mixi/articles/bcbffa798b084f
author:
  - "[[Zenn]]"
published: 2024-06-25
created: 2025-05-06
description: 
tags:
  - Tech
  - バックエンド
  - バックエンド設計
  - 通信
  - "#クラウド"
read: false
---
[MIXI DEVELOPERS Tech Blog](https://zenn.dev/p/mixi)

20[tech](https://zenn.dev/tech-or-idea)

MIXI でモンストサーバチームとセキュリティ室を兼務している、atpons です。

モンストサーバチームでは、 [モンスターストライク](https://www.monster-strike.com/) や [モンスターストライク スタジアム](https://www.stadium.monster-strike.com/) といったアプリゲームのバックエンド開発や運用を主に行っています。今回は、チームで運用している TURN サーバについて、運用の見直しを行い Arm インスタンスの活用を行っている事例について紹介します。

## モンスターストライクと TURN サーバ

モンスターストライクでは、 [マルチプレイ機能](https://www.monster-strike.com/connect/) を使って最大4人で協力してクエストをプレイが出来るゲームになっています。そのため、プレイヤーのアクションがリアルタイムで他のユーザーに同期されていく必要があります。これに TURN サーバを利用しています。

モンスターストライクにおける TURN サーバの立ち位置や、STUN/TURN の仕組みについては [モンスターストライクのリアルタイム通信を支える技術](https://logmi.jp/tech/articles/321751) という記事でも紹介していますので、ぜひご覧ください。

サービス全体においては、オンプレミスとクラウドを組み合わせたハイブリッドクラウドの構成を取っていますが、TURN サーバにはクラウドのみを利用するようにしています。TURN サーバは仕組み上、グローバル IP アドレスの数でスケールするような構成になっているため、積極的にクラウドを利用することによりスケーラビリティを確保しています。

## モンスターストライクにおける TURN サーバ運用の見直し

モンスターストライクでは TURN サーバについて、以下のような構成を取っていました。

- Amazon Web Services (AWS)
	- EC2 インスタンス
	- 1 インスタンスにつき 2 Elastic IP を付与
- Google Cloud
	- GCE インスタンス
	- 1 インスタンスにつき 1 外部 IP アドレスを付与

モンスターストライクでは、これまで AWS EC2 で TURN サーバを運用してきましたが、2021 年頃より TURN サーバを Google Cloud で立てることで、冗長性を確保してきました。

今回は、これまでの運用の見直しをきっかけに以下の点を変更しました。

- EC2 で立てる TURN サーバの 1 IP 化
- EC2 への Arm インスタンスの導入
- coturn の [Prometheus メトリクス](https://github.com/coturn/coturn/blob/master/docs/Prometheus.md) の収集

EC2 で立てる TURN サーバはインスタンス性能を使い切れないことを考慮してインスタンスに対して 2 つの Elastic IP を付与してきましたが、Arm インスタンスの導入によりコスト効率化を図れるため、1 IP 化も並行して行うことができました。

## Arm インスタンス導入について

EC2 インスタンスには [Graviton](https://aws.amazon.com/jp/ec2/graviton/) を採用したインスタンスタイプが用意されており、Arm インスタンスを簡単に構築することができます。

TURN サーバについては、 [coturn](https://github.com/coturn/coturn) を利用していますが、バグフィックスなどが含まれている最新版を利用したい、ディストリビューションのリポジトリからだと Prometheus 対応していないので、対応したビルドを作りたいので、ソースからビルドしています。

基本的にはマニュアルに沿ってソースからビルドすることで、対応したビルドを作成することができたので特に苦労することもなく TURN サーバを Arm 化することができました。

Arm 化後、特に不具合もなく本番で運用することができています。また、コストについてもこれまで大きいインスタンスを利用していたのを、小さいインスタンスで捌けることが分かったため、その分 IP アドレスを 1 つだけにしても、コストを削減することができました。

## 導入に際して同時に行ったこと

Arm インスタンスの導入と同時に、AWS [Systems Manager](https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/what-is-systems-manager.html) / Google Cloud [Identity-Aware Proxy](https://cloud.google.com/iap/docs/using-tcp-forwarding?hl=ja) ・ [OS Login](https://cloud.google.com/compute/docs/oslogin?hl=ja) による管理の効率化も行っています。これにより、SSH を行って管理する際はクラウドの認証情報と連携しているので、普段トラブルシュートを行うことが少ない・問題が起きた場合には作り直せばいい、といった TURN サーバにおいては、プロビジョニングの手間を低減できる、このようなサービスが非常に合っています。

## 終わりに

今回は TURN サーバを Arm インスタンスで問題なく運用している話について書きました。TURN サーバにおいては、パフォーマンス比較というよりは、単純に捌けるコネクション数がポート数に比例してしまうので、そこまでのポートをきちんと捌ければ特に性能差について気にする必要がありません。このため、 Arm インスタンスでコストパフォーマンスを最適化していくのにぴったりな用途とも言えます。ぜひ参考になれば幸いです。

20

### Discussion

![](https://static.zenn.studio/images/drawing/discussion.png)

ログインするとコメントできます