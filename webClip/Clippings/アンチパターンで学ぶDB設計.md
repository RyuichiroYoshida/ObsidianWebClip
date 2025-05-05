---
title: アンチパターンで学ぶDB設計
source: https://qiita.com/nem_Nuco/items/6a00e1764628b1b3fefb
author:
  - "[[Qiita]]"
published: 2024-07-29
created: 2025-05-06
description: はじめにデータベース（DB）の設計は、システムの性能や保守性に大きな影響を与えます。この記事では、最低限パフォーマンスの低下や管理の複雑化を引き起こさないようにするために覚えておくべきことを、ア…
tags:
  - 1
  - バックエンド
  - データベース
  - バックエンド設計
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

[@nem\_Nuco](https://qiita.com/nem_Nuco) in [株式会社Nuco](https://qiita.com/organizations/nuco-inc)

投稿日

## はじめに

データベース（DB）の設計は、システムの性能や保守性に大きな影響を与えます。

この記事では、 **最低限パフォーマンスの低下や管理の複雑化を引き起こさないようにする** ために覚えておくべきことを、アンチパターンとしてまとめました。

本記事は、

- 現在仕事でデータベースを扱っており、データ設計について今一度おさらいしたい
- データベースについての基礎知識やお作法を身に付けたい

という人を対象として想定しています。

これらに当てはまる方はぜひ一度確認してみてください！

弊社Nucoでは、他にも様々なお役立ち記事を公開しています。よかったら、Organizationのページも覗いてみてください。  
また、Nucoでは一緒に働く仲間も募集しています！興味をお持ちいただける方は、 [こちら](https://www.recruit.nuco.co.jp/?qiita_item_id=6a00e1764628b1b3fefb) まで。

## DB設計アンチパターン

早速、DB設計におけるアンチパターンを紹介します。  
それぞれアンチパターンのテーブルを見て、どのような修正が必要かを考えてみてください。

## 1\. データをまとめすぎる

データを一つのテーブルにまとめすぎると可読性が低下し、修正が必要になった場合の影響範囲が広がります。  
さらに、同じ値が複数回登場することによってストレージの容量も無駄に食われます。  
データをまとめるとER図がすっきりするためやってしまいがちですが、デメリットの方が大きいことを覚えておきましょう。

**アンチパターンの例:**

| ID | Name | Affiliation | Email | Phone |
| --- | --- | --- | --- | --- |
| 0 | 高橋さん | 経理課 | [takahashi@example.com](https://qiita.com/nem_Nuco/items/) | 123-456-7890 |
| 1 | 鈴木さん | 営業課 | [suzuki@example.com](https://qiita.com/nem_Nuco/items/) | 234-567-8901 |
| 2 | 田中さん | 開発課 | [tanaka@example.com](https://qiita.com/nem_Nuco/items/) | 345-678-9012 |
| 3 | 佐藤さん | 営業課 | [sato@example.com](https://qiita.com/nem_Nuco/items/) | 456-789-0123 |

この例では、Affiliation(所属)に重複した値が入っており、  
このような例はよく「正規化が不十分」と言われます。

**対策:**  
必要な正規化をする

**修正例**

`Users` テーブル

| ID | Name | AffiliationID | Email | Phone |
| --- | --- | --- | --- | --- |
| 0 | 高橋さん | 0 | [takahashi@example.com](https://qiita.com/nem_Nuco/items/) | 123-456-7890 |
| 1 | 鈴木さん | 1 | [suzuki@example.com](https://qiita.com/nem_Nuco/items/) | 234-567-8901 |
| 2 | 田中さん | 2 | [tanaka@example.com](https://qiita.com/nem_Nuco/items/) | 345-678-9012 |
| 3 | 佐藤さん | 1 | [sato@example.com](https://qiita.com/nem_Nuco/items/) | 456-789-0123 |

`Affiliation` テーブル

| AffiliationID | Name |
| --- | --- |
| 0 | 経理課 |
| 1 | 営業課 |
| 2 | 開発課 |

## 2\. 途中で格納するデータの種類が変更されている

格納するデータの種類が途中で変更されると、ダブルミーニング(一つのカラムに複数の種類のデータが格納されている状態のこと)が発生します。

**アンチパターンの例:**

`Affiliation` テーブル

| AffiliationID | Name |
| --- | --- |
| 0 | 経理課 |
| 1 | 営業課 |
| 2 | 0 |
| 3 | 1 |
| 4 | 0 |

この例では、 `Status` カラムに異なる種類のデータが混在しており、Statusカラムの一貫性が失われています。このテーブルを処理する時に不具合が起きるのは明白でしょう。

**対策:**  
データの種類を一貫させる

**修正例**

`Affiliation` テーブル

| AffiliationID | Name |
| --- | --- |
| 0 | 経理課 |
| 1 | 営業課 |
| 2 | 開発課 |

## 3\. 抽象的なカラム名で管理する

適切な命名がめんどくさくて一旦仮の命名をしてしまったパターンです。  
特に複数人のチームで管理する場合はダブルミーニングなどのミスが起きやすくなってしまいます。

**アンチパターンの例:**

`Table1` テーブル

| ID | Data1 | Data2 | Data3 | Data4 |
| --- | --- | --- | --- | --- |
| 0 | 高橋さん | 0 | [takahashi@example.com](https://qiita.com/nem_Nuco/items/) | 123-456-7890 |
| 1 | 鈴木さん | 1 | [suzuki@example.com](https://qiita.com/nem_Nuco/items/) | 234-567-8901 |
| 2 | 田中さん | 2 | [tanaka@example.com](https://qiita.com/nem_Nuco/items/) | 345-678-9012 |
| 3 | 佐藤さん | 1 | [sato@example.com](https://qiita.com/nem_Nuco/items/) | 456-789-0123 |

Data2に至っては何のIDなのかが分からなくなってしまっています。  
テーブル名まで抽象的な命名がされていますね。

**対策:**  
カラムの意味を明確に定義し、カラム名でそれがはっきり分かるように命名する。

**修正例**

`Users` テーブル

| ID | Name | AffiliationID | Email | Phone |
| --- | --- | --- | --- | --- |
| 0 | 高橋さん | 0 | [takahashi@example.com](https://qiita.com/nem_Nuco/items/) | 123-456-7890 |
| 1 | 鈴木さん | 1 | [suzuki@example.com](https://qiita.com/nem_Nuco/items/) | 234-567-8901 |
| 2 | 田中さん | 2 | [tanaka@example.com](https://qiita.com/nem_Nuco/items/) | 345-678-9012 |
| 3 | 佐藤さん | 1 | [sato@example.com](https://qiita.com/nem_Nuco/items/) | 456-789-0123 |

## 4\. 1つのカラムを複数の目的で使用する

柔軟性を求めて、1つのカラムに複数の意味を持たせようとするパターンです。  
これはEAV（Entity-Attribute-Value）モデルと呼ばれ、管理が複雑になる原因となります。

**アンチパターンの例:**  
`EAV` テーブル

| EntityID | Attribute | Value |
| --- | --- | --- |
| 0 | Name | 高橋さん |
| 0 | Email | [takahashi@example.com](https://qiita.com/nem_Nuco/items/) |
| 1 | Name | 鈴木さん |
| 1 | Phone | 234-567-8901 |
| 2 | Name | 田中さん |
| 2 | Affiliation | 開発課 |

EntityIDごとに、それぞれ異なる種類のデータを格納していることが分かります。  
この場合だと、例えば電話番号を取得したい時にこのテーブルを参照してしまうと、不具合が起きることが分かります。  
また、 `Value` カラムが柔軟な列であることから、格納される値のデータ型が定まらないというデメリットがあります。

**対策:**

- できる限りこの形は使わない
- 抽象的なカラムは許さない。意味をはっきりとさせる。

## 5\. 複数の意味を持ったIDを1つのカラムで管理する

世の中に存在するIDは、1つのIDで複数の意味を持っていることが多いです。  
それをそのまま1つのカラムで管理してしまうと、後で複雑な処理が必要になってしまいます。

**アンチパターンの例:**

`StudentID` テーブル

| ID | StudentID |
| --- | --- |
| 0 | A-22-0123 |
| 1 | A-23-0053 |
| 2 | B-24-0156 |
| 3 | B-24-0158 |

この例では、学籍番号が"A","22","0123"という3つの意味の区切りに分かれていることが分かります。  
これらを1つのカラムで管理するのは危険です。  
この例では"22"部分は入学年を想定していますが、学生を入学年で絞り込むクエリを作りたい時に余計な処理が必要になってしまいます。

**対策:**

- 複数の意味を持つIDを管理する必要がある場合は、別々のカラムを使用する

**修正例**

`StudentID` テーブル

| ID | Affiliation | Admission | Individual |
| --- | --- | --- | --- |
| 0 | A | 22 | 0123 |
| 1 | A | 23 | 0053 |
| 2 | B | 24 | 0156 |
| 3 | B | 24 | 0158 |

## 6\. カラムに2つ以上のデータを持つ形で格納されている

1つのカラムの1つのレコードに複数のデータを持たせてしまうパターンで、ジェイウォークと呼ばれます。  
また、この複数の値が許されているデータ型は非スカラ値とも呼ばれます。

**アンチパターンの例:**

`Users` テーブル

| ID | Name | AffiliationID | Email |
| --- | --- | --- | --- |
| 0 | 高橋さん | 0 | " [taka@example.com](https://qiita.com/nem_Nuco/items/) "," [hashi@example.com](https://qiita.com/nem_Nuco/items/) " |
| 1 | 鈴木さん | 1 | [suzuki@example.com](https://qiita.com/nem_Nuco/items/) |
| 2 | 田中さん | 2 | " [ta@example.com](https://qiita.com/nem_Nuco/items/) "," [naka@example.com](https://qiita.com/nem_Nuco/items/) " |
| 3 | 佐藤さん | 1 | [sato@example.com](https://qiita.com/nem_Nuco/items/) |

複数のメールアドレスを持つ人が、Emailカラムにカンマ区切りで格納されてしまっています。  
Emailを取得するクエリを実行する際に、どう見ても不具合が起きそうですね。

**対策:**

- データを正規化し、それぞれのデータとして分離させる

**修正例**

`Email` テーブル

| ID | UserID | Email1 |
| --- | --- | --- |
| 0 | 0 | [taka@example.com](https://qiita.com/nem_Nuco/items/) |
| 1 | 0 | [hashi@example.com](https://qiita.com/nem_Nuco/items/) |
| 2 | 1 | [suzuki@example.com](https://qiita.com/nem_Nuco/items/) |
| 3 | 2 | [ta@example.com](https://qiita.com/nem_Nuco/items/) |
| 4 | 2 | [naka@example.com](https://qiita.com/nem_Nuco/items/) |
| 5 | 3 | [sato@example.com](https://qiita.com/nem_Nuco/items/) |

今回は `Email` テーブルとして作成しました。

## 7\. 複数の要素があるデータを複数の列として列挙する

これは列持ちテーブルと呼ばれる形で、嫌う人も多いです。  
この形だと、データをいくつ持つかによって事前にカラムの数を変更しなければいけません。

**アンチパターンの例:**

`Email` テーブル

| UserID | Email1 | Email2 |
| --- | --- | --- |
| 0 | [taka@example.com](https://qiita.com/nem_Nuco/items/) | [hashi@example.com](https://qiita.com/nem_Nuco/items/) |
| 1 | [suzuki@example.com](https://qiita.com/nem_Nuco/items/) |  |
| 2 | [ta@example.com](https://qiita.com/nem_Nuco/items/) | [naka@example.com](https://qiita.com/nem_Nuco/items/) |
| 3 | [sato@example.com](https://qiita.com/nem_Nuco/items/) |  |

**対策:**

- 行持ちテーブルの形に変更する

**修正例**

`Email` テーブル

| ID | UserID | Email1 |
| --- | --- | --- |
| 0 | 0 | [taka@example.com](https://qiita.com/nem_Nuco/items/) |
| 1 | 0 | [hashi@example.com](https://qiita.com/nem_Nuco/items/) |
| 2 | 1 | [suzuki@example.com](https://qiita.com/nem_Nuco/items/) |
| 3 | 2 | [ta@example.com](https://qiita.com/nem_Nuco/items/) |
| 4 | 2 | [naka@example.com](https://qiita.com/nem_Nuco/items/) |
| 5 | 3 | [sato@example.com](https://qiita.com/nem_Nuco/items/) |

## 8\. ルール(制約)を設けない

適切な制約がないとありえない値がデータベースに格納され、処理の際に不具合を引き起こす可能性があります。

**アンチパターンの例:**

`Users` テーブル

| ID | Name | AffiliationID | Age |
| --- | --- | --- | --- |
| 0 | 高橋さん | 0 | 29 |
| 1 | 鈴木さん | 1 | \-10 |
| 2 | 田中さん | 2 | 0 |
| 3 | 佐藤さん | 1 | 100 |

年齢にあり得ない値が入力されてしまっています。  
年齢を参照する際に、不具合が発生してしまいそうです。

**対策:**

- 適切な制約を設定する（例: `CHECK` 制約）

データ入力の段階で、格納できる値の制約をつけておきましょう。

## 9\. 過去の値を上書きして更新する

過去の値を上書きしてしまうと、過去のデータを扱う際に参照できない値が生まれてしまいます。  
さらに変更情報を履歴として残しておかないと、情報の改ざんやデータの隠蔽の温床となってしまいます。

**アンチパターンの例:**

以下の `Users` テーブルの、AffiliationID(所属)を変更する場合を考えましょう。

| ID | Name | AffiliationID |
| --- | --- | --- |
| 0 | 高橋さん | 0 |
| 1 | 鈴木さん | 1 |
| 2 | 田中さん | 2 |
| 3 | 佐藤さん | 1 |

この時にやってはいけないのが、履歴情報を残さずにAffiliationIDを上書きしてしまうことです。

**対策:**

- 履歴を管理するテーブルを作成する

**修正例**  
以下のような履歴情報テーブルを作成することが方法として挙げられます。

`History_affiliation` テーブル

| UserID | Before | After | Date |
| --- | --- | --- | --- |
| 0 | 2 | 0 | 2024-04-01 |
| 1 | 0 | 1 | 2024-04-01 |
| 2 | 1 | 0 | 2023-04-01 |
| 2 | 0 | 2 | 2024-04-01 |

もし過去の情報が必要になった場合は、履歴情報のテーブルから参照することができます。  
他にも、在籍した期間で管理するなどの方法も考えられます。

## 10\. 過剰な正規化

「正規化が重要だ！」とはりきって正規化しすぎてしまうと、かえって参照するテーブルが多くなってしまいます。  
結果的に必要なクエリが多くなり、処理の効率が悪くなります。

**アンチパターンの例:**

`Name` テーブル

| ID | Name |
| --- | --- |
| 0 | 高橋さん |
| 1 | 鈴木さん |
| 2 | 田中さん |
| 3 | 佐藤さん |

`Affiliation` テーブル

| ID | AffiliationID |
| --- | --- |
| 0 | 0 |
| 1 | 1 |
| 2 | 2 |
| 3 | 1 |

`Email` テーブル

| ID | AffiliationID |
| --- | --- |
| 0 | [takahashi@example.com](https://qiita.com/nem_Nuco/items/) |
| 1 | [suzuki@example.com](https://qiita.com/nem_Nuco/items/) |
| 2 | [tanaka@example.com](https://qiita.com/nem_Nuco/items/) |
| 3 | [sato@example.com](https://qiita.com/nem_Nuco/items/) |

名前、所属、メールアドレスは一意に管理できるにも関わらず、それぞれで正規化されています。  
これでは可読性も落ちますし、処理の際に参照するテーブルが増えてしまっています。

**対策:**

- 正規化する意味を考えて、必要な場合のみ正規化する

**修正例**

`Users` テーブル

| ID | Name | AffiliationID | Email |
| --- | --- | --- | --- |
| 0 | 高橋さん | 0 | [takahashi@example.com](https://qiita.com/nem_Nuco/items/) |
| 1 | 鈴木さん | 1 | [suzuki@example.com](https://qiita.com/nem_Nuco/items/) |
| 2 | 田中さん | 2 | [tanaka@example.com](https://qiita.com/nem_Nuco/items/) |
| 3 | 佐藤さん | 1 | [sato@example.com](https://qiita.com/nem_Nuco/items/) |

メールアドレスなどその人固有の情報は変更する際なども複雑な処理を必要としないため、正規化する必要はありません。

## 11\. 同じ役割を担うマスタテーブルが複数存在する

データ統合などの際に同じ役割を担うテーブルが複数存在したままになっていると、参照されずに使用されないデータが存在してしまう可能性が生まれます。

**アンチパターンの例:**

`Users` テーブル

| ID | Name | AffiliationID | Email |
| --- | --- | --- | --- |
| 0 | 高橋さん | 0 | [takahashi@example.com](https://qiita.com/nem_Nuco/items/) |
| 1 | 鈴木さん | 1 | [suzuki@example.com](https://qiita.com/nem_Nuco/items/) |
| 2 | 田中さん | 2 | [tanaka@example.com](https://qiita.com/nem_Nuco/items/) |
| 3 | 佐藤さん | 1 | [sato@example.com](https://qiita.com/nem_Nuco/items/) |

`User` テーブル

| ID | Name | AffiliationID | Email |
| --- | --- | --- | --- |
| 0 | 山田さん | 1 | [yamada@example.com](https://qiita.com/nem_Nuco/items/) |
| 1 | 加藤さん | 0 | [kato@example.com](https://qiita.com/nem_Nuco/items/) |
| 2 | 田中さん | 2 | [tanaka@example.com](https://qiita.com/nem_Nuco/items/) |
| 3 | 佐藤さん | 1 | [sato@example.com](https://qiita.com/nem_Nuco/items/) |

同じ役割のテーブルが2つ存在したままになっており、一部データが重複して格納されてしまっています。  
こうなってしまってからのデータの整理は難しく、場合によっては地獄の作業が必要になる可能性があります。

**対策:**

- 統合時には余計なテーブルを残さない

## 12\. ビューを残したままにする

ビューは一時的な作業効率を上げてくれることがありますが、ビューを大量に残したままにしておくとパフォーマンスの低下や管理の複雑化に繋がります。

**アンチパターンの例:**

`Users_development` ビュー

| ID | Name | AffiliationID | Email |
| --- | --- | --- | --- |
| 0 | 齊藤さん | 2 | [saito@example.com](https://qiita.com/nem_Nuco/items/) |
| 1 | 伊藤さん | 2 | [ito@example.com](https://qiita.com/nem_Nuco/items/) |
| 2 | 田中さん | 2 | [tanaka@example.com](https://qiita.com/nem_Nuco/items/) |
| 3 | 渡辺さん | 2 | [watanabe@example.com](https://qiita.com/nem_Nuco/items/) |

**対策:**

- 不要になったビューは都度削除する

## 13\. ポリモーフィック関連

ポリモーフィック関連とは、複数のテーブルへの参照を1つのカラムで行うデータ設計のことを指します。  
実際に以下のアンチパターンを見た方が理解しやすいかと思います。

**アンチパターンの例:**

`Price` テーブル

| ID | Price | EntityID | EntityType |
| --- | --- | --- | --- |
| 0 | 200 | 15 | vegetable |
| 1 | 150 | 3 | drink |
| 2 | 100 | 58 | drink |
| 3 | 300 | 14 | Bread |

この例では、野菜、飲み物、パンという複数のエンティティへの参照を1つのテーブルで行っています。  
それぞれのテーブルにおけるIDと、エンティティのタイプを指定することで一意に参照しています。  
今回のように値段の情報が共通している時などに使えますが、EntityTypeごとに複数のテーブルを参照する必要があり、クエリが複雑になってしまいます。

**対策:**

- どうしても使わざるを得ない場合も、デメリットをおさえておく

## 14\. 全てのカラムにインデックスを貼る

全てのカラムにインデックスを貼ると、更新時のパフォーマンスを低下させることがあります。  
インデックスは検索を高速化しますが、過剰なインデックスはシステムの負荷を増加させます。

**アンチパターンの例:**  
`Users` テーブル

| ID | Name | AffiliationID | Email |
| --- | --- | --- | --- |
| 0 | 高橋さん | 0 | [takahashi@example.com](https://qiita.com/nem_Nuco/items/) |
| 1 | 鈴木さん | 1 | [suzuki@example.com](https://qiita.com/nem_Nuco/items/) |
| 2 | 田中さん | 2 | [tanaka@example.com](https://qiita.com/nem_Nuco/items/) |
| 3 | 佐藤さん | 1 | [sato@example.com](https://qiita.com/nem_Nuco/items/) |

このようなテーブルに対して、以下のようなクエリで全てのカラムにインデックスを作成することを指します。

users.sql

```sql
-- Nameカラムに対するインデックスの作成
CREATE INDEX idx_name ON Users(Name);

-- AffiliationIDカラムに対するインデックスの作成
CREATE INDEX idx_affiliation_id ON Users(AffiliationID);

-- Emailカラムに対するインデックスの作成
CREATE INDEX idx_email ON Users(Email);
```

**対策:**

- 必要な場合にのみインデックスを作成する

## 15\. 想定していないnull

想定していないnullは更新時に不具合を起こします。

**アンチパターンの例:**

`Users` テーブル

| ID | Name | AffiliationID | Email | Phone |
| --- | --- | --- | --- | --- |
| 0 | 高橋さん | 0 | [takahashi@example.com](https://qiita.com/nem_Nuco/items/) | 123-456-7890 |
| 1 |  | 1 | [suzuki@example.com](https://qiita.com/nem_Nuco/items/) | 234-567-8901 |
| 2 | 田中さん |  | [tanaka@example.com](https://qiita.com/nem_Nuco/items/) | 345-678-9012 |
| 3 |  | 1 | [sato@example.com](https://qiita.com/nem_Nuco/items/) | 456-789-0123 |

この例では、必須なはずの名前と所属がnullになってしまっています。

**対策:**

- not null制約を使う
- デフォルト値を設定する

**修正例**

`Users` テーブル

| ID | Name | AffiliationID | Email | Phone |
| --- | --- | --- | --- | --- |
| 0 | 高橋さん | 0 | [takahashi@example.com](https://qiita.com/nem_Nuco/items/) | 123-456-7890 |
| 1 | No Name | 1 | [suzuki@example.com](https://qiita.com/nem_Nuco/items/) | 234-567-8901 |
| 2 | 田中さん | No Information | [tanaka@example.com](https://qiita.com/nem_Nuco/items/) | 345-678-9012 |
| 3 | No Name | 1 | [sato@example.com](https://qiita.com/nem_Nuco/items/) | 456-789-0123 |

この例ではデフォルト値を設定するようにしています。  
ただし、エラーが出ないことによって問題に気づきづらくなるというデメリットもあるため、場合によって対応を変えるようにしましょう。

## 16\. 安易なテーブル分割

人為的なミスも発生しやすい。

**アンチパターンの例:**

`Users1` テーブル

| ID | OriginalID | Name | AffiliationID | Email |
| --- | --- | --- | --- | --- |
| 0 | 0 | 高橋さん | 0 | [takahashi@example.com](https://qiita.com/nem_Nuco/items/) |
| 1 | 2 | 田中さん | 2 | [tanaka@example.com](https://qiita.com/nem_Nuco/items/) |
| 2 | 4 | 齊藤さん | 2 | [saito@example.com](https://qiita.com/nem_Nuco/items/) |

`Users2` テーブル

| ID | OriginalID | Name | AffiliationID | Email |
| --- | --- | --- | --- | --- |
| 0 | 1 | 鈴木さん | 1 | [suzuki@example.com](https://qiita.com/nem_Nuco/items/) |
| 1 | 3 | 佐藤さん | 1 | [sato@example.com](https://qiita.com/nem_Nuco/items/) |
| 2 | 5 | 伊藤さん | 2 | [ito@example.com](https://qiita.com/nem_Nuco/items/) |

このように、1つのテーブルを水平に分割したようなテーブルのことを指します。  
データの数が大きくなり、このような形をとらざるを得ない時があると思います。  
管理を徹底しないと、片方しか使われていないなどの人為的なミスが発生する可能性があります。

**対策:**

- データを統合する
- パーティショニングを活用してビューを作成する
- テーブルを扱うクエリの分に注釈をつけて人為的ミスを防ぐ

**修正例**

`Users` テーブル

| ID | Name | AffiliationID | Email |
| --- | --- | --- | --- |
| 0 | 高橋さん | 0 | [takahashi@example.com](https://qiita.com/nem_Nuco/items/) |
| 1 | 鈴木さん | 1 | [suzuki@example.com](https://qiita.com/nem_Nuco/items/) |
| 2 | 田中さん | 2 | [tanaka@example.com](https://qiita.com/nem_Nuco/items/) |
| 3 | 佐藤さん | 1 | [sato@example.com](https://qiita.com/nem_Nuco/items/) |
| 4 | 齊藤さん | 2 | [saito@example.com](https://qiita.com/nem_Nuco/items/) |
| 5 | 伊藤さん | 2 | [ito@example.com](https://qiita.com/nem_Nuco/items/) |

1つのテーブルでも問題なく処理が行えそうなときはデータを統合しましょう。  
ビューを作成例は以下の通りです。

```sql
CREATE VIEW UnifiedUsers AS
SELECT * FROM Users1
UNION ALL
SELECT * FROM Users2;
```

## まとめ

いかがだったでしょうか。  
今回はアンチパターンを基にしてデータベースの設計について整理してみました。

使いやすい形でデータベースが設計できるか、気遣いの力が試されます。

使いづらいデータベースを設計してしまい、過去の自分を恨まないように日頃から気をつけられるようにしましょう。

弊社Nucoでは、他にも様々なお役立ち記事を公開しています。よかったら、Organizationのページも覗いてみてください。  
また、Nucoでは一緒に働く仲間も募集しています！興味をお持ちいただける方は、 [こちら](https://www.recruit.nuco.co.jp/?qiita_item_id=6a00e1764628b1b3fefb) まで。

[9](https://qiita.com/nem_Nuco/items/#comments)

[451](https://qiita.com/nem_Nuco/items/6a00e1764628b1b3fefb/likers)

494