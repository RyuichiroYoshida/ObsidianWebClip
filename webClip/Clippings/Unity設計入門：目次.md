---
title: Unity設計入門：目次
source: https://qiita.com/wolfmagnate/items/cf7081a0352667b12d6a
author:
  - "[[Qiita]]"
published: 2023-06-30
created: 2025-05-06
description: はじめにUnityで開発している際に雑に書いてしまうと、コードが膨れ上がって混沌とすることが多いと思います。そして、それを解決するためにネットで検索してみると「MVPやMVCなどの技法があるらし…
tags:
  - Tech
  - Unity
  - 設計
  - CSharp
read:
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から1年以上が経過しています。

## はじめに

Unityで開発している際に雑に書いてしまうと、コードが膨れ上がって混沌とすることが多いと思います。  
そして、それを解決するためにネットで検索してみると「MVPやMVCなどの技法があるらしいぞ」とか「疎結合にするといいっぽい」とか、いろいろな手法の情報が得られます。  
しかし、「 **概念だけ言われても実際どう書くんだよ！** 」「 **単純なサンプルだけじゃわかんないよ！** 」「 **とりあえず言われたライブラリを使ってみたけどよくなった気がしない** 」と思ってしまうこともしばしばあります。また、それぞれの記事で前提知識が多かったり、「責務の問題」とかいう曖昧な言葉でお茶を濁されたりしがちです。  
そこで、このシリーズでは、マクロな設計について、「 **基礎の基礎から** 」そして「 **実際のコードを踏まえて** 」解説していきたいと思います。

## 対象読者

- C#・Unityがなんとなく分かる人
- UniRXの超基本的なところが分かる人(具体的には IObservable, IObserver, Subject, ReactiveProperty, Subscribe, OnNext程度です)
- Unityでもう少しいい感じのコードが書きたい人
- マクロな設計についての知識を増やしたい人
- ※「switch文を消したい」や、「個々のオブジェクトの生成方法」などのコードレベルのミクロな設計については対象外です

## リンク

[第1章「ModelとView」・理論編](https://qiita.com/wolfmagnate/items/0e5f0e29b1d01e359eeb)  
[第1章「ModelとView」・実践編](https://qiita.com/wolfmagnate/items/579a7b8dd007f6fbb5c3)  
[第1章(補足)「変わるものと変わらないものの分離」](https://qiita.com/wolfmagnate/items/5aa039e027ca8a1cdfd8)  
[第1章(補足)「オブジェクト指向とコンポーネント指向」](https://qiita.com/wolfmagnate/items/f9fc2ac39ef74d9d8fbd)  
[第2章「MVXの思想」・理論編その1](https://qiita.com/wolfmagnate/items/314c084b41b8086d466a)  
[第2章「MVXの思想」・理論編その2](https://qiita.com/wolfmagnate/items/acc990e5ed0990e149c1)  
[第2章「MVXの思想」・実践編その1](https://qiita.com/wolfmagnate/items/cc97802018babf18102a)  
[第2章「MVXの思想」・実践編その2](https://qiita.com/wolfmagnate/items/f0cef95b74177074624f)  
[第3章その1「監視型View」・理論編](https://qiita.com/wolfmagnate/items/7a23864070f8b0392506)  
[第3章その2「Passive View」・理論編](https://qiita.com/wolfmagnate/items/a0ac9458ef395249d4c0)

## 続編

続編、 [Unity・MVP設計実践](https://qiita.com/wolfmagnate/items/4b1a1d0cf8db7ff7d66f) を連載しています。Unity設計入門で得られた理論的な知見をもとに、具体的なプロジェクトを設計して、UnityでのMVP設計の実際的な手法について解説しています。第3章の理論的な知見をもとに、実際の開発への応用を目指しました。

[0](https://qiita.com/wolfmagnate/items/#comments)

コメント一覧へ移動

[100](https://qiita.com/wolfmagnate/items/cf7081a0352667b12d6a/likers)

いいねしたユーザー一覧へ移動

136