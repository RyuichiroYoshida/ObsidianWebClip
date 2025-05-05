---
title: GitHubをコードで管理 ! Terraformを導入して安全な管理を実現しました
source: https://tech.route06.co.jp/entry/2024/10/18/153755
author:
  - "[[ROUTE06 Tech Blog]]"
published: 2024-10-18
created: 2025-05-06
description: ROUTE06 では GitHub の管理に Terraform を導入しました。今回はその導入の背景、実際に導入してどう変わったのか、導入方法について紹介したいと思います。 Terraform とは Terraform は、IaC（Infrastructure as Code）ツールの一種です。 インフラの設定をコードとして管理することで、設定の変更履歴が明確になり、誤った設定によるトラブルを防ぐことができます。 なぜ GitHub を Terraform で管理するのか ROUTE06 では、全社的に GitHub を使用しています。そのため、GitHub の管理は非常に重要です。 Ter…
tags:
  - IaC
  - 1
  - Terraform
  - インフラ
read: false
---
ROUTE06 では GitHub の管理に Terraform を導入しました。今回はその導入の背景、実際に導入してどう変わったのか、導入方法について紹介したいと思います。

## Terraform とは

Terraform は、IaC（Infrastructure as Code）ツールの一種です。  
インフラの設定をコードとして管理することで、設定の変更履歴が明確になり、誤った設定によるトラブルを防ぐことができます。

## なぜ GitHub を Terraform で管理するのか

ROUTE06 では、全社的に GitHub を使用しています。そのため、GitHub の管理は非常に重要です。  
Terraform 導入前には、以下のような課題がありました。

- 手動での設定変更時にミスが発生する
- 設定変更の履歴が追いにくい
- 重要な変更（リポジトリの作成や Organization へのユーザー招待など）に対してレビューができない

## 実際に Terraform を導入してどう変わったか

### 手動での変更 → Pull request ベースでの変更へ

リポジトリの作成やユーザー招待といった重要な設定変更は、すべて Terraform で管理する運用に変更しました。  
これにより、作業者とは別の人が変更内容をレビューできるようになり、ミスを防ぐことができるようになりました。  
以下に運用のイメージを示します。

```mermaid
ReviewerGitHub ActionsRepositoryUserReviewerGitHub ActionsRepositoryUserPull Request を作成terraform plan を実行結果をコメントPull Request をレビューPull Request にコミットを pushterraform plan を再実行結果をコメントレビューを承認\`trigger-terraform-apply\` ラベルを Pull Request に付けるterraform apply を実行結果をコメントPull Request をマージ
```

### 操作履歴が残る

すべての操作が Pull Request 経由で実行されるため、過去の操作を再現したい場合には、該当する Pull Request を参照できます。  
また、設定内容を以前の状態に戻したい場合も、過去の Pull Request を revert することで簡単に設定を復元できるようになりました。

### Invite しようとしているユーザーの本人確認ができるようになった

GitHub のユーザー名（Username）は、使用されていなければ誰でも取得できてしまうため、例えば「before」というユーザーが「before」から「after」へユーザー名を変更した場合、別のユーザーが「before」という名前を新たに取得し、元のユーザーになりすますことが可能です。  
このような **なりすましによるセキュリティリスク** に対処するため、組織にユーザーを招待する際に、招待対象の GitHub ユーザー名とその GitHub ID の一致性を事前に確認する仕組みを導入しました。

## Terraform導入の要件

Terraform導入にあたり、以下の要件がありました。

- Terraform の設定ファイルは、社員なら誰でも編集して Pull Request を作成できるようにしたい
- tfstate ファイルへのアクセスは、権限を持つ人に限定したい
- Terraform Apply で不具合が発生した場合、tfstate ファイルを不具合発生前の状態に戻せるようにしたい
- tfstate ファイルにアクセスできるユーザーと各 Organization の Owner 権限を持つユーザーとが必ず一致するようにしたい

これらの要件を満たすために、通常では Terraform の状態管理ファイル（tfstate）を AWS S3 などの外部ストレージに保存することが一般的ですが、今回は状態管理ファイルも GitHub リポジトリで管理することで、GitHub 内で完結した運用を実現しました。以下、その実装方法を説明します。

## 実装方法

### 1\. リポジトリの用意

以下、2つのリポジトリを作成します。

- Terraform コードを管理するリポジトリ
	- 適切な名前をつけてください。例として `terraform-operations` とします。
- tfstate ファイルを管理するリポジトリ
	- 適切な名前をつけてください。例として `terraform-state-files` とします。
	- `tfstate ファイルはアクセスできる人を限定したい` という要件を満たすため、tfstate ファイル専用のリポジトリを別途作成します。
	- tfstate ファイルは GitHub Actions によって `terraform apply` 実行後に保存されます。
	- 特に分ける必要が無ければ、このリポジトリを作成する必要はありません。

### 2\. 作成したリポジトリへアクセス権限を追加する

作成したリポジトリに適切なアクセス権限を設定してください。  
tfstate ファイルを管理するリポジトリ（例と同じように作成している場合 `terraform-state-files` ）については、基本的に手動では操作しないため、アクセス権限を付与しなくても問題ありません。

### 3\. ラベル作成

Terraform コマンドを GitHub Action で実行します。  
`terraform plan` は Pull Request を作成した時点で実行されるように設定しますが、 `terraform apply` は指定したタイミング（今回はレビューが完了し、マージする前）で実行されるようにしたいため、 `terraform apply` を実行するためのトリガーを作成します。  
トリガーは運用ニーズに合わせてさまざまな選択肢がありますが、ここではラベルをトリガーとする方法を紹介します。  
以下のラベル名で、トリガーに使用するラベルを作成してください。

ラベル名: `trigger-terraform-apply`

### 4\. GitHub App の作成

Terraform Plan と Terraform Apply で使用する GitHub App を作成します。  
GitHub App の作成には Organization の Owner 権限が必要です。

#### Terraform Plan で使用する GitHub App

1. GitHub App の Name には適切な名前をつける（ Name は34文字の制限があるので収まるように設定してください）
2. Homepage URL には Terraform コードを記入するリポジトリ（例と同じように作成している場合 `terraform-operations` ）の URL を設定する
3. Webhook は不要なので Active のチェックボックスを外す
4. Repository permissions を以下のように設定する
	- Administration: `Read-only`
	- Contents: `Read and write`
	- Metadata: `Read-only`
	- Pages: `Read-only` （ 注1 ）
	- Secrets: `Read-only`
	- Variables: `Read-only`
5. Organization permissions を以下のように設定する
	- Members: `Read-only` （ 注2 ）
	- Secrets: `Read-only`
	- Variables: `Read-only`
6. "Where can this GitHub App be installed?" を `Only on this account` に設定する（ 注3 ）
7. "Create GitHub App" を押す
8. App の作成が完了したら "Install App" タブから Organization 名の横にある Install を押す
9. Repository access の設定を行うの項目で `Only select repositories` を選択して、検証用に使用するリポジトリなどを選択する
	- Terraform で管理するリポジトリを増やす際には、 `All repositories` としておくと、個別に許可設定する必要がなくなります。

#### Terraform Applyで使用するGitHub App

1. GitHub App の Name には適切な名前をつける（ Name は34文字の制限があるので収まるように設定してください）
2. Homepage URL には Terraform コードを記入するリポジトリ（例と同じように作成している場合 `terraform-operations` ）のURLを設定する
3. Webhook は不要なので Active のチェックボックスを外す
4. Repository permissions を以下のように設定する
	- Administration: `Read and write`
	- Contents: `Read and write`
	- Metadata: `Read-only`
	- Pages: `Read and write` （ 注1 ）
	- Secrets: `Read-only`
	- Variables: `Read-only`
5. Organization permissions を以下のように設定する
	- Administration: `Read and write`
	- Custom organization roles: `Read and write`
	- Custom properties: `Read and write`
	- Custom repository roles: `Read and write`
	- Members: `Read and write` （ 注2 ）
	- Secrets: `Read-only`
	- Variables: `Read-only`
6. "Where can this GitHub App be installed?" を `Only on this account` にする（ 注3 ）
7. "Create GitHub App" を押す
8. App の作成が完了したら "Install App" タブから Organization 名の横にある Install を押す
9. Repository access の項目で `Only select repositories` を選択して、検証用に使用するリポジトリなどを選択する
	- Terraform で管理するリポジトリを増やす際には、 `All repositories` としておくと、個別に許可設定する必要がなくなります。
- 注1: 設定は任意です。GitHub Pages の設定を Terraform 管理する場合は設定してください。
- 注2: 設定は任意です。Organization のユーザー招待を Terraform 管理する場合は設定してください。
- 注3: 設定は任意です。検証を行う場合は `Only on this account` にすることをお勧めします。

### 5\. secrets and variables の設定

secrets and variables key の設定を行います。

- [GitHub App の作成手順](https://tech.route06.co.jp/entry/2024/10/18/#githubapp) で作成した2つの GitHub App に対して、App ID、Installation ID、PRIVATE\_KEY のそれぞれの値を設定します。
	- App ID、PRIVATE\_KEYの確認方法は、 [GitHub App 登録の変更 - GitHub App 設定への移動](https://docs.github.com/ja/apps/maintaining-github-apps/modifying-a-github-app-registration#github-app-%E8%A8%AD%E5%AE%9A%E3%81%B8%E3%81%AE%E7%A7%BB%E5%8B%95) を参照してください。  
		該当箇所イメージ  
		![](https://cdn-ak.f.st-hatena.com/images/fotolife/r/route06/20241018/20241018153756.png) ![](https://cdn-ak.f.st-hatena.com/images/fotolife/r/route06/20241018/20241018153804.png)
	- Installation IDの確認方法は、 [インストールした GitHub App の確認と変更 - 確認または変更したい GitHub App に移動する](https://docs.github.com/ja/apps/using-github-apps/reviewing-and-modifying-installed-github-apps#%E7%A2%BA%E8%AA%8D%E3%81%BE%E3%81%9F%E3%81%AF%E5%A4%89%E6%9B%B4%E3%81%97%E3%81%9F%E3%81%84-github-app-%E3%81%AB%E7%A7%BB%E5%8B%95%E3%81%99%E3%82%8B) を参照してください。  
		該当箇所イメージ  
		![](https://cdn-ak.f.st-hatena.com/images/fotolife/r/route06/20241018/20241018153800.png)
- PRIVATE\_KEY は secrets の Repository secrets に作成してください。
- App ID と Installation ID は variables の Repository variables に作成してください。

### 6\. Terraform 操作するリポジトリ（ terraform-operations ）に必要なファイルを作成する

- サンプルコードはこちらになります。 [https://github.com/route06/github-terraform-examples](https://github.com/route06/github-terraform-examples)
- 必要なファイルをダウンロードするか、コピーして使用してください。
- 以下に、各ファイルの説明を記載します。

#### .terraform-version

- Terraform のバージョンを指定するためのファイルです。
- 使用したい Terraform のバージョンを記載してください。
- バージョンの一覧はこちらから確認できます。 [https://releases.hashicorp.com/terraform/](https://releases.hashicorp.com/terraform/)

#### tfcmt.yml

- [suzuki-shunsuke/tfcmt](https://github.com/suzuki-shunsuke/tfcmt) を使うと、Terraform コマンドの出力を GitHub の Pull Request にコメントとして投稿できます。
- 設定用の.yml ファイルの詳しい書き方は [https://suzuki-shunsuke.github.io/tfcmt/config](https://suzuki-shunsuke.github.io/tfcmt/config) を参照してください。

#### main.tf

- Terraform で実行したい操作を記述するメインのファイルです。
- 各リソースの詳しい内容は [公式ドキュメント](https://registry.terraform.io/providers/integrations/github/latest/docs) を参照してください。
- ここでは、ポイントとなる部分を説明します。
```hcl
resource "github_membership" "org_owner" {
  for_each = toset(local.org_owner_usernames)

  username = each.value
  role     = "admin"

  depends_on = [
    module.security
  ]
}
```

GitHub Organization へユーザーを招待するリソースを定義しています。  
ここでは、 `role = "admin"` となっているので Organization Owner を招待するリソースになっています。  
member 権限で招待したい場合は `role = "member"` にすると招待できます。

```hcl
resource "github_repository" "terraform_operations" {
  name                   = "terraform-operations"
  description            = "Organization 配下のリソースを管理する Terraform ソースの置き場所"
  visibility             = "private"
  allow_auto_merge       = false
  allow_merge_commit     = true
  allow_rebase_merge     = true
  allow_squash_merge     = true
  allow_update_branch    = false
  delete_branch_on_merge = true
  has_discussions        = false
  has_issues             = true
  has_projects           = false
  has_wiki               = false
  homepage_url           = null
  is_template            = false
  vulnerability_alerts   = true
  security_and_analysis {
    advanced_security {
      status = "disabled"
    }
    secret_scanning {
      status = "disabled"
    }
    secret_scanning_push_protection {
      status = "disabled"
    }
  }
}
```

ここでは、リポジトリの定義を行っています。  
サンプルでは、 Terraform 操作するリポジトリ（ `terraform-operations` ） と tfstate ファイルを管理するリポジトリ（ `terraform-state-files` ）のリソースを定義していますが、必要に応じて変更してください。

```hcl
module "security" {
  source = "./module/security"

  users_defined = var.users
}
```

ここでは、なりすましをチェックする module を呼び出しています。  
module内の処理については後ほど説明します。

```hcl
variable "pem_file" {
  description = "The content of the PEM file"
  type        = string
}

# Organization owner として任命するユーザー名を受け取るための変数
variable "org_owners" {
  description = "List of users to assign the 'owner' role for the organization"
  type        = list(string)
}

# ユーザー情報を users.tfvars から受け取るための変数
variable "users" {
  description = "List of users"
  type = list(object({
    username = string
    id       = string
  }))
}
```

variable は、変数を定義するためのブロックです。  
ここでは、GitHub App の PRIVATE KEY を受け取るための変数と、 ユーザー情報を受け取るための変数を定義しています。

#### module security

- module とは、Terraform のコードを再利用するための仕組みです。詳しくは、 [公式ドキュメント](https://developer.hashicorp.com/terraform/language/modules) を参照してください。
- module security では、ユーザーの GitHub Username と GitHub ID が `users.tfvars` の内容と一致するかチェックを行っています。
```hcl
data "github_user" "this" {
  for_each = { for user in var.users_defined : user.username => user }
  username = each.value.username
}
```

ここでは、GitHub から users.tfvars の GitHub Username に紐づくユーザー情報を取得します。

```hcl
locals {
  users_map = { for user in var.users_defined : user.username => user }
  users_with_unexpected_id = [
    for username, user_info in data.github_user.this :
    username if user_info.id != local.users_map[username].id
  ]
  all_users_have_correct_id = length(local.users_with_unexpected_id) == 0
}
```

data.github\_user.this から取得したユーザーの GiHub ID と local.users\_map に格納されている ID（ users.tfvars に記載されている GitHub ID ）が一致するかをチェックしています。  
一致しない場合、そのユーザー名を `all_users_have_correct_id` リストに追加します。

```hcl
resource "null_resource" "fail_with_unexpected_user_id" {
  # 全てのユーザーの name-id が意図通りなら count が 0 になるので評価されない(エラーにならないので Action が失敗せず apply できる) 
  count = local.all_users_have_correct_id ? 0 : 1

  provisioner "local-exec" {
    command     = "echo 'Error: One or more users have mismatching IDs.' && exit 1"
    interpreter = ["/bin/sh", "-c"]
  }

  triggers = {
    always_run = timestamp() # timestamp() の内容は常に変化するので count が 1 ならば毎回評価される
  }
}
```

`all_users_have_correct_id` リストが空であれば、全てのユーザーの GitHub ID が一致していることになります。  
一致していない場合、エラーを出力します。

#### users.tfvars

- `.tfvars` ファイルは、変数の値を定義するためのファイルです。 `users.tfvars` では、ユーザー情報を定義します。
- users ブロックにはユーザーの GitHub Username と GitHub ID を記入します。 GitHub ID は、なりすまし検知に利用します。
- GitHub ID とは、GitHub が自動的に生成する一意の識別番号です。一度割り当てられた GitHub ID は変更できません。

#### imports.tf

- `import.tf` ファイルには、 `terraform import` コマンドを経由して既存のリソースを Terraform の管理下に置くために利用します。
- 既存のリポジトリを Terraform 管理下に置くためには import ブロックが必要です。
- 複数の import ブロックを一つのファイルにまとめたものが imports.tf です。
- 詳しくは [公式ドキュメント - import](https://developer.hashicorp.com/terraform/cli/import) を参照してください。

サンプルでは、 Terraform コードを管理するリポジトリ（ `terraform-operations` ） 、 tfstate ファイルを管理するリポジトリ（ `terraform-state-files` ）、および Organization owner 設定に対する import ブロックを記載していますが、必要に応じて変更してください。

#### Makefile

- `Makefile` を利用して、各種 Terraform コマンド実行時の引数が適切に設定されるようにしています。
- `terraform plan` や、 `terraform apply` コマンド以外にも、フォーマットチェックや TFLint の設定も行っていますので、必要に応じて変更してください。

#### ワークフローについて

以下のワークフローを実行することで、 `terraform fmt` 、 `terraform plan` 、 `terraform apply` を実行することができます。 それぞれのワークフローの役割は以下の通りです。

- terraform-all-test.yml
	- `terraform fmt` や `terraform validate` を実行することで、 `plan/apply` 以前にコードが最低限の品質をクリアしているかをチェックしています。（サンプルコードでの実行タイミングは、Pull Request 作成時と条件に一致したファイルが更新された場合です）
- terraform-apply.yml
	- `terraform apply` を行います。（サンプルコードでの実行タイミングは、Pull Request にラベルが付与された時かつ条件に一致したファイルが更新された場合です）
- terraform-plan.yml
	- `terraform plan` を行います。（サンプルコードでの実行タイミングは、Pull Request 作成時と条件に一致したファイルが更新された場合です）

### 7\. 実行する

Pull Request を作成すると Terraform Plan が実行されます。 処理が完了すると、コメントに結果が表示されます。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/r/route06/20241018/20241018153807.png)

Terraform Plan の結果が問題なければ、Pull Request に `trigger-terraform-apply` ラベルをつけて、Terraform Apply を行ってください。 処理が完了すると、 `terraform` コマンドの出力がコメントとして投稿されます。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/r/route06/20241018/20241018153811.png)

## まとめ

本記事では、Terraform を活用して GitHub の設定を管理する方法を紹介しました。

Terraform の導入を決定した当初、筆者は "Terraform" というツールの存在は知っていたものの、実際に使ったことはなく、未経験の状態でした。  
しかし、社内エンジニアのサポートにより、無事に導入を完了し、個人としても知識と経験を深めることができました。

ROUTE06 では、全ての Organization に Terraform を導入しましたが、現状の課題として、エンジニアではない（Terraform の経験がない）メンバーが Pull Request を申請するのは、まだまだハードルが高い状況です。

より多くのメンバーにとって使いやすくなるように、今後も改善を進めていきます。

この記事が、皆様の GitHub 運用の改善に少しでもお役に立てれば幸いです。