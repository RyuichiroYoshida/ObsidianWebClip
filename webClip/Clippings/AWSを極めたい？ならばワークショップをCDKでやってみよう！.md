---
title: AWSを極めたい？ならばワークショップをCDKでやってみよう！
source: https://qiita.com/har1101/items/02b5129cdf6101d6afc0
author:
  - "[[Qiita]]"
published: 2024-09-01
created: 2025-05-06
description: ANGEL Calendar開幕！2024年ANGEL Dojo参加者用アドベントカレンダー「ANGEL Calendar」スタートしました〜！ https://qiita.com/har1101…
tags:
  - Tech
  - インフラ
  - クラウド
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

[@har1101](https://qiita.com/har1101) in [2024-ANGEL-Dojo](https://qiita.com/organizations/2024-angel-dojo)

投稿日

## ANGEL Calendar開幕！

[2024年ANGEL Dojo](https://aws.amazon.com/jp/blogs/psa/2024-07-angel-dojo-kickoff/) 参加者用アドベントカレンダー「ANGEL Calendar」スタートしました〜！

本記事は記念すべき1日目の投稿となっております！

9月1日から30日まで、毎日記事が更新されていきますので [2024-ANGEL-Dojo Organization](https://qiita.com/organizations/2024-angel-dojo) もチェックしてみてください！

## はじめに

こんにちは、ふくちと申します。  
今回、ANGEL Calendarの企画&運営をしております。

記念すべき1本目ということで、私からは **「AWSの理解度・技術力向上にはCDKとAPI Referenceがいいぞ…」** という話をさせていただければと思います！

## 記事作成背景

AWSのハンズオンをする時、このような課題があると個人的に考えています。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/ee36fb04-a7f5-2dfb-ec61-85ff5a0daf39.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2Fee36fb04-a7f5-2dfb-ec61-85ff5a0daf39.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=597829287c2c794904ec9ab0e6dcb8b2)

せっかくハンズオンをするのであれば、より実践で生きる力に昇華させたいですよね。

そこで、個人的に行なっている学習法として、 **「AWSのハンズオンをCDKでやってみる」** というのがあります！

これを行うメリットは大きく3つあります。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/c833264b-14ae-f9aa-640f-abf0619e9c00.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2Fc833264b-14ae-f9aa-640f-abf0619e9c00.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=f61d609af423c3078135fe268cc286c4)

ただハンズオンの手順をなぞるだけより、より効果的な学習ができると思うので、ぜひ共有させていただきたく思います！

### 当記事の想定読者

- AWS SAA相当の知識をお持ちで、さらにレベルアップしたい方
- AWS CDKを触ったことがない方
	- 知識がなくても進められるようにしていますが、CDKの構成要素・仕組みなどは基本的に説明省略しているので、そこは各自で知識を補完してください

難易度に個人差があると思うので、自分に合ったペースでやっていただければと思いますmm

また、CDKもTypeScriptもわからん！という方は [こちらの開発入門ワークショップ](https://catalog.workshops.aws/typescript-and-cdk-for-beginner/ja-JP) を先にやってみるのもおすすめです！

## やり方

基本的な流れとしては、以下の通りです。

1. コンソール上でワークショップをやってみる
2. ワークショップの内容をCDKで実装する  
	2.1. API Referenceで実装方法を調べる  
	2.2. 実際にコードを書く  
	2.3. コンソールとAPI Referenceを見比べて設定を確認する  
	2.4. CDKでデプロイし、AWSリソースを作成する

**とりあえず、これだけ覚えて実際にやってみてください！！！  
この記事で言いたいことはこれだけです！！！**

「とは言っても、CDK書いたことないしAPI Referenceとかよくわからん…」という方向けに、この後具体的な流れをご紹介します。

ですのでこの続きでは、ワークショップの内容をCDKに実装するフェーズをやっていきます！  
※当記事ではコンソール上のワークショップ手順を省略しています。

## ワークショップの準備

今回実施にするのはこちらの [「実践力を鍛えるBootcamp -クラウドネイティブ編-」ワークショップ](https://catalog.us-east-1.prod.workshops.aws/workshops/a9b0eefd-f429-4859-9881-ce3a7f1a4e5f/ja-JP) です。

下記のように、主要なAWSサービスを一通り構築することができます。  
また、 [ワークショップの中で開発環境を手軽に準備する方法](https://catalog.us-east-1.prod.workshops.aws/workshops/a9b0eefd-f429-4859-9881-ce3a7f1a4e5f/ja-JP/setup-vscode/02-configure-ide) も掲載されており、非常に始めやすいです！初心者の方もぜひやってみてください！  
(余談：ANGEL Dojoの運営をしてくださっている宇賀神さんが作成されたワークショップだったりします)

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/32716bb4-fcc1-f9a3-624c-5ea3e51a0f8e.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F32716bb4-fcc1-f9a3-624c-5ea3e51a0f8e.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=1472da3bd5d531cc447e9910fcf0f58b)

ここでは、CDKを用いて構築する際の準備を行います。  
ターミナルで下記コマンドを実行し、CDKのプロジェクトを作成します。

```bash
cdk init sample-app --language typescript
```

すると、CDKのサンプルプロジェクトが作成されます。基本的には、 lib フォルダ配下の〇〇stack.tsを書き換えていくことになります。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/d8169911-221c-8027-02b2-4b63b9b853ed.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2Fd8169911-221c-8027-02b2-4b63b9b853ed.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=f3dac0abc2a55e2566c2bdcd4010db24)

また、CDKをデプロイする際に必要なものを準備するために、下記コマンドも実行します。

```bash
cdk bootstrap
```

実行後、CloudFormationコンソールに「CDKToolkit」スタックが作成されます。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/022fa680-6476-f0e2-69ba-65fe31563ae8.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F022fa680-6476-f0e2-69ba-65fe31563ae8.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=5a17e9db2497848aa8cc1fc2edd088c8)

## CloudFrontとS3でウェブサイトを公開する

当記事で行うのは、S3とCloudFrontを用いたウェブサイトの公開です。

これをコンソール上で作ろうと思った場合、ざっくり言うと

1. S3を作成してその中にオブジェクトを格納する
2. S3のウェブホスティングを有効化し、Webサイトを公開する
3. CloudFrontを作成してWebサイトをキャッシュする

という手順になります。  
(詳しい要件はワークショップ上に記載があるので、そちらをご確認ください。)

これを、API Referenceを参考にしながらCDK上でやっていきます。

## 1\. S3を作成してその中にオブジェクトを格納する

ということで、まずは [API ReferenceのS3ページ](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3-readme.html) を参照します。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/90f0b37a-8f25-c50a-f447-5485ee50c5aa.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F90f0b37a-8f25-c50a-f447-5485ee50c5aa.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=5f871951fdf7e21a92de5750b22b1a16)

見るべきポイント👀  
①左の一覧から、まずはS3を見つける  
②「Overview」を開いてみる  
③ページトップにある表を参考に、このモジュールをimportするための方法を確認する  
④S3バケットを定義してみる

まずはこの順で見て、S3を作成するための大枠を確認します。

③では、ページトップにある表を参照します。この表は各ページに存在するので、見方を覚えておきましょう。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/12732104-8f11-58ed-cb21-86d119b9150d.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F12732104-8f11-58ed-cb21-86d119b9150d.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=2f82aa2a3e323231d22378df02d22f53)

今回はTypeScriptでソースコードを書いているので、表で一番下の `aws-cdk-lib` >> `aws_s3` を見ます。

これは、「aws-cdk-lib配下にあるaws\_s3というモジュールをインポートすることで使えますよ」という意味です。  
なので、ソースコード上でまずはこれを書いてあげる必要があります。

また、④では、S3バケットの定義方法が書かれています。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/69c66bfc-7004-8ca3-7417-a0faf29ed9c1.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F69c66bfc-7004-8ca3-7417-a0faf29ed9c1.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=be8a934c73c3e487b19d608c95854b67)

こちらはこのままコードに記述しましょう。まとめると以下のようになります。

bootcamp-cdk-stack.ts

```typescript
import { Duration, Stack, StackProps } from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as s3 from 'aws-cdk-lib/aws-s3';

export class BootcampCdkStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    // S3の設定
    const bucket = new s3.Bucket(this, "MyFirstBucket");
  }
}
```

これで、ひとまずS3バケットが作成できるようになりました。

最初なので、ここで一度実際にデプロイして動作確認してみましょう。  
ターミナルで下記コマンドを叩いてください。

```bash
cdk deploy
```

すると、こんな感じで動き出し、リソースの作成が完了します！  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/a98c6612-059e-4c34-0815-318228527fcb.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2Fa98c6612-059e-4c34-0815-318228527fcb.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=fac78804724e0eb38fade28fec4f0aaa)  
↓ [![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/2b2ab702-033b-9487-b474-7f401dced650.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F2b2ab702-033b-9487-b474-7f401dced650.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=2889bdfb634068c68dc8abab605aaa03)

CloudFormationコンソールで確認すると、実際にスタックが作成されており、その中でMyFirstBucketが作成されていることも確認できました！🎉  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/228f6f84-a3d1-7d79-3196-7ae0e30739a4.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F228f6f84-a3d1-7d79-3196-7ae0e30739a4.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=68112fc7a04ba642ce0189d1408b7739)

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/b86ab272-fc89-32f3-2983-cd4f0690ae09.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2Fb86ab272-fc89-32f3-2983-cd4f0690ae09.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=fd6a41d71037fbd32d927d5a44713c0b)

確認が取れたら、このバケットはこの後不要なので、CloudFormationコンソールの「削除」からスタックを削除してしまいましょう！  
ただしこの時、設定上S3バケットは削除されずに残ってしまうので、S3コンソールから削除しておきましょう。

---

まずはCDKへの第一歩を踏み出しました！

さて次は、「ワークショップの要件に沿ったS3バケットの作成」がミッションです。

今回の要件は以下の通り。

> - S3バケットを作成してください✅
> - バケット名は yyyyMMdd-name-step1 としてください  
> 	yyyyMMdd は本日の日付、 nameは氏名に置き換えてください（バケット名には大文字は使えないのでご注意ください。）  
> 	バケット名はグローバルで一意である必要があり、エラーになる場合もあります
> - リージョンはアジアパシフィック(東京)を選択してください
> - バケット作成時はパブリックアクセスをすべてブロックしてください  
> 	後ほど変更しますが、セキュリティの観点からバケット作成時には「パブリックアクセスを許可しない」ことを意識してください

続いて、どこを見ていけば良いでしょうか。  
わからない時は自由にググっていただいて大丈夫です。ただ、その調べた内容を確認するためにも、このAPI Referenceへ返ってくることをお忘れなく！

ここでは、下記のページを見ていきます。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/97184fbb-72c4-3cb0-0f79-aa8d392b2901.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F97184fbb-72c4-3cb0-0f79-aa8d392b2901.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=d452c31f02d18ea78efccce0554a9587)

見るべきポイント👀  
①Constructs > Bucket を参照する  
②Initializerを確認する  
③Exampleとして例文があり、Initializerの書き方を確認する

②において、先ほど定義したS3バケットに追加で設定可能な様々なパラメータが書かれています。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/5a064c15-9541-68fe-74ae-d5dc6850f5ee.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F5a064c15-9541-68fe-74ae-d5dc6850f5ee.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=4d2fcc515e76769adf00410161ebb725)  
・scope: 親となるConstructを指定します。ここでは 'this' を指定します。  
・id: CloudFormationを作成する際の論理IDとなります。  
・props: S3の追加設定をここに記述することができます。  
　※「props?」の「?」は、オプションパラメータであることを示しています。  
　　あってもなくてもOKということです。

そんなpropsの具体的な設定方法として、③の例文があります。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/0974d7d4-77a4-d173-088e-d4a891b7508b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F0974d7d4-77a4-d173-088e-d4a891b7508b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=6474c0fa70b40f0d53734a9ecf202187)  
よく見ると、 `「blockPublicAccess」` というパラメータがありますね！  
これを使えば、先ほどの要件のうち1つを満たせそうです。

また、同じページの下の方を見ると、 `「Construct Props」` として、どんなパラメータを設定できるか記載されています。  
例えば、バケット名を設定できる `「bucketName」` パラメータは、名前指定のところで使えそうです。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/f6dca192-00d3-12a7-6f65-629353a05b8a.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2Ff6dca192-00d3-12a7-6f65-629353a05b8a.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=d646c1b856abdd6059f9c1c8ef0e29e9)

例文を参考にすれば、これでコードは書けそうですね！(リージョンの設定は後述)

bootcamp-cdk-stack.ts

```typescript
import { Duration, Stack, StackProps } from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as s3 from 'aws-cdk-lib/aws-s3';

// 今日の日付を取得する(yyyyMMdd形式)
const today = new Date;
const date = \`${today.getFullYear()}${today.getMonth() + 1}${today.getDate()}\`;

// 自分の名前を設定する(小文字)
const name = 'fukuchi';

export class BootcampCdkStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    // S3の設定(idは先ほどと違うものを設定)
    const bucket = new s3.Bucket(this, "Step1Bucket", {
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL,
      bucketName: \`${date}-${name}-step1\`,
    });
  }
}
```

そしてリージョンについては、CDKではこのような方法で指定します。  
作成するスタックそのもののリージョン指定、という形になります。

bin/bootcamp-cdk.ts

```typescript
#!/usr/bin/env node
import * as cdk from 'aws-cdk-lib';
import { BootcampCdkStack } from '../lib/bootcamp-cdk-stack';

const app = new cdk.App();
new BootcampCdkStack(app, 'BootcampCdkStack', {
    // スタックをデプロイするリージョンを指定
    env: {
        region: 'ap-northeast-1',
    }
});
```

こんな感じでしょうか。さて、では先ほど同様にデプロイを…

  

**の前に！！S3のコンソールを思い出してください！！  
もっと他にも設定項目ありませんでしたか！？**

  

ここではたった2つの設定しかしていません。他の設定がどうなっているのか、全く把握しない状態でデプロイするのは少しリスキーです。

早くデプロイしたい気持ちをグッと堪えて、コンソールを見に行きましょう。

例えば、ACLの設定。無効にするのが推奨となっていますが、これはCDK上だとどうなっているのでしょうか。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/349b040e-2165-630a-93e0-57e12a0a9ef2.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F349b040e-2165-630a-93e0-57e12a0a9ef2.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=4caf8ffb681705382759b8464ee8c8f2)  
↓  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/a26c7b97-ad3e-4647-00ce-f29e30952b8a.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2Fa26c7b97-ad3e-4647-00ce-f29e30952b8a.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=eb83a50a9582abe3dda4b7afd2270f9d)

API ReferenceのBucketページの下の方に各パラメータの詳細情報が載っています。

そこには *(default: BucketAccessControl.PRIVATE)* と記載されています。  
これは、 **「このパラメータの設定を省略した際に自動で設定される値」** のことです。

この *PRIVATE* がどういう意味かさらに調査するため、 [BucketAccessControl](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.BucketAccessControl.html) を確認すると、下記のように「誰もアクセス権がない」となっていたので、デフォルト設定(=パラメータ省略)で問題ないということがわかります。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/bf98f8f0-6bfc-8378-c25b-c2d86a258411.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2Fbf98f8f0-6bfc-8378-c25b-c2d86a258411.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=fe08e8a6b06e3be78c414e3a18191003)

他にも…  
バージョニングの設定は、基本的に *false* になっています。ハンズオンでは好み次第ですが、業務で使うなら *True* にするケースが多そうです。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/9fc728ab-577e-d4c7-b24b-f1d2c97739e1.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F9fc728ab-577e-d4c7-b24b-f1d2c97739e1.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=75e3dc81ab9f0925b5c588eafe704247)  
↓  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/f9ca7218-7928-e2d5-b9de-4c2c194e56ba.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2Ff9ca7218-7928-e2d5-b9de-4c2c194e56ba.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=6b8e8b9a5f25f65e240ad8f1a245fb86)

そして重要な暗号化設定に関しては…英語が長い…笑  
要約すると、「下記2つのパラメータが省略された場合はデフォルトでSSE-S3の暗号化が適用」されます。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/b4327e45-685d-ed76-0f72-862d651b7c44.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2Fb4327e45-685d-ed76-0f72-862d651b7c44.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=55d53776caed1088f1c5d16dc19d1f99)  
↓  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/d1bebf56-cba0-56af-c237-a42552ad7b21.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2Fd1bebf56-cba0-56af-c237-a42552ad7b21.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=f9dca8457486afad394e4a17ee3cf94b)

※翻訳し、まとめるとこんな感じです。(クリックすると開きます)
- デフォルトの暗号化設定：
	- encryptionKeyが指定されている場合：KMSによる暗号化が適用される
	- encryptionKeyが指定されていない場合：一見UNENCRYPTED（暗号化なし）と表示されるが、実際にはS3\_MANAGED（S3管理の暗号化）が自動的に適用される
- KMS暗号化の詳細設定：
	- encryptionをKMSに設定した場合：
		- encryptionKey（暗号化キー）を指定すると、そのキーが使用される
		- encryptionKeyを指定しないと、新しいKMSキーが自動的に作成され、バケットに関連付けられる

と、このようにコンソールとAPI Referenceを見比べてみることで、 そのAWSリソースにおけるさまざまな設定への知識が深まったり、知らない設定に気付けたり します。

全てを調べるのは時間がかかりすぎますが、自分が気になった箇所・セキュリティ的に不安な箇所を重点的にチェックしてみるだけでも学びが多くあります！

---

さて、ここまで来たら先ほどのS3バケットをデプロイしてみましょう。  
再度、ターミナルで下記コマンドを実行します。

```bash
cdk deploy
```

問題なくデプロイされれば、S3バケットの作成は完了です！  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/202890b5-264d-a91f-91eb-be1a09fd9d93.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F202890b5-264d-a91f-91eb-be1a09fd9d93.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=ba9717d11b3d50c9d4c8a1a61db61758)  
↓  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/534eb525-3895-92ef-fed9-5d0d5463f882.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F534eb525-3895-92ef-fed9-5d0d5463f882.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=29478e088718a8bacd72513cf8ee5d8d)  
~~(投稿日前日に書いていることがバレてしまう)~~

---

次いで、このバケットの中に静的コンテンツを格納していきます。

今回参照するのは [`aws_s3_deployment` > `Constructs` > `BucketDeployment`](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3_deployment.BucketDeployment.html) です。  
説明を読むと、ローカルや別のS3バケットから、zipファイルを渡せるようです。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/3d74175a-1db3-e521-e5d5-9f029f837f63.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F3d74175a-1db3-e521-e5d5-9f029f837f63.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=140671f174a4ebc75c1218d733d12feb)

書き方はこんな感じ。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/42ceecd2-a244-0b44-eee8-742f0e44b5dd.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F42ceecd2-a244-0b44-eee8-742f0e44b5dd.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a9f93c931438849c7d5e99a5d1de6e70)

これを踏まえてコードを書くと、こんな感じ。

bootcamp-cdk-stack.ts

```typescript
import { Duration, Stack, StackProps } from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as s3 from 'aws-cdk-lib/aws-s3';
import * as s3deploy from 'aws-cdk-lib/aws-s3-deployment'; //追加モジュール
import * as path from 'path' //追加モジュール

// 今日の日付を取得する(yyyyMMdd形式)
const today = new Date;
const date = \`${today.getFullYear()}${today.getMonth() + 1}${today.getDate()}\`;

// 自分の名前を設定する(小文字)
const name = 'fukuchi';

export class BootcampCdkStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    // S3の設定
    const bucket = new s3.Bucket(this, "Step1Bucket", {
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL,
      bucketName: \`${date}-${name}-step1\`,
    });

    // 以下追加。ローカルファイルを上記のS3へアップロードする。
    const deployment = new s3deploy.BucketDeployment(this, 'UploadStaticContents', {
      sources: [s3deploy.Source.asset(path.join('./contents.zip'))], // npmコマンド実行階層から見てのパスを指定
      destinationBucket: bucket, // 上記のS3を指定
    });
  }
}
```

フォルダ構造はこんな感じ。contents.zipは [ワークショップのページ](https://catalog.us-east-1.prod.workshops.aws/workshops/a9b0eefd-f429-4859-9881-ce3a7f1a4e5f/ja-JP/step1-webhosting/challenge1) からダウンロードできます。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/4d98bafa-c869-6cbf-ce68-a1502743a002.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F4d98bafa-c869-6cbf-ce68-a1502743a002.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=105d4b8b4cc38480c944e93769556932)

デプロイ時、下記のような表示が出たら、「y」を押せばデプロイ開始となります。  
(内部処理として、Lambdaが作られてそれが動作したらコンテンツをS3へ移すみたいです)  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/39899b60-fcc8-149a-bd26-0b322580f6d0.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F39899b60-fcc8-149a-bd26-0b322580f6d0.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=0fb917fb8f5c5da50b089385e570f6aa)

そしてS3バケットを確認すると、無事コンテンツが格納されていました！🙌  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/a6a882b1-5b06-73fe-29e1-2df77dd610c8.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2Fa6a882b1-5b06-73fe-29e1-2df77dd610c8.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=59d1442604b67b572e94637982833f2d)

長かったですが、ようやく手順1が完了です！

## 2\. S3のウェブホスティングを有効化し、Webサイトを公開する

続いて、S3からデータを公開していきます。  
ここからは手順省略気味でサクサクいきます。ぜひ一緒にやりましょう！

ここでの要件は以下の通りです。

> - S3の機能を使ってWebサイトを公開してください
> - パブリックアクセスでは、ファイルの読み取りのみを許可するようポリシーを設定してください

まずは1つ目の要件を満たしにいきましょう！API Referenceで見るのはこの辺でしょうか。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/2047df57-6e4c-3d98-7991-6c6f893298a4.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F2047df57-6e4c-3d98-7991-6c6f893298a4.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=3874309c103923133362edd1f08d0ea3)

余談(興味ある方だけ読んでください)

> 正直、自分で手を動かしてWebサイトホスティングやった記憶がないので、調べていい学びの機会になりました。リダイレクトできるのとか知らなかったですね…。
> 
> でも実際問題、知識として「S3で静的サイト公開できる」っていうのを知っていても、自分で手を動かして作ったことない人も多いのではないでしょうか。だからこそ、こういうワークショップで環境を触ったり調べたりすることには大きな価値があると思っています！

続いて2つ目。  
まずは先ほど設定した `blockPublicPolicy` の設定を見直します。  
その後、バケットポリシーを書き換えていきます。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/17de6895-8e16-ee00-c173-592cbd113e68.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F17de6895-8e16-ee00-c173-592cbd113e68.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=34d17744d222ab377dc153459d2ad841)

[ポリシーを定義するためのクラスはIAMのところにある](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.PolicyStatement.html) のがトラップ！

そしてそのポリシーをS3バケットに付与します。BucketコンストラクトのMethodとして用意されている `addToResourcePolicy` が使えそうですね。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/329c2be1-2ac3-247c-3259-913a6b1bc2b9.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F329c2be1-2ac3-247c-3259-913a6b1bc2b9.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=f4f9d761e1231a7f8b03d88907af8cfe)

(上記を推奨していますって `Constricts` > `BucketPolicy` に書いてあった)  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/790ca605-148b-84c5-b558-21498983b0dc.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F790ca605-148b-84c5-b558-21498983b0dc.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a39ee41cf68b76499227e2ebf8a7759d)

最終的に出来上がったコードがこんな感じです。コメントのある箇所が追加した内容です。

bootcamp-cdk-stack.ts

```typescript
import { Duration, Stack, StackProps, RemovalPolicy } from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as s3 from 'aws-cdk-lib/aws-s3';
import * as s3deploy from 'aws-cdk-lib/aws-s3-deployment';
import * as path from 'path'
import { PolicyStatement, Effect, ArnPrincipal } from 'aws-cdk-lib/aws-iam';

const today = new Date;
const date = \`${today.getFullYear()}${today.getMonth() + 1}${today.getDate()}\`;
const name = 'fukuchi';

export class BootcampCdkStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    const bucket = new s3.Bucket(this, 'Step1Bucket', {
      // パブリックアクセスを許可する
      publicReadAccess: true,
      // 静的ウェブホスティングを有効化する
      websiteIndexDocument: 'index.html',
      // ACLを通じたアクセスをブロックしつつ、バケットポリシーを通じたアクセスを許可する
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ACLS,
      bucketName: \`${date}-${name}-step1\`,
    });

    const deployment = new s3deploy.BucketDeployment(this, 'UploadStaticContents', {
      sources: [s3deploy.Source.asset(path.join('./contents.zip'))], 
      destinationBucket: bucket, 
    });

    // 読み取り専用のバケットポリシーを作成する
    const allowPublicAccessBucketPolicy = new PolicyStatement({
      sid: 'Allow Public Access',
      effect: Effect.ALLOW,
      actions: ['s3:GetObject'],
      principals: [new ArnPrincipal('*')],  //難しめ
      resources: [bucket.bucketArn + '/*'], //ちょっと難しめ
    });

    // バケットポリシーをS3バケットに適用
    bucket.addToResourcePolicy(allowPublicAccessBucketPolicy);
  }
}
```

そして再び `cdk deploy` した結果、設定がきちんと反映され、S3のコンテンツへアクセス可能になっていました！  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/9cc2e4a9-9a89-cdce-1501-aecf1d80678f.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F9cc2e4a9-9a89-cdce-1501-aecf1d80678f.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=ecb272af31030fdfb371c19dbf435d0a)  
↓  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/5a0ef40e-c0dd-9354-04f0-2fd0610c8f99.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F5a0ef40e-c0dd-9354-04f0-2fd0610c8f99.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=f67b1e1cd0aff9aaedae232f67d4f143)  
↓  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/773a12d4-f278-2680-e36d-1841775aee91.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F773a12d4-f278-2680-e36d-1841775aee91.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=b6d8d61f4a718b826eb7e18bd52778fc)

ただ一点、全く同じポリシー内容が重複していました。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/67536967-dea0-5fa7-180e-9d634471332a.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F67536967-dea0-5fa7-180e-9d634471332a.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=af415d255077925153264184bf4a01d1)

これは恐らく `publicReadAccess: true` の設定と、 `allowPublicAccessBucketPolicy` の設定が重複してしまっているために発生しているのだと考えました。

`publicReadAccess` の設定をコメントアウトして、 `cdk diff` コマンドを使用します。  
これによって、先ほどAWS環境へデプロイするのに使用したCloudFormationテンプレートと、現在のVSCode上で生成されるCloudFormationテンプレートを比較することができます。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/efcfd8a9-a743-f929-f525-df95009bb2cc.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2Fefcfd8a9-a743-f929-f525-df95009bb2cc.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=f77d24e903dca4949471c851ef381653)

すると、上記のようにバケットポリシー1つ分が差分として出てきました。なのでこの設定は要らなそうですね。

コメントアウトしたまま、再度 `cdk deploy` を実行すると、1つだけポリシーが残り、パブリックアクセスは変わらず可能でした。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/ba83ad80-bfee-c5e7-5c82-69f17efcabee.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2Fba83ad80-bfee-c5e7-5c82-69f17efcabee.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=91f9bf7b190370b9e2032991d43ad0da)  
↓  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/27d71fcd-9790-b688-ca8a-7c3ec7eb1dfa.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F27d71fcd-9790-b688-ca8a-7c3ec7eb1dfa.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=ca4ee92c6d7f67c106c53745e3ed29fb)

`publicReadAccess` の説明文をよく読むと `「バケット内のすべてのオブジェクトに対して、パブリックな読み取りアクセスを許可します。」` と書かれていたので、読んでなかった僕が悪いです。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/4ec038ff-691a-fda2-48b9-5843312d29af.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F4ec038ff-691a-fda2-48b9-5843312d29af.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=846b172710f9bac1ea8eb5b836194a61)

あとよくよく見たら、 `cdk deploy` 時の確認で同じポリシー2つ作成されてたというオチです。ここで気付けなかった…  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/240d1cdc-db22-4afe-4776-7947615e8fe3.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F240d1cdc-db22-4afe-4776-7947615e8fe3.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=7f599bca4fa45c0318615e24514cb932)

でもこうやって間違えることによる気づき・収穫もあるので、自分の頭で考え、調べることの重要性が身に染みて分かりますね！(ポジティブシンキング)

これにて、手順2も完了です！

## 3.CloudFrontを作成してWebサイトをキャッシュする

最後にCloudFrontの出番です。

要件はこちら。

> - Amazon CloudFrontを使ってWebサイトをキャッシュしてください
> - Amazon CloudFront経由のURLとAmazon S3のウェブサイトホスティングのURLは異なります
> - SSL(HTTPS)の設定は今回はスキップしてください
> - S3のコンテンツにCloudFront経由でアクセスできるようにしてください
> - Origin Access Control (OAC) の設定は不要です  
> 	今回のワークショップではアクセスを制限していませんが、 よりセキュリティを高めるにはS3にはCloudFrontからのアクセスのみ許可する方が望ましいです。余力がある方はその方法についても調べてみましょう。

となっていますが、より実践で活きる力を身につけるためにも、下記のように要件を変更します！

> - Amazon CloudFrontを使ってWebサイトをキャッシュしてください
> - **ウェブサイトホスティングは使いません**
> - SSL(HTTPS)の設定 **も行います**
> - S3のコンテンツにCloudFront経由で **のみ** アクセスできるようにしてください
> - Origin Access Control (OAC) の設定は **必要です**  
> 	よりセキュリティを高めるにはS3にはCloudFrontからのアクセスのみ許可する方が望ましいためです

さて、ちょっと難易度上がりますがやっていきましょう。

まず先ほど設定した静的ウェブホスティングですが、 **もうお役御免です。** 奴はこの先の戦いにはついて来(ら)れないので…  
というのも、静的ウェブホスティングには致命的な欠点が2つあります。

1. HTTPしか対応していない
2. S3バケットをパブリックに公開しなければいけない

前者については、ドキュメントの中に書かれていました。  
また、コンソール上でウェブサイトエンドポイントを使用したCloudFrontを作成しようとすると、プロトコルが「HTTPのみ」で固定となっていました。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/86a4e07f-24b5-a014-9466-f510883aa8dd.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F86a4e07f-24b5-a014-9466-f510883aa8dd.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=07c9052bb2ec93a2459d558228caaabe)  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/cba9ec2c-6faf-58a9-bc67-4b9242494086.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2Fcba9ec2c-6faf-58a9-bc67-4b9242494086.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=1effa3fb4edaaa296af2d2519d2cd610)

後者についてもドキュメントで記載があります。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/413e171d-6b87-f111-fbea-9306dcf7d0b5.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F413e171d-6b87-f111-fbea-9306dcf7d0b5.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=d2a539288a5d85565c4670d63b4af57d)

そして極め付けは、 [「S3のセキュリティベストプラクティス」](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/userguide/security-best-practices.html#security-best-practices-prevent) として、こんなことが書かれています。

> - **アクセスコントロールリスト (ACL) の無効化**
> - **Amazon S3 バケットに正しいポリシーが使用され、バケットが公開されていないことを確認する**

もうお分かりですね。早急に修正しましょう！

修正後のコードはこちら

bootcamp-cdk-stack.ts

```typescript
import { Duration, Stack, StackProps, RemovalPolicy } from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as s3 from 'aws-cdk-lib/aws-s3';
import * as s3deploy from 'aws-cdk-lib/aws-s3-deployment';
import * as path from 'path'
import { PolicyStatement, Effect, ArnPrincipal } from 'aws-cdk-lib/aws-iam';

const today = new Date;
const date = \`${today.getFullYear()}${today.getMonth() + 1}${today.getDate()}\`;
const name = 'fukuchi';

export class BootcampCdkStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    const bucket = new s3.Bucket(this, 'Step1Bucket', {
      // 静的ウェブホスティングを有効化しない
      // websiteIndexDocument: 'index.html',
      // パブリックアクセスは遮断
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL,
      bucketName: \`${date}-${name}-step1\`,
    });

    const deployment = new s3deploy.BucketDeployment(this, 'UploadStaticContents', {
      sources: [s3deploy.Source.asset(path.join('./contents.zip'))], 
      destinationBucket: bucket, 
    });

    /* 以下削除
    const allowPublicAccessBucketPolicy = new PolicyStatement({
      sid: 'Allow Public Access',
      effect: Effect.ALLOW,
      actions: ['s3:GetObject'],
      principals: [new ArnPrincipal('*')],  //難しめ
      resources: [bucket.bucketArn + '/*'], //ちょっと難しめ
    });
    bucket.addToResourcePolicy(allowPublicAccessBucketPolicy);
    */
  }
}
```

綺麗さっぱり  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/491ba945-63c0-ca22-fe39-011ff0d21640.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F491ba945-63c0-ca22-fe39-011ff0d21640.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=d83afba1057002dce1bc7b96de87272f)

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/61bc6e3b-10ed-8b08-9671-65d8a6d7373e.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F61bc6e3b-10ed-8b08-9671-65d8a6d7373e.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=ce14a76ddac2d1c7113d6449cc2961d7)

ということで、HTTPSのみ対応して進めていきます。そのためにはまずOACが必要ということでOACの項目を探します。  
(L1 Constructしか対応してないんですね)  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/84ceda20-877f-df4b-d9c8-f7c770a33bd7.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F84ceda20-877f-df4b-d9c8-f7c770a33bd7.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a2557d3536129f9babd87b2b101644c2)  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/2f485bd8-5431-149a-4830-eb020c838e9d.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F2f485bd8-5431-149a-4830-eb020c838e9d.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a20fd7e5136a4436ed5f469ccd963942)

コンソールの情報も参考にしていきましょう！どの項目がどのパラメータと結びつくのかを考える・調べる！  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/f708491b-8033-8cee-b2e2-e14715a5d185.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2Ff708491b-8033-8cee-b2e2-e14715a5d185.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=567e3c42451dc72f165cd2a7fca03fb3)

署名動作(signingBehavior)について補足説明

この署名動作ってなんだ…？と思い、調査したので簡単にまとめておきます。

上記画像の通り、署名動作には3つあります。

- 署名リクエスト(cdk: `always`)
	- 文字通り、すべてのオリジンリクエストに署名する
	- もしビューワーリクエストに `Authorization` ヘッダーが存在する場合は、それを上書き署名します
	- ユースケース: 公開ウェブサイトでありながら、S3バケットへの直接アクセスを防ぎたい場合
- 署名リクエスト・認証ヘッダーを上書きしない(cdk: `no-override`)
	- オリジンリクエストに署名する
	- ただし、ビューワーリクエストに `Authorization` ヘッダーが含まれている場合は署名せず、そのヘッダーをそのままオリジンリクエストとして送る
	- ユースケース: ユーザー認証を必要とするプライベートコンテンツと、公開コンテンツが混在するアプリ
- リクエストに署名しない(cdk: `never`)
	- オリジンリクエストに一切の署名を行わない
	- もはやOACを設定している意味がないレベル(多分)
	- ユースケース: 完全に公開されたコンテンツで、S3バケットへのアクセスも許可する場合

OACの設定ができたら、次はCloudFrontディストリビューションを作成します。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/7239cd11-7b92-b15e-d253-11adde584743.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F7239cd11-7b92-b15e-d253-11adde584743.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=05130734ea5d91070d9a994e9445beec)

その後、OACとCloudFrontディストリビューションを紐づけるのですが、ここでまた一苦労。ちょっとややこしめの設定が必要になります。  
具体的には、 `CfnDistribution` の `addPropertyOverride` メソッドを使う必要があるようです。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/b6207f94-70ea-3025-335a-4fc435eeefd8.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2Fb6207f94-70ea-3025-335a-4fc435eeefd8.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=dc93072a6f694c4de6c5d3e7e977c7df)

あと、完全に余談ですが…自動でOAI用バケットポリシーが付与されてしまうみたいなので、不要な場合は消しちゃいましょう。  
これは `CfnBucketPolicy` > `addPropertyOverride` メソッドを使います。

そして最終的に出来上がったコードがこちら。

bootcamp-cdk-stack.ts

```typescript
import { 
  aws_cloudfront as cloudfront, 
  aws_cloudfront_origins as origins, 
  aws_iam as iam,
  Stack, 
  StackProps, 
} from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as s3 from 'aws-cdk-lib/aws-s3';
import * as s3deploy from 'aws-cdk-lib/aws-s3-deployment';
import * as path from 'path'

const today = new Date;
const date = \`${today.getFullYear()}${today.getMonth() + 1}${today.getDate()}\`;
const name = 'fukuchi';

export class BootcampCdkStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    const bucket = new s3.Bucket(this, 'Step1Bucket', {
      // パブリックアクセスは遮断
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL,
      bucketName: \`${date}-${name}-step1\`,
    });

    const deployment = new s3deploy.BucketDeployment(this, 'UploadStaticContents', {
      sources: [s3deploy.Source.asset(path.join('./contents.zip'))], 
      destinationBucket: bucket, 
    });

    // OACの設定
    const originAccessControl = new cloudfront.CfnOriginAccessControl(this, 'MyCfnOriginAccessControl', {
      originAccessControlConfig: {
        name: 'MyOAC',
        originAccessControlOriginType: 's3',
        signingBehavior: 'always',
        signingProtocol: 'sigv4',
        description: \`OAC for ${bucket.bucketName}\`
      }
    });

    // CloudFrontディストリビューションの設定
    const distribution = new cloudfront.Distribution(this, 'MyCloudFrontDistribution', {
      defaultBehavior: {
        origin: new origins.S3Origin(bucket),
        viewerProtocolPolicy: cloudfront.ViewerProtocolPolicy.REDIRECT_TO_HTTPS
      },
      defaultRootObject: 'index.html',
    })

    // 自動生成されるOAI用バケットポリシーを削除
    const cfnBucket = bucket.policy?.node.defaultChild as s3.CfnBucketPolicy;
    cfnBucket.addPropertyOverride('PolicyDocument.Statement.0', undefined);

    // OACとCloudFrontディストリビューションの紐付け
    const cfnDistribution = distribution.node.defaultChild as cloudfront.CfnDistribution;
    // 自動生成されるOAI削除
    cfnDistribution.addPropertyOverride('DistributionConfig.Origins.0.S3OriginConfig.OriginAccessIdentity', '');
    // 上記で作成したOACを設定
    cfnDistribution.addPropertyOverride('DistributionConfig.Origins.0.OriginAccessControlId', originAccessControl.attrId);

    // OACからのアクセスのみ許可するS3バケットポリシーの作成
    const contentsBucketPolicy = new iam.PolicyStatement({
      actions: ['s3:GetObject'],
      effect: iam.Effect.ALLOW,
      principals: [
        new iam.ServicePrincipal('cloudfront.amazonaws.com'),
      ],
      resources: [\`${bucket.bucketArn}/*\`],
    });
    contentsBucketPolicy.addCondition('StringEquals', {
      'AWS:SourceArn': \`arn:aws:cloudfront::${props?.env?.account}:distribution/${distribution.distributionId}\`
    })
    bucket.addToResourcePolicy(contentsBucketPolicy);
  }
}
```

これをデプロイすると、きちんと設定が反映されており、CloudFrontからウェブサイトへのアクセスもできました！  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/c5a2f75f-cbea-e867-178f-b949da6b45ca.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2Fc5a2f75f-cbea-e867-178f-b949da6b45ca.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=e887618ecfc4a0b5bd8354ddeb43ed60)  
↓  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/2af04453-bcb5-b1a3-e729-84bc350991b7.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F2af04453-bcb5-b1a3-e729-84bc350991b7.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a7f6dbdcff1f0685b53a1a2826cb81c7)  
↓  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/fef54c84-bb5b-4dea-10a0-e7d8a5adf2d8.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2Ffef54c84-bb5b-4dea-10a0-e7d8a5adf2d8.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=5822a22e331a535f6116acbb5651f920)  
↓  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3616210/1247208a-49de-cf22-d28c-30c40f7667d5.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3616210%2F1247208a-49de-cf22-d28c-30c40f7667d5.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=0e1f26bb6ee28f99f28d19d88af9dc2b)

以上で手順3.も完了です！

## 終わりに

長くなってしまいましたが、こんな要領で進めてもらえればと思います！  
(ワークショップ自体はまだ続きがあるので、また時間を見つけて進めます！)

再掲ですが、こちらだけでも覚えて帰っていただけると幸いです！

やり方：

1. コンソール上でワークショップをやってみる
2. ワークショップの内容をCDKで実装する  
	2.1. API Referenceで実装方法を調べる  
	2.2. 実際にコードを書く  
	2.3. コンソールとAPI Referenceを見比べて設定を確認する  
	2.4. CDKでデプロイし、AWSリソースを作成する

最初は難しいと思うので、調べたりAIを使ったりして効率的に進めていきましょう！

そして実際にコンソール+コードの二重学習を進めることで、知識が深まり、技術力向上に繋がると思いますので、ぜひ挑戦してみてください！！

私自身、まだまだ至らない点だらけだなというのをANGEL Dojoで痛感しています。

ただ、同年代で非常に技術力のある方々や、先輩エンジニアの方々と切磋琢磨しながら技術力を向上できているこの環境が非常に楽しく、刺激をたくさんいただいています！

このモチベーションを維持しつつANGEL Dojoを最後まで駆け抜け、最終的には2024年度のJr.Championsに選出されるよう頑張ります！

読んでいただきありがとうございました！  
9/1から始まるANGEL Calendar、お楽しみに〜！！！

## 参考資料

- [API Reference](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-construct-library.html)
- [実践力を鍛えるBootcamp - クラウドネイティブ編 -](https://catalog.us-east-1.prod.workshops.aws/workshops/a9b0eefd-f429-4859-9881-ce3a7f1a4e5f/ja-JP/step1-webhosting/challenge3)
- [ビューワーと CloudFront の間の通信に HTTPS を要求する](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/using-https-viewers-to-cloudfront.html)
- [チュートリアル: Amazon S3 での静的ウェブサイトの設定](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/userguide/HostingWebsiteOnS3Setup.html)
- [Amazon S3 のセキュリティのベストプラクティス](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/userguide/security-best-practices.html#security-best-practices-prevent)
- [【AWSCDK】CloudFront+S3でOACを使いたい](https://zenn.dev/thyt_lab/articles/d6423c883882b7)
- [AWS CDK v2でOACってどうやってやるの？CloudFront+S3+OAC構築（コード付き-Typescript）](https://qiita.com/ta__k0/items/bd700a074c394aa4d6f4)
- [【AWS】S3でバケットを公開せずHTTPSのみで静的サイトを公開する方法](https://qiita.com/polarbear08/items/84b7add0ddd309abda74)

[0](https://qiita.com/har1101/items/#comments)

[71](https://qiita.com/har1101/items/02b5129cdf6101d6afc0/likers)

58