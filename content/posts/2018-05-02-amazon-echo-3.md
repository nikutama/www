---
title:  "Amazon Echoで照明をオン・オフする 3（SQS編）"
slug: "amazon-echo-3"
date:   2018-05-02 00:00:00 +0900
tags: 
  - alexa
  - arduino
  - node.js
  - raspberrypi
  - sqs
---
**※ [はてなブログ](https://tatata.hatenablog.jp/entry/2018/05/02/163904)から引っ越しました（2019/10/21）**

今回は、[前回](/2018/04/29/amazon-echo-2)作成したプログラムを改良して、SQSからRaspberry Piにメッセージを取得して、照明をオン・オフする処理を作る。

## SQSキュー作成

メッセージを入れるSQSのキューをAWSコンソールから作成

## プログラム(turnOnOff.js)

SQSからメッセージを取得して、シリアルポート経由でArduinoに命令を送る常駐プログラムを作成

```javascript
var AWS = require('aws-sdk');
var async = require('async');
var SerialPort = require('serialport');

// Raspberry Piのシリアルポートを指定
var port = new SerialPort('/dev/ttyACM0');

// 環境変数からAWSのアクセスキーを取得
let KEY = process.env.AWS_ACCESS_KEY;
// 環境変数からAWSのシークレットキーを取得
let SECRET = process.env.AWS_SECRET_KEY;
// 環境変数からSQSのキューURLを取得
let URL = process.env.SQS_QUEUE_URL;
// 認証情報をセット
AWS.config.update({accessKeyId: KEY, secretAccessKey: SECRET});

// SQSのリージョンを指定
var sqs = new AWS.SQS({region:'ap-northeast-1'});

// SQSの情報をセット
var params = {
  QueueUrl: URL,
  AttributeNames: [
    'ApproximateFirstReceiveTimestamp'
  ],
  MaxNumberOfMessages: 1,
  MessageAttributeNames: [
    'LightStatus' //メッセージの属性LightStatusの値で判断するため設定。任意の名前でOK
  ]
};

// 引数で渡ってきた値をシリアルポートに書く
function tunOnOff(state) {
  setTimeout(function() {
    port.write(state + "\n", function(err) {
      if (err) {
        return console.log('Error on write: ', err.message);
      }
      console.log('message written');
    });
  }, 2000);
};

// エラーが発生した場合はコンソールに出力
port.on('error', function(err) {
  console.log('Error: ', err.message);
});

// 処理開始
console.log('start');
setInterval(function() {
  sqs.receiveMessage(params, function(err, data) {
    if (err) {
      console.log(err, err.stack); // an error occurred
      console.log('error dayo');
    } else if (Array.isArray(data['Messages']) && data['Messages'].length > 0) {
      // SQSのメッセージの属性LightStatusが0の場合、シリアルポートに0を書き出す
      if (data['Messages'][0]['MessageAttributes']['LightStatus']['StringValue'] == '0') {
        tunOnOff('0');
      // SQSのメッセージの属性LightStatusが1の場合、シリアルポートに1を書き出す
      } else if (data['Messages'][0]['MessageAttributes']['LightStatus']['StringValue'] == '1') {
        tunOnOff('1');
      }

      // 取得したメッセージをキューから削除
      var deleteParams = {
        QueueUrl: URL,
        ReceiptHandle: data.Messages[0].ReceiptHandle
      };
      sqs.deleteMessage(deleteParams, function(err, data) {
        if (err) {
          console.log("Delete Error", err);
        };
      });
    }, 20000 );
```

## 環境変数設定

.bashrcに次の３行を追加

```bash
export AWS_ACCESS_KEY="<AWSのアクセスキー>"
export AWS_SECRET_KEY="<AWSのシークレットキー>"
export SQS_QUEUE_URL="<SQSのキューURL>"
```

## 動作確認
プログラム実行

Raspberry Piのコンソールよりプログラムを実行

```bash
$ node turnOnOff.js
```

## SQSのキューにメッセージ送信

SQSのコンソールからメッセージ属性LightStatusを追加してメッセージを送信

![SQS メッセージ属性](/img/20180502/01.png)

属性LightStatusの値が1だと照明がオンになり、0だと照明がオフになる。

node.jsは詳しくないので、ネットに転がっているコードを真似して作成したが、node.jsの非同期の考え方がいまいち理解できておらず、今回もコードがイケてない。コードを綺麗にしたいけど、時間がないのでこのまま。。。node-redでやればよかったかな。

[次回](/2018/05/06/amazon-echo-4)はAlexaからSQSにメッセージを送信して、照明をオン・オフする。

