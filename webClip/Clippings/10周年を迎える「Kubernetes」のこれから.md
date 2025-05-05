---
title: 10周年を迎える「Kubernetes」のこれから
source: https://japan.zdnet.com/article/35219744/
author:
  - "[[ZDNET Japan]]"
published: 2024-06-06
created: 2025-05-06
description: コンテナーオーケストレーションのデファクトスタンダードであるグーグルの「Kubernetes」が10周年を迎えた。クラウドネイティブコンピューティングの進化とともに、Kubernetesは今後、どのような方向に進んでいくのだろうか。
tags:
  - 1
  - Tools
  - "#Kubernetes"
read:
---
- [Tweet](https://twitter.com/share?ref_src=twsrc%5Etfw)
- [noteで書く](https://note.mu/intent/post?url=https%3A%2F%2Fjapan.zdnet.com%2Farticle%2F35219744%2F&ref=https%3A%2F%2Fjapan.zdnet.com%2Farticle%2F35219744%2F&hashtags=ZDNET)
- - 印刷する
- - メールで送る
	- テキスト
	- HTML
	- 電子書籍
	- PDF
- - ダウンロード
	- テキスト
	- 電子書籍
	- PDF
- - クリップした記事をMyページから読むことができます

　「Linux」、クラウド、コンテナー、「Kubernetes」のいずれかがなかったとしたら、現在のようなテクノロジーの世界は実現しないだろう。Linuxはすべての基盤となるOSだ。すべてのアプリケーションとリソースへのアクセスはクラウドが担っている。アプリケーションを格納しているのがコンテナーだ。そしてすべてのコンテナーのオーケストレーションはKubernetesが担当している。このうちのどれか1つでもなくなると、われわれの生活と仕事の世界はもっと原始的なものになってしまう。

　進化を続けるクラウドネイティブコンピューティングにおいて、Kubernetesほど大きな影響力を持つ技術はほとんどない。10周年を迎えるKubernetesは、オープンソースが持つ協力と革新の力を証明する存在になっている。Googleでつつましく始まったKubernetesは、アプリケーションのデプロイ、管理、スケーリングのあり方を変え、コンテナーオーケストレーションのデファクトスタンダードになった。

　今後、Kubernetesが減速する兆しはない。プラットフォームとして進化を続けており、新機能や改良が定期的に加えられている。Kubernetesのコミュニティーは、ユーザー体験のシンプル化、セキュリティの向上、スケーラビリティーの強化などの方策を模索している。

　Chainguardの共同創業者でKubernetesの開発者の1人でもあるVille Aikas氏は次のように述べている。

> 　Cloud Native Computing Foundation（CNCF）がこのように大きく花開いたのは、プラットフォームチームにもたらされるツールとインフラのオプションの多様性の点ですばらしいことだ。それだけでなく、これによりKubernetesを運用するために必要な選択肢もたくさん生み出され、その光景が広大なものになったものと考えている。いつも思うのは、Kubernetesがこのような人気を博した大きな理由の1つは、APIがシンプルで、使用する認知負荷が比較的少ないことだと常々思っている。Kubernetesは成熟し続けるが、メンタルモデルのシンプルさとAPIの使いやすさはなんとかして維持する必要がある。

　言うは易く行うは難しだ。Kubernetesとクラウドネイティブコンピューティングのパラダイムをうまく使いこなすのは次第に難しくなってきた。

　eBPFパフォーマンスモニタリング企業Groundcoverの共同創業者で最高経営責任者（CEO）を務めるShahar Azulay氏は次のように述べている。

> 　Kubernetesは多様なタスクを効果的に管理できる能力が証明されているが、その複雑さから、相当なセットアップと継続的なメンテナンスが必要だ。Linuxが信頼できるOSへと発展していったのと同じように、Kubernetesはユーザーフレンドリーな抽象化レイヤーへと変容していくだろう。Kubernetesは10年間で採用の拡大が続き、効率とコスト最適化に対するニーズがますます重要になってきている。

　間もなく実現する楽しみな展開の1つが、Kubernetesとサーバーレスコンピューティングとの統合だ。「Kubeless」や「Fission」などのプロジェクトでは、Kubernetesにサーバーレス機能を組み込み、開発者が既存のKubernetesクラスターに加えてFaaS（Function as a Service）をデプロイできるようにしようとしている。サーバーレスコンピューティングとKubernetesの統合は、クラウドネイティブアプリケーションの新たな可能性に扉を開くに違いない。

　エッジコンピューティングとKubernetesの組み合わせも増えている。エッジに移るデバイスとアプリケーションが増加し、エッジへのデプロイに対応するためにKubernetesが採用されている。Kubernetesのコミュニティーは、エッジデバイスで実行できる軽くて効率的なKubernetesクラスターを実現するため、「KubeEdge」「MicroK8s」「Red Hat Device Edge」などのプロジェクトに取り組んでいる。

![提供：rvbox/Getty Images](https://japan.zdnet.com/storage/2024/06/06/dd97a486fa4ffe135c941c8807502f5c/steersailgettyimages-524856023_1280-1.jpg)  
提供：rvbox/Getty Images

この記事は海外Red Ventures発の [記事](https://www.zdnet.com/article/kubernetes-turns-10-how-it-steered-cloud-native-computing-for-the-last-decade-and-whats-next/) を朝日インタラクティブが日本向けに編集したものです。

ZDNET Japan 記事を毎朝メールでまとめ読み（登録無料）

[メールマガジン登録のお申し込み](https://japan.zdnet.com/newsletter/)

### ホワイトペーパー

#### 新着

- ビジネスアプリケーション
	[
	NTTデータの基幹システム刷新に学ぶ、柔軟性と継続的進化を支える新たな業務基盤とは
	](https://japan.zdnet.com/paper/30000956/30008280/)
- OS
	[
	Windows 11移行に備える--PCの運用管理とコスト最適化を両立する実践的アプローチとは
	](https://japan.zdnet.com/paper/30001793/30008279/)
- ビジネスアプリケーション
	[
	RAG やベクトル埋め込みは可能か、生成 AI 活用で求められるデータベースの要件を探る
	](https://japan.zdnet.com/paper/30001001/30008277/)
- 運用管理
	[
	AWSに移行することのメリットと複雑さ--監視ソリューションの導入から活用までを徹底解説
	](https://japan.zdnet.com/paper/30001584/30008278/)
- モバイル
	[
	DX推進を阻む情シスの業務課題、解決に役立つ7アイデアと4つのソリューション
	](https://japan.zdnet.com/paper/30001737/30008271/)

#### ランキング

1. ビジネスアプリケーション
	[
	生成 AI を活用した革新的な事例 62 選　課題と解決方法を一挙紹介
	](https://japan.zdnet.com/paper/30001001/30008155/)
2. ビジネスアプリケーション
	[
	調査結果が示す「生成 AI 」活用によるソフトウェア開発の現状、ツール選定のポイントも解説
	](https://japan.zdnet.com/paper/30001001/30008154/)
3. セキュリティ
	[
	新入社員に教えるべき情報セキュリティの基礎知識--企業全体を守るための基本ルールを徹底解説
	](https://japan.zdnet.com/paper/30001727/30008229/)
4. セキュリティ
	[
	【実態調査】情シス担当者に聞いた--中小企業の4割以上がセキュリティ人材の確保に課題
	](https://japan.zdnet.com/paper/20022581/30008245/)
5. セキュリティ
	[
	KADOKAWAらの事例に学ぶ、2024年サイバー攻撃の傾向と対策
	](https://japan.zdnet.com/paper/30001727/30008225/)

[ホワイトペーパーライブラリー](https://japan.zdnet.com/paper/)

## ZDNET Japan クイックポール

所属する組織のデータ活用状況はどの段階にありますか？

ZDNET Japanは、CIOとITマネージャーを対象に、ビジネス課題の解決とITを活用した新たな価値創造を支援します。  
ITビジネス全般については、 [CNET Japan](https://japan.cnet.com/) をご覧ください。