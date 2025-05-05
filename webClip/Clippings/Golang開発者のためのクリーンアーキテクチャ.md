---
title: Golang開発者のためのクリーンアーキテクチャ
source: https://zenn.dev/edash_tech_blog/articles/b4629f9cd73240
author:
  - "[[Zenn]]"
published: 2024-07-01
created: 2025-05-06
description: 
tags:
  - Go
  - Tech
  - バックエンド
  - 設計
read: false
---
119

3[tech](https://zenn.dev/tech-or-idea)

## はじめに

クリーンアーキテクチャは、ソフトウェア設計の分野で非常に重要な概念です。

しかし、その理解は容易ではなく、明確な正解が存在するわけではありません。

多くの人が異なる解釈を持ち、他の設計思想と混在していることもあります。

この記事では、自分なりの視点からクリーンアーキテクチャを解釈し、その整理した内容を共有します。

このアーキテクチャの目的は、システムの各層を独立させ、変更に強く、テストしやすい設計を実現することです。

この記事では、クリーンアーキテクチャの基本概念、Golangでの実装方法、およびディレクトリ構成について詳しく説明します。

なお、この記事では個人的な見解を述べており、必ずしも正解を書いているわけではありません。もし誤りがあれば、ぜひご指摘いただけると幸いです。

## クリーンアーキテクチャの基本概念

クリーンアーキテクチャの元となったのは、ロバート・C・マーチン（通称「アンクルボブ」）による **The Clean Code Blog** です。

特に以下の図は、クリーンアーキテクチャの依存関係を示すものとしてよく知られています。

![](https://res.cloudinary.com/zenn/image/fetch/s--hrA1jqEs--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://blog.cleancoder.com/uncle-bob/images/2012-08-13-the-clean-architecture/CleanArchitecture.jpg)

引用元: [The Clean Code Blog](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)

この図が示す通り、クリーンアーキテクチャは複数の同心円で表され、それぞれが異なるレイヤーを表しています。

各レイヤーは、内側のレイヤーに依存し、外側のレイヤーからは独立しています。

## インフラストラクチャ（Infrastructure）

インフラストラクチャ層は、外部システムやフレームワークとの連携を扱います。

**例：MySQL接続**

```
package infrastructure

func NewDB() *sql.DB {
    db, err := sql.Open("mysql", "user:password@tcp(127.0.0.1:3306)/dbname")
    if err != nil {
        log.Fatal(err)
    }
    if err := db.Ping(); err != nil {
        log.Fatal(err)
    }
    return db
}
```

## コントローラー(Controller)

コントローラー層は、外部からのリクエストを受け取り、適切なユースケースを呼び出し、レスポンスを生成する役割があります。

この層では、入力値のバリデーションやエラーハンドリングも行います。

コントローラーにはビジネスロジックを含まず、処理はすべてUsecaseに移譲されます。

**例：Echoフレームワークを使用したコントローラーの実装**

```
package controllers

type UserController struct {
    updateUserNameInteractor usecases.IUpdateUserNameInteractor
    userPresenter            presenters.IUserPresenter
    errorPresenter           presenters.IErrorPresenter
}

func NewUserController(
    updateUserNameInteractor usecases.IUpdateUserNameInteractor,
    userPresenter presenters.IUserPresenter,
    errorPresenter presenters.IErrorPresenter,
) *UserController {
    return &UserController{
        updateUserNameInteractor: updateUserNameInteractor,
        userPresenter:            userPresenter,
        errorPresenter:           errorPresenter,
    }
}

func (c *UserController) UpdateUserName(e echo.Context, userID string) error {
    req := api.UpdateUserNameRequest{}
    if err := e.Bind(&req); err != nil {
        return c.errorPresenter.PresentBadRequest(e, "invalid request")
    }

    input := &input.UpdateUserNameInput{
        UserID:  userID,
        NewName: req.Name,
    }
    ctx := e.Request().Context()

    output, err := c.updateUserNameInteractor.Execute(ctx, input)
    if err != nil {
        return c.errorPresenter.PresentInternalServerError(e, err)
    }

    return c.userPresenter.PresentUpdateUserName(e, output)
}
```

## プレゼンター（Presenter）

プレゼンター層は、ユースケースの出力を受け取り、それをユーザーインターフェース（UI）に適した形式に変換する役割を果たします。

この役割には、データのフォーマット変更やエラーメッセージの設定など、レスポンスの生成が含まれます。

**例：正常系のレスポンス生成**

```
package presenters

type IUserPresenter interface {
    PresentUpdateUserName(c echo.Context, output *output.UpdateUserNameOutput) error
}

type UserPresenter struct{}

func NewUserPresenter() IUserPresenter {
    return &UserPresenter{}
}

func (p *UserPresenter) PresentUpdateUserName(c echo.Context, output *output.UpdateUserNameOutput) error {
    response := api.UpdateUserNameResponse{
        ID:   output.User.GetID(),
        Name: output.User.GetName(),
    }

    return c.JSON(http.StatusOK, response)
}
```

  

**例：共通のエラーレスポンス生成**

```
package presenters

type IErrorPresenter interface {
    PresentBadRequest(c echo.Context, message string) error
    PresentInternalServerError(c echo.Context, err error) error
}

type ErrorPresenter struct{}

func NewErrorPresenter() IErrorPresenter {
    return &ErrorPresenter{}
}

func (p *ErrorPresenter) PresentBadRequest(c echo.Context, message string) error {
    response := struct {
        Error string \`json:"error"\`
    }{
        Error: message,
    }

    return c.JSON(http.StatusBadRequest, response)
}

func (p *ErrorPresenter) PresentInternalServerError(c echo.Context, err error) error {
    response := struct {
        Error string \`json:"error"\`
    }{
        Error: err.Error(),
    }

    return c.JSON(http.StatusInternalServerError, response)
}
```

## ユースケース（Use Cases）

ユースケースは、アプリケーション固有のビジネスルールを定義します。

これはシステムがどのように動作するかを決定するもので、ユーザーや他のシステムがどのようにシステムと対話するかを記述します。

### Input PortとOutput Port

Input Port と Output Port は、ユースケースの境界を定義するDTO（Data Transfer Object）です。

Input Portはユースケースに必要なデータを提供し、Output Portはユースケースの結果を外部に返します。

### Publicメソッドが1つのみになるように設計する

アンクルボブ氏がクリーンアーキテクチャを提唱する際に、明確に「複数のPublicメソッドを持つべきではない」と述べているわけではありません。

しかし、単一責任の原則を遵守するために、UsecaseがPublicメソッドを1つだけ持つように設計することが推奨されます。

Usecaseに複数のPublicメソッドが存在すると責任範囲が広がりすぎ、ビジネスロジックが散逸しやすくなります。

1つのPublicメソッドにすることで、特定のビジネスロジックに集中でき、修正の影響範囲を最小限に抑えることができます。

**例：ユーザー登録**

```
package usecases

type IUpdateUserNameInteractor interface {
    Execute(ctx context.Context, r *input.UpdateUserNameInput) (*output.UpdateUserNameOutput, error)
}

type UpdateUserNameInteractor struct {
    userRepository repositories.IUserRepository
}

func NewUpdateUserNameInteractor(
    userRepository repositories.IUserRepository,
) IUpdateUserNameInteractor {
    return &UpdateUserNameInteractor{
        userRepository: userRepository,
    }
}

func (i *UpdateUserNameInteractor) Execute(ctx context.Context, input *input.UpdateUserNameInput) (*output.UpdateUserNameOutput, error) {
    user, err := i.userRepository.GetUser(ctx, input.UserID)
    if err != nil {
        return nil, err
    }

    user.SetName(input.NewName)

    err = i.userRepository.UpdateUser(ctx, user)
    if err != nil {
        return nil, err
    }

    output := &output.UpdateUserNameOutput{
        User: user,
    }

    return output, nil
}
```

## エンティティ（Entities）

エンティティは、ビジネスルールやオブジェクトの集合を表します。

これらはシステムの中で最も中心的な存在であり、システムの核となるビジネスロジックを含みます。

### ドメインモデルとデータモデルを明確に区別する

データモデルは、データベースのテーブル設計に対応し、永続化されたデータを表します。

ドメインモデルは、ビジネスロジックやルールを反映したものです。

例えば、ユーザーの年齢を扱う場合、年齢は生年月日から計算される派生データであり、ドメインモデルに含まれます。

ドメインモデルは対応するデータモデルがなくても存在することができ、ビジネスロジックを中心に設計されています。

また、 **ドメインモデルは、不用意なアクセスを防ぐためにフィールドをPrivate** にすることを心掛けましょう。

**例：データモデル**

```
package models

type User struct {
    ID         string    \`json:"id"\`
    Name       string    \`json:"name"\`
    Birthdate  time.Time \`json:"birthdate"\`
}
```

**例：ドメインモデル**

```
package entities

type User struct {
    id       string
    name     string
    age      int
}

func NewUser(id int, name string, birthDate time.Time) User {
    return User{
        id:   id,
        name: name,
        age:  calculateAge(birthDate),
    }
}

func calculateAge(birthdate time.Time) int {
    today := time.Now()
    age := today.Year() - birthdate.Year()
    if today.YearDay() < birthdate.YearDay() {
        age--
    }
    return age
}
```

## ドメインサービス(Domain Service)

Domainサービスには、複数のドメインにまたがる共通の振る舞いを格納します。

単一のドメインに収まるロジックは、Entitiesに格納するようにします。

これにより、エンティティやユースケースが過度に複雑化するのを防ぐことができ、データ整合性を保ちます。

**例：ユーザーサービス**

### Serviceはむやみに作成しない

特定のドメインのみに依存し、データを取得するのみの処理の場合は、ユースケースで行い、むやみにService層を作成すべきではありません。

下記のような場面でも、Serviceにメソッドを作成しがちですが、Usecaseで直接処理すべきです。

**例：悪い実装方法**

```
func (s *UserService) GetUserByID(id int) (*entities.User, error) {
    return s.userRepo.FindByID(id)
}
```

### Service内で別ServiceをDIしない

別のサービスをDIすることは、コードの再利用性を高める一方で、依存関係が複雑になりやすく、循環参照の問題が発生しがちです。個人的には避けることを推奨します。

The Clean Code Blogでは、層の数に制限はなく、必要に応じて層を追加しても良いとされています。

もし層を追加する場合、Facadeのような名前が適しているかもしれません。

## リポジトリ(Repository)

**例：ユーザーリポジトリ 抽象**  
domain配下のrepositoriesに格納されています。

```
package repositories

type IUserRepository interface {
    GetUser(ctx context.Context, id string) (*entities.User, error)
    UpdateUser(ctx context.Context, user *entities.User) error
}
```

**例：ユーザーリポジトリ 実装**  
infrastructure配下のrepositoriesに格納されています。

```
package repositories

type UserRepository struct {
    db *sql.DB
}

func NewUserRepository(db *sql.DB) repositories.IUserRepository {
    return &UserRepository{db: db}
}

func (r *UserRepository) GetUser(ctx context.Context, id string) (*entities.User, error) {
    row := r.db.QueryRowContext(ctx, "SELECT id FROM users WHERE id = ?", id)
    user := &models.User{}
    err := row.Scan(&user.ID)
    if err != nil {
        return nil, err
    }

    return entities.NewUser(model.ID, model.Name, model.Email), nil
}

func (r *UserRepository) UpdateUser(ctx context.Context, user *entities.User) error {
    model := user.ToDataModel()
    _, err := r.db.ExecContext(ctx, "UPDATE users SET name = ? WHERE id = ?", model.Name, model.ID)
    return err
}
```

### 依存性逆転の法則(Dependency Inversion Principle)とは

依存性逆転の法則（DIP: Dependency Inversion Principle）は、ソフトウェア設計の基本原則の一つです。

簡単に言えば、「高レベルのモジュールは低レベルのモジュールに依存しないようにする」という考え方です。

具体的には、以下の2つのルールを含みます

1. 高レベルのモジュール（重要なロジックを含む部分）は、低レベルのモジュール（詳細な実装部分）に依存してはいけない。どちらも抽象（インターフェース）に依存するべき。
2. 抽象（インターフェース）は具体的な詳細に依存してはいけない。具体的な詳細が抽象に依存するべき。

#### Repository層での重要性

Repository層では、この依存性逆転の原則が特に重要です。

UsecaseはRepository層に依存しますが、RepositoryはDBの接続をDIしており、Infrastructure層に依存してしまいます。

![](https://res.cloudinary.com/zenn/image/fetch/s--gvgdT_E9--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://github.com/k-takeuchi220/zenn-images/blob/master/DIP1.png%3Fraw%3Dtrue)

この問題を解決するために、Usecaseよりも内側の層にRepository層の抽象を置き、実装をInfrastructure層に配置する方法が取られています。

![](https://res.cloudinary.com/zenn/image/fetch/s--gvzTuLSP--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://github.com/k-takeuchi220/zenn-images/blob/master/DIP2.png%3Fraw%3Dtrue)

これにより、依存方向が一方向になり、問題が解消されます。

また、他の箇所（UsecaseやServiceなど）でも、抽象を配置していますが、これは各層を疎結合にし、テストを容易にする等の目的があります。

## ディレクトリ構成

クリーンアーキテクチャを実装するためのプロジェクト構造は、コードの可読性とメンテナンス性を高めるために重要です。以下は、Golangプロジェクトの一般的なディレクトリ構成の例です。

```
/your_project
├── main.go
├── domain
│   ├── entities
│   │   └── user.go
│   ├── services
│   │   └── user_service.go
│   └── repositories
│       └── user_repository_interface.go
├── usecases
│   ├── input
│   │   └── update_user_name_input.go
│   ├── output
│   │   └── update_user_name_output.go
│   └── update_user_name_interactor.go
├── infrastructure
│   ├── repositories
│   │   └── user_repository.go
│   ├── models
│   │   └── user_model.go
│   ├── db.go
│   └── router.go
├── controllers
│   └── user_controller.go
├── presenters
│   ├── error_presenter.go
│   └── user_presenter.go
├── api
│   ├── openapi.yaml
│   ├── types.go
│   └── server.go
└── go.mod
```

### ディレクトリの役割

- **`main.go`**: プロジェクトのエントリーポイント。アプリケーションの初期化と設定を行います。
- **`domain`**: ビジネスドメインに関するコード。エンティティ、ドメインモデル、ドメインサービス、リポジトリの抽象などが含まれます。
	- **`entities`**: ビジネスルールを表すエンティティ。
	- **`services`**: ドメインサービス。
	- **`repositories`**: リポジトリのインターフェース。
- **`usecases`**: アプリケーション固有のビジネスロジック。
	- **`input`**: ユースケースへ渡す引数型。
	- **`output`**: ユースケースから返却される型。
- **`infrastructure`**: インフラストラクチャ関連のコード。データベース接続やWebルーターの設定が含まれます。
	- **`models`**: データモデル。
	- **`repositories`**: リポジトリの実装。
- **`controllers`**: コントローラー。
- **`api`**: swaggerからの型生成を想定しています。oapi-codegenによって生成されたtypes.go, server.go）を含みます。

### まとめ

クリーンアーキテクチャは、ソフトウェアの設計と開発において多くの利点をもたらします。

特に、変更に強く、テストしやすいシステムを構築するための強力なガイドラインとなります。

この記事を通じて、クリーンアーキテクチャの基本概念とGolangでの実装方法について理解が深まれば幸いです。

  

以下にサンプルを用意しました。ぜひスターをいただけると嬉しいです。

119

3