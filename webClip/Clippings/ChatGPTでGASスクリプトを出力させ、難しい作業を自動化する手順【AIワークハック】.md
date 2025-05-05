---
title: ChatGPTでGASスクリプトを出力させ、難しい作業を自動化する手順【AIワークハック】
source: https://www.lifehacker.jp/article/2409-chatgpt-gas/
author:
  - "[[重田信]]"
published: 2024-09-04
created: 2025-05-06
description: ChatGPTを活用すると様々な業務を効率化することができますが、その一つにコーディングが挙げられます。この記事では、ChatGPTを使ってGAS（Google Apps Script）のスクリプトを出力し、スプレッドシートに記入したTo doリストをGoogleカレンダーに反映させる方法を紹介します。
tags:
  - AI
  - GAS
  - Tech
  - Tools
read: false
---
- [グローバルナビゲーションへジャンプ](https://www.lifehacker.jp/article/2409-chatgpt-gas/#globalNav)
- [フッターへジャンプ](https://www.lifehacker.jp/article/2409-chatgpt-gas/#footer)

- [TOP](https://www.lifehacker.jp/)
- ChatGPTでGASスクリプトを出力させ、難しい作業を自動化する手順【AIワークハック】

[使える！AIワークハック](https://www.lifehacker.jp/regular/ai_workhacks/)

著者 重田信

2024.09.04 lastupdate

![Image: Shutterstock](https://media.loom-app.com/loom/2024/09/03/7eb82b5d-a45c-4703-b2fe-a8cb68313576/original.png?w=563)

Image: Shutterstock

Advertisement

ChatGPTを活用すると様々な業務を効率化することができますが、 **その一つにコーディングが挙げられます。**

この記事では、 **ChatGPTを使ってGAS（Google Apps Script）のスクリプトを出力** し、スプレッドシートに記入したTo doリストをGoogleカレンダーに反映させる方法を紹介します。

## Google WorkspaceをカスタマイズできるGASとは

「GAS」とは「Google Apps Script」の略で、Googleが提供するスクリプト言語です。 **GASを使うと、データ処理やメールの自動返信、ファイル整理などが可能になります。** Googleアカウントがあれば、特別な設定を行うことなく、ブラウザ上でコードを構築することができます。

とはいえ、プログラミング初心者にとって、いきなりスクリプトを構築するのはハードルが高い気がしますよね。

そこで、ChatGPTを活用してみましょう。ChatGPTにGASのスクリプトを出力してもらい、スクリプトエディタに貼り付けて修正を繰り返していくことで、専門的なコーディングの知識や経験がなくても、Google Workspaceのアプリを連携させることができます。

Advertisement

## 作業の手順からコードの作成までChatGPTが出力

まず、ChatGPTに以下のプロンプトを投げかけます。

なお、この記事で紹介するプロンプトや出力された内容は、全てGPT-4oを使用した例になります。GASの構築はGoogle Chrome上で行ないました。

**プロンプト：**

**Googleスプレッドシートに入力されたタスクのTo-DoリストをGoogleカレンダーに反映するGoogle Apps Script (GAS) を作成してください。**

**このスクリプトは、スプレッドシートからタスク情報を取得し、優先度と所要時間に基づいてスケジュールを計算し、そのスケジュールをGoogleカレンダーに追加します。タスク情報には、タスク名、優先度、所要時間（時間単位）、および期限が含まれます。具体的には以下の機能が必要です:**

**1\. スプレッドシートからタスク情報を取得する**

**2\. タスクの期限に基づいてスケジュールを計算する**

**3\. 計算されたスケジュールをGoogleカレンダーに追加する**

**この要件を満たす完全なコードを提供してください。**

すると、以下の画像のように、具体的な手順とコードが提示されます。

![](https://media.loom-app.com/loom/2024/08/29/4af147d6-29f8-40dd-aa70-9fdcb30f85bb/original.png?w=640)

プロンプト： Googleスプレッドシートに入力されたタスクのTo-DoリストをGoogleカレンダーに反映するGoogle Apps Script (GAS) を作成してください。 このスクリプトは、スプレッドシートからタスク情報を取得し、優先度と所要時間に基づいてスケジュールを計算し、そのスケジュールをGoogleカレンダーに追加します。タスク情報には、タスク名、優先度、所要時間（時間単位）、および期限が含まれます。具体的には以下の機能が必要です: 1. スプレッドシートからタスク情報を取得する 2. タスクの期限に基づいてスケジュールを計算する 3. 計算されたスケジュールをGoogleカレンダーに追加する この要件を満たす完全なコードを提供してください。

まず新規のスプレッドシートを作成し、「タスク名」「優先度」「所要時間」「期限」を入力していきます。優先度は5段階（1が最も緊急度が高い）で表しました。

![](https://media.loom-app.com/loom/2024/08/29/a1050a36-f4a8-4af6-860d-7e69022d0d52/original.png?w=640)

プロンプト： Googleスプレッドシートに入力されたタスクのTo-DoリストをGoogleカレンダーに反映するGoogle Apps Script (GAS) を作成してください。 このスクリプトは、スプレッドシートからタスク情報を取得し、優先度と所要時間に基づいてスケジュールを計算し、そのスケジュールをGoogleカレンダーに追加します。タスク情報には、タスク名、優先度、所要時間（時間単位）、および期限が含まれます。具体的には以下の機能が必要です: 1. スプレッドシートからタスク情報を取得する 2. タスクの期限に基づいてスケジュールを計算する 3. 計算されたスケジュールをGoogleカレンダーに追加する この要件を満たす完全なコードを提供してください。

入力したらメニューバーの拡張機能からApps Scriptを選択します。

![](https://media.loom-app.com/loom/2024/08/29/212e9e6a-ccf7-4641-867a-eb697b1682e9/original.png?w=640)

プロンプト： Googleスプレッドシートに入力されたタスクのTo-DoリストをGoogleカレンダーに反映するGoogle Apps Script (GAS) を作成してください。 このスクリプトは、スプレッドシートからタスク情報を取得し、優先度と所要時間に基づいてスケジュールを計算し、そのスケジュールをGoogleカレンダーに追加します。タスク情報には、タスク名、優先度、所要時間（時間単位）、および期限が含まれます。具体的には以下の機能が必要です: 1. スプレッドシートからタスク情報を取得する 2. タスクの期限に基づいてスケジュールを計算する 3. 計算されたスケジュールをGoogleカレンダーに追加する この要件を満たす完全なコードを提供してください。

すると新しいタブに以下のような画面が立ち上がります。

![](https://media.loom-app.com/loom/2024/08/29/db74cda6-4a50-4623-b187-683b0c746e23/original.png?w=640)

プロンプト： Googleスプレッドシートに入力されたタスクのTo-DoリストをGoogleカレンダーに反映するGoogle Apps Script (GAS) を作成してください。 このスクリプトは、スプレッドシートからタスク情報を取得し、優先度と所要時間に基づいてスケジュールを計算し、そのスケジュールをGoogleカレンダーに追加します。タスク情報には、タスク名、優先度、所要時間（時間単位）、および期限が含まれます。具体的には以下の機能が必要です: 1. スプレッドシートからタスク情報を取得する 2. タスクの期限に基づいてスケジュールを計算する 3. 計算されたスケジュールをGoogleカレンダーに追加する この要件を満たす完全なコードを提供してください。

ChatGPTに示されたコードをコピーしてスクリプトエディタに貼り付けます。

![](https://media.loom-app.com/loom/2024/08/29/63d70cc0-17ba-4d81-927d-94078dbc8ff2/original.png?w=640)

プロンプト： Googleスプレッドシートに入力されたタスクのTo-DoリストをGoogleカレンダーに反映するGoogle Apps Script (GAS) を作成してください。 このスクリプトは、スプレッドシートからタスク情報を取得し、優先度と所要時間に基づいてスケジュールを計算し、そのスケジュールをGoogleカレンダーに追加します。タスク情報には、タスク名、優先度、所要時間（時間単位）、および期限が含まれます。具体的には以下の機能が必要です: 1. スプレッドシートからタスク情報を取得する 2. タスクの期限に基づいてスケジュールを計算する 3. 計算されたスケジュールをGoogleカレンダーに追加する この要件を満たす完全なコードを提供してください。

貼り付けたら保存のマークを選択し、実行します。

![](https://media.loom-app.com/loom/2024/08/29/f3a0add4-847c-4324-8301-e79af67b32dc/original.png?w=640)

プロンプト： Googleスプレッドシートに入力されたタスクのTo-DoリストをGoogleカレンダーに反映するGoogle Apps Script (GAS) を作成してください。 このスクリプトは、スプレッドシートからタスク情報を取得し、優先度と所要時間に基づいてスケジュールを計算し、そのスケジュールをGoogleカレンダーに追加します。タスク情報には、タスク名、優先度、所要時間（時間単位）、および期限が含まれます。具体的には以下の機能が必要です: 1. スプレッドシートからタスク情報を取得する 2. タスクの期限に基づいてスケジュールを計算する 3. 計算されたスケジュールをGoogleカレンダーに追加する この要件を満たす完全なコードを提供してください。

[次のページ＞＞  
初めてGASを連携するときの注意点](https://www.lifehacker.jp/article/2409-chatgpt-gas/?page=2)

[1](https://www.lifehacker.jp/article/2409-chatgpt-gas/) [2](https://www.lifehacker.jp/article/2409-chatgpt-gas/?page=2)

新着記事![](https://media.loom-app.com/loom/2025/04/07/fecc4e59-9432-43b4-be11-b9d834024cba/original.png?w=180&h=120)

[

【今日のナンプレ問題 vol.44｜難易度★★】ロジカル思考、試してみる？

2025.05.05

](https://www.lifehacker.jp/article/number_place-44/)

[

![](https://media.loom-app.com/loom/2025/04/30/8f037964-7bd9-4c98-bed3-925063b110e8/original.png?w=180&h=120)

音質に実用性も備わって無敵に。JBLの新作スピーカー2種が想像を超えてきた！【今日のライフハックツール】

2025.05.05

](https://www.lifehacker.jp/article/2505-lht-jbl-flip7-charge6/)[

![](https://media.loom-app.com/loom/2025/05/02/d1240a75-de2d-4013-901e-55a117b99b16/original.png?w=180&h=120)

AI知能爆発…タイムリミットは目前！ オックスフォードからの生存戦略

2025.05.05

](https://www.lifehacker.jp/article/2505-ai-intelligence-explosion/)[

![](https://media.loom-app.com/loom/2025/05/05/1b9647fa-4922-4898-8fba-cb7977f2a3a0/original.png?w=180&h=120)

インストール不要！ 写真編集はこの無料ツールで十分かも？

2025.05.05

](https://www.lifehacker.jp/article/2505mini-photo-editor-is-a-browser-tool-for-quick-image-edits/)[

![](https://media.loom-app.com/loom/2025/04/23/07133d96-0e67-4f63-a8e1-ce3e07041078/original.jpg?w=180&h=120)

アンカー・ジャパンの中の人が厳選！新社会人に刺さる「後悔しない」Ankerグループのアイテム最適解3つ

2025.05.05

](https://www.lifehacker.jp/article/2504-ankerjapan-3selections/)[

![](https://media.loom-app.com/loom/2025/05/01/9f41f6bf-a060-4031-bd51-ce457bdfa2c8/original.png?w=180&h=120)

連休明け、“休みボケ”から即復帰！仕事に勢いをつける3つのテクニック

2025.05.05

](https://www.lifehacker.jp/article/2505-work-efficiency-techniques-matome/)

[記事をもっと見る](https://www.lifehacker.jp/articles/)