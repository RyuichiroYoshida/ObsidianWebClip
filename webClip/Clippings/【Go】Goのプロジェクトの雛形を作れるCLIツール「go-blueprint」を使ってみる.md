---
title: 【Go】Goのプロジェクトの雛形を作れるCLIツール「go-blueprint」を使ってみる
source: https://qiita.com/_ken_/items/316d682b43d61cc5e34b
author:
  - "[[Qiita]]"
published: 2024-06-20
created: 2025-05-06
description: はじめにこんにちは、kenです。お仕事ではGoをよく書いています。最近毎日GitHubのトレンドを見るようにしているのですが、先日いつものようにトレンドをチェックしていたら何やら面白そうなリポジ…
tags:
  - Go
  - 1
  - Tools
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

## はじめに

こんにちは、kenです。お仕事ではGoをよく書いています。

最近毎日GitHubのトレンドを見るようにしているのですが、先日いつものようにトレンドをチェックしていたら何やら面白そうなリポジトリを見つけたので今日はそれについて紹介しようと思います。  
その名も「 **[go-blueprint](https://github.com/Melkeydev/go-blueprint)** 」です。

## go-blueprintとは

> Go Blueprint is a CLI tool that allows users to spin up a Go project with the corresponding structure seamlessly. It also gives the option to integrate with one of the more popular Go frameworks (and the list is growing with new features)!

go-blueprintはリポジトリのREADMEにもあるとおり、 **Goのプロジェクトをシームレスに立ち上げることができるCLIツール** です。この後実際に触ってみる様子をお見せしますが、使用したいwebフレームワークやDBMSを選択するだけであっという間にGoのプロジェクトの雛形が生成されます。イメージとしてはReactの `Create React App` が近いのかなと思います。

`docker-compose.yml` や `Makefile` もその雛形のなかに入っているので最初の面倒な環境構築をスキップできるのが良いところです。

## 早速使ってみる

では早速go-blueprintを使ってGoプロジェクトを作ってみます。  
まずはREADMEに書いている通り、 `go install` でgo-blueprintをインストールします。

```text
$ go install github.com/melkeydev/go-blueprint@latest
```

その後

```text
$ go-blueprint create
```

を実行すると

[![スクリーンショット 2024-06-02 19.02.08.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1649218/e7062326-c86c-8cb2-9a9b-b10e081ca466.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1649218%2Fe7062326-c86c-8cb2-9a9b-b10e081ca466.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=2f7075a375b9112d0ca9a5b1ac73e359)

対話式のCLIが起動しました！  
ここではプロジェクト名と使いたいwebフレームワーク、そしてDBMSを選択します。  
今回はwebフレームワークとして `chi` 、DBMSとして `Postgres` を選択してみました。  
[![スクリーンショット 2024-06-02 19.04.41.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1649218/a732d24a-5036-eab8-bab6-3873ac54c12f.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1649218%2Fa732d24a-5036-eab8-bab6-3873ac54c12f.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=f5812cc857af478507e6220ba7e26786)

すると、指定したプロジェクト名でカレントディレクトリ配下にGoプロジェクトが生成されます。  
今回は `sample-project` という名前のプロジェクトにしたので最後に

```text
Next steps:
* cd into the newly created project with: \`cd sample-project\`
```

という案内が出ていますね。優しい！  
素直に従ってディレクトリを移動し、生成されたディレクトリ群を確認してみます。

```text
go-blueprint % cd sample-project
sample-project % tree -a -I '.git'
.
├── .air.toml
├── .env
├── .gitignore
├── Makefile
├── README.md
├── cmd
│   └── api
│       └── main.go
├── docker-compose.yml
├── go.mod
├── go.sum
├── internal
│   ├── database
│   │   └── database.go
│   └── server
│       ├── routes.go
│       └── server.go
└── tests
    └── handler_test.go

7 directories, 13 files
```

先ほども書きましたが `Makefile` や `docker-compose.yml` も生成されていますね。 [^1]  
Makefileには開発を進めるのに便利なコマンドがいくつか登録されています。

たとえば `make run` を実行するとアプリケーションが起動し、 `localhost:8080` にアクセスするとレスポンスを受け取ることができます。生成したばかりですがすでにサーバーとして機能しています。

[![スクリーンショット 2024-06-02 19.24.55.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1649218/dcea7d14-29ee-5de7-d40a-6b2cb37e3f50.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1649218%2Fdcea7d14-29ee-5de7-d40a-6b2cb37e3f50.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=898a4fe0e782f66d1ff92df5f1b1ba00)  
また `make docker-run` とするとDockerコンテナが立ち上がり、Postgresが使えるようになります。

[![スクリーンショット 2024-06-02 19.26.56.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1649218/0ced6509-0cad-b7a9-b72b-14e81df1aab3.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1649218%2F0ced6509-0cad-b7a9-b72b-14e81df1aab3.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=e3f99a56856f09de72316ca0f8a7e810)  
試しに `tasks` テーブルを作ってレコードを直接INSERTし、アプリケーションのコードに `GET /tasks` が来たらそのtasksテーブルの全レコードを返す実装を加えてみると普通に動いてくれました。

```text
root@4edb061ace5b:/# psql -U melkey -d blueprint
psql (16.3 (Debian 16.3-1.pgdg120+1))
Type "help" for help.

blueprint=# CREATE TABLE tasks (
    task_id SERIAL PRIMARY KEY,
    task_name TEXT NOT NULL,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_completed BOOLEAN DEFAULT FALSE
);
CREATE TABLE
blueprint=# INSERT INTO tasks (task_name, description, is_completed) VALUES
('Buy groceries', 'Milk, Eggs, Bread, Butter', FALSE),
('Write report', 'Finish the quarterly report by Friday', FALSE),
('Call plumber', 'Fix the leaking sink in the kitchen', TRUE),
('Schedule meeting', 'Set up a meeting with the project team', FALSE),
('Pay bills', 'Pay electricity and water bills', FALSE);
INSERT 0 5
```

[![スクリーンショット 2024-06-02 19.53.26.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1649218/4aaa28bc-c82f-e9b9-0ead-467b31ef294c.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1649218%2F4aaa28bc-c82f-e9b9-0ead-467b31ef294c.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=640ae47de37cdbfa4b1f1a3dea8bfd9e)

データベースとの接続を確立する部分がすでに生成された雛形に含まれており、また `docker-compose.yml` にPostgresのコンテナの設定がすでに含まれているのでアプリケーションのコードには全レコードを取得するための `SELECT` 文を書くだけで済みました。楽すぎる...！

さらに `make run` の代わりに `make watch` を実行するとGoのホットリロードツールである [air](https://github.com/air-verse/air) が起動するので、快適に開発を行うことができます。このairの設定ファイルである `air.toml` も先程の雛形に含まれているので追加で設定する必要はありません。至れり尽くせりですね！  
[![スクリーンショット 2024-06-02 19.56.22.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1649218/6ad141eb-b0f0-aaf4-b5a7-4f0f5d27e79e.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1649218%2F6ad141eb-b0f0-aaf4-b5a7-4f0f5d27e79e.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=5000b5a7873546074b7e6a64fd5cd3d9)

## advancedオプションも使ってみる

これで `go-blueprint create` で生成された雛形については大方説明できたかなと思いますが、このgo-blueprintには `advanced` オプションがあるのでそちらも触っていきたいと思います。  
先ほどの雛形を生成するコマンドに `--advanced` を付け加えて実行すると、前と同じ3つの質問にプラスして1つ質問が追加されます。

```text
$ go-blueprint create --advanced
```

[![スクリーンショット 2024-06-02 20.09.35.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1649218/5939bcf9-6aa8-ee8b-391f-74b288f75baf.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1649218%2F5939bcf9-6aa8-ee8b-391f-74b288f75baf.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=8c25a06cc95b2e3308a93940a17436e2)

ここで聞かれているのは

- Templを使ったHTMXをサポートするか
- GitHub Actionsを使ったCI/CDワークフローの設定も追加するか
- Websocketをサポートするか

です。今回は試しに「Templを使ったHTMX」のみにチェックを入れてみます。

ちなみに [templ](https://templ.guide/) とはGoでHTMLを書くためのツールです。  
`.templ` という拡張子のファイルにGoとHTMLをミックスしたような独自の記法に従ってコーディングをし、 `templ generate` を実行すると`.templ` のファイルからGo言語のソースコードファイルが生成されます。その中にはHTMLをレンダリングするための関数が含まれており、それを使えばサーバーから動的に生成されたHTMLを返すことが可能になります。

生成されたディレクトリに移動し、初回なのでtemplをインストールします。またtemplのファイルを生成するために `templ generate` も実行します。

```text
go-blueprint % cd sample-project-advanced
sample-project-advanced % go install github.com/a-h/templ/cmd/templ@latest
go: downloading github.com/natefinch/atomic v1.0.1
go: downloading github.com/a-h/protocol v0.0.0-20230224160810-b4eec67c1c22
go: downloading github.com/cenkalti/backoff/v4 v4.3.0
go: downloading github.com/cli/browser v1.3.0
go: downloading go.lsp.dev/jsonrpc2 v0.10.0
go: downloading github.com/a-h/parse v0.0.0-20240121214402-3caf7543159a
go: downloading go.uber.org/zap v1.27.0
go: downloading golang.org/x/mod v0.17.0
go: downloading github.com/andybalholm/brotli v1.1.0
go: downloading go.lsp.dev/uri v0.3.0
go: downloading golang.org/x/sys v0.19.0
go: downloading go.lsp.dev/pkg v0.0.0-20210717090340-384b27a52fb2
go: downloading github.com/segmentio/encoding v0.4.0
go: downloading github.com/segmentio/asm v1.2.0
sample-project-advanced % templ generate
(✓) Complete [ updates=2 duration=2.385834ms ]
```

ここまでを実行した状態で `localhost:8080/web` にアクセスすると、下のGIF画像のようなサイトが表示されます。

[![web.gif](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1649218%2F85806d11-970f-f2f0-70c5-c79d42f219ec.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=6b6bfd747ec342bed0565f57be6e0949)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1649218%2F85806d11-970f-f2f0-70c5-c79d42f219ec.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=6b6bfd747ec342bed0565f57be6e0949)

Helloに続いてInputに入力した文字列が続けて表示されるだけの単純なものですが、これを参考にして先ほどつくったtasksテーブルにWEB上からタスクをINSERTする実装を行っていきます。（TemplやHTMXを触るのは初めてなので、これから先のコードで拙い部分があったらごめんなさい）

まずはモデルを表す `Task` とタスク追加のリクエストを表す `CreateTaskRequest` をtasks.goに定義します。

tasks.go

```go
package types

import "github.com/google/uuid"

type Task struct {
    ID          uuid.UUID \`json:"id"\`
    Name        string    \`json:"name"\`
    Description string    \`json:"description"\`
    CreatedAt   string    \`json:"created_at"\`
    IsCompleted bool      \`json:"is_completed"\`
}

type CreateTaskRequest struct {
    TaskName    string \`json:"name"\`
    Description string \`json:"description"\`
    IsCompleted bool   \`json:"is_completed"\`
}
```

次に動的なHTMLを生成するために必要な`.templ` のコードを書きます。  
ここではタスクを追加するためのフォーム欄と、上で定義した `Task` のスライスを受取りそれを表形式で表示するためのコンポーネントを記述しています。

tasks.templ

```templ
package web

import "sample-project-advanced/types"

templ CreateTaskForm(tasks []types.Task) {
    @Base() {
        <h1>Create Task Form</h1>
        <form action="/tasks" method="POST">
            <input id="name" name="name" type="text"/>
            <input id="description" name="description" type="text"/>
            <input id="is_completed" name="is_completed" type="checkbox"/>
            <button type="submit">Create!</button>
        </form>
        @TaskList(tasks)
    }
}

templ TaskList(tasks []types.Task) {
  <style>
    table {
      margin: 16px 0px;
      width: 100%;
      border-collapse: collapse;
    }
    th, td {
      border: 1px solid black;
      padding: 8px;
      text-align: left;
    }
    th {
      background-color: #f2f2f2;
    }
  </style>

  <table>
    <thead>
      <tr>
        <th>Completed</th>
        <th>Name</th>
        <th>Description</th>
      </tr>
    </thead>
    <tbody>
      for _, task := range tasks {
        <tr>
          <td>
            if task.IsCompleted {
              <input type="checkbox" checked disabled/>
            } else {
              <input type="checkbox" disabled/>
            }
          </td>
          <td>{ task.Name }</td>
          <td>{ task.Description }</td>
        </tr>
      }
    </tbody>
  </table>
}
```

最後にアプリケーションサーバー側の実装をしていきます。  
ハンドラーに `HandleTasks` と `CreateTaskHandler` を追加し、 `HandleTasks` では先程の `tasks.templ` からHTMLを生成しクライアントに返す実装を、 `CreateTaskHandler` ではリクエストに受け取った内容でTaskをINSERTする実装をしていきます。  
また `GetTasks` ではTasksテーブルのすべてのレコードを取得する処理を書いており、これを使って取得したTasksのスライスを `HandleTasks` 内で `CreateTaskForm` の引数として渡しています。

```go
package server

import (
    "encoding/json"
    "log"
    "net/http"

    "sample-project-advanced/cmd/web"
    "sample-project-advanced/types"

    "github.com/a-h/templ"
    "github.com/go-chi/chi/v5"
    "github.com/go-chi/chi/v5/middleware"
    "github.com/google/uuid"
)

func (s *Server) RegisterRoutes() http.Handler {
    r := chi.NewRouter()
    r.Use(middleware.Logger)

    r.Get("/", s.HelloWorldHandler)

    r.Get("/health", s.healthHandler)

    fileServer := http.FileServer(http.FS(web.Files))
    r.Handle("/assets/*", fileServer)
    r.Get("/web", templ.Handler(web.HelloForm()).ServeHTTP)
    r.Post("/hello", web.HelloWebHandler)

    // my original Handler
    r.Get("/web/tasks", s.HandleTasks) // ←追加
    r.Post("/tasks", s.CreateTaskHandler) // ←追加
    return r
}

func (s *Server) HelloWorldHandler(w http.ResponseWriter, r *http.Request) {
  // 生成された雛形に存在するものなので省略
}

func (s *Server) healthHandler(w http.ResponseWriter, r *http.Request) {
  // 生成された雛形に存在するものなので省略
}

func (s *Server) CreateTaskHandler(w http.ResponseWriter, r *http.Request) {
    err := r.ParseForm()
    if err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    var req = types.CreateTaskRequest{
        TaskName:    r.FormValue("name"),
        Description: r.FormValue("description"),
        IsCompleted: r.FormValue("is_completed") == "on",
    }

    id := uuid.New()

    query := "INSERT INTO tasks (id, name, description, is_completed) VALUES ($1, $2, $3, $4)"
    _, err = s.db.Exec(query, id, req.TaskName, req.Description, req.IsCompleted)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    s.HandleTasks(w, r)
}

func (s *Server) HandleTasks(w http.ResponseWriter, r *http.Request) {
    tasks, err := s.GetTasks()
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    web.CreateTaskForm(tasks).Render(r.Context(), w)
}

func (s *Server) GetTasks() ([]types.Task, error) {
    rows, err := s.db.Query("SELECT id, name, description, created_at, is_completed FROM tasks")
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    var tasks []types.Task

    for rows.Next() {
        var task types.Task
        err := rows.Scan(&task.ID, &task.Name, &task.Description, &task.CreatedAt, &task.IsCompleted)
        if err != nil {
            return nil, err
        }
        tasks = append(tasks, task)
    }

    if err = rows.Err(); err != nil {
        return nil, err
    }

    return tasks, nil

}
```

ここまで書けたら `Task` を追加するフォームが完成しているはずです。  
templの扱いに習熟する必要はありますが、サクッと動的なHTMLを返す実装もできました。ヤッター！  
[![web.gif](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1649218%2Fb8f44faf-df31-d4c7-80a7-c5d712ea86f2.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=83afff0948d38c6efafe9036830daabd)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1649218%2Fb8f44faf-df31-d4c7-80a7-c5d712ea86f2.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=83afff0948d38c6efafe9036830daabd)

## さいごに

今回記事内で作成したプロジェクトは下のGitHubから確認していただけます。

実際に使ってみた感想ですが、ちょっとした実装をしたいときには面倒な環境設定が不要なため便利かなと思いました。ただ良くも悪くも最初に生成されたプロジェクトの構造に縛られてしまうところはあるなとも感じたので、自分の理想のプロジェクトの形がある人にとっては使いづらいかもしれません。興味がある方はぜひお手元で動かしてみてください。

ここまで読んでいただきありがとうございました、間違いなどありましたらコメントにてご指摘ください。

[1](https://qiita.com/_ken_/items/#comments)

[38](https://qiita.com/_ken_/items/316d682b43d61cc5e34b/likers)

29

[^1]: ちなみに [go-blueprint.dev](https://go-blueprint.dev/) というサイトで、質問の回答によってどう生成されるファイルが変わるかをWEB上でシミュレートすることもできます。