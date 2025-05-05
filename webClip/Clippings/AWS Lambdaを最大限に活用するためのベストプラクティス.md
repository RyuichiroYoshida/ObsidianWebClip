---
title: AWS Lambdaを最大限に活用するためのベストプラクティス
source: https://qiita.com/kazuya-sugawara/items/6ae659fd760a9c486605
author:
  - "[[Qiita]]"
published: 2024-07-12
created: 2025-05-06
description: はじめにサーバーレス大好きなエンジニアです！みなさん、AWS Lambdaを使ってますか？日常的に利用しているけど、最適化についてはあまり考えていない方も多いのではないでしょうか。実際、私も…
tags:
  - 1
  - クラウド
  - バックエンド
  - バックエンド設計
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

## はじめに

サーバーレス大好きなエンジニアです！

みなさん、AWS Lambdaを使ってますか？  
日常的に利用しているけど、最適化についてはあまり考えていない方も多いのではないでしょうか。

実際、私もあまり意識せずに使っていました。  
使い方をちょっと工夫するだけで、 **Lambdaのパフォーマンスがぐんと上がる** んです！

今回は、AWSのドキュメントに書かれているベストプラクティスを参考にしながら、Lambdaを最大限に活用する方法をお伝えします。

各セクションのタイトルを見ると、ちょっと難しそうに感じるかもしれませんが、できるだけわかりやすく解説していきますので、ぜひ参考にしてみてください！

## 対象読者

- Lambdaを使ったことがある人
- もっと効果的に使いたいと感じている方
- チームの開発効率とコードの品質を向上させたい方
- Lambdaに興味がある方

では、早速始めていきましょう！

## 目次

1. ハンドラー関数について
2. SDKクライアントとデータベース接続の初期化
3. 静的アセットのローカルキャッシュ
4. 環境変数を使用して関数に設定を渡す
5. デプロイパッケージ内の依存関係を管理する
6. 再帰的なLambda関数の呼び出しを避ける
7. 非公開APIを使用しない

## 1\. ハンドラー関数について

まずは、Lambdaの最初に宣言するハンドラー関数についてお話します。

**Lambdaハンドラーをコアロジックから分離することが重要です。**

AWS Lambda関数の設計において、ハンドラーをコアロジックから分離することは、コードの保守性やテストのしやすさを向上させるために重要です。このアプローチにより、関数の各部分を独立してテストできるため、バグの発見と修正が容易になります。

主なメリットは次の3つです。

1. ユニットテストの容易さ
2. コードの再利用
3. コードの可読性

まず、コアロジックを別の関数として定義します。

```python
def process_data(data):
    # ビジネスロジックの実装
    processed_data = data.upper()
    return processed_data
```

次に、Lambdaハンドラーではコアロジック関数を呼び出します。

```python
def lambda_handler(event, context):
    # イベントからデータを抽出
    data = event['data']
    
    # コアロジックを呼び出す
    result = process_data(data)
    
    # 結果を返す
    return {
        'statusCode': 200,
        'body': result
    }
```

このようにすることで、Lambda関数の保守性が向上し、テストもしやすくなります。ぜひ試してみてください！

## 2\. SDKクライアントとデータベース接続の初期化

AWS Lambda関数では、SDKクライアントやデータベース接続を **関数ハンドラーの外部で初期化すること** が推奨されます。

### 関数ハンドラーの外部で初期化するとは？

通常、AWS Lambda関数は以下のように関数ハンドラー内でコードを実行します。

```python
def lambda_handler(event, context):
    # SDKクライアントの初期化
    s3 = boto3.client('s3')
    # データベース接続の初期化
    connection = create_database_connection()
    # 実際の処理
    ...
```

このようにすると、関数が呼び出されるたびにSDKクライアントやデータベース接続が再初期化され、パフォーマンスが低下します。

### 外部で初期化する方法

関数ハンドラーの外部でこれらのリソースを初期化する方法は以下の通りです。

```python
# SDKクライアントの初期化
s3 = boto3.client('s3')
# データベース接続の初期化
connection = create_database_connection()

def lambda_handler(event, context):
    # 実際の処理
    ...
```

この方法により、関数の呼び出しごとに初期化を行わず、パフォーマンスを向上させることができます！

## 3\. 静的アセットのローカルキャッシュ

次のパフォーマンス最適化方法は、 **静的アセットをローカルにキャッシュすること** です。ここでは、その具体的な意味と方法について説明します。

### 静的アセットのキャッシュとは？

静的アセットとは、関数が実行時に必要とする変更のないファイルやデータ（例：画像ファイル、設定ファイル、テンプレートファイルなど）を指します。これらのファイルをLambda関数の一時ディレクトリである `/tmp` にキャッシュすることで、後続の呼び出しで再ダウンロードや再読み込みの必要がなくなり、パフォーマンスが向上します。

主なメリットは次の3つです。

1. 関数の実行時間の短縮
2. リソースの節約
3. コスト削減

静的アセットをローカルキャッシュする例を示します。

```python
import os
import boto3

# S3クライアントの初期化
s3 = boto3.client('s3')
bucket_name = 'your-bucket-name'
file_key = 'path/to/your/static/asset'
local_path = '/tmp/asset_file'

# /tmpディレクトリにファイルが存在しない場合、ダウンロード
if not os.path.exists(local_path):
    s3.download_file(bucket_name, file_key, local_path)

def lambda_handler(event, context):
    # /tmpディレクトリにキャッシュされたファイルを使用
    with open(local_path, 'r') as file:
        data = file.read()
    
    # 実際の処理
    ...
```

このコードでは、最初にS3からファイルをダウンロードし、 `/tmp` ディレクトリに保存します。後続の呼び出しでは、このファイルが既に存在するため、再ダウンロードせずにそのまま利用できます。

ファイルの再ダウンロードや再読み込みが不要になり、関数の実行時間が短縮されパフォーマンスが向上します。  
また、データ転送や処理にかかるリソースを節約できるためコスト削減にもつながります。

## 4\. 環境変数を使用して関数に設定を渡す

AWS Lambda関数で設定やパラメータを渡す際には、 [環境変数](https://docs.aws.amazon.com/lambda/latest/dg/configuration-envvars.html) を使用することが推奨されます。これにより、コードの柔軟性と保守性が向上します。

例えば、Amazon S3バケットに書き込む場合、 **バケット名をコードに直接書き込むのではなく、環境変数として設定する** 方法です。  
デプロイごとに異なる設定を簡単に適用でき、異なる環境（開発、テスト、本番）に対応しやすくなります。

```python
import os
import boto3
from botocore.exceptions import ClientError

# 環境変数からバケット名を取得
bucket_name = os.environ['S3_BUCKET_NAME']

s3_client = boto3.client('s3')

def lambda_handler(event, context):
    # S3に書き込むためのパラメータ設定
    key = 'example.txt'
    body = 'Hello from Lambda!'

    try:
        response = s3_client.put_object(Bucket=bucket_name, Key=key, Body=body)
        return {
            'statusCode': 200,
            'body': f'Successfully uploaded to {bucket_name}/{key}'
        }
    except ClientError as e:
        return {
            'statusCode': 500,
            'body': f'Error: {str(e)}'
        }
```

このコードでは、 `os.environ['S3_BUCKET_NAME']` を使用して環境変数からバケット名を取得し、そのバケットにデータを書き込んでいます。これにより、バケット名をコード内にハードコーディングする必要がなくなり、柔軟性と保守性が向上します。

ぜひ、環境変数を活用してLambda関数をより効率的に管理してみてください！

## 5\. デプロイパッケージ内の依存関係を管理する

AWS Lambda関数を実行する環境には、Node.jsやPythonランタイム用のAWS SDKなど、多数のライブラリが含まれています。  
これらのライブラリは定期的に更新され、最新の機能やセキュリティパッチが適用されますが、それに伴ってLambda関数の動作が微妙に変わる可能性があります。したがって、関数が使用する依存関係を完全に制御するためには、 **すべての依存関係をデプロイパッケージに含めることが重要です。**

### Pythonの例

```text
my-lambda-function/
├── boto3/
├── botocore/
├── lambda_function.py
└── <other dependencies>
```

### Node.jsの例

```text
my-lambda-function/
├── node_modules/
├── package.json
└── index.js
```

依存関係を適切に管理することで、Lambda関数のパフォーマンスとセキュリティを向上させることができます。ぜひ、デプロイパッケージに依存関係を含める方法を実践してみてください！

## 6\. 再帰的なLambda関数の呼び出しを避ける

**AWS Lambda関数内で任意の条件が満たされるまで、その関数自身を再帰的に呼び出すようなコードを使用しないでください** 。これを行うと、意図しないボリュームで関数が呼び出される可能性があり、結果として料金が急増する危険性があります。

もし誤ってこのようなコードを使用してしまった場合は、すぐに関数の予約済同時実行数を0に設定して、コードを更新している間のすべての関数の呼び出しをスロットリングすることが推奨されます。

再帰的呼び出しを避けるべき理由は次の3つです。

1. コストの急増
2. リソースの枯渇
3. デバッグの難しさ

### 再帰的呼び出しの例（避けるべき）

```python
def lambda_handler(event, context):
    # 任意の条件が満たされるまで再帰的に自身を呼び出す
    if not event.get('stop'):
        invoke_lambda_again(event)
    return "Function executed"

def invoke_lambda_again(event):
    import boto3
    client = boto3.client('lambda')
    client.invoke(
        FunctionName='my-function',
        InvocationType='Event',
        Payload='{"stop": false}'
    )
```

### 再帰的呼び出しを避ける方法

再帰的呼び出しを避けるために、ループを使用して条件が満たされるまで処理を行う方法に変更します。

```python
def lambda_handler(event, context):
    while not event.get('stop'):
        # 任意の処理
        event['stop'] = check_condition(event)
    return "Function executed"

def check_condition(event):
    # 条件をチェックする処理
    # 例：条件が満たされた場合にTrueを返す
    return True  # 条件が満たされた場合
```

このように、再帰的呼び出しを避けることで、コストの急増やリソースの枯渇を防ぎ、デバッグを容易にすることができます。ぜひ、このベストプラクティスを取り入れて、Lambda関数を効率的に運用してください！

## 7\. 非公開APIを使用しない

AWS Lambda関数を作成する際、文書化されていない非公開のAPIを使用しないようにすることが重要です。Lambdaのマネージドランタイムは、セキュリティと機能の更新を定期的に適用しますが、これらの更新には後方互換性がないことがあります。

非公開APIに依存している場合、これらの更新により関数の呼び出しが失敗するなどの予期しない結果が発生する可能性があります。  
公開されている API のリストについては、 [API リファレンス](https://docs.aws.amazon.com/ja_jp/lambda/latest/api/welcome.html) を参照してください。

## おわりに

本ブログで紹介した方法を実践することで、パフォーマンス、信頼性、セキュリティを大幅に向上させることができます。  
これらのベストプラクティスを開発に取り入れて、AWS Lambdaをより効果的に活用していく参考になれば嬉しいです！

## 参考

[0](https://qiita.com/kazuya-sugawara/items/#comments)

[59](https://qiita.com/kazuya-sugawara/items/6ae659fd760a9c486605/likers)

52