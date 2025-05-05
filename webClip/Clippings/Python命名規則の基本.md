---
title: Python命名規則の基本
source: https://zenn.dev/the_exile/articles/python-naming-convention-
author:
  - "[[Zenn]]"
published: 2024-05-25
created: 2025-05-06
description: 
tags:
  - Python
  - 1
  - 思想系
read:
---
61

2[tech](https://zenn.dev/tech-or-idea)

## はじめに

Pythonの命名規則は、コードの可読性を高めるために非常に重要です。  
実はPeP8というPythonのスタイルガイドには、命名規則に関する詳細なガイドラインが記載されています。  
本記事では、Pythonの命名規則について、PeP8に基づいてまとめたいと思います。

## なぜ命名規則が重要なのか

命名規則(Naming Convention)は、コードの可読性を高めるために非常に重要です。  
最も重要なのは一貫性(Consistency)で、コードが一貫性のある命名規則に従っていると、変数や関数の目的が明確になり、コードの理解が容易になります。  
また、命名規則に従っていると、他の開発者がコードを読んだり、メンテナンスしたりする際にも、迷うことなく作業を進られるため、作業効率UPにもつながります。

## Pythonの命名規則のタイプ

Pythonの命名規則には、大きく分けて以下の4つのタイプがあります。

```python
UPPER_UNDERSCORES = "HOGE_HOGE"
CamelCase = "HogeHoge"
lower_underscores = "hoge_hoge"
mixedCase = "hogeHoge" # あまり使われない
```

### 1\. UPPER\_UNDERSCORES

UPPER\_UNDERSCORESは、全て大文字で、単語間をアンダースコア(\_)で区切る命名規則です。  
主に定数やモジュールレベルの変数に使用されます。  
一度定義された変数には、値の変更ができないことを示すため、大文字で定義されることが多いです。  
ただし、Pythonでは定数が存在しないため、あくまで命名上の約束として使用されます。

```python
MAX_LENGTH = 10
```

### 2\. CamelCase

CamelCaseは、単語の先頭を大文字で始め、単語間をアンダースコア(\_)で区切らない命名規則です。  
CamelCaseはクラス名の命名にしか使われません。

```python
class MyClass:
    pass
```

### 3\. lower\_underscores

lower\_underscoresは、全て小文字で、単語間をアンダースコア(\_)で区切る命名規則です。  
基本定数とクラス以外のところ全て使用されます。

例:

- モジュール名
- 関数名
- メソッド名
- 変数名

```python
import my_module

my_variable = 10

def my_function():
    pass

class MyClass:
    def my_method(self):
        pass
```

## 他の命名規則

### \_だけの変数

`_` だけの変数は、慣習的に「この変数は使わない」という意味で使われます。  
例えば下記のfor文の例では、 `_` だけの変数は、ループ内で使われない意味になります。

```python
import random

for _ in range(10):
    print(random.randint(1, 10))
```

### \_から始まる変数や関数

`_` から始まる変数や関数は、慣習的に「プライベート」という意味ですが、  
`_` だけでは強制的にアクセスを制限することはできないので、主に約束事として使われます。

```python
_my_data: dict = {}

def _my_function():
    if x in _my_data:
        return _my_data[x]
    else:
        return None

class MyClass:
    def _get_data(self):
        print("hello")
       
# 強制的にアクセスを制限することはできないため、使おうとすれば使える 
a = MyClass()
a._get_data()

---
hello
```

### \_\_から始まる変数や関数

`__` から始まる変数や関数は、 `_` と同じく「プライベート」という意味ですが、  
強めの制限がかかります。

```python
class MyClass:
    def __get_data(self):
        print("hello")

# エラーが発生する
a = MyClass()
a.__get_data()

---
Traceback (most recent call last):
  File "xxx.py", line 6, in <module>
    a.__get_data()
    ^^^^^^^^^^^^
AttributeError: 'MyClass' object has no attribute '__get_data'
```

強めの制限だというものの、Pythonでは `_ClassName__method` という形でアクセスするこことができるため、  
完全にプライベートにすることはできません。

```python
class MyClass:
    def __get_data(self):
        print("hello")

# エラーが発生しない
a = MyClass()
a._MyClass__get_data()

---
hello
```

### \_\_から始まり、\_\_で終わる変数

`__` から始まり、 `__` で終わる変数は、特殊な変数で、Pythonによって予約されています。  
これらの変数は、クラス内でのみ使用される特殊な変数です。

```python
class MyClass:
    def __init__(self):
        self.__data__ = 10
```

個人的には、 `__` から始まり、 `__` で終わる変数は、自分たちで定義することは避けるべきだと思います。

### built-in関数

Pythonには、組み込み関数として定義されている関数があります。  
これらの関数は、変数名や関数名として使用することはできますが、  
変なBugを引き起こす可能性があるため、避けるべきです。  
また、Pythonファイルの命名にもbuilt-in関数名を使わないようにしましょう。

```python
# 避けるべき
list = [1, 2, 3]
dict = {"a": 1, "b": 2}
```

## まとめ

Pythonの命名規則は、コードの可読性を高めるために非常に重要です。  
コードがきれいになるだけではなく、効率UPにもつながるため、しっかりとした命名規則を守るようにしましょう。

61

2