---
title: 【Unity】DOTS(ECS)が正式リリースされたので試してみた
source: https://zenn.dev/k41531/articles/5168cc291b4dfa
author:
  - "[[Zenn]]"
published: 2023-06-13
created: 2025-05-06
description: 
tags:
  - 設計
  - 1
  - CSharp
  - Unity
read:
---
15[tech](https://zenn.dev/tech-or-idea)

6月1日のUnity 2022 LTS で、ECS(Dots)が正式リリースされたので試してみました。

Unity 2022 LTS の新機能については、Unity Japanのこちらの動画が非常にわかりやすいです。  
ぜひこちらをご覧ください。<iframe src="https://www.youtube-nocookie.com/embed/BdwnSrbcC1E" allow="accelerometer; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

この記事は、実際に試してみることに主眼を置くので、ECSの詳しい説明は割愛します。（間違いなく説明できる自信がない）  
ECSについて解説は、古いですがこちらの記事が個人的に好きです。

この動画もいいかも

また、Unity JapanがECSに関する動画を出すと話していたので、そちらも楽しみです。  
(追記:2023/07/05)↓  
7月4日に公開されていました。とても分かりやすいのでおすすめです。  
[はじめての Unity ECS - Entity Component System を使ってみよう！](https://www.youtube.com/watch?v=vzF00Wb6wNY&ab_channel=UnityJapan)

## ECSとは

Entity Component Systemの略で、その名の通り3つの概念で構成されています。

1. Entity：ゲームやプログラムに登場するエンティティ、つまり「もの」です。
2. Component：エンティティに関連付けられたデータ。例えば、エンティティの位置、速度、色などです。
3. System：コンポーネトの状態を変化させるルール。例えば、位置を更新したり、色を変更することができます。

従来のECSを使用しないゲーム開発では、キャラクターなどのオブジェクトが持つべき **情報と振る舞いをキャラクタークラス** のように単一にまとめていました。  
一方、ECSでは、 **キャラクターの情報をコンポーネント** に、 **振る舞いをシステム** にと別々に分けて管理します。

## やってみる

基本的にはこちらにこちらに則ってやっていきます。

### 環境

```
Unity:  2022.3.1f1
Chip:   Apple M1
Memory: 16GB
```

### インストール

Unity Hubから、Unity 2022.3.1f1をインストールできてるものとします。  
プロジェクトをURPもしくはHDRで作成して、Package ManagerのUnity Registryから必要なものをインストールします。

- com.unity.entities
- com.unity.entities.graphics

通常のプロジェクトで作成した場合は、以下のいずれをインストールする必要があります。

- Universal Render Pipeline
- High Definition Render Pipeline

#### Domain Reloadの設定

Entities プロジェクトのパフォーマンスを最大化するためには、UnityのDomain Reload設定を無効にすべきとのこと。  
Edit > Project Settings > Editor メニューに移動、Enter Play Mode Optionsの設定を有効にし、Reload DomainとReload Sceneのチェックボックスは無効にしておきます。

### Sub Sceneの作成

Unityの通常のシーンとECSには互換性がないため、代わりにサブシーンを使用します。  
Hierarchyウィンドウを右クリックして、Create > New Sub Scene > Empty Sceneで作成できます。  
![Sub Scene](https://res.cloudinary.com/zenn/image/fetch/s--ctjz_uky--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/d7702a78214a0f18ed256482.png%3Fsha%3Ddd7a442590609e75fd923b8b29741619fc7116bd)

### Componentの作成

今回作るのは、オブジェクトを生成するSpawnerです。

Spawnerコンポーネントを作成します。  
冒頭でも述べた通り、コンポーネントはデータを持つだけで、振る舞いは持ちません。  
ここでのSpawnerは次のデータを持ちます。

- Prefab：生成するオブジェクトのプレハブ
- SpawnPosition：生成する位置
- NextSpawnTime：次に生成する時間
- SpawnRate：生成する間隔

```csharp
using Unity.Entities;
using Unity.Mathematics;

public struct Spawner : IComponentData
{
    public Entity Prefab;
    public float3 SpawnPosition;
    public float NextSpawnTime;
    public float SpawnRate;
}
```

### Entityの作成

SpawnerというゲームオブジェクトをEntityに変換するSpawnerAuthoringスクリプトを作成します。  
Bakerを使用することで、ゲームオブジェクトをECSで使えるように変換することができます。

1. SpawnerAuthoringスクリプトを作成します。

```csharp
using UnityEngine;
using Unity.Entities;

class SpawnerAuthoring : MonoBehaviour
{
    public GameObject Prefab;
    public float SpawnRate;
}

class SpawnerBaker : Baker<SpawnerAuthoring>
{
    public override void Bake(SpawnerAuthoring authoring)
    {
        var entity = GetEntity(TransformUsageFlags.None);
        AddComponent(entity, new Spawner
        {
            // デフォルトでは、各オーサリングGameObjectはEntityに変換されます。
            // GameObject（またはオーサリングコンポーネント）が与えられると、GetEntityは生成されるEntityを検索します。
            Prefab = GetEntity(authoring.Prefab, TransformUsageFlags.Dynamic),
            SpawnPosition = authoring.transform.position,
            NextSpawnTime = 0.0f,
            SpawnRate = authoring.SpawnRate
        });
    }
}
```

- TransformUsageFlags: トランスフォーム（位置、回転、スケール）がどのように使用されるべきかを示す。
- GetEntity: エンティティを返します。Authoring GameObject(Unityの通常のGameObject)またはAuthoring Component(Unityの通常のコンポーネント)が与えられると、それに対応したエンティティを返します。  
	[https://docs.unity.cn/Packages/com.unity.entities@1.0/api/Unity.Entities.IBaker.GetEntity.html](https://docs.unity.cn/Packages/com.unity.entities@1.0/api/Unity.Entities.IBaker.GetEntity.html)
- AddComponent: 与えられたエンティティにコンポーネントを追加します。
1. Spawnerオブジェクト(空のオブジェクト)をSub Sceneに追加して、SpawnerAuthoringスクリプトをアタッチします。  
	![Spawner](https://res.cloudinary.com/zenn/image/fetch/s--nWq1g3Dx--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/cb786f5b27045737cdd78001.png%3Fsha%3D0ac58c7b1d2fc401e4f33dd4d269d02fff339dab)
2. 任意のPrefabをSpawnerオブジェクトのPrefabに設定します。(私は適当にCubeを作成しました)
3. Spawn Rate を 2 に設定します。
4. Window > Entities > HierarchyからEntities Hierarchyウィンドウを開きます。
5. Entities Hierarchyウィンドウの右上にある丸いマークをクリックして、データモードをRuntimeに切り替えます。これでSub Sceneに含まれるエンティティが表示されます。  
	![DataMode](https://res.cloudinary.com/zenn/image/fetch/s--sSsLPKbk--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/0edbf83df1c4774a06d42f75.png%3Fsha%3D30ff731400213aafd45f699ba18a75002adaee46)

データモードについては、こちらを参照してください。

1. SpawnerエンティティのInspectorで、Entity Baking Previewを開くと、アタッチされたSpawnerコンポーネントと、Bakerが設定したコンポーネント値が表示されます。  
	![EntityBakingPreview](https://res.cloudinary.com/zenn/image/fetch/s--IhW5IazP--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/46fe38bfe09ea9e893e332a4.png%3Fsha%3D86afc3d8e154d3596172d889d9969c6ca468e790)

### Systemの作成

1. `SpawnerSystem` という名前で次のスクリプトを作成します。
2. このスクリプトはアタッチをしなくても動作します。実行すると、指定した時間ごとにPrefabが生成されるているのがわかると思います。

```csharp
using Unity.Entities;
using Unity.Transforms;
using Unity.Burst;

[BurstCompile]
public partial struct SpawnerSystem : ISystem
{
    public void OnCreate(ref SystemState state) { }

    public void OnDestroy(ref SystemState state) { }

    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        // すべてのSpawnerコンポーネントをクエリします。このシステムは、
        // コンポーネントから読み取りと書き込みを行う必要があるため、RefRWを使用します。
        // システムが読み取り専用のアクセスのみを必要とする場合は、RefROを使用します。
        foreach (RefRW<Spawner> spawner in SystemAPI.Query<RefRW<Spawner>>())
        {
            ProcessSpawner(ref state, spawner);
        }
    }

    private void ProcessSpawner(ref SystemState state, RefRW<Spawner> spawner)
    {
        // 次のスポーン時間が経過している場合    
        if (spawner.ValueRO.NextSpawnTime < SystemAPI.Time.ElapsedTime)
        {
            // スポナーの持つプレファブを使って、新しいエンティティを生成します。
            Entity newEntity = state.EntityManager.Instantiate(spawner.ValueRO.Prefab);
            // LocalPosition.FromPositionは、指定された位置で初期化されたTransformを返します。
            state.EntityManager.SetComponentData(newEntity, LocalTransform.FromPosition(spawner.ValueRO.SpawnPosition));

            // 次のスポーン時間をリセットします。
            spawner.ValueRW.NextSpawnTime = (float)SystemAPI.Time.ElapsedTime + spawner.ValueRO.SpawnRate;
        }
    }
}
```

### Systemの最適化をする

IJobEntityを使用して、Systemの処理を最適化します。  
具体的には、複数のスレッドを並行して実行されるようにジョブをスケジュールします。  
先ほどと振る舞いは変わりませんが、並列実行されるようです。

```csharp
using Unity.Collections;
using Unity.Entities;
using Unity.Transforms;
using Unity.Burst;

[BurstCompile]
public partial struct OptimizedSpawnerSystem : ISystem
{
    public void OnCreate(ref SystemState state) { }

    public void OnDestroy(ref SystemState state) { }

    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        EntityCommandBuffer.ParallelWriter ecb = GetEntityCommandBuffer(ref state);

        // ジョブの新しいインスタンスを作成し、必要なデータを割り当て、ジョブを並列してスケジュールします。
        new ProcessSpawnerJob
        {
            ElapsedTime = SystemAPI.Time.ElapsedTime,
            Ecb = ecb
        }.ScheduleParallel();
    }

    private EntityCommandBuffer.ParallelWriter GetEntityCommandBuffer(ref SystemState state)
    {
        var ecbSingleton = SystemAPI.GetSingleton<BeginSimulationEntityCommandBufferSystem.Singleton>();
        var ecb = ecbSingleton.CreateCommandBuffer(state.WorldUnmanaged);
        return ecb.AsParallelWriter();
    }
}

[BurstCompile]
public partial struct ProcessSpawnerJob : IJobEntity
{
    public EntityCommandBuffer.ParallelWriter Ecb;
    public double ElapsedTime;

    // IJobEntityは、その\`Execute\`メソッドのパラメータに基づいて、コンポーネントデータクエリを生成します。
    // この例では、すべてのSpawnerコンポーネントをクエリし、\`ref\`を使用して、操作に読み取りと書き込みのアクセスが必要であることを指定します。
    // Unityは、コンポーネントデータクエリに一致する各エンティティに対して\`Execute\`を処理します。

    private void Execute([ChunkIndexInQuery] int chunkIndex, ref Spawner spawner)
    {
        // 次のスポーン時間が経過している場合
        if (spawner.NextSpawnTime < ElapsedTime)
        {
            // 新しいエンティティを生成し、スポナーに配置します。
            Entity newEntity = Ecb.Instantiate(chunkIndex, spawner.Prefab);
            Ecb.SetComponent(chunkIndex, newEntity, LocalTransform.FromPosition(spawner.SpawnPosition));

            // 次のスポーン時間をリセットします。
            spawner.NextSpawnTime = (float)ElapsedTime + spawner.SpawnRate;
        }
    }
}
```

- EntityCommandBuffer: エンティティに対する操作（作成、破棄、コンポーネントの追加/削除など）をバッファリング（キューに入れる）するためのもの

正直、最適化したコードについては、いまいち理解できていません。  
特にこの呪文。

```csharp
var ecbSingleton = SystemAPI.GetSingleton<BeginSimulationEntityCommandBufferSystem.Singleton>();
var ecb = ecbSingleton.CreateCommandBuffer(state.WorldUnmanaged);
return ecb.AsParallelWriter();
```

とにかく、EntityCommandBufferを作るためのシングルトンと、そのシングルトンからEntityCommandBufferを作っているようです。そして、そのEntityCommandBufferを並列実行するためのものに変換しているようです。

また、ドキュメントにはProfilerを見れば、並列実行されている様子がわかるとありましたが、私には全然わかりませんでした...  
一応、Main ThreadのSpawnerSystemはOptimizedの方が早く終わっているので、並列実行されているのかなと思います。  
**SpawnerSystem (0.045ms)**  
![SpawnerSystem](https://res.cloudinary.com/zenn/image/fetch/s--nhLyVMZ1--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/128a553a79df56f8f2e9f5e1.png%3Fsha%3D45552bf7caaba101575f9bc6ce0504aad83d814c)  
**OptimizedSpawnerSystem (0.005ms)**  
![OptimizedSpawnerSystem](https://res.cloudinary.com/zenn/image/fetch/s--a6hXU7hz--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/0c898c7a21c2b978c0f1fdaf.png%3Fsha%3D024c538f47a451f7e11f8e3bff3f766bb724b0a7)

## 感想

今までのUnityだったら一瞬で終わるようなコードですが、ECSだと結構書くことが多いですね。  
従来と書き方が違いすぎで、まだまだ使用感が掴めませんが、ECSの考え方は面白いと思っているので色々と遊んでみようかなと思います。  
また、今回の内容はチュートリアル程度なので、もう少しECSの真価がわかるようなものも作ってみる予定です。

余談ですが、チュートリアルついでに記事も書くと、より理解が深まるのでおすすめです。

15