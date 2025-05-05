---
title: TerraformとAnsibleで作るさくらのクラウドのKubernetesクラスタ
source: https://knowledge.sakura.ad.jp/38914/
author:
  - "[[さくらのナレッジ]]"
published: 2024-08-29
created: 2025-05-06
description: CAMPHOR-の上田蒼一朗です。今回は「TerraformとAnsibleで作るさくらのクラウドのKubernetesクラスタ」というテーマでお話しします。よろしくお願いします。 目次自己紹介CAMPHOR-とはCAM […]
tags:
  - 1
  - Terraform
  - インフラ
  - クラウド
  - IaC
read: false
---
![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/08/camphor-202407-1-1340x754.jpg)

CAMPHOR-の上田蒼一朗です。今回は「TerraformとAnsibleで作るさくらのクラウドのKubernetesクラスタ」というテーマでお話しします。よろしくお願いします。

## 自己紹介

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/08/camphor-202407-2-680x383.jpg)

まず自己紹介です。上田蒼一朗と申します。今年から京都大学の大学院生としてやっています。昨年度(2023年度)、未踏スーパークリエータに認定されました。技術領域としては、低レイヤとかインフラとかウェブとか、いろいろ好きで、結構何でもやるという感じです。CAMPHOR-では、これからお話しするKubernetesクラスタの運用をやっていたり、他にもいろいろとソフトウェアの管理をしていたりします。

## CAMPHOR-とは

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/08/camphor-202407-3-680x383.jpg)

まずCAMPHOR-(かんふぁー)がどういう組織なのかを軽くご紹介しようと思います。

[CAMPHOR-](https://camph.net/) というのは京都のIT系の学生コミュニティなのですが、京都の古民家を拠点にしているというのが特徴になっています。2010年から始まっていますので、今年でたぶん14年目になります。現在8人の学生でこのコミュニティの運営をしています。Slackには卒業した人も含めて200人ぐらいいますので、コミュニティとしての規模はそれぐらいになっています。

コミュニティの活動として何をしているのかを具体的に挙げますと、CAMPHOR- HOUSEの運営、そしてイベントや勉強会の開催をしています。

### CAMPHOR- HOUSE

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/08/camphor-202407-5-680x383.jpg)

CAMPHOR- HOUSEは、このCAMPHOR-という団体の拠点となっている古民家なのですが、ここを、学生だったら誰でも使えるコワーキングスペースとして開放しています。この家はWi-Fiでネットワークが使えたり、本棚には技術書がたくさん並んでいるのでその貸し出しをやったりしています。モニターやホワイトボード、冬には炬燵があったりして、作業をしやすい環境になっています。来ている学生たちは、そこで開発をしていてもいいし、大学の課題をしていてもいいし、集まった人達で雑談したり、技術的な話で盛り上がったり、自由に過ごせる空間になっています。

### イベントや勉強会

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/08/camphor-202407-6-680x383.jpg)

イベントや勉強会もいろいろやっています。イベントは企業と一緒にやっていることも多く、今年は「クラウドを支えるインフラ技術」というテーマで、サイバーエージェントさんに来ていただき、OpenStack(クラウドを構築できるオープンソースのソフトウェア)を実際にさわってみようというイベントをしました。

また、昨年はSTORESさんと一緒に「鴨川ハッカソン」というイベントをやりました。2日間のハッカソンなのですが、鴨川でやるというのが特徴です。鴨川は京都市内を流れる川なのですが、その川のそばにStarlinkを置いて、屋外でもインターネットを使えるようにし、お日さまの下で秋風を感じながらハッカソンをする、ということをしました。

こういういろいろなイベントや勉強会に、学生さんに来てもらって楽しんでもらうということをしています。これらがCAMPHOR-がやっていることです。

## CAMPHOR-を支える技術・kuina

今回は、CAMPHOR-を支える技術の中から、kuinaというKubernetesクラスタについてお話しします。

### CAMPHOR-が管理しているサービス群

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/08/camphor-202407-8-680x383.jpg)

CAMPHOR-では、ウェブサービスをいろいろと管理しています。その中のいくつかをご紹介します。まず、CAMPHOR-のウェブサイトがあります。こちらには、コミュニティの概要や、CAMPHOR- HOUSEがいつ開いてるかといったスケジュールなども書いてあります。

また、昨年ぐらいからMisskeyサーバを立てています。 [Misskey](https://misskey.io/) というのは、ActivityPubという仕様にのっとったオープンソースのSNSなのですが、ActivityPubに準拠していれば、他のActivityPubに対応しているSNS(例えばMastodonなど)とつなげることができます。各自がActivityPubに対応したサーバを立てると、それらが連携してひとつのSNSのようにふるまうため、分散SNSと呼ばれていたりもします。そのMisskeyサーバをCAMPHOR-でも立てて、誰でも登録してもらえるようにしています。

あとはブログがあります。これはWordPressを利用していて、開催したイベントのレポートを書いたりしています。

さらに、CAMPHOR- HOUSEで書籍を貸し出しているという話をしましたが、それを管理するために内製した書籍管理システムを動かしたりもしています。

### Kubernetesクラスタ kuina

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/08/camphor-202407-9-680x383.jpg)

これらのサービスたちをどうやって動かしているかというところで、Kubernetesクラスタkuinaが出てきます。この上でいろいろなサービスを動かしています。

このKubernetesクラスタは、 [K3s](https://k3s.io/) という、組み込みでも使える軽量なKubernetesのディストリビューションがあって、それを使って構築しています。

CAMPHOR-は、さくらインターネットさんから学生コミュニティへの支援として [さくらのクラウド](https://cloud.sakura.ad.jp/) を使わせていただいているので、さくらのクラウド上にkuinaを構築して運用しています。

CAMPHOR-のサービス名は京都の地名から取るという慣習があるのですが、このKubernetesクラスタは「くいな橋」という京都の駅の名前からとってkuinaになりました。後付けで「KUbernetes INtegrated Applications」の略にもなっているという話があったりもします。規模的には5台のNodeで動いていて、Podが90個ぐらい立っているという規模感です。

### ArgoCDによるGitOps

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/08/camphor-202407-10-680x383.jpg)

ここから、kuinaをどのように管理しているかという話をしていきます。

まず、マニフェストの管理には [ArgoCD](https://argoproj.github.io/cd/) を使っています。ArgoCDは、KubernetesにおけるGitOpsを実現するオープンソースのソフトウェアで、Kubernetesに適用するマニフェストをGitHubにpushすると、それをArgoCDが自動的に拾ってapplyしてくれるという、GitHubの状態とKubernetesの状態を一致させてくれるソフトウェアとなっています。

マニフェストを書き換える部分についても、例えばウェブサイトを更新した場合、そのコミットをGitHubにpushすると、コンテナイメージを作成して、新しいコンテナイメージを使うようにマニフェストを書き換えるという部分を [GitHub Actions](https://github.co.jp/features/actions) で自動化しているので、Kubernetesの知識がなくてもCAMPHOR-内のサービスを開発して、自動でデプロイすることが可能になっています。運営しているメンバーが全員Kubernetesに詳しいわけではないので、こういった仕組みを使って誰でもサービスをさわれるようになっています。

### ネットワーク

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/08/camphor-202407-11-680x383.jpg)

マネージドなKubernetes as a Serviceの機能を使うことはできないので、ネットワーク周りについても自分たちで構築する必要があります。

まず、クラスタ内のサービスをクラスタの外部に公開するために [ingress-nginx](https://github.com/kubernetes/ingress-nginx) コントローラを使い、Ingressリソースを立ててサービスを外に公開するということをしています。さらにそのingress-nginxコントローラに対して、クラスタの外部からアクセスできるIPアドレスを付与する必要があるので、そこは [MetalLB](https://metallb.universe.tf/) を使っています。

これらのKubernetesクラスタは、さくらのクラウドでVPCを構築して、そのVPCの中でそれらを構築しています。よって、全体的な構成としては、まずインターネットからリクエストが飛んできて、それをさくらのクラウドのVPCルータで受けて、そのVPCルータがMetalLBで払い出されたIPアドレスに対してリクエストをフォワードすることによって、リクエストをKubernetesクラスタ内に届けています。この辺の細かい構成などは後ほどお話しします。CNIは [Calico](https://www.tigera.io/project-calico/) を使っています。

### ストレージ

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/08/camphor-202407-12-680x383.jpg)

ストレージに関しては、CAMPHOR-のサービスにはデータの永続化が必要なものがいくつかあるため、Podに対してPersistent Volume(永続化ボリューム)を払い出せる必要があり、そのための仕組みを [Rook Ceph](https://rook.io/) (るーくせふ)というOSSで構築しています。

Rook Cephというのは、Kubernetesのオペレータの仕組みを使って、マニフェストを書くだけでKubernetesクラスタ上でCephが立ち、そのCephがPod等に対してPersistent Volumeを払い出すという仕組みを作ってくれます。これによってPersistent Volumeなどを使えるようにしています。

さくらのクラウドで立てた各VMに、フォーマットされていない空のディスクを挿しておくと、Rookの機能で自動的に空のディスクをCephのストレージとして使うように設定してくれるので、こういった仕組みを使って、クラスタをスケールさせるとCephクラスタのストレージもスケールするような仕組みにしています。この辺の仕組みについてはまた後ほどお話しします。

### その他にkuinaでやっていること

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/08/camphor-202407-13.jpg)

その他に使っているものを紹介します。

まず [Prometheus](https://prometheus.io/) や [Grafana](https://grafana.com/ja/) を使ってクラスタ内のモニタリングをしています。それからKubesealは、GitOpsでマニフェストを管理しているので、機密情報を暗号化してGitHubにアップロードするための仕組みとして導入しています。 [cert-manager](https://cert-manager.io/) は証明書の管理を自動化するものです。また、手元のkubectlからkuinaのクラスタにアクセスするために、 [Auth0](https://auth0.com/jp) を使った認証認可をやったりしています。

## さくらのクラウドを用いたインフラ構築

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/08/camphor-202407-14.jpg)

ここまではkuinaのクラスタ内のコントローラやアプリケーションをどうやって管理するかという話でしたが、ここからは、このkuina自体のインフラをどうやって管理しているのかということについてお話しします。

先ほども言ったように、kuinaはさくらのクラウドを使って構築していて、ネットワークの構成としてはVPCの中にすべてのサーバを入れています。

さくらのクラウドでVPCを構築する場合は、VPCルータとスイッチを使って構築します。まずスイッチですべてのサーバとルータをL2で接続し、各サーバにはプライベートIPアドレスを振ります。このままでは外からサーバにアクセスすることができないので、VPCルータのポートフォワーディングという機能を使ってそれを可能にしています。つまり、インターネットからVPCルータのグローバルIPアドレスに向けてリクエストが飛んでくると、それをKubernetesクラスタの方にフォワーディングするという設定をすることによってインバウンド通信を受け入れる、ということをしています。

## Terraform for さくらのクラウド

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/08/camphor-202407-15.jpg)

これらのインフラは [Terraform](https://www.terraform.io/) を使って構築し管理しています。 [さくらのクラウドにはTerraformプロバイダがある](https://docs.usacloud.jp/terraform/) ので、先ほど紹介した構成はすべてTerraformを使って記述することができます。このように構成をコード化して、Infrastructure as Code(IaC)を実現しています。こうすることで、GitHubを使ってTerraformを管理をすることが可能になります。構成の変更などもTerraformのコードを書き換えてGitHubにpushすると、Terraform Cloudを使って自動的にapplyされる仕組みになっています。

### Terraformのコードを見てみる

Terraformのコードを見ながら、どういう設定になっているかをご紹介しようと思います。

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/08/camphor-202407-16.jpg)

まずこれがkuinaの各サーバのリソース定義です。このようにサーバのスペックなどを指定してサーバを立てることができます。下の方はネットワークまわりの設定です。例えば、各サーバのプライベートIPアドレスや、各サーバのNICをスイッチに接続することや、ゲートウェイのIPアドレスや、そういったものを設定することができます。こうすることで、例えばTerraformをいじってクラスタをスケールアウトさせると、新しくできたサーバが自動的にこのVPCの中に入って動き出すということになります。

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/08/camphor-202407-17.jpg)

次に、VPCルータのリソース定義です。ここで紹介したいのが、ポートフォワーディングの設定もTerraformで管理できるということです。上図にポートフォワーディングの設定のエントリーが2つあります。上の方は443番のTCPに来たらフォワードすると書いてあるので、これはHTTPSのリクエストになりますね。これをprivate\_ip行に書いてあるプライベートIPアドレスにフォワードするのですが、これがMetalLBによって払い出された、クラスタ外部からアクセスできるIPアドレスになっているので、HTTPSのリクエストがVPCルータに届くと、それがMetalLBによって払い出されたIPアドレスにフォワードされ、そのままingress-nginxコントローラに渡るということになります。下の方はprivate\_portが6443番となっていますが、これはkuinaのコントロールプレーンのAPIです。これにアクセスできるように設定しています。

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/08/camphor-202407-18.jpg)

もう1つご紹介したいのが、サーバに挿すディスクのリソース定義についてです。各VMにはディスクを2本挿しています。まず1つ目はUbuntuがインストールされたディスクで、ここからLinuxが起動します。もう1つはraw\_diskと書いてあるように、フォーマットされていない空のディスクをVMに挿しています。これが先ほどもご紹介した、Rookの機能によって空のディスクが自動的にCephのディスクとして使われるというものです。

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/08/camphor-202407-19.jpg)

ここまで紹介したコードはTerraformのモジュールとして定義されているので、これを呼び出すことによって、kuinaのすべてのリソースが立ち上がってくることになります。kuinaをスケールさせたい場合、例えばスケールアウトさせたいのであればserver\_countを増やせばよく、スケールアップさせたいのであればserver\_coreやserver\_memoryを変更するだけで可能です。

Terraformのコードをモジュール化するもう1つのメリットとして、環境ごとに値を変えて構築できるというのがあります。CHAMPHOR-ではテスト環境としてもう1つのkuinaを構築していて、新しい運営メンバーが加わったときにkuinaに関してキャッチアップするときの実験場として使うということもしています。

## Ansibleを使った自動化

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/08/camphor-202407-20.jpg)

ここまで、Terraformを使ってさくらのクラウドのVMを管理していることをご紹介しましたが、その各VM上のソフトウェア、つまりKubernetesなどを構築する部分も自動化していて、そちらは [Ansible](https://www.ansible.com/) を使っています。Ansibleを使うことによって、各サーバに対してK3sのインストール、設定、実行を自動的に行っています。このAnsibleのコードについては、K3sのコミュニティが公式に出しているAnsibleのプレイブック ["k3s-ansible"](https://github.com/k3s-io/k3s-ansible) をもとに記述していきました。

## Terraform＋Ansibleでスケールを簡単に実現

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/08/camphor-202407-21.jpg)

TerraformとAnsibleでやっていることをいろいろとご紹介しました。これらを合わせることにより、スケールがすごく簡単に実現できるというのがうれしい点です。スケールアウト、スケールアップ、スケールダウン、何でもよいのですが、そういうスケールをさせたいと思ったら、わずか2ステップで実現できます。まず、クラスタの構成を変更するためにTerraformを修正してGitHubにpushすると、Terraform Cloudを使って自動的にapplyされます。スケールアウトさせた場合は、新しくできたサーバに対してAnsibleを回すことによって、すぐにkuinaのクラスタに参加できるようになります。このように、2ステップで簡単にスケールきるというのがうれしいポイントです。

実際に運用していて特にうれしかったのがMisskeyサーバを立てたときでした。Misskeyサーバを立てても、どれぐらいリソースが使われるか読めない部分があったのですが、このように簡単にスケールができるため、インクリメンタルに様子を見ながらやることができました。最初にサーバを立てると、どんどんユーザーが増えていきます。そうすると使用するリソースもどんどん増えていくので、それに応じてスケールアウト・スケールアップさせながらいい感じに設定することができました。

## まとめ

![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/08/camphor-202407-22.jpg)

我々CAMPHOR-は、さくらのクラウド上でKubernetesクラスタkuinaを構築して運用しています。このときに、インフラの構成をすべてコード化するということをしています。具体的にはTerraformを使ってさくらのクラウドのVMを管理し、Ansibleを使ってその上のパッケージなどを管理し、クラスタ内の構成に関してはArgoCDを使ってマニフェストGitHubで管理しています。このようにOSSを活用すると、さくらのクラウドの管理もいろいろと自動化して便利に使うことができます。みなさんもいろんなOSSを使いながら、さくらのクラウドを便利に使ってみるのはいかがでしょうか。ありがとうございました。