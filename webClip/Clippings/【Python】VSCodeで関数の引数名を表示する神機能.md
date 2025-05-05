---
title: 【Python】VSCodeで関数の引数名を表示する神機能
source: https://qiita.com/eycjur/items/bb67d5927755658786c8
author:
  - "[[Qiita]]"
published: 2024-09-26
created: 2025-05-06
description: 結論引数が多くて、わかりずらかったコードが↓このように、引数の名前が表示されることでめちゃくちゃわかりやすくなります！設定方法設定方法は、設定からPython Analysis Inlay…
tags:
  - Tech
  - Python
  - "#VSCode"
  - Tools
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

## 結論

引数が多くて、わかりずらかったコードが  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/678160/83b2efed-838f-0865-cce3-0296a6243d52.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F678160%2F83b2efed-838f-0865-cce3-0296a6243d52.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=bce3ee83dfb47aa44b47103eceaf66f1)  
↓  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/678160/5921752d-ef7c-9e96-2a9c-589bd274fa9e.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F678160%2F5921752d-ef7c-9e96-2a9c-589bd274fa9e.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=d8606bbd01b9e152054a10f7b57b5bd1)  
このように、引数の名前が表示されることでめちゃくちゃわかりやすくなります！

## 設定方法

設定方法は、設定からPython Analysis Inlay Hints: callArgumentNamesをallにするだけ。（画像の一番上の設定です）  
なお、この設定は、Pylanceという拡張機能のものです（ただし、Microsoft公式のPythonの拡張機能パックに入っているのでPythonユーザーであれば概ね設定可能だと思います）。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/678160/557e4a84-04df-2c0e-0ec7-3b138c9d264e.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F678160%2F557e4a84-04df-2c0e-0ec7-3b138c9d264e.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=13c6b941b557594e34452de218f6a21f)

jsonで設定する場合は以下のようにsettings.jsonに追記してください。

settings.json

```json
{
  "python.analysis.inlayHints.variableTypes": true,
}
```

## その他のTips

ちなみに、こちらはそこそこ有名だと思うので、設定済みの方が多いかと思いますが、画像の他の設定では、戻り値や引数の型を表示することができます。

こちらもjsonの場合は以下

settings.json

```json
{
  "python.analysis.inlayHints.functionReturnTypes": true,
  "python.analysis.inlayHints.variableTypes": true,
}
```

それでは、良いPythonライフを！

[2](https://qiita.com/eycjur/items/#comments)

コメント一覧へ移動

[150](https://qiita.com/eycjur/items/bb67d5927755658786c8/likers)

いいねしたユーザー一覧へ移動

122