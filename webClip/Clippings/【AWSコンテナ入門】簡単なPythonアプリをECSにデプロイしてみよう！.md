---
title: 【AWSコンテナ入門】簡単なPythonアプリをECSにデプロイしてみよう！
source: https://qiita.com/minorun365/items/84bef6f06e450a310a6a
author:
  - "[[Qiita]]"
published: 2024-08-22
created: 2025-05-06
description: 8/17 20:16更新： 記事全体を大幅に更新しました！この記事は何？最近、生成AIブームで「Pythonの簡単なチャットアプリを作ってみる」機会が増えたのではないでしょうか。特に、Str…
tags:
  - Docker
  - Python
  - 1
  - クラウド
  - インフラ
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

8/17 20:16更新： 記事全体を大幅に更新しました！

## この記事は何？

最近、生成AIブームで「Pythonの簡単なチャットアプリを作ってみる」機会が増えたのではないでしょうか。

特に、Streamlitという便利なライブラリを使えば、Reactなどが書けなくても簡単にフロントエンドをPythonで作ることができます。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/1591935b-3b57-33ce-4128-43cea0c0d477.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F1591935b-3b57-33ce-4128-43cea0c0d477.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=6f24b4b86f097d12f894543fdd7beca1)

開発端末のローカルやCloud9などでこれを動かすのは簡単なのですが、いざ他の人にも使ってもらおうとするとクラウド上にデプロイする必要があります。

しかし、 **アプリをコンテナに固めてAWSにデプロイ！** といった王道の作業をGUIで分かりやすく解説する記事が意外と少なかったので、初心者向けハンズオンとしてまとめてみます。

## ハンズオンの概要

### 作成するアーキテクチャ

[![スクリーン ショット 2024-08-16 に 20.11.28 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/6febdb6c-bf8e-33f7-20eb-2a91865a2fe9.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F6febdb6c-bf8e-33f7-20eb-2a91865a2fe9.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=ef862da561b419b41d5753c2823ec548)

### 作業環境

- 端末：Macbook（Appleシリコン）
- ブラウザ：Google Chrome
- コードエディター：VS Code
- Python：3.9以降のバージョン

### 注意事項

- AWS利用料が発生します（超ざっくり目安：1日3ドル程度）。
- 設定を誤ると、Webアプリへ第三者からアクセスされてしまうなど、意図せぬセキュリティ事故に繋がります。

上記を認識のうえ、自らの責任で管理しましょう。検証終了後は不要なリソースを停止・削除し、AWSアカウントも閉鎖してしまうことをおすすめします。

## 手順

## 1\. AWS環境の構築（前編）

最初に、アプリケーションのローカル実行、およびコンテナのプッシュが実施できるようになるための、最低限のAWSリソースを構築していきます。

### 1-1. AWSアカウント・IAMユーザーの作成

まずはAWSアカウントを作成します。

ルートユーザー（登録したEメールアドレス）でサインインしたら、作業用のIAMユーザーを作成し、そのユーザーでサインインし直します。

IAMユーザーには以下のポリシーを「直接アタッチ」しておきましょう。

- IAMFullAccess
- AmazonVPCFullAccess
- AmazonEC2FullAccess
- AmazonECS\_FullAccess
- AmazonEC2ContainerRegistryFullAccess
- AmazonDynamoDBFullAccess
- AmazonBedrockFullAccess
- AWSMarketplaceFullAccess

[![スクリーンショット 2024-08-17 14.24.31.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/55b00463-360e-a665-54f2-9170d22198ae.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F55b00463-360e-a665-54f2-9170d22198ae.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=514a3a84f97585d59fe1d30b0c53ea03)

セキュリティ事故防止のため、ルートユーザー・IAMユーザーともに、MFA（多要素認証）を設定しておくことを強くおすすめします。

本ハンズオンは東京リージョンでの実施を想定しています。上記のIAMユーザーで再サインインしたら、AWSマネジメントコンソールの画面右上からリージョンを切り替えておきましょう。

※バージニア北部をはじめ、他の主要リージョンでも実施可能です。適宜読み替えてご利用ください。

### 1-2. Bedrockのモデル有効化

今回のアプリのキモとなる、チャット返答を生成するAIモデルを有効化しましょう。

Amazon Bedrockのコンソールにアクセスし、左サイドバー下部の「モデルアクセス」より、以下の生成AIモデルを有効化しておきます。

- Anthropic > Claude 3.5 Sonnet

[![スクリーン ショット 2024-08-16 に 21.42.32 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/0501904f-7907-295a-4f94-ccd758fb6e70.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F0501904f-7907-295a-4f94-ccd758fb6e70.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=6c438a85c340ec2cdaa62dbb9f57d1cd)

※まとめて全モデル有効化してしまっても大丈夫です。モデルの数だけメールが来ますが驚かないでください。

最新モデルであるClaude 3.5 Sonnetは、ステータスが「使用不可」となりモデル有効化ができないことがあります。その場合、Claude 3 Haikuなど他のモデルで代替しましょう。

[![スクリーンショット 2024-08-17 14.30.05.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/d619fe71-eece-8dc1-d120-6ae4addb86f0.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2Fd619fe71-eece-8dc1-d120-6ae4addb86f0.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a937d33615f447a2349236c31ffcb15c)

※どうしてもClaude 3.5を使いたい場合、AWSサポートへ問い合わせてみましょう。

Anthropic社のモデルは、利用にあたり用途の申告を求められます。社内検証であることが分かるように、ざっくり記載すればOKです。

### 1-3. DynamoDBのテーブル作成

AIとのチャット履歴を保存するためのデータベースを作成します。

Amazon DynamoDBのコンソールにアクセスし、 `テーブル > テーブルの作成` から、以下の設定で新規テーブルを作成しましょう。

※今回は「Bedrock Simple Chat」というアプリを作成するため、各AWSリソースの名前には「bsc」という略称を利用します。

- テーブルの詳細
	- テーブル名： `bsc_db`
	- パーティションキー： `SessionId` （データ型：文字列）
- テーブル設定： 設定をカスタマイズ
- 読み込み/書き込みキャパシティーの設定
	- キャパシティーモード： オンデマンド

[![スクリーン ショット 2024-08-17 に 18.43.33 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/2225aa38-b247-9a51-a2fe-759b6d3cffbd.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F2225aa38-b247-9a51-a2fe-759b6d3cffbd.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=20f8c29076045ee47fe5a91cb82a37b9)

他はデフォルトでOKです。

なぜかテーブル名にハイフンを使用すると、StreamlitからDBスキーマが正しく認識されなくなりました。そのためアンダースコアを利用しています。

### 1-4. コンテナ用のレジストリ作成

ECRのコンソールへ移動し、 `bsc` という名前のプライベートリポジトリを作成しておきます。ここにコンテナを登録できます。

[![スクリーン ショット 2024-08-17 に 16.57.31 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/41defc52-0906-8551-77cd-939108c86421.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F41defc52-0906-8551-77cd-939108c86421.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a0cf80a2ff4ebd11bb89201a272e14e4)

ここまで実施すれば、今回デプロイするアプリケーションを開発PCのローカル上で起動したり、コンテナ化してAWS上でプッシュすることができます。

## 2\. アプリケーション開発

### 2-1. Pythonコードの作成

作業PCのローカルでVS Codeを起動します。

ターミナルを開き、今回用の作業ディレクトリを作成します。

```bash
# ホームディレクトリへ移動
cd ~

# 作業ディレクトリを作成
mkdir bedrock-simple-chat
```

作成したディレクトリをVS Codeで開き、配下に以下のPythonファイルを作成します。

main.py

```py
# Pyhton外部モジュールのインポート
import streamlit as st
from langchain_aws import ChatBedrock
from langchain_community.chat_message_histories import DynamoDBChatMessageHistory
from langchain_core.messages import HumanMessage
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

# セッションIDを定義
if "session_id" not in st.session_state:
    st.session_state.session_id = "session_id"

# セッションに会話履歴を定義
if "history" not in st.session_state:
    st.session_state.history = DynamoDBChatMessageHistory(
        table_name="bsc_db", session_id=st.session_state.session_id
    )

# セッションにLangChainの処理チェーンを定義
if "chain" not in st.session_state:
    # プロンプトを定義
    prompt = ChatPromptTemplate.from_messages(
        [
            ("system", "絵文字入りでフレンドリーに会話してください"),
            MessagesPlaceholder(variable_name="messages"),
            MessagesPlaceholder(variable_name="human_message"),
        ]
    )

    # チャット用LLMを定義
    chat = ChatBedrock(
        model_id="anthropic.claude-3-5-sonnet-20240620-v1:0",
        model_kwargs={"max_tokens": 4000}, streaming=True,
    )

    # チェーンを定義
    st.session_state.chain = prompt | chat

# タイトルを画面表示
st.title("Bedrockと話そう！")

# 履歴クリアボタンを画面表示
if st.button("履歴クリア"):
    st.session_state.history.clear()

# メッセージを画面表示
for message in st.session_state.history.messages:
    with st.chat_message(message.type):
        st.markdown(message.content)

# チャット入力欄を定義
if prompt := st.chat_input("何でも話してね！"):
    # ユーザーの入力をメッセージに追加
    with st.chat_message("user"):
        st.markdown(prompt)

    # モデルの呼び出しと結果の画面表示
    with st.chat_message("assistant"):
        response = st.write_stream(
            st.session_state.chain.stream(
                {
                    "messages": st.session_state.history.messages,
                    "human_message": [HumanMessage(content=prompt)],
                },
                config={"configurable": {"session_id": st.session_state.session_id}},
            )
        )

    # 会話を履歴に追加
    st.session_state.history.add_user_message(prompt)
    st.session_state.history.add_ai_message(response)
```

BedrockのモデルをClaude 3.5 Sonnet以外に変更した方は、上記コード31行目のモデルIDを変更先のものへ修正しておきましょう。  
[https://docs.aws.amazon.com/bedrock/latest/userguide/model-ids.html](https://docs.aws.amazon.com/bedrock/latest/userguide/model-ids.html)

これをPythonで動かすために必要なライブラリを、以下のファイルに記載しておきます。これも同じく、 `bedrock-simple-chat` ディレクトリ直下に作成します。

requirements.txt

```py
boto3==1.34.87
streamlit==1.33.0
langchain==0.2.0
langchain-aws==0.1.4
langchain-community==0.2.0
```

### 2-2. ローカルにAWS認証情報を設定

このアプリを開発PC上で実行したり、コンテナ化してECRに登録するためには、AWS認証情報のセットアップが必要です。

最初にAWS CLIをインストールしましょう。

上記のドキュメントに記載のとおり、Macへのインストールコマンドは次のとおりです。

ターミナル

```bash
# パッケージのダウンロード
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"

# AWS CLIのインストール
sudo installer -pkg AWSCLIV2.pkg -target /

# インストールされたことを確認
aws --version
```

無事にインストールできたら、これを使って自分のIAMユーザーの認証情報を開発PCのローカルに設定しましょう。

IAMのコンソールから先ほど作成したユーザーを開き、 `セキュリティ認証情報 > アクセスキー` からCLI用のIAMアクセスキーを作成します。

[![スクリーン ショット 2024-08-17 に 17.38.02 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/769f279b-5f22-929f-5b3a-11ad58292060.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F769f279b-5f22-929f-5b3a-11ad58292060.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=1a69018be799aa6702d8e316b258f61a)

表示されるアクセスキーとシークレットアクセスキーは、1回限りしか表示されないため、画面を閉じずにローカルの設定作業を進めます。

このシークレットキーやシークレットアクセスキーは、いわば「ユーザー名とパスワード」のように、知っているだけでIAMサインインができてしまう権限を有する情報なので、第三者に悪用されないよう慎重に管理しましょう。

VS Codeのターミナルで以下コマンドを実行し、ウィザードに従って必要な情報を入力します。

ターミナル

```bash
# AWS認証情報の初期設定コマンド
aws configure

# 実行すると以下の情報を求められる
AWS Access Key ID [None]: 前述のアクセスキーを入力
AWS Secret Access Key [None]: 前述のシークレットアクセスキーを入力
Default region name [None]: ap-northeast-1
Default output format [None]: json
```

すでに業務や個人開発などで別のAWSプロファイルを設定している方は、以下を参考に新規プロファイルを作成し、ターミナルを開くたびに環境変数を設定して切り替えてください。

設定後、以下のコマンドを実行して、起動中のターミナルが正しく今回のAWSアカウントを参照できているか確認しましょう。 `bsc` という名前のリポジトリが出力されればOKです。

ターミナル

```bash
# ECRのリポジトリを一覧表示
aws ecr describe-repositories
```

### 2-3. Streamlitアプリの動作確認

まずは、このアプリをPC上で動かしてみましょう。VS Codeのターミナルで以下を実行します。

ターミナル

```bash
# 作業ディレクトリのルートに移動
cd ~/bedrock-simple-chat

# Python外部ライブラリのインストール
pip install -r requirements.txt

# Streamlitアプリの実行
streamlit run main.py
```

上手くいくとブラウザで `http://localhost:8501` が開き、アプリをローカルで操作できます。

[![スクリーン ショット 2024-08-17 に 17.27.37 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/8963618d-1174-9bfc-613a-0dceeedf1787.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F8963618d-1174-9bfc-613a-0dceeedf1787.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=3da690a6c0a7abd8998a8d67301ad08d)

実際にチャットを操作してみて、BedrockのモデルやDynamoDBのテーブルとの通信が正しく実施できるか確認しましょう。

確認完了後は、VS Codeのターミナルで `Ctrl + c` を押すとStreamlitを停止できます。

### 2-4. アプリのコンテナ化

このPythonアプリをコンテナ化し、ECRへ登録します。

まず、コンテナ起動時のベースイメージ＋初期設定コマンドを記載したDockerfileというファイルを、拡張子なしで作業ディレクトリのルートに保存します。

Dockerfile

```dockerfile
FROM python:3.9

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . /app

CMD ["streamlit", "run", "main.py"]
```

Python 3.9のコンテナをベースに、 `pip install` で外部ライブラリをインストールし、Streamlitを起動するコマンドを記載しています。

ここまでで次のようなディレクトリ構成になっています。

[![スクリーン ショット 2024-08-17 に 11.57.57 午前.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/5ba351ba-3d60-656a-1edf-d42ee83785b5.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F5ba351ba-3d60-656a-1edf-d42ee83785b5.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=b3eebfddd8459b01876b76e2ecc60f4c)

その後、いよいよアプリをコンテナ化していきます。Docker Desktopを開発PCへインストールして、起動しておきましょう。（一定規模以上での商用利用は有償化されましたが、個人利用であれば無料です）

ローカルのアプリをコンテナ化＆プッシュするためのコマンドは、ECRで作成したプライベートレジストリ（ `bsc` ）の画面より「プッシュコマンドの表示」ボタンから確認することができます。

[![スクリーン ショット 2024-08-16 に 22.58.08 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/282d244f-7623-4358-ff4e-0f856985b2b4.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F282d244f-7623-4358-ff4e-0f856985b2b4.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=9d7c092a397857e33e05b3a12e24b7c7)

ただし、ビルドコマンドには `--platform linux/amd64` オプションを付与する必要があります。ビルドを実行するMacと、実際にコンテナを稼働させるFargateでは、CPUのアーキテクチャが異なるためです。

以下をVS Codeのターミナルから実行します。

成功すると、ECR上でコンテナイメージがプッシュされていることが確認できます。

[![スクリーン ショット 2024-08-17 に 19.16.35 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/06130caa-0280-f0c5-f6ee-cd2568a29432.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F06130caa-0280-f0c5-f6ee-cd2568a29432.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=f68e0dbc761c18970208d15db83b67c3)

## 3\. AWS環境の構築（後編）

アプリケーションのコンテナ登録が無事に実施できたら、これを稼働させるための残りのAWSインフラを構築します。

### 3-1. VPCの作成

今回のStreamlitアプリを稼働させるECSのコンテナ、およびALBを配置するためのVPCを作成します。

VPCコンソールの「VPCを作成」ボタンより、以下設定でVPCを作成しましょう。

- 作成するリソース： VPCなど
- 名前タグの自動生成： `bsc` （bedrock-simple-chatの略）

[![スクリーン ショット 2024-08-16 に 21.30.57 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/107b9bb2-bad6-786a-6c82-d627f33652e8.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F107b9bb2-bad6-786a-6c82-d627f33652e8.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=2151a646dce543b85d72c95fe2167f08)

他はデフォルトでOKです。東京リージョンにマルチAZでVPCが作成されます。

ECSサービスを作成するために、AZの数は2以上が必要となります。

### 3-2. VPCエンドポイント＆セキュリティグループの作成

次に、プライベートサブネットのECSタスク（コンテナ）から、VPC外のAWSサービスへアクセスするためのエンドポイントを作成します。

まずは「ゲートウェイ型」のエンドポイントを作成します。これは無料で利用できます。

- com.amazonaws.ap-northeast-1.s3（S3用） ※VPC作成時に自動作成される
- com.amazonaws.ap-northeast-1.dynamodb（DynamoDB用）

VPCは `bsc-vpc` を選択し、ルートテーブルはプライベートサブネットに自動作成されている2つを指定します。

[![スクリーン ショット 2024-08-17 に 19.21.58 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/1da3bbf5-a7b5-ae9c-cd01-723ca02594cd.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F1da3bbf5-a7b5-ae9c-cd01-723ca02594cd.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=6227a0a0e8333ff95da52e3e4297d0c9)

次に「インターフェイス型」のエンドポイントも作成します。こちらは稼働時間＋通信量でそれぞれ課金が発生します。

- com.amazonaws.ap-northeast-1.ecr.api（ECR用）
- com.amazonaws.ap-northeast-1.ecr.dkr（ECR用）
- com.amazonaws.ap-northeast-1.logs（CloudWatch用）
- com.amazonaws.ap-northeast-1.bedrock-runtime（Bedrock用）

いずれも `bsc-vpc` を選択し、サブネットはいずれか一つのみにチェックを入れ、プライベートサブネットのIDを選択します。セキュリティグループは `default` にチェックを入れます。

[![スクリーン ショット 2024-08-17 に 19.27.42 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/07f840d5-5c99-b6ec-01e1-7d9aff7de2ca.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F07f840d5-5c99-b6ec-01e1-7d9aff7de2ca.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=f587bde70fb09f61cd3a1560e358db2b)

S3用のエンドポイントは自動作成されているので、計5つのエンドポイントを追加作成しましょう。

[![スクリーン ショット 2024-08-17 に 19.30.37 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/66c10ce5-0661-5c37-f122-a2e77badecbd.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F66c10ce5-0661-5c37-f122-a2e77badecbd.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=f5de6665149b1eef4d0b64259430e3e3)

また、セキュリティの観点から、今回作成するWebアプリへのアクセスを自分のPCのみに制限するため、ALB用のセキュリティグループを事前に作成しておきます。 `VPC > セキュリティグループ` より作成できます。

- 基本的な詳細
	- セキュリティグループ名： `bsc-sg-alb`
	- 説明： `SG for ALB`
	- VPC： `bsc-vpc` のIDを選択
- インバウンドルール
	- タイプ： HTTP
	- ソース： マイIP

[![スクリーン ショット 2024-08-17 に 19.34.17 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/9126eeea-dc60-e5bc-f9b9-d6451cbd8f0a.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F9126eeea-dc60-e5bc-f9b9-d6451cbd8f0a.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=e1b780fdcc3ea6ee8e6abdbf464798aa)

### 3-3. コンテナ用のIAMロール作成

IAMのコンソールより、ECSの「タスクロール」用のIAMロールを次のとおり作成しておきましょう。

- 信頼されたエンティティタイプ： AWSサービス
- ユースケース： Elastic Container Service Task
- 許可ポリシー：
	- AmazonBedrockFullAccess
	- AmazonDynamoDBFullAccess
- ロール名： `bsc-task-role`

[![スクリーン ショット 2024-08-16 に 22.25.08 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/26bf6c8f-2759-b87f-c239-93cee2cc93a2.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F26bf6c8f-2759-b87f-c239-93cee2cc93a2.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=06b2e92cfaead3c26cf85554e5096d02)

### 3-4. ECSのクラスター＆サービス作成

このアプリケーションが実際に稼働するコンテナをホストするインフラを構築します。

Amazon ECSのコンソールへ移動し、タスク定義を作成します。これがコンテナの設定ファイルとなります。

- タスク定義の設定
	- タスク定義ファミリー： `bsc`
- インフラストラクチャの要件
	- タスクロール： `bsc-task-role`
- コンテナ - 1
	- 名前： `bsc`
	- イメージURL： ECRのプライベートレジストリ `bsc` のURLをコピー
	- コンテナポート： 8501

[![スクリーン ショット 2024-08-16 に 22.24.22 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/4a123076-38a7-4b96-192b-5fd1cb3542ae.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F4a123076-38a7-4b96-192b-5fd1cb3542ae.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=52bf7609c2448ffc801c836f30500244)

他はデフォルトでOKです。

次に、コンテナを稼働させるクラスターを作成します。名前以外はデフォルトでOKです。

- クラスター名： `bsc`

クラスターが作成完了したら、この中にサービスを作成します。

- 環境
	- コンピューティングオプション： 起動タイプ
- デプロイ設定
	- ファミリー： `bsc`
	- リビジョン： `1（最新）`
	- サービス名： `bsc`
- ネットワーキング
	- VPC： `bsc-vpc` のIDを選択
	- サブネット： プライベートサブネット2つを選択
	- パブリックIP： オフ
- ロードバランシング
	- ロードバランサーの種類： Application Load Balancer
	- コンテナ： `bsc 8501:8501` ※ポート番号が異なる場合はタスク定義を再確認
	- ロードバランサー名： `bsc`
	- ターゲットグループ名； `bsc`

[![スクリーン ショット 2024-08-17 に 19.57.57 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/946039ff-47f5-917d-ad8b-0a2292e87dd7.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F946039ff-47f5-917d-ad8b-0a2292e87dd7.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=3b117dccb4ca5c237ae2a9b7bbe743a9)

他はデフォルトでOKです。

作成後、「タスク」というコンテナの入れ物が自動で起動されます。その他にもALBなど、指定したAWSリソースがCloudFormationにより自動で作成されていきます。数分かかるのでコンソールを確認しながら待ちましょう。

### 3-5. ALBの追加設定

タスク定義から自動作成されるALBは、タスクと同じくVPCのプライベートサブネットに配置されてしまうので、インターネットからアクセスできるようにパブリックサブネットへ移動します。

作成したECSサービスの「ロードバランサーを表示」ボタンからALBのコンソールへ移動できます。

[![スクリーン ショット 2024-08-17 に 20.09.28 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/7ef43f66-67fd-2d13-d0b5-f7437ea23dbc.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F7ef43f66-67fd-2d13-d0b5-f7437ea23dbc.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=3a82fcb719db453adb183eedd9ec9008)

ALB（ `bsc` ）設定画面の `アクション > サブネットを編集` から、両方のAZともパブリックサブネットへ変更しておきます。

[![スクリーン ショット 2024-08-17 に 20.12.50 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/214a2d18-2fb9-dae5-d38f-e4d440254927.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F214a2d18-2fb9-dae5-d38f-e4d440254927.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=522c11d4d465a13df5ac6ba61234dbfa)

また、 `アクション > セキュリティグループを編集` から、先ほど作成した外部アクセス許可用のセキュリティグループ `bsc-sg-alb` を追加でアタッチしておきましょう。これにより、あなたの開発PCからALBへのアクセスが可能になります。

[![スクリーン ショット 2024-08-17 に 20.15.13 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/a719620c-b5c0-8d6e-281f-770ae394ffbb.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2Fa719620c-b5c0-8d6e-281f-770ae394ffbb.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=2d58c243954a02bab49b84e7d498dc7d)

## 4\. デプロイ後の動作確認

ECSタスクが無事に起動したら、あなたのStreamlitアプリにWebブラウザからアクセスできるようになります。

マネコンから今回作成したALBのDNS名をコピーし、ブラウザのアドレスバーに `http://ALBのDNS名` を貼り付けてアクセスしてみましょう。

※http **s** ではないので注意！

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/1591935b-3b57-33ce-4128-43cea0c0d477.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2F1591935b-3b57-33ce-4128-43cea0c0d477.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=6f24b4b86f097d12f894543fdd7beca1)

### うまくアクセスできないときは

VPC周りの設定は地味に難しいため、初心者は一回でスッと繋がるケースの方が珍しいのではないかと思います。以下をヒントにして、ChatGPTなどをうまく使いながらトラブルシューティングしてみてください。

切り分けのためのヒント：

- ALBのSGに設定した、あなたの開発PCのグローバルIPは変わっていないか？
- ECSタスクのログにアプリケーションのエラーは出ていないか？
- ブラウザからアクセスしたときの挙動
	- 読み込みのグルグルが終わらなければ、セキュリティグループなどネットワーク系の設定に阻まれている可能性あり

### 今回作成したコード

以下のGitHubリポジトリに公開しています。

## 次のステップ

ここまでうまくいった方は、以下にも挑戦してみましょう。

- Route 53などで独自ドメインを取得し、ACMの証明書をALBに設定してHTTPSで暗号化通信できるようにする
- Cognitoで認証機能を導入して、セキュリティグループによるソースIP制限を不要にする
- Bedrockのナレッジベースやエージェント機能を利用して、バックエンドをさらに多機能にしてみる

## おまけ

今回利用したサンプルアプリは、以下書籍の第3章のハンズオンから抜粋したものです。

他にもAWSの生成AI入門にぴったりなハンズオンを多数掲載していますので、興味ある方はぜひお手にとってみてください！

[![重版.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/aa798a3d-ae31-43ff-eab2-9005c9dd1ec1.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1633856%2Faa798a3d-ae31-43ff-eab2-9005c9dd1ec1.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=f6a80812d772bf5fd27520f58aefc4ff)

[0](https://qiita.com/minorun365/items/#comments)

[216](https://qiita.com/minorun365/items/84bef6f06e450a310a6a/likers)

222