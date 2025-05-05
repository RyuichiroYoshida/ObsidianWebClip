---
title: ShellScriptで自動化を楽にしたい時に知っておいても良いこと | sreake.com | 株式会社スリーシェイク
source: https://sreake.com/blog/shellscript-good-practices/
author:
  - "[[kobayashikenta]]"
published: 2024-07-15
created: 2025-05-06
description: はじめに こんにちは、皆さん。今日は、シェルスクリプトを使った高度な自動化のベストプラクティスとパターンについ
tags:
  - Tech
  - Tools
  - "#ShellScript"
read: false
---
## はじめに

こんにちは、皆さん。今日は、シェルスクリプトを使った高度な自動化のベストプラクティスとパターンについて解説します。これらは、ちょっとした知識で実行でき、作業を大幅に効率化できるTipsです。シェルスクリプトは、特にUNIX系システムでの自動化タスクに欠かせないツールです。適切に使用すれば、複雑なタスクを効率的に、そして信頼性高く実行できます。

トイルとは、反復的でマニュアルな作業のことを指します。これには、例えば、手動でのシステムのスケーリングや、エラーのトラブルシューティング、ルーティンなメンテナンス作業などが含まれます。トイルを特定し、それを自動化することで、エンジニアはより創造的なタスクやプロジェクトに焦点を合わせることができます。

トイルを判別する方法としては、以下のような基準が挙げられます：

1. 手作業であること 完全な手作業だけでなく、「あるタスクを自動化するためのスクリプトを、手作業で実行する」ことも含まれます。
2. 繰り返されること 1度、2度で終了する作業ではなく、繰り返し何度も行われる作業です。
3. 自動化が可能なこと そのタスクの処理において「手作業と同レベルで自動化が可能」または「タスクの必要性がなくなる仕組みを作れる」場合を指します。
4. 戦術的であること 戦略的なタスク、または予測に基づくタスクではなく、通常タスクに割り込んで行われる問題対応的なタスクを指します。
5. 長期的な価値がないこと 「古いコードや設定を踏み込んで整理する」といった、短期的には必要だが、長期的にサービスに価値をもたらすわけではないタスクを指します。
6. サービスの成長に比例して増加すること サービスのサイズ、トラフィックの量、ユーザー数などに正比例して増加するタスクは、おそらくトイルである可能性が高いといえます。

これらの基準に当てはまるタスクは、自動化の良い候補となります。シェルスクリプトを活用することで、これらのトイルを効果的に自動化し、エンジニアの時間と労力を節約することができます。

このブログでは、シェルスクリプトを使ったトイル削減の具体的な方法と、それを実現するためのいくつかのTips について詳しく見ていきます。

**参考資料:**

- [ShellCheckで自動化の品質を向上させる](https://sreake.com/blog/shellcheck-automation-enhancement/)
- [Identifying and tracking toil using SRE principles](https://cloud.google.com/blog/products/management-tools/identifying-and-tracking-toil-using-sre-principles)
- [SREにおけるトイルの判断と切り分け方](https://sreake.com/blog/sre-toil-select/)

## 1\. エラーハンドリング：失敗に備える

### set -e：エラーで即座に停止

```
#!/bin/bash
set -e
set -o pipefail

# このスクリプトは、エラーが発生したらすぐに停止します
echo "Starting the script"
non_existent_command  # この行でスクリプトは停止します
echo "This line will never be executed"
```

`set -e` を使用すると、スクリプトはエラーが発生した時点で即座に停止します。 `set -o pipefail` を追加することで、パイプラインの一部でエラーが発生した場合も確実にスクリプトを停止させることができます。

### トラップを使用したクリーンアップ

```
#!/bin/bash

function cleanup() {
    echo "Cleaning up temporary files..."
    rm -f /tmp/tempfile_$$
}

trap cleanup EXIT

# スクリプトの主要な処理
echo "Creating temporary file..."
touch /tmp/tempfile_$$
# ... その他の処理 ...
```

`trap` コマンドを使用することで、スクリプトが終了する際（正常終了でもエラー終了でも）に特定の処理を実行することができます。これは一時ファイルの削除などのクリーンアップ処理に特に有用です。

### 構造化ログの実装

何が起こったのかを知るためにはログが重要です。構造化ログを実装することで、イベントの詳細を効率的に記録し、分析できます。日時、ログレベル、メッセージを含む一貫したフォーマットを使用し、JSONなどの機械可読形式で出力することで、ログの検索や集計が容易になります。これにより、問題の迅速な特定と解決が可能になります。

```
#!/bin/bash

log() {
    local level="$1"
    shift
    echo "$(date '+%Y-%m-%d %H:%M:%S') [$level] $*"
}

# 使用例
log INFO "Script started"
log DEBUG "Processing file: $filename"
log ERROR "Failed to connect to the database"
```

構造化ログを使用することで、ログの解析や問題の追跡が容易になります。

### 再実行可能なスクリプトを書く

冪等性のあるコードを書くことは重要です。これは、スクリプトを何度実行しても、同じ結果が得られることを意味します。例えば、ファイルの作成やディレクトリの設定など、システムの状態を変更する操作を行う場合、既に目的の状態になっているかどうかを最初にチェックします。これにより、不要な処理を避け、エラーを防ぐことができます。また、スクリプトの途中で失敗した場合でも、再実行時に問題なく続きから処理を行えるようになります。冪等性を意識することで、より信頼性の高い、メンテナンスしやすいスクリプトを作成することができます。

#### ディレクトリの作成をする時

```
#!/bin/bash
create_directory() {
    if [ ! -d "$1" ]; then
        mkdir -p "$1" && echo "Directory $1 created"
    else
        echo "Directory $1 already exists"
    fi
}

# 使用例
create_directory "/path/to/my/directory"
```

この関数は、ディレクトリが存在しない場合のみ作成を行います。これにより、スクリプトを何度実行しても安全です。実はもう少し良い方法で実行する方法があるのでどこかで共有したいです。

#### パッケージのインストールをする時

```
#!/bin/bash

install_package() {
    if ! dpkg -l | grep -q "^ii  $1"; then
        sudo apt-get install -y "$1"
        echo "Package $1 installed"
    else
        echo "Package $1 is already installed"
    fi
}

# 使用例
install_package "nginx"
```

この関数は、パッケージがまだインストールされていない場合のみインストールを行います。

## 2\. パフォーマンス最適化

### ループの最適化

スクリプトでのループ最適化には、一般的に以下の二つの方法があります：

```
#!/bin/bash
# 方法1: seqコマンドを使用
for i in $(seq 1 1000000); do
    echo $i > /dev/null
done

# 方法2: bashの算術式を使用
for ((i=1; i<=1000000; i++)); do
    echo $i > /dev/null
done
```

従来、方法2（bashの算術式）が方法1（seqコマンド）より効率的だと考えてきました。その理由として、外部プロセスの呼び出し回避、メモリ使用量の削減、ループオーバーヘッドの削減が挙げられていました。

しかし、実際の測定結果は予想と異なる場合があります：

1. `seq` を使用する方法が実際には高速である可能性
2. 両者の性能差がシステムや使用状況により変動する可能性

この予想外の結果には、以下の要因が考えられます：

- `seq` コマンドの最適化された実装
- Bashの算術演算の相対的な遅さ
- メモリとプロセス生成のトレードオフ
- システムの特性（CPUキャッシュ、メモリ管理など）

したがって、ループ最適化には以下の点を考慮することが重要です：

1. 実際の環境での測定の重要性
2. コンテキストと具体的なユースケースの考慮
3. 大規模データを扱う際の別アプローチ（例：ストリーミング処理）の検討
4. ループ構造だけでなく、ループ内の処理も含めた全体的な最適化

結論として、スクリプトの最適化は複雑で、直感に反する結果をもたらすことがあります。常に具体的なユースケースに基づいてベンチマークを行い、その結果に基づいて最適化を行うことが重要です。これは神 [yteraoka](https://x.com/yteraoka) さんの指摘で気付くことができました。ありがとうございます。強すぎる…。

### パイプラインの使用

```
#!/bin/bash

# ファイルを1行ずつ読み込んで処理
while IFS= read -r line; do
    echo "Processing: $line"
done < large_file.txt

# パイプラインを使用した効率的な処理
cat large_file.txt | while IFS= read -r line; do
    echo "Processing: $line"
done
```

大きなファイルを処理する場合、パイプラインを使用することで、メモリ使用量を抑えつつ効率的に処理を行うことができます。

## 3\. セキュリティの考慮事項

### 入力のサニタイズ

```
#!/bin/bash

read -p "Enter a filename: " filename

# 危険な方法（ユーザー入力をそのまま使用）
# cat $filename

# 安全な方法（入力をサニタイズ）
if [[ $filename =~ ^[A-Za-z0-9._-]+$ ]]; then
    cat "$filename"
else
    echo "Invalid filename"
    exit 1
fi
```

ユーザー入力を処理する際は、常に入力をサニタイズし、潜在的な悪意のある入力を防ぐことが重要です。

### 変数の適切な引用

```
#!/bin/bash

filename="My Document.txt"

# 悪い例（スペースを含むファイル名で問題が発生する可能性がある）
rm $filename

# 良い例（変数を適切に引用することで、スペースを含むファイル名でも正しく動作する）
rm "$filename"
```

変数を適切に引用することで、予期せぬ動作や潜在的なセキュリティリスクを回避できます。

### 最小権限の原則

```
#!/bin/bash

# 一時的に権限を下げる
if [[ $EUID -eq 0 ]]; then
    original_user=$(logname)
    sudo -u $original_user bash << EOF
        # 権限を必要としない操作をここで実行
        echo "Running with reduced privileges"
EOF
    # 権限が必要な操作はここで実行
    echo "Running with elevated privileges"
else
    echo "This script must be run as root"
    exit 1
fi
```

スクリプトが root 権限で実行される場合、必要最小限の操作のみを root で行い、それ以外は通常のユーザー権限で実行するようにします。しかし、デバッグが複雑になったりもするので多用は禁物

### 一時ファイルの安全な作成

```
#!/bin/bash

# mktemp を使用して安全に一時ファイルを作成
temp_file=$(mktemp)

# スクリプトの終了時に一時ファイルを削除
trap 'rm -f "$temp_file"' EXIT

# 一時ファイルを使用した処理をここに記述
```

`mktemp` を使用することで、安全に一時ファイルを作成でき、 `trap` を使用することでスクリプト終了時に確実に削除できます。

## 4\. クロスプラットフォームの考慮

### 可搬性のあるシバン

```
#!/usr/bin/env bash
```

`#!/bin/bash` の代わりに `#!/usr/bin/env bash` を使用することで、異なるシステムでも bash の場所を正しく特定できます。

### OS 依存の処理

```
#!/usr/bin/env bash

case "$(uname -s)" in
    Linux*)     machine=Linux;;
    Darwin*)    machine=Mac;;
    CYGWIN*)    machine=Cygwin;;
    MINGW*)     machine=MinGw;;
    *)          machine="UNKNOWN:${unameOut}"
esac

echo "Running on $machine"

if [ "$machine" = "Linux" ]; then
    # Linux 固有の処理
elif [ "$machine" = "Mac" ]; then
    # macOS 固有の処理
fi
```

`uname` コマンドを使用して実行環境を判別し、OS 固有の処理を適切に分岐させることができます。

## 5\. テストとデバッグ

### ユニットテストの導入

```
#!/usr/bin/env bash

# テスト対象の関数
add() {
    echo $(($1 + $2))
}

# テスト関数
test_add() {
    result=$(add 2 3)
    if [ "$result" -eq 5 ]; then
        echo "Test passed"
    else
        echo "Test failed"
    fi
}

# テストの実行
test_add
```

シンプルなユニットテストを導入することで、スクリプトの信頼性を向上させることができます。

### デバッグモード

```
#!/usr/bin/env bash

# デバッグモードを有効にする
if [ "$DEBUG" = "true" ]; then
    set -x
fi

# スクリプトの主要な処理
echo "Performing main operation"
# ... その他の処理 ...

# デバッグモードを無効にする
if [ "$DEBUG" = "true" ]; then
    set +x
fi
```

環境変数 `DEBUG` を設定することで、スクリプトの実行過程を詳細に追跡することができます。

## 6\. バージョン管理との統合

### Git フックの活用

```
#!/usr/bin/env bash

# .git/hooks/pre-commit に配置

# シェルスクリプトの構文チェック
for file in $(git diff --cached --name-only | grep '\\.sh$')
do
    if ! bash -n "$file"; then
        echo "Syntax error in $file"
        exit 1
    fi
done

# ShellCheck による静的解析
if command -v shellcheck > /dev/null; then
    for file in $(git diff --cached --name-only | grep '\\.sh$')
    do
        if ! shellcheck "$file"; then
            echo "ShellCheck failed for $file"
            exit 1
        fi
    done
else
    echo "ShellCheck not installed. Skipping."
fi
```

Git のpre-commitフックを使用して、コミット前に自動的にシェルスクリプトの構文チェックと静的解析を行うことができます。

## 7\. 実際のプロジェクトケーススタディ

### ログ解析と報告生成

```
#!/usr/bin/env bash

# Apacheのアクセスログを解析し、日次レポートを生成するスクリプト

log_file="/var/log/apache2/access.log"
report_file="/tmp/apache_daily_report_$(date +%Y%m%d).txt"

# 総アクセス数
total_access=$(wc -l < "$log_file")

# ユニーク訪問者数
unique_visitors=$(awk '{print $1}' "$log_file" | sort -u | wc -l)

# 最もアクセスの多いページ
top_page=$(awk '{print $7}' "$log_file" | sort | uniq -c | sort -rn | head -n 1)

# レポート生成
{
    echo "Apache Daily Report - $(date +%Y-%m-%d)"
    echo "=================================="
    echo "Total Accesses: $total_access"
    echo "Unique Visitors: $unique_visitors"
    echo "Most Accessed Page: $top_page"
} > "$report_file"

echo "Report generated: $report_file"
```

このスクリプトは、Apacheのアクセスログを解析し、総アクセス数、ユニーク訪問者数、最もアクセスの多いページなどの情報を含む日次レポートを生成します。cron ジョブとして設定することで、毎日自動的にレポートを生成することができます。このような実践的なシェルスクリプトを学びたい時には是非、「 **[マスタリングLinuxシェルスクリプト 第2版 ―Linuxコマンド、bashスクリプト、シェルプログラミング実践入門](https://www.oreilly.co.jp/books/9784814400119/ "https://www.oreilly.co.jp/books/9784814400119/")** 」を読んでください。

## まとめ

シェルスクリプトによる高度な自動化は、エラーハンドリング、パフォーマンス最適化、セキュリティ考慮、クロスプラットフォーム対応、テストとデバッグ、バージョン管理との統合、そして実際のユースケースの理解など、多くの要素を考慮する必要があります。これらのベストプラクティスとパターンを適用することで、より信頼性が高く、保守しやすい、そして効率的な自動化スクリプトを作成することができます。

シェルスクリプトの世界は深く、常に学ぶべきことがあります。この記事で紹介した技術を基礎として、さらに高度な自動化に挑戦してください。皆さんの自動化の旅が実り多きものになることを願っています！シェルスクリプトは強力ですが、同時にすべての問題に適しているわけではありません。シェルスクリプトの限界を理解する必要もあります

## SREの導入実績豊富な弊社がサポートします

SREを導入する際、トイルをどう判別するかは重要なポイントです。今回紹介したトイルの定義に基いて判別し、解決へのステップを進めることでトイル削減に着手できます。しかし、「 **トイルを判別できたが解決方法が分からない** 」「 **トイルが多すぎて優先順位がつけられない** 」「 **最適な自動化方法が分からない** 」といったケースもあるかと思います。

弊社はこれまで多くの企業様のSRE導入のお手伝いをしてきました。トイルの削減はもちろん、サービスの戦略策定から設計・構築・運用、ならびSREに必要なSaaSの導入支援までサービスの成長に必要な要素を統合的に提供可能です。

もし少しでもSREに興味があるという企業様がいらっしゃいましたら、気軽に [お問い合わせ](https://sreake.com/contact/) ください。

## 関連サービス

- [![AWS/Google Cloud/Azure 構築運用支援](https://sreake.com/wp-content/themes/sreake/assets/images/home/service_thumb01.jpg)
	AWS/Google Cloud/Azure 構築運用支援
	プロフェッショナルチームがAWS / Google Cloud / Azureの主要プラットフォームで導入を支援いたします。
	](https://sreake.com/service-sre/)
- [![Kubernetes 構築運用支援](https://sreake.com/wp-content/themes/sreake/assets/images/home/service_thumb02.jpg)
	Kubernetes 構築運用支援
	Kubernetesの圧倒的導入実績を誇るプロフェッショナルチームが、AWS / Google Cloud / Azureの各主要プラットフォームで導入を支援いたします。
	](https://sreake.com/service-sre/)

## 関連記事[インシデント管理ツール「PagerDuty」とはなにか \[特徴・機能・メリット\]](https://sreake.com/blog/incident-tool-pagerduty/)

[

この記事では、SRE導入に欠かせないインシデント管理ツール「PagerDuty」について解説します。

nwiizo

2021.10.1

](https://sreake.com/blog/incident-tool-pagerduty/)[aws-vault のすすめ](https://sreake.com/blog/aws-vault/)

[

Daichi Murota

2023.1.19

](https://sreake.com/blog/aws-vault/)[リソース不足の組織が、SREに取り組む際の3つのポイントについて解説 \[SREチーム構築\]](https://sreake.com/blog/sre-team-building/)

[

この記事では、SREチームを組織する際どのように進行すればスムーズにいくのかを解説します。

nwiizo

2022.1.17

](https://sreake.com/blog/sre-team-building/)

[ブログ一覧へ戻る](https://sreake.com/blog/)