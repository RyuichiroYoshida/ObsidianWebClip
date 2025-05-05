---
title: Docker / Docker Desktop / Rancher Desktop って何が違うの？
source: https://qiita.com/megmogmog1965/items/dc73525f2c76170405c1
author:
  - "[[Qiita]]"
published: 2022-07-09
created: 2025-05-06
description: 今更の記事ですが、ざっくり説明用に書きました。背景近年の開発では、各自のローカル PC 上での開発として docker (docker-compose) を使う事が多くなりました。例えば、最近…
tags:
  - Docker
  - Tech
  - バックエンド
read: false
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3551653)

この記事は最終更新日から1年以上が経過しています。

今更の記事ですが、ざっくり説明用に書きました。

## 背景

近年の開発では、各自のローカル PC 上での開発として [docker](https://www.docker.com/) ([docker-compose](https://docs.docker.com/compose/)) を使う事が多くなりました。

例えば、最近の Web Application の殆どは以下の３つを使って動きます。

- RDB (e.g. [mysql](https://www.mysql.com/))
- in-memory data store (e.g. [redis](https://redis.io/))
- Object Storage (e.g. [Amazon S3](https://aws.amazon.com/s3), [MinIO](https://min.io/))

昔は開発者 wiki や `README.md` に上記の構築方法が書かれていて、開発者みんなが頑張って自前でローカルマシン上に構築をしていました。

> もしくは VMWare イメージを配布する〜、Vagrant 等が試みられていた。

最近では、もうこれ一発で **開発者の環境差分の影響もなく** (※ M1 Mac.. 😭) 簡単に構築できますね。素晴らしい時代です。

```shell
docker-compose up -d
```

とは言え、ブラックボックス化され初期設計した人以外は *「手順通りにやったら動いたけど、仕組みが全然わからない！」* になっている事が多い様に思えます。

## 目的

と言うことで、本書では以下の３つを説明します。

1. そもそも Docker って何？
2. [Docker Desktop](https://www.docker.com/products/docker-desktop/) for Mac / Windows って、素の [Docker](https://www.docker.com/) とどう違うの？
3. [Docker Desktop](https://www.docker.com/products/docker-desktop/) 有料になっちゃった。 [Rancher Desktop](https://rancherdesktop.io/) がいいらしいけど、ナニソレ？

## コンテナ仮想化

### そもそもコンテナって？ (VM とは違うよ)

良くVM (仮想マシン) と混同されますが、そもそも大きく違うのは次の点です。

- VM (仮想マシン) は **OS** を仮想化する
- コンテナ技術は **プロセス** を仮想化する

例えば VM (仮想マシン) 以下の様に図示されます。

> AWS とかは Hypervisor 型で、 [Xen](https://xenproject.org/) を使ってます。

[![dac1699e-1d81-bf57-e24a-59cfb228424a.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/60131/0424af70-fe5a-3b3a-5510-65c0a7d0bec7.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F60131%2F0424af70-fe5a-3b3a-5510-65c0a7d0bec7.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=f2c390ab0200c348a71809296533b9be)

一方コンテナ技術はこうです。  
実は Linux OS 上で直接プロセスを起動しているだけで、普通のプロセスと大きく変わりはありません。

[![a8bcbaec-26c6-9843-77aa-7e2d0af2ab11.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/60131/dc6bbfe5-3326-7622-39ce-60bd8ff2109c.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F60131%2Fdc6bbfe5-3326-7622-39ce-60bd8ff2109c.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=12638bdd4c208c3915e1cd87aba0160c)

つまり、コンテナ仮想化とは Linux 上で **「プロセスを動かす空間を分離する」** 事ですが、それが具体的にどういう事か見ていきましょう。

### プロセス空間分離

AWS 上に立てた Amazon Linux 2 インスタンス上で実際に確認してみます。  
以下のコマンドで [nginx](https://www.nginx.co.jp/) コンテナを起動しましょう。

```shell
docker run -d --name nginx -p 8080:80 nginx:1.21-alpine
```

ブラウザで [http://localhost:8080](http://localhost:8080/) で、Docker Image から起動した [nginx](https://www.nginx.co.jp/) にアクセスできます。

[![fbe4ea27-cf71-995f-f1d6-25eac1aacd75.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/60131/5bbec629-ee46-e218-b6e5-122e497eb427.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F60131%2F5bbec629-ee46-e218-b6e5-122e497eb427.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=7a1cbc1c324d2a1ca1dd99cbbe7b5176)

以降の説明では、実際にコンテナの中に入って実例で説明をします。以下のコマンドで入れます。

```shell
docker exec -it nginx sh
```

#### PID 名前空間

「PID 名前空間」とは `ps aux` した時に見えるプロセスの一覧の事です。

実際にホスト OS 上で `ps aux` してみます。  
実行されている全プロセスが見れますが、 **実はコンテナとして起動したプロセスも見えています。**

```shell
[ec2-user@ip-172-30-1-40 ~]$ ps auxf
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.2 123716  5608 ?        Ss   06:46   0:03 /usr/lib/systemd/systemd --switched-root --system --deserialize 21
root         2  0.0  0.0      0     0 ?        S    06:46   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        I<   06:46   0:00 [rcu_gp]
...

root      3757  0.0  4.1 1446720 82940 ?       Ssl  06:48   0:05 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock 
root      2531  0.0  0.4 711720  9136 ?        Sl   09:21   0:00 /usr/bin/containerd-shim-runc-v2 -namespace moby -id ff6516b31969bfa33b
root      2554  0.0  0.2   6280  4516 ?        Ss   09:21   0:00  \_ nginx: master process nginx -g daemon off;
101       2622  0.0  0.0   6736  1776 ?        S    09:21   0:00  |   \_ nginx: worker process
root      2742  0.0  0.0   1692  1180 pts/0    Ss+  09:26   0:00  \_ sh
...
```

> PID `2554`, `2622` は、コンテナとして起動した `nginx` プロセス達.

一方、コンテナ内部で `ps aux` をすると、３つのプロセスしか見えません。  
ホスト OS やその他コンテナとは PID 空間が隔離されており、 **外部プロセスに影響を与えない** 訳です。

```shell
/ # ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 nginx: master process nginx -g daemon off;
   32 nginx     0:00 nginx: worker process
   39 root      0:00 sh
```

コンテナ中で稼働するプロセスは基本的に一つで、それは [Dockerfile で ENTRYPOINT, CMD に指定したコマンド](https://github.com/nginxinc/docker-nginx/blob/1.21.6/Dockerfile-alpine.template#L123) です。  
これは必ず PID `1` になります。

上記のその他プロセス (`32`, `39`) は次の理由で生成されました。

| PID | Process name | プロセス起動の経緯 |
| --- | --- | --- |
| `1` | `nginx` | [Dockerfile で ENTRYPOINT, CMD に指定したコマンド](https://github.com/nginxinc/docker-nginx/blob/1.21.6/Dockerfile-alpine.template#L123) |
| `32` | `nginx` | `nginx` (PID `1`) が起動時に生成した子プロセスです |
| `39` | `sh` | コンテナに入る際のおまじない `docker exec -it nginx sh` は、この通りコンテナ内に `sh` プロセスを起動する為の物でした |

> コンテナ仮想化は VM (OS 仮想化) とは異なり、例えば以下の様な Linux OS 起動時の処理は一切行われません。
> 
> - 起動スクリプトの実行 (e.g. `/etc/init.d`)
> - サービスの起動 (e.g. `crond`, `sshd`)

#### ネットワーク名前空間

「ネットワーク名前空間」とは `ifconfig` した時に見えるネットワーク構成 (NIC) の事です。

実際にホスト OS 上で `ifconfig` してみます。  
このマシンに接続されているネットワーク・インターフェース (LAN コネクタや Wi-fi の事) が一覧表示されます。

| NIC 名 | アドレス (例) | 説明 |
| --- | --- | --- |
| docker0 | `172.17.0.1` | このホストマシン上に Docker コンテナが仮想ネットワークがあります。その入口 (Bridge) です。      コンテナの IP アドレス範囲は `172.17.0.2` 〜 `172.17.0.255` になります。 |
| eth0 | `172.30.1.40` | インターネット LAN 接続の出入り口です。 AWS VPC 上で IP アドレス `172.30.1.40` が割り振られています。 |
| lo | `127.0.0.1` | ループバックアドレスと言い、所謂 `localhost` (`127.0.0.1`) アドレスです。 |
| veth | \-- | [※ 省略](https://qiita.com/megmogmog1965/items/f5682de2068a79d196a2) |

```shell
[ec2-user@ip-172-30-1-40 ~]$ ifconfig 
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ...

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 9001
        inet 172.30.1.40  netmask 255.255.255.0  broadcast 172.30.1.255
        ...

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        ...

veth3f835b9: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        ...

veth902e697: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        ...
```

一方、コンテナ内部で `ifconfig` をすると、２つのネットワーク・インターフェースしか見えません。  
このコンテナはホストマシン上の Docker 仮想ネットワークに所属し、IP アドレス `172.17.0.2` でアクセスが可能です。

| NIC 名 | アドレス (例) | 説明 |
| --- | --- | --- |
| eth0 | `172.17.0.2` | インターネット LAN 接続の出入り口です。ホストマシンの Docker 仮想ネットワーク上で IP アドレス `172.17.0.2` が割り振られています。 |
| lo | `127.0.0.1` | ループバックアドレスと言い、所謂 `localhost` (`127.0.0.1`) アドレスです。 |

```shell
/ # ifconfig 
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:03  
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          ...

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          ...
```

[過去に書いた記事](https://qiita.com/megmogmog1965/items/f5682de2068a79d196a2) からネットワーク図を転載します。

[![53556ca8-5db5-5eef-0275-bd5a7bd1c2be.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/60131/4fb5d875-3fee-2704-986a-af970ec67019.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F60131%2F4fb5d875-3fee-2704-986a-af970ec67019.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=c894ec1ea75b2326009b1ca3c7ce9dba)

#### マウント名前空間 (Docker Image の事)

「マウント名前空間」とは `ls` した時に見えるファイルシステム (ファイルやディレクトリ) の事です。

実際にホスト OS 上で `ls /` してみます。  
このマシンのルートディレクトリ `/` にあるファイル・ディレクトリが一覧されました。

```shell
[ec2-user@ip-172-30-1-40 ~]$ ls /
bin  boot  dev  etc  home  lib  lib64  local  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

一方、コンテナ内部で `ls /` をすると、全く別のファイル・ディレクトリが一覧されています。

```shell
/ # ls /
bin                   etc                   mnt                   run                   tmp
dev                   home                  opt                   sbin                  usr
docker-entrypoint.d   lib                   proc                  srv                   var
docker-entrypoint.sh  media                 root                  sys
```

このディレクトリ構造は皆さんご存知の **Docker Image そのもの** です。

- docker hub → [nginx:1.21-alpine](https://hub.docker.com/layers/nginx/library/nginx/1.21-alpine/images/sha256-529db430e042ecef071f2e88267cee6da18f8ab44d66a0c44348886fdc2e60fc?context=explore)

例えば、 [nginx:1.21-alpine](https://hub.docker.com/layers/nginx/library/nginx/1.21-alpine/images/sha256-529db430e042ecef071f2e88267cee6da18f8ab44d66a0c44348886fdc2e60fc?context=explore) Image の元となった [Dockerfile](https://github.com/nginxinc/docker-nginx/blob/d039609e3a537df4e15a454fdb5a004d519e9a11/mainline/alpine/Dockerfile) は以下の様になっています。

- ベース Image に [Alpine Linux](https://www.alpinelinux.org/) を用いている。コンテナ中のファイル・ディレクトリは、ほぼここから来てる
- Image のビルド過程で `nginx` をインストールしている
- この Image からコンテナを起動すると `nginx -g daemon off;` が実行され最初のプロセス (PID `1`) になる

```dockerfile
FROM alpine:3.15

RUN set -x \
    ... \
    && apk add -X "https://nginx.org/packages/mainline/alpine/v$(egrep -o '^[0-9]+\.[0-9]+' /etc/alpine-release)/main" --no-cache $nginxPackages \

...

CMD ["nginx", "-g", "daemon off;"]
```

> [https://github.com/nginxinc/docker-nginx/blob/d039609e3a537df4e15a454fdb5a004d519e9a11/mainline/alpine/Dockerfile](https://github.com/nginxinc/docker-nginx/blob/d039609e3a537df4e15a454fdb5a004d519e9a11/mainline/alpine/Dockerfile)

この様に、コンテナ中で `ls` 等をすると隔離されたコンテナ特有のファイルシステム (ファイルやディレクトリ) が見えます。

これらコンテナのファイル・ディレクトリ実体はホストマシン上の下記ディレクトリにあります。

```shell
/var/lib/docker/overlay2/
```

## Docker

### Linux コンテナ技術と Docker

実は、これまで話してきた事の殆どは名前空間を分離する Linux 標準の機能を使って実現されています。  
[Docker](https://www.docker.com/) は Linux のコンテナ技術を API 化し、万人に使いやすくしたソフトウェアです。

### Docker クライアントと dockerd

Amazon Linux 2 等で `yum install -y docker` して入る Docker のコマンド・サービスは次の２種類に分かれています。

| # | サービス | 説明 |
| --- | --- | --- |
| 1 | Docker クライアント | 皆さん馴染み深い `docker` コマンドは、 `dockerd` に [HTTP 通信で指示を出す為の CLI](https://docs.docker.com/engine/api/v1.24/) です。 |
| 2 | dockerd (moby) | Docker サービスの本体で、コンテナの管理等をしている [Engine API (HTTP)](https://docs.docker.com/engine/api/v1.24/) を持ったデーモンプロセスです。 |

前述の様に `docker run -d nginx:1.21-alpine` をすると、 `dockerd` 配下のプロセスとして `nginx` は起動します。

```shell
[ec2-user@ip-172-30-1-40 ~]$ ps auxf
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
...
root      3566  0.0  2.1 1367168 43612 ?       Ssl  06:47   0:04 /usr/bin/containerd
root      3757  0.0  4.1 1446720 82940 ?       Ssl  06:48   0:05 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock 
root      2531  0.0  0.4 711720  9136 ?        Sl   09:21   0:00 /usr/bin/containerd-shim-runc-v2 -namespace moby -id ff6516b31969bfa33b
root      2554  0.0  0.2   6280  4516 ?        Ss   09:21   0:00  \_ nginx: master process nginx -g daemon off;
101       2622  0.0  0.0   6736  1776 ?        S    09:21   0:00  |   \_ nginx: worker process
root      2742  0.0  0.0   1692  1180 pts/0    Ss+  09:26   0:00  \_ sh
```

Docker クライアント (`docker` コマンド) は `dockerd` に対し HTTP 通信による API 呼び出しをしています。

その際に Docker クライアントは、TCP による通信ではなく [/var/run/docker.sock](https://docs.docker.com/engine/api/v1.24/) ソケットファイルによる [UNIX ドメインソケット](https://ja.wikipedia.org/wiki/UNIX%E3%83%89%E3%83%A1%E3%82%A4%E3%83%B3%E3%82%BD%E3%82%B1%E3%83%83%E3%83%88) 通信を行います。

[![6153dbac-56d5-49c4-d482-b726b63ba233.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/60131/af5cebeb-222d-85db-2f91-73d7898c2a45.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F60131%2Faf5cebeb-222d-85db-2f91-73d7898c2a45.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=17431985014e493dd172ad4958560c1e)

例えば Docker サービスがまだ起動してない時に `docker` コマンドを実行すると良く次のエラーに見舞われますが、これは [UNIX ドメインソケット](https://ja.wikipedia.org/wiki/UNIX%E3%83%89%E3%83%A1%E3%82%A4%E3%83%B3%E3%82%BD%E3%82%B1%E3%83%83%E3%83%88) 接続が確立できない為に発生しています。

```shell
$ docker ps
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

## Docker Desktop for Mac / Windows

前述の通り、 [Docker](https://www.docker.com/) は Linux のコンテナ技術を使って実現されています。  
その為、Mac / Windows 上では `dockerd` サービスは動きません。

そこで [Docker Desktop](https://www.docker.com/products/docker-desktop/) は `dockerd` サービスを動かす基盤として Linux VM (仮想マシン) を導入しています。

しかし、当然ながら Linux VM (仮想マシン) を用いた場合は、 **VM 内に入らなければ `docker` コマンドや、立ち上がったコンテナ (プロセス) にアクセスできません。**

そこで [Docker Desktop](https://www.docker.com/products/docker-desktop/) では VM に入らなくても大丈夫な様に、下図に示す仕組みが導入されています。

[![Screen Shot 2022-06-14 at 13.55.40.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/60131/654741ab-6255-8124-a7f5-a39c65d6411c.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F60131%2F654741ab-6255-8124-a7f5-a39c65d6411c.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a550d3c46c90afcab95ed242297dc9a3)

### /var/run/docker.sock がマウントされている

[Docker Desktop](https://www.docker.com/products/docker-desktop/) では、ホストマシン上の Docker クライアント (`docker` コマンド) が、Linux VM 中の `dockerd` と通信出来るように、ホストマシン上の `/var/run/docker.sock` が用意されています。

```shell
$ ls -alht /var/run/docker.sock 
lrwxr-xr-x  1 root  daemon    38B Jun  9 15:13 /var/run/docker.sock -> /Users/{user-name}/.rd/docker.sock
```

その為、ホストマシン上での `docker ps` の様なコマンドは、Linux VM 中の `dockerd` へ伝達されます。

### コンテナのローカルポートが Port-Forward されている

前述の様に、コンテナを起動する際に `-p 8080:80` option を指定する事で、ローカルポート `:8080` をコンテナが `:80` で LISTEN できます。

```shell
docker run -d --name nginx -p 8080:80 nginx:1.21-alpine
```

ただし、この場合は **Linux VM (仮想マシン) のローカルポート `:8080` を LISTEN している** 事になるので、ホストマシンのブラウザで次の URL にアクセスしても、通常は HTTP 接続が確立できない筈です。

- [http://localhost:8080](http://localhost:8080/)

[![d2b924bc-34d8-8e61-a2fe-54c269669e99.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/60131/a2796657-bb82-144a-da30-1f669fcf115e.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F60131%2Fa2796657-bb82-144a-da30-1f669fcf115e.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=9937bfc4141f44d94f704a0f304aef52)

> ↑ 本来こうなる筈だが...?

しかし実際には、 [Docker Desktop](https://www.docker.com/products/docker-desktop/) ではホストマシンから直接アクセスできます。  
Linux VM (仮想マシン) 側のコンテナのローカルポートが、ホストマシン側に Port-Forwarding されている為です。

[![Screen Shot 2022-06-14 at 13.55.40.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/60131/654741ab-6255-8124-a7f5-a39c65d6411c.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F60131%2F654741ab-6255-8124-a7f5-a39c65d6411c.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a550d3c46c90afcab95ed242297dc9a3)

## Rancher Desktop

残念ながら [Docker Desktop](https://www.docker.com/products/docker-desktop/) は 2022年 2月 に有料化されました。

- [https://www.docker.com/pricing/](https://www.docker.com/pricing/)

引っ越し先として 2022年 6月 時点でオススメなのが、 [Rancher Desktop](https://rancherdesktop.io/) です。

- [https://rancherdesktop.io/](https://rancherdesktop.io/)

[Rancher Desktop](https://rancherdesktop.io/) は、ほぼ [Docker Desktop](https://www.docker.com/products/docker-desktop/) と同じ感覚で使え、Docker の操作はホストマシン上で完結できます。

### CPU Emulation 対応 (M1 Mac arm64 問題)

[Rancher Desktop](https://rancherdesktop.io/) は [Docker Desktop](https://www.docker.com/products/docker-desktop/) と同様に、CPU Emulation に対応しており `arm64` アーキテクチャマシン (※ つまり M1 Mac の事) 上で、 `x86_64` (Intel CPU) の Docker Image を実行できます！

開発環境を [docker-compose](https://docs.docker.com/compose/) で構成しているケースは多いと思いますが、Docker Hub から Image を取得する際は、自動的に利用しているホストマシンの CPU アーキテクチャに合わせた Docker Image が `docker pull` されます。

[![e4641c83-82e0-f46f-18df-21204d3b527e.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/60131/10a80ef5-ca83-458a-52a9-e71b10b453b5.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F60131%2F10a80ef5-ca83-458a-52a9-e71b10b453b5.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=f7c60147a3ef080631c3598475b33de0)

その為 [mysql](https://hub.docker.com/layers/mysql/library/mysql/5.7.38/images/sha256-06c614dfc9720ccc0177acf700d0e0794f0efe3a032e78ea5318c30886ce62c1?context=explore) の様な OS/ARCH が `linux/amd64` しか無い Docker Image を使っている場合、M1 Mac では対応するイメージが無い為、動作しなくなります。

[![7a6347bc-279f-48da-8df2-44527d0c78cb.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/60131/d25509ab-45b8-45c3-fefc-5dba7a7442d4.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F60131%2Fd25509ab-45b8-45c3-fefc-5dba7a7442d4.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a30657c680485a7649c27d079f638e45)

しかし [Rancher Desktop](https://rancherdesktop.io/) では CPU Emulation に対応している為、次の様に OS/ARCH を `linux/amd64` に固定して `docker pull` させる様にする事で、これまで通り動作させる事ができます。

```diff
version: '3.1'

services:

  db:
    image: mysql
+   platform: 'linux/amd64'
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: example
```

> とは言え、性能面は残念ながら劣化しますが...

[0](https://qiita.com/megmogmog1965/items/#comments)

[23](https://qiita.com/megmogmog1965/items/dc73525f2c76170405c1/likers)

37