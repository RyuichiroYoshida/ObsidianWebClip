---
title: GORMを使ってGoでデータベース操作の説明
source: https://qiita.com/atsutama/items/68773112208c7fc05069
author:
  - "[[Qiita]]"
published: 2023-04-18
created: 2025-05-06
description: はじめにGORM（Go Object-Relational Mapping）は、Go言語で開発されたオープンソースのORM（Object-Relational Mapping）ライブラリです。デー…
tags:
  - Go
  - 1
  - バックエンド
  - "#データベース"
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から1年以上が経過しています。

[@atsutama](https://qiita.com/atsutama)

投稿日

### はじめに

`GORM` （Go Object-Relational Mapping）は、Go言語で開発されたオープンソースの `ORM` （Object-Relational Mapping）ライブラリです。データベースとGo言語の構造体をマッピングすることで、データベース操作を簡単かつ効率的に行うことができます。この記事では、 `GORM` の基本的な使い方を紹介し、一般的なデータベース操作のサンプルコードを提供します。

### GORMのセットアップ

まずは、 `GORM` をプロジェクトにインポートします。ターミナルで以下のコマンドを実行してください。

```go
go get -u gorm.io/gorm
go get -u gorm.io/driver/mysql
```

次に、 `GORM` とデータベースを接続するための設定を行います。今回は `MySQL` を使用しますが、 `GORM` は `SQLite` 、 `PostgreSQL` 、 `SQL Server` など、他のデータベースエンジンもサポートしています。

```go
package main

import (
    "fmt"
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
)

type User struct {
    ID    uint   \`gorm:"primaryKey"\`
    Name  string
    Email string \`gorm:"uniqueIndex"\`
}

func main() {
    dsn := "username:password@tcp(localhost:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
    if err != nil {
        panic("failed to connect database")
    }

    // テーブルのマイグレーション
    db.AutoMigrate(&User{})

    // GORMを使ったデータベース操作をここに記述
}
```

### CRUDとは？

CRUDは、データベース操作の基本的な機能を表す英語の頭文字を並べたものです。具体的には、以下の4つの操作を指します。

1. Create（作成）：データベースに新しいレコードを挿入する操作です。
2. Read（読み取り）：データベースから特定のレコードを検索・取得する操作です。
3. Update（更新）：データベースの既存のレコードを変更・修正する操作です。
4. Delete（削除）：データベースから特定のレコードを削除する操作です。

CRUD操作は、ほとんどのアプリケーションやシステムでデータベースを扱う際の基本的な機能です。そのため、データベースを操作するプログラミング言語やライブラリには、通常CRUD操作をサポートする機能が提供されています。GORMも、Go言語でデータベース操作を行う際にCRUD操作を簡単に実現できるように設計されています。

### 基本的なCRUD操作

`GORM` を使ってデータベース操作を行う方法を説明するために、以下のようなUser構造体を例にします。

```go
type User struct {
    ID    uint   \`gorm:"primaryKey"\`
    Name  string
    Email string \`gorm:"uniqueIndex"\`
}
```

1. Create（作成）  
	データベースに新しいレコードを挿入するには、 `Create` メソッドを使います。

```go
user := User{Name: "John Doe", Email: "john@example.com"}
result := db.Create(&user) // ポインタを渡す
if result.Error != nil {
    panic("failed to insert record")
}
```

1. Read（読み取り）  
	データベースからレコードを検索し、構造体にマッピングするには、 `First` 、 `Find` 、 `Where` などのメソッドを使います。

```go
var user User

// 最初のレコードを取得
db.First(&user)

// 条件を指定して検索
db.Where("name = ?", "John Doe").Find(&user)

// 複数のレコードを取得
var users []User
db.Find(&users)
```

1. Update（更新）  
	データベースのレコードを更新するには、 `Save` 、 `Update` 、 `Updates` メソッドを使います。

```go
// Saveメソッドで更新
user.Name = "Jane Doe"
db.Save(&user)

// Updateメソッドで特定のカラムを更新
db.Model(&user).Update("name", "John Doe")

// Updatesメソッドで複数のカラムを更新
db.Model(&user).Updates(map[string]interface{}{"name": "John Doe", "email": "john@example.com"})
```

1. Delete（削除）  
	データベースからレコードを削除するには、 `Delete` メソッドを使います。

```go
db.Delete(&user)
```

##### 特定の条件に一致するレコードを削除する：

```go
db.Where("email = ?", "john@example.com").Delete(&User{})
```

##### 複数の条件を指定してレコードを削除する：

```go
db.Where("email = ? AND name = ?", "john@example.com", "John Doe").Delete(&User{})
```

##### 主キー（Primary Key）を使ってレコードを削除する：

```go
db.Delete(&User{}, 1) // 1は主キーの値
```

必要に応じて、これらのコードを組み合わせて使用することができます。  
ただし、削除操作はデータベースからレコードが永久に削除されるため、注意して使用してください。

### 高度な検索と連鎖的な操作

`GORM` では、検索条件を組み合わせたり、連鎖的な操作を行うことができます。

```go
// WhereとOrを使った検索条件の組み合わせ
db.Where("name = ?", "John Doe").Or("email = ?", "john@example.com").Find(&users)

// ページング処理を行う
db.Limit(10).Offset(20).Find(&users)

// レコードの並び順を指定
db.Order("created_at desc").Find(&users)

// 関連レコードの事前読み込み
type Post struct {
    ID     uint
    Title  string
    UserID uint
}
db.Preload("Posts").Find(&users)
```

##### 特定のカラムを指定して、重複しないレコードのみ取得する：

```go
var users []User
db.Distinct("email").Find(&users)
```

##### テーブルを結合して検索する：

```go
type Profile struct {
    ID     uint
    UserID uint
    City   string
}

// UserテーブルとProfileテーブルを結合し、特定の都市のユーザーを検索
db.Joins("INNER JOIN profiles ON users.id = profiles.user_id").Where("profiles.city = ?", "Tokyo").Find(&users)
```

##### グループ化して集計関数を使用する：

```go
type Result struct {
    City  string
    Count int
}

var results []Result

// プロフィールの都市ごとにユーザー数をカウント
db.Model(&Profile{}).Select("city, COUNT(*) as count").Group("city").Scan(&results)
```

##### ソート（Order）を指定してレコードを取得する：

```go
var users []User

// 名前（Name）で昇順にソートしてレコードを取得
db.Order("name").Find(&users)

// 名前（Name）で降順にソートしてレコードを取得
db.Order("name DESC").Find(&users)
```

##### レコードをページングして取得する：

```go
var users []User
var count int64
pageSize := 10
pageNumber := 1

// 総レコード数を取得
db.Model(&User{}).Count(&count)

// 指定したページ数に応じてレコードを取得
db.Limit(pageSize).Offset((pageNumber - 1) * pageSize).Find(&users)
```

##### 検索条件を複数のフィールドに適用する：

```go
var users []User
searchQuery := "Doe"

// 検索クエリが名前（Name）またはメール（Email）に含まれているレコードを取得
db.Where("name LIKE ? OR email LIKE ?", "%"+searchQuery+"%", "%"+searchQuery+"%").Find(&users)
```

##### サブクエリを使用する：

```go
var usersWithMaxID []User

// IDが最大のユーザーを取得
db.Where("id = ?", db.Table("users").Select("MAX(id)")).Find(&usersWithMaxID)
```

##### カウント（Count）を使用して条件に一致するレコード数を取得する：

```go
var userCount int64
searchQuery := "Doe"

// 検索クエリが名前（Name）に含まれているレコード数をカウント
db.Model(&User{}).Where("name LIKE ?", "%"+searchQuery+"%").Count(&userCount)
```

##### Not（否定）を使用して条件に一致しないレコードを取得する：

```go
var users []User
excludedName := "John Doe"

// 名前（Name）が除外されたレコードを取得
db.Not("name = ?", excludedName).Find(&users)
```

##### Preload（事前読み込み）を使用してリレーションシップを自動的に取得する：

```go
type Order struct {
    ID     uint
    UserID uint
    Total  float64
}

type UserWithOrders struct {
    User
    Orders []Order \`gorm:"foreignKey:UserID"\`
}

var usersWithOrders []UserWithOrders

// ユーザーと関連するオーダーを事前に読み込む
db.Preload("Orders").Find(&usersWithOrders)
```

##### 検索条件を動的に指定する：

```go
var users []User
nameFilter := "John Doe"
emailFilter := "john@example.com"

query := db
if nameFilter != "" {
    query = query.Where("name = ?", nameFilter)
}
if emailFilter != "" {
    query = query.Where("email = ?", emailFilter)
}
query.Find(&users)
```

##### カスタムスコープを作成して、クエリの再利用を行う：

```go
func ActiveUsers(db *gorm.DB) *gorm.DB {
    return db.Where("status = ?", "active")
}

var activeUsers []User
db.Scopes(ActiveUsers).Find(&activeUsers)
```

##### フックを使用して、モデルの操作前後で追加の処理を実行する：

```go
func (u *User) BeforeCreate(tx *gorm.DB) (err error) {
    fmt.Println("ユーザーを作成する前の処理")
    return
}

func (u *User) AfterCreate(tx *gorm.DB) (err error) {
    fmt.Println("ユーザーを作成した後の処理")
    return
}
```

##### テーブルに別名を付けてクエリを実行する：

```go
var users []User
db.Table("users AS u").Select("u.name, u.email").Where("u.email = ?", "john@example.com").Find(&users)
```

##### モデルとカラムのエイリアスを使用する：

```go
type AliasUser struct {
    UserName string \`gorm:"column:name"\`
    UserEmail string \`gorm:"column:email"\`
}

var aliasUsers []AliasUser
db.Table("users").Select("name AS user_name, email AS user_email").Find(&aliasUsers)
```

##### トランザクションを使用して、一連のデータベース操作を管理する：

```go
err := db.Transaction(func(tx *gorm.DB) error {
    user := User{Name: "John Doe", Email: "john@example.com"}
    if err := tx.Create(&user).Error; err != nil {
        return err
    }

    order := Order{UserID: user.ID, Total: 100}
    if err := tx.Create(&order).Error; err != nil {
        return err
    }

    return nil
})
```

##### FindInBatches（バッチ検索）を使用して、大量のレコードを効率的に処理する：

```go
db.FindInBatches(&users, 100, func(tx *gorm.DB, batch int) error {
    for _, user := range users {
        fmt.Printf("バッチ %v でユーザーを処理: %v\n", batch, user.Name)
    }
    return nil
})
```

##### モデルが他のテーブルにリレーションを持っている場合に、カスケード削除を実行する：

```go
db.SetupJoinTable(&User{}, "Orders", &Order{})
db.Where("id = ?", userID).Delete(&
```

##### リレーション先のテーブルを使って検索条件を指定する：

```go
type Company struct {
    ID   uint
    Name string
}

type Employee struct {
    ID        uint
    Name      string
    CompanyID uint
    Company   Company
}

var employees []Employee

// リレーション先の会社名を条件に従業員を検索
db.Joins("Company").Where("companies.name = ?", "OpenAI").Find(&employees)
```

##### FirstOrInit（最初のレコードまたは初期値）とFirstOrCreate（最初のレコードまたは作成）を使用して、レコードを取得または作成する：

```go
var user User

// 条件に一致する最初のレコードを取得するか、初期値を設定する
db.Where("name = ?", "John Doe").FirstOrInit(&user)
if user.ID == 0 {
    fmt.Println("ユーザーが存在しないため、初期値が設定されました")
}

// 条件に一致する最初のレコードを取得するか、新しいレコードを作成する
db.Where("name = ?", "Jane Doe").Attrs(User{Email: "jane@example.com"}).FirstOrCreate(&user)
```

### GORMを使って複数レコードを一度に更新する方法

```go
package main

import (
    "fmt"
    "gorm.io/driver/sqlite"
    "gorm.io/gorm"
)

type User struct {
    ID    uint
    Name  string
    Email string
    Age   int
}

func main() {
    // SQLiteデータベースに接続
    db, err := gorm.Open(sqlite.Open("test.db"), &gorm.Config{})
    if err != nil {
        panic("データベース接続に失敗しました")
    }

    // テーブルを作成
    db.AutoMigrate(&User{})

    // 複数のレコードを更新する条件を設定
    updateCondition := "age >= ?"
    updateAge := 30

    // 複数のレコードを更新
    db.Model(&User{}).Where(updateCondition, updateAge).Updates(map[string]interface{}{
        "email": "updated_email@example.com",
    })

    // 更新後のレコードを表示
    var users []User
    db.Find(&users)
    for _, user := range users {
        fmt.Printf("ID: %d, Name: %s, Email: %s, Age: %d\n", user.ID, user.Name, user.Email, user.Age)
    }
}
```

### まとめ

この記事では、 `GORM` を使ってGoでデータベース操作を行う方法について解説しました。 `GORM` は、データベースとGo言語の構造体をマッピングすることで、データベース操作を簡単かつ効率的に行うことができます。一般的なデータベース操作だけでなく、高度な検索や連鎖的な操作もサポートしており、データベース操作の多くのニーズに対応できる強力なツールです。

---

ORMとは？

> ORM（Object-Relational Mapping）とは、オブジェクト指向プログラミング言語とリレーショナルデータベースの間のデータ変換を行うプログラミング技術です。リレーショナルデータベースは、行と列を持つテーブル構造でデータを表現しますが、オブジェクト指向プログラミングでは、データと振る舞いを持つオブジェクト（クラスや構造体）を使ってデータを表現します。これら二つのデータ表現方法は異なるため、データの変換が必要になります。

> ORMは、オブジェクト指向プログラミング言語のオブジェクト（クラスや構造体）とリレーショナルデータベースのテーブルとの間で、自動的にデータのマッピング（変換）を行うことで、データベース操作を簡単かつ効率的に行えるようにする技術です。ORMを使うことで、プログラマは直接SQLクエリを書くことなく、プログラム内でオブジェクトを操作するだけでデータベースとのやりとりができるようになります。これにより、コードの可読性や保守性が向上し、開発の生産性が高まります。

> GORMは、Go言語で開発されたORMライブラリの一つで、データベースとGo言語の構造体の間でデータのマッピングを行い、CRUD操作などを簡単に実現できるように設計されています。

[0](https://qiita.com/atsutama/items/#comments)

[13](https://qiita.com/atsutama/items/68773112208c7fc05069/likers)

14