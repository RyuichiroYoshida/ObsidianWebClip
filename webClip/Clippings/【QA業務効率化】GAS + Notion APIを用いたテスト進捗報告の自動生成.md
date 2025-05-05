---
title: 【QA業務効率化】GAS + Notion APIを用いたテスト進捗報告の自動生成
source: https://techblog.enechain.com/entry/automated-daily-test-progress-reporting
author:
  - "[[enechain Tech Blog]]"
published: 2024-11-06
created: 2025-05-06
description: enechainのQAチームにおける業務効率化の取り組みの一つとして、GASとNotion APIを用いた日々のテスト進捗報告の自動生成の取り組みについて紹介します。
tags:
  - GAS
  - 1
  - Tools
  - Notion
  - QA
read: false
---
![ogp](https://cdn-ak.f.st-hatena.com/images/fotolife/e/enechain/20241106/20241106103051.png)

## はじめに

こんにちは！enechainでQAチームのマネージャーを務める杉田 ([@ *sug1san*](https://x.com/_sug1san_)) です。

QAチームでは先日、初の試みとして「QAオフサイト」と題したイベントを社外の会場を借りて実施し、日頃眼の前の業務に忙殺されて後回しになりがちな品質改善、QA業務改善に、メンバー各人が自身でテーマを決め、丸一日かけて取り組みました。

今回は、私がそこで取り組んだGAS (Google Apps Script) とNotion APIを用いたテスト進捗報告の自動生成の取り組みについてご紹介します。

## enechainでのテスト管理

本題に入る前に、まずは前提となるenechainでのテスト管理について説明します。

enechainのQAチームでは、複数あるプロダクトをそれぞれ専任のQAエンジニアが担当しており、その業務の1つとして、リリース前のシステムテストを主導しています。QA業務の内容については、QAエンジニアのtaise-の記事もご参照ください。

[techblog.enechain.com](https://techblog.enechain.com/entry/enechain-qa-attempt)

### テストケースとテスト進捗の管理

テストケースの管理にはGoogleスプレッドシートを利用しています。TestRail等、専用ツールの導入も定期的に検討してはいますが、現時点では扱い慣れており柔軟性も高いGoogleスプレッドシートで特に課題を感じていません。

各プロダクトのシステム開発プロジェクト毎に、Googleスプレッドシートで作成されたテストケーステンプレートのファイルを複製して利用します。このファイルには機能テスト、シナリオテスト、リグレッションテスト等、テストの種類毎に異なるテンプレートを持つシートが複数用意されており、これらのシートにテストケースを追加した上で、それを実行していきます。

そして、実行中のテストの進捗状況やその結果は、集計用のシートにすべて集約され、集計される作りとなっています。以下は集計シートの抜粋ですが、表の「シート名」列にシート名を入れると、自動的にそのシート内のテストケースが集計されます。

![集計シート](https://cdn-ak.f.st-hatena.com/images/fotolife/e/enechain/20241106/20241106103056.png)

### バグチケットの管理

テストで検出されたバグチケットの管理にはNotionを利用しています。enechainでは2022年にプロジェクト管理やナレッジマネジメントのツールをJIRA / ConfluenceからNotionに完全移行し、全社で活用しています。そのため、バグチケットの管理についてもNotionのデータベースを用いています。

enechainでは、プロジェクト毎にPdMが社内でSpec Documentと呼ばれるドキュメントを作成します。Spec DocumentもNotionのデータベースを用いて管理しており、バグチケットデータベースとリレーションを張ることで、バグチケットとプロジェクトの紐づけを管理しています。さらにこのSpec Documentデータベースはプロダクトを管理するデータベースともリレーションが張られているため、バグチケットをプロジェクト単位やプロダクト単位でフィルタし、集計・分析が行えるようになっています。

```mermaid
プロダクト DBtitleproduct_nameプロダクト名relationspec_docSpec Documentotherotheretc..Spec Document DBtitleproject_name施策名relationproductプロダクトrelationbug_ticketバグチケットstatusstatusステータスotherotheretc..バグチケット DBtitlesummary概要relationspec_docSpec Documentrollupproductプロダクトstatusstatusステータスselectseverity重大度created_atcreated_at起票日otherotheretc..1つのプロダクトには複数のSpec Documentが紐づく1つのSpec Documentには複数のバグチケットが紐づく
```

システムテストの実施期間中は、これらのデータをもとに毎日業務終了のタイミングでQAエンジニアからその日のテスト進捗状況やバグチケットの情報をSlackにポストしています。

## 日々のテスト進捗報告の自動生成

ここからが本題です。

私は定型的な業務を極力自動化、省力化することで、コストを抑え、スピードを上げ、そしてミスを減らしたいと考えています。

日々のテスト進捗報告についても、ある程度報告する内容が定まっており、かつデータと客観的事実に基づいて構成されているため、定型業務の1つとして自動化・省力化すべき対象と考えました。

さらに、必要なデータはすべてGoogleスプレッドシートおよびNotion上にあり、GASとNotion APIを用いれば簡単に活用できる状態に既になっています。そのため、これに取り組まない手はありませんでした。

それでは、テスト進捗報告の生成方法について説明していきます。ただ、細かな実装内容まで踏み込んでしまうと長くなってしまうため、要点に絞ってお話します。

### テスト進捗情報の取得

テストの進捗情報はすでにGoogleスプレッドシートの集計シートに集約、集計されています。そのため、今回必要なのは集計シートの内容をGASで取得しパースすることだけでした。

テスト全体、もしくは特定のテストケースを指定して進捗情報をシートから抽出し、テスト進捗オブジェクトを作成します。

```jsx
// テスト進捗データを取得する
function createTestProgressObject(sheetData, sheetName = '') {
  // 検索するシート名を決定（空文字の場合は "全体" を使用）
  const searchName = sheetName === '' ? '全体' : sheetName;

  // B列から該当する行を探す
  const targetRow = sheetData.find(row => row[1] === searchName);

  // 必要なデータを抽出してオブジェクトを作成
  const result = {
    "テストケース名": targetRow[1],  // B列
    "ケース数": targetRow[4],        // E列
    "未実施": targetRow[5],          // F列
    "OK": targetRow[6],              // G列
    "NG": targetRow[7],              // H列
    "FIXED": targetRow[8],           // I列
    "PEND": targetRow[9],            // J列
    "N/A": targetRow[10]             // K列
  };

  return result;
}
```

### バグチケット情報の取得

Notionのデータベースで管理されているバグチケット情報は、GASからNotion APIの `POST https://api.notion.com/v1/databases/{database_id}/query` を用いて取得します。

取得したいのは、実施中のシステムテストで検出されたバグチケットの情報のみです。そのため、集計シートに記載されたSpec DocumentのURLを参照して、そのSpec Documentとリレーションが張られているバグチケットのみをフィルタして取得します。

またこのとき、1回のシステムテストで複数のSpec Documentに紐づく施策のテストを行うこともあるため、OR条件でいずれかのSpec Documentとリレーションが張られているバグチケットをすべて取得しています。

```jsx
function getBugTicket(specs) {
  const properties = PropertiesService.getScriptProperties();
  const token = properties.getProperty("INTEGRATION_TOKEN");
  const databaseId = properties.getProperty("BUG_DB_ID");
  const url = URL_DATABASE_API + "/" + databaseId + "/query";

  // filterオブジェクトを作成
  const specFilter = {
    or: specs.map(spec => ({
      property: "Spec Document",
      relation: {
        contains: spec
      }
    }))
  };

  const sorts = [
    {
      property: "重大度",
      direction: "ascending"
    },
    {
      timestamp: "created_time",
      direction: "ascending"
    }
  ];

  let allResults = [];
  let hasMore = true;
  let nextCursor = undefined;

  while (hasMore) {
    let payload = {
      filter: specFilter,
      sorts: sorts
    };

    if (nextCursor) {
      payload.start_cursor = nextCursor;
    }

    const options = {
      "method": "POST",
      "headers": {
        "Content-type": "application/json",
        "Authorization": "Bearer " + token,
        "Notion-Version": API_VERSION
      },
      "payload": JSON.stringify(payload)
    }

    try {
      const response = UrlFetchApp.fetch(url, options);
      const result = JSON.parse(response.getContentText());

      allResults = allResults.concat(result.results);
      hasMore = result.has_more;
      nextCursor = result.next_cursor;

      // APIレート制限を考慮して少し待機
      if (hasMore) {
        Utilities.sleep(300);
      }
    } catch (error) {
      console.error('Error fetching bug tickets:', error);
      hasMore = false;
    }
  }

  return allResults;
}
```

こうして取得したバグチケット情報ですが、報告に不要なものも含め多くのプロパティを持つため、報告に必要な要素のみを抽出し、報告用バグチケットオブジェクトを作成します。

```jsx
// バグチケットのデータから必要な要素のみを抽出してオブジェクトに詰めて返却する
function extractBugTicketData(bugTickets) {
  return bugTickets.map(ticket => {
    const properties = ticket.properties;
    const pageUrl = \`https://www.notion.so/${ticket.id.replace(/-/g, '')}\`;

    return {
      概要: properties['概要']?.title[0]?.plain_text || '',
      ステータス: properties['ステータス']?.status?.name || '',
      重大度: properties['重大度']?.select?.name || '',
      起票日: properties['起票日']?.created_time || '',
      URL: pageUrl
    };
  });
}
```

### 報告の整形とSlackへのポスト

テスト進捗情報、バグチケット情報がそれぞれ取得できたため、これらのデータを用いて報告用のテキストを整形します。

報告用テキストのフォーマットは以下の通りです。情報量が多すぎると認知負荷が高くなり誰にも読まれなくなるため、ポイントを絞って記載しています。

```mermaid
${当日の日付}終了時点での『${プロジェクト名}』のテスト進捗状況を報告します。

【テスト進捗状況 (${テストケース名})】
・テスト完了率 
${テスト完了率}% (${完了ケース数}ケース/${ケース数}ケース)

・本日起票されたバグチケット
${重大度}: <${URL}|${概要}> (${ステータス})
・・・

【バグチケットのステータス】
・チケット総数: ${チケット数}件
　・解決済み: ${解決済みチケット数}件 (${解決済み率}%)
　・未解決: ${未解決チケット数}件 (${未解決率}%)
　・Backlog: ${Backlgチケット数}件 (${Backlog率}%)

【コメント】
ToDo: コメントを記載する
```

各用語の定義は以下の通りです。

- テスト完了率
	- 対象のテストケースのケース数に対する、ステータスが”OK”または”FIXED” (※ 一度NGになったあと修正確認でOKになったもの) のケースの割合
- バグチケットのステータス
	- 解決済み
		- チケットのステータスが”Done”のもの
	- 未解決
		- チケットのステータスが”Open” (未修正), “In Progress” (修正中), または“Settled” (修正確認待ち) のもの
	- Backlog
		- 軽微な内容のためリリースと切り離して管理するとプロダクトチームで合意したもの

コメント欄については、QAエンジニアがあとから編集した上で各プロダクトチームに展開できるよう、ToDoの表記を含めています。テスト進捗が計画に対して順調かどうかや検出されているバグの傾向等は、現状QAエンジニア自身で記載してもらう必要があるためです。

最後に、整形されたテキストをSlack APIを用いてSlackにポストします。

![Slackへのポスト](https://cdn-ak.f.st-hatena.com/images/fotolife/e/enechain/20241106/20241106103100.png)

### 進捗報告生成のトリガー

ここまでで説明したテスト進捗報告の自動生成ですが、現状実行のトリガーは手動としています。GASではトリガーの種類として時間帯指定での定期実行等もサポートされていますが、QAエンジニアがその日の業務を終了するタイミングで最新の情報をもとに報告を生成できたほうが良いと考えました。

具体的には、Googleスプレッドシートのテストケーステンプレートのファイルに独自メニューを追加し、そこから実行できるようにしています。

![テストケーステンプレートの独自メニュー](https://cdn-ak.f.st-hatena.com/images/fotolife/e/enechain/20241106/20241106103106.png)

また、メニューを実行した際、プロンプトを表示して集計対象のシート名を入力できるようにしており、特定のテストだけを対象に進捗を報告するのか、システムテスト全体の進捗を報告するのかを指定できるようにしています。

## 今後の展望

まずは一旦仕組みが完成したばかりで、これから実運用に乗せていく段階です。報告内容や運用等、実際に利用してもらい、フィードバックを得ながら、より良いものに改善していければと思いますが、具体的に考えている今後の改善のアイデア (妄想) を1つお話します。

現状は、一旦テンポラリーのSlackチャンネルに自動生成された進捗報告をポストし、QAエンジニアがそこにコメントを追記した上で各プロダクトのSlackチャンネルにポストし直すという運用を想定しています。

ただ、理想的にはやはり直接各プロダクトのチャンネルにテスト進捗報告がポストされる状態を目指したいですよね？

そこで、スケジュール情報やバグチケットの内容等もデータとして取り込み、生成AIと繋ぎこむことで、テスト進捗が計画に対して順調かどうかや検出されているバグの傾向等を含むコメントについても、将来的に自動生成できればと考えています。

## まとめ

今回はシステムテストの日々の進捗報告の自動生成の取り組みについてご紹介しました。

enechainのQAチームでは他にも、自動テストの導入、Notionのバグチケットデータベースに蓄積されたデータをもとにしたプロダクト別品質ダッシュボードの構築等の取り組みが進行中です。これからも、システムや仕組みでQA業務やプロダクト品質の改善につながる様々な活動を推進しています。

我々が目指すのは、プロダクト品質と顧客への価値提供スピードのトレードオンです。QAチームでは、その実現を技術面でリードしていただけるリードQAエンジニアを募集しています。

[herp.careers](https://herp.careers/v1/enechain/Csc6wdOGIrla)

[« プロダクトやチームの特性から考えるPdMの…](https://techblog.enechain.com/entry/role-of-product-manager) [デザインシステムのダークモード対応 »](https://techblog.enechain.com/entry/designsystem-dark-mode-dev)