---
title: C#でAESを使用して文字列を暗号化、複合化する
source: https://qiita.com/YoshijiGates/items/6c331924d4fcbcf6627a
author:
  - "[[Qiita]]"
published: 2023-02-28
created: 2025-05-06
description: 1. はじめにC#で文字列をハッシュ化する関数を作りたいMicrosoft社のAESクラスを使用して暗号化、複合化したい2. AESとは「Advanced Encryption Standa…
tags:
  - CSharp
  - Tech
  - "#セキュリティ"
read:
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から1年以上が経過しています。

[@YoshijiGates](https://qiita.com/YoshijiGates)

投稿日

## 1\. はじめに

- C#で文字列をハッシュ化する関数を作りたい
- Microsoft社のAESクラスを使用して暗号化、複合化したい

## 2\. AESとは

- 「Advanced Encryption Standard (AES)」の略でアメリカが2001年に標準暗号として定めた共通鍵暗号アルゴリズムである
- AESはSPN構造のブロック暗号で、ブロック長は128ビット、鍵長は128ビット・192ビット・256ビットの3つが利用できる

## 3\. サンプルコード

Utility.cs

```csharp
using System.Security.Cryptography;
using System.Text;

namespace Utility
{
    /// <summary>
    /// 暗号化に関するユーティリティクラス
    /// </summary>
    public static class CryptographyUtil
    {

        // AES暗号化 key生成するための文字列 (256bitキー(32文字))
        private const string _aesKey = "12345678901234567890123456789012";
        // AES暗号化 初期化ベクトルを生成するための文字列 (128bit(16文字))
        private const string _aesIv = "1234567890123456";

        /// <summary>
        /// AESで暗号化する関数
        /// </summary>
        /// <param name="plain_text">暗号化する文字</param>
        /// <returns></returns>
        public static string AesEncrypt(string plain_text)
        {
            // 暗号化した文字列格納用
            string encrypted_str;

            // Aesオブジェクトを作成
            using (Aes aes = Aes.Create())
            {
                // Encryptorを作成
                using ICryptoTransform encryptor =
                    aes.CreateEncryptor(Encoding.UTF8.GetBytes(_aesKey), Encoding.UTF8.GetBytes(_aesIv));
                // 出力ストリームを作成
                using MemoryStream out_stream = new();
                // 暗号化して書き出す
                using (CryptoStream cs = new(out_stream, encryptor, CryptoStreamMode.Write))
                {
                    using StreamWriter sw = new(cs);
                    // 出力ストリームに書き出し
                    sw.Write(plain_text);
                }
                // Base64文字列にする
                byte[] result = out_stream.ToArray();
                encrypted_str = Convert.ToBase64String(result);
            }

            return encrypted_str;
        }

        /// <summary>
        /// AESで復号する関数
        /// </summary>
        /// <param name="base64_text">複合化する文字</param>
        /// <returns></returns>
        public static string AesDecrypt(string base64_text)
        {
            string plain_text;

            // Base64文字列をバイト型配列に変換
            byte[] cipher = Convert.FromBase64String(base64_text);

            // AESオブジェクトを作成
            using (Aes aes = Aes.Create())
            {
                // 復号器を作成
                using ICryptoTransform decryptor =
                    aes.CreateDecryptor(Encoding.UTF8.GetBytes(_aesKey), Encoding.UTF8.GetBytes(_aesIv));
                // 復号用ストリームを作成
                using MemoryStream in_stream = new(cipher);
                // 一気に復号
                using CryptoStream cs = new(in_stream, decryptor, CryptoStreamMode.Read);
                using StreamReader sr = new(cs);
                plain_text = sr.ReadToEnd();
            }
            return plain_text;
        }

    }
}
```

## 4\. 参考文献

[0](https://qiita.com/YoshijiGates/items/#comments)

[4](https://qiita.com/YoshijiGates/items/6c331924d4fcbcf6627a/likers)

3