---
title: Unity C#の関数を誰でも扱えるように。『FF7EC』におけるスクリプト言語「Lua」活用テクニック【CAGC2024】｜ゲームメーカーズ
source: https://gamemakers.jp/article/2024_03_22_63130/
author:
  - "[[ゲームづくりの最新情報やアイデアを紹介！｜ゲームメーカーズ]]"
published: 
created: 2025-05-06
description: サイバーエージェントのゲーム・エンターテイメント事業部（SGE）初開催のカンファレンス「CyberAgent Game Conference 2024」が、2024年3月7日に開催されました。『FINAL FANTASY VII EVER CRISIS』（以下、『FF7EC』）の掛け合い部分や、ダンジョン中のギミックなどで用いられたLuaスクリプトの活用方法について語られた、アプリボット クライアントエンジニア 佐藤 彩理氏によるセッション「大規模開発におけるLuaスクリプトの活用方法」をレポートします。
tags:
  - Tech
  - Unity
  - "#Lua"
read:
---
###### サイバーエージェントのゲーム・エンターテイメント事業部（SGE）初開催のカンファレンス「CyberAgent Game Conference 2024」が、2024年3月7日に開催されました。 『FINAL FANTASY VII EVER CRISIS』（以下、『FF7EC』）の掛け合い部分や、ダンジョン中のギミックなどで用いられたLuaスクリプトの活用方法について語られた、アプリボット クライアントエンジニア 佐藤 彩理氏によるセッション「大規模開発におけるLuaスクリプトの活用方法」をレポートします。

###### TEXT / じく EDIT / 神谷 優斗

##### 目次

- 1.
	###### Luaスクリプトを採用した理由とは
	[View original](https://gamemakers.jp/article/2024_03_22_63130/#section_1)
- 2.
	###### 『FF7EC』におけるLuaスクリプト使用例
	[View original](https://gamemakers.jp/article/2024_03_22_63130/#section_2)
- 3.
	###### C#とLuaを用いた開発フロー
	[View original](https://gamemakers.jp/article/2024_03_22_63130/#section_3)
- 4.
	###### Luaのメリットと採用時の考慮点
	[View original](https://gamemakers.jp/article/2024_03_22_63130/#section_4)

## Luaスクリプトを採用した理由とは

###### 『FF7EC』の開発では、「プレイアブルなエリアでのイベント管理」や「キャラクター同士の掛け合いのあるアドベンチャーパート」を共通の仕組みで制作したいニーズがありました。開発メンバーのスキルセットを考慮に入れつつニーズを達成する手段を検討した結果、スクリプト言語を採用するに至ったそうです。

![](https://gamemakers.jp/cms/wp-content/uploads/2024/03/c92549afa37f7101881715d04fdc4c5f.jpg)

###### 採用するスクリプト言語としてLuaを選択した理由としては、メジャーである点、スクリプターの採用がしやすい点の2つが挙げられました。

## 『FF7EC』におけるLuaスクリプト使用例

###### 『FF7EC』で、Luaは「ダンジョン中のギミック」「アドベンチャーパートの掛け合い」「チョコボ農場」の3か所で使用されています。

### ダンジョン中のギミック

###### 「特定の場所を通ったときに石が落ちてくるイベント」や「選択肢に応じてアイテムが取得できるオブジェクト」といった、ダンジョンにある一部のギミックはLuaスクリプトで実装されています。

![](https://gamemakers.jp/cms/wp-content/uploads/2024/03/50e16150455ebceff97b9587f4068668.jpg)

ストーリーパートで使われるマップにあるギミックもLuaで実装されている

###### ギミックのLuaスクリプトは、プランナーがマップの制作時に併せて作成しているとのこと。

### アドベンチャーパートの掛け合い

###### ストーリー中のアドベンチャーパートにおける、キャラクターのモーション再生やエフェクト・効果音の再生に加え、セリフ表示などの機能を用いた演出はすべてLuaスクリプトによって構成されています。

![](https://gamemakers.jp/cms/wp-content/uploads/2024/03/e259a366f6da23e527acfc9b92824cf9.jpg)

###### 各機能の実装はエンジニアが担当し、Luaによる演出のスクリプト作成はスクリプターが担当しています。

### チョコボ農場

###### チョコボ農場では、「遠征（※）」で得られた報酬に応じてオブジェクトの見た目が変わる仕組みや、チョコボを含めたNPCとの会話などの実装がLuaスクリプトです。 ※　チョコボを遠征地に探索に出し、アイテムやキャラクターの経験値を持ち帰らせるシステム

![](https://gamemakers.jp/cms/wp-content/uploads/2024/03/22dfc9b424c9b7b0b744a63210dd4cb7.jpg)

チョコボに話しかけるたびにセリフが変わる仕組みもLuaスクリプトで実装されている

## C#とLuaを用いた開発フロー

###### 次に、佐藤氏はLuaスクリプトを用いた具体的な開発フローについて説明しました。

### C#の関数定義とLuaからの呼び出し

###### 『FF7EC』の開発では、C#で定義した関数をLuaスクリプトから呼び出す形で実装しています。Luaからの呼び出しを想定していないC#関数もあるため、事前に登録された関数のみがLuaスクリプトから呼び出せる仕組みになっています。 C#で「move(float x, float y, float z)」という関数を定義した場合、LuaからはC#と同じように「move(1.0, 2.0, 3.0)」と記述することでmove関数を呼び出せるようになっています。 また、Luaは計算式やif文に対応しています。そのため、選択肢に応じた分岐処理もC#側で実装することなくLuaスクリプト内で完結します。

![](https://gamemakers.jp/cms/wp-content/uploads/2024/03/1f66f48471c6939f546ce91b859391eb.jpg)

上記右のように「インデックスが1の選択肢を選んだ場合は座標（1.0, 2.0, 3.0）に移動する」処理を記述できる

### C#のクラスをLuaに公開する独自Attribute

###### 『FF7EC』では、C#のクラスをLuaに公開する独自のAttributeを活用。Attributeを付与したクラスのインスタンスは、Lua上でC#と同様にメンバ変数やメンバ関数にアクセスできます。

![](https://gamemakers.jp/cms/wp-content/uploads/2024/03/5d35680417da289fc0fb739de0c3bda2.jpg)

Luaスクリプトで扱うクラスに「LuaClass」Attributeを付与することで、C#上と同じようにインスタンスを扱える

### C#とLuaを組み合わせた演出の実装例

###### ここで、制作フローの具体例として「キャラクターが歩いてきてセリフを言う」シーンが挙げられました。 まず、エンジニアがC#で特定の座標まで歩くmove関数とセリフを表示するtalk関数を定義します。関数は、Luaスクリプトを特に意識せずに実装します。

![](https://gamemakers.jp/cms/wp-content/uploads/2024/03/79090237bfbe41930966c22b4164d37d.jpg)

###### 次に、スクリプターやプランナーがLua上でC#関数を使用した演出のスクリプトを記述します。

![](https://gamemakers.jp/cms/wp-content/uploads/2024/03/c98f8bf5f6ab5823768cb627640f32d7.jpg)

###### 上記画像のLuaスクリプトは、まずCharacterClassのインスタンスであるCharacterのmove関数を呼び出して指定の座標に移動させ、移動が完了するのを待ってからtalk関数で「到着」というセリフを表示します。Luaスクリプトでは、演出を作る上で欠かせない非同期処理も記述できるようになっています。

### Visual Studio Codeスニペット生成の自動化

###### Luaスクリプトの記述には、Visual Studio Codeが使用されています。 開発では、C#関数に付与したAttributeからVisual Studio Codeのスニペットを自動生成する仕組みを開発し、スクリプティングの効率化を図っています。

![](https://gamemakers.jp/cms/wp-content/uploads/2024/03/c9b96f67f44b58f457d8112684530597.jpg)

関数に「LuaFunctionSetting」Attributeと説明文を記述すると、自動的にスニペットが出力される

###### 『FF7EC』のワークフローでは、基本的に関数を用意する人と使用する人は異なります。そのため、なるべく低コストでドキュメントに近いものが出力できるよう工夫をしているそうです。

![](https://gamemakers.jp/cms/wp-content/uploads/2024/03/aeabc5e9c492045ff91416514f51b81b.jpg)

## Luaのメリットと採用時の考慮点

###### 最後に、佐藤氏はLuaスクリプトを採用するメリットと、採用時に考慮すべき点を紹介しました。

### Luaを採用する3つのメリット

###### Luaスクリプト採用のメリットは、以下の3点が挙げられました。

###### エンジニアがC#の関数を用意した後はスクリプターのみで完結するため、自走しやすい環境にできる UnityにはLua言語を扱うためのライブラリが豊富にあるため、大規模なツールを開発する必要がない プロジェクト独自のスクリプト言語ではないため、使用経験のあるスクリプターを採用できる。スクリプターの市場価値が向上する

###### スクリプターが自走しやすい環境の具体例としては、下記画像に示す「回転する扉」が紹介されました。

![](https://gamemakers.jp/cms/wp-content/uploads/2024/03/131e8f5b21212c543fae69fa56cb7be2.jpg)

プレイヤーがアクションすると扉が回転し、向こう側に移動できるギミック

###### このギミックは、「キャラクターのモーション再生」「オブジェクトの回転」「キャラクターの移動」という関数の組み合わせで実装されています。事前に基本的な関数をエンジニアで用意しておけば、ギミックをスクリプターだけで制作できます。

![](https://gamemakers.jp/cms/wp-content/uploads/2024/03/1742e81f65db1989bb23c5bfe9932c4c.jpg)

### Luaを取り入れる上での考慮点

###### Luaを採用する際に考慮すべき点もいくつか挙げられました。 スクリプターが自走しやすい環境はメリットである一方、エンジニアが把握していない新しい機能が生まれる可能性もあります。エンジニアの工数が減る反面、いつの間にか想定していない関数の使い方によってバグが発生していた、ということが起こり得ます。そのため、フロー整備などを事前に考慮しておく対策が必要とのこと。 また、自由度が高いがゆえに制御しづらくなってしまう懸念点もあります。Luaスクリプトが煩雑化しバグ対応などがしづらくなってしまったため、実装を最終的にC#へ寄せたギミックもいくつかあったそうです。先ほど例に挙げた回転する扉も、中断処理などを考慮した結果、最終的にはC#で実装されることとなりました。 C#への移行作業自体は、Lua経由で呼んでいた関数をそのままC#で呼び出すようにするのみで完結するため、コストは大きくありません。ただし、「どこまでLuaスクリプトで作るのか」を意識して開発を進めた方がよいと佐藤氏は語りました。

![](https://gamemakers.jp/cms/wp-content/uploads/2024/03/b3d6c25b5e93d21907aa48007b5697e8.jpg)

###### 最後に佐藤氏は、「Luaを導入する際は、コーディングルールを決めるなど開発の地盤を整えておくと扱いやすくなる。Luaスクリプトに頼りすぎず、時にはC#の実装に寄せる判断も大事である。」と語り、講演を締めくくりました。

[「大規模開発におけるLuaスクリプトの活用方法」CAGC2024公式サイト](https://cagc.cyberagent.co.jp/2024/session/index.html?id=Tr7uAzLn) [『FINAL FANTASY VII EVER CRISIS』公式サイト](https://jp.ffviiec.com/)

ゲーム会社で16年間、マニュアル・コピー・シナリオとライター職を続けて現在フリーライターとして活動中。 ゲーム以外ではパチスロ・アニメ・麻雀などが好きで、パチスロでは他媒体でも記事を執筆しています。 SEO検定１級（全日本SEO協会）、日本語検定 準１級＆２級（日本語検定委員会）、DTPエキスパート・マイスター（JAGAT）など。

### 注目記事ランキング

2025.04.29 - 2025.05.06###### [1](https://gamemakers.jp/article/2025_04_30_101293/)

[オープンソースのRust製ゲームエンジン「Bevy Engine」、バージョン0.16にアップデート。GPU駆動レンダリングの導入や、各Entityに自由な関係性を設定可能に](https://gamemakers.jp/article/2025_04_30_101293/)2

『GUILTY GEAR』シリーズで採用された、アウトラインを部分的に隠す技法を紹介。アークシステムワークスによる「背面法」解説動画シリーズが更新

[View original](https://gamemakers.jp/article/2025_05_01_101474/)3

「Unity Asset Manager」の長所や使用方法、Unity公式が解説。Blender上で編集したアセットの更新内容をUnityに反映する手順なども紹介

[View original](https://gamemakers.jp/article/2025_04_30_101317/)4

Unityでのマルチプレイヤーゲームのネットワークシステムの入門書、公式サイトにて日本語版電子書籍が無料公開。基本概念からUnity 6での実践例まで

[View original](https://gamemakers.jp/article/2025_04_28_100978/)5

Razer初の“縦型”ワイヤレスマウス「Razer Pro Click V2 Vertical」が発表。長時間の仕事やゲームプレイも快適にこなせると謳う設計が特徴

[View original](https://gamemakers.jp/article/2025_04_28_101037/)###### [1](https://gamemakers.jp/article/2025_04_30_101293/)

[オープンソースのRust製ゲームエンジン「Bevy Engine」、バージョン0.16にアップデート。GPU駆動レンダリングの導入や、各Entityに自由な関係性を設定可能に](https://gamemakers.jp/article/2025_04_30_101293/)2

『GUILTY GEAR』シリーズで採用された、アウトラインを部分的に隠す技法を紹介。アークシステムワークスによる「背面法」解説動画シリーズが更新

[View original](https://gamemakers.jp/article/2025_05_01_101474/)3

「Unity Asset Manager」の長所や使用方法、Unity公式が解説。Blender上で編集したアセットの更新内容をUnityに反映する手順なども紹介

[View original](https://gamemakers.jp/article/2025_04_30_101317/)4

Unityでのマルチプレイヤーゲームのネットワークシステムの入門書、公式サイトにて日本語版電子書籍が無料公開。基本概念からUnity 6での実践例まで

[View original](https://gamemakers.jp/article/2025_04_28_100978/)5

Razer初の“縦型”ワイヤレスマウス「Razer Pro Click V2 Vertical」が発表。長時間の仕事やゲームプレイも快適にこなせると謳う設計が特徴

[View original](https://gamemakers.jp/article/2025_04_28_101037/)###### [1](https://gamemakers.jp/article/2025_04_09_97671/)

[【制作期間3年半】『Skyrim』『Fallout』開発者が教える、個人開発でオープンワールドゲーム『The Axis Unseen』を作る方法【GDC2025】](https://gamemakers.jp/article/2025_04_09_97671/)2

『VALORANT』のFPSにおけるアニメーションの極意。快適なプレイもクールな演出も、わずか数名のチームで作り上げる！【GDC2025】

[View original](https://gamemakers.jp/article/2025_04_30_101205/)3

『呪術廻戦 ファンパレ』Unityで高品質2Dアニメーションを実装。アニメライクなカットイン演出や、ADVに適したリアルタイム映像表現の制作フロー【サムザップテックナイトvol9】

[View original](https://gamemakers.jp/article/2025_04_25_96303/)4

ゲームづくりに役立つ無料&商用利用可能な素材サイトまとめ。3Dモデルやサウンド、フォントなどを一気に揃えよう

[View original](https://gamemakers.jp/article/2024_01_26_59516/)5

RPG制作を支援する7種の主要ソフトを厳選して紹介【アクションパズル・シミュレーションなどの制作にも】

[View original](https://gamemakers.jp/article/2024_06_18_68300/)###### [1](https://gamemakers.jp/article/2025_05_01_101474/)

[『GUILTY GEAR』シリーズで採用された、アウトラインを部分的に隠す技法を紹介。アークシステムワークスによる「背面法」解説動画シリーズが更新](https://gamemakers.jp/article/2025_05_01_101474/)2

ゲームづくりに役立つ無料&商用利用可能な素材サイトまとめ。3Dモデルやサウンド、フォントなどを一気に揃えよう

[View original](https://gamemakers.jp/article/2024_01_26_59516/)3

【Unity】よく使うショートカットまとめ

[View original](https://gamemakers.jp/article/2024_04_04_63739/)4

業界のプロも薦めるゲームクリエイター向けの書籍28冊を一挙紹介！【プログラミングからプロデュースまで】

[View original](https://gamemakers.jp/article/2024_06_04_69465/)5

アークライト 野澤 邦仁のボードゲームを作るには Vol.01「企画編」

[View original](https://gamemakers.jp/article/2023_04_17_36392/)###### [1](https://gamemakers.jp/article/2025_04_09_97671/)

[【制作期間3年半】『Skyrim』『Fallout』開発者が教える、個人開発でオープンワールドゲーム『The Axis Unseen』を作る方法【GDC2025】](https://gamemakers.jp/article/2025_04_09_97671/)2

『VALORANT』のFPSにおけるアニメーションの極意。快適なプレイもクールな演出も、わずか数名のチームで作り上げる！【GDC2025】

[View original](https://gamemakers.jp/article/2025_04_30_101205/)3

『呪術廻戦 ファンパレ』Unityで高品質2Dアニメーションを実装。アニメライクなカットイン演出や、ADVに適したリアルタイム映像表現の制作フロー【サムザップテックナイトvol9】

[View original](https://gamemakers.jp/article/2025_04_25_96303/)4

初音ミクと『Fit Boxing』がコラボ。公式イメージを守りつつ「スポーツするミク」を3Dで表現するためのモーキャプや衣装の工夫をレポート【GCC2025】

[View original](https://gamemakers.jp/article/2025_04_23_99605/)5

作画崩壊を防ぐには“違和感”を減らすべし。アニメ監督直伝「リアルで綺麗な映像作り」が学べる講演をレポート【サムザップテックナイトvol9】

[View original](https://gamemakers.jp/article/2025_04_25_96345/)###### [1](https://gamemakers.jp/article/2025_05_02_101798/)

[App Storeの外部決済による手数料が廃止へ――米連邦地裁がAppleに命令。裁判で争ったEpic GamesはiOS版『フォートナイト』復活を予告](https://gamemakers.jp/article/2025_05_02_101798/)2

フォートナイト クリエイティブとUEFNで使える仕掛け一覧 Vol.1「アイテム系」

[View original](https://gamemakers.jp/article/2023_04_10_36123/)3

フォートナイト クリエイティブとUEFNで使える仕掛け一覧

[View original](https://gamemakers.jp/article/2023_05_09_37697/)4

『フォートナイト』で動く本格的なゲームが作れるツール「UEFN」とは？従来のクリエイティブモードから進化したポイントを一挙紹介！

[View original](https://gamemakers.jp/article/2023_04_06_35870/)5

【CHALLENGE1】「クリエイター ポータル」を使って、UEFNで作成した島を世界中に公開する

[View original](https://gamemakers.jp/article/2023_06_09_40315/)###### [1](https://gamemakers.jp/article/2025_04_28_101037/)

[Razer初の“縦型”ワイヤレスマウス「Razer Pro Click V2 Vertical」が発表。長時間の仕事やゲームプレイも快適にこなせると謳う設計が特徴](https://gamemakers.jp/article/2025_04_28_101037/)2

ゲームづくりに役立つ無料&商用利用可能な素材サイトまとめ。3Dモデルやサウンド、フォントなどを一気に揃えよう

[View original](https://gamemakers.jp/article/2024_01_26_59516/)3

RPG制作を支援する7種の主要ソフトを厳選して紹介【アクションパズル・シミュレーションなどの制作にも】

[View original](https://gamemakers.jp/article/2024_06_18_68300/)4

国内ゲームデベロッパー 一覧

[View original](https://gamemakers.jp/company-list)5

【2025年5月版】注目のゲーム展示会・コンテスト・カンファレンス・勉強会情報まとめ

[View original](https://gamemakers.jp/article/2025_04_30_101121/)