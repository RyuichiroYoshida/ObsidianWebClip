---
title: 簡単に自動化が実現！ 無料で始める「Google Apps Script」で作業効率を上げる方法【Google活用基本のき】
source: https://serai.jp/living/1197929
author:
  - "[[サライ.jp｜小学館の雑誌『サライ』公式サイト]]"
published: 2025-01-15
created: 2025-05-06
description: 毎日のルーチンワークが負担になっていませんか？ 「もっと効率よくできないか……」と感じることは、誰しもが一度はあるでしょう。そんな悩みを解決してくれるのがGoogle Apps Scriptです。プログラミングの経験がな…
tags:
  - Tech
  - Tools
  - "#GAS"
read: false
---
生活

- [生活](https://serai.jp/living)
- [Google活用基本のき](https://serai.jp/living/google)

2025/1/15

- [![Facebook](https://serai.jp/wp-content/themes/serai2020/assets/images/icon_fb.png)](http://www.facebook.com/share.php?u=https://serai.jp/living/1197929)
- [![Twitter](https://serai.jp/wp-content/themes/serai2020/assets/images/icon_twitter.png)](http://twitter.com/share?url=https://serai.jp/living/1197929&text=%E7%B0%A1%E5%8D%98%E3%81%AB%E8%87%AA%E5%8B%95%E5%8C%96%E3%81%8C%E5%AE%9F%E7%8F%BE%EF%BC%81%20%E7%84%A1%E6%96%99%E3%81%A7%E5%A7%8B%E3%82%81%E3%82%8B%E3%80%8CGoogle%20Apps%20Script%E3%80%8D%E3%81%A7%E4%BD%9C%E6%A5%AD%E5%8A%B9%E7%8E%87%E3%82%92%E4%B8%8A%E3%81%92%E3%82%8B%E6%96%B9%E6%B3%95%E3%80%90Google%E6%B4%BB%E7%94%A8%E5%9F%BA%E6%9C%AC%E3%81%AE%E3%81%8D%E3%80%91)
- [![LINE](https://serai.jp/wp-content/themes/serai2020/assets/images/icon_line.png)](http://line.me/R/msg/text/?%E7%B0%A1%E5%8D%98%E3%81%AB%E8%87%AA%E5%8B%95%E5%8C%96%E3%81%8C%E5%AE%9F%E7%8F%BE%EF%BC%81%20%E7%84%A1%E6%96%99%E3%81%A7%E5%A7%8B%E3%82%81%E3%82%8B%E3%80%8CGoogle%20Apps%20Script%E3%80%8D%E3%81%A7%E4%BD%9C%E6%A5%AD%E5%8A%B9%E7%8E%87%E3%82%92%E4%B8%8A%E3%81%92%E3%82%8B%E6%96%B9%E6%B3%95%E3%80%90Google%E6%B4%BB%E7%94%A8%E5%9F%BA%E6%9C%AC%E3%81%AE%E3%81%8D%E3%80%91%0D%0Ahttps://serai.jp/living/1197929)
- [![はてなブックマーク](https://serai.jp/wp-content/themes/serai2020/assets/images/icon_hatena.png)](http://b.hatena.ne.jp/add?mode=confirm&url=https://serai.jp/living/1197929?text_=%E7%B0%A1%E5%8D%98%E3%81%AB%E8%87%AA%E5%8B%95%E5%8C%96%E3%81%8C%E5%AE%9F%E7%8F%BE%EF%BC%81%20%E7%84%A1%E6%96%99%E3%81%A7%E5%A7%8B%E3%82%81%E3%82%8B%E3%80%8CGoogle%20Apps%20Script%E3%80%8D%E3%81%A7%E4%BD%9C%E6%A5%AD%E5%8A%B9%E7%8E%87%E3%82%92%E4%B8%8A%E3%81%92%E3%82%8B%E6%96%B9%E6%B3%95%E3%80%90Google%E6%B4%BB%E7%94%A8%E5%9F%BA%E6%9C%AC%E3%81%AE%E3%81%8D%E3%80%91)

![](https://serai.jp/wp-content/uploads/2024/08/image1-8.jpg)

毎日のルーチンワークが負担になっていませんか？　「もっと効率よくできないか……」と感じることは、誰しもが一度はあるでしょう。そんな悩みを解決してくれるのがGoogle Apps Scriptです。プログラミングの経験がなくても簡単に始められ、Google スプレッドシートやGmailなどの日常的に使うツールと連携して、手間のかかる作業を自動化できます。

この機会に、無料で使える便利なツールをマスターし、毎日の業務や生活をより快適にしてみませんか？

**目次**  
[Google Apps Scriptとは？](https://serai.jp/living/#01)  
[Google Apps Scriptの活用例](https://serai.jp/living/#02)  
[Google Apps Scriptの始め方](https://serai.jp/living/#03)  
[Google Apps Scriptの活用アイデア](https://serai.jp/living/#04)  
[最後に](https://serai.jp/living/#05)  

## Google Apps Scriptとは？

Google Apps Scriptは、Googleが提供するプログラミングツールで、Google スプレッドシート、Gmail、Google カレンダーなどのサービスと連携して、手作業を自動化できる便利な機能です。例えば、スプレッドシートに入力したデータを自動で集計してメールで送信するなど、日常業務や家庭の作業を効率化することが可能です。

### Google Apps Scriptのメリット

Google Apps Scriptの最大の魅力は、無料で簡単に利用できること。複雑なプログラムの知識がなくても、少しの操作で自動化が実現します。さらに、Googleアカウントさえあれば、誰でもすぐに利用できるため、費用や時間の節約につながります。

![](https://serai.jp/wp-content/uploads/2024/12/image1-1-640x327.jpg)

Google Apps Scriptの画面

## Google Apps Scriptの活用例

### 家庭やビジネスのシーンでの活用

例えば、次のような場面で活用することで、日常の作業効率が劇的に向上します。

#### １．家計簿の自動化

スプレッドシートに毎月の収支を入力し、Google Apps Scriptを活用して、月次の自動集計やレポートを作成。

#### ２．地域活動のイベント管理

自治会やサークル活動で、メンバーへの自動メール送信や出欠確認の自動集計を実現。

#### ３．職場での予定調整

Google カレンダーと連携して会議のリマインダーや予定の通知を自動化。

## Google Apps Scriptの始め方

ここからは、実際にGoogle Apps Scriptの使い方をステップごとに解説していきます。この記事では、WindowsとMacどちらの環境でも対応できる操作を紹介します。

### ステップ１：Google スプレッドシートを開く

まず、Google スプレッドシートを開きます。Googleアカウントがあれば、無料で簡単にアクセスできます。

![](https://serai.jp/wp-content/uploads/2024/12/image2-1-640x279.jpg)

### ステップ２：スクリプトエディタを開く

スプレッドシートの上部メニューから「拡張機能」をクリックし、「Apps Script」を選択します。これで、プログラムを書くための画面（スクリプトエディタ）が開きます。

![](https://serai.jp/wp-content/uploads/2024/12/image3-1-640x401.jpg)

### ステップ３：最初のプログラムを書く

スクリプトエディタが開いたら、次のコードを入力してみましょう。このコードは、スプレッドシート上でポップアップメッセージを表示するためのものです。

```
＜javascript＞

function showAlert() {
  SpreadsheetApp.getUi().alert("こんにちは、サライ.jpの読者の皆様！");
}
```

上記のコードを入力することで、簡単なメッセージをポップアップとして表示できます。初心者向けのプログラムとして、シンプルですが効果的です。

![](https://serai.jp/wp-content/uploads/2024/12/image4-1-640x235.jpg)

### ステップ４：プログラムを保存して実行

入力が完了したら、画面右上の保存アイコンを押してプログラムを保存します。その後、「実行」ボタンをクリックすると、先ほどのスクリプトが動作し、ポップアップが表示されます。

![](https://serai.jp/wp-content/uploads/2024/12/image5.jpg)

保存アイコンを押すことで、プログラムが保存されます

↓

![](https://serai.jp/wp-content/uploads/2024/12/image6.jpg)

「実行」を押します

↓

![](https://serai.jp/wp-content/uploads/2024/12/image7-1-640x414.jpg)

最初に開いたスプレッドシートに戻ると、ポップアップが表示されます

## Google Apps Scriptの活用アイデア

Google Apps Scriptを活用すると、日常の手間を省き、効率的な作業が可能になります。ここでは、GmailやGoogle ドライブを使った実用的な活用例をご紹介します。少しの設定で、日々の作業が格段に楽になりますよ。

### メールの自動送信

例えば、業務連絡や顧客対応のメールを手動で送信するのは面倒と感じるときはありませんか？　そんなとき、Google Apps Scriptを使えば、特定の条件に応じてGmailからメールを自動送信することができます。例えば、スプレッドシートに記録されたデータを元に、定期的なリマインドメールを自動で送るといったことが可能です。

具体的な活用シーン：

・請求書や支払い確認のリマインドメールを自動送信して、経理業務を効率化。  
・顧客へのフォローメールや誕生日メッセージを自動的に送信し、顧客満足度を向上。  
・チームメンバーへのタスク通知を自動で送ることで、プロジェクト管理の手間を軽減。

![](https://serai.jp/wp-content/uploads/2024/12/image8-1-640x238.jpg)

プログラムの全体像

![](https://serai.jp/wp-content/uploads/2024/12/image9-1-640x193.jpg)

プログラムによってメールが送信されました

サンプルコード（メールの自動送信の基本的な例）：

```
＜javascript＞

function sendReminderEmail() {
  var emailAddress = "example@example.com"; // 送信先のメールアドレス
  var subject = "リマインド: 重要なタスクがあります";
  var message = "こんにちは、期日が近づいているタスクがあります。ご確認ください。";
  MailApp.sendEmail(emailAddress, subject, message);
}
```

上記のコードをスクリプトエディタにコピーして、保存後に実行すると、指定したアドレスにメールが自動で送信されます。「送信先のメールアドレス」は、実際の送信先のアドレスに書き換えてください。

また、コードの実行時に権限確認を要求されることがあります。表示される手順に従って、許可してください。途中に、警告のような画面が表示されますが、「詳細」をクリックし、コードを保存しているプロジェクト名をクリックしてください。確認手順が進行します。

![](https://serai.jp/wp-content/uploads/2024/12/a30b0918f7d7ea012c0ac7b6e7210a72-640x359.jpg)

警告のような画面ですが「詳細」を押します

↓

![](https://serai.jp/wp-content/uploads/2024/12/d4d35c4990fd623192af59c1f1ca2a79-640x472.jpg)

コードを保存しているプロジェクト名をクリックしてください。この記事では、「メール送信テストプロジェクト」という名称にしました

### データの自動バックアップ

Google ドライブとGoogle Apps Scriptを活用することで、重要なデータのバックアップを自動化できます。これにより、手動でバックアップを取る手間が省け、万が一のデータ紛失も防ぐことができます。例えば、定期的に更新されるスプレッドシートの内容を、自動でGoogle ドライブ内の特定のフォルダにコピーするスクリプトを設定できます。

具体的な活用シーン：

・毎週の売上データや月次の会計報告書を自動的にバックアップして、万が一に準備。  
・重要なドキュメントや契約書を定期的にアーカイブし、クラウド上で安全に保管。  
・プロジェクト資料や顧客リストを定期的にバックアップして、チーム全体で共有。

![](https://serai.jp/wp-content/uploads/2024/12/image10-640x160.jpg)

バックアップするプログラムの全体

![](https://serai.jp/wp-content/uploads/2024/12/image11-640x296.jpg)

プログラムによってバックアップされたファイル

サンプルコード（バックアップの基本的な例）：

```
＜javascript＞

function backupSpreadsheet() {
  var currentSpreadsheet = SpreadsheetApp.getActiveSpreadsheet(); // 現在のスプレッドシート
  var currentSpreadsheetFile = DriveApp.getFileById(currentSpreadsheet.getId()); // 現在のスプレッドシートのファイルを取得
  var folder = DriveApp.getFolderById("フォルダIDを入力"); // 保存先のフォルダID

  currentSpreadsheetFile.makeCopy("バックアップ_" + new Date().toISOString(), folder);
}
```

上記のコードをスクリプトエディタに入力し、フォルダIDを設定することで、現在のスプレッドシートの内容が自動的にバックアップされます。

フォルダIDは、例えば、フォルダURLが「https://drive.google.com/drive/folders/1a2b3c4d5e6f7g8h」の場合、一番後ろの「1a2b3c4d5e6f7g8h」がフォルダIDです。

また、前述のメール送信同様、コードの実行時に権限確認を要求されることがあります。  
  
これらの活用方法をマスターすれば、日常業務の効率化に大きく役立ちます。難しい設定は不要で、少しの工夫で作業の自動化が実現できます。ぜひ試してみてください！

## 最後に

Google Apps Scriptは、手軽に始められるだけでなく、使い方次第で日常業務の効率化に大きく貢献するツールです。最初はシンプルなスクリプトからスタートし、徐々に活用範囲を広げていけば、より多くの作業を自動化することが可能です。今回紹介した内容を参考に、まずは一歩を踏み出してみてください。

生活や仕事の質を向上させる新しい可能性が広がるはずです。読者の皆さんが、効率化と快適さを手に入れる一助となれば幸いです。

●監修／三鷹 れい（みたか れい｜京都メディアライン・ [https://kyotomedialine.com](https://kyotomedialine.com/) [FB](https://www.facebook.com/kyotomedialine/) ）  
プログラマー。中小企業のDX化など、デジタル分野での支援をしている。主な開発分野はバックエンド・フロントエンド・アプリ。その他、歴史などの人文系にも興味を持つ。

![](https://serai.jp/wp-content/uploads/2024/12/bf91db4ff03f96b4488ada3c2b59c58e.jpeg)

この記事が気に入ったら  
いいね！しよう

- [![FaceBook](https://serai.jp/wp-content/themes/serai2020/assets/images/icon_fb.png) シェア](http://www.facebook.com/share.php?u=https://serai.jp/living/1197929)
- [![Twitter](https://serai.jp/wp-content/themes/serai2020/assets/images/icon_twitter.png) ツイート](http://twitter.com/share?url=https://serai.jp/living/1197929&text=%E7%B0%A1%E5%8D%98%E3%81%AB%E8%87%AA%E5%8B%95%E5%8C%96%E3%81%8C%E5%AE%9F%E7%8F%BE%EF%BC%81%20%E7%84%A1%E6%96%99%E3%81%A7%E5%A7%8B%E3%82%81%E3%82%8B%E3%80%8CGoogle%20Apps%20Script%E3%80%8D%E3%81%A7%E4%BD%9C%E6%A5%AD%E5%8A%B9%E7%8E%87%E3%82%92%E4%B8%8A%E3%81%92%E3%82%8B%E6%96%B9%E6%B3%95%E3%80%90Google%E6%B4%BB%E7%94%A8%E5%9F%BA%E6%9C%AC%E3%81%AE%E3%81%8D%E3%80%91)
- [![LINE](https://serai.jp/wp-content/themes/serai2020/assets/images/icon_line.png) シェア](http://line.me/R/msg/text/?%E7%B0%A1%E5%8D%98%E3%81%AB%E8%87%AA%E5%8B%95%E5%8C%96%E3%81%8C%E5%AE%9F%E7%8F%BE%EF%BC%81%20%E7%84%A1%E6%96%99%E3%81%A7%E5%A7%8B%E3%82%81%E3%82%8B%E3%80%8CGoogle%20Apps%20Script%E3%80%8D%E3%81%A7%E4%BD%9C%E6%A5%AD%E5%8A%B9%E7%8E%87%E3%82%92%E4%B8%8A%E3%81%92%E3%82%8B%E6%96%B9%E6%B3%95%E3%80%90Google%E6%B4%BB%E7%94%A8%E5%9F%BA%E6%9C%AC%E3%81%AE%E3%81%8D%E3%80%91%0D%0Ahttps://serai.jp/living/1197929)
- [![はてなブックマーク](https://serai.jp/wp-content/themes/serai2020/assets/images/icon_hatena.png) ブックマーク](http://b.hatena.ne.jp/add?mode=confirm&url=https://serai.jp/living/1197929?text_=%E7%B0%A1%E5%8D%98%E3%81%AB%E8%87%AA%E5%8B%95%E5%8C%96%E3%81%8C%E5%AE%9F%E7%8F%BE%EF%BC%81%20%E7%84%A1%E6%96%99%E3%81%A7%E5%A7%8B%E3%82%81%E3%82%8B%E3%80%8CGoogle%20Apps%20Script%E3%80%8D%E3%81%A7%E4%BD%9C%E6%A5%AD%E5%8A%B9%E7%8E%87%E3%82%92%E4%B8%8A%E3%81%92%E3%82%8B%E6%96%B9%E6%B3%95%E3%80%90Google%E6%B4%BB%E7%94%A8%E5%9F%BA%E6%9C%AC%E3%81%AE%E3%81%8D%E3%80%91)

## ランキング

NEW

[![](https://serai.jp/wp-content/uploads/2025/04/31710465_s-1.jpg)](https://serai.jp/living/1227405)

[生活](https://serai.jp/living) [【定年後の歩き方】定年離婚後、孤独を脱した63歳男性が語…](https://serai.jp/living/1227405)

2025/05/04

No.

NEW

[![](https://serai.jp/wp-content/uploads/2023/05/0003d5d087f89835dfba4bb036754b7f.jpg)](https://serai.jp/living/1228276)

[生活](https://serai.jp/living) [ニャンコ先生の開運星占い（5/5～5/11）射手座、山羊…](https://serai.jp/living/1228276)

2025/05/05

No.

NEW

[![](https://serai.jp/wp-content/uploads/2023/05/0003d5d087f89835dfba4bb036754b7f.jpg)](https://serai.jp/living/1228274)

[生活](https://serai.jp/living) [ニャンコ先生の開運星占い（5/5～5/11）牡羊座、牡牛…](https://serai.jp/living/1228274)

2025/05/05

No.

NEW

[![](https://serai.jp/wp-content/uploads/2023/05/0003d5d087f89835dfba4bb036754b7f.jpg)](https://serai.jp/living/1228275)

[生活](https://serai.jp/living) [ニャンコ先生の開運星占い（5/5～5/11）獅子座、乙女…](https://serai.jp/living/1228275)

2025/05/05

No.

[![](https://serai.jp/wp-content/uploads/2023/05/0003d5d087f89835dfba4bb036754b7f.jpg)](https://serai.jp/living/1227378)

[生活](https://serai.jp/living) [ニャンコ先生の開運星占い（4/28～5/4）射手座、山羊…](https://serai.jp/living/1227378)

2025/04/28

No.

## 新着記事

NEW

[![](https://serai.jp/wp-content/uploads/2020/09/A75010003_0b.jpg)](https://serai.jp/shop/1198715)

[植物なめしの牛革を使用した、職人仕上げの…](https://serai.jp/shop/1198715)

2025/05/05

NEW

[![](https://serai.jp/wp-content/uploads/2023/04/4167177_s.jpg)](https://serai.jp/living/1123465)

[受け取り方で総額が変わる！「損をしない」…](https://serai.jp/living/1123465)

2025/05/05

NEW

[![](https://serai.jp/wp-content/uploads/2025/04/DSCF5774.jpg)](https://serai.jp/gourmet/1226929)

[コンビニの「サラダチキン」で作る、火を使…](https://serai.jp/gourmet/1226929)

2025/05/05

NEW

[![](https://serai.jp/wp-content/uploads/2025/03/31902758_s.jpg)](https://serai.jp/health/1222244)

[体が変わる！ ４つの「やせ習慣」【60歳…](https://serai.jp/health/1222244)

2025/05/05

NEW

[![](https://serai.jp/wp-content/uploads/2022/01/1-2.jpg)](https://serai.jp/living/1056789)

[【友人という病】「50年の友情も冷める不…](https://serai.jp/living/1056789)

2025/05/05

[記事一覧へ](https://serai.jp/archives)

## ピックアップ

 [![](https://serai.jp/wp-content/uploads/2025/02/3d46a548c170ba3731a532f1d4388608.jpg) PR](https://serai.jp/premium/1217947?utm_source=serai&utm_medium=referral&utm_campaign=_pickup)

[伝統と革新が融合する匠の美｜江戸切子職人…](https://serai.jp/premium/1217947?utm_source=serai&utm_medium=referral&utm_campaign=_pickup)

2025/03/10

 [![](https://serai.jp/wp-content/uploads/2025/02/4c85c234076325c638e2c305400f29d6-2.jpg) PR](https://serai.jp/tour/1218757?utm_source=serai&utm_medium=referral&utm_campaign=_pickup)

[山手線開業100周年に“まちびらき” 高…](https://serai.jp/tour/1218757?utm_source=serai&utm_medium=referral&utm_campaign=_pickup)

2025/03/07

 [![](https://serai.jp/wp-content/uploads/2025/02/001.jpg) PR](https://serai.jp/living/1217966?utm_source=serai&utm_medium=referral&utm_campaign=_pickup)

[今の使いやすさをそのままに料金を下げる！…](https://serai.jp/living/1217966?utm_source=serai&utm_medium=referral&utm_campaign=_pickup)

2025/03/03

 [![](https://serai.jp/wp-content/uploads/2025/01/63F0014.jpg) PR](https://serai.jp/tour/1214137?utm_source=serai&utm_medium=referral&utm_campaign=_pickup)

[【ふらり大人のひとり旅～料理家・山脇りこ…](https://serai.jp/tour/1214137?utm_source=serai&utm_medium=referral&utm_campaign=_pickup)

2025/01/20

 [![](https://serai.jp/wp-content/uploads/2024/12/1723290622210.jpg) PR](https://serai.jp/hobby/1211646?utm_source=serai&utm_medium=referral&utm_campaign=_pickup)

[通信教育と現地授業で学び直し｜歴史・文化…](https://serai.jp/hobby/1211646?utm_source=serai&utm_medium=referral&utm_campaign=_pickup)

2025/01/15

[記事一覧へ](https://serai.jp/pr)

[![](https://serai.jp/wp-content/uploads/2024/04/71ySFYyklBL._SL1441_.jpg)](https://www.shogakukan.co.jp/books/09682453)

[![](https://serai.jp/wp-content/uploads/2024/01/469dfc234814a48d670280ddfb7d1653.png)](https://www.shogakukan.co.jp/books/09386711)

[![](https://serai.jp/wp-content/uploads/2022/11/U5A_44311m-scaled.jpg)](https://serai.jp/kajin)

[![](https://serai.jp/wp-content/uploads/2023/02/36c82d99acdaa1959333b6364212caa4-500x309.jpg)](https://www.shogakukan.co.jp/books/09389100)

[![](https://serai.jp/wp-content/uploads/2023/02/73605962b14c734daa62fed9f13ae79c-500x171.jpg)](https://www.ringbell.co.jp/serai/)

[![](https://serai.jp/wp-content/uploads/2020/12/2a39a78613973b5ab9d0ad44d83cdc7f.jpg)](https://www.amazon.co.jp/%E9%80%86%E8%BB%A2%E3%81%AE%E6%88%A6%E5%9B%BD%E5%8F%B2-%E3%80%8C%E5%A4%A9%E6%89%8D%E3%80%8D%E3%81%A7%E3%81%AF%E3%81%AA%E3%81%8B%E3%81%A3%E3%81%9F%E4%BF%A1%E9%95%B7%E3%80%81%E3%80%8C%E5%8F%9B%E8%87%A3%E3%80%8D%E3%81%A7%E3%81%AF%E3%81%AA%E3%81%8B%E3%81%A3%E3%81%9F%E5%85%89%E7%A7%80-%E7%A0%82%E5%8E%9F-%E6%B5%A9%E5%A4%AA%E6%9C%97/dp/409386604X/?tag=vc-1-869-22&linkCode=ure)