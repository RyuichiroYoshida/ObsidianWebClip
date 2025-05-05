---
title: GitHub ActionsのJobが落ちたときに何をするべきかを記述するPlaybookの仕組みを作って運用している話
source: https://tech.newmo.me/entry/2024/09/04/130000
author:
  - "[[newmo 技術ブログ]]"
published: 2024-09-04
created: 2025-05-06
description: newmoではGitHub Actionsを自動テスト、Lint、デプロイなどに利用しています。 また、newmoではmonorepoで開発しているため、1つのリポジトリに複数のチーム/複数のアプリケーションが存在しています。 GitHub Actionsではpathsを使うことで、特定のファイルが変更された場合のみ特定のWorkflowが実行できます。 newmoのmonorepoのworkflowでは基本的にpathsが指定されていますが、それでも普段は触らないファイルを変更して意図せずにCIが落ちることがあります。 GitHub ActionsのCIが落ちたときに、そのCIの仕組みを作っ…
tags:
  - CI-CD
  - 1
read: false
---
newmoではGitHub Actionsを自動テスト、Lint、デプロイなどに利用しています。 また、newmoではmonorepoで開発しているため、1つのリポジトリに複数のチーム/複数のアプリケーションが存在しています。

GitHub Actionsでは [paths](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions#onpushpull_requestpull_request_targetpathspaths-ignore) を使うことで、特定のファイルが変更された場合のみ特定のWorkflowが実行できます。 newmoのmonorepoのworkflowでは基本的に `paths` が指定されていますが、それでも普段は触らないファイルを変更して意図せずにCIが落ちることがあります。 GitHub ActionsのCIが落ちたときに、そのCIの仕組みを作った人やチーム以外だと何をすべきかわからないことがあります。

この問題の解決するを手助けするシンプルな仕組みとして、GitHub ActionsにCIが落ちたときに何をするべきかを表示するPlaybookの仕組みを導入しました。

## Playbookの仕組み

Playbookといってもやっていることはとても単純です。

GitHub ActionsのWorkflowのJobが失敗したときに、そのJobが失敗した理由とその対処方法を表示するだけです。

具体的には次のような [composite action](https://docs.github.com/en/actions/sharing-automations/creating-actions/creating-a-composite-action) を作成し、各Jobの最後に `if: failure()` で実行するようにしています。 Composite actionはGitHub ActionsのWorkflowから呼び出せる関数的なActionを定義する仕組みです。

次のようなComposite actionを作成します。

`.github/actions/playbook/action.yaml`:

```yaml
---
name: "playbook"
description: "CIのJOBが落ちた時にどのように対応するべきかを書く"
inputs:
  message:
    description: "How to fix?"
    required: true
runs:
  using: "composite"
  steps:
    - name: How to Fix?
      uses: actions/github-script@v7.0.1
      env:
        INPUT_MESSAGE: ${{ inputs.message }}
      with:
        script: |
          const message = process.env.INPUT_MESSAGE;
          core.summary.addRaw(message, true);
          core.summary.write(); // Summaryへ出力
          core.setFailed(message); // Jobページ開いた時に自動的に開いた状態にするためFailedを設定し直す
```

そして、Workflowの各Jobの最後に次のように記述するだけです。

`.github/workflows/ci.yaml`:

```yaml
name: CI Build

jobs:
  build:
    permissions:
      contents: "read"
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - name: 既存の色々な処理
        run: echo "既存の処理"
      # Jobが落ちた場合のみ、Playbookを実行する
      - name: Playbook
        if: failure()
        uses: ./.github/actions/playbook
        with:
          message: |
            # ${{ github.job }} が失敗しました

            ## 影響
            - 何が影響を受けるか

            ## 調査方法
            - Jobの落ちてるステップのエラーログを確認してください

            ## 修正方法 
            - どのように対応するか

            ## 修正後の確認方法
            - 修正後にどのように確認するか
```

これで、CIが落ちたときにGitHub Actionsのログや [Job Summaries](https://github.blog/news-insights/product-news/supercharging-github-actions-with-job-summaries/) に対処方法が表示されるようになります。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/n/newmo/20240830/20240830225409.png)

PlaybookのJob Summaryの表示例

あとは、これを見て対処するだけです。

newmoのCIでは、各Jobに次のようなテンプレートでPlaybookを記述しています。

```markdown
# ${{ github.job }} が失敗しました

## 影響
- 何が影響を受けるか
- e.g. デプロイが失敗しているので、本番環境に反映されていない

## 調査方法
- 原因となるエラーを特性する方法を記述する
- e.g. Jobの落ちてるステップのエラーログを確認してください

## 修正方法 
- どのように対応するか
- e.g. どのファイルを修正すればいいか

## 修正後の確認方法
- 修正後にどのように確認するか
- e.g. どのコマンドを実行すればいいか
```

CIのWorkflowを書いた人は、CI上に表示されるエラーを見るだけでわかるかもしれませんが、必ずしも他の人がわかるとは限りません。 そのため、インシデント対応のプレイブックのように、CIが落ちたときに何をすべきかを表示するPlaybookの仕組みを導入することで、CIの運用を円滑にすることができます。

実際にこの仕組みを導入してから、フロントエンドに関するCIが落ちた場合にも、普段はフロントエンドを触っていないバックエンドのエンジニアがCIが落ちた原因を修正できる事例もありました。

## おわりに

newmoでは、GitHub ActionsのWorkflowが落ちたときに何をすべきかを表示するPlaybookの仕組みを導入しています。 仕組み的にはとてもシンプルで、 `if: failure()` で何をすればいいかを書けるだけの仕組みです。 単純な仕組みですが、CIの運用を円滑にするためにはとても有効な仕組みだと感じています。

PR: newmoではエンジニアを募集しています！ 興味がある方は、次の採用情報をご覧ください。

- [採用情報 newmo株式会社(ニューモ)](https://careers.newmo.me/)