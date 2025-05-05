---
title: 【Python, Unity】（ほぼ）コピペでUnity・Pythonの双方向通信を簡単に記述する
source: https://qiita.com/konbraphat51/items/f9d1a36218ee127a7118
author:
  - "[[Qiita]]"
published: 2024-03-09
created: 2025-05-06
description: はじめに機械学習をUnityゲーム上に持ち出したいとき、機械学習部分をどうしてもPythonで動かしたいとき、ありますよね。 （機械学習界隈がよりによってPythonの砂の城の上で遊んでいるため）そこで…
tags:
  - 1
  - Python
  - "#Unity"
  - "#通信"
  - "#バックエンド"
read:
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から1年以上が経過しています。

## はじめに

機械学習を `Unity` ゲーム上に持ち出したいとき、機械学習部分をどうしても `Python` で動かしたいとき、ありますよね。 ~~（機械学習界隈がよりによって `Python` の砂の城の上で遊んでいるため）~~

そこで、 `Unity` と `Python` のTCPによる双方向通信を抽象化したコードを書いたので、その導入方法をご紹介します。

## やりたいこと

TCP通信にて、

- `Unity` -> `Python`
- `Python` -> `Unity`

の間で `JSON` を送りあいます。

一方からもう片方へメッセージが送られたら、そのメッセージを正しく処理できる **コールバック関数** を呼ぶ、というインフラを両者にスタンバイさせます。

ここでは、 `Python` 側がTCPサーバー、 `Unity` 側をTCPクライアントにしています。（起動順が変わるだけで、接続後は両者とも対等です）

### データ形式

**形式名** と **JSON部分** に分けます。

- **形式名**  
	JSON部分の形式を指定します。 **送られてきたデータが何のデータか識別する** ためのものです。
- **JSON部分**  
	ほんまに普通のJSONです。

ここは抽象化しているので **意識しなくともいい** 部分ですが、TCP通信においては、

```text
<s>{json_format_name}!{json}<e>
```

の文字列として送り合っています。

なので、形式名、JSON部分双方に `<s>`, `<e>` が入らないように、  
形式名には`!`が入らないように指定してください。（データ内部に`!`が入る分は大丈夫です）

## 導入方法

僕が書いたコードを取り込むだけです。

こちらのレポジトリです。 **スターつけていただける** と泣きながら喜びます><

[https://github.com/konbraphat51/UnityPythonConnectionModules](https://github.com/konbraphat51/UnityPythonConnectionModules)

### Python側

上記 `GitHub` レポジトリから直接 `pip install` します。

```text
pip install "git+https://github.com/konbraphat51/UnityPythonConnectionModules.git#egg=UnityConnector&subdirectory=PythonSocket"
```

導入終わり。

### Unity側

[UnitySocket/Assets/Scriptsフォルダー](https://github.com/konbraphat51/UnityPythonConnectionModules/tree/main/UnitySocket/Assets/Scripts) のうち、

- `DataClass.cs`
- `DataDecoder.cs`
- `PythonConnector.cs`

の3コードを `Unity` プロジェクトに **コピペ** してください。

その後、プロジェクトに合わせて `DataClass` と `DataDecoder` の **継承クラス** を作る必要があります。

#### DataClassの継承

**`JSON` 形式に対応したクラス** です。  
**データ形式の数だけ作ります**

もし、 `JSON` 設計が

```json
{
    "id": 334,
    "hp": 22.6,
    "name": "hanshin"
}
```

のようであれば、下記のように記述してください

CharacterData.cs

```c#
using System;

namespace PythonConnection
{
    [Serializable]
    public class CharacterData : DataClass
    {
        public int id;
        public float hp;
        public string name;
    }
}
```

`[Serializable]` を忘れないようにして、 `JSON` のキー名と変数名を **一致** させてください。  
ここらは [`JsonUtility`](https://docs.unity3d.com/ScriptReference/JsonUtility.html) の魔法を使っています。

#### DataDecoderの継承

`DataToType()` をオーバーライドして、 **形式名とクラスの対応** を明示していただきます。

MyDecoder.cs

```c#
using System;
using System.Collections.Generic;

namespace PythonConnection
{
    public class MyDecoder : DataDecoder
    {
        protected override Dictionary<string, Type> DataToType()
        {
            return new Dictionary<string, Type>() { { "character", typeof(CharacterData) }, };
        }
    }
}
```

#### 送受信オブジェクトを作る

`MonoBehaviour` にしているので、オブジェクトにアタッチする必要があります。  
**空オブジェクト** に対して、

- 自作の `DataDecoder`
- `PythonConnector`

をアタッチしてください。

これで終わりです。  
`Unity` 側のセットアップを1クリックぐらいにできたら [抽象化の王、抽象化キング](https://qiita.com/uts1_6/items/11e348ead4bae71571b8) になれるんですけどね。

## 使い方

### Python側

[サンプルコード](https://github.com/konbraphat51/UnityPythonConnectionModules/blob/main/PythonSocket/tests/ManualTester.py) を紹介します。

```python
from UnityConnector import UnityConnector

#タイムアウト時のコールバック
def on_timeout():
    print("timeout")

#Unityから停止命令が来たときのコールバック
def on_stopped():
    print("stopped")

#インスタンス
connector = UnityConnector(
    on_timeout=on_timeout,
    on_stopped=on_stopped
)

#データが飛んできたときのコールバック
def on_data_received(data_type, data):
    print(data_type, data)

print("connecting...")

#Unity側の接続を待つ
connector.start_listening(
    on_data_received
)

print("connected")

#デモ用のループ
while(True):
    #Enterで送信を開始（入力内容は送信内容と関係ない）
    input_data = input()

    #Unityへ停止命令
    if input_data == "q":
        connector.stop_connection()
        break

    #送るデータをdictionary形式で
    data = {
        "testValue0": 334,
        "testValue1": [0.54,0.23,0.12,],
    }
    
    print(data)

    #Unityへ送る
    connector.send(
        "test",
        data
    )
```

簡単に言うと、

- `UnityConnector` インスタンスを作って
- `start_listening()` で接続して
- `start_listening()` で登録したコールバックで **受け取り**
- `send()` で **送信** ってことですね。

受信時コールバックで渡される引数は **形式名** と **JSON代わりのdictionary** なので、そこからどう処理するかはご自身で用意してください。

### Unity側

[サンプルコード](https://github.com/konbraphat51/UnityPythonConnectionModules/blob/main/UnitySocket/Assets/Scripts/Test/ConnectionTest.cs) を紹介します。

```c#
using System;
using System.Collections;
using System.Collections.Generic;
using PythonConnection;
using UnityEngine;

public class ConnectionTest : MonoBehaviour
{
    //Pythonへ送信するデータ形式
    [Serializable]
    private class SendingData
    {
        public SendingData(int testValue0, List<float> testValue1)
        {
            this.testValue0 = testValue0;
            this.testValue1 = testValue1;
        }

        public int testValue0;

        [SerializeField]
        private List<float> testValue1;
    }

    void Start()
    {
        //データ受信時時のコールバックを登録
        PythonConnector.instance.RegisterAction(typeof(TestDataClass), OnDataReceived);

        //Pythonへの接続を開始
        if (PythonConnector.instance.StartConnection())
        {
            Debug.Log("Connected");
        }
        else
        {
            Debug.Log("Connection Failed");
        }
    }

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            PythonConnector.instance.StopConnection();
            Debug.Log("Stop");
        }
    }

    public void OnTimeout()
    {
        Debug.Log("Timeout");
    }

    public void OnStop()
    {
        Debug.Log("Stopped");
    }

    //データ受信時のコールバック
    public void OnDataReceived(DataClass data)
    {
        //DataClass型で渡されてしまうため、明示的に型変換
        TestDataClass testData = data as TestDataClass;

        //受け取り結果表示
        Debug.Log("testValue0: " + testData.testValue0);
        foreach (float v in testData.v1)
        {
            Debug.Log("testValue1: " + v);
        }

        //Python側へ送るデータを生成
        int v1 = UnityEngine.Random.Range(0, 100);
        List<float> v2 = new List<float>()
        {
            UnityEngine.Random.Range(0.1f, 0.9f),
            UnityEngine.Random.Range(0.1f, 0.9f)
        };
        SendingData sendingData = new SendingData(v1, v2);

        Debug.Log("Sending Data: " + v1 + ", " + v2[0] + ", " + v2[1]);

        //Python側へ送信
        PythonConnector.instance.Send("test", sendingData);
    }
}
```

要するに

- `PythonConnector.instance.RegisterAction(typeof(データクラス), 関数名);`  
	で受信時コールバックを登録し、（データクラスの種類ごとで別途登録）
- `PythonConnector.instance.StartConnection();`  
	で接続開始（こちらはクライアント側なので **先に `python` 側を起動しておくように** ）
- `PythonConnector.instance.Send("形式名", データクラス);`  
	で送信。（クラスには `[Serializable]` をかけておく必要があります）

## 最後に

以上でTCP通信ができるはずです。

なにか不明瞭な点があれば、 **テストコード**

- Python側  
	[https://github.com/konbraphat51/UnityPythonConnectionModules/blob/main/PythonSocket/tests/ManualTester.py](https://github.com/konbraphat51/UnityPythonConnectionModules/blob/main/PythonSocket/tests/ManualTester.py) だけ
- Unity側  
	[https://github.com/konbraphat51/UnityPythonConnectionModules/tree/main/UnitySocket](https://github.com/konbraphat51/UnityPythonConnectionModules/tree/main/UnitySocket) 全体

が動作しているので、こちらを確認してください。

- この記事にいいねをつけてくださり、
- [レポジトリ](https://github.com/konbraphat51/UnityPythonConnectionModules/tree/main) にスターをつけてくださると

泣きながら喜びます

## 追記 2024/3/9

Unity: 2022.3.15f1  
Python: 3.12  
での動作を確認しております。

[0](https://qiita.com/konbraphat51/items/#comments)

[44](https://qiita.com/konbraphat51/items/f9d1a36218ee127a7118/likers)

50