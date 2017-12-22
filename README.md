# bismark-VyOS
VyOS版Project BISmarkプローブ
# VyOS 版 BISmark プローブ導入手順書

## 1. はじめに

本書では、VyOS 版の BISmark プローブ環境のインストール方法について説明します。

## 2. 準備

プローブのインストール対象のハードウェアとして、以下のいずれかを準備します:

1. PC Engines APU2 (amd64 アーキテクチャの小型 PC)
2. ESXi 用 VM イメージ (仮想プローブを構築する場合)
    - CPU: x86 (64 ビット (amd64))
    - メモリ: 512 MB 以上
    - ストレージ: 2 GB 以上
    - NIC: 2 ポート以上
        - eth0: Internet 側 (DHCP サーバが存在すること)
        - eth1: LAN 側 (初期状態で 172.16.56.0/24 セグメント)

作業用端末に、以下のファイルをダウンロードします。

1. PC Engines APU2 を利用する場合
    1. vyos-1.1.7-amd64-apu2.iso (VyOS 本体)
        - VyOS 1.1.7 のインストール用 ISO イメージです
        - VyOS 1.1.7 の公式配布物をもとに、APU2 に対応するために以下の修正を加えています:
            - シリアルの通信速度を 115200 bps に変更
            - interface eth0: DHCP クライアントををブート時に有効化
            - interface eth1: IP アドレス 192.168.56.100/24 を割当
            - SSH サーバ: 有効化 (ポート 22)
            - ログイン用アカウント: vyos
            - ログイン用パスワード: vyos
            - NTP サーバ: ntp.nict.jp (タイムゾーン: Asia/Tokyo)
    2. vyperf-2017091500.tar.gz
        - VyOS 1.1.7 向けの BISmark プローブ環境一式のインストーラです
2. ESXi 用 VM を利用する場合
    1. vyos-1.1.7-amd64.iso (VyOS 本体)
        - VyOS 1.1.7 (Helium) の公式配布物を利用します
        - 公式サイト (配布元): <https://vyos.io/>
    2. vyperf-2017091500.tar.gz
        - VyOS 1.1.7 向けの BISmark プローブ環境一式のインストーラです

## 3. VyOS 1.1.7 インストール

### (1) VyOS インストーラを用いて起動しログイン

#### a) APU2 に導入する場合

1. ブート用 USB メモリ作成
    - VyOS 本体の ISO イメージより、ブート可能な USB メモリを作成します
    - Rufus <https://rufus.akeo.ie/> などを用いて作成して下さい
2. USB メモリを用いてブート
    - APU2 の背面 USB ポートに差し込み、電源アダプタを接続します
    - シリアルコンソールに接続する場合は 115200 bps に設定してください
    - ブート後、シリアルコンソール側に以下の表示が出てログイン可能な状態となります

```
[  289.876679] reboot: Restarting system
PC Engines APU BIOS build date: Sep  8 2014
Total memory 2048 MB
AMD G-T40E Processor
CPU MHz=1001
USB MSC blksize=512 sectors=7669824
USB MSC blksize=512 sectors=62333952
Press F10 key now for boot menu:
Select boot device:

1. USB MSC Drive Multiple Card  Reader 1.00
2. USB MSC Drive TOSHIBA TransMemory PMAP
3. Payload [setup]
4. Payload [memtest]

drive 0x000f2ab0: PCHS=0/0/0 translation=lba LCHS=951/128/63 s=7669824
drive 0x000f2a80: PCHS=0/0/0 translation=lba LCHS=1024/255/63 s=62333952
Booting from Hard Disk...

SYSLINUX 4.07 EDD 2013-07-25 Copyright (C) 1994-2013 H. Peter Anvin et al
[    3.463327] i8042: No controller found
[    4.854745] sd 6:0:0:0: [sda] No Caching mode page found
[    4.860082] sd 6:0:0:0: [sda] Assuming drive cache: write through
[    4.871452] sd 6:0:0:0: [sda] No Caching mode page found
[    4.876816] sd 6:0:0:0: [sda] Assuming drive cache: write through
[    4.894963] sd 6:0:0:0: [sda] No Caching mode page found
[    4.900293] sd 6:0:0:0: [sda] Assuming drive cache: write through

Welcome to VyOS - vyos ttyS0

vyos login: 
```

3. APU2 にログイン
    - シリアル経由でログインします (アカウント: vyos / vyos)
    - APU2 用イメージの場合、この段階で eth0 または eth1 経由で SSH を用いてログインすることも可能です

#### b) ESXi 用 VM に導入する場合

1. ブート用 ISO イメージを ESXi に配置
    - 公式サイトの vyos-1.1.7-amd64.iso を ESXi 側にアップロードします
2. ISO イメージを用いてブート
    - VyOS の ISO イメージを VM 側の D: ドライブに接続し、起動します
3. VM にログイン
    - シリアル経由でログインします (アカウント: vyos / vyos)

### (2) VyOS 1.1.7 のインストール

ログイン後 `install image` コマンドを発行し、以下のように VyOS 1.1.7 のインストール作業を行います。

```
vyos@vyos:~$ install image
Welcome to the VyOS install program.  This script
will walk you through the process of installing the
VyOS image to a local hard drive.
Would you like to continue? (Yes/No) [Yes]: yes                     ← yes と入力
Probing drives: OK
Looking for pre-existing RAID groups...none found.
You have two disk drives:
	sda 	3926 MB
	sdb 	31914 MB
Would you like to configure RAID-1 mirroring on them? (Yes/No) [Yes]:no ← noと入力
Ok.  Not configuring RAID-1.
The VyOS image will require a minimum 1000MB root.
Would you like me to try to partition a drive automatically
or would you rather partition it manually with parted?  If
you have already setup your partitions, you may skip this step

Partition (Auto/Parted/Skip) [Auto]:                                ← リターン

I found the following drives on your system:
 sda	3926MB             ← USB メモリ
 sdb	31914MB            ← 本体のストレージ


Install the image on? [sda]:sdb             ← 本体のストレージを指定

This will destroy all data on /dev/sdb.
Continue? (Yes/No) [No]: yes                                        ← yes と入力

How big of a root partition should I create? (1000MB - 31914MB) [31914]MB: 

Creating filesystem on /dev/sdb1: OK
Done!
Mounting /dev/sdb1...
What would you like to name this image? [1.1.7]:                    ← リターン
OK.  This image will be named: 1.1.7
Copying squashfs image...
Copying kernel and initrd images...
Done!
I found the following configuration files:
    /config/config.boot
    /opt/vyatta/etc/config.boot.default
Which one should I copy to sdb? [/config/config.boot]:              ← リターン

Copying /config/config.boot to sdb.
Enter password for administrator account
Enter password for user 'vyos':                         ← 初期パスワード設定
Retype password for user 'vyos':
I need to install the GRUB boot loader.
I found the following drives on your system:
 sda	3926MB
 sdb	31914MB


Which drive should GRUB modify the boot partition on? [sda]:sdb

Setting up grub: OK
Done!
```

完了後 `poweroff` コマンドを発行してプローブの電源を停止します。

```
vyos@vyos:~$ poweroff
Proceed with poweroff? (Yes/No) [No] yes

Broadcast message from root@vyos (ttyS0) (M[  262.366896] reboot: Power down
```

電源断の後に、USB メモリ (または VyOS 1.1.7 の ISO イメージの仮想 CD-ROM ドライブ) を本体より抜いて下さい。

その後、再度プローブを起動してログインします。

## 4. VyOS の初期設定

プローブをルータとして動作させるために、以下の追加設定を投入します。

```
configure
set nat source rule 10 outbound-interface eth0
set nat source rule 10 protocol all
set nat source rule 10 translation address masquerade
set policy route mssfix rule 10 protocol tcp
set policy route mssfix rule 10 set tcp-mss 1414
set policy route mssfix rule 10 tcp flags SYN
set interfaces ethernet eth1 policy route mssfix
delete interface ethernet eth1 address
set interfaces ethernet eth1 address 192.168.56.1/24
set service dhcp-server shared-network-name lan subnet 192.168.56.0/24 dns-server '192.168.56.1'
set service dhcp-server shared-network-name lan subnet 192.168.56.0/24 start 192.168.56.225 stop '192.168.56.254'
set service dhcp-server shared-network-name lan subnet 192.168.56.0/24 default-router 192.168.56.1
set service dns forwarding dhcp 'eth0'
set service dns forwarding listen-on 'eth1'
set interfaces ethernet eth0 address 'dhcp'
set interfaces ethernet eth0 description 'Global'
set interfaces ethernet eth1 address '192.168.56.1/24'
set interfaces ethernet eth1 description 'LAN'
set service ssh port 22
set system time-zone 'Asia/Tokyo'

commit
save
exit
```

設定変更後のサンプル config を以下に示します:

```
vyos@vyos:~$ show config commands
set interfaces ethernet eth0 address 'dhcp'
set interfaces ethernet eth0 description 'Global'
set interfaces ethernet eth0 hw-id '00:0d:b9:3e:a8:28'
set interfaces ethernet eth1 address '192.168.56.1/24'
set interfaces ethernet eth1 description 'LAN'
set interfaces ethernet eth1 hw-id '00:0d:b9:3e:a8:29'
set interfaces ethernet eth1 policy route 'mssfix'
set interfaces ethernet eth2 hw-id '00:0d:b9:3e:a8:2a'
set interfaces loopback 'lo'
set nat source rule 10 outbound-interface 'eth0'
set nat source rule 10 protocol 'all'
set nat source rule 10 translation address 'masquerade'
set policy route mssfix rule 10 protocol 'tcp'
set policy route mssfix rule 10 set tcp-mss '1414'
set policy route mssfix rule 10 tcp flags 'SYN'
set service dhcp-server shared-network-name lan subnet 192.168.56.0/24 default-router '192.168.56.1'
set service dhcp-server shared-network-name lan subnet 192.168.56.0/24 dns-server '192.168.56.1'
set service dhcp-server shared-network-name lan subnet 192.168.56.0/24 start 192.168.56.225 stop '192.168.56.254'
set service dns forwarding dhcp 'eth0'
set service dns forwarding listen-on 'eth1'
set service ssh port '22'
set system config-management commit-revisions '20'
set system console device ttyS0 speed '115200'
set system login user vyos authentication encrypted-password '$1$Gq0e/Cyd$NDqYXaB/J3rTTfIL2cQyB.'
set system login user vyos authentication plaintext-password ''
set system login user vyos level 'admin'
set system ntp server 'ntp.nict.jp'
set system package repository community components 'main'
set system package repository community distribution 'helium'
set system package repository community url 'http://packages.vyos.net/vyos'
set system syslog global facility all level 'notice'
set system syslog global facility protocols level 'debug'
set system time-zone 'Asia/Tokyo'
```

## 5. BISmark 環境導入

buildとinstallの方法は執筆中。

あとで追記します。




## 6. BISmark プローブ設定変更

プローブ側で、以下のファイルの設定変更を行います。

1. /usr/local/etc/bismark/bismark.conf
2. /usr/local/etc/bismark-data-transmit.conf 
3. /usr/local/etc/bismark-passive-iface.conf
4. /usr/local/etc/bismark-passive-whitelist.conf
5. /usr/local/lib/bismark/dns_targets.list
6. /usr/local/lib/bismark/ping_targets.list
7. /usr/local/lib/bismark/paristraceroute_targets.list

各ファイルの変更方法を以下に記載します。

#### (1) bismark.conf
management-server の IP アドレスを、実環境にあわせて変更します。

```
vyos@vyos:~$ sudo vi /usr/local/etc/bismark/bismark.conf
```

以下の行を、実環境の management-server の IP アドレスに変更して下さい:

```
SERVER=172.16.10.138
```

#### (2) bismark-data-transmit.conf
計測結果のログを送信する data-server (データ送信先サーバ) の URL を実環境にあわせて変更します。

```
vyos@vyos:~$ sudo vi /usr/local/etc/bismark-data-transmit.conf 
```

以下の行を、実環境の data-server (データ送信先サーバ) の URL に変更して下さい:

```
export DATA_TRANSMIT_ARGUMENTS="http://172.16.10.173:8443/upload/"
```

#### (3) /usr/local/etc/bismark-passive-iface.conf
パッシブ計測 (bismark-passive) の対象とするインターフェースを指定します。

```
vyos@vyos:~$ sudo vi /usr/local/etc/bismark-passive-iface.conf
```

LAN 側インターフェースを `eth1` 以外に変更した場合は、このファイルを修正してください。

例) LAN 側を `eth2` にする場合、ファイルの内容を以下のように修正する

```
eth2
```

#### (4) /usr/local/etc/bismark-passive-whitelist.conf
パッシブ計測の対象とするドメイン (ホワイトリスト) の一覧の設定ファイルです。
必要に応じて追加してください。

BISmark のデフォルト設定に加えて、以下のドメインが登録されています。

- Alexa Top Sites in Japan のうち上位 20 サイト
    - "Top Sites in Japan", <https://www.alexa.com/topsites/countries%3B0/JP>, 2017年7月14日確認
- 日本国内の主要 CDN リスト
    - 参考: 「国内 CDN シェア (2017年4月)」, <https://tech.jstream.jp/blog/cdn/cdn_share_apr2017/>, Jstream社調べ, 2017年7月14日確認
    - 登録ドメイン:
        - cloudflare.net (Cloudflare)
        - akamai.net (Akamai)
        - cloudfront.net (Amazon CloudFront)
        - panthercdn.com (CDNetworks)
        - incapdns.net (Incapsula)
        - llnwd.net (Limelight)
        - edgecastcdn.net (Edgecast)
        - fastly.net (Fastly)
        - durasite.net (Accelia)
        - idcfcloud.com (IDCF)
        - cas.iijgio.jp (IIJ)

例)

```
google.com
facebook.com
youtube.com
yahoo.com
amazon.com
  : (後略)
```

#### (5) /usr/local/lib/bismark/dns_targets.list
アクティブ計測 (DNS 順引き検索) の対象となる FQDN のリストを登録します。

デフォルトで以下が登録されています。

```
www.google.com
www.facebook.com
www.youtube.com
www.yahoo.com
www.live.com
www.wikipedia.org
www.blogger.com
www.msn.com
www.twitter.com
www.amazon.com
```

#### (6) /usr/local/lib/bismark/ping_targets.list
アクティブ計測 (ping) の対象となる IP アドレスを登録します。
デフォルトで以下が登録されています。

```
203.138.149.127 0 0
```

#### (7) /usr/local/lib/bismark/paristraceroute_targets.list
アクティブ計測 (paris-traceroute) の対象となる IP アドレスを登録します。
デフォルトで以下が登録されています。

```
203.138.149.127
```

## 7. リブート

設定完了後、VyOS プローブをリブートします。

```
vyos@vyos:~$ reboot
```

## 8. 動作確認

プローブ上で以下のプロセスが起動していることを確認します。

- bismark-data-transmit (ログデータ送信用)
- bismark-passive (パッシブ計測)
- dropbear (BISmark management-server からの SSH 接続用)

## 9. 運用・その他

### パッシブ計測について

#### パッシブ測定の設定ファイルを変更する場合
パッシブ測定の設定変更をする場合の手順を説明します。
はじめに、以下のファイルを修正します。

- /usr/local/etc/bismark-passive-iface.conf
- /usr/local/etc/bismark-passive-whitelist.conf

次に、`bismark-passive` プロセスを再起動します。これで設定変更が反映されます。

```
$ sudo service bismark-passive stop
$ sudo service bismark-passive start
```

#### パッシブ測定を停止する
パッシブ計測を停止し、リブート後も停止したままにする場合は、以下のようにコマンドを投入して下さい。

```
$ sudo service bismark-passive stop
$ sudo update-rc.d bismark-passive disable
```

#### パッシブ測定を開始(再開)する
パッシブ計測を開始(再開)する場合は、以下のコマンドを入力します。

```
$ sudo service bismark-passive start
$ sudo update-rc.d bismark-passive enable
```


### アクティブ計測について

#### アクティブ測定の設定ファイルを変更する場合
アクティブ計測については、設定ファイル修正後、次回の計測タイミングより新しい設定が反映されます。
プロセスの再起動などは必要ありません。

#### アクティブ測定を停止する
ファイル `/etc/cron.d/cron-bismark-active` を編集し、行頭にコメント記号 (`#`) を追加します。

例:

```
# * * * * * root /usr/local/bin/bismark-measure-wrapper
```

#### アクティブ計測を開始(再開)する
ファイル `/etc/cron.d/cron-bismark-active` を編集し、コメント記号 (`#`) を削除します。

```
* * * * * root /usr/local/bin/bismark-measure-wrapper
```

### その他の設定ファイルの変更 (bismark-data-transmit 等)
bismark-data-transmit プロセスを再起動することで設定変更が反映されます。

```
$ sudo service bismark-data-transmit stop
$ sudo service bismark-data-transmit start
