---
title: UE5 ちょっと便利かもしれないプログラミングの書き方
source: https://papersloth.hatenablog.com/entry/2023/12/25/001252
author:
  - "[[PaperSloth’s diary]]"
published: 2023-12-25
created: 2025-05-06
description: 環境 概要 if文 / branchノードの削減または単純化 (Min / Max / Clampの利用) アイテムや魔法での回復処理を組むケース ダメージを受けてHPが減るケース Clampについて 配列のwrap around(上限に達したら最初に戻る) マイナス方向の配列のwrap around(下限に達したら最大数に戻る) 小数点の比較 (floatの比較) 早期returnの活用 まとめ 環境 ・UE5.3.2 ・Rider 2023.3.2 概要 対象読者の想定としてはプロの現役プログラマーというよりは、そこを目指して勉学に励んでいる方のレベルを想定しています。 非プログラマーのア…
tags:
  - UE
  - 1
  - Cpp
read:
---
### 環境

・UE5.3.2  
・Rider 2023.3.2

### if文 / branchノードの削減または単純化 (Min / Max / Clampの利用)

ここではBlueprintや [C++](https://d.hatena.ne.jp/keyword/C%2B%2B) 向けに紹介していますが、Min / Max / [Clamp](https://d.hatena.ne.jp/keyword/Clamp) はMaterialでも使用可能です。  
また、0.0 ～ 1.0の [Clamp](https://d.hatena.ne.jp/keyword/Clamp) に関してはMaterialではsaturateノードがあり  
個人的にはそちらの方が0 - 1の [Clamp](https://d.hatena.ne.jp/keyword/Clamp) であることが伝わりやすくて好みです。

#### アイテムや魔法での回復処理を組むケース

まずは以下のようにゲームで回復を行った時に  
現在HP + 回復量の値が最大HPを超えたら最大HPとして  
そうでない場合は回復後のHPを割り当てるような処理があった場合  
通常は以下のように組むことがあると思います。  
![](https://cdn-ak.f.st-hatena.com/images/fotolife/P/PaperSloth/20231224/20231224211502.png)

こういったケースではif文は不要でMinノードを使用することで簡潔に記述できます。  
![](https://cdn-ak.f.st-hatena.com/images/fotolife/P/PaperSloth/20231224/20231224211802.png)

さらに、Blueprintの場合はMathExpressionでより一層簡潔に記述が可能です。  
![](https://cdn-ak.f.st-hatena.com/images/fotolife/P/PaperSloth/20231224/20231224214815.png)  

#### ダメージを受けてHPが減るケース

実際にはAny | Point | Radial Damage Event等を使用しますが、ここでは簡潔に説明するためCustomEventで定義しています。

先程同様にダメージ処理を組んでいきます。  
HP - ダメージが0以下ならHPを0とし、そうでない場合はHP - ダメージが現在のHPとなります。  
![](https://cdn-ak.f.st-hatena.com/images/fotolife/P/PaperSloth/20231224/20231224213350.png)

今度はMaxノードを使用します。  
![](https://cdn-ak.f.st-hatena.com/images/fotolife/P/PaperSloth/20231224/20231224213656.png)

Minのケースと同様にMath Expression化可能です。  
![](https://cdn-ak.f.st-hatena.com/images/fotolife/P/PaperSloth/20231224/20231224214236.png)  

#### Clampについて

[Clamp](https://d.hatena.ne.jp/keyword/Clamp) は上記のMinとMaxを組み合わせたものです。  
[Value](https://d.hatena.ne.jp/keyword/Value), Min, Maxの3つのPinがあり、 [Value](https://d.hatena.ne.jp/keyword/Value) がMin以上でMax以下となるように制限をかけてくれます。  
![](https://cdn-ak.f.st-hatena.com/images/fotolife/P/PaperSloth/20231224/20231224214657.png)

最後に [C++](https://d.hatena.ne.jp/keyword/C%2B%2B) での記述を載せておきます。

```c
// 実際にはMaxHealth等のパラメーターはDataTableやDataAsset等にして管理するのが望ましいです。
void OnHeal(const float Heal)
{
    Health = FMath::Min(MaxHealth, Health + Heal);
}
void OnDamage(const float Damage)
{
    Health = FMath::Max(0.0f, Health - Damage);
}
```

### 配列のwrap around(上限に達したら最初に戻る)

先ずは、以下のように手持ちの武器を次の武器に切り替えるような処理を組んだとします。  
![](https://cdn-ak.f.st-hatena.com/images/fotolife/P/PaperSloth/20231224/20231224220915.png)

このままでも現在選択している武器のindex情報は正しく  
剣 -> 銃 -> 拡張武器 -> 剣... といった形で次の武器に切り替えることが可能です。

こういった処理をより簡潔に組むには、index % Max といった形で配列の最大数で剰余してやればよいです。  
![](https://cdn-ak.f.st-hatena.com/images/fotolife/P/PaperSloth/20231224/20231224221914.png)

wrap aroundをよく使う場面としては  
ゲームの設定画面やアイテム選択画面なんかが特に多いと思います。  
設定内の画面品質のUIで  
クオリ [ティー](https://d.hatena.ne.jp/keyword/%A5%C6%A5%A3%A1%BC) 低 -> 中 -> 高 -> 最高 -> 低... といった場面ですね。

[C++](https://d.hatena.ne.jp/keyword/C%2B%2B) で組むと以下のようになります。

```c
TArray<FString> WeaponArray{"Sword", "Gun", "ExtraWeapon"};
void ChangeNextWeapon()
{
    CurrentWeanponIndex = (WeaponArray.Num() % ++CurrentWeanponIndex);
}
```

### マイナス方向の配列のwrap around(下限に達したら最大数に戻る)

~~残念ながらマイナス方向のwrap aroundは剰余で求めることができません。~~

Xにて親切なお犬様がマイナス方向のwrap aroundについて教えてくださいました。

まずは愚直に組むと以下のような処理になるかと思います。  
![](https://cdn-ak.f.st-hatena.com/images/fotolife/P/PaperSloth/20231224/20231224224052.png)

先程教えていただいた内容を元にノードを組むと以下のようになります。  
Blueprintの配列操作にはLastIndexというノードがあり、配列の末尾のindexを取得可能です。  
この場合は要 [素数](https://d.hatena.ne.jp/keyword/%C1%C7%BF%F4) が3つなので、Lengthは3となり  
LastIndexは0, 1, 2と0始まりでカウントされるため末尾の2が取得できます。  
内容としてはLength - 1と同義です。  
![](https://cdn-ak.f.st-hatena.com/images/fotolife/P/PaperSloth/20231225/20231225145505.png)

最後に [C++](https://d.hatena.ne.jp/keyword/C%2B%2B) で組むと以下のようになります。

```c
void ChangePrevWeapon()
{
    const int WeaponNum = WeaponArray.Num();
    CurrentWeanponIndex = (CurrentWeanponIndex + (WeaponNum - 1)) % WeaponNum;
}
```

  
先程のプラス方向のwrap aroundと組み合わせて  
プラスマイナスの双方向を考慮したWrap関数を作っておくと便利かもしれません。  
  

### 小数点の比較 (floatの比較)

例えば以下のように何かしらのfloat値に計算をして、==ノードで0.0 かどうかを比較したいといったケースがあると思います。  
![](https://cdn-ak.f.st-hatena.com/images/fotolife/P/PaperSloth/20231224/20231224234304.png)

この場合に、良くも悪くもBlueprintではわりと動いてtrue になることがあるのですが  
基本的に小数点の比較というのはプログラミングでは御八度です。  
小数点の誤差でfalse になるケースがほとんどだったりします。

こういった誤差を考慮してだいたい同じ値という比較用のノードが用意されています。  
それがNearly Equalノードです。  
原則として、このノードを使うようにするとよいでしょう。  
![](https://cdn-ak.f.st-hatena.com/images/fotolife/P/PaperSloth/20231224/20231224234633.png)

[C++](https://d.hatena.ne.jp/keyword/C%2B%2B) で記述すると以下です。

```c
bool IsZeroFloat(const float Value)
{
    // C++の場合は0との比較用にIsNearlyZero()があります
    return FMath::IsNearlyZero(Value);
    // return FMath::IsNearlyEqual(Value, 0.0f);
}
```

余談ですが、UnityではMathf.Approximatelyを使用することで比較が可能です。  
  

### 早期returnの活用

正直なところ、Blueprintでは特にシンプルになった感じがなくあまり恩恵を感じられないかもですが・・・  
攻撃用の関数を以下のように組んだとします。  
HPが0以上で攻撃のクールタイムが0になっていたら攻撃をするといった関数です。  
![](https://cdn-ak.f.st-hatena.com/images/fotolife/P/PaperSloth/20231224/20231224235613.png)

条件式として  
HPが0以上 かつ  
クールタイムが0  
であれば攻撃となります。

こういったケースでは早期returnを使用すると良いとされています。  
HPが0であれば攻撃しない  
クールタイムが残っていれば攻撃しない  
といった条件式に置き換えることができます。  
![](https://cdn-ak.f.st-hatena.com/images/fotolife/P/PaperSloth/20231225/20231225000035.png)

Blueprintでは恩恵を受けれている感じがないですね。  
[C++](https://d.hatena.ne.jp/keyword/C%2B%2B) で記述してみます。  
先ずは早期return なしver

```c
void Attack()
{
    if (Health > 0.0f)
    {
        if (FMath::IsNearlyZero(CoolTime))
        {
            // 何かしらの攻撃処理
        }
    }
}
```

このようにネストが深くなって処理が読みにくいです。  
早期returnで書き換えると以下のようにスッキリします。

```c
void Attack()
{
    if (FMath::IsNearlyZero(Health))
        return;
    if (0.0f < CoolTime)
        return;
    // 何かしらの攻撃処理
}
```

### まとめ

[C++](https://d.hatena.ne.jp/keyword/C%2B%2B) に絞ればもっとたくさんのネタがありますが、Blueprintと [C++](https://d.hatena.ne.jp/keyword/C%2B%2B) で考えてみるとあまり浮かびませんでした。  
いずれも学生時代の頃に知って便利だと感じた小ネタになります。

早期return や説明用のlocal 変数定義等については、 [C++](https://d.hatena.ne.jp/keyword/C%2B%2B) やその他の言語では読みやすくなって便利なのですが  
いかんせんノードエディタではその恩恵があまり受けられず  
むしろlocal変数を増やしたりすることで余計にノード郡が大きくなってしまったりと逆効果かな？と感じることがあります。

こういった浅いネタでなく、もう少しちゃんとしたネタが浮かべばまた書いていこうと思います。

[« UE5 KawaiiPhysicsをBlueprintプロジェク…](https://papersloth.hatenablog.com/entry/2024/02/14/201629) [UE5 1フレーム後に処理を実行する方法(Del… »](https://papersloth.hatenablog.com/entry/2023/04/26/225312)