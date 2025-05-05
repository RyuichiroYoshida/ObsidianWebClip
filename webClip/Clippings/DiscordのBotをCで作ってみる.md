---
title: DiscordのBotをC#で作ってみる
source: https://qiita.com/HAGITAKO/items/fff2e029064ea38ff13a
author:
  - "[[Qiita]]"
published: 2018-02-20
created: 2025-05-06
description: DiscordのBotをC#で作ってみる開発環境Visual Studio 2017 (C#)Discord.Net 1.0.2初めにスプラトゥーン2のAPIをコールするBotを@otuh…
tags:
  - 1
  - CSharp
  - アプリ
  - "#Discord"
  - 学習系
read:
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から5年以上が経過しています。

## 開発環境

- Visual Studio 2017 (C#)
- Discord.Net 1.0.2

## 初めに

スプラトゥーン2のAPIをコールするBotを [@otuhs\_dさん](https://qiita.com/otuhs_d/items/86fc7cda767b5682311f) が作成されたのをみて、自分のチーム用に何かできないかを考えました。

そこで、チーム内プライベートマッチでルールとステージをランダムで決定できるものをC#コンソールアプリで作ってみました。

## 参考

`@otuhs_d` さんの記事  
[Splatoon2チーム向けDiscord Bot『Roboty』を公開しました](https://qiita.com/otuhs_d/items/86fc7cda767b5682311f)  
[DiscordのBotにSplatoon2のステージ情報を教えてもらう](https://qiita.com/otuhs_d/items/3882472cd36cd31f8503)

## 実装後のイメージ

[![WS000018.JPG](https://qiita-image-store.s3.amazonaws.com/0/90597/d71274aa-55f5-1598-96ca-0a37e43e193e.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F90597%2Fd71274aa-55f5-1598-96ca-0a37e43e193e.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=446a10d109df5c93fb2988844a7c87dc)

「.rl」とText Chatで入力すると、ルールとステージをランダムで表示するという簡単なBot

## 事前準備としてDiscordでBotの登録

**※すでにディスコサーバを作成済みの手順です。**

- ディスコのWebページから More > Depelopers > My Apps を選択。
- もしくは、 [ここから](https://discordapp.com/developers/applications/me)

[![FireShot Capture 19 - Discord - Developer Documentat_ - https___discordapp.com_developers_docs_intro.png](https://qiita-image-store.s3.amazonaws.com/0/90597/45811dfd-0864-c450-470b-e81232159d12.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F90597%2F45811dfd-0864-c450-470b-e81232159d12.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=65b42a33508bff334ad79b6677125be0)

- 「New Apps」を押下する。

[![FireShot Capture 21 - Discord - My Apps - https___discordapp.com_developers_applications_me.png](https://qiita-image-store.s3.amazonaws.com/0/90597/0f595165-71f4-f093-c9bb-6d121ac16a63.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F90597%2F0f595165-71f4-f093-c9bb-6d121ac16a63.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=353bee1b06b9b3fbde08fdb99dd8c3bf)

- 「App Name」にBotの名前を登録します。
- 「App Icon」に画像ファイルを置くことで、アイコン化できます。
- 「Create App」を押下します。

[![FireShot Capture 22 - Discord - Develope_ - https___discordapp.com_developers_applications_me_create.png](https://qiita-image-store.s3.amazonaws.com/0/90597/9a474b67-2a4c-548f-afee-50bdbd9469d8.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F90597%2F9a474b67-2a4c-548f-afee-50bdbd9469d8.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=aaeec7bc7f1f63d2f9d360c2b72823d3)

- 「Create Bot User」を押下します。

[![FireShot Capture 23 - Discord - Developer Documentation_ - https___discordapp.com_developers_.png](https://qiita-image-store.s3.amazonaws.com/0/90597/748cb188-1e02-e819-9781-08732ac95728.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F90597%2F748cb188-1e02-e819-9781-08732ac95728.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=c6a2a44c3f27fe5b575878cec6bd07bc)

- 確認メッセージは、そのまま「Yes do it!」を押下します。

[![FireShot Capture 24 - Discord - Developer Documentation_ - https___discordapp.com_developers_.png](https://qiita-image-store.s3.amazonaws.com/0/90597/9598b580-909c-c964-1057-e67b424bf8ae.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F90597%2F9598b580-909c-c964-1057-e67b424bf8ae.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=aec028e3047e7c0bd0333fd865f71f23)

- 「Public Bot」にチェックを入れます。

[![FireShot Capture 24 - Discord - Developer Documentation_ - https___discordapp.com_developers_.png](https://qiita-image-store.s3.amazonaws.com/0/90597/7b278425-53e6-8a80-e252-9cfac768f989.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F90597%2F7b278425-53e6-8a80-e252-9cfac768f989.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=76903b8d706c15281004b0a55194dda6)

- 「click to reveal」を押下します。
- そこで、表示される `Tokenをメモ` っておきます。
- 画面最下部の「Save changes」を押下します。

[![FireShot Capture 29 - Discord - Developer Documentation_ - https___discordapp.com_developers_.png](https://qiita-image-store.s3.amazonaws.com/0/90597/9430688b-29c6-3646-0644-214492d6a499.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F90597%2F9430688b-29c6-3646-0644-214492d6a499.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=5c3d9923da1d97aa5e13d9dd9b7f557c)

- 「Generate OAuth2 URL」を押下します。

[![FireShot Capture 26 - Discord - Developer Documentation_ - https___discordapp.com_developers_.png](https://qiita-image-store.s3.amazonaws.com/0/90597/32d09912-231d-2737-c345-59a83870653b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F90597%2F32d09912-231d-2737-c345-59a83870653b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=848d089f662694e2137e566321382e78)

- 「Copy」を押下して、ブラウザで、そのURLを開きます。
- この際、 **ClientIDをメモ** しておいてください。

[![FireShot Capture 27 - Discord - Developer Documentation_ - https___discordapp.com_developers_.png](https://qiita-image-store.s3.amazonaws.com/0/90597/2a4c77a2-1372-ef36-3d1b-56cbd4d2cdd2.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F90597%2F2a4c77a2-1372-ef36-3d1b-56cbd4d2cdd2.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=3f89fda57011552925526215eafea2a1)

- Botを追加するサーバを選択し、「認証」を押下します。
- 認証完了後、ブラウザを閉じます。

[![FireShot Capture 28 - アカウントへのアクセスを許可します_ - https___discordapp.com_oauth2_authorize.png](https://qiita-image-store.s3.amazonaws.com/0/90597/5979ed51-a197-d701-e750-5039bfc63969.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F90597%2F5979ed51-a197-d701-e750-5039bfc63969.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=459381c7bb9326fbbf9568e089ed737f)

- Discordを起動すると、作成したBotが表示されています。

[![WS000021.png](https://qiita-image-store.s3.amazonaws.com/0/90597/5ebe0194-f707-6bb8-e747-32ff64737ce8.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F90597%2F5ebe0194-f707-6bb8-e747-32ff64737ce8.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=57169e6b1383666e87bdf4a487ddfabf)

- 自身のギアのアイコンから設定を開き > テーマ を選択します。
- 開発者モードをトグルスイッチをオンにします。

[![WS000022.JPG](https://qiita-image-store.s3.amazonaws.com/0/90597/7ac17927-144c-350e-93f2-2af194e609f4.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F90597%2F7ac17927-144c-350e-93f2-2af194e609f4.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=c1bd3351686bf5bcefbe7d430f013af5)

- サーバのIDをコピーするのに、サーバーを右クリックし「IDのコピー」を選択しメモっておきます。

[![WS000023.JPG](https://qiita-image-store.s3.amazonaws.com/0/90597/68e9dc66-57d1-8805-6b01-3d9c7124e824.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F90597%2F68e9dc66-57d1-8805-6b01-3d9c7124e824.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=bd7c22f7476120b41ec8b15da2c5f0ba)

### 準備完了！

これで、Botの準備ができたので、この後は、Visual Studioでの作業になります。

| メモしたもの | 場所 | 今回使用 |
| --- | --- | --- |
| ClientID | Discord WebページでメモしたClientID | × |
| Token | Discord WebページでメモしたToken | ○ |
| サーバーID | Discordアプリの画面左上の「IDをコピー」 | × |

今回の説明する機能まででは、○のものだけが必要です。

## Visual Studio

### プロジェクトの作成

- Windowsクラシックデスクトップ > コンソールアプリ(.Net Framework) を選択して、「OK」を押下する。

[![WS000019.JPG](https://qiita-image-store.s3.amazonaws.com/0/90597/9e60c34c-5294-76b9-dfa2-4f8129359fdf.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F90597%2F9e60c34c-5294-76b9-dfa2-4f8129359fdf.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=b0efeddca1a44f6da841f1edf5dd2ba9)

### Nugetから「Discord.Net 1.0.2」インストール

- 「プロジェクトエクスプローラ」>「参照設定」を右クリック >「NuGetパッケージ管理」
- 「参照」タブを選択 > 検索欄に「Discord.Net」を入力します。
- 「Discord.Net」を選択します。
- 最新の安定版を選択して「インストール」ボタンを押下します。

[![WS000020.JPG](https://qiita-image-store.s3.amazonaws.com/0/90597/e8cb98a6-28a7-6f1c-67eb-e538340af0fc.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F90597%2Fe8cb98a6-28a7-6f1c-67eb-e538340af0fc.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=ab0d6a6a62b4470b18c10993f2103fca)

### Messagesクラスを作成

- コメントを受けとった処理を作成します。

```csharp
:Messages.cs

using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Discord.Commands;
using System.Collections.Generic;

namespace Splatoo2ConsoleApp
{
    public class Messages : ModuleBase
    {
        /* ルール列挙 */
        Dictionary<int, string> rule = new Dictionary<int, string>()
        {
            {0, "ナワバリ"},
            {1, "エリア"},
            {2, "ホコ"},
            {3, "ヤグラ"},
            {4, "アサリ"},
        };

        /* ステージ列挙 */
        Dictionary<int, string> stage = new Dictionary<int, string>()
        {
            {0,   "バッテラストリート" },
            {1 ,  "フジツボスポーツクラブ"},
            {2 ,  "ガンガゼ野外音楽堂"},
            {3 ,  "チョウザメ造船"},
            {4 ,  "海女美術大学"},
            {5 ,  "コンブトラック"},
            {6 ,  "マンタマリア号"},
            {7 ,  "ホッケふ頭"},
            {8 ,  "タチウオパーキング"},
            {9 , "エンガワ河川敷"},
            {10, "モズク農園"},
            {11,  "Ｂバスパーク"},
            {12,  "デボン海洋博物館"},
            {13,  "ザトウマーケット"},
            {14,  "ハコフグ倉庫"},
            {15,  "アロワナモール"}
        };

        /// <summary>
        /// [rl]というコメントが来た際の処理
        /// </summary>
        /// <returns>Botのコメント</returns>
        [Command("rl")]
        public async Task rl()
        {

            Random random = new System.Random();
            int randomRule = random.Next(5);
            int randomStage = random.Next(16);

            string Messages = "次の試合のルールは、\n ・**" + rule[randomRule].ToString() + "**\n\n";
            Messages += "次の試合のステージは、\n ・**" + stage[randomStage].ToString() + "**\n";

            await ReplyAsync(Messages);
        }
        
    }
}
```

### Program.csクラスを作成

Program.cs

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

using Discord.Commands;
using Discord.WebSocket;
using Microsoft.Extensions.DependencyInjection;
using System.Reflection;
using Discord;

namespace Splatoo2ConsoleApp
{
    class Program
    {
        public static DiscordSocketClient client;
        public static CommandService commands;
        public static IServiceProvider services;

        static void Main(string[] args) => new Program().MainAsync().GetAwaiter().GetResult();

        /// <summary>
        /// 起動時処理
        /// </summary>
        /// <returns></returns>
        public async Task MainAsync()
        {
            client = new DiscordSocketClient();
            commands = new CommandService();
            services = new ServiceCollection().BuildServiceProvider();
            client.MessageReceived += CommandRecieved;

            client.Log += Log;
            string token = "メモしたTokeを指定";
            await commands.AddModulesAsync(Assembly.GetEntryAssembly());
            await client.LoginAsync(TokenType.Bot, token);
            await client.StartAsync();

            await Task.Delay(-1);
        }

        /// <summary>
        /// メッセージの受信処理
        /// </summary>
        /// <param name="msgParam"></param>
        /// <returns></returns>
        private async Task CommandRecieved(SocketMessage messageParam)
        {
            var message = messageParam as SocketUserMessage;
            Console.WriteLine("{0} {1}:{2}", message.Channel.Name, message.Author.Username, message);

            if (message == null) { return; }
            // コメントがユーザーかBotかの判定
            if (message.Author.IsBot) { return; }

            int argPos = 0;

            // コマンドかどうか判定（今回は、「.」で判定）
            if (!(message.HasCharPrefix('.', ref argPos) || message.HasMentionPrefix(client.CurrentUser, ref argPos))) { return; }
                
            var context = new CommandContext(client, message);
            
            // 実行
            var result = await commands.ExecuteAsync(context, argPos, services);
            
            //実行できなかった場合
            if (!result.IsSuccess) { await context.Channel.SendMessageAsync(result.ErrorReason); }
                
        }

        /// <summary>
        /// コンソール表示処理
        /// </summary>
        /// <param name="msg"></param>
        /// <returns></returns>
        private Task Log(LogMessage msg)
        {
            Console.WriteLine(msg.ToString());
            return Task.CompletedTask;
        }
    }
}
```

### 作成完了！

- ビルドを実行します。

[![WS000024.JPG](https://qiita-image-store.s3.amazonaws.com/0/90597/7400425b-630b-7638-c787-b50fa68ad798.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F90597%2F7400425b-630b-7638-c787-b50fa68ad798.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=fb79d92cfaa4f79ca76d7e62df48836b)

- Discordで「.rl」コメントしてみると、Botがランダムで返してくれます。

[![WS000025.JPG](https://qiita-image-store.s3.amazonaws.com/0/90597/b9b29949-62f4-c2db-b1f0-1745f5a44f96.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F90597%2Fb9b29949-62f4-c2db-b1f0-1745f5a44f96.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=826dddb984d3f1f27cf67d6bfaf9e53e)

## まとめ

- NuGetに `Discord .Net` があるので、比較的簡単に作成することができました。
- 他にもDiscordの機能をいろいろと触れそうですが、とっかかりとしては、まずまずでした。

[2](https://qiita.com/HAGITAKO/items/#comments)

[22](https://qiita.com/HAGITAKO/items/fff2e029064ea38ff13a/likers)

37