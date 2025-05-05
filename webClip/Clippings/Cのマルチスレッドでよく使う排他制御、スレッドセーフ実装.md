---
title: C#のマルチスレッドでよく使う排他制御、スレッドセーフ実装
source: https://qiita.com/smi/items/c3f7b3c9bf01213634ff
author:
  - "[[Qiita]]"
published: 2022-07-22
created: 2025-05-06
description: C#でマルチスレッドで処理を行う際の排他制御や、スレッドセーフな実装に関して、よく使うものをまとめたいと思います。ReaderWriterLockSlim基本的なWriteLockとReadLo…
tags:
  - Tech
  - 設計
  - CSharp
read:
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から1年以上が経過しています。

[@smi (槙之介 久保)](https://qiita.com/smi)

最終更新日 投稿日 2022年07月19日

C#でマルチスレッドで処理を行う際の排他制御や、スレッドセーフな実装に関して、よく使うものをまとめたいと思います。

## ReaderWriterLockSlim

基本的なWriteLockとReadLockの排他制御に使えます。

```c#
private static ReaderWriterLockSlim _locker = new ReaderWriterLockSlim();
private List<int> list = new List<int>();

public void ThreadSafeWriteMethod()
{
    _locker.EnterWriteLock();
    try
    {
        list.Add(1);
    }
    finally
    {
        locker.ExitWriteLock()
    }
}

public List<int> ThreadSafeReadMethod()
{
    _locker.EnterReadLock();
    try
    {
        return list.ToList();
    }
    finally
    {
        locker.ExitReadLock()
    }
}
```

WriteLockが制御を取得している状態では、ReadとWriteがブロックされます。  
ReadLockが制御を取得している状態では、他のReadは処理できますが、Writeはブロックされます。

似たものにReaderWriterLockがありますが、古いVersionで現在はSlimが推奨です。

必ずfinallyでロックを開放しなければいけないので関数化してしまうことをお勧めします。

```c#
public static class LockUtility
{
    public static Write(Action action, ReaderWriterLockSlim locker)
    {
        locker.EnterWriteLock();
        try
        {
            action();
        }
        finally
        {
            locker.ExitWriteLock()
        }
    }

    public static Read<T>(Func<T> func, ReaderWriterLockSlim locker)
    {
        locker.EnterReadLock();
        try
        {
            return func();
        }
        finally
        {
            locker.ExitReadLock()
        }
    }
}
```

## ConcurrentBag or SyncronizedCollection

ReaderWriterLockSlimで紹介した例ですが、スレッドセーフなListを作りたいだけならこのどちらかで十分です。  
ConcurrentBagはListとは少し勝手が違うので、おすすめはListとほぼ同様のインターフェースのSyncronizedCollectionです。  
SyncronizedCollectionはMicrosoft製ですが、デフォルトには入っていないのでnugetなどでSystem.ServiceModel.Primitives.dllを入れる必要があります。

使い方はListと同じなので割愛させてもらいますが。  
追加、削除、検索がスレッドセーフになっています。

## ConcurrentDictionnary

名前から予測はできると思いますが、スレッドセーフなDictionaryです。  
通常のDictionaryにスレッドセーフならではの関数が追加されてます。

例えば、GetOrAdd関数などは、Keyがあれば取得、なければ第2引数の値を追加するような関数

```c#
ConcurrentDictionanry<string, int> _intDictionary = new ConcurrentDictionanry<string, int>()

// 1
int num1 = _intDictionary.GetOrAdd("key1", 1);
// 1
int num2 = _intDictionary.GetOrAdd("key1", 2);
// 3
int num3 = _intDictionary.GetOrAdd("key2", 3);
```

## 拡張編　特定のKeyごとに排他制御を掛ける

ConcurrentDictionnaryを使えば、ある特定のKey同士では排他制御を掛けて、別のKeyであれば並列で実行をするような挙動ができます。

```c#
private static ConcurrentDictionanry<string, ReaderWriterLockSlim> _lockerDictionary = new ConcurrentDictionanry<string, ReaderWriterLockSlim>()

private void ThreadSafeAction(string key)
{
    var locker = _lockerDictionary.GetOrAdd(key, new ReaderWriterLockSlim());
    LockUtility.Write(() => 
    { 
        Thread.Sleep(1000);
        Console.WriteLine(key);
    }, locker);
}

private void Main()
{
    Task.Run(() => 
    {
        ThreadSafeAction("key1");
    });
    Task.Run(() => 
    {
        ThreadSafeAction("key1");
    });
    Task.Run(() => 
    {
        ThreadSafeAction("key2");
    });
 
    // 出力
    // key1
    // key2
    // key1
}
```

## Mutex

ちょっと特殊で、プログラムを超えてPC内で排他制御を掛けることができます。  
Mutexは名前を付けることができて、付けた名前と同一のMutexは作成できない性質からよく、プログラムの多重起動防止によく使われています。

## その他

その他にも排他制御用のクラスやスレッドセーフなクラスが用意されているので、  
用途に合いそうなものを探してみてください。

[https://docs.microsoft.com/ja-jp/dotnet/api/system.threading?view=net-6.0](https://docs.microsoft.com/ja-jp/dotnet/api/system.threading?view=net-6.0)  
[https://docs.microsoft.com/ja-jp/dotnet/api/system.collections.concurrent?view=net-6.0](https://docs.microsoft.com/ja-jp/dotnet/api/system.collections.concurrent?view=net-6.0)

[0](https://qiita.com/smi/items/#comments)

[9](https://qiita.com/smi/items/c3f7b3c9bf01213634ff/likers)

10