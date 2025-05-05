---
title: Notion APIを使って実践向け実装
source: https://chigusa-web.com/blog/notion-api/2/#google_vignette
author:
  - "[[チグサウェブ]]"
published: 2023-10-09
created: 2025-05-06
description: NotionのAPIを実装して試してみました。
tags:
  - 1
  - Tools
  - TS
  - Notion
read:
---
[

自分だけのクイズを作成しよう - Quipha

](https://chigusa-web.com/quipha/)

## 実装について

### 概要

APIを使った実装を行う場合、APIにアクセスするだけですので、 どの言語でもcURLコマンドでも可能 です。

仕様に沿ってリクエストを送信するだけですが、SDKを使うと実装時は便利になります。  
公式でJavaScriptのSDKもあります。

[

![](https://opengraph.githubassets.com/643a741f3c1a73297eb0fab2423a3c3275e76411fa3395fa7cfc5f052cbb8fde/makenotion/notion-sdk-js)

](https://github.com/makenotion/notion-sdk-js "GitHub - makenotion/notion-sdk-js: Official Notion JavaScript Client")

非公式でも、各言語向けのSDKを開発している方もいます。

今回はPythonを使って実装しますが、 SDKは使用せずAPIにリクエストを送信 して行います。  
これにより、他の言語でも参考になるかと思います。

cURLでもPHPでも、本記事に記載しているリクエストを参考にすることで同様の操作が可能です。

### トークンについて

インテグレーションを作成した際に表示されたトークンを使用します。  
この トークンは秘密情報 であり、知られてしまうとNotionを操作することができてしまします。

そのためトークンは環境変数に保持し、 ソースにはハードコーディングしないように注意 しましょう。

```
$ export NOTION_KEY=secret_...
```

ソース上では以下のように取得します。

```python
import os

# token
NOTION_TOKEN = os.getenv('NOTION_KEY')
```

以降、NOTION\_TOKENは上記のように取得する前提で説明します。

### 解説について

API仕様につきまして、 本記事執筆時点の情報 になります。  
機能は今後さらに増えていくと思いますので、最新情報は公式ドキュメントをご覧ください。

## 検索

コネクトされているページ、データベース、子ページなど、すべての情報を検索できます。

応答結果は以下のようになり、ページやデータベースの情報などを取得することができます。

```json
{
  "object": "list",
  "results": [
    {
      "object": "page",
      "id": "ページID",
      "created_time": "2023-01-15T11:18:00.000Z",
      "last_edited_time": "2023-01-15T11:21:00.000Z",
      "created_by": {
        "object": "user",
        "id": "XXX"
      },
      "last_edited_by": {
        "object": "user",
        "id": "XXX"
      },
      "cover": null,
      "icon": null,
      "parent": {
        "type": "workspace",
        "workspace": true
      },
      "archived": false,
      "properties": {
        "title": {
          "id": "title",
          "type": "title",
          "title": [
            {
              "type": "text",
              "text": {
                "content": "Notion APIテスト",
                "link": null
              },
              "annotations": {
                "bold": false,
                "italic": false,
                "strikethrough": false,
                "underline": false,
                "code": false,
                "color": "default"
              },
              "plain_text": "Notion APIテスト",
              "href": null
            }
          ]
        }
      },
      "url": "https://www.notion.so/Notion-API-XXX"
    }
  ],
  "next_cursor": null,
  "has_more": false,
  "type": "page_or_database",
  "page_or_database": {}
}
```

コネクトした直後であれば、そのページの情報のみ取得できます。  
上記のページIDは以降で使用しますので保持しておいてください。

## データベース

### データベース作成

コネクトしたページに、データベースを作成しましょう。  
ページIDは先ほど取得したIDを指定します。

```python
url = f"https://api.notion.com/v1/databases/"

headers = {
    "accept": "application/json",
    "Notion-Version": "2022-06-28",
    "Authorization": f"Bearer {NOTION_TOKEN}"
}

json_data = {
    "parent": {
        "type": "page_id",
        "page_id": "ページID"
    },
    "icon": {
        "type": "emoji",
        "emoji": "?"
    },
    "title": [
        {
            "type": "text",
            "text": {
                "content": "テストデータベース",
                "link": None
            }
        }
    ],
    # プロパティ
    "properties": {
        "タイトル": {
            "title": {}
        },
        "備考": {
            "rich_text": {}
        },
        # チェックボックス
        "チェック": {
            "checkbox": {}
        },
        # 単一選択。オプションも作成できる
        "優先度": {
            "select": {
                "options": [
                    {
                        "name": "高",
                        "color": "green"
                    },
                    {
                        "name": "中",
                        "color": "red"
                    },
                    {
                        "name": "低",
                        "color": "yellow"
                    }
                ]
            }
        },
        # 複数選択。オプションも可能
        "カテゴリ": {
            "type": "multi_select",
            "multi_select": {
                "options": [
                    {
                        "name": "カテゴリ1",
                        "color": "blue"
                    },
                    {
                        "name": "カテゴリ2",
                        "color": "gray"
                    }
                ]
            }
        },
    },
    # インラインデータベース
    "is_inline": True
}

response = requests.post(url, json=json_data, headers=headers)
print(response.text)
```

コネクトしたページに、 データベースを追加 することができました。  
複数選択やチェックボックス、セレクトなどのプロパティも追加できます。

![](https://chigusa-web.com/wp-content/uploads/2023/01/2023-01-15_22h10_13-1024x359.png.webp)

応答結果は 作成したデータベースの詳細 が取得できます。  
データベースのIDも取得できますので、控えておきます。

```python
{
  "object": "database",
  "id": "データベースID",
...
```

他にもプロパティは追加可能です。詳しくは公式ドキュメントをご覧ください。

### データベースの情報取得

データベースの情報を取得します。  
データベースIDは先ほど取得したIDを指定します。

応答結果は、データベースの詳細を取得できます。

```json
{
  "object": "database",
  "id": "データベースID",
  "cover": null,
  "icon": {
    "type": "emoji",
    "emoji": "?"
  },
  "created_time": "2023-01-15T13:09:00.000Z",
  "created_by": {
    "object": "user",
    "id": "XXX"
  },
  "last_edited_by": {
    "object": "user",
    "id": "XXX"
  },
  "last_edited_time": "2023-01-15T13:09:00.000Z",
  "title": [
    {
      "type": "text",
      "text": {
        "content": "テストデータベース",
        "link": null
      },
      "annotations": {
        "bold": false,
        "italic": false,
        "strikethrough": false,
        "underline": false,
        "code": false,
        "color": "default"
      },
      "plain_text": "テストデータベース",
      "href": null
    }
  ],
  "description": [],
  "is_inline": true,
  "properties": {
    "優先度": {
      "id": "XXX",
      "name": "優先度",
      "type": "select",
      "select": {
        "options": [
          {
            "id": "XXX",
            "name": "高",
            "color": "green"
          },
          {
            "id": "XXX",
            "name": "中",
            "color": "red"
          },
          {
            "id": "XXX",
            "name": "低",
            "color": "yellow"
          }
        ]
      }
    },
    "備考": {
      "id": "XXX",
      "name": "備考",
      "type": "rich_text",
      "rich_text": {}
    },
    "チェック": {
      "id": "XXX",
      "name": "チェック",
      "type": "checkbox",
      "checkbox": {}
    },
    "カテゴリ": {
      "id": "XXX",
      "name": "カテゴリ",
      "type": "multi_select",
      "multi_select": {
        "options": [
          {
            "id": "XXX",
            "name": "カテゴリ1",
            "color": "blue"
          },
          {
            "id": "XXX",
            "name": "カテゴリ2",
            "color": "gray"
          }
        ]
      }
    },
    "タイトル": {
      "id": "title",
      "name": "タイトル",
      "type": "title",
      "title": {}
    }
  },
  "parent": {
    "type": "page_id",
    "page_id": "XXX"
  },
  "url": "https://www.notion.so/XXX",
  "archived": false
}
```

### データベースのページリスト取得

データベースのページのリストを取得します。

ページがあると、以下のように データベースに追加されているページのリスト を取得できます。

```json
{
  "object": "list",
  "results": [
    {
      "object": "page",
      "id": "XXX",
      "created_time": "2023-01-15T13:10:00.000Z",
      "last_edited_time": "2023-01-15T13:20:00.000Z",
      "created_by": {
        "object": "user",
        "id": "XXX"
      },
      "last_edited_by": {
        "object": "user",
        "id": "XXX"
      },
      "cover": null,
      "icon": null,
      "parent": {
        "type": "database_id",
        "database_id": "XXX"
      },
      "archived": false,
      "properties": {
        "優先度": {
          "id": "XXX",
          "type": "select",
          "select": null
        },
        "備考": {
          "id": "XXX",
          "type": "rich_text",
          "rich_text": [
            {
              "type": "text",
              "text": {
                "content": "備考です",
                "link": null
              },
              "annotations": {
                "bold": false,
                "italic": false,
                "strikethrough": false,
                "underline": false,
                "code": false,
                "color": "default"
              },
              "plain_text": "備考です",
              "href": null
            }
          ]
        },
        "チェック": {
          "id": "XXX",
          "type": "checkbox",
          "checkbox": true
        },
        "カテゴリ": {
          "id": "XXX",
          "type": "multi_select",
          "multi_select": [
            {
              "id": "XXX",
              "name": "カテゴリ1",
              "color": "blue"
            }
          ]
        },
        "タイトル": {
          "id": "title",
          "type": "title",
          "title": [
            {
              "type": "text",
              "text": {
                "content": "ページタイトルです",
                "link": null
              },
              "annotations": {
                "bold": false,
                "italic": false,
                "strikethrough": false,
                "underline": false,
                "code": false,
                "color": "default"
              },
              "plain_text": "ページタイトルです",
              "href": null
            }
          ]
        }
      },
      "url": "https://www.notion.so/XXX"
    }
  ],
  "next_cursor": null,
  "has_more": false,
  "type": "page",
  "page": {}
}
```

データベースに手動でページを追加して確認してみてください。

次のページでは、データベースにページやブロックを追加する実装を解説します。

[1](https://chigusa-web.com/blog/notion-api/) 2 [3](https://chigusa-web.com/blog/notion-api/3/)

タイトルとURLをコピーしました