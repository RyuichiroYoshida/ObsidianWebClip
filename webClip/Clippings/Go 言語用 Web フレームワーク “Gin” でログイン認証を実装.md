---
title: Go 言語用 Web フレームワーク “Gin” でログイン認証を実装
source: https://go.genzouw.com/2023/03/31/post-331/
author:
  - "[[Go言語用ポストイット]]"
published: 2023-03-31
created: 2025-05-06
description: はじめに 前回までで、Gin フレークワームを使った本当に簡単なプログラムを組むことができました。 今回はログイン認証機構を実装していきます。 JSON Web Token とは？ JSON Web ...
tags:
  - Go
  - Tech
  - バックエンド
read: false
---
## はじめに

前回までで、Gin フレークワームを使った本当に簡単なプログラムを組むことができました。

今回はログイン認証機構を実装していきます。

## JSON Web Token とは？

JSON Web Token ( JWT ) は、RFC 7519 で定義された標準規格です。  
JWT は、クライアントとサーバー間で情報を安全にやり取りするための仕組みです。  
オブジェクトを使用して、クライアントとサーバー間で情報を暗号化し、署名し、またはその両方を行うことができます。

JWT についてはここまでにして本題に入りたいと思います。

### REST

3 つのエンドポイントを作成します。

1. 「サインアップ」エンドポイント
2. 「ログイン」エンドポイント
3. 「ログイン後にしか呼び出しできない」エンドポイント

### 「サインアップ」エンドポイント

当然ながら、ログイン前にユーザーのサインアップ ( ユーザー登録 ) が必要です。  
このパスは JWT を不要にし公開したままにします。

- `/api/register`

### 「ログイン」エンドポイント

ログインに利用されるパスです。  
「ユーザー名」と「パスワード」を受け取り、JWT を生成して返却します。

- `/api/ogin`

### 「ログイン後にしか呼び出しできない」エンドポイント

このパスは JWT が必須となります。

- `/api/admin/user`

## 作業開始 ( セットアップ )

まずはプロジェクトフォルダを作成します。

```bash
$ mkdir jwt-gin

$ cd jwt-gin
```

Bash

お約束の `go.mod` ファイルを作成します。  
ここで作成するモジュール(アプリケーション)名は `jwt-gin` としました。

```bash
$ go mod init jwt-gin
```

Bash

必要なモジュールをインストールします。

```bash
# Gin フレームワーク
$ go get -u github.com/gin-gonic/gin

# ORM ライブラリ
$ go get -u github.com/jinzhu/gorm

# JWT の認証と生成に使用するパッケージ
$ go get -u github.com/dgrijalva/jwt-go

# .env ファイルから環境変数を読み込む
$ go get -u github.com/joho/godotenv

# パスワード暗号化
$ go get -u golang.org/x/crypto
```

Bash

必要なモジュールをインストールして準備を完了したら、コーディングを開始します。プログラムの main 関数を作成し、最初のエンドポイントを作成します。

## コーディング開始

## プログラムの main 関数作成

プロジェクトルートディレクトリ に `main.go` ファイルを作ります。

```bash
$ vi main.go
```

Bash

最初のエンドポイントを作成します。

**main.go**

実行してみます。

```bash
$ go run main.go
```

Bash

ブラウザや `curl` でアクセスし、JSON が出力されれば成功です。

次にサインアップのメインロジックをコーディングしていきます。

## 「サインアップ」エンドポイント Controller 作成

登録処理を追加します。

Controller を格納するディレクトリ `controllers` を作成します。

```bash
$ mkdir ./controllers
```

Bash

`auth.go` というファイルを作成し、Controller の処理を実装します。

```bash
$ vi ./controllers/auth.go
```

Bash

**auth.go**

作成した `Register` 関数が `/register` エンドポイントと結びつくよう `main.go` を変更します。

**main.go**

`main.go` が起動済みの場合は一度停止してから、起動します。

```bash
$ go run main.go
```

Bash

ブラウザや `curl` でアクセスし、JSON が出力されれば成功です。

`Register` 関数を更に充実させていきます。

## 「サインアップ」エンドポイント 入力チェック

サインアップには、ユーザー名とパスワードの情報だけが必要です。

**binding** と呼ばれる Gin の入力情報のチェック機能を利用します。

**auth.go**

`go run main.go` を起動し直した後、動作確認します。

1. ユーザー名だけ設定したリクエスト
2. パスワードだけ設定したリクエスト
3. ユーザー名、パスワードの両方を設定したリクエスト

## 「サインアップ」エンドポイント DB 初期化 + Model 作成

認証情報を MySQL データベースに保存ようと思います。

既存の MySQL データベースがあればそちらを使っても構いません。  
ここでは Docker を使って、MySQL を用意します。

```bash
# user=dev, password=dev, database=dev
$ docker run --rm -p 3307:3306 -e MYSQL_ROOT_PASSWORD=dev -e MYSQL_USER=dev -e MYSQL_PASSWORD=dev -e MYSQL_DATABASE=dev -d mysql/mysql-server:5.7
```

Bash

正しく起動すれば `mysql` コマンドで接続できるはずです。

```bash
$ mysql -h 127.0.0.1 -P 3307 -u dev -pdev dev
```

Bash

データベースへの接続処理と Model を格納するためのパッケージとコードを作成します。

```bash
$ mkdir -p ./models/

$ vi ./models/setup.go
```

Bash

`./models/setup.go` に以下のコードを記述します。

**setup.go**

```
package models

import (
    "fmt"
    "log"
    "os"

    "github.com/jinzhu/gorm"
    _ "github.com/jinzhu/gorm/dialects/mysql"
    "github.com/joho/godotenv"
)

var DB *gorm.DB

func ConnectDataBase() {
    err := godotenv.Load()

    if err != nil {
        log.Fatalf("Error loading .env file")
    }

    driver := os.Getenv("DB_DRIVER")
    dbUser := os.Getenv("DB_USER")
    dbPass := os.Getenv("DB_PASS")
    dbName := os.Getenv("DB_NAME")
    dbHost := os.Getenv("DB_HOST")
    dbPort := os.Getenv("DB_PORT")

    dbURI := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?charset=utf8&parseTime=True&loc=Local", dbUser, dbPass, dbHost, dbPort, dbName)

    DB, err := gorm.Open(driver, dbURI)

    if err != nil {
        log.Fatal("Could not connect to the database", err)
    }

    DB.AutoMigrate(&User{})
}
```

Go

以下の 2 つのものも必要となります。

- `.env`
- `User` 構造体

引き続き作成していきます。

プロジェクトのルートディレクトリに `.env` ファイルを作り、 MySQL サーバーへの接続情報を記述します。

```bash
$ cat <<EOF >>.env
DB_DRIVER=mysql
DB_USER=dev
DB_PASS=dev
DB_NAME=dev
DB_HOST=localhost
DB_PORT=3307
EOF
```

Bash

`User` Model を作成します。

```bash
$ vi ./models/user.go
```

Bash

**user.go**

```
package models

import "github.com/jinzhu/gorm"

type User struct {
    gorm.Model
    Username string \`gorm:"size:255;not null;unique" json:"username"\`
    Password string \`gorm:"size:255;not null;" json:"password"\`
}
```

Go

`main.go` にデータベース接続処理を追記します。

**main.go**

ようやくデータベース接続処理の動作確認ができるところまで来ました。

`main.go` を立ち上げ直します。

```bash
# 立ち上げの前に、使用するパッケージが増えたため追加処理を行っておきます
$ go mod tidy

# 起動
$ go run main.go
```

Bash

MySQL データベース内のテーブルを一覧表示してみます。

```bash
$ mysql -h 127.0.0.1 -P 3307 -u dev -pdev dev -e "show tables;"
mysql: [Warning] Using a password on the command line interface can be insecure.
+---------------+
| Tables_in_dev |
+---------------+
| users         |
+---------------+
```

Bash

データベースのマイグレーション処理が正しく動作し `users` テーブルが追加されました。

## 「サインアップ」エンドポイント ユーザー情報登録

サインアップの実装の最終段階です。

`users` テーブルへのデータ登録ロジックを実装します。

**user.go**

```
package models

import (
    "strings"

    "github.com/jinzhu/gorm"
    "golang.org/x/crypto/bcrypt"
)

type User struct {
    gorm.Model
    Username string \`gorm:"size:255;not null;unique" json:"username"\`
    Password string \`gorm:"size:255;not null;" json:"password"\`
}

func (u User) Save() (User, error) {
    err := DB.Create(&u).Error
    if err != nil {
        return User{}, err
    }
    return u, nil
}

func (u *User) BeforeSave() error {
    hashedPassword, err := bcrypt.GenerateFromPassword([]byte(u.Password), bcrypt.DefaultCost)
    if err != nil {
        return err
    }

    u.Password = string(hashedPassword)

    u.Username = strings.ToLower(u.Username)

    return nil
}

func (u User) PrepareOutput() User {
    u.Password = ""
    return u
}
```

Go

`User` 構造体に追加したメソッドを呼び出し、データを保存します。

**auth.go**

`main.go` を立ち上げ直します。

```bash
# 起動
$ go run main.go
```

Bash

再度、 `curl` などでリクエストを投げてみます。

データが登録され、かつパスワードはハッシュ化されています。

## 「ログイン」エンドポイント 作成

ログイン用のエンドポイントはとてもシンプルです。

ユーザー名とパスワードを受け取り、データベース内の認証情報 ( `users` テーブル) と突き合わせを行います。

データが見つからなかった場合は、エラーレスポンスを返します。

`main.go` を編集します。

`controllers.Login` メソッドの作成のために、 `auth.go` も編集します。

**auth.go**

```
package controllers

import (
    "net/http"

    "jwt-gin/models"

    "github.com/gin-gonic/gin"
)

type RegisterInput struct {
    Username string \`json:"username" binding:"required"\`
    Password string \`json:"password" binding:"required"\`
}

func Register(c *gin.Context) {
    var input RegisterInput

    if err := c.ShouldBindJSON(&input); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    user := models.User{Username: input.Username, Password: input.Password}

    user, err := user.Save()

    if err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    c.JSON(http.StatusOK, gin.H{
        "data": user,
    })
}

type LoginInput struct {
    Username string \`json:"username" binding:"required"\`
    Password string \`json:"password" binding:"required"\`
}

func Login(c *gin.Context) {
    var input LoginInput

    if err := c.ShouldBindJSON(&input); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    token, err := models.GenerateToken(input.Username, input.Password)

    if err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    c.JSON(http.StatusOK, gin.H{
        "token": token,
    })
}
```

Go

Controller である `Login` 関数から呼び出している `GenerateToken` 関数を `user.go` ファイルに追記します。

**user.go**

```
package models

import (
    "jwt-gin/utils/token"
    "strings"

    "github.com/jinzhu/gorm"
    "golang.org/x/crypto/bcrypt"
)

type User struct {
    gorm.Model
    Username string \`gorm:"size:255;not null;unique" json:"username"\`
    Password string \`gorm:"size:255;not null;" json:"password"\`
}

func (u User) Save() (User, error) {
    err := DB.Create(&u).Error

    if err != nil {
        return User{}, err
    }
    return u, nil
}

func (u *User) BeforeSave() error {
    hashedPassword, err := bcrypt.GenerateFromPassword([]byte(u.Password), bcrypt.DefaultCost)

    if err != nil {
        return err
    }

    u.Password = string(hashedPassword)

    u.Username = strings.ToLower(u.Username)

    return nil
}

func (u User) PrepareOutput() User {
    u.Password = ""
    return u
}

func GenerateToken(username string, password string) (string, error) {
    var user User

    err := DB.Where("username = ?", username).First(&user).Error

    if err != nil {
        return "", err
    }

    err = bcrypt.CompareHashAndPassword([]byte(user.Password), []byte(password))

    if err != nil {
        return "", err
    }

    token, err := token.GenerateToken(user.ID)

    if err != nil {
        return "", err
    }

    return token, nil
}
```

Go

トークンを操作する関数は `token.go` というファイルを新規に作成しこちらに含めます。

```bash
$ mkdir -p ./utils/token

$ vi ./utils/token/token.go
```

Bash

以下の３つの関数を外部から利用できるようにします。

- `GenerateToken()`
- `TokenValid()`
- `ExtractTokenId()`

ただし、 `TokenValid()` 、 `ExtractTokenId()` の 2 つは後ほど利用します。

**token.go**

```
package token

import (
    "fmt"
    "os"
    "strconv"
    "strings"
    "time"

    "github.com/dgrijalva/jwt-go"
    "github.com/gin-gonic/gin"
)

func GenerateToken(id uint) (string, error) {
    tokenLifespan, err := strconv.Atoi(os.Getenv("TOKEN_HOUR_LIFESPAN"))

    if err != nil {
        return "", err
    }

    claims := jwt.MapClaims{}
    claims["authorized"] = true
    claims["user_id"] = id
    claims["exp"] = time.Now().Add(time.Hour * time.Duration(tokenLifespan)).Unix()
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)

    return token.SignedString([]byte(os.Getenv("API_SECRET")))
}

func extractTokenString(c *gin.Context) string {
    bearToken := c.Request.Header.Get("Authorization")
    strArr := strings.Split(bearToken, " ")
    if len(strArr) == 2 {
        return strArr[1]
    }

    return ""
}

func parseToken(tokenString string) (*jwt.Token, error) {
    token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
        if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
            return nil, fmt.Errorf("There was an error")
        }
        return []byte(os.Getenv("API_SECRET")), nil
    })

    if err != nil {
        return nil, err
    }

    return token, nil
}

func TokenValid(c *gin.Context) error {
    tokenString := extractTokenString(c)

    token, err := parseToken(tokenString)

    if err != nil {
        return err
    }

    return nil
}

func ExtractTokenId(c *gin.Context) (uint, error) {
    tokenString := extractTokenString(c)

    token, err := parseToken(tokenString)

    if err != nil {
        return 0, err
    }

    claims, ok := token.Claims.(jwt.MapClaims)

    if ok && token.Valid {
        userId, ok := claims["user_id"].(float64)

        if !ok {
            return 0, nil
        }

        return uint(userId), nil
    }

    return 0, nil
}
```

Go

`.env` にコードに登場した環境変数を 2 つ追加します。

```bash
$ cat <<EOF >>.env
TOKEN_HOUR_LIFESPAN=1
API_SECRET=jwt-gin-dev
EOF
```

Bash

`main.go` を立ち上げ直します。

```bash
# 立ち上げの前に、使用するパッケージが増えたため追加処理を行っておきます
$ go mod tidy

# 起動
$ go run main.go
```

Bash

`curl` でいくつかリクエストを投げて動作を確認してみます。

1. ユーザー名だけ正しい
2. パスワードだけ正しい
3. ユーザー名、パスワードが正しい

ログインに成功したときにトークンが生成されるようになりました。

## 認証用ミドルウェア ( Filter ) 作成

Gin にはミドルウェアと呼ばれる機能があります。  
Controller にリクエストが到達する前に処理を行うことができます。

`middlewares.go` ファイルを新規に作成します。

```bash
$ mkdir -p ./middlewares/

$ vi ./middlewares/middlewares.go
```

Bash

**middlewares.go**

```
package middlewares

import (
    "jwt-gin/utils/token"
    "net/http"

    "github.com/gin-gonic/gin"
)

func JwtAuthMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        err := token.TokenValid(c)

        if err != nil {
            c.JSON(http.StatusUnauthorized, gin.H{"error": err.Error()})
            c.Abort()
            return
        }

        c.Next()
    }
}
```

Go

`main.go` に新しいエンドポイント `/api/admin/user` の追加とこれに対するミドルウェアの設定を行います。

**main.go**

`auth.go` に Controller となる `CurrentUser` 関数を追加します。

```
package controllers

import (
    "net/http"

    "jwt-gin/models"
    "jwt-gin/utils/token"

    "github.com/gin-gonic/gin"
)

type RegisterInput struct {
    Username string \`json:"username" binding:"required"\`
    Password string \`json:"password" binding:"required"\`
}

func Register(c *gin.Context) {
    var input RegisterInput

    if err := c.ShouldBindJSON(&input); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    user := models.User{Username: input.Username, Password: input.Password}

    user, err := user.Save()

    if err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    c.JSON(http.StatusOK, gin.H{
        "data": user.PrepareOutput(),
    })
}

type LoginInput struct {
    Username string \`json:"username" binding:"required"\`
    Password string \`json:"password" binding:"required"\`
}

func Login(c *gin.Context) {
    var input LoginInput

    if err := c.ShouldBindJSON(&input); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    token, err := models.GenerateToken(input.Username, input.Password)

    if err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    c.JSON(http.StatusOK, gin.H{
        "token": token,
    })
}

func CurrentUser(c *gin.Context) {
    userId, err := token.ExtractTokenId(c)

    if err != nil {
        c.JSON(http.StatusUnauthorized, gin.H{"error": err.Error()})
        return
    }

    var user models.User

    err = models.DB.First(&user, userId).Error

    if err != nil {
        c.JSON(http.StatusUnauthorized, gin.H{"error": err.Error()})
        return
    }

    c.JSON(http.StatusOK, gin.H{
        "data": user.PrepareOutput(),
    })
}
```

Go

`main.go` を立ち上げ直します。

```bash
# 起動
$ go run main.go
```

Bash

動作確認してみます。

まずは `/api/login` エンドポイントに対してログインを行い、トークンを取得します。

この操作でトークンが取得できたので、改めて `jq` コマンドを使ってパースしてみます。

結果シェル変数に保存しておきます。

次に、 `/api/admin/user` エンドポイントにリクエストしてみます。

まずはトークンを使わない場合。

```bash
$ curl --silent -X GET http://localhost:8080/api/admin/user
{"error":"token contains an invalid number of segments"}%
```

Bash

エラーとなりました。

今度は正しいトークンをヘッダーにセットしてリクエストしてみます。

```bash
$ curl --silent -X GET http://localhost:8080/api/admin/user -H "Authorization: Bearer ${TOKEN}"
{"data":{"ID":7,"CreatedAt":"2023-03-29T17:28:07+09:00","UpdatedAt":"2023-03-29T17:28:07+09:00","DeletedAt":null,"username":"genzouw","password":""}}
```

Bash

## ひとこと

Controller をどういう単位でファイルに分割するとチーム開発しやすいのでしょうか？

---

## 参考ページ

- [Create your first Go REST API with JWT Authentication in Gin Framework | by Seef Nasrul | Medium](https://seefnasrul.medium.com/create-your-first-go-rest-api-with-jwt-authentication-in-gin-framework-dbe5bda72817)

---