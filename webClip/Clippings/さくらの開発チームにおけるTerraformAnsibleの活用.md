---
title: さくらの開発チームにおけるTerraform/Ansibleの活用
source: https://knowledge.sakura.ad.jp/38635/
author:
  - "[[さくらのナレッジ]]"
published: 2024-08-13
created: 2025-05-06
description: 目次はじめに目的IaCの理解と実践さくらのクラウドでTerraformを利用するチームに合ったスタイルをモジュールのディレクトリ構成も同様サーバログイン設定サーバ名を名前解決可能にする使い回せるロールを作るまとめ はじめ […]
tags:
  - 1
  - Terraform
  - クラウド
  - バックエンド
  - IaC
read: false
---
![](https://knowledge.sakura.ad.jp/wp-content/uploads/2024/07/kawai-cover.jpg)

目次

- [はじめに](https://knowledge.sakura.ad.jp/38635/#i)
- [目的](https://knowledge.sakura.ad.jp/38635/#i-2)
- [IaCの理解と実践](https://knowledge.sakura.ad.jp/38635/#IaC)
- [さくらのクラウドでTerraformを利用する](https://knowledge.sakura.ad.jp/38635/#Terraform)
- [チームに合ったスタイルを](https://knowledge.sakura.ad.jp/38635/#i-3)
- [モジュールのディレクトリ構成も同様](https://knowledge.sakura.ad.jp/38635/#i-4)
- [サーバログイン設定](https://knowledge.sakura.ad.jp/38635/#i-5)
- [サーバ名を名前解決可能にする](https://knowledge.sakura.ad.jp/38635/#i-6)
- [使い回せるロールを作る](https://knowledge.sakura.ad.jp/38635/#i-7)
- [まとめ](https://knowledge.sakura.ad.jp/38635/#i-8)

## はじめに

さくらのクラウドにはいくつかの開発チームがありますが、その中で私が所属しているガンマチームにおけるTerraformやAnsibleの活用というテーマで川井が発表させていただきます。

内容としては、まずこの発表の目的を説明し、IaC (Infrastructure as Code)とはそもそも何かという話をして、それからさくらのクラウドでTerraformをどのように活用しているか、またAnsibleをどのように活用しているかを発表します。

## 目的

今回はIaCの勉強会ということで、IaCの理解と実践を目的としています。この勉強会に参加することで皆さんがTerraformやAnsibleを理解し、インフラ構築に活用できるようになることを目指したいと思います。

## IaCの理解と実践

この発表ではIaCを以下のように定義します。

「IaC(Infrastructure as Code)とは、手動でインフラストラクチャを構成するのではなく、コードを使用してインフラストラクチャの構成管理とプロビジョニングを行うプロセスです」

もちろん他にもいろいろな定義のしかたはあると思いますが、この発表ではいったんこういうものだということで話を進めていきます。

IaCを実現するには「宣言型」と「命令型」があります。宣言型はシステムやリソースの望ましい状態を定義するアプローチです。もう一方の命令型は、望ましいリソース量やシステム設定を実現するためのステップ(コマンド)を定義するアプローチです。宣言型のアプローチで使われるツールがTerraformで、命令型で使われるのがAnsibleというような分け方がなされることが多いですね。IaCはDevOpsやCI/CDとの関連も深いです。

## さくらのクラウドでTerraformを利用する

さくらのクラウドでTerraformを利用する話ですが、Terraformには公式サイトに [Style Guide](https://developer.hashicorp.com/terraform/language/style) というものがあって、それを参考にしていただくと便利です。Terraformのディレクトリ構成をどうしようかと悩む人が結構いると思いますが、基本的にStyle Guideを参考にしながらチームのメンバーと協議して、これがいいんじゃないかというのを選択していけばよいと思います。

例えば、開発環境と本番環境という2種類の環境がある場合、下図のような感じでdevとprdというディレクトリがあり、その中にbackend.tf, main.tf, variables.tfという3つのtfファイルをそれぞれ作成して、この他にmodulesディレクトリがあるというような、こんな構成を取ることができます。これは一応、Style Guideに沿ったディレクトリ構成です。

```
.
├── dev
│   ├── backend.tf
│   ├── main.tf
│   └── variables.tf
├── modules
├── prd
│   ├── backend.tf
│   ├── main.tf
│   └── variables.tf
```

## チームに合ったスタイルを

ここで、Style Guideをすべて真似するという方もいると思います。そういうやり方でもいいですが、少なくとも意識してほしいのは、チームに合ったスタイルを選択すればOKということで、すべてを真似する必要はないと思っています。

これはどういうことかというと、私がさくらに入って最初に関わったのがオブジェクトストレージのプロジェクトでした。オブジェクトストレージでは下図のような感じでディレクトリが作られていました。

```
└── providers
    └── sakuracloud
        └── storage-api
            ├── common
            │   ├── main.tf
            ├── extra_resources
            │   └── network
            │       └─ main.tf
            ├── ws.dev
            │   ├── common.tf -> ../common/main.tf
```

providersの下にsakuracloud、さらにその下にstorage-apiという階層があり、さらにその下にもディレクトリがあると、こんな感じで結構階層が深かったんですね。

当時はかなり熟練した専任のエンジニアがこのTerraformを管理していたので運用できていたのですが、そのエンジニアが退職し、その後を僕らが引き継ぎました。しかし、結構ディレクトリが深かったりして、なかなか運用がつらかったです。そういうわけで、体制が変わったら、新しい体制に合ったスタイルを選択した方がよいということを言いたいです。

## モジュールのディレクトリ構成も同様

モジュールのディレクトリ構成についても同様です。こちらもオブジェクトストレージを例にしますが、従来は下図のような構成になっていました。

```
├── modules
│   └── sakuracloud
│       ├── archive
│       │   ├── public
│       │   │   ├── main.tf
│       │   │   ├── output.tf
│       │   │   └── versions.tf -> ../../common/versions.tf
│       ├── common
│       ├── compute
│       │   ├── k8s-node
│       │   │   ├── main.tf
│       │   │   ├── output.tf
│       │   │   └── versions.tf -> ../../common/versions.tf
│       │   ├── remote-storage
│       │   │   ├── main.tf
│       │   │   ├── note_additional_disk.sh
│       │   │   └── versions.tf -> ../../common/versions.tf
```

上記の例ではmodulesの下にsakuracloudがあります。これはsakuracloud以外のところでもモジュールを使いたいということでこうなっていました。そしてsakuracloudの下にリソースが多数並んでいますが、これも結構階層が深いですよね。

そこで、ガンマチームで新規にサービス開発をする際にディレクトリ構成を変更し、モジュールをコンポーネント単位で分けてしまって、それより下のリソースをグループ化せずに、リソースをすべて同一の階層に置いて管理することにしました(下図参照)。

```
├── modules
│   ├── controller-server
│   │   ├── archive.tf
│   │   ├── ...
│   │   ├── locals.tf.tf
│   │   ├── providers.tf
│   │   ├── terraform.tf
│   │   └── variables.tf
│   ├── general
│   └── hosting-server
```

最初に紹介したオブジェクトストレージのディレクトリ構成は、Style Guideには則ってはいるんですよね。リソースごとにディレクトリを作り、その下にmain.tfやoutput.tfなどを配置していくというのはまさにStyle Guide通りなのですが、これがガンマチームには合わなかったので、チームに合うように、そして運用可能なように、ディレクトリ構成を選択していくのがよいと思います。

## サーバログイン設定

次に、さくらのクラウドでTerraformをどんどん使っていくようになると、当然ながら多くのサーバを作ることになると思いますが、例えばAWSやGCPなどのメガクラウドだと、パスワードログインってあまりやらないと思うんですよね。さくらのクラウドにはまだパスワードログインの機能がありますが、他のメガクラウドと同様に、SSHの鍵認証によるログインももちろん設定できます。

この設定をするためのパラメータとして重要なのが、下図に掲げる設定ファイルにおけるdisable\_pw\_authとssh\_key\_idsです。この2つを設定すると、SSHの鍵認証によるログインができるようになり、パスワードログインはできなくなるので、ログイン周りのセキュリティが強化されます。

```
% cat modules/controller-server/worker.tf
…
resource "sakuracloud_server" "worker_server" {
name   = "worker-${format("%03d", count.index + 1)}.${local.dns_zone}"
…
  disk_edit_parameter {
    hostname        = "worker-${format("%03d", count.index + 1)}"
    password        = local.user_password
    disable_pw_auth = true # Here
    ssh_key_ids     = sakuracloud_ssh_key.ssh_keys.*.id # Here
  }
```

では、さくらのクラウドではSSHの公開鍵はどうやって登録するかというと、sakuracloud\_ssh\_keyというリソースを宣言します。下図に例を示しますが、ここのresource行にて宣言しています。

```
% cat modules/controller-server/ssh.tf
resource "sakuracloud_ssh_key" "ssh_keys" {
  count      = 1
  name       = "${terraform.workspace}-${format("%03d", count.index + 1)}"
  public_key = var.ssh_public_key
}
```

これを宣言すると、下図のmain.tfファイルに記述されているssh\_public\_keyに記載された公開鍵のファイルをアップロードします。これで公開鍵が登録されるので、サーバに鍵認証でログインしてセットアップできるようになります。

```
% cat dev/main.tf
variable "ssh_public_key_file_name" {
  default = "id_rsa.pub"
}
module "controller_server" {
  source             = "../../modules/controller-server"
  ssh_public_key     = file(var.ssh_public_key_file_name)
}
```

## サーバ名を名前解決可能にする

他のクラウドプロバイダには、例えばAmazon Route 53 Resolverのように、DNSホスト名を自動で付与する機能があります。この機能を使うと、グローバルIPアドレスを指定してサーバにログインするのではなく、DNSホスト名でアクセスできるようになるのですが、これと同じような体験をさくらのクラウドでも行うことができます。

これはどうやって実現するかというと、仕組みは簡単で、サーバ名を名前解決可能なホスト名にしてあげればよいというだけの話です。具体的にどうするのかですが、下記にDNSに関するTerraformの設定ファイル例を示します。

```
% cat modules/controller-server/dns.tf
resource "sakuracloud_dns" "env_zone" {
  zone = local.dns_zone
  dynamic "record" {
    for_each = sakuracloud_server.worker_server
    iterator = server
    content {
      name  = "worker-${format("%03d", server.key + 1)}"
      type  = "A"
      value = server.value.ip_address
    }
  }
```

ここではcontentの中に、DNSに登録する情報を記載しています。nameがホスト名、typeはDNSのレコード種別(ここではAレコード)、valueがIPアドレスです。

こうすると何が便利かというと、例えばいったんTerraformでサーバを構築し、その後Ansibleでサーバの設定をするときに、 [sacloud-ansible-inventory](https://github.com/sakura-internet/sacloud-ansible-inventory) というダイナミックインベントリツールを使ってサーバ情報を取得することで、サーバに名前解決可能なDNS名を付けることができます。

これはどういうことかというと、sacloud-ansible-inventoryはダイナミックインベントリなので、サーバから情報を取ってきてインベントリファイルを作ってくれるわけですが、先ほど例示したようにDNSホスト名＝サーバ名にしておくと、下図の実行例のような感じで結果を取得することができます。これらのホスト名はDNSにも登録されているので、IPアドレスに変換しなくても、この状態のままAnsibleの実行ができるというわけです。

```
% ./workspaces/dev/blue/inventory.sh | tail -n 14 | head -n 8
  "controller": [
    "master-001.example.sakura.ad.jp",
    "master-002.example.sakura.ad.jp",
    "master-003.example.sakura.ad.jp",
    "worker-001.example.sakura.ad.jp",
    "worker-002.example.sakura.ad.jp",
    "worker-003.example.sakura.ad.jp"
  ],
```

あとは、上記のinventory.shを実行して、下記の例のようにplaybook.ymlの中で"hosts: controller"という行を記述すると、上記で列挙されたサーバすべてに対してAnsibleが実行できるようになっています。これはかなり便利なのでさくらのクラウドではおすすめの設定です。

```
% cat playbook.yml
…
- hosts: controller
  become: yes
  module_defaults:
    apt:
      cache_valid_time: 86400
  roles:
    - role: node_exporter
      tags:
        - node_exporter
```

## 使い回せるロールを作る

もうひとつのTipsとして、Ansibleのロールをその場で作ってもいいのですが、それを使い回せる形で作っておくと便利ですよいう話をします。

私が所属するガンマチームや、 [さくらのクラウドシェル](https://www.sakura.ad.jp/services/cloudshell/) のプロジェクトでは、ロールを別リポジトリで管理し、結構汎用的なロールを作っています。下図に示すようにgit submoduleでロールを呼び出して参照しています。こうしてあげると汎用的なロールをそのまま使えるのでかなり便利だと思います。

```
% cat .gitmodules
…
[submodule "ansible/roles/user"]
        path = ansible/roles/user
        url = https://github.com/Asya-kawai/ansible-role-user.git
        branch = main
[submodule "ansible/roles/ssh"]
        path = ansible/roles/ssh
        url = https://github.com/Asya-kawai/ansible-role-ssh.git
        branch = main
[submodule "ansible/roles/nginx"]
        path = ansible/roles/nginx
        url = https://github.com/Asya-kawai/ansible-role-nginx-with-naxsi.git
        branch = main
```

例えば、ユーザを作るとかグループを作るっていうのは皆さんもやると思いますが、それに加えて例えばシェル環境ですね。私はサーバにログインしたときに素のbashはあまり触りたくない人で、ゴリゴリにカスタマイズされたシェルを使いたいです。そこで、私が開発して公開している [userロール](https://github.com/Asya-kawai/ansible-role-user) というツールでは、便利なシェル環境を設定するために以下のタスクを定義しています。

```
- name: Install zsh
  package:
    name:
      - zsh
    state: present
  tags:
    - …

- name: Download zsh settings
  git:
    repo: "{{ zsh_settings_repo }}"
    dest: "{{ zsh_settings_dest }}"
    version: "{{ zsh_settings_version }}"
  when:
    - git_path is defined and (git_path.stdout | default('') | length > 0)
  tags:
    - …

- name: Copy zsh settings
  copy:
    remote_src: yes
    src: "{{ zsh_settings_dest.rstrip('/') }}/{{ item }}"
    dest: "{{ user.home_dir.rstrip('/') }}/{{ item }}"
    owner: "{{ user.name }}"
    group: "{{ user.name }}"
    mode: 0644
  with_items:
    - .zshrc
```

こんな感じでzshをダウンロードして、設定ファイルをコピーするようなことをやっておくと、かなりリッチなシェルを利用できます。これはとても便利なのでおすすめです。

また、よく使うミドルウェアも汎用的なロールとして定義しておくとかなり便利です。例えばnginxの設定も [nginxロール](https://github.com/Asya-kawai/ansible-role-nginx-with-naxsi) として定義し管理しています。私の場合は、nginxにnaxsiというWAFがあるのですが、それを設定したり、あとはLuaも稀に使うので設定しています。

設定例として、luaのインストールタスクを以下に示します。

```
…
      nginx : Download luajit2 from github      TAGS: [nginx, nginx-download-luajit2, webserver]
      nginx : Install luajit2   TAGS: [nginx, nginx-install-luajit2, webserver]
      nginx : Download lua resty core from github       TAGS: [nginx, nginx-download-lua-resty-core, webserver]
      nginx : Install lua resty core    TAGS: [nginx, nginx-install-lua-resty-core, webserver]
      nginx : Download lua resty lrucache       TAGS: [nginx, nginx-download-lua-resty-luacache, webserver]
      nginx : Install lua resty lrucache        TAGS: [nginx, nginx-install-lua-resty-luacache, webserver]
```

naxsiなどの動的モジュールのインストールタスクです。

```
…
      nginx : Download ngx_devel_kit from github        TAGS: [nginx, nginx-download-ngx-devel-kit, webserver]
      nginx : Download lua-nginx-module from github     TAGS: [nginx, nginx-install-ngx-devel-kit, webserver]
      nginx : Download naxsi from github        TAGS: [nginx, nginx-download-naxsi, webserver]
      nginx : Download ngx_http_geoip2_module from github       TAGS: [nginx, nginx-install-naxsi, webserver]
```

nginxの設定タスクです。

```
…
      nginx : Create nginx group        TAGS: [nginx, nginx-create-nginx-group, webserver]
      nginx : Create nginx user TAGS: [nginx, nginx-create-nginx-user, webserver]
      nginx : Create temp directory for nginx   TAGS: [nginx, nginx-create-temp-directory, webserver]
      nginx : Download nginx tar file   TAGS: [nginx, nginx-download-nginx-tar-file, webserver]
      nginx : Unarchive nginx tar file  TAGS: [nginx, nginx-unarchive-nginx-tar-file, webserver]
      nginx : Upload config.sh file     TAGS: [nginx, nginx-upload-config-sh-files, webserver]
      nginx : Run config.sh     TAGS: [nginx, nginx-run-config-sh, webserver]
      nginx : Copy naxsi_core.rules     TAGS: [nginx, nginxconf, webserver]
      nginx : Copy nginx basic conf file        TAGS: [nginx, nginxconf, webserver]
      nginx : Copy nginx conf files     TAGS: [nginx, nginxconf, webserver]
      nginx : Copy nginx service setting file   TAGS: [nginx, nginxconf, webserver]
      nginx : Check nginx service exists        TAGS: [nginx, nginxconf, webserver]
      nginx : Enable nginx service      TAGS: [nginx, nginxconf, webserver]
      nginx : Copy nginx logrotate setting file TAGS: [nginx, nginxconf, webserver]
```

このロールは、Debian系、Red Hat系、FreeBSDで動作するように設定しています。このように汎用的に作っておくと便利です。

## まとめ

というわけでまとめです。今回はTerraformやAnsibleの活用についてお話ししました。

Terraformについては、いろいろなディレクトリ構成やStyle Guideもありますが、それはすべてチームの成熟度や条件に合わせて選択していきましょう。それから、Terraformとsacloud-ansible-inventoryを使って、さくらのクラウドのインフラ構築や設定を効率的にやっていきましょう。あとは、Ansibleロールをうまく活用して、使い回して素早く設定できるようにしましょう。

私の発表は以上です。ありがとうございました。