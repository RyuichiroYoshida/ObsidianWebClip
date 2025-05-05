---
title: GitHub ActionsからSimpleにDiscordに通知を送る
source: https://zenn.dev/midra_lab/articles/d404404365fef4
author:
  - "[[Zenn]]"
published: 2023-05-23
created: 2025-05-06
description: 
tags:
  - CI-CD
  - Discord
  - 1
  - Tools
  - バックエンド
read: false
---
[![](https://storage.googleapis.com/zenn-user-upload/avatar/c00b4e0b95.jpeg) MidraLab(ミドラボ)](https://zenn.dev/p/midra_lab) [Publicationへの投稿](https://zenn.dev/faq#what-is-publication)

2[tech](https://zenn.dev/tech-or-idea)

## 概要

GitHub Actionsからライブラリなどを使わずにDiscordに通知を送る方法です(curlを使う方法)

## 実装

## Discord Webhookの取得

1. Discordのサーバーの設定からWebhookを作成します。
2. WebhookのURLを取得します。

## GitHub Actionsの設定

1. GitHubのリポジトリのSettingsからSecretsを開きます。
2. New repository secretをクリックします。
3. Nameに `DISCORD_WEBHOOK_URL` を入力し、ValueにDiscord WebhookのURLを入力します。

## GitHub Actionsの設定

1. `.github/workflows` に `discord-notify.yml` を作成します。
2. 以下処理をビルド後に追加記述します

```yml
# 終了後に discordのwebhookで通知を行う
    - name: Notify Discord
      run: |
          curl -H "Content-Type: application/json" \
          -X POST \
          -d '{
            "content": "ビルドが完了しました！https://back-to-back.vercel.app/"
          }' \
          ${{ secrets.DISCORD_WEBHOOK_URL }}
```

2

### Discussion

![](https://static.zenn.studio/images/drawing/discussion.png)

ログインするとコメントできます