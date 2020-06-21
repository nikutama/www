---
title:  "Raspberry Pi Zero WからBluetoothでJBL Flip 3に音を出してみた"
slug: "raspberrypi-flip3"
date:   2017-03-18 00:10:00 +0900
tags: 
  - bluetooth
  - raspberrypi
---
**※ [はてなブログ](https://tatata.hatenablog.jp/entry/2017/03/18/173454)から引越しました（2019/10/21）**

Raspberry Pi Zero WからBluetoothでJBL Flip 3に音を出せるようにしてみました。

## モジュールインストール

```bash
$ sudo apt-get install pulseaudio-module-bluetooth bluez-tools
```

## グループ設定

```bash
$ sudo gpasswd -a pi pulse
$ sudo gpasswd -a pi lp
$ sudo gpasswd -a pulse lp
$ sudo gpasswd -a pi audio
$ sudo gpasswd -a pulse audio
```

## Bluetoothの設定

classの後に指定する値については、[http://bluetooth-pentest.narod.ru/software/bluetooth_class_of_device-service_generator.html](http://bluetooth-pentest.narod.ru/software/bluetooth_class_of_device-service_generator.html) を参照

```bash
$ sudo sh -c "echo 'extra-arguments = --exit-idle-time=-1 --log-target=syslog' >> /etc/pulse/client.conf"
$ sudo hciconfig hci0 up
$ sudo hciconfig hci0 class 0x240414
$ sudo reboot
```

## ペアリング

```bash
$ sudo bluetoothctl
[NEW] Controller XX:XX:XX:XX:XX:XX RaspberryPi [default]
[bluetooth]# agent KeyboardOnly
Agent registered
[bluetooth]# default-agent
Default agent request successful
[bluetooth]# scan on
Discovery started
[CHG] Controller  Discovering: yes
[bluetooth]# pair XX:XX:XX:XX:XX:XX
[agent] Enter PIN code: 0000  # 0000を入力
[CHG] Device XX:XX:XX:XX:XX:XX Paired: yes
Pairing successful
[CHG] Device XX:XX:XX:XX:XX:XX Connected: no
[CHG] Device XX:XX:XX:XX:XX:XX RSSI: -60
[bluetooth]# trust XX:XX:XX:XX:XX:XX
[bluetooth]# connect XX:XX:XX:XX:XX:XX
[bluetooth]# exit
```

## Pulseaudioの起動と設定

```bash
$ pulseaudio --start
```

このコマンドで値を確認

```bash
$ pacmd list-sinks
```

確認した値を引数として渡す

```bash
$ pacmd set-default-sink bluez_sink.XX_XX_XX_XX_XX_XX
```

音量調整

```bash
$ alsamixer
```

## 再生

```bash
$ mplayer hoge.mp3
```

Raspberry Piのパワーがないためか、設定が悪いためか音質はあまりよくありませんでした。それとこの設定だけだと再起動すると設定が消えますが、今回は試験的に設定しただけなので設定を永続化する方法まで調べておりません。

## 参考にしたサイト

- https://www.raspberrypi.org/magpi/bluetooth-audio-raspberry-pi-3/
- https://thecodeninja.net/2016/06/bluetooth-audio-receiver-a2dp-sink-with-raspberry-pi/
- http://youness.net/raspberry-pi/bluetooth-headset-raspberry-pi
