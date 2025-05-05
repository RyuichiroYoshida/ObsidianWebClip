---
title: Unityにおけるコンポーネント指向（単一責任の法則）
source: https://zenn.dev/akirakido/articles/94c33d3c1493d8e19f70
author:
  - "[[Zenn]]"
published: 2020-10-11
created: 2025-05-06
description: 
tags:
  - CSharp
  - Tech
  - 設計
  - Unity
read:
---
13[tech](https://zenn.dev/tech-or-idea)

この記事はUnityにおけるコンポーネント指向をいかに設計するかを検討する記事になります。以下に記すのは私の一つの結論になります。

## サンプルケース

今回のサンプルケースとして [2Dシューティングのチュートリアル](https://github.com/unity3d-jp-tutorials/2d-shooting-game/wiki/%E7%AC%AC04%E5%9B%9E-%E6%95%B5%E3%82%92%E4%BD%9C%E6%88%90%E3%81%97%E3%82%88%E3%81%86) を利用します。転載に問題があるようでしたら連絡して頂ければ直ちに記事を取り下げます。

では早速コードを見てみましょう。この記事で注目するのは以下の `Spaceship` とそれを利用する `Player` 、 `Enemy` のクラスになります。

```csharp
using UnityEngine;

// Rigidbody2Dコンポーネントを必須にする
[RequireComponent(typeof(Rigidbody2D))]
public class Spaceship : MonoBehaviour
{
    // 移動スピード
    public float speed;
    
    // 弾を撃つ間隔
    public float shotDelay;
    
    // 弾のPrefab
    public GameObject bullet;
    
    // 弾の作成
    public void Shot (Transform origin)
    {
        Instantiate (bullet, origin.position, origin.rotation);
    }
    
    // 機体の移動
    public void Move (Vector2 direction)
    {
        GetComponent<Rigidbody2D>().velocity = direction * speed;
    }
}
```

```csharp
using UnityEngine;
using System.Collections;

public class Player : MonoBehaviour
{
    // Spaceshipコンポーネント
    private Spaceship spaceship;

    IEnumerator Start ()
    {
        // Spaceshipコンポーネントを取得
        spaceship = GetComponent<Spaceship> ();

        while (true) {

            // 弾をプレイヤーと同じ位置/角度で作成
            spaceship.Shot (transform);

            // shotDelay秒待つ
            yield return new WaitForSeconds (spaceship.shotDelay);
        }
    }

    void Update ()
    {
        // 右・左
        float x = Input.GetAxisRaw ("Horizontal");

        // 上・下
        float y = Input.GetAxisRaw ("Vertical");

        // 移動する向きを求める
        Vector2 direction = new Vector2 (x, y).normalized;

        // 移動
        spaceship.Move (direction);
    }
}
```

```csharp
using UnityEngine;
using System.Collections;

public class Enemy : MonoBehaviour
{
    // Spaceshipコンポーネント
    Spaceship spaceship;

    void Start ()
    {
        // Spaceshipコンポーネントを取得
        spaceship = GetComponent<Spaceship> ();

        // ローカル座標のY軸のマイナス方向に移動する
        spaceship.Move (transform.up * -1);
    }
}
```

## コード依存からエディタ依存へ

まずは `GetComponent` は全部エディタ指定にします。これは私の一つの結論ではありますが、Unity を使う以上エディタを使わなければもったいないと感じます（というか、エディタが必要ないのであればUnityを利用する必要が無いでしょう）。また、感覚的にはDI（Dependency Injection）に似ています。これを前提に `Spaceship` クラスをリファクタしてみます。なお、 `_rigidbody` にアンダースコアを入れているのは `GameObject` の `rigidbody` と衝突しないためです。

```csharp
using UnityEngine;

// Rigidbody2Dコンポーネントを必須にする
[RequireComponent(typeof(Rigidbody2D))]
public class Spaceship : MonoBehaviour
{
    // 移動スピード
    [SerializeField] private float speed;
    // 弾を撃つ間隔
    [SerializeField] private float shotDelay;
    // 弾のPrefab
    [SerializeField] private GameObject bullet;
    // SpaceshipのRigidbody
    [SerializeField] private Rigidbody2D _rigidbody;
    
    // 弾の作成
    public void Shot (Transform origin)
    {
        Instantiate (bullet, origin.position, origin.rotation);
    }
    
    // 機体の移動
    public void Move (Vector2 direction)
    {
        _rigidbody.velocity = direction * speed;
    }
}
```

`Player` や `Enemy` の `Spaceship` も同様の変更を行いました。

## コンポーネント指向の適用

さて、次にクラスの分け方が妥当かを考えます。コンポーネント指向を適用する場合、現在の実装はふさわしくないでしょう。

例えば、敵のように移動する「隕石」を実装したいと考えます。その場合、 `Enemy` コンポーネントを `Asteroid` ゲームオブジェクトにつけることでそれを実現できますが、厳密には隕石は「敵」ではありません。どちらかといえば「もの」であり、 `Enemy` というコンポーネントを使うのは名前からしておかしいでしょう。ですので `Asteroid` というクラスを用意することになるのですが、移動するためにはやはり `Spaceship` クラスを利用したくなります。結局、 *移動の関数をSpaceshipに用意した* が故に、宇宙船以外にも `Spaceship` クラスを利用してしまうことが考えられます。

ではクラスはどう分割するのが良いでしょう。私はこれを「作用ごとに分割」します。オブジェクト指向のように聞こえると思いますが、基本的には大差ないでしょう。ただし、機能を集約するのはエディタ上の `GameObject` になります。

`Spaceship` から独立できるのは以下二つの機能です。

- `Shot`
- `Move`

関数に分けられていることから、そのような機能を想定していることが考えられます。であれば、これらの機能は分離されるべきでしょう。以下のように変更します。

```csharp
using UnityEngine;

public class Shooter : MonoBehaviour
{
    // 弾を撃つ間隔
    [SerializeField] private float shotDelay;
    // 弾のPrefab
    [SerializeField] private GameObject bullet;
    
    public float ShotDelay { get { return shotDelay; } }

    // 弾の作成
    public void Shot (Transform origin)
    {
        Instantiate (bullet, origin.position, origin.rotation);
    }

}
```

```csharp
using UnityEngine;

public class Movable : MonoBehaviour
{
    // 移動スピード
    [SerializeField] private float speed;
    // SpaceshipのRigidbody
    [SerializeField] private Rigidbody2D _rigidbody;

    // 機体の移動
    public void Move (Vector2 direction)
    {
        _rigidbody.velocity = direction * speed;
    }
}
```

これで、SOLID原則における「単一責任の法則」を満たすことができたのではないでしょうか。この二つのクラスを利用した `Player` と `Enemy` は以下のようになります。

```csharp
using UnityEngine;
using System.Collections;

public class Player : MonoBehaviour
{
    // 移動用コンポーネント
    [SerializeField] private Movable movable;
    // 射撃用コンポーネント
    [SerializeField] private Shooter shooter;

    IEnumerator Start ()
    {
        while (true) {

            // 弾をプレイヤーと同じ位置/角度で作成
            shooter.Shot (transform);

            // shotDelay秒待つ
            yield return new WaitForSeconds (shooter.ShotDelay);
        }
    }

    void Update ()
    {
        // 右・左
        float x = Input.GetAxisRaw ("Horizontal");

        // 上・下
        float y = Input.GetAxisRaw ("Vertical");

        // 移動する向きを求める
        Vector2 direction = new Vector2 (x, y).normalized;

        // 移動
        movable.Move (direction);
    }
}
```

Enemy.cs

```csharp
using UnityEngine;
using System.Collections;

public class Enemy : MonoBehaviour
{
    // 移動用コンポーネント
    [SerializeField] private Movable movable;

    void Start ()
    {
        // ローカル座標のY軸のマイナス方向に移動する
        movable.Move (transform.up * -1);
    }
}
```

こうすることにより、 `Player` も `Enemy` も、最悪「 `Spaceship` 」である必要がなくなります。  
これを使った `Asteroid` クラスは、以下のようになります。

```csharp
using UnityEngine;
using System.Collections;

[RequireComponent(typeof(Movable))]
public class Asteroid : MonoBehaviour
{
    // 移動用コンポーネント
    [SerializeField] private Movable movable;

    void Start ()
    {
        // ローカル座標のY軸のマイナス方向に移動する
        movable.Move (transform.up * -1);
    }
}
```

`Enemy` クラスと全く同じではありますが、そこには「宇宙船」の概念は取り払われています。つまるところ、 `Player`,`Enemy`,`Asteroid` クラスはすべて、「コンポーネントの利用方法を知るクラス」になり、集約の主役はエディタ上のゲームオブジェクトとなるのです。

以上が、 `Unity` におけるコンポーネント指向の考え方だと理解しています。いかがだっただろうか。私もまだまだ発展途上ではあるので、以上が正しいかの判断はこれからになりますが、今のところ私は上記のような運用方法でうまくいっていると考えています。結局のところ、Unityでは「エディタをちゃちゃっといじれば簡単にゲームを作れるよ！」という思想だと思っているので、なるべくコード依存からエディタ依存にしていくべきだと考えています。

---

お役に立てましたらライク、サポートしていただけますと嬉しいです！🙇🙇

13