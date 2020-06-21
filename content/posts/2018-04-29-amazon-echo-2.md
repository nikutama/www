---
title:  "Amazon Echoで照明をオン・オフする 2（Raspberry Pi編）"
slug: "amazon-echo-2"
date:   2018-04-29 00:00:00 +0900
tags: 
  - arduino
  - node.js
  - raspberrypi
---
**※ [はてなブログ](https://tatata.hatenablog.jp/entry/2018/04/29/123744)から引っ越しました（2019/10/21）**

今回は、Raspberry Piを[前回](/2018/04/22/amazon-echo-1)作成したArduinoに接続して、Raspbery PiからArduinoに照明のオン・オフを命令する処理を作る

## 事前準備

1. Raspberry PiとArduinoをUSBで接続
1. node.jsをインストール
1. Node Serialportをインストール[^1] 

## スクリプト作成

オンとオフで違うのはport.writeの引数（1または0）のみ

### on.js (照明オン)

```javascript
var SerialPort = require('serialport');
var port = new SerialPort('/dev/ttyACM0');

setTimeout(function() {
  port.write("1\n", function(err) {
    if (err) {
      return console.log('Error on write: ', err.message);
    }
    console.log('message written');
  });
}, 2000);


port.on('error', function(err) {
  console.log('Error: ', err.message);
})
```

### off.js (照明オフ)

```javascript
var SerialPort = require('serialport');
var port = new SerialPort('/dev/ttyACM0');

setTimeout(function() {
  port.write("0\n", function(err) {
    if (err) {
      return console.log('Error on write: ', err.message);
    }
    console.log('message written');
  });
}, 2000);


port.on('error', function(err) {
  console.log('Error: ', err.message);
})
```

## 動作確認

コマンドを実行して、照明をオン・オフできることを確認

```bash
# 照明オン
$ node on.js

# 照明オフ
$ node off.js
```

コードがイケてないけど、Raspberry Piから操作できるようになったのでよしとする。
[次回](/2018/05/02/amazon-echo-3)は、SQSからメッセージを取得して、照明をオン・オフ。

[^1]: インストールにはまったけど、メモを取ってなかった
