---
title: 【前編】UnityのAddressableを個人制作で使いこなす
source: https://orotiyamatano.hatenablog.com/entry/Addressabes1#google_vignette
author:
  - "[[YAMADA TAISHI’s diary]]"
published: 2024-02-09
created: 2025-05-06
description: こんにちは、やまだたいし( https://twitter.com/OrotiYamatano )です。 UnityのAddressablesを個人でも導入したいと思い今回はそれの備忘録です。 目次 Addressablesを個人でも導入したいわけ そもそもAddressablesとはなにか? どうやって管理していくか ダウンロード 初期設定 AddressableAssetSettings Profile(プロファイル) Diagnostics (診断) Catalog (カタログ) Update a Previous Build(ビルド更新設定) Downloads (ダウンロード) Bu…
tags:
  - CSharp
  - Tech
  - 設計
  - Unity
read: false
---
こんにちは、やまだたいし( [https://twitter.com/OrotiYamatano](https://twitter.com/OrotiYamatano) )です。  
UnityのAddressablesを個人でも導入したいと思い今回はそれの備忘録です。

> **目次**

---

## Addressablesを個人でも導入したいわけ

---

Unityでアセットを読み込む場合いくつかの方法がある。  

・Unityのシーンファイルに紐づけてしまう方法  
・Resourcesに格納してしまう方法  
・Streaming Assetsとして格納してしまう方法  
・AssetBundlesとして管理する方法  
・Addressablesを使う方法  
・ContentLoadModuleを使う方法

後、例外的にEntitiesはContentLoadModule上に独自システムが構築されており、Content managementという名前の機能がある。

現在企業でも使われている方法がAddressablesだ。  
AssetBundlesを使っている企業もいるがAssetBundlesは使いづらくAddressablesへ移行が進んでいる。  

特段気にせず、Unityシーンに全て紐づけてもいいが、アプリのサイズが大きくなってしまう問題点や、  
Unityのシーンファイルを開いたときのメモリ使用量が格段に上がってしまう。

次に考えるのはResourcesだが、Unityとしては非推奨になっている。  

[learn.unity.com](https://learn.unity.com/tutorial/assets-resources-and-assetbundles#5c7f8528edbc2a002053b5a7)

個人的には個人開発者であればResourcesでも構わないと思うが、  
ストアのアプリを更新せずに中のリソースを更新したりできる利点などもあるので是非に一歩先に行きたい個人開発者は導入を考えたい。

また、多言語対応する場合はこの機能を使うのが主流となっている。

今回は私の勉強がてらAddressableについて深堀りしていこうと考えている。

正直使うだけなら他の方々のブログを参考にしていただいたほうが良いだろう。  
[www.hanachiru-blog.com](https://www.hanachiru-blog.com/entry/2022/03/21/120000)

## そもそもAddressablesとはなにか?

---

アセット管理システム。  
非同期ロードを使用して、あらゆる依存関係のコレクションが存在する、任意の場所からロードを行うことができる。

[unity.com](https://unity.com/ja/how-to/simplify-your-content-management-addressables)

## どうやって管理していくか

---

Addressableはかなり複雑だ。  
しっかり管理するなら [CDN](https://d.hatena.ne.jp/keyword/CDN) も必要だし、何と言ってもメモリ管理が複雑になる。  
まずは手始めにローカルのリソースを使う方法を調べていく。

## ダウンロード

---

まずは `PackageManager` によりインストールを行います。  
現在(2023/12/23)インストールできるもので最新は2.0.6となっている。  
安定版は1.21.19だ。

[docs.unity3d.com](https://docs.unity3d.com/Packages/com.unity.addressables@1.21/manual/index.html)

![](https://cdn-ak.f.st-hatena.com/images/fotolife/O/OrotiYamatano/20231223/20231223222701.png)

インストールすると、以下のようなメニューが追加される。 ![](https://cdn-ak.f.st-hatena.com/images/fotolife/O/OrotiYamatano/20231223/20231223222957.png)

Create Addressables Settingsをクリック。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/O/OrotiYamatano/20231223/20231223223048.png)

するとAssets配下にAddressableAssetsDataという名前でAddressablesを利用するためのデフォルト設定で初期設定ファイル郡が生成される。

## 初期設定

---

Addressablesの初期設定はいくつかある。  
各種ScriptableObjectで構成され保存されている。

## AddressableAssetSettings

---

基本設定だ。  

[docs.unity3d.com](https://docs.unity3d.com/ja/Packages/com.unity.addressables@1.20/manual/AddressableAssetSettings.html)

どうやってAddressablesを使うかの設定。  

- Local  
	ローカルでの設定。
- Remote  
	リモートでの設定。
- BuildTarget  
	ビルドターゲットの名前
- New Entry  
	ビルドしたアセットの保存先

LocalとRemoteで設定できる内容はバンドルの場所だ。  
ちなみにバンドルとはAddressablesの対象としたアセットのことだが、圧縮済みのビルド済みデータのこと。  
Local設定でありながらリモートの場所の設定ができたり、  
その逆、Remote設定でありながらLocalの指定ができたりする。ややこしい。

- Built-In  
	ローカルコンテンツ用の場所プレイヤービルドに自動的に含まれる
- Editor Hosted  
	エディターの \[[ホスティング](https://d.hatena.ne.jp/keyword/%A5%DB%A5%B9%A5%C6%A5%A3%A5%F3%A5%B0) サービス\] で使用するパス定義
- Cloud Content Delivery  
	Cloud Content Deliveryのパス。ちなみにCloud Content DeliveryとはUnity公式の [CDN](https://d.hatena.ne.jp/keyword/CDN) みたいなもの。
- Custom  
	自由に設定ができる。

→ [ホスティング](https://d.hatena.ne.jp/keyword/%A5%DB%A5%B9%A5%C6%A5%A3%A5%F3%A5%B0) サービスってどこのことを言っているかというとココ  
![](https://cdn-ak.f.st-hatena.com/images/fotolife/O/OrotiYamatano/20231224/20231224011332.png)

今回はLocalで使うのでどちらとも一旦ビルドインにしておく。

ちなみにLocalとRemoteどちらを使うのか指定は別途する箇所があるのだがコチラは後述する。

### Diagnostics (診断)

ログ出力のためのオプション。

- Send Profiler Events  
	プロファイラーイベントを有効化
- Log Runtime Exceptions  
	アセットのロード操作で発生したランタイム例外をログに記録

### Catalog (カタログ)

どこからなんのアセットをとってくるか情報が書かれているCatalog,そのカタログの設定。  
ちなみにLocalのみでアセットを管理している場合はCatalogの存在はほとんど意識する必要はない  

- Player Version Override  
	カタログのPlayerVersionの上書き
- Compress Local Catalog  
	Localのカタログの圧縮設定をするか
- Build Remote Catalog  
	リモート カタログを構築設定.  
	読み込む場所を指定する。
- Only update catalogs manually  
	リモート カタログの自動チェックが無効化

### Update a Previous Build(ビルド更新設定)

- Check for Update Issues  
	Check for Content Update Restrictions ツール(アセットに問題があったときに対処するツール)で  
	どのように対処するかの設定?もしくはどのような場合に更新させるかの設定。読んでもよくわからんかった。  
	List Restricted Assets (recommended) と書いてる通りList Restricted Assetsが推奨そうなのでList Restricted Assetsで良さそうだ。  
	多分リストに差分があった場合に更新してくれる。
- Content State Build Path  
	コンテンツ状態ビルド先の設定

### Downloads (ダウンロード)

- Custom certificate handler  
	[Https](https://d.hatena.ne.jp/keyword/Https) の証明書確認用。  
	Unityengine.Network.CertificateHandlerクラスを拡張して実装するらしい。
- Max Concurrent Web Requests  
	最大WebrRequest数,2~4 が最適な同時リク [エス](https://d.hatena.ne.jp/keyword/%A5%A8%A5%B9) ト数らしい。
- Catalog Download Timeout  
	カタログダウンロードの [タイムアウト](https://d.hatena.ne.jp/keyword/%A5%BF%A5%A4%A5%E0%A5%A2%A5%A6%A5%C8) 時間の設定。  
	0は [タイムアウト](https://d.hatena.ne.jp/keyword/%A5%BF%A5%A4%A5%E0%A5%A2%A5%A6%A5%C8) なし.

### Build (ビルド)

- Build Addressables on Player Build  
	Addressableがプレイタービルドの一部としてビルドされるかどうか?  
	Build Addressables content on Player Build  
	→プレイヤーのビルド時に常に Addressables コンテンツをビルドする  
	Do not Build Addressables content on Player Build  
	→Addressables コンテンツをビルドしない。Addressables コンテンツを変更した場合は手動でのビルドが必要。  
	Use global Settings (stored in preferences)  
	→ エディターの環境設定に従う。
- Ignore Invalid/Unsupported Files in Build  
	無効なファイルがあったときの挙動の指定。  
	チェックがある場合はビルドを中止するのではなく、それらのファイルを除外
- Unique Bundle IDs  
	ビルドで一意のバンドル名を作成するかどうか
- Contiguous Bundles  
	効率的なバンドルレイアウトを作成するかどうか。
- Non-Recursive [Dependency](https://d.hatena.ne.jp/keyword/Dependency) Calculation  
	有効にすると、アセットに循環依存関係があるときにビルド時間が短縮  
	注意点:循環依存関係によっては、このオプションを有効にするとロードできない場合があるらしい.

### Build and Play Mode Scripts (ビルドスクリプトと再生モードスクリプト)

Play Mode Scriptでの読み込み設定  
どのようにビルドプロセスを処理をさせるかそれぞれ処理が入っている。  
特に弄ることはなさそう。

### Asset Group Templates(アセットグループテンプレート)

作られるアセットグループのテンプレ設定を置いておく。

### Initialization Objects(初期化オブジェクト)

Addressables.InitializeAsync呼んだときの初期設定をここに詰め込むっぽい。  
Unity公式として設定があるのはキャッシュ初期設定だけだけど、  
ユーザーでカスタマイズしてScriptableObject形式で処理を追加できるらしい?  
特段触る必要はなさそう。

## Group settings(グループ設定)

Addressablesにはそれぞれグループごとに設定ができる。  

### Content Update Restriction

- Prevent Updates(アップデート禁止)  
	チェックが入っている場合、バンドル内のアセットが変更されている場合のみ、バンドル全体が再構築  
	チェックがない場合は毎回上書きされる
- Build & LoadPaths  
	biuld path設定.  
	Profile(プロファイル)で設定した内容
- Advanced Options
	- Asset Bundle Compression  
		圧縮形式の指定
	- Include In Build  
		このグループのアセットをコンテンツビルドに入れるかどうか。
	- Force Unique Provider
	- Use Asset Bundle Cache  
		リモート配布するバンドルをキャッシュするかどうか。
	- Asset Bundle [CRC](https://d.hatena.ne.jp/keyword/CRC)  
		ロード前に、バンドルの整合性を確認するかどうか  
		Disabled(無効)、  
		Enabled, Including Cached (有効、キャッシュされたものを含む)、  
		Enabled, Excluding Cached (有効、キャッシュされたものを除く)
	- Use UnityWebRequest for Local Asset Bundles  
		ローカル AssetBundle [アーカイブ](https://d.hatena.ne.jp/keyword/%A5%A2%A1%BC%A5%AB%A5%A4%A5%D6) をロードするときに、AssetBundle.LoadFromFileAsync ではなく  
		UnityWebRequestAssetBundle.GetAssetBundle を使用
	- Request Timeout  
		リモートバンドルのダウンロードの [タイムアウト](https://d.hatena.ne.jp/keyword/%A5%BF%A5%A4%A5%E0%A5%A2%A5%A6%A5%C8) 間隔。
	- Use Http Chunked Transfer  
		バンドルのダウンロード時に HTTP/1.1 チャンク転送 [エンコーディング](https://d.hatena.ne.jp/keyword/%A5%A8%A5%F3%A5%B3%A1%BC%A5%C7%A5%A3%A5%F3%A5%B0) 方式を使用するか  
		非推奨らしいので使わなくていい。
	- Http Redirect Limit  
		バンドルのダウンロード時のHttpリダイレクト許容数。-1だと無制限.
	- Retry Count  
		ダウンロードが失敗したときに再試行する回数。
	- Include Addresses in Catalog  
		カタログにアドレス文字列を入れるかどうか  
		入れない場合はカタログサイズが小さくなるらしい.
	- Include GUIDs in Catalog  
		カタログに GUID 文字列を入れるかどうか。  
		AssetReference を使用してアセットにアクセスするなら必須らしい。  
		入れない方が軽量化できるらしい
	- Include Labels in Catalog  
		カタログにラベル文字列を入れるかどうか  
		ラベルを使用しない場合にOffにできる、まあない方が軽量化できるらしい
	- Internal Asset Naming Mode  
		AssetBundle 内のアセットの ID を決定の指定  
		Full Path: プロジェクト内のアセットのパス  
		Filename: アセットのファイル名(もちろん同一ファイル名禁止)  
		GUID:GUID  
		Dynamic:(文字情報がすくなくなるから)推奨らしい グループ内のアセットに基づいて作成できる最も短い ID
	- Internal Bundle Id Mode  
		AssetBundle が内部的にどのように識別されるかを決定の指定  
		Group Guid:推奨グループの一意の ID  
		Group Guid Project Id Hash:グループ GUID と [クラウド](https://d.hatena.ne.jp/keyword/%A5%AF%A5%E9%A5%A6%A5%C9) プロジェクト ID  
		Group Guid Project Id Entries Hash: グループ GUID、 [クラウド](https://d.hatena.ne.jp/keyword/%A5%AF%A5%E9%A5%A6%A5%C9) プロジェクト ID
	- Cache Clear Behavior  
		キャッシュから AssetBundle を消去するタイミングを決定  
		ClearWhenSpaceIsNeededInCache:キャッシュにスペースが必要な場合はクリア  
		ClearWhenWhenNewVersionLoaded:新しいバージョンが正常にロードされると、バンドルはキャッシュから削除  
		古いの残ってても仕方ないしClearWhenWhenNewVersionLoadedでよくね?
	- Bundle Mode  
		バンドルにパックする方法の指定  
		Pack Together: すべてのアセットを含む 1 つのバンドルを作成  
		Pack Separately: グループ内のプライマリアセットごとにバンドルを作成  
		Pack Together by Label: 同じ組み合わせのラベルを共有するアセットのバンドルを作成  
		うーん、特にこだわりが無ければPack Togetherか?
	- Bundle Naming Mode  
		AssetBundle のファイル名を作成する方法の指定  
		Filename: グループ名から派生した文字列  
		Append Hash to Filename: グループ名から派生した文字列+バンドルハッシュを付加した文字列  
		Use Hash of AssetBundle: バンドルハッシュ  
		Use Hash of Filename: Filenameから計算されたハッシュ
	- Asset Load Mode  
		アセットを個別にロードする (デフォルト) か、グループ内のすべてのアセットを常にまとめてロードするか指定  
		Requested Asset and Dependencies(Requestアセットとその依存)が推奨らしい
	- Asset Provider  
		「AssetBundle内のアセット」をロードするためのプロバイダークラスを定義  
		カスタムで作ってなければAssets from Bundles Provider
	- Asset Bundle Provider  
		「AssetBundle」をロードするためのプロバイダークラスを定義  
		カスタムで作ってなければ AssetBundle Provider

## だいぶ理解が進んだ

---

だいぶ理解が進んだが、行数も多くなってきたので今回の記事はココまでにしておく

[orotiyamatano.hatenablog.com](https://orotiyamatano.hatenablog.com/entry/Addressabes2)

[« URPのライティングについて](https://orotiyamatano.hatenablog.com/entry/URPLight) [広告掲載初めて1年経ったので広告収益を晒… »](https://orotiyamatano.hatenablog.com/entry/2024/02/04/revenue)