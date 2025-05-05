---
title: LINQ -> Python チートシート
source: https://zenn.dev/shimat/articles/d8812d20e803cf
author:
  - "[[Zenn]]"
published: 2023-12-16
created: 2025-05-06
description: 
tags:
  - Python
  - "#CSharp"
  - 設計
read:
---
11[tech](https://zenn.dev/tech-or-idea)

C# Advent Calendar 2023の16日目の記事です。が主題はPython！

「C#のLINQのアレは、Pythonではどう書くんだっけ？」という際の自分用メモです。

## 想定読者

- LINQは空気のように慣れ親しんでいる
- Pythonは月並みに書けはする

時代はPythonですからね [^1] 。うまく付き合わないといけません。C#が母語という人がPythonを書くシーンを想定しています。

## 環境

- .NET 8 / C# 12
- Python 3.12.0

Pythonについてはできるだけ第三者のライブラリを使わなくて済む方法という前提にします。 [itertools](https://docs.python.org/ja/3/library/itertools.html) など標準添付のモジュールはもちろん多用します。シンプルな答えが無いものは [more-itertools](https://more-itertools.readthedocs.io/en/stable/) の例も示しています。

LINQの多様なメソッド・オーバーロードすべてをカバーできてはいませんので悪しからず。また入力が空・ソートが同点の場合等々すべてのケースを網羅できているわけでももちろんありませんし、細かい仕様の差を見逃しているかもしれません。

## メソッド一覧

## Select

C#

```cs
int[] values = [1, 2, 3, 4, 5];
var plus1 = values.Select(x => x + 1);  // 2, 3, 4, 5, 6
```

Pythonでは定番のlist内包表記・ジェネレータ式のほかに、 [map](https://docs.python.org/ja/3/library/functions.html#map) を使うこともできます。戻り値はイテレータなのでLINQに使い心地が近いと言えます [^2] 。ラムダ式を指定するところもLINQ話者にはなじみやすいと言えます。Python界では内包表記のほうが好まれるようですが。

Python (リスト内包表記)

```cs
values = [1, 2, 3, 4, 5]
plus1 = [x + 1 for x in values]
print(plus1)  # [2, 3, 4, 5, 6]
```

Python (map)

```cs
values = [1, 2, 3, 4, 5]
plus1 = map(lambda x: x + 1, values)
print(plus1)  # <map object at 0x0000029C7097A770>
print(list(plus1))  # [2, 3, 4, 5, 6]
```

Python (ジェネレータ式)

```cs
values = [1, 2, 3, 4, 5]
plus1 = (x + 1 for x in values)
print(plus1)  # <generator object <genexpr> at 0x7f987b681930>
print(list(plus1))  # [2, 3, 4, 5, 6]
```

## Where

C#

```cs
int[] values = [1, 2, 3, 4, 5];
var moreThan2 = values.Where(x => x > 2);  // 3, 4, 5
```

こちらも、内包表記・ジェネレータ式のほかに [filter](https://docs.python.org/ja/3/library/functions.html#filter) を使うこともできます。map同様に戻り値はイテレータです。

Python (リスト内包表記)

```cs
values = [1, 2, 3, 4, 5]
more_than_2 = [x for x in values if x > 2]
print(more_than_2)  # [3, 4, 5]
```

Python (filter)

```cs
values = [1, 2, 3, 4, 5]
more_than_2 = filter(lambda x: x > 2, values)
print(more_than_2)  # <filter object at 0x000002B103889B40>
print(list(more_than_2))  # [3, 4, 5]
```

Python (ジェネレータ式)

```cs
values = [1, 2, 3, 4, 5]
more_than_2 = (x for x in values if x > 2)
print(more_than_2)  # <generator object <genexpr> at 0x7fba3209d930>
print(list(more_than_2))  # [3, 4, 5]
```

### itertools.filterfalse

ちなみに [itertools.filterfalse](https://docs.python.org/ja/3/library/itertools.html#itertools.filterfalse) を使えば、条件が偽となる要素を集めることができます。LINQではWhereの条件式を反転させるしかないですね [^3] 。

Python

```cs
import itertools

values = [1, 2, 3, 4, 5]
le_2 = itertools.filterfalse(lambda x: x > 2, values)
print(list(le_2))  # [1, 2]
```

### Select & Where

SelectとWhereは1回のクエリの中で一緒に使うことも多いと思います。LINQでは何回でもメソッドチェーンができますが、内包表記は複雑になってくると入れ子になるので可読性が厳しくなります。

おとなしく一時変数で受けて処理をばらすのが無難と前置きし、好みはあるかもしれませんが、walrus operatorによりifのところでSelect相当を書いてしまえるようになったので、以下のようにできる例はあります。LINQでいう `.Select().Where().Select()` 相当です。

Python

```cs
user_db = {
    1: "Taro",
    3: "Jiro",
    4: "Saburo",
    5: "Shiro",
}
query_ids = [1, 4]

# 入れ子の例
names = [name.upper() for name in (user_db.get(id) for id in query_ids) if name]
print(names)  # ['TARO', 'SABURO']

# walrus operatorを使う例
names = [name.upper() for id in query_ids if (name := user_db.get(id))]
print(names)  # ['TARO', 'SABURO']
```

---

脱線しますが、上のコード例はC#でもいまいちきれいにLINQで書けない例として知られています [^4] 。主な原因は、LINQとnullableの食い合わせが悪いことです。

C#

```cs
Dictionary<int, string> userDb = new(){
    [1] = "Taro",
    [3] = "Jiro",
    [4] = "Saburo",
    [5] = "Shiro",
};
int[] queryIds = [1, 4];

// 例1: 型が string? になるのが微妙。かといって .Select(s => s!) を付けるのはだるい。
var names1 = queryIds
    .Select(id => userDb.GetValueOrDefault(id)?.ToUpper())
    .Where(name => name is not null);
Console.WriteLine(string.Join(",", names1));  // TARO,SABURO

// 例2: SelectManyが奇抜だけどnullableの問題はない個人的ベスト案
var names2 = queryIds
    .SelectMany(id => userDb.TryGetValue(id, out var name) ? [name.ToUpper()] : Array.Empty<string>());
Console.WriteLine(string.Join(",", names2));  // TARO,SABURO
```

## ToArray, ToList

C#

```cs
int[] values = [1, 2, 3, 4, 5];
int[] array = values.Select(x => x * x).ToArray();  // 1, 4, 9, 16, 25
```

Python

```cs
values = [1, 2, 3, 4, 5]

list1 = [x * x for x in values]  # [1, 4, 9, 16, 25]
list2 = list(x * x for x in values)  # [1, 4, 9, 16, 25]
list3 = list(map(lambda x: x * x, values))  # [1, 4, 9, 16, 25]

tuple1 = tuple(x * x for x in values)  # (1, 4, 9, 16, 25)
```

## ToDictionary

C#

```cs
int[] values = [1, 2, 3];

var dict = values.ToDictionary(x => $"No.{x}", x => x);  // {'No.1': 1, 'No.2': 2, 'No.3': 3}
```

Python

```cs
values = [1, 2, 3]

d = {f"No.{x}": x for x in values}  # {'No.1': 1, 'No.2': 2, 'No.3': 3}
```

キーが重複する場合は `ToDictionary` は例外となりますが、dict内包表記は後ろの要素が勝ちます。

Python

```cs
# Aliceは消えます
d = {s[0]: s for s in ["Alice", "Alex", "Bob"]}    # {'A': 'Alex', 'B': 'Bob'}
```

## ToHashSet

C#

```cs
int[] values = [1, 2, 3, 4, 5];

var hashSet = values.Select(x => x / 3).ToHashSet();  // 0, 1
```

Python

```cs
values = [1, 2, 3, 4, 5]

set1 = {x // 3 for x in values}  # {0, 1}
set2 = set(x // 3 for x in values)  # {0, 1}
```

## SelectMany

SelectManyのリファレンスにあるサンプルコードを今風にしたものです。

C#

```cs
PetOwner[] petOwners = [
  new("Higa, Sidney", ["Scruffy", "Sam"]),
  new("Ashkenazi, Ronen", ["Walker", "Sugar"]),
  new("Price, Vernette", ["Scratches", "Diesel"])
];

var result = petOwners.SelectMany(o => o.Pets);
Console.WriteLine(string.Join("\n", result));

record PetOwner(string Name, string[] Pets);
```

出力結果

```cs
Scruffy
Sam
Walker
Sugar
Scratches
Diesel
```

Pythonでの平坦化には [itertools.chain\_from\_iterable](https://docs.python.org/3/library/itertools.html#itertools.chain.from_iterable) です。

Python

```cs
import itertools
from typing import NamedTuple

class PetOwner(NamedTuple):
    name: str
    pets: list[str]

pet_owners = [
  PetOwner("Higa, Sidney", ["Scruffy", "Sam"]),
  PetOwner("Ashkenazi, Ronen", ["Walker", "Sugar"]),
  PetOwner("Price, Vernette", ["Scratches", "Diesel"])
]

result = itertools.chain.from_iterable(o.pets for o in pet_owners)
print("\n".join(result))
```

出力結果

```cs
Scruffy
Sam
Walker
Sugar
Scratches
Diesel
```

ほかには、 [`more_itertools.flatten`](https://more-itertools.readthedocs.io/en/stable/api.html#more_itertools.flatten) を使うこともできます。SelectManyやchainよりも名前が直感的ですね。

リスト内包表記だけで書いてしまうこともできます。

```cs
result = [p for o in pet_owners for p in o.pets]
```

## Take, Skip

C#

```cs
var taken = Foo().Take(3);  // 1, 2, 3
var skipped = Foo().Skip(3);  // 4, 5
var both = Foo().Skip(1).Take(3);  // 2, 3, 4

IEnumerable<int> Foo()
{
    yield return 1;
    yield return 2;
    yield return 3;
    yield return 4;
    yield return 5;
}
```

対象がlistであれば、もちろんスライスでOKです。

Python

```cs
values = [1, 2, 3, 4, 5]

print(values[:3])  # [1, 2, 3]
print(values[3:])  # [4, 5]
print(values[1:4])  # [2, 3, 4]
```

Iterableとして処理したいなら、 [itertools.islice](https://docs.python.org/3/library/itertools.html#itertools.islice) が良いでしょう。

Python

```cs
import itertools
from typing import Generator

def foo() -> Generator[int, None, None]:
    yield 1
    yield 2
    yield 3
    yield 4
    yield 5

taken = itertools.islice(foo(), 3)  # Take(3)
skipped = itertools.islice(foo(), 3, None)  # Skip(3)
both = itertools.islice(foo(), 1, 4)  # Skip(1).Take(3)

print(taken)  # <itertools.islice object at 0x000001C6D968F150>
print(list(taken))  # [1, 2, 3]
print(list(skipped))  # [4, 5]
print(list(both))  # [2, 3, 4]
```

要素数を超える値を指定しても例外にならず、末尾で止まる点もTakeと同じです。

Python

```cs
over_taken = itertools.islice(foo(), 100)
print(list(over_taken))  # [1, 2, 3, 4, 5]
```

なお、 [more-itertools](https://more-itertools.readthedocs.io/en/stable/) を使ってよければ、 [take](https://more-itertools.readthedocs.io/en/stable/api.html#more_itertools.take) が用意されています。

Python

```cs
import more_itertools

more_itertools.take(3, foo())
```

## TakeLast

C#

```cs
int[] values = [1, 2, 3, 4, 5];

var taken = values.TakeLast(2);  // 4, 5
```

itertoolsのページに便利レシピ集があり、 `tail` というメソッドの例があります。以下抜粋です。これで解決ですね。

Python

```cs
def tail(n, iterable):
    return iter(collections.deque(iterable, maxlen=n))

values = [1, 2, 3, 4, 5]
print(list(tail(2, values)))  # [4, 5]
```

なお、これもmore-itertoolsに [`tail`](https://more-itertools.readthedocs.io/en/stable/api.html#more_itertools.tail) が定義済みです。

Python

```cs
import more_itertools

values = [1, 2, 3, 4, 5]
print(list(more_itertools.tail(2, values)))  # [4, 5]
```

## TakeWhile

C#

```cs
char[] chars = ['a', 'b', 'c', 'D'];

var taken1 = chars.TakeWhile(char.IsLower);  // 'a', 'b', 'c'
var taken2 = chars.TakeWhile(char.IsUpper);  // (列挙されない)
```

[itertools.takewhile](https://docs.python.org/3/library/itertools.html#itertools.takewhile) があります。

Python

```cs
import itertools

chars = ["a", "b", "c", "D"]

taken1 = itertools.takewhile(lambda c: c.islower(), chars)
taken2 = itertools.takewhile(lambda c: c.isupper(), chars)
print(list(taken1))  # ['a', 'b', 'c']
print(list(taken2))  # []
```

## SkipWhile

C#

```cs
char[] chars = ['a', 'b', 'c', 'D'];

var skipped1 = chars.SkipWhile(char.IsLower);  // 'D'
var skipped2 = chars.SkipWhile(char.IsUpper);  // 'a', 'b', 'c', 'D'
```

[itertools.dropwhile](https://docs.python.org/3/library/itertools.html#itertools.dropwhile) があります。こちらは名前がLINQと食い違うので注意。

Python

```cs
import itertools

chars = ["a", "b", "c", "D"]

skipped1 = itertools.dropwhile(lambda c: c.islower(), chars)
skipped2 = itertools.dropwhile(lambda c: c.isupper(), chars)
print(list(skipped1))  # ['D']
print(list(skipped2))  # ['a', 'b', 'c', 'D']
```

## First, FirstOrDefault

### 単に先頭の要素を取る(predicate無し)

C#

```cs
string[] strings = ["A", "B", "C"];
var f1 = strings.First();  // "A"
var f2 = strings.FirstOrDefault();  // "A"

string[] emptyStrings = [];
var f3 = emptyStrings.First();  // System.InvalidOperationException: 'Sequence contains no elements'
var f4 = emptyStrings.FirstOrDefault();  // null
```

list, tupleなら当然 `[0]` で終わりなので割愛し、Iterableの場合です。 [next](https://docs.python.org/ja/3/library/functions.html#next) が使えます。第2引数はイテレータが枯渇した場合のデフォルト値を指定でき、指定すればFirstOrDefault相当になります。指定しなければStopIterationの例外が発生しFirst相当と言えます。

Python

```cs
strings = ["A", "B", "C"]
empty_strings = []

print(next(iter(strings)))  # A
print(next(iter(strings), None))  # A
print(next(iter(empty_strings)))  # エラー (StopIteration)
print(next(iter(empty_strings), None))  # None
```

なお、 [more-itertools](https://more-itertools.readthedocs.io/en/stable/) を使えば、 [first](https://more-itertools.readthedocs.io/en/stable/api.html#more_itertools.first) が用意されています。

Python

```cs
import more_itertools

strings = ["A", "B", "C"]

print(more_itertools.first(iter(strings)))  # A
```

### 条件に合致した最初の要素を取る(predicateあり)

C#

```cs
string[] values = ["Alice", "Bob", "Mallory"];
var f1 = values.First(s => s.StartsWith("M"));  // Mallory
var f2 = values.FirstOrDefault(s => s.StartsWith("M"));  // Mallory
var f3 = values.First(s => s.StartsWith("Z"));  // System.InvalidOperationException: 'Sequence contains no matching element'
var f4 = values.FirstOrDefault(s => s.StartsWith("Z"));  // null
```

[itertoolsの便利レシピ集](https://docs.python.org/3/library/itertools.html#itertools-recipes) にて、 `first_true` というメソッドの例があります。以下抜粋です。

```python
def first_true(iterable, default=False, pred=None):
    return next(filter(pred, iterable), default)
```

これを使えば、FirstOrDefault相当は簡単に書けます。もしFirst相当、つまり見つからないとき例外にしたい場合は、 `first_true` の中のnext関数についてdefault指定を取り除きましょう。

Python

```cs
strings = ["Alice", "Bob", "Mallory"]

print(first_true(strings, default=None, pred=lambda s: s.startswith("M")))  # Mallory
print(first_true(strings, default=None, pred=lambda s: s.startswith("Z")))  # None
```

なお、 [more-itertools](https://more-itertools.readthedocs.io/en/stable/) を使えば、 [first\_true](https://more-itertools.readthedocs.io/en/stable/api.html#more_itertools.first_true) が用意されています [^5] 。仕様は上記first\_trueと同じです。

Python

```cs
import more_itertools

strings = ["Alice", "Bob", "Mallory"]

print(more_itertools.first_true(strings, default=None, pred=lambda s: s.startswith("M")))  # Mallory
```

## Last, LastOrDefault

### 単に末尾の要素を取る(predicate無し)

C#

```cs
string[] strings = ["A", "B", "C"];
var f1 = strings.Last();  // "C"
var f2 = strings.FirstOrDefault();  // "C"

string[] emptyStrings = [];
var f3 = emptyStrings.Last();  // System.InvalidOperationException: 'Sequence contains no elements'
var f4 = emptyStrings.LastOrDefault();  // null
```

First同様に、対象がリストならばインデックス指定で終わりです。  
以下、対象がIterableの場合の方法を3つ載せておきます。方法1と2はスマートですが、Iterableが空の場合は例外になります。その回避は面倒で、特に方法1ではOrDefault相当の処理は対応しにくいです。方法3はその点問題なく、LastOrDefaultに適した方法です。

Python

```cs
strings = ["Alice", "Bob", "Mallory"]

# 方法1
*_, last = iter(strings)
print(last)  # Mallory

# 方法2
from collections import deque
d = deque(iter(strings), maxlen=1)
print(d[0])  # Mallory

# 方法3
last = None  # 任意のデフォルト値を指定可
for last in iter(strings):
    pass
print(last)  # Mallory
```

ところでLINQは、例えば `IEnumerable<T>` の実体が `IList<T>` ならばインデックスアクセスに切り替えるような賢い最適化が入っています [^6] 。Pythonの上記3つのうち、少なくとも方法3はおそらく常に愚直に最後まで舐めますので、負荷に気を付けて使うべきと思います。

more-itertoolsにはズバリの [last](https://more-itertools.readthedocs.io/en/stable/api.html#more_itertools.last) が用意されています。LINQ同様に賢く、実体がSequenceなら `[-1]` を取り、もしくは `__reversed__` を持つなら逆順にして先頭を得るような実装になっています（ [ソースコード参照](https://more-itertools.readthedocs.io/en/stable/api.html#more_itertools.last) ）。使えるなら率先して使いましょう。

Python

```cs
import more_itertools

strings = ["Alice", "Bob", "Mallory"]

print(more_itertools.last(iter(strings)))  # Mallory
print(more_itertools.last([]))  # ValueError
print(more_itertools.last([], None))  # None
```

### 条件に合致した最後の要素を取る(predicateあり)

C#

```cs
string[] values = ["Alice", "Bob", "Matilda", "Mallory", "Charlie"];
var r1 = values.Last(s => s.StartsWith("M"));  // "Mallory"
var r2 = values.LastOrDefault(s => s.StartsWith("M"));  // "Mallory"

var r3 = values.Last(s => s.StartsWith("Z"));  // System.InvalidOperationException: 'Sequence contains no matching element'
var r4 = values.LastOrDefault(s => s.StartsWith("Z"));  // null
```

これについてはPythonでは関数一発で終わるような良い方法が見つかりませんでした。以下のような感じで、ここまでの知見の合わせ技のようにすれば求まりはします。LINQで例えると`.Where(predicate).Last()` 相当です。

Python

```cs
import more_itertools

strings = ["Alice", "Bob", "Matilda", "Mallory", "Charlie"]

print(more_itertools.last(s for s in strings if s.startswith("M")))  # Mallory
print(more_itertools.last(s for s in strings if s.startswith("M"), None))  # Mallory

print(more_itertools.last(s for s in strings if s.startswith("Z")))  # ValueError
print(more_itertools.last(s for s in strings if s.startswith("Z"), None))  # None
```

## Single,SingleOrDefault,

要素が1個だけの場合にその値を返します。 `Single()` については、0個や2個以上の場合に例外となります。 `SingleOrDefault()` については、0個の時に `default(T)` を返し、2個以上の時は同じく例外です。

C#

```cs
string[] values1 = ["A"];
string[] values2 = ["A", "B", "C"];
string[] values3 = [];

Console.WriteLine(values1.Single());  // "A"
Console.WriteLine(values2.Single());  // System.InvalidOperationException: Sequence contains more than one element
Console.WriteLine(array3.Single());  // System.InvalidOperationException: Sequence contains no elements

Console.WriteLine(values1.SingleOrDefault());  // "A"
Console.WriteLine(values2.SingleOrDefault());  // System.InvalidOperationException: Sequence contains more than one element
Console.WriteLine(values3.SingleOrDefault());  // (null)
```

### 標準ライブラリ縛り

Pythonでは、標準の関数・ライブラリでストレートに再現する方法は見つけられませんでした。1つ目の案として、その場でパッとやるならunpackが楽かもしれません。

Python (unpack案)

```cs
values = ["A"]
# values = ["A", "B", "C"]
# values = []

# 要素数が0だとここで例外: ValueError: not enough values to unpack (expected at least 1, got 0)
first, *rest = values
if not rest:
    print("成功")
```

ただしunpackは最後まで列挙してしまうので、それが不都合なケースでは使えません。読むのを2回だけに抑えたいなら自作が良さそうです。

Python (single自作案)

```cs
from collections.abc import Iterable
from typing import TypeVar

T = TypeVar('T')

def single(iterable: Iterable[T]) -> T:
    it = iter(iterable)
    result = next(it)
    try:
        next(it)
    except StopIteration:
        return result
    raise ValueError("Iterable has more than one item")

print(single(["A"]))  # A
```

### more\_itertools.one, only

さてここで困ったときのmore-itertools様、 [`one`](https://more-itertools.readthedocs.io/en/stable/api.html#more_itertools.one) や [`only`](https://more-itertools.readthedocs.io/en/stable/api.html#more_itertools.only) があります。基本的にoneがSingle、onlyがSingleOrDefaultに相当します。 `more_itertools.one` の実装は、上の自前実装 `single` と同じような実装になっています ([参考](https://more-itertools.readthedocs.io/en/stable/_modules/more_itertools/more.html#one))。

more-itertools案 (one)

```cs
import more_itertools

data1 = ["A"]
data2 = ["A", "B", "C"]
data3 = []

print(more_itertools.one(data1))  # 1
print(more_itertools.one(data2))  # ValueError: Expected exactly one item in iterable, but got 'A', 'B', and perhaps more.
print(more_itertools.one(data3))  # ValueError: too few items in iterable (expected 1)
```

more-itertools案 (only)

```cs
import more_itertools

data1 = ["A"]
data2 = ["A", "B", "C"]
data3 = []

print(more_itertools.only(data1))  # 1
print(more_itertools.only(data2))  # ValueError: Expected exactly one item in iterable, but got 'A', 'B', and perhaps more.
print(more_itertools.only(data3))  # None
```

なお、predicate付きのSingle/SingleOrDefaultオーバーロードについては、Lastのとき同様に自分で絞り込んでから処理にかけることになります。

## Any, All

### 要素の有無を確認する(predicateなし)

C#

```cs
int[] values1 = [];
int[] values2 = [1, 2, 3];

Console.WriteLine(values1.Any());  // False
Console.WriteLine(values2.Any());  // True
```

Pythonですが、listやtupleあたりであれば、言うまでもなく直接ifで指定すればよいだけでしょう。ちなみに `len(array) > 0` のように判定するのはPEP8的には無しらしいです。

Python

```cs
values1 = []
values2 = [1, 2, 3]

if values1:
    print("values1 is not empty")
if values2:
    print("values2 is not empty")
    
# bool値として欲しい場合
print(bool(values1))  # False
print(bool(values2))  # True
```

iterableとして確認したい場合は、 [`any`](https://docs.python.org/ja/3/library/functions.html#any) が使えます。  
ここでもしiterator等を渡す場合、確認のため読み進められるため、何度も処理すると結果が変わることに注意が必要です。この例に限りませんが。

Python

```cs
a = [1]
it = iter(a)

print("not empty" if any(it) else "empty")  # not empty
print("not empty" if any(it) else "empty")  # empty
```

### 条件に合致した要素の有無を判定(predicateあり)

C#

```cs
int[] values = [1, 2, 3];

Console.WriteLine(values.Any(x => x >= 3));  // True
Console.WriteLine(values.Any(x => x > 3));  // False

Console.WriteLine(values.All(x => x <= 3));  // True
Console.WriteLine(values.All(x => x <= 2));  // False
```

Pythonでは [`any`](https://docs.python.org/ja/3/library/functions.html#any) や [`all`](https://docs.python.org/ja/3/library/functions.html#all) にジェネレータ式を渡すのが良いと思います。  
私はたまに間違えて `all(x for x in values if x > 3)` のように後ろのifに条件を書いてしまうのですが、こうすると列挙結果から欠けてしまって正しい出力にならないので注意です。

Python

```cs
values = [1, 2, 3]

print(any(x >= 3 for x in values))  # True
print(any(x > 3 for x in values))  # False

print(all(x <= 3 for x in values))  # True 
print(all(x <= 2 for x in values))  # False
```

### 空のシーケンスに対する結果

C#もPythonも同様で、predicateに関わらず、空の場合は `Any` はFalse、 `All` はTrueです。

C#

```cs
int[] emptyValues = [];

Console.WriteLine(emptyValues.Any(_ => true));  // False

Console.WriteLine(emptyValues.All(_ => true));  // True
```

Python

```cs
print(any([]))  # False
print(all([]))  # True
```

## OrderBy, OrderByDescending, Order, OrderDescending

### 単純なソート (keySelector無し)

.NET 8で追加された"By"無しの `Order` や `OrderDescending` 相当の話です。従来で言うと `OrderBy(x => x)` のように書いていたものです。

C#

```cs
int[] values = [7, 3, 1, 5, 2, 6, 4];

Show(values.Order());  // 1,2,3,4,5,6,7
Show(values.OrderBy(x => x));  // 1,2,3,4,5,6,7
Show(values.OrderDescending());  // 7,6,5,4,3,2,1
Show(values.OrderByDescending(x => x));  // 7,6,5,4,3,2,1

void Show<T>(IEnumerable<T> enumerable) => Console.WriteLine(string.Join(",", enumerable));
```

Pythonでは [`sorted`](https://docs.python.org/ja/3/library/functions.html#sorted) ですね。Descending(降順)にしたければ `reverse=True` を指定します。

Python

```cs
values = [7, 3, 1, 5, 2, 6, 4]

print(sorted(values))  # [1, 2, 3, 4, 5, 6, 7]
print(sorted(values, reverse=True))  # [7, 6, 5, 4, 3, 2, 1]
```

違いとして、.NETのOrder系は戻り値が `IEnumerable<T>` (正確には `IOrderedEnumerable<T>`)ですが、Pythonのsortedはリストで返ります。

C#にて最大・最小の要素が欲しい時に `.OrderBy(...).First()` のように書くシーンが多かったと思います（今は後述の `MinBy` `MaxBy` により過去のものですが [^7] ）。LINQのこのセットは最適化の特別対応が入っています([参考](https://qiita.com/RyotaMurohoshi/items/5b515bf2ee544cc1b016))。  
しかしPythonの `sorted` はリストで返るということは、もし先頭しか要らなくても全要素並び変えてしまうと考えられます。LINQの感覚で `sorted(my_list)[0]` のように書くのは避けて、後述する `min` `max` を使いましょう。

listであれば [`list.sort`](https://docs.python.org/ja/3/library/stdtypes.html#list.sort) も使えます。key指定などの仕様は同じように使えます。違いとして、戻り値はなくインプレースに置き換わるので、巨大なリストの場合は効率が良いと思います。

### キーを指定したソート (keySelector有り)

C#

```cs
string[] names = ["Alice", "Bob", "Caroline"];

Show(names.OrderBy(s => s.Length));  // Bob,Alice,Caroline
Show(names.OrderByDescending(s => s.Length));  // Caroline,Alice,Bob

void Show<T>(IEnumerable<T> enumerable) => Console.WriteLine(string.Join(",", enumerable));
```

[`sorted`](https://docs.python.org/ja/3/library/functions.html#sorted) では `key=` の指定により同様のことが行えます。

Python

```cs
print(sorted(names, key=len))  # ['Bob', 'Alice', 'Caroline']
print(sorted(names, key=len, reverse=True))  # ['Caroline', 'Alice', 'Bob']

# lambdaで指定する場合
print(sorted(names, key=lambda s: len(s)))  # ['Bob', 'Alice', 'Caroline']
```

### ThenBy, ThenByDescending

第2、第3... の順序尺度を指定したい時は、使いやすいのはkey関数で優先順に値を並べたタプルを返す方法です。

Python

```cs
from typing import NamedTuple

class Point(NamedTuple):
    x: int
    y: int

points = [
    Point(5, 50),
    Point(1, 10),
    Point(2, 20),
    Point(2, 0),
    Point(4, 40),
]

print("x昇順ソート")
print(*sorted(points, key=lambda p: p.x), sep='\n')
print("x,y昇順ソート")
print(*sorted(points, key=lambda p: (p.x, p.y)), sep='\n')
```

2,3番目が変わっているのがわかります。

出力結果

```cs
x昇順ソート
Point(x=1, y=10)
Point(x=2, y=20)
Point(x=2, y=0)
Point(x=4, y=40)
Point(x=5, y=50)
x,y昇順ソート
Point(x=1, y=10)
Point(x=2, y=0)
Point(x=2, y=20)
Point(x=4, y=40)
Point(x=5, y=50)
```

`OrderBy().ThenByDescending()` のように昇順降順が入り混じる場合は、Pythonではkey関数の値にマイナスを付ける等して対応することになります。 `(p.x, -p.y)` のような感じです。

他の方法として、Pythonのドキュメントでも示されるようにkey指定を変えながら複数回 `sorted` をかける方法もあります。優先度が低い尺度から先に実行します。

## Min, Max

C#

```cs
int[] values = [7, 3, 1, 5, 2, 6, 4];
Console.WriteLine(values.Min());  // 1
Console.WriteLine(values.Max());  // 7

// 空は例外
int[] emptyValues = [];
Console.WriteLine(emptyValues.Min());  // System.InvalidOperationException: Sequence contains no elements
Console.WriteLine(emptyValues.Max());  // 同上
```

わかりやすく、 [min](https://docs.python.org/ja/3/library/functions.html#min) や [max](https://docs.python.org/ja/3/library/functions.html#max) が対応します。

Python

```cs
values = [7, 3, 1, 5, 2, 6, 4]
print(min(values))  # 1
print(max(values))  # 7

# 空は例外なのも同じ
print(min([]))  # ValueError: min() iterable argument is empty
# 空の場合のデフォルト値を指定可
print(min([], default="空だよ"))  # "空だよ"
```

## MinBy, MaxBy

上でも述べた `.OrderBy().First()` パターンです。

C#

```cs
string[] names = ["Alice", "Bob", "Caroline"];

Console.WriteLine(names.MinBy(s => s.Length));  // Bob
Console.WriteLine(names.MaxBy(s => s.Length));  // Caroline
```

Pythonではこれに [min](https://docs.python.org/ja/3/library/functions.html#min) や [max](https://docs.python.org/ja/3/library/functions.html#max) が対応します。

Python

```cs
names = ["Alice", "Bob", "Caroline"]

print(min(names, key=len))  # Bob
print(max(names, key=len))  # Caroline

# lambdaで指定する場合
print(min(names, key=lambda s: len(s)))  # Bob
```

## Reverse

C#

```cs
var range = Enumerable.Range(0, 10);
var reversed = range.Reverse();
Console.WriteLine(string.Join(",", reversed));  // 9,8,7,6,5,4,3,2,1,0
```

### スライス

まずそもそも、対象がlistなどスライスに対応していれば、スライスで逆順にするのは1つの常套手段です。

Python

```cs
values = [1, 2, 3, 4, 5]

print(values[::-1])  # [5, 4, 3, 2, 1]
```

### reversed

スライス以外とすると、逆順化はPythonでは [`reversed`](https://docs.python.org/ja/3/library/functions.html#reversed) が対応するわけですが、どんなiterableでも可能なわけではありません。以下いずれかに対応している必要があります。当てはまらないときは、予めlist等として全要素をメモリに置いたうえで逆転させることになろうかと思います。

- [`__reversed__()`](https://docs.python.org/ja/3/reference/datamodel.html#object.__reversed__) メソッドを持つ
- sequenceプロトコルに対応 ([`__len__()`](https://docs.python.org/ja/3/reference/datamodel.html#object.__len__) と [`__getitem__()`](https://docs.python.org/ja/3/reference/datamodel.html#object.__getitem__) を持つ)

戻り値はiteratorです。

Python

```cs
values = [1, 2, 3, 4, 5]

print(reversed(values))  # <list_reverseiterator object at 0x7f8733452b00>
print(list(reversed(values)))  # [5, 4, 3, 2, 1]

print(reversed(x*x for x in values))  # TypeError: 'generator' object is not reversible
```

### more\_itertools.always\_reversible

[`more_itertools.always_reversible`](https://www.notion.so/Circuit-bc-sameimage-matcher-686fbc7c7609405fae20771543d55bc4) は、ジェネレータ式といった逆順に出来ないものを突っ込んでも逆順可能にしてくれる代物です。

Python

```cs
print(*always_reversible(x for x in range(3)))  # 2 1 0
```

なにかすごいことをやっているわけでは全くなく、 `reversed(list(iterable))` を返しているだけだったりします（ [参考](https://more-itertools.readthedocs.io/en/stable/_modules/more_itertools/more.html#always_reversible) ）。

## Sum

C#

```cs
var range = Enumerable.Range(1, 10);
int[] empty = Array.Empty<int>();

Console.WriteLine(range.Sum());  // 55
Console.WriteLine(empty.Sum()); // 0
```

[`sum`](https://docs.python.org/ja/3/library/functions.html#sum) が使えます。

Python

```cs
my_range = range(1, 11)

print(sum(my_range))  # 55
print(sum([]))  # 0
```

## Count

C#

```cs
var range = Enumerable.Range(1, 10);

Console.WriteLine(range.Count());  // 10
Console.WriteLine(range.Count(i => i % 3 == 1));  // 4
```

単純な要素数であればもちろん [`len`](https://docs.python.org/ja/3/library/functions.html#len) が第一候補です。しかし本記事では繰り返しになりますが、ジェネレータ等の非対応なオブジェクトがあります。この場合は [`sum`](https://docs.python.org/ja/3/library/functions.html#sum) を使うのが簡単です。条件付きのパターンはTrueが1、Falseが0として加算されることを利用しており、慣れないとやや気持ち悪いかもしれません。

Python

```cs
# lenが使えるとき
print(len(range(1, 11)))  # 10

# lenが使えないとき
it = iter(range(1, 11))
print(sum(1 for _ in it))  # 10

# 条件に合致する数を調べるとき
print(sum(x % 3 == 1 for x in range(1, 11)))  # 4
```

ほかには [`more_itertools.ilen`](https://more-itertools.readthedocs.io/en/stable/api.html#more_itertools.ilen) を使うこともできます。

Python

```cs
import more_itertools

print(more_itertools.ilen(x for x in range(1, 11) if x % 3 == 1))  # 4
```

## Average

C#

```cs
var data = Enumerable.Range(1, 10);

Console.WriteLine(data.Average());  // 5.5
```

[statistics](https://docs.python.org/3/library/statistics.html#module-statistics) というモジュールが標準で用意されており、 [`mean`](https://docs.python.org/3/library/statistics.html#statistics.mean) 関数があります。  
そのほか、numpyを使ってよければ [`numpy.mean`](https://numpy.org/doc/stable/reference/generated/numpy.mean.html) や [`numpy.average`](https://numpy.org/doc/stable/reference/generated/numpy.average.html) が相当します。

```python
import statistics

data = range(1, 11)
print(statistics.mean(data))  # 5.5
```

## Aggregate

C#

```cs
Point[] points = [
    new (1, 10),
    new (2, 11),
    new (3, 12),
    new (4, 13),
];

var sum = points.Aggregate((a, b) => new Point(a.X + b.X, a.Y + b.Y));
Console.WriteLine(sum);  // Point { X = 10, Y = 46 }

record Point(int X, int Y);
```

Pythonでは [`functools.reduce`](https://docs.python.org/ja/3/library/functools.html#functools.reduce) が有効です。

Python

```cs
from functools import reduce
from typing import NamedTuple

class Point(NamedTuple):
    x: int
    y: int

points = [
    Point(1, 10),
    Point(2, 11),
    Point(3, 12),
    Point(4, 13),
]

result = reduce(lambda a, b: Point(a.x + b.x, a.y + b.y), points)
print(result)  # Point(x=10, y=46)
```

説明は省略しますが、オプションの `initializer` パラメータもあり、 `Aggregate` の `seed` パラメータに相当します。

## Zip

C#

```cs
int[] values1 = [1, 2, 3];
int[] values2 = [4, 5, 6];
int[] values3 = [7, 8, 9, 10];

var zip12 = values1.Zip(values2);
var zip13 = values1.Zip(values3);

Show(zip12);  // (1, 4), (2, 5), (3, 6)
Show(zip13);  // (1, 7), (2, 8), (3, 9)

static void Show<T>(IEnumerable<T> enumerable) => Console.WriteLine(string.Join(", ", enumerable));
```

Pythonでは [`zip`](https://docs.python.org/ja/3/library/functions.html#zip) 関数です。長さが違う場合に短いほうに合わせられる仕様も同じです。 `strict=True` を指定すると異なる長さの時に例外になり、おすすめです。これはLINQには無さそうです。

Python

```cs
values1 = [1, 2, 3]
values2 = [4, 5, 6]
values3 = [7, 8, 9, 10]

result12 = zip(values1, values2)
result13 = zip(values1, values3)
print(list(result12))  # [(1, 4), (2, 5), (3, 6)]
print(list(result13))  # [(1, 7), (2, 8), (3, 9)]

result13 = zip(values1, values3, strict=True)
# ValueError: zip() argument 2 is longer than argument 1
```

そのほかの違いとして、LINQのZipは取れる `IEnumerable<T>` の数が2つか3つ限定ですが、Pythonのzipはそれ以上でも可能です。

## GroupBy

ユーザの年齢(Age)によって仕分ける例です。

C#

```cs
User[] users = [
    new("Alice", 14),
    new("Bob", 33),
    new("Carol", 25),
    new("Dave", 52),
    new("Ellen", 17),
    new("Frank", 67),    
];

var groups = users.GroupBy(u => u.Age < 20);
foreach(var g in groups)
{
    Console.WriteLine("##### IsChild={0} ##### ", g.Key);
    Console.WriteLine(string.Join("\n", g));
    Console.WriteLine();
}

record User(string Name, int Age);
```

出力結果

```cs
##### IsChild=True ##### 
User { Name = Alice, Age = 14 }
User { Name = Ellen, Age = 17 }

##### IsChild=False ##### 
User { Name = Bob, Age = 33 }
User { Name = Carol, Age = 25 }
User { Name = Dave, Age = 52 }
User { Name = Frank, Age = 67 }
```

### itertools.groupbyによる例

Pythonには [`itertools.groupby`](https://docs.python.org/3/library/itertools.html#itertools.groupby) というズバリの関数があるのですが、利用には注意が必要で、一般に **入力はソート済みである必要** があります。

Python

```cs
import itertools
from typing import NamedTuple

class User(NamedTuple):
    name: str
    age: int

users = [
    User("Alice", 14),
    User("Bob", 33),
    User("Carol", 25),
    User("Dave", 52),
    User("Ellen", 17),
    User("Frank", 67),    
]

users = sorted(users, key=lambda u: u.age)

groups = itertools.groupby(users, key=lambda u: u.age < 20)
for key, group in groups:
    print(f"##### is_child={key} #####")
    print("\n".join(str(u) for u in group))
    print()
```

出力結果

```cs
##### is_child=True #####
User(name='Alice', age=14)
User(name='Ellen', age=17)

##### is_child=False #####
User(name='Carol', age=25)
User(name='Bob', age=33)
User(name='Dave', age=52)
User(name='Frank', age=67)
```

もしこの例で `sorted` を取り除くと、以下のような結果になりました。4グループ発生します。

出力結果(ソートなし)

```cs
##### is_child=True #####
User(name='Alice', age=14)

##### is_child=False #####
User(name='Bob', age=33)
User(name='Carol', age=25)
User(name='Dave', age=52)

##### is_child=True #####
User(name='Ellen', age=17)

##### is_child=False #####
User(name='Frank', age=67)
```

### more\_itertools.bucket による例

`itertools.groupby` はソートを忘れてはまりやすいため、 [`more_itertools.bucket`](https://more-itertools.readthedocs.io/en/stable/api.html#more_itertools.bucket) の方が使いやすいのではないかと考えます。若干使い方は異なるため注意です。

Python

```cs
import more_itertools

users = [...]  # 省略

bucket = more_itertools.bucket(users, key=lambda u: u.age < 20)

print(list(bucket))  # [True, False]

print("\n".join(str(u) for u in bucket[True]))
# User(name='Alice', age=14)
# User(name='Ellen', age=17)

print("\n".join(str(u) for u in bucket[False]))
# User(name='Bob', age=33)
# User(name='Carol', age=25)
# User(name='Dave', age=52)
# User(name='Frank', age=67)
```

## Chunk

C#

```cs
var chunks = Enumerable.Range(1, 10).Chunk(3);
foreach(var c in chunks)
{
    Console.WriteLine(string.Join(",", c));
}
```

出力結果

```cs
1,2,3
4,5,6
7,8,9
10
```

### itertools.batched

Python 3.12 から、 [`itertools.batched`](https://docs.python.org/3/library/itertools.html#itertools.batched) が追加されています。

Python

```cs
import itertools

chunks = itertools.batched(range(1, 11), 3)
print(list(chunks))
# [(1, 2, 3), (4, 5, 6), (7, 8, 9), (10,)]
```

### more\_itertools.batched

`itertools.batched` が使えない場合は [`more_itertools.batched`](https://more-itertools.readthedocs.io/en/stable/api.html#more_itertools.batched) が良いでしょう。仕様は同じです。

Python

```cs
import more_itertools

chunks = more_itertools.batched(range(1, 11), 3)
print(list(chunks))
# [(1, 2, 3), (4, 5, 6), (7, 8, 9), (10,)]
```

[`more_itertools.chunked`](https://more-itertools.readthedocs.io/en/stable/api.html#more_itertools.chunked) というのもあります。batchedと差がほとんどないように見えます。  
わかる差としては、各要素がtupleかlistかという点と、 `strict` パラメータがある点です。 `strict=True` の場合、長さが割り切れないときに例外となります。

## ElementAt, ElementAtOrDefault

C#

```cs
var fib = Fibonacci();
Console.WriteLine(fib.ElementAt(2));  // 2
Console.WriteLine(fib.ElementAt(3));  // 3
Console.WriteLine(fib.ElementAt(4));  // 5
Console.WriteLine(fib.ElementAt(50));  // 20365011074

IEnumerable<long> Fibonacci()
{
    for (long c = 1, n = 1; ; (c, n) = (n, c + n))
        yield return c;
}
```

リストの添え字指定で済むケースは当然それを使うとして、 [itertoolsのレシピ集](https://docs.python.org/ja/3/library/itertools.html#itertools-recipes) に `nth` があります。 `next` のdefault指定次第で、OrDefault動作の有無を再現できます。下記の状態ではデフォルトNoneなのでOrDefault相当です。

nthの定義

```cs
import itertools

def nth(iterable, n, default=None):
    "Returns the nth item or a default value"
    return next(itertools.islice(iterable, n, None), default)
```

Python

```cs
def fibonacci():
    c, n = 1, 1
    while True:
        yield c
        c, n = n, c + n

print(nth(fibonacci(), 2))  # 2
print(nth(fibonacci(), 3))  # 3
print(nth(fibonacci(), 4))  # 5
print(nth(fibonacci(), 50))  # 20365011074
```

例によって、more-itertoolsにはこの [`nth`](https://more-itertools.readthedocs.io/en/stable/api.html#more_itertools.nth) がそのまま取り入れられています。

## Concat

C#

```cs
int[] a = [1, 2, 3];
int[] b = [4, 5, 6];

var concat = a.Concat(b);
Console.WriteLine(string.Join(",", concat));  // 1,2,3,4,5,6
```

[`itertools.chain`](https://docs.python.org/ja/3/library/itertools.html#itertools.chain) です。

Python

```cs
import itertools

a = [1, 2, 3]
b = [4, 5, 6]

concat = itertools.chain(a, b)
print(list(concat))  # [1, 2, 3, 4, 5, 6]
```

## Append, Prepend

C#

```cs
int[] a = [1, 2, 3];

Console.WriteLine(string.Join(",", a.Append(100)));  // 1,2,3,100
Console.WriteLine(string.Join(",", a.Prepend(100)));  // 100,1,2,3
```

Concat同様に [`itertools.chain`](https://docs.python.org/ja/3/library/itertools.html#itertools.chain) で行うのがシンプルかと思いました。

Python

```cs
import itertools

a = [1, 2, 3]

append = itertools.chain(a, [100])
print(list(append))  # [1, 2, 3, 100]
prepend = itertools.chain([100], a)
print(list(prepend))  # [100, 1, 2, 3]
```

なおPrependについては [itertoolsレシピ](https://docs.python.org/ja/3/library/itertools.html#itertools-recipes) に `prepend` が載っており、more-itertoolsにも [`more_itertools.prepend`](https://more-itertools.readthedocs.io/en/stable/api.html#more_itertools.prepend) として用意があります。

## Intersect, Union

C#

```cs
int[] values1 = [1, 1, 2];
int[] values2 = [2, 3];

var intersection = values1.Intersect(values2);  // 2
var union = values1.Union(values2);  // 1, 2, 3
```

普通にsetとしてくっつけるしか思いつかず。

Python

```cs
values1 = [1, 1, 2]
values2 = [2, 3]

intersection = set(values1) & set(values2)  # {2}
union = set(values1) | set(values2)   # {1, 2, 3}
```

## Distinct

C#

```cs
int[] values = [1, 2, 3, 2, 1];
var distinct = values.Distinct();

Console.WriteLine(string.Join(",", distinct));  // 1,2,3
```

シンプルにsetへ。

Python

```cs
values = [1, 2, 3, 2, 1]

print(set(values))  # {1, 2, 3}
```

## DistinctBy

わかりやすい方法は思いついていません（というか記事が長くなって力尽きているともいう）。

以下参考になります。

他の集合演算のBy系も調査中...

## Except

他の集合演算系のLINQもそうですが、特に操作しているつもりがない `1` の重複が取り除かれる点に注目です。

C#

```cs
int[] a = [1, 1, 2, 3, 5];
int[] b = [2];
int[] c = [100];

Console.WriteLine(string.Join(",", a.Except(b)));  // 1,3,5
Console.WriteLine(string.Join(",", a.Except(c)));  // 1,2,3,5
```

Pythonではやっぱりsetです。

Python

```cs
a = [1, 1, 2, 3, 5]
b = [2]
c = [100]

print(set(a) - set(b))  # {1, 2, 3}
print(set(a) - set(c))  # {1, 2, 3, 5}
```

## DefaultIfEmpty

C#

```cs
int[] a = [100, 200, 300];
int[] b = [];

Show(a.DefaultIfEmpty());  // 100, 200, 300
Show(b.DefaultIfEmpty());  // 0

static void Show<T>(IEnumerable<T> enumerable) => Console.WriteLine(string.Join(", ", enumerable));
```

Pythonでは一発でやる方法を見つけられず、こんな感じでどうでしょうか。

Python

```cs
import more_itertools
from collections.abc import Iterable
from typing import TypeVar

T = TypeVar('T')

def default_if_empty(iterable: Iterable[T], default: T) -> Iterable[T]:
    head, iterable = more_itertools.spy(iterable)
    return iterable if head else [default]

a = [100, 200, 300]
b = []

print(list(default_if_empty(a, 0)))  # [100, 200, 300]
print(list(default_if_empty(b, 0)))  # [0]
```

## 参考: イテラブルとイテレータ

最後にやや脱線しますが注意点です。 [iterable](https://docs.python.org/ja/3/glossary.html#term-iterable) と [iterator](https://docs.python.org/ja/3/glossary.html#term-iterator) は、以下のように.NETと対応する理解で基本的には正しいはずです。

- Pythonのiterableは、C# (.NET) でいうところの [`IEnumerable`](https://learn.microsoft.com/ja-jp/dotnet/api/system.collections.generic.ienumerable-1?view=net-8.0)
- Pythonのiteratorは、C# (.NET) でいうところの [`IEnumerator`](https://learn.microsoft.com/ja-jp/dotnet/api/system.collections.generic.ienumerator-1?view=net-8.0)

ただし1つ明確に違うのは、 **Pythonのiteratorもまたiterableである** ということです。

Python

```cs
# iteratorを得てからforで回すことができる
it = iter([1, 2, 3])
for i in it:
    print(i)
```

C#ではこれに相当することはできません。（それはそうというか、話が逆な気がしますからね。）

C#

```cs
int[] array = [1, 2, 3];
var enumerator = array.GetEnumerator();

// foreach statement cannot operate on variables of type 'IEnumerator' because 'IEnumerator' does not contain a public instance or extension definition for 'GetEnumerator'
foreach (var i in enumerator)
{
    Console.WriteLine(i);
}
```

よって、 `Iterable` として型を付けると、そこにはlist, tuple, rangeのような何度でも再列挙可能なオブジェクトが来るかもしれないし、またはIterator、ジェネレータ式、ファイルオブジェクトといった列挙が一度きりのモノが来るかもしれないのです。

先に結論として、この節で述べたいことは、 **Pythonのiterableを受け付ける処理では、それを1回だけ舐めるよう実装するべき** ということです [^8] 。C#に慣れた人向けに言えば、PythonのIterableは「Seek不能なStream」と考えるのがわかりやすいかもしれません。

つまづく例を挙げます。IEnumerable/Iterableの最初の3つの要素(0~2番目)と、続く3つの要素(3~5番目)を得てprintするコードです。

C#

```cs
Foo(Enumerable.Range(0, 10).ToArray());
Foo(Enumerable.Range(0, 10));

void Foo(IEnumerable<int> numbers)
{
    var first3 = numbers.Take(3);
    var next3 = numbers.Skip(3).Take(3);
    Console.WriteLine(string.Join(", ", first3));
    Console.WriteLine(string.Join(", ", next3));
}

/* 出力
// 引数がint[]のとき
0, 1, 2
3, 4, 5
// 引数がIEnumerable<int>のとき
0, 1, 2
3, 4, 5
*/
```

これをPythonで書くとき、以下のようにしてしまうと混乱を生むことになります。

Python

```cs
import itertools
from collections.abc import Iterable

def foo(numbers: Iterable[int]) -> None:
    # (numbersの長さは問題ない仮定です)
    
    first_3 = itertools.islice(numbers, 3)  # Take(3)
    next_3 = itertools.islice(numbers, 3, 6)  # Skip(3).Take(3)

    print(list(first_3))
    print(list(next_3))

foo(range(10))
foo(x for x in range(10))

"""
出力
range(10)を指定したとき
[0, 1, 2]
[3, 4, 5]
ジェネレータ式を指定したとき: 結果が変わる
[0, 1, 2]
[6, 7, 8]
"""
```

この問題は、Pythonだけ気をつけろという話ではなく、実際はC#のコードもこの例はあまりよくありません。無駄な再列挙処理が走るためで、列挙対象によってはわかりにくいバグの元になるため、例えばReSharperを使っていれば警告が入ります。とはいえ、もし対象が `IEnumerable<T>` ではなく `T[]` や `List<T>` 等だとしたら特段実害はないありがちなコードでしょう。Pythonはより実害に出くわしやすいと言えます。

Pythonのtypingにあたって「 `list` として型を付けるのはイケてない。列挙さえできればよいのだから `Iterable` としよう」と安易に調子をこくとこの罠を踏むかもしれません。

今回の例についてIterableで受ける仕様を維持したければ、一案としては以下のように対イテレータの処理として統一したうえで、1回で舐め上げるよう変えるのが良いと思います。たぶん。

Python

```cs
def foo(numbers: Iterable[int]) -> None:
    numbers_it = iter(numbers)
    first_3 = itertools.islice(numbers_it, 3)  # Take(3)
    next_3 = itertools.islice(numbers_it, 3)  # Skip(3).Take(3)
    ...
```

または、「すこし読みたいけど、もとのiterableも維持したい」という場合は、 [`itertools.tee`](https://docs.python.org/ja/3/library/itertools.html#itertools.tee) 、 [`more_itertools.spy`](https://more-itertools.readthedocs.io/en/stable/api.html#more_itertools.spy) 、 [`more_itertools.peekable`](https://more-itertools.readthedocs.io/en/stable/api.html#more_itertools.peekable) などが便利です。spyの例は前述 `DefaultIfEmpty` にあるので、以下はpeekableの例です。 [^9]

Python (peekableの例)

```cs
import more_itertools

numbers = iter(range(10))
p = more_itertools.peekable(numbers)

first_3 = p[:3]  # Take(3)
next_3 = p[3:6]  # Skip(3).Take(3)
print(first_3)  # [0, 1, 2]
print(next_3)  # [3, 4, 5]
print(list(p))  # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

## おわりに

- 基本的に表現力はお互い遜色ありません。ただ、配列にドットを打てばすぐ全てのカタログが出てくるLINQに比べ、Pythonではあちこちに分散しているし方法も一貫してはいないので、この手引きは意義があるんじゃないでしょうか。自分に。
	- LINQを基準にすると「分散している」と見えますが、Pythonの中で見ればそうとも限らない点に留意です。
	- LINQとジェネレータ式（内包表記）の決定的な差は「補完の効きやすさ」と感じます。C#の神髄はこれですよね。
- しかしそれにしてもC#のサンプルコードで配列を使うと、出力 (`foreach` なり `string.Join` なり) がとてもだるい問題。20年待っています。改善求む！Main関数すら無くした最近のこの流れならできるでしょう。

脚注

11

[^1]: C# Advent Calendarを読む層には殴られそう

[^2]: 内包表記の例についても、ジェネレータ式すなわち内包表記の括弧を()に変えれば同じです

[^3]: ここの例では明らかに条件式を反転させた方がわかりやすいわけですが

[^4]: 個人的に

[^5]: 見つからないとき例外にはできないようです

[^6]: .NET Frameworkの実装: [https://github.com/microsoft/referencesource/blob/51cf7850defa8a17d815b4700b67116e3fa283c2/System.Core/System/Linq/Enumerable.cs#L1090](https://github.com/microsoft/referencesource/blob/51cf7850defa8a17d815b4700b67116e3fa283c2/System.Core/System/Linq/Enumerable.cs#L1090)

[^7]: MinBy, MaxByはnullableが返るため、微妙なところで従来と完全互換な書き換えにはならないですが

[^8]: 後述するように、C#のIEnumerable<T>も本来その点同様ではあります。つまづくシーンはより少ないですが。

[^9]: peekableは `peek()` のみならず `__getitem__` も実装しており、インデックスやスライスでの操作が可能です。