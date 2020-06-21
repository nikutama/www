---
title:  "「サイバーセキュリティ レッドチーム実践ガイド」の「Chat Support Systems アプリケーション」仮想マシンをVirtualBoxで使う"
slug: "cybersecurity-virtualbox"
date:   2019-03-13 00:00:00 +0900
tags: 
  - security
  - virtualbox
notoc: true
---
**※ [はてなブログ](https://tatata.hatenablog.jp/entry/2019/03/13/225233)から引っ越しました（2019/10/21）**

「[サイバーセキュリティ レッドチーム実践ガイド | マイナビブックス](https://book.mynavi.jp/ec/products/detail/id=101584) – 第3章」で紹介されている「Chat Support Systems アプリケーション」の仮想マシンはVMware用であり、そのままではVirtualBoxで使えないため、解決方法を書いておく。

### 具体的な問題点

VirtualBoxではネットワークインターフェースが認識されないためIPアドレスが割り当てられない。ログインして設定変更すれば良いのだが、パスワードは公開されていない。


### 解決策

メールすればパスワードを教えてもらえるらしい[^1]が面倒なので、以下の手順で解決した。

#### 1. VirtulBoxの別の仮想マシンに仮想ディスクをアタッチする

別の仮想マシンのストレージ設定画面より、ディスクの追加アイコンをクリックする。

![ストレージ設定画面](/img/20190313/01.png)

「既存のディスクを選択」をクリックする。

![既存のディスクを選択](/img/20190313/02.png)

「Chat Support Systems アプリケーション」の仮想ディスクを選択する。

![仮想ディスクを選択](/img/20190313/03.png)

#### 2. 別の仮想マシンを起動して、アタッチした仮想ディスクをマウントする

アタッチされたディスクを確認する。/dev/sdbにアタッチされたことがわかる。

```bash
 dmesg | grep disk
[    3.067551] sd 2:0:0:0: [sdb] Attached SCSI disk
[    3.119617] sd 1:0:0:0: [sda] Attached SCSI disk
```

仮想ディスクのパーティションを確認する。/dev/sdb5がルートパーテションであると思われる。

```bash
# fdisk -l /dev/sdb
Disk /dev/sdb: 20 GiB, 21474836480 bytes, 41943040 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x5fdb1238

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sdb1  *       2048   999423   997376  487M 83 Linux
/dev/sdb2       1001470 41940991 40939522 19.5G  5 Extended
/dev/sdb5       1001472 41940991 40939520 19.5G 8e Linux LVM
```

ファイルマネージャー（この例ではFilesを使用）を起動してマウントする。

![マウント](/img/20190313/04.png)

/media/root/f305128a-5d16-4dd0-a8be-4f83f63f622fにマウントされたことがわかる。

```bash
# df
Filesystem                        1K-blocks     Used Available Use% Mounted on
udev                                1000480        0   1000480   0% /dev
tmpfs                                204332     7012    197320   4% /run
/dev/sda1                          38958432 22889684  14060072  62% /
tmpfs                               1021644        8   1021636   1% /dev/shm
tmpfs                                  5120        0      5120   0% /run/lock
tmpfs                               1021644        0   1021644   0% /sys/fs/cgroup
tmpfs                                204328       16    204312   1% /run/user/130
tmpfs                                204328       36    204292   1% /run/user/0
/dev/mapper/LethalNodeJS--vg-root  18982780  2118044  15877396  12% /media/root/f305128a-5d16-4dd0-a8be-4f83f63f622f
```

#### 3. ネットワーク設定を変更する

ネットワークの設定ファイルを確認するとens33になっているので、enp0s3に書き換える。

```bash
# vi /media/root/f305128a-5d16-4dd0-a8be-4f83f63f622f/etc/network/interfaces
```

変更前

```bash
# The primary network interface
auto ens33
iface ens33 inet dhcp
```

変更後

```bash
# The primary network interface
auto enp0s3
iface enp0s3 inet dhcp
```

OSを再起動するとIPアドレスが割り当てられたことがわかる。

![IPアドレスの確認](/img/20190313/05.png)

[^1]: https://github.com/cheetz/THP-ChatSupportSystem/issues/1
