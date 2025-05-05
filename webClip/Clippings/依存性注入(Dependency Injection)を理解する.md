---
title: 依存性注入(Dependency Injection)を理解する
source: https://zenn.dev/atsu77/articles/230612_dependency_injection#
author:
  - "[[Zenn]]"
published: 2023-06-12
created: 2025-05-06
description: 
tags:
  - 1
  - 設計
read:
---
10[tech](https://zenn.dev/tech-or-idea)

## 依存性注入とは何か？

依存性注入とは、一つのオブジェクトが別のオブジェクトを活用する際に、そのオブジェクトを外部から供給する手法です。これにより、オブジェクト間の依存関係を解消することが可能となります。

## 依存性注入の利点

- テストの容易性が増す
- オブジェクト間の依存関係が明瞭になる
- オブジェクトの再利用性が向上する

## サンプルコードによる解説

ここでは、CarクラスとEngineクラスが存在する状況を考えます。CarクラスはEngineクラスを用いているため、CarはEngineに依存しているといえます。

### 依存性注入を適用しないケース

以下の例では、CarクラスのコンストラクタでEngineクラスのインスタンスを生成しています。これにより、Carクラスは特定のEngineクラス（この例では `V8` エンジン）に直接依存してしまいます。結果として、柔軟性が失われ、テストや再利用が難しくなります。

```java
class Car {
    private String name;
    private Engine engine;

    public Car(String name) {
        this.name = name;
        this.engine = new Engine("V8");
    }
}

class Engine {
    private String name;

    public Engine(String name) {
        this.name = name;
    }
}
```

### 依存性注入を適用するケース

依存性注入を適用した場合、CarクラスのコンストラクタはEngineクラスのインスタンスを引数として受け取り、Carクラスのインスタンス変数に代入します。これにより、Carクラスは特定のEngineクラスに直接依存しない形になります。つまり、Engineクラスの任意の具体的な実装をCarクラスに注入することが可能となります。

```java
class Car {
    private String name;
    private Engine engine;

    public Car(String name, Engine engine) {
      this.name = name;
      this.engine = engine;
    }
}

class Engine {
    private String name;

    public Engine(String name) {
      this.name = name;
    }
}
```

10

### Discussion

![](https://static.zenn.studio/images/drawing/discussion.png)

ログインするとコメントできます