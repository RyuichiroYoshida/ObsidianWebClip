---
title: GitHub Projects で実現する社内資産管理
source: https://tech.route06.co.jp/entry/2024/11/12/171933
author:
  - "[[ROUTE06 Tech Blog]]"
published: 2024-11-12
created: 2025-05-06
description: ROUTE06 では全社ワークスペースとして GitHub を利用しており、社内資産をはじめとしたあらゆる情報を GitHub Projects で管理しています。 例えば、プロジェクトごとのアサイン状況、社内で承認/活用されているツールやライセンス、費用承認稟議など、その範囲は多岐にわたります。 GitHub という強力なプラットフォーム上に、社内のあらゆる情報が一元化されることよって生じるメリットは、非常に大きいと感じています。 一方で、デフォルトの機能だけではやや課題が残ることも事実です。 本記事は GitHub Actions でワークフローを構築することで課題を補ってみたよ、という記…
tags:
  - Git-GitHub
  - 1
read: false
---
ROUTE06 では [全社ワークスペースとして GitHub を利用](https://note.route06.co.jp/n/n5b21649308cf) しており、社内資産をはじめとしたあらゆる情報を GitHub Projects で管理しています。  
例えば、プロジェクトごとのアサイン状況、社内で承認/活用されているツールやライセンス、費用承認稟議など、その範囲は多岐にわたります。

GitHub という強力なプラットフォーム上に、社内のあらゆる情報が一元化されることよって生じるメリットは、非常に大きいと感じています。  
一方で、デフォルトの機能だけではやや課題が残ることも事実です。

本記事は GitHub Actions でワークフローを構築することで課題を補ってみたよ、という記事です。

## 課題

例えば、プロジェクトごとのアサイン状況であれば、以下のようなカスタムフィールドを準備し、管理します。

```
- プロジェクト名
- 稼働割合
- 稼働開始日
- 稼働終了日
- 雇用形態
- 役割
- ...
```

![](https://cdn-ak.f.st-hatena.com/images/fotolife/r/route06/20241112/20241112171935.png)

GitHub Projects で情報管理をしていくなかで、以下のような点が課題でした。

- カスタムフィールドの更新に気付けない
- カスタムフィールドは更新履歴を持てない

## アプローチ

端的には、カスタムフィールドと Issue の Description とを同期させる、というアプローチをとりました。

具体的には以下のとおりです。

- カスタムフィールドに合わせた [Issue フォームテンプレート](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/configuring-issue-templates-for-your-repository#creating-issue-forms) を作成
- Issue フォーム経由で作成された内容をパースし、カスタムフィールドに値を設定するワークフローを構築
- カスタムフィールドの値が更新されたら Issue の Description に反映し、更新前後の差分内容を Issue にコメントするワークフローを構築

Issue へのコメントは Slack 等へ通知することも容易なため、カスタムフィールドの更新をすぐに検知することができます。

なお、本アプローチを取るにあたり Issue の Description が更新されることまでカバーすることは過剰なため、注意書きを挿入することで運用上の制限を明示するなどの工夫を入れています。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/r/route06/20241112/20241112171939.png)

※以降、記載しているスクリプトはところどころ省略・改変しています。

```yaml
...

jobs:
  insert_caution_description_to_issue:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Insert caution description to issue
        id: insert_caution_description
        uses: actions/github-script@v7
        with:
          script: |
            /**
             * description冒頭に注意書きを挿入する
             * @param {object} issue
             * @returns {void}
             */
            const insertCautionDescription = async (issue) => {
              // 別途用意している注意書きテンプレートを利用
              const fs = require('fs');
              const cautionText = fs.readFileSync('./.github/workflows/_templates/caution.md', 'utf8');
              
              const { data: issueData } = await github.rest.issues.get({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: issue.number
              });

              await github.rest.issues.update({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: issue.number,
                body: \`${cautionText}\n${issueData.body}\`
              });
            }

            await insertCautionDescription(context.payload.issue);

...
```

## 構築したなかでのあれこれ

Issueのパースはフォーム経由で作成されることを前提として組んでます。

```yaml
...

steps:
    - name: Parse issue description for custom fields values
    id: parse_issue_description
    uses: actions/github-script@v7
    with:
        script: |
        const body = context.payload.issue.body || '';
        const fields = JSON.parse(process.env.FIELDS);
        const extractField = (field) => {
            // Issueフォームテンプレート経由で作成されることを想定
            const regex = new RegExp(\`###\\s*${field}\\s*\\n([\\s\\S]*?)(?=\\n###|$)\`, "m");
            const match = body.match(regex);
            return match ? match[1].trim() : '';
        };
        for (const [field, varName] of Object.entries(fields)) {
            const value = extractField(field);
            core.exportVariable(varName, value);
        }

...
```

Projects をAPIで操作するにあたっては、以下のドキュメントを中心に参照していました（ちなみに GraphQL API を叩くときは [Explorer](https://docs.github.com/en/graphql/overview/explorer) 便利ですよ。  
[Using the API to manage Projects - GitHub Docs](https://docs.github.com/en/issues/planning-and-tracking-with-projects/automating-your-project/using-the-api-to-manage-projects)

なお、ワークフローから ProjectV2 系 の API を叩くには GitHub App で認証を行う必要があるため、こちらもドキュメントを参照しつつ進めるとよいです。  
[Automating Projects using Actions - GitHub Docs](https://docs.github.com/en/issues/planning-and-tracking-with-projects/automating-your-project/automating-projects-using-actions)

```yaml
...

- name: Generate token
  id: generate_token
  uses: actions/create-github-app-token@v1
  with:
    app-id: ${{ vars.APP_ID_ROUTE06_PROJECT_OPERATOR }}
    private-key: ${{ secrets.APP_PEM_ROUTE06_PROJECT_OPERATOR }}

- name: Get project ID
  id: get_github_project_id
  uses: actions/github-script@v7
  with:
    github-token: ${{ steps.generate_token.outputs.token }}
    script: |
      const organization = process.env.ORGANIZATION;
      const projectName = process.env.PROJECT_NAME;
      const query = \`
        query ($organization: String!) {
          organization(login: $organization) {
            projectsV2(first: 100) {
              nodes {
                id
                title
              }
            }
          }
        }
      \`;
      const result = await github.graphql(query, { organization });
      const projects = result.organization.projectsV2.nodes;
      const project = projects.find(p => p.title === projectName);
      if (project) {
        core.setOutput('project_id', project.id);
      } else {
      }

...
```

ワークフローについては、先述の通り「 Issue 作成をトリガーにカスタムフィールドへ値をセットする」ものと、「カスタムフィールドの値を Issue に同期し、差分をコメントする」ものと2つ実装しているのですが、後者にはProjectV2 に関するトリガーは現時点で存在しないため projects に登録されたすべての Issue を走査し、差分があれば同期するようにしています。

Issue 作成をトリガーにカスタムフィールドへ値をセットするワークフローのスクリプトはこのような感じ。

```yaml
...

# FIELDSというカスタムフィールドをマッピングした変数を持たせてます
jobs:
  set_custom_fields:
    runs-on: ubuntu-latest
    env:
      FIELDS: |
        {
          "Status": "status",
          "プロジェクト": "project_name",
          "稼働割合": "work_rate",
          "稼働開始日": "work_start_date",
          "稼働終了日": "work_end_date",
          "契約コード": "contract_code",
          "雇用形態": "employment_type",
          "役割": "role"
        }

...

- name: Add issue to GitHub Project
  id: add_to_project
  uses: actions/github-script@v7
  with:
    github-token: ${{ steps.generate_token.outputs.token }}
    script: |
      const projectId = '${{ steps.get_github_project_id.outputs.project_id }}';
      const contentId = context.payload.issue.node_id;

      const response = await github.graphql(\`
        mutation($projectId: ID!, $contentId: ID!) {
          addProjectV2ItemById(input: {projectId: $projectId, contentId: $contentId}) {
            item {
              id
            }
          }
        }
      \`, {
        projectId: projectId,
        contentId: contentId,
      });

      const itemId = response.addProjectV2ItemById.item.id;
      core.setOutput("item_id", itemId);

- name: Set custom fields
  id: set_custom_fields
  uses: actions/github-script@v7
  with:
    github-token: ${{ steps.generate_token.outputs.token }}
    script: |
      const projectId = '${{ steps.get_github_project_id.outputs.project_id }}';
      const itemId = '${{ steps.add_to_project.outputs.item_id }}';

      // NOTE: https://docs.github.com/ja/graphql/reference/interfaces#projectv2fieldcommon
      const getFieldsMetaData = await github.graphql(\`
        query($projectId: ID!) {
          node(id: $projectId) {
            ... on ProjectV2 {
              fields(first: 20) {
                nodes {
                  ... on ProjectV2FieldCommon {
                    id
                    name
                    dataType
                  }
                  ... on ProjectV2SingleSelectField {
                    id
                    name
                    dataType
                    options {
                      id
                      name
                    }
                  }
                }
              }
            }
          }
        }
      \`, {
        projectId: projectId,
      });

      const projectAllFields = getFieldsMetaData.node.fields.nodes;
      const targetFields = JSON.parse(process.env.FIELDS);
      let fields = {};
      let singleSelectFieldNames = [], dateFieldNames = [], numberFieldNames = [];

      // カスタムフィールドの型に合わせてあげる必要があります
      projectAllFields.forEach(field => {
        const fieldName = field.name;
        if (targetFields[fieldName]) {
          if (field.dataType === 'SINGLE_SELECT') {
            singleSelectFieldNames.push(targetFields[fieldName]);
            for (const option of field.options) {
              if (option.name === process.env[targetFields[fieldName]].trim()) {
                fields[targetFields[fieldName]] = {
                  'fieldId': field.id,
                  'value': option.id,
                };
                break;
              }
            }
          } else if (field.dataType === 'DATE') {
            dateFieldNames.push(targetFields[fieldName]);
            fields[targetFields[fieldName]] = {
              'fieldId': field.id,
              'value': process.env[targetFields[fieldName]].trim(),
            };
          } else if (field.dataType === 'NUMBER') {
            numberFieldNames.push(targetFields[fieldName]);
            fields[targetFields[fieldName]] = {
              'fieldId': field.id,
              'value': process.env[targetFields[fieldName]].trim(),
            };
          } else {
            fields[targetFields[fieldName]] = {
              'fieldId': field.id,
              'value': process.env[targetFields[fieldName]].trim(),
            };
          }
        }
      });

      async function updateCustomFields(github) {
        for (const [key, { fieldId, value }] of Object.entries(fields)) {
          let mutation, variables;
          // 単一選択フィールドの更新
          if (singleSelectFieldNames.includes(key)) {
            mutation = \`
              mutation($projectId: ID!, $itemId: ID!, $fieldId: ID!, $singleSelectOptionId: String!) {
                updateProjectV2ItemFieldValue(input: {
                  projectId: $projectId,
                  itemId: $itemId,
                  fieldId: $fieldId,
                  value: { singleSelectOptionId: $singleSelectOptionId }
                }) {
                  projectV2Item {
                    id
                  }
                }
              }
            \`;
            variables = {
              projectId,
              itemId,
              fieldId,
              singleSelectOptionId: value
            };
          // 日付フィールドの更新
          } else if (dateFieldNames.includes(key)) {
            mutation = \`
              mutation($projectId: ID!, $itemId: ID!, $fieldId: ID!, $date: Date!) {
                updateProjectV2ItemFieldValue(input: {
                  projectId: $projectId,
                  itemId: $itemId,
                  fieldId: $fieldId,
                  value: { date: $date }
                }) {
                  projectV2Item {
                    id
                  }
                }
              }
            \`;
            variables = {
              projectId,
              itemId,
              fieldId,
              date: value
            };
          // 数値フィールドの更新
          // ...省略
          // テキストフィールドの更新
          // ...省略

          await github.graphql(mutation, variables);
        }
      }
      await updateCustomFields(github);

...
```

カスタムフィールドの値を Issue に同期し、差分をコメントするワークフローのスクリプトはこんな感じです。

```yaml
...

# diffを取るためにjsdiffを別stepで入れてます https://www.npmjs.com/package/diff

- name: Sync project issues
  id: sync_project_issues
  uses: actions/github-script@v7
  with:
    github-token: ${{ steps.generate_token.outputs.token }}
    script: |
      const projectId = '${{ steps.get_github_project_id.outputs.project_id }}';

      // Projectに登録されたIssueを取得
      const fetchProjectIssues = async (projectId, cursor = null) => {
        return await github.graphql(\`
          query($projectId: ID!, $cursor: String) {
            node(id: $projectId) {
              ... on ProjectV2 {
                items(first: 100, after: $cursor) {
                  pageInfo {
                    hasNextPage
                    endCursor
                  }
                  nodes {
                    content {
                      ... on Issue {
                        id
                      }
                    }
                    fieldValues(first: 100) {
                      nodes {
                        ... on ProjectV2ItemFieldSingleSelectValue {
                          field {
                            ... on ProjectV2FieldCommon {
                              name
                            }
                          }
                          name
                        }
                        ... on ProjectV2ItemFieldTextValue {
                          field {
                            ... on ProjectV2FieldCommon {
                              name
                            }
                          }
                          text
                        }
                        // ... 省略
                      }
                    }
                  }
                }
              }
            }
          }
        \`, {
          projectId: projectId,
          cursor: cursor
        });
      };

      // Projectに登録されたすべてのIssueを取得
      let allIssueNodes = [], cursor = null, hasNextPage = true;
      do {
        const projectIssuesResult = await fetchProjectIssues(projectId, cursor);
        allIssueNodes = allIssueNodes.concat(projectIssuesResult.node.items.nodes);
        cursor = projectIssuesResult.node.items.pageInfo.endCursor;
        hasNextPage = projectIssuesResult.node.items.pageInfo.hasNextPage;
      } while (hasNextPage);

      // 新しいIssueの説明を作成する
      const constructNewIssueDescription = (issueNode) => {
        // ...省略

        for (const fieldValue of fieldValues) {
          if (fieldValue.field && fieldValue.field.name) {
            const fieldName = fieldValue.field.name;
            if (fieldName !== "Title") {
              let fieldValueText = fieldValue.name || fieldValue.text || fieldValue.number || fieldValue.date;
              if (fieldValueText !== null && fieldValueText !== undefined) {
                description += \`### ${fieldName}\n\`;
                description += \`${fieldValueText}\n\n\`;
              }
            }
          }
        }
        return description.trim();
      };

      // 現在のIssueの説明を取得する
      const getCurrentIssueDescription = async (issueId) => {
        const issueResult = await github.graphql(\`
          query($issueId: ID!) {
            node(id: $issueId) {
              ... on Issue {
                body
              }
            }
          }
        \`, {
          issueId: issueId,
        });
        return issueResult.node.body;
      };

      // 説明を更新する
      const updateIssueDescription = async (issueId, newDescription) => {
        await github.graphql(\`
          mutation($issueId: ID!, $description: String!) {
            updateIssue(input: {id: $issueId, body: $description}) {
              issue {
                id
              }
            }
          }
        \`, {
          issueId: issueId,
          description: newDescription,
        });
      }

      // 差分をコメントする
      const commentDifferences = async (issueId, differences) => {
        let commentBody = "Changes in the issue description:\n\n\`\`\`diff\n";
        // ...省略

        await github.graphql(\`
          mutation($issueId: ID!, $body: String!) {
            addComment(input: {subjectId: $issueId, body: $body}) {
              commentEdge {
                node {
                  id
                }
              }
            }
          }
        \`, {
          issueId: issueId,
          body: commentBody
        });
      }

      const issueNodes = allIssueNodes;
      for (const node of issueNodes) {
        if (!node.content || !node.content.id) continue;
        const newDescription = constructNewIssueDescription(node);
        const currentDescription = await getCurrentIssueDescription(node.content.id);

        const diff = require('diff');
        const differences = diff.diffLines(currentDescription, newDescription);
        // 説明が変更されていない場合はスキップ
        if (differences.length === 1 && differences[0].added === false && differences[0].removed === false) {
          console.log(\`Log: No changes in the issue description. Issue ID: ${node.content.id}\`);
          continue;
        }

        // 説明を更新する
        await updateIssueDescription(node.content.id, newDescription);

        // 差分をコメントする
        await commentDifferences(node.content.id, differences);
      }

...
```

また、今回構築したものは [Reusable Workflow](https://docs.github.com/en/actions/sharing-automations/reusing-workflows) にしたかったのですが create-github-app-token の仕様上 job をまたいだトークンの引き渡しができないということで、一旦見送りました。  
[output token cannot be used across jobs · Issue #66 · actions/create-github-app-token](https://github.com/actions/create-github-app-token/issues/66)

## 最後に

本記事では GitHub Projects を活用した社内資産管理の取り組みと、その課題を補うためのワークフロー構築についてご紹介しました。

資産管理に特化した SaaS の利用や、別の手段での管理ツール構築など、ほかの選択肢もあるなか、 GitHub Projects を選択し、試行錯誤しながら運用を進めていく中で、その可能性や価値に気付いた点もありました。  
Issue・PR との自然な連携、 Actions による自動化、 API を活用した拡張性など GitHub というプラットフォームを「使い倒していく」プロセスそのものが、新しいワークフローやツールの創造につながっていくのだと考えています。

開発者のワークスペースとして既に確立されている GitHub 上で資産管理を行うことは、情報の一元化や権限管理の統一といった明確なメリットをもたらします。  
本記事で触れたような機能面での制約は確かにありますが、それらを乗り越えるプロセスが、むしろプラットフォームとしての GitHub の新しい可能性を発見する機会となっています。

この取り組みは、まだ始まったばかりです。今後も GitHub の持つ可能性を探求し、より良い仕組みづくりにチャレンジしていきたいと考えています。