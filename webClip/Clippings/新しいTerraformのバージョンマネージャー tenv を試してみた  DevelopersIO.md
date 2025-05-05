---
title: 新しいTerraformのバージョンマネージャー tenv を試してみた | DevelopersIO
source: https://dev.classmethod.jp/articles/try-tenv-terraform-version-manager/
author:
  - "[[Takuya Shibata]]"
published: 2025-05-08
created: 2025-05-06
description: 
tags:
  - Tech
  - Terraform
  - 思想系
read: false
---
しばたです。

私は普段Windows環境でTerraformを使っており、Terraformのバージョン管理には自作ツールを使っていました。

つい先日新しいバージョンマネージャーである [tenv](https://tofuutils.github.io/tenv/) というツールがあることを知ったので試してみることにしました。

## tfenvのつらみ

Terraformのバージョンマネージャーとしては `tfenv` が一番メジャーかと思います。

- [tfenv](https://github.com/tfutils/tfenv)

ただ、この `tfenv` はシェルスクリプト(Bashスクリプト)の集合体でありWindows環境ではGit Bashでのみ動作する状況でした。  
加えて2023年末ごろから開発停止状態になっていいます。

## 新しいバージョンマネージャー tenv

細かい経緯を正確に把握できていないのですが、今年に入りOpenTofuのコミュニティによりOpenTofu向けのtfenv派生である `tofuenv` が生まれ、

- [tofuutils / tofuenv](https://github.com/tofuutils/tofuenv)

続けて両者を統合して扱うために `tenv` が生まれた模様です。

- [tofuutils / tenv](https://github.com/tofuutils/tenv)

`tenv` はその実装をBashスクリプトからGo言語に変更しており [^1] 、本日時点ではさらに追加で [Terragrunt](https://terragrunt.gruntwork.io/) と [Atmos](https://atmos.tools/) のバージョン管理もできる様になっています。

実装がBashスクリプトからGo言語に変わったことでプラットフォーム別にバイナリが提供される形となりWindows環境でもネイティブに動作する様になっています。  
Windowsユーザーの私としては非常に嬉しい感じです。

加えて実装に [Cobra](https://cobra.dev/) を使っているため各種シェルでの補完機能も標準搭載されています。

### asdfとの違い

Windowsユーザーとしてはこれだけで十分 `tenv` を採用できるのですが、非Windowsユーザー向けに `asdf` との違いについて以下の様に表明されています。

- [Difference with asdf](https://tofuutils.github.io/tenv/#difference-with-asdf)

こちらによると、

- 複数ツールのバージョン管理という目的は両者同じだが、tenvはTerraform、OpenTofu、Terragrunt、AtmosといったHCLを扱う製品に特化している
- tenvはシェル環境や他のCLIツールに依存しないバイナリファイルで提供していく
- tenvパフォーマンスと各プラットフォームの互換性に優れている
- tenvはダウンロードしたバイナリのシグニチャチェック機能を持っている
- tenvはtfenvとtoufenvのコマンド構文に互換性がある

といった点でtenvの思想と優位性が語られています。  
決して「asdfを捨てろ」という話では無いので用途や環境に応じて好きな方を選ぶと良いでしょう。

## 試してみた

それでは早速試していきます。

### 検証環境

今回は私の開発環境である、

- 64bit版 Windows 11
- PowerShell 5.1 (および PowerShell 7.4.3、Nushell 0.95.0)

で試していきます。  
私は普段PowerShell 7とNushellを使っているのですが、今回はWindowsにデフォルトでインストール済みのPowerShell 5.1で作業を行い、tenvの言うプラットフォーム互換性を確かめたいと思います。

### インストール

Windows向けのパッケージマネージャとしてはChocolateyとNixがサポートされており、これらを使っている場合は以下のコマンドでインストール可能です。

```powershell
# Chocolatey を使っている場合
choco install tenv

# Nix を使っている場合
nix-env -i tenv
```

私はどちらも使っていないので、今回はGitHubのリリースバイナリを直接ダウンロードして使います。

- [Release](https://github.com/tofuutils/tenv/releases)

本日時点はVer.2.4.0が最新なので、64bit版のバイナリ(`tenv_v2.4.0_Windows_x86_64.zip`)をダウンロードし、任意のディレクトリに展開します。  
今回は `C:\tofuutils\tenv\` にツールを展開します。

PowerShell 5.1

```powershell
# インストール先ディレクトリを作成
$basePath = 'C:\tofuutils\tenv\'
mkdir $basePath

# 最新バージョンをダウンロード
$uri = 'https://github.com/tofuutils/tenv/releases/download/v2.4.0/tenv_v2.4.0_Windows_x86_64.zip'
Invoke-WebRequest -Uri $uri -OutFile './tenv_windows_x86_64.zip'
# 解凍
Expand-Archive -LiteralPath './tenv_windows_x86_64.zip' -DestinationPath $basePath

# ダウンロードしたzipを削除
Remove-Item -LiteralPath './tenv_windows_x86_64.zip'
```

最終的に下図の様に `C:\tofuutils\tenv\` にバイナリファイルが展開されていればOKです。

![try-tenv-terraform-version-manager-01](https://devio2024-media.developers.io/image/upload/v1720929512/2024/07/14/ybjyymo27zckma1cpzuy.png)

tenv本体である `tenv.exe` の他にShimとなる `terraform.exe`, `tofu.exe`, `terragrunt.exe`, `atmos.exe` もあるのが見て取れますね。

続けてこのディレクトリにPATHを通して各バイナリを名前だけで実行可能にすればインストールは完了です。

PowerShell 5.1

```powershell
# 非永続で良い場合
$env:Path += ';C:\tofuutils\tenv\'

# User環境変数に永続保存したい場合
[Environment]::SetEnvironmentVariable('PATH', [Environment]::GetEnvironmentVariable('PATH', 'User') + ";C:\tofuutils\tenv\", 'User')

# Machine環境変数に永続保存したい場合 : 要管理者権限
[Environment]::SetEnvironmentVariable('PATH', [Environment]::GetEnvironmentVariable('PATH', 'Machine') + ";C:\tofuutils\tenv\", 'Machine')
```

#### Optional: cosignを使ったシグニチャチェック

[cosign](https://github.com/sigstore/cosign) を使ってバイナリのシグニチャチェックをする場合は事前にインストールをしておく必要があります。  
Windows環境では `go install` コマンドを使いソースから直接バイナリを生成する手順となっています。

```powershell
go install github.com/sigstore/cosign/v2/cmd/cosign@latest
```

ただ、GitHubのリリースバイナリを直接ダウンロードしても期待した動作はしたのでこちらのほうが楽だと思います。

- [Releases](https://github.com/sigstore/cosign/releases)

PATHの通ったディレクトリに `cosign.exe` の名前で実行ファイルを保存してください。

### 動作確認

この状態で `tenv` コマンドを実行するとヘルプが表示されます。

![try-tenv-terraform-version-manager-02](https://devio2024-media.developers.io/image/upload/v1720929513/2024/07/14/dmbqoq36cbokuoehffgl.png)

基本的な使い方としては

```powershell
tenv [対象ツール] [対象コマンド]
```

となり、ツールの一覧は下表のとおりです。

| ツール | 指定コマンド |
| --- | --- |
| Terraform | tf |
| OpenTofu | tofu |
| Terragrunt | tg |
| Atmos | at |

Terraformを扱う場合は

```powershell
tenv tf [対象コマンド]
```

となります。  
対象コマンドの部分はtfenv互換で、

- tenv tf list
- tenv tf list-remote
- tenv tf install
- tenv tf use
- tenv tf uninstall

を基本としつつ独自の

- tenv tf constraint
- tenv tf detect
- tenv tf reset

といったコマンドが用意されています。  
より細かい点はヘルプを見て頂くと良いでしょう。

![try-tenv-terraform-version-manager-03](https://devio2024-media.developers.io/image/upload/v1720929513/2024/07/14/uzayl29aotvxxovwp9u5.png)

#### Terraformのインストールと利用

ここからはtfenvと同様に

- `tenv tf install` コマンドで指定バージョンのTerraformをインストール
- `tenv tf use` コマンドで利用するTerraformを選択

する形となります。  
なお、Terraformをインストールする前に `terraform` コマンドを実行してしまうとこんな感じで事前準備を促されます。

![try-tenv-terraform-version-manager-04](https://devio2024-media.developers.io/image/upload/v1720929512/2024/07/14/zskvrstikdi5orsprztd.png)

PowerShell 5.1

```powershell
# 最新バージョンTerraformのインストール
tenv tf install latest
```

![try-tenv-terraform-version-manager-05](https://devio2024-media.developers.io/image/upload/v1720929514/2024/07/14/lqf33lp7ny3k8s7k9alp.png)

PowerShell 5.1

```powershell
# インストール済みバージョンを確認
tenv tf list

# Ver.1.9.2を使用
tenv tf use 1.9.2

# 使用状況を確認
tenv tf list
```

![try-tenv-terraform-version-manager-06](https://devio2024-media.developers.io/image/upload/v1720929514/2024/07/14/jny9w4zcftxu34gyl518.png)

この状態で `terraform` コマンドを実行すると期待通りVer.1.9.2が起動してくれました。

PowerShell 5.1

```
terraform version
```

![try-tenv-terraform-version-manager-07](https://devio2024-media.developers.io/image/upload/v1720929512/2024/07/14/rpinoud0oc8psw0tuvzq.png)

実体となるバイナリはtfenv同様に `~\.tenv\Terraform\バージョン番号\` に保存されています。

![try-tenv-terraform-version-manager-08](https://devio2024-media.developers.io/image/upload/v1720929512/2024/07/14/hfo6hcmgt6kaw6m6okdb.png)

#### その他独自コマンド

tenv独自のコマンドについて簡単に解説しておきます。

- tenv tf constraint
	- デフォルトのバージョン制限(tenv独自機能)を設定します
- tenv tf detect
	- カレントディレクトリを基準として`.terraform-version` ファイルを探しバージョン制限の状態を確認します
- tenv tf reset
	- 利用バージョンを未指定の状態にします

## 補足: PowerShell 7やNushellでの利用

tenvはバイナリツールなのでWindows PowerShellだけでなくPowerShell 7やNushellでも普通に動作します。  
これは便利ですね。

![try-tenv-terraform-version-manager-09](https://devio2024-media.developers.io/image/upload/v1720929515/2024/07/14/mmeu0xrxhttjqeb1t1ek.png)

![try-tenv-terraform-version-manager-10](https://devio2024-media.developers.io/image/upload/v1720929515/2024/07/14/fcw3s4hnj4pepboatvce.png)

もちろんコマンドプロンプトでも動作します。

![try-tenv-terraform-version-manager-11](https://devio2024-media.developers.io/image/upload/v1720929515/2024/07/14/izcx2yri3si8mqpr91wh.png)

## 最後に

以上となります。

非常によいツールですね。  
これはWindowsユーザーにとってはマストでしょう。私も自作ツールからtenvに鞍替えする予定です。

本記事ではTerraformだけ試しましたが他のツールも同じく便利に扱えますので気になる方は是非試してみてください。

脚注

[^1]: GitHubのコミット履歴を見る限り、最初はシェルスクリプトでの統合を図ったものの、割と早期にGo言語へ切り替えていました