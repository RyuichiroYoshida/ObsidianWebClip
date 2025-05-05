---
title: 実践テスト駆動開発を読んだのでモックを使ったロンドン学派的なテストをGoとmoqで考える
source: https://zenn.dev/jy8752/articles/cee5f11639ca83
author:
  - "[[Zenn]]"
published: 2024-08-18
created: 2025-05-06
description: 
tags:
  - Go
  - 1
  - QA
  - バックエンド
  - 思想系
read: false
---
37[tech](https://zenn.dev/tech-or-idea)

## はじめに

この記事を読んでいるみなさんは [単体テストの考え方/使い方](https://book.mynavi.jp/ec/products/detail/id=134252) はもう読まれましたでしょうか？わたしは単体テストの考え方を読んだことでテストに対して漠然と思っていたことが言語化されたように感じ大変感銘を受けました。その勢いで以下のようなzenn本も書かせていただきました。

なのでしばらくはモックを極力使わない古典学派的な思想に寄っていたのですがモックを使ったテストの開発現場のレビューにて

> 「テスト対象の依存関係をモックにしたらテストすることがなくなりました。」

というコメントをチラッと見かけてしまい、モヤっとしました。というのもその感覚には覚えがあったからでテスト対象のオブジェクトの依存関係が多く、処理内容がその依存関係を手続き的に呼び出しているだけだったりするとそれら全てをモックやスタブに置き換え、期待する動作を自分で設定することになるのでテストが成功しても「そりゃそうだよな」という気持ちになったことがあります。この **自作自演** のような感覚が上記のコメントのように感じたんじゃないかなと勝手に思ったりします。

そもそも、古典学派のテストを単体テストの考え方/使い方で学んだ一方でロンドン学派のテストについて学べていない。これは **盲目的に古典学派を推すのではなく一度ロンドン学派についてもちゃんと学んだ方がいい** なと思ったところ以下のようなt-wadaさんのPostを見つけました。

ロンドン学派にもバイブルが存在する！！

ということで [実践テスト駆動開発](https://www.shoeisha.co.jp/book/detail/9784798124582) を読んでみたところいろいろ考えることが多かったので実際に手を動かしながら整理したいというのが本記事の内容になります。

一応、実践テスト駆動開発を読んだ感想文はこちらです。

## 対象読者

- Goで普段からテスト書いている人
- モックを使ったテストに難しさを感じている人
- 単体テストの考え方/使い方を読んだ人
- どちらかと言えば古典学派な人
- どちらかと言えばロンドン学派な人

## 使用技術

- go version go1.22.0 darwin/arm64
- [moq](https://github.com/matryer/moq)

モックにはmoqというライブラリを使用しています。moqに関しては以下の記事を読んで最近使ってるのですがいい感じだったので今回も使います。

## 今回作るもの

ある程度処理が多い方が良いかなと思いつつ何も思いつかなかったのでいつもやってるガチャシステム作ります。スキーマは以下のような感じでお金払ってアイテムを抽選するみたいなのを想定。

![](https://storage.googleapis.com/zenn-user-upload/690778123903-20240815.png)

## アプリケーションコードを実装する

```
go mod init go-mock-test-demo
```

とりあえずざっくり以下のように実装しちゃいます。

cmd/gacha/main.go

cmd/gacha/main.go

```
package main

import (
    "database/sql"
    "errors"
    "fmt"
    "log"
    "math/rand/v2"
    "os"
    "strconv"

    _ "github.com/go-sql-driver/mysql"
)

type User struct {
    ID   int64
    Name string
    Coin int64
}

type Item struct {
    ID     int64
    Name   string
    Rare   string
    Weight int
}

type UserItem struct {
    ID     int64
    UserId int64
    ItemId int64
    Count  int
}

const (
    GachaPrice = 10
)

func main() {
    if len(os.Args) < 2 {
        log.Fatal("実行時引数にユーザーのIDを指定してください")
    }

    userIdStr := os.Args[1]
    userId, err := strconv.ParseInt(userIdStr, 10, 64)
    if err != nil {
        log.Fatal("ユーザーIDは整数値で指定してください")
    }

    db, err := sql.Open("mysql", "root:password@tcp(localhost:13306)/app")
    if err != nil {
        log.Fatal(err)
    }

    defer func() {
        _ = db.Close()
    }()

    if err = db.Ping(); err != nil {
        log.Fatal(err)
    }

    // ユーザー情報を取得する
    var user User
    if err = db.QueryRow("SELECT id, name, coin FROM users WHERE id = ?", userId).Scan(&user.ID, &user.Name, &user.Coin); err != nil {
        log.Fatalf("ユーザーの取得に失敗しました err: %v\n", err)
    }

    if user.Coin < GachaPrice {
        log.Fatal("コインが足りません")
    }

    // ガチャを抽選する
    rows, err := db.Query("SELECT id, name, rare, weight FROM items")
    if err != nil {
        log.Fatalf("アイテムの取得に失敗しました err: %v\n", err)
    }
    defer func() {
        _ = rows.Close()
    }()

    columns, err := rows.Columns()
    if err != nil {
        log.Fatal(err)
    }

    items := make([]Item, 0, len(columns))
    weights := make([]int, 0, len(columns))

    for rows.Next() {
        var item Item
        if err := rows.Scan(&item.ID, &item.Name, &item.Rare, &item.Weight); err != nil {
            log.Fatal(err)
        }
        items = append(items, item)
        weights = append(weights, item.Weight)
    }

    i, err := linearSearchLottery(weights)
    if err != nil {
        log.Fatalf("アイテムの抽選に失敗しました err: %v\n", err)
    }

    result := items[i]

    // 取得したアイテムを記録する
    var (
        userItem UserItem
        firstGet bool
    )

    if err := db.QueryRow(
        "SELECT id, user_id, item_id, count FROM user_items WHERE user_id = ? AND item_id = ?", userId, result.ID,
    ).Scan(&userItem.ID, &userItem.UserId, &userItem.ItemId, &userItem.Count); err != nil && errors.Is(err, sql.ErrNoRows) {
        firstGet = true
    } else if err != nil && !errors.Is(err, sql.ErrNoRows) {
        log.Fatal(err)
    }

    tx, err := db.Begin()
    if err != nil {
        log.Fatal(err)
    }

    if firstGet {
        if _, err = tx.Exec("INSERT INTO user_items(id, user_id, item_id, count) VALUES(NULL, ?, ?, 1)", userId, result.ID); err != nil {
            log.Fatal(err)
        }
    } else {
        if _, err := tx.Exec("UPDATE user_items SET count = count + 1 WHERE user_id = ? AND item_id = ?", userId, result.ID); err != nil {
            log.Fatal(err)
        }
    }

    // コインを消費する
    if _, err = tx.Exec("UPDATE users SET coin = coin - ? WHERE id = ?", GachaPrice, userId); err != nil {
        _ = tx.Rollback()
        log.Fatal(err)
    }

    tx.Commit()

    fmt.Printf("{\"itemName\": %s, \"rare\": %s}\n", result.Name, result.Rare)
}

/*
線形探索で重み付抽選する
@return 当選した要素のインデックス
*/
func linearSearchLottery(weights []int) (int, error) {
    //  重みの総和を取得する
    var total int
    for _, weight := range weights {
        total += weight
    }

    // 乱数取得
    rnd := rand.IntN(total)

    var currentWeight int
    for i, w := range weights {
        // 現在要素までの重みの総和
        currentWeight += w

        if rnd < currentWeight {
            return i, nil
        }
    }

    // たぶんありえない
    return 0, errors.New("the lottery failed")
}
```

DBを使うので以下のコマンドでMySQLコンテナを起動しておきます。

```
docker run --rm --name go-mock-test-db \
  -e MYSQL_ROOT_PASSWORD=password \
  -e MYSQL_DATABASE=app \
  -p 13306:3306 \
  -v $(pwd)/database:/docker-entrypoint-initdb.d \
  -d mysql:9.0
```

database/create-table.sql

database/create-table.sql

```sql
CREATE TABLE \`users\`(
    \`id\` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    \`name\` VARCHAR(50) NOT NULL,
    \`coin\` BIGINT NOT NULL
);
CREATE TABLE \`user_items\`(
    \`id\` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    \`user_id\` BIGINT UNSIGNED NOT NULL,
    \`item_id\` BIGINT UNSIGNED NOT NULL,
    \`count\` INT NOT NULL
);
CREATE TABLE \`items\`(
    \`id\` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    \`name\` VARCHAR(255) NOT NULL,
    \`rare\` CHAR(2) NOT NULL,
    \`weight\` INT NOT NULL
);
ALTER TABLE
    \`user_items\` ADD CONSTRAINT \`user_items_item_id_foreign\` FOREIGN KEY(\`item_id\`) REFERENCES \`items\`(\`id\`);
ALTER TABLE
    \`user_items\` ADD CONSTRAINT \`user_items_user_id_foreign\` FOREIGN KEY(\`user_id\`) REFERENCES \`users\`(\`id\`);

INSERT INTO users (\`id\`, \`name\`, \`coin\`)
VALUES(NULL, "user1", 100);

INSERT INTO items (\`id\`, \`name\`, \`rare\`, \`weight\`) VALUES 
(NULL, "item1", "N", 15),
(NULL, "item2", "N", 15),
(NULL, "item3", "N", 15),
(NULL, "item4", "N", 15),
(NULL, "item5", "N", 15),
(NULL, "item6", "R", 6),
(NULL, "item7", "R", 6),
(NULL, "item8", "R", 6),
(NULL, "item9", "R", 6),
(NULL, "item10", "SR", 1);
```

コンテナが起動できたら以下のコマンドを実行します。

```
USER_ID=1
go run cmd/gacha/main.go $USER_ID

> {"itemName": item7, "rare": R}
```

とりあえずアイテムがちゃんと抽選できました！

## テストを書く

実装ができたところで本題のテストを書いていきましょう！

### テストファーストについて

実践テスト駆動開発はタイトル通りテスト駆動開発(TDD)について書籍の大半を使い書かれています。単体テストの考え方で推奨されている古典学派もTDDを使ったテストについての考え方です。そのため、どちらの流派に寄せるにせよ完全にその流派に従うのであればTDDを実践すべきです。

今回はモックを使ったテスト方法についてフォーカスしたかったので先に実装を書いてしまっていますが、TDDを実践するのであれば **テストファーストで先にテストを書く** ことを意識するといいと思います。

![](https://storage.googleapis.com/zenn-user-upload/17086c7fe9ce-20240815.png)

テスト駆動開発としての流れはざっくり以下のような感じになると思います。

1. 機能実装するためのTODOリストを作る
2. 失敗するテストを書く
3. 実装する
4. テストを成功させる

TODOリストを全てチェックできるまで2から4までのサイクルを繰り返す感じでしょうか。実際はリファクタリングもこのサイクルに入ってくると思います。

ここで実践テスト駆動開発に書かれている内容として特筆したいのは **最初の失敗するテストとしてE2Eテストを受け入れテストとして書く** ということです。書籍の中では以下のような図が記載されています。

![](https://storage.googleapis.com/zenn-user-upload/59638a4ce858-20240815.png)

実践テスト駆動開発はタイトル通り実践的な内容で実際の開発フロー目線でテストについて書かれているように感じます。最初にE2Eテストのような広い範囲のテストを用意するのは早い段階で **動くシステム** を用意したいという意図があるのだと思います。加えて、ビルドやデプロイ、自動テストなどのCI/CD環境を早い段階で用意することも重要だと説明していて最も重要で問題の多い部分であるからこそ開発の初期の段階でこういった仕組みを作り、 **デプロイできるシステム** の状態を保証することが重要としています。

そして、その後で前述したようなTDDのより小さなサイクルを回せばいいとしています。

ここら辺は単体テストよりも大きな枠組みの話でロンドン学派と呼ばれる流派のやり方なのかまではわかりませんがロンドン学派がバイブルとしている実践テスト駆動開発におけるTDDのサイクルの回し方という意味では覚えておくと良い内容でしょう。

### テストを書きやすくする

こちらもモックを使ったテストとはまだ関係ありませんが実践テスト駆動開発では **テストが書きづらいと感じたらまずリファクタリングすべき** としています。これは単体テストの考え方で書かれていたように思うのですがロンドン学派のような **モックを使ったテストは設計の悪さから目をそらしている** というようなことが書かれていたと思います。テストが書きづらいと感じたならそれはテストを書く前に多くのデータや依存オブジェクトを用意する必要があったり、良くない設計になっていることが多く、モックはそれらの問題を無視してテストを書くことになるからだというような内容だったと思います。

しかし、ロンドン学派がバイブルとしている実践テスト駆動開発では **テストが書きづらいと感じたならばそれは設計が良くない証拠なのでテストを書く前にリファクタリングすべき** だと説明しています。

これもいろんなところで言われていることで **コードの良くない臭い** を感じろみたいな話だと思うのですが、それを感じたらテストを書く前にまずリファクタリングしろと明確に説明しているのはあまりなかったような気がしています。

少なくとも、ロンドン学派がバイブルとしている実践テスト駆動開発では **良くない設計に目を背けずむしろ積極的にリファクタリングをする** こととしているようです。

今回実装したコードを見てみると `main` 関数に全てベタ書きの状態でテストが書きやすいとは言えない状態なのでテストを書く前に少しリファクタリングしましょう。 `main` 関数の処理をテストが書きやすいように切り出してみます。

gacha/gacha.go

```
package gacha

import (
    "database/sql"
    "errors"
    "fmt"
    "math/rand/v2"
)

type Gacha struct {
    db *sql.DB
}

func NewGacha(db *sql.DB) *Gacha {
    return &Gacha{db: db}
}

type User struct {
    ID   int64
    Name string
    Coin int64
}

type Item struct {
    ID     int64
    Name   string
    Rare   string
    Weight int
}

type UserItem struct {
    ID     int64
    UserId int64
    ItemId int64
    Count  int
}

const (
    GachaPrice = 10
)

func (g *Gacha) Draw(userId int64) (string, error) {
    // ユーザー情報を取得する
    var user User
    if err := g.db.QueryRow("SELECT id, name, coin FROM users WHERE id = ?", userId).Scan(&user.ID, &user.Name, &user.Coin); err != nil {
        return "", fmt.Errorf("ユーザーの取得に失敗しました err: %v\n", err)
    }

    if user.Coin < GachaPrice {
        return "", errors.New("コインが足りません")
    }

    // ガチャを抽選する
    rows, err := g.db.Query("SELECT id, name, rare, weight FROM items")
    if err != nil {
        return "", fmt.Errorf("アイテムの取得に失敗しました err: %v\n", err)
    }
    defer func() {
        _ = rows.Close()
    }()

    columns, err := rows.Columns()
    if err != nil {
        return "", err
    }

    items := make([]Item, 0, len(columns))
    weights := make([]int, 0, len(columns))

    for rows.Next() {
        var item Item
        if err = rows.Scan(&item.ID, &item.Name, &item.Rare, &item.Weight); err != nil {
            return "", err
        }
        items = append(items, item)
        weights = append(weights, item.Weight)
    }

    i, err := linearSearchLottery(weights)
    if err != nil {
        return "", fmt.Errorf("アイテムの抽選に失敗しました err: %v\n", err)
    }

    result := items[i]

    // 取得したアイテムを記録する
    var (
        userItem UserItem
        firstGet bool
    )

    if err = g.db.QueryRow(
        "SELECT id, user_id, item_id, count FROM user_items WHERE user_id = ? AND item_id = ?", userId, result.ID,
    ).Scan(&userItem.ID, &userItem.UserId, &userItem.ItemId, &userItem.Count); err != nil && errors.Is(err, sql.ErrNoRows) {
        firstGet = true
    } else if err != nil && !errors.Is(err, sql.ErrNoRows) {
        return "", err
    }

    tx, err := g.db.Begin()
    if err != nil {
        return "", err
    }

    if firstGet {
        if _, err = tx.Exec("INSERT INTO user_items(id, user_id, item_id, count) VALUES(NULL, ?, ?, 1)", userId, result.ID); err != nil {
            return "", err
        }
    } else {
        if _, err = tx.Exec("UPDATE user_items SET count = count + 1 WHERE user_id = ? AND item_id = ?", userId, result.ID); err != nil {
            return "", err
        }
    }

    // コインを消費する
    if _, err = tx.Exec("UPDATE users SET coin = coin - ? WHERE id = ?", GachaPrice, userId); err != nil {
        _ = tx.Rollback()
        return "", err
    }

    tx.Commit()

    return fmt.Sprintf("{\"itemName\": %s, \"rare\": %s}\n", result.Name, result.Rare), nil
}

/*
線形探索で重み付抽選する
@return 当選した要素のインデックス
*/
func linearSearchLottery(weights []int) (int, error) {
    //  重みの総和を取得する
    var total int
    for _, weight := range weights {
        total += weight
    }

    // 乱数取得
    rnd := rand.IntN(total)

    var currentWeight int
    for i, w := range weights {
        // 現在要素までの重みの総和
        currentWeight += w

        if rnd < currentWeight {
            return i, nil
        }
    }

    // たぶんありえない
    return 0, errors.New("the lottery failed")
}
```

まだテストが書きやすい状態とは言えませんが `main` 関数にベタ書きしていた処理を切り出すことでテストが書ける状態にすることができました。

### DB操作を抽象化して切り出す

ここまででテストが書ける状態にまですることはできましたが、まだテストが書きやすい状態とは言えません。特にDBが絡んだ処理があるため以下の2点について準備する必要があります。

- テスト実行時に使うためのDBを用意する必要がある
- テスト実行前にテストデータを用意する必要がある

単体テストの考え方で主張される **古典学派ではこのようなDBを扱う処理をモック化すべきではない** としています。古典学派がモックを使うべきではないと主張する理由は以下の通りです。

- モックを使うことで実装と密になり偽陽性を生じさせるため
- モックを使わないことでリポジトリ層のテストを書かなくても良くなる(リポジトリを使うControllerをモックを使わずにテストすることで動作を検証できるから)

逆にモックを使う利点は以下のようなことが挙げられます。

- テストの高速化(実際のDBを使ったテストは遅い)
- 依存オブジェクトをモック化することで依存オブジェクトを用意するのが容易
- モックを使うことでテストデータを用意する必要がない

モックを使わない古典学派は **偽陽性を持ち込まないこと** を重要視しているようにうも思えますし、ロンドン学派は **TDDのサイクルをなるべく早く回すこと** を重要視しているようにも思えます。

何度も言うようですがどちらが優れているかという話ではありませんが、今回はモックを使ったテスト手法について学ぶことが目的のためDB操作の部分をモック化しやすいように **リポジトリ** として切り出します。

Transaction

tx/tx.go

```
package tx

import (
    "database/sql"
    "errors"
)

const (
    NotYetCompletedErr = "not yet commit or rollback"
    NotBeginErr        = "not begin transaction"
)

type Transaction interface {
    Begin() error
    Commit() error
    Rollback() error
    Exec(query string, args ...any) error
}

type transaction struct {
    db *sql.DB
    tx *sql.Tx
}

func NewTransaction(db *sql.DB) *transaction {
    return &transaction{db: db}
}

func (t *transaction) Begin() error {
    if t.tx != nil {
        return errors.New(NotYetCompletedErr)
    }

    tx, err := t.db.Begin()
    if err != nil {
        return err
    }
    t.tx = tx

    return nil
}

func (t *transaction) Commit() error {
    if t.tx == nil {
        return errors.New(NotBeginErr)
    }

    if err := t.tx.Commit(); err != nil {
        return err
    }
    t.tx = nil

    return nil
}

func (t *transaction) Rollback() error {
    if t.tx == nil {
        return errors.New(NotBeginErr)
    }

    if err := t.tx.Rollback(); err != nil {
        return err
    }
    t.tx = nil

    return nil
}

func (t *transaction) Exec(query string, args ...any) error {
    if t.tx == nil {
        return errors.New(NotBeginErr)
    }

    _, err := t.tx.Exec(query, args...)
    return err
}
```

UserRepository

gacha/repository/user.go

```
package repository

import (
    "database/sql"
    "fmt"
    "go-mock-test-demo/gacha/domain"
    "go-mock-test-demo/tx"
)

type User interface {
    FindById(id int64) (user *domain.User, err error)
    DecreaseCoinsWithTx(tx tx.Transaction, userId int64, amount int) (err error)
}

type user struct {
    db *sql.DB
}

func NewUser(db *sql.DB) *user {
    return &user{db: db}
}

func (u *user) FindById(id int64) (*domain.User, error) {
    var user domain.User
    if err := u.db.QueryRow("SELECT id, name, coin FROM users WHERE id = ?", id).Scan(&user.ID, &user.Name, &user.Coin); err != nil {
        return nil, fmt.Errorf("ユーザーの取得に失敗しました err: %v", err)
    }
    return &user, nil
}

func (u *user) DecreaseCoinsWithTx(tx tx.Transaction, userId int64, amount int) (err error) {
    return tx.Exec("UPDATE users SET coin = coin - ? WHERE id = ?", amount, userId)
}
```

ItemRepository

gacha/repository/item.go

```
package repository

import (
    "database/sql"
    "fmt"
    "go-mock-test-demo/gacha/domain"
)

type Item interface {
    FindItemAndWeights() (items []*domain.Item, weights []int, err error)
}

type item struct {
    db *sql.DB
}

func NewItem(db *sql.DB) *item {
    return &item{db: db}
}

func (i *item) FindItemAndWeights() (items []*domain.Item, weights []int, err error) {
    rows, err := i.db.Query("SELECT id, name, rare, weight FROM items")
    if err != nil {
        return nil, nil, fmt.Errorf("アイテムの取得に失敗しました err: %v", err)
    }
    defer func() {
        _ = rows.Close()
    }()

    columns, err := rows.Columns()
    if err != nil {
        return nil, nil, err
    }

    items = make([]*domain.Item, 0, len(columns))
    weights = make([]int, 0, len(columns))

    for rows.Next() {
        var item domain.Item
        if err = rows.Scan(&item.ID, &item.Name, &item.Rare, &item.Weight); err != nil {
            return nil, nil, err
        }
        items = append(items, &item)
        weights = append(weights, item.Weight)
    }

    return items, weights, nil
}
```

UserItemRepository

gacha/repository/user\_item.go

```
package repository

import (
    "database/sql"
    "errors"
    "go-mock-test-demo/tx"
)

type UserItem interface {
    Exist(userId, itemId int64) (bool, error)
    CreateWithTx(tx tx.Transaction, userId, itemId int64) (err error)
    IncrementCountWithTx(tx tx.Transaction, userId, itemId int64) (err error)
}

type userItem struct {
    db *sql.DB
}

func NewUserItem(db *sql.DB) *userItem {
    return &userItem{db: db}
}

func (u *userItem) Exist(userId, itemId int64) (bool, error) {
    if err := u.db.QueryRow(
        "SELECT id, user_id, item_id, count FROM user_items WHERE user_id = ? AND item_id = ?", userId, itemId,
    ).Err(); err != nil && errors.Is(err, sql.ErrNoRows) {
        return false, nil
    } else if err != nil && !errors.Is(err, sql.ErrNoRows) {
        return false, err
    } else {
        return true, nil
    }
}

func (u *userItem) CreateWithTx(tx tx.Transaction, userId, itemId int64) (err error) {
    return tx.Exec("INSERT INTO user_items(id, user_id, item_id, count) VALUES(NULL, ?, ?, 1)", userId, itemId)
}

func (u *userItem) IncrementCountWithTx(tx tx.Transaction, userId, itemId int64) (err error) {
    return tx.Exec("UPDATE user_items SET count = count + 1 WHERE user_id = ? AND item_id = ?", userId, itemId)
}
```

### ランダム性の抽象化

単体テストの考え方では副作用の少ない関数がテストが書きやすいものとしています。副作用が少ないとは同じ入力であれば同じ結果が返るということです。

現在の `Gacha.Draw()` はアイテムの抽選処理に `math/rand` を使用しているため疑似乱数を使用します。疑似乱数は代表的な副作用であるため `Gacha.Draw()` は毎回違う結果を返します。逆に、そうでなければ毎回同じアイテムを抽選してしまいます。

このような、副作用は抽象化して外部依存としてしまいましょう。

random/random.go

```
package random

import "math/rand/v2"

type RandGenerator interface {
    IntN(n int) int
}

type randGenerator struct {
}

func NewRandGenerator() *randGenerator {
    return &randGenerator{}
}

func (r *randGenerator) IntN(n int) int {
    return rand.IntN(n)
}
```

この `RandomGenerator` を依存オブジェクトとして扱うことでモック化することが可能となりました。ちなみに、古典学派では極力モックを使用しないとしていますが **全く使わないわけではありません** 。古典学派でモックの使用が許されているのはこちらでは管理ができない **プロセス外依存** のみとしています。今回の疑似乱数のようにこちらで完全に管理することができないものは古典学派でもモックまたはスタブを利用するとされています。

### モックとスタブ

一通りモックを利用できる準備が整ったので依存オブジェクトをモック化していきましょう。モック化する前にモックとスタブの違いについて軽く説明しておきます。モックライブラリはモックとスタブをあまり区別せず使えるようになっているものも多いようですがモックを使ったテストをちゃんとやるならばちゃんと区別したほうがいいでしょう。

モックもスタブもテストのために動作を模倣したテストダブルの一種です。モックは外部に向かった挙動を模倣するものでスタブは内部に向かった挙動を模倣するものです。例えばメールシステムに依存していてメールを送信する部分をテストダブルに置き換えるならそれは **外向きなのでモック** になります。今回作っているガチャの疑似乱数を作る部分をテストダブルに置き換えるならばそれはテスト対象に乱数を提供する部分で **内向きなのでこれはスタブ** になります。

今回はモックとスタブを区別してシンプルに使用できるmoqを使ってモックもしくはスタブを作成します。moqに関しては以下の記事が大変わかりやすいです。

```
go install github.com/matryer/moq@latest
```

モックの作成には以下のように各ファイルに `go generate` として記載して生成します。

```
//go:generate moq -out item_moq.go -stub . Item
```

```
go generate ./...
```

### アローアンスとエクスペクテーション

**エクスペクテーション** とはテスト対象のオブジェクトが依存している隣接オブジェクトのモックがテスト対象のオブジェクトとどのようなコミュニケーションをするかについて設定した期待する振る舞いの定義のことです。一方、 **アローアンス** とは呼ばれても呼ばれなくてもどちらでもよく、テスト対象のオブジェクトが正しく実行できるようにするための補助的な機構のことです。

実践テスト駆動開発ではこのエクスペクテーションとアローアンスを区別することで **テスト対象のオブジェクトの何をテストしていてどのオブジェクトのどの動作が重要なのか** を明確にしています。逆に、全てのモックオブジェクトのエクスペクテーションを作成してしまいそのテストが何をテストしているのかがわかりづらくなるということをアンチパターンとして紹介もしていました。

モックを使ったテストを書くにはテスト対象の依存オブジェクトが **内向きに作用する** のか **外向きに作用する** のかを知るために **モック** と **スタブ** を区別して使うのがいいでしょう。そして、置き換えたモックオブジェクトの中で **外向きに副作用のある** もののみ **エクスペクテーション** を作成し、そのほかの副作用のない振る舞いは全てアローアンスとして無視してしまってよいでしょう。そうすることで、そのテストがどういう振る舞いをテストしていて、何が重要なのかが明確になるはずです。

### モックを使ったテストを作成する

前置きが長くなりましたがモックの作成までできたので実際にテストを書いてみます。moqで作成した依存オブジェクトは以下のようにして振る舞いを定義することができます。

```
userRep = &repository.UserMock{
  FindByIdFunc: func(id int64) (*domain.User, error) {
    return &domain.User{Coin: 100}, nil
  },
```

ここで `Gacha.Draw()` のテストを実行するために必要な依存オブジェクトは以下の通りです。

- `repository.User`
	- `FindById()`
	- `DecreaseCoinsWithTx()`
- `repository.Item`
	- `FindItemAndWeights()`
- `repository.UserItem`
	- `Exist()`
	- `CreateWithTx()`
	- `IncrementCountWithTx()`
- `random.RandGenerator`
	- `IntN()`
- `tx.Transaction`
	- `Begin()`
	- `Commit()`
	- `Rollback()`

これらをスタブとモックに分類すると以下のようになります。

**モック**

- `repository.User.DecreaseCoinsWithTx()`
- `repository.UserItem.CreateWithTx()`
- `repository.UserItem.IncrementCountWithTx()`
- `tx.Transaction.Begin()`
- `tx.Transaction.Commit()`
- `tx.Transaction.Rollback()`

**スタブ**

- `repository.User.FindById()`
- `repository.Item.FindItemAndWeights()`
- `repository.UserItem.Exist()`
- `random.RandGenerator.IntN()`

スタブに分類できるものは副作用のない処理で何回呼んだとしても結果が変わらない処理です。スタブに分類できるものは主に **問い合わせ** の処理や内部的に必要な内向きの処理です。上記で分類したものを見てみてもDBへの問い合わせと乱数生成の処理で外部の状態を変化させるような副作用のある処理はなさそうです。

モックに分類できるものは副作用のある処理で基本的には複数回呼ぶと結果が変わってしまうような処理になり、主に更新系の処理が該当します。トランザクションの処理はどちらになるか悩んだのですがこれらの処理はDBという外向きのシステムにたいして作用するものなのでモックに分類しましたが、もし間違っていたらコメントください。

テストを書くにあたってまずスタブに関してはテスト対象の関数を正常に動作させるために必要な処理で **それらがどう呼ばれたかに関しては興味がありません** 。前述したエクスペクテーションとアローアンスでいうとアローアンスになります。

モックに関しては **アイテムを1件抽選し、記録する** という振る舞いをテストするのに検証が必要な部分はエクスペクテーションとなるため引数や呼び出し回数の検証が必要です。

これらのことを踏まえて以下のようなテストを作成しました。

gacha/gacha\_test.go

```
package gacha_test

import (
    "fmt"
    "go-mock-test-demo/gacha"
    "go-mock-test-demo/gacha/domain"
    "go-mock-test-demo/gacha/repository"
    "go-mock-test-demo/random"
    "go-mock-test-demo/tx"
    "testing"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
)

func TestDraw(t *testing.T) {
    // Arrange
    var (
        items = []*domain.Item{
            {ID: 1, Name: "item1", Rare: "N"},
            {ID: 2, Name: "item2", Rare: "N"},
            {ID: 3, Name: "item3", Rare: "N"},
            {ID: 4, Name: "item4", Rare: "N"},
            {ID: 5, Name: "item5", Rare: "N"},
            {ID: 6, Name: "item6", Rare: "R"},
            {ID: 7, Name: "item7", Rare: "R"},
            {ID: 8, Name: "item8", Rare: "R"},
            {ID: 9, Name: "item9", Rare: "R"},
            {ID: 10, Name: "item10", Rare: "SR"},
        }

        weights = []int{
            15,
            15,
            15,
            15,
            15,
            6,
            6,
            6,
            6,
            1,
        }

        userId int64 = 1
    )

    var (
        userRep = &repository.UserMock{
            FindByIdFunc: func(id int64) (*domain.User, error) {
                return &domain.User{Coin: 100}, nil
            },
        }
        itemRep = &repository.ItemMock{
            FindItemAndWeightsFunc: func() ([]*domain.Item, []int, error) {
                return items, weights, nil
            },
        }
        userItemRep = &repository.UserItemMock{}
        tx          = &tx.TransactionMock{}
        rnd         = &random.RandGeneratorMock{
            IntNFunc: func(n int) int {
                return 99
            },
        }
    )

    var (
        expectedItemName         = "item10"
        expectedItemRare         = "SR"
        expectedItemId     int64 = 10
        expectedGachaPrice       = 10
    )

    sut := gacha.NewGacha(userRep, itemRep, userItemRep, tx, rnd)

    // Act
    result, err := sut.Draw(userId)

    // Assertion
    require.Nil(t, err)
    assertResult(t, expectedItemName, expectedItemRare, result)

    if assert.Len(t, userItemRep.CreateWithTxCalls(), 1) {
        assert.Equal(t, userId, userItemRep.CreateWithTxCalls()[0].UserId)
        assert.Equal(t, expectedItemId, userItemRep.CreateWithTxCalls()[0].ItemId)
    }

    if assert.Len(t, userRep.DecreaseCoinsWithTxCalls(), 1) {
        assert.Equal(t, userId, userRep.DecreaseCoinsWithTxCalls()[0].UserId)
        assert.Equal(t, expectedGachaPrice, userRep.DecreaseCoinsWithTxCalls()[0].Amount)
    }

    assert.Len(t, userItemRep.IncrementCountWithTxCalls(), 0)

    assert.Len(t, tx.BeginCalls(), 1)
    assert.Len(t, tx.CommitCalls(), 1)
    assert.Len(t, tx.RollbackCalls(), 0)
}

func assertResult(t *testing.T, itemName, rare, act string) {
    t.Helper()
    assert.Equal(t, fmt.Sprintf("{\"itemName\": %s, \"rare\": %s}\n", itemName, rare), act)
}
```

必要最低限のモックオブジェクトとエクスペクテーションの検証をすることでこのテストが何をテストしているかが明確になっているかと思います。このように、モックを使ったテストを書くには **モックとスタブの区別** 、 **アローアンスとエクスペクテーションの区別** をちゃんと意識することで **何をテストしているかが明確な振る舞いをテストする良い単体テスト** が書けるのではないかと思います。

## 感想

実践テスト駆動開発を読む前はモックを使ったテストは依存オブジェクトを全てモックにして、結果をアサーションするくらいに思っていました。モックオブジェクトに設定した振る舞いを検証できることは知っていましたが依存オブジェクトが多いと検証も多くなりテストが長くなることからあまりモックオブジェクト自体の検証はしてこなかったのと、モックオブジェクトの検証はあまりテストになっている気がしなく、なんとなくで書いていました。

そういったこともあり、単体テストの考え方/使い方を読んでからはモックを極力使わない古典学派寄りの思想でテストを書いていました。DBの接続には [Testcontainers](https://testcontainers.com/) というテストコードからコンテナを操作できるライブラリを筆者は好んで使っているのでDBコンテナをテスト前に起動してテストしていましたが以下のような不満もありました。

- コンテナの起動に時間がかかる(2,30秒くらいかかることが多い)
- ライブラリがあるにしてもコンテナの起動、終了、マイグレーションなどの下準備のコードが多くなる
- コンテナの起動まわりでエラーが出るとハマる
- テストを実行するためのデータの準備、および破棄が大変
- CI環境で動かそうとすると大変なときがある

古典学派というよりもコンテナを使ったテストへの不満みたいなものですがやはりテストデータの準備や実行速度は問題になりやすいなと感じています。

実践テスト駆動開発を読んだことで改めてモックを使ったテストについて考えることができましたが古典学派的なテストに比べて以下のような利点があるなと感じました。

- テストが早い
- 使用するモックライブラリによるがモックオブジェクトを作成することでテスト対象のオブジェクトを作成するのが容易
- DBやテストデータを事前に準備する必要がない

実際の開発現場において依存オブジェクトが多すぎたり、テストデータを大量に用意する必要があったりなどの理由でテストが書きづらいといったことはよくあると思います。そのような時は、設計が良くないことが多いと単体テストの考え方でも実践テスト駆動開発でも説明されており、まずはリファクタリングすべきとされていますが全ての開発現場で積極的にリファクタリングができるわけではないと筆者は思います。

複雑度の増す現代の開発において依存オブジェクトをモック化してテストできるほうがテストの作成、修正は容易になると筆者は思います。

古典学派とロンドン学派のどちらを採用するかは何を重視するかに寄ると思いますが、システムの振る舞いを実際の挙動に寄せてテストしたいならば古典学派が良いでしょう。多少の偽陽性を持ち込むことを許容したうえで複雑なシステムを容易にテストしたいと思うならロンドン学派のテストの方が良いかもしれません。

どちらを採用するにせよ、 **振る舞いをテストする** 、 **テストを書きながら設計の悪さを検知する** 、 **リファクタリングは積極的にする** といったことを心がけることが重要だと筆者は思います。

個人的な感想ですがテストを書くのはモックを使ったほうが容易だと思っていますが、 **良いテストを書くのはモックを使ったテストの方が難しい** と思います。本記事で紹介したようなモックとスタブ、エクスペクテーションなどの知識がないと何をテストしているのかが分かりづらいテストとなり、それこそ実装と密になり偽陽性を持ち込みやすくなってしまうと思います。

そのような難しさみたいなものがあるため本記事の最初に書いたような何をテストしたらいいかわからないといったような感覚になってしまうことがあるのかなと思ったりします。

## おわりに

本記事ではロンドン学派がバイブルとする実践テスト駆動開発を参考に以下について実際にガチャのシステムのテストをモックを使用して書くことで紹介しました。

- 実践テスト駆動開発におけるTDDのサイクルについて
- 実践テスト駆動開発におけるリファクタリングの考え方
- モックを使いやすくするための抽象化
- モックとスタブの違いについて
- アローアンスとエクスペクテーションについて

実際に実践テスト駆動開発の内容を実践する場合、本記事では実装を先に書きましたがテストファーストで書き始めたり、結合テスト以上のテストを書く必要があるかもしれません。モックを使ったテストはテスト対象の依存オブジェクトと実際に結合した振る舞いはテストできていないためです。結合テストは本記事でテストをした `Gacha.Draw()` に関して実際のDBを接続したテストを書いたり各依存オブジェクトの結合テストを実際にDBにつないで書くなどが考えられます。

本記事で紹介したようなモックを使ったロンドン学派的なテストと単体テストの考え方/使い方で推奨されているような古典学派的なテスト、どちらが良いかは人やプロジェクト次第と思いますが **モックを使うことがいい悪い** みたいな話にはならないと筆者は考えます。

モックを使ったテストも使わないテストもどちらも知った上でどのようなテストを書くのがいいかを選んでいただければいいなと思います。そして、本記事が少しでもその役に立てれば嬉しく思います。

今回は以上です🐼

## 成果物

[GitHubで編集を提案](https://github.com/JY8752/zenn_article/blob/main/articles/cee5f11639ca83.md)

37