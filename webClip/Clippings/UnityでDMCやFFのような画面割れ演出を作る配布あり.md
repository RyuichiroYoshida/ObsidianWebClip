---
title: UnityでDMCやFFのような画面割れ演出を作る[配布あり]
source: https://qiita.com/KTN44295080/items/e9f8106943ebee15aa2a#%E4%BD%9C%E3%82%8A%E6%96%B9-%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%A0%E7%B7%A8
author:
  - "[[Qiita]]"
published: 2023-11-06
created: 2025-05-06
description: はじめにこの記事はLife is Tech！Members Advent Calendar 2020、17日目の記事です。デビルメイクライ4のゲームオーバー画面や、FFXの戦闘移行演出のような画…
tags:
  - 1
  - Unity
  - "#Effect"
read:
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から1年以上が経過しています。

## はじめに

この記事は [Life is Tech！Members Advent Calendar 2020](https://qiita.com/advent-calendar/2020/life-is-tech-members) 、17日目の記事です。  
デビルメイクライ4のゲームオーバー画面や、FFXの戦闘移行演出のような画面割れの演出を作っていきます。  
[![4B8E6E2B-535E-442E-9B9E-47C3C040D244.gif](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2F6276585c-d168-9852-ac4d-b8c7958fc0c7.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=4dc1771faf406be04a0eb48aeeb1bfa6)](https://youtu.be/rUQorfbW0Ro)  
[DMC4](https://store.steampowered.com/app/329050)  
[![41DFA790-8851-4BA7-A8AA-A76BDC16B8C3.gif](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2F51955f83-1180-99ed-5eaa-527ae511e772.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=78c5c8eef17f821587270365d68e9df0)](https://youtu.be/9ayiw528tOo)  
[FFX](https://store.steampowered.com/app/359870)

[![index.gif](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2F627e3270-03bb-95ea-9da9-abbe5be57deb.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=8de7897a652ec0519ecdc716a02d9421)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2F627e3270-03bb-95ea-9da9-abbe5be57deb.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=8de7897a652ec0519ecdc716a02d9421)  
[完成品](https://qiita.com/KTN44295080/items/#%E9%85%8D%E5%B8%83%E7%89%A9)

~~DMCはいいぞ。~~

## 目次

1. [はじめに](https://qiita.com/KTN44295080/items/#%E3%81%AF%E3%81%98%E3%82%81%E3%81%AB)
2. [目次](https://qiita.com/KTN44295080/items/#%E7%9B%AE%E6%AC%A1)
3. [必要なもの](https://qiita.com/KTN44295080/items/#%E5%BF%85%E8%A6%81%E3%81%AA%E3%82%82%E3%81%AE)
4. [作り方 モデリング編](https://qiita.com/KTN44295080/items/#%E4%BD%9C%E3%82%8A%E6%96%B9-%E3%83%A2%E3%83%87%E3%83%AA%E3%83%B3%E3%82%B0%E7%B7%A8)
5. [作り方 プログラム編](https://qiita.com/KTN44295080/items/#%E4%BD%9C%E3%82%8A%E6%96%B9-%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%A0%E7%B7%A8)
6. [さいごに](https://qiita.com/KTN44295080/items/#%E3%81%95%E3%81%84%E3%81%94%E3%81%AB)
7. [配布物](https://qiita.com/KTN44295080/items/#%E9%85%8D%E5%B8%83%E7%89%A9)
8. [参考元](https://qiita.com/KTN44295080/items/#%E5%8F%82%E8%80%83%E5%85%83)

## 必要なもの

- [パソコン](https://jp.store.asus.com/store/asusjp/ja_JP/pd/productID.5425822500?utm_medium=affiliate&utm_campaign=649426&utm_source=2501754&ranMID=43708&ranEAID=7kD3zFflDUo&ranSiteID=7kD3zFflDUo-pYdlomgrWvVicN45lovTug)
- [Unity](https://unity3d.com/jp/get-unity/download) (記事では2020.1を使用)
- [Blender](https://www.blender.org/download/) (割れモデルの制作用)

一応割れ用のモデルを作るのが面倒だという人のために現在筆者が用いているモデルを配布しています。ご自由にお使いください。 [記事下部よりダウンロードできます。](https://qiita.com/KTN44295080/items/#%E9%85%8D%E5%B8%83%E7%89%A9)

## 作り方 モデリング編

まずはBlenderを起動します。  
[![1.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/04b03603-d702-6fb1-0c9f-ee890c026add.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2F04b03603-d702-6fb1-0c9f-ee890c026add.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=2eced649fc8ab7b0c9b4f14dda65dac7)  
上記のような画面が出てくると思うので、画面中央にあるウィンドウの外の部分をクリックして、このウィンドウを閉じましょう。  
[![2.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/17594811-88b9-264a-788a-2ea482387a5f.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2F17594811-88b9-264a-788a-2ea482387a5f.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=8461b3b6a081e490277e98bb1ef47584)  
画面上に表示されている３つのオブジェクトも不要なのでクリックして選択し、Deleteキーで削除しておきましょう。  
これで下準備は完了です。  
[![3.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/b67d2ce6-0778-ec01-c24e-8770cf3ffdb4.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2Fb67d2ce6-0778-ec01-c24e-8770cf3ffdb4.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=9b2da746aa37b03717a46139c836cc7e)  
では作っていきます。  
Shift＋Aで上記のようなウィンドウが表示されるので、Mesh→Planeと選択します。  
[![4.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/f35652d7-4c54-6361-0585-9c42fe5549ef.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2Ff35652d7-4c54-6361-0585-9c42fe5549ef.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=2e0f13c791d6e12cbd6a6d0a2e9f2e35)  
これで画面割れの元となるオブジェクトを作れました。  
[![5.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/ef2d60b7-afa6-d35a-bc82-89513d13b79c.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2Fef2d60b7-afa6-d35a-bc82-89513d13b79c.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=41ba45d61270a3dde16472b915970215)  
次にオブジェクトのサイズを画面のサイズに合わせます。  
今回はPCゲーム用の比率(16:9)で作るのでXを16、Yを9にします。  
[![6.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/cbd4d62f-3855-b19d-4fe5-fff0720b68e1.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2Fcbd4d62f-3855-b19d-4fe5-fff0720b68e1.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a96b77fac707316219898316bb12a2bb)  
少し見づらいのでカメラの位置をオブジェクトの正面にしましょう。  
テンキーの7を押すか、右上にあるZと書いてある箇所\*をクリックする事で上記画像と同じカメラ位置になります。  
*上記画像の赤枠  
[![7.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/e5b77976-c4e3-3a9a-a607-0482dbda749c.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2Fe5b77976-c4e3-3a9a-a607-0482dbda749c.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=63d4c6aaad5999e98055f63597e6c653)  
ではオブジェクトをカットしバラバラにしていきます。  
まずは上記画像内の左上にある赤枠に位置するObject Modeと書かれた部分をクリックし、Edit Modeに切り替えます。  
[![7.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/33a1f52a-522a-fbff-ad11-86a17461a522.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2F33a1f52a-522a-fbff-ad11-86a17461a522.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=05cc966787700a55cb729eff08cc82c4)  
切り替えてKキーを押すとマウスカーソルがナイフに変わります。  
この状態でクリックしながらオブジェクトを切るようにマウスを動かしていくと、上記画像のように切れ込みが入っていきます。  
ここで切った通りにオブジェクトが切れますので、一切れを大きくしすぎると演出の派手さに欠けますし、かえって切りすぎると重くなるのでいい塩梅で。  
[![8.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/b68703e5-7496-4840-c70e-dfe53a9c212d.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2Fb68703e5-7496-4840-c70e-dfe53a9c212d.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=7227cda6ab23b01f3dae124cfa9c111c)  
切り終えたらEnterキーを押してください。  
メッシュが切った通りに分割されます。  
[![9.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/234aae24-a55f-093b-12b0-4df60d3a6fd2.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2F234aae24-a55f-093b-12b0-4df60d3a6fd2.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a7112d2eaa27af0c8b5d8dabc8042e98)  
次に左上にある選択モード* を左から三つ目の面に切り替えます。  
\*上記画像の赤枠の位置  
[![10.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/56cd17b0-7f1e-bf55-e9bf-9f7078aea7f6.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2F56cd17b0-7f1e-bf55-e9bf-9f7078aea7f6.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=63d3783ab5aeddee778c8ea4cb515718)  
分割したメッシュ一つを選択し、Pキーを押して出てきたウィンドウ内のSelectionをクリックしてください。  
するとその選択されたメッシュが一つのオブジェクトとして独立するので、残りの切断したメッシュも同じ手順で全て独立させてください。  
[![11.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/63380ed9-7179-fefb-1d34-fb75b63269dd.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2F63380ed9-7179-fefb-1d34-fb75b63269dd.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=37559459d2e4f4f4f4a5d29694898a7a)  
終わったらEdit Modeから再びObject Modeに戻します。  
Bキーを押すと、複数選択出来るようになるのでマウスを動かし全てのオブジェクトを網羅させてください。  
全て選択したらまたEdit Modeに変えて、テンキーの1を押すか、画面右上にあるYと書かれた部分をクリックしてください。  
[![12.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/d15bf663-adfc-8330-923e-39f10accd8b4.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2Fd15bf663-adfc-8330-923e-39f10accd8b4.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=0c4160218641b62909f3bab6fc28bb06)  
するとカメラの位置がオブジェクトの真上の位置になります。(オブジェクトが平面なのでわかりにくいですが)  
次にEキーを押し、マウスを上下に動かすことでオブジェクトに厚みが生まれます。  
[![13.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/c9a4cbcc-0a20-23d5-71e7-84ebab05d55b.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2Fc9a4cbcc-0a20-23d5-71e7-84ebab05d55b.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=1d9c5c645cbfcb3f6297fd39d24be15d)  
大体これぐらいの厚さになったらクリックして厚みを決定します。  
[![14.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/a190acea-4d9b-0e5e-e974-89eef8955d9e.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2Fa190acea-4d9b-0e5e-e974-89eef8955d9e.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=2209add1a2d48bf10575cd1f29b0f7e1)  
最後にFile→ExportからFBXを選びこのモデルを出力してBlenderでの作業は終了です。お疲れ様でした。

## 作り方 プログラム編

[![15.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/9138092f-925a-6018-d89b-6cbe91b7f04b.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2F9138092f-925a-6018-d89b-6cbe91b7f04b.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=fbfccfb01eaf961c14a83248bb87f95c)  
Assetsフォルダ内に先ほど出力した割れ用のモデルを入れ、  
Sceneにこのモデルを配置します。  
[![16.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/681f4de7-7a75-69a5-56cd-dce4f1403d3e.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2F681f4de7-7a75-69a5-56cd-dce4f1403d3e.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=2ebace8f18a63ac5d46392632158a9a6)  
Projectタブ内で右クリック→CreateからRender Textureを選択し作ります。  
作ったレンダーテクスチャのSizeを画面と同じ(1920\*1080)に調整します。  
このレンダーテクスチャに画面の映像を投影し、このテクスチャを割れ用のモデルに貼り付ける事で画面割れを演出します。  
[![18.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/b0f7981e-038b-c4e5-251c-fdcd12158fd4.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2Fb0f7981e-038b-c4e5-251c-fdcd12158fd4.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=76f3569d59ecae3f477da17f8b88b876)  
まずマテリアルを作り、シェーダーをUnlit→Textureに変更します。  
光の影響を受けなくして画面の色を変えずに出力させるためです。  
[![19.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/a223d7b7-3fd2-4d7e-74e3-fca7cce20857.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2Fa223d7b7-3fd2-4d7e-74e3-fca7cce20857.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=544118a039317e39304492904e777954)  
今作成したマテリアルを割れ用のモデルに適用しましょう。割れ用のモデルを選択、次にMaterialsを選択し、No Nameとなっている部分に作成したマテリアルを入れます。  
[![20.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/4d180d4b-3200-58cb-a061-af3cc228063c.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2F4d180d4b-3200-58cb-a061-af3cc228063c.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=259a786bee269ed9dc6dce62d835cfa2)  
カメラの画角ぴったりに割れ用のモデルが収まるようにカメラの位置を調整します。Z座標を-2にするとぴったり合うと思います。  
(背景色を変えたりテキストを追加したりしましたが、本機能とは関係ない部分なので省きます。)  
[![21.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/25ca7c6f-9854-1130-4c55-73edb494cfa8.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2F25ca7c6f-9854-1130-4c55-73edb494cfa8.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=0ab37045174c65fb6aed8869f95cbddc)  
ではいよいよスクリプトを書いていきます。Projectタブで右クリックしCreate→C# Scriptを選択し作ります。名前はScreenBreak。  
これに以下のコードをコピペしてください。

ScreenBreak.cs

```c#
using UnityEngine;
using System.Collections;

public class ScreenBreak : MonoBehaviour
{
    [SerializeField] private bool useGravity = true;                            // 重力を有効にするかどうか
    [SerializeField] private Vector3 explodeVel = new Vector3(0, 0, 0.1f);      // 爆発の中心地
    [SerializeField] private float explodeForce = 200f;                         // 爆発の威力
    [SerializeField] private float explodeRange = 10f;                          // 爆発の範囲
    private Rigidbody[] rigidBodies;

    void Start()
    {
        rigidBodies = GetComponentsInChildren<Rigidbody>();                     // 子(破片)のRigidbodyを取得しておく
        StartCoroutine("BreakStart");                                           // 動作にディレイを掛けるためコルーチンを使用
    }

    IEnumerator BreakStart()
    {
        foreach (Rigidbody rb in rigidBodies)
        {
            rb.isKinematic = false;
            rb.useGravity = useGravity;
            rb.AddExplosionForce(explodeForce / 5, transform.position + explodeVel, explodeRange);
        }
        yield return new WaitForSeconds(0.02f);                                 // 一瞬動かすことでひび割れを演出

        foreach (Rigidbody rb in rigidBodies)
        {
            rb.isKinematic = true;
        }
        yield return new WaitForSeconds(0.8f);

        foreach (Rigidbody rb in rigidBodies)
        {
            rb.isKinematic = false;
            rb.AddExplosionForce(explodeForce, transform.position + explodeVel, explodeRange);
        }
    }
}
```

スクリプトの解説はコメントを参照。  
[![1index.gif](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2F23b786e4-3d02-1a58-5c15-d37c6e08d8a6.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=073ce127b75293e8853c409566ac1779)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2F23b786e4-3d02-1a58-5c15-d37c6e08d8a6.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=073ce127b75293e8853c409566ac1779)  
ScreenBreakスクリプトを割れ用モデルの親にアタッチし、実行してみると確かに割れる演出を確認できました。

しかしこのままではただ真っ黒の物体が爆発してるだけです。  
なのでカメラの映像をテクスチャに投影しましょう。 ~~投影、開始~~  
[![22.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/dbc977d9-181e-9e5e-a2ff-09803a7e9de1.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2Fdbc977d9-181e-9e5e-a2ff-09803a7e9de1.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=4bf2d150bf88b7692b1700c43e917d65)  
今回は別のシーンを作り、条件を満たしたとき本シーンに遷移し割れるようにするので、新規シーンを作成します。  
[![23.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/87dd926f-b3d0-42d4-586b-b170b8299751.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2F87dd926f-b3d0-42d4-586b-b170b8299751.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=616b2e50a2dae3dfb9f6bb5e47be85c6)  
新規シーンに適当にオブジェクトを配置しました。まあ見栄えは十分でしょう。  
では画面を投影するための準備をします。  
[![25.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/918507/1de25984-17e5-8693-96d2-5feac283c9ba.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F918507%2F1de25984-17e5-8693-96d2-5feac283c9ba.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=bbc153c5f623c8f5b0cc9d874e965f6f)  
カメラの映像をレンダーテクスチャに投影している間はカメラが映像を出力できなくなってしまうので、投影用のカメラをメインカメラの子として作ります。  
普段は使わないので投影用のカメラコンポーネントのチェックを外し無効にしておきます。

新しくスクリプトを作ります。名前はTraceScreen。  
これに以下のコードをコピペしてください。

TraceScreen.cs

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class TraceScreen : MonoBehaviour
{
    [SerializeField] private RenderTexture renderTexture;
    private Camera renderCamera;

    // Start is called before the first frame update
    void Start()
    {
        renderCamera = Camera.main.transform.GetChild(0).GetComponent<Camera>();    //映像投影用のカメラを取得
    }

    // Update is called once per frame
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.F))
            StartCoroutine("TraceOn");                                              // 画面割れを始めたいタイミングに置く
    }

    IEnumerator Trace()
    {
        renderCamera.enabled = true;
        renderCamera.targetTexture = renderTexture;                                 // 投影、開始
        yield return null;                                                          // 1f待ってもらって映像をレンダーテクスチャに投影する
        renderCamera.targetTexture = null;
        renderCamera.enable = false;                                                // 全行程、完了
        SceneManager.LoadSceneAsync("ScreenBreak");                                 // シーンを遷移する
    }
}
```

targetTextureで映像をどこに出力するのかを決めています。  
nullなら画面に、レンダーテクスチャが指定されているならそのテクスチャに映します。

これを適当なオブジェクトにアタッチします。  
そして実行しFキーを押してみると、シーンが遷移し画面が割れます。  
(もし割れ用モデルに投影された画像が反転している場合、割れモデルのY軸を180度回すと解決するはずです)

## さいごに

ScreenBreakスクリプトのシリアライズ化された変数、つまりインスペクターから見える変数をいじくってみるだけで、演出が大きく変化します。  
色々微調整して遊んでみてください。そしたら何かが見えてくるはずです。

## 配布物

[割れるオブジェクト](https://www.dropbox.com/s/vr9czr80uhym4uv/BreakBlock.fbx?dl=1)  
[本プロジェクト](https://github.com/KTN44295080/ScreenBreak)

## 参考元

[FFの戦闘開始演出っぽいアレを作ってみた](http://iwashigame.hatenadiary.jp/entry/2015/10/05/033459)  
本機能の作成にあたり、上記の記事を参考にさせて頂きました。  
[Qiita記事投稿用テンプレート](https://qiita.com/tkrtmy1031/items/80c292fab5d631c6d4ba)  
本記事執筆にあたり、上記記事のテンプレを基に作らせて頂きました。  
[GIFメーカー](https://syncer.jp/gif-maker)  
本記事上部のGIFを作る為に用いさせて頂きました。

[1](https://qiita.com/KTN44295080/items/#comments)

[13](https://qiita.com/KTN44295080/items/e9f8106943ebee15aa2a/likers)

16