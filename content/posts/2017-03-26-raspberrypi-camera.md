---
title:  "Raspberry Pi Zero Wにカメラをつけてみた"
slug: "raspberrypi-camera"
date:   2017-03-26 00:00:00 +0900
tags: 
  - camera
  - raspberrypi
---
**※ [はてなブログ](https://tatata.hatenablog.jp/entry/2017/03/26/220111)から引っ越しました（2019/10/21）**

Raspberry Pi Zero Wにカメラをつけて、cronで定期的に部屋の中を撮影してみました。

## Raspberry Pi Zero Wにカメラをつける

### 用意するもの

- Raspbianをセットアップ済みのRaspberry Pi Zero W (他のRaspberry PiでもOK)
- Raspberry Pi Camera V2
- 公式ケース(なくてもOK)
- Raspberry Pi Zero用のカメラコネクタ[^1]

### 組み立てる

コネクタを使ってRaspberry Piとカメラを接続しケースに収めます。公式ケースはカメラがきれいに収まるようになっています。

FRISKと並べてみました。FRISKより一回り程度大きなサイズです。

![FRISKとRaspberry Pi Zero](/img/20170326/01.jpg)

## カメラを有効化する

rasp-configを起動します。

```bash
$ sudo raspi-config
```

「5 Interfacing Options」を選択します。

![raspi-config menu](/img/20170326/02.png)

「P1 Camera」を選択します。

![raspi-config select intercace](/img/20170326/03.png)

「Yes」を選択します。

![raspi-config enable camera intercace](/img/20170326/04.png)

「Ok」を選択します。

![raspi-config OK](/img/20170326/05.png)

「Finish」を選択してraspi-configを終了します。

![raspi-config Finish](/img/20170326/06.png)

## Webサーバーを導入する

Webサーバーとしてnginxを導入

```bash
$ sudo apt-get install nginx
```

/etc/nginx/sites-available/defaultに60から62行目を追加して、ファイルの一覧を表示できるようにします。

```
54         # deny access to .htaccess files, if Apache's document root
55         # concurs with nginx's one
56         #
57         #location ~ /\.ht {
58         #       deny all;
59         #}
60         location /pic {
61           autoindex on;
62         }
63 }
```

nginxを再起動して設定を有効化します。

```bash
$ sudo service nginx restart
```

写真を保存するディレクトリを作成

```bash
$ sudo mkdir /var/www/html/pic
$ sudo chown pi.www-data
```

## 撮影用スクリプトを作成する

以下のスクリプトを作成します。今回はファイル名をpic.shとしました。

```bash
#!/bin/bash
/usr/bin/raspistill -w 480 -h 360 -ex auto -awb auto -rot 270 -o /var/www/html/pic/$(date +%Y%m%d%H%M).jpeg
```

スクリプトに実行権限を付与します。

```bash
$ chmod +x pic.sh
```

## cronに登録する

5分ごとに撮影用スクリプトを実行するようcrontabに以下の一行を追加します。

```
*/5 * * * * /home/pi/pic.sh
```

## ブラウザから確認

http:://<raspberry piのipアドレス>/pic にアクセスするとファイルの一覧が表示されます。

![撮影したファイル一覧](/img/20170326/07.png)

## 感想

リビングに置いていると不在時の様子がわかるので、いつ誰が何をしていたかわかり意外に面白いです。当然、家族の許可がないとできませんが。。。知人に話したら不在時のペットの様子の確認に使ってみたいと言っておりました。確かに誰もいない部屋のペットがどんな行動をしているかは気になりますね。

[^1]: カメラにコネクタが付属していますが、Raspberry Pi Zeroには接続できないので別途購入する必要があります。ちなみに公式ケースにはケースに収まる短いケーブルが付属しています。私は知らずに長いケーブルを買ってしまいました。
