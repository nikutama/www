---
title:  "Amazon Echoで照明をオン・オフする 4（Alexa編）"
slug: "amazon-echo-4"
date:   2018-05-06 00:00:00 +0900
tags: 
  - lambda
  - node.js
  - raspberrypi
  - sqs
---
**※ [はてなブログ](https://tatata.hatenablog.jp/entry/2018/05/06/193102)から引っ越しました（2019/10/21）**

今回は、AlexaスキルからLambda経由で[前回](/2018/05/02/amazon-echo-3)作成したSQSにメッセージを送って、照明をオン・オフする処理を作成する。

## Lambda関数作成

Alexaスキルからの命令を受け取って、SQSにメッセージを投げるLambda関数を作成する。

![Lambda関数の作成](/img/20180506/01.png)

### alexa-sdkをインストール

```bash
$ npm install --save alexa-sdk
```

### 次の２つのファイルを作成

index.js

```javascript
const AWS = require('aws-sdk');
const Alexa = require('alexa-sdk');
let sqs = new AWS.SQS({region:'ap-northeast-1'});
let url = process.env.SQS_QUEUE_URL;
var state = '-1';
AWS.config.update({accessKeyId: 'KEY', secretAccessKey: 'SECRET'});

function changeState(state) {
        var params = {
        MessageBody: state,
        QueueUrl: url,
        DelaySeconds: 0,
        MessageAttributes: {
            LightStatus: {
              DataType: 'Number',
              StringValue: state
            },
        },
    };
    sqs.sendMessage(params, function(err, data) {
        if (err) console.log(err, err.stack);
        else     console.log(data);
    });
};

const handlers = {
    'turnOn' : function() {
        changeState('1');
        this.emit(':responseReady');
    },
    'turnOff' : function() {
        changeState('0');
        this.emit(':responseReady');
    }
};

exports.handler = function(event, context, callback) {
    const alexa = Alexa.handler(event, context, callback);
    alexa.appId = process.env.APP_ID // APP_ID is your skill id which can be found in the Amazon developer console where you create the skill.
    alexa.registerHandlers(handlers);
    alexa.execute();
};
```

package.json

```json
{
  "name": "light",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "alexa-sdk": "^1.0.25"
  }
}
```

### zipファイルにしてアップロードする

```bash
$ zip -r light.zip *
```

![Lambda関数の画面](/img/20180506/02.png)

### 環境変数の設定

APP_IDには後ほど作成するAlexaスキルのIDを、SQS_QUEUE_URLには前回作成したSQSキューのURLを設定する。

![環境変数の画面](/img/20180506/03.png)

## Alexaスキル作成

[Alexaコンソール](https://developer.amazon.com/ja/alexa)にログインしてスキルを作成する。  
ログインに使用するアカウントは、Amazon Echoを買ったamazon.co.jpのアカウントを使う。[^1]

### 呼び出し名

「スキルの呼び出し名」にAlexaで呼び出すときの名前を設定する。ここでは、「ほげほげまん」とした。

![呼び出し名の画面](/img/20180506/04.png)

### インテント

インテントには照明をオンにするときのフレーズを設定する。  
この例では、「Alexa、ほげほげまんであかるくして」、「Alexa、ほげほげまんでつけて」と言うと照明がオンになる。

![照明オンのインテント設定画面](/img/20180506/05.png)

同様に照明をオフにするときのフレーズを設定する。

![照明オフのインテント設定画面](/img/20180506/06.png)

### エンドポイント

ここに表示されている「スキルID」をLambda関数の環境変数APP_IDに設定する。  
「デフォルトの地域」にLambda関数のARNを設定する

![エンドポイントの設定画面](/img/20180506/07.png)

## 動作確認

照明オン・オフをAmazon Echoからできることを確認

### 照明オン

- 「Alexa、ほげほげまんであかるくして」
- 「Alexa、ほげほげまんでつけて」

### 照明オフ

-「Alexa、ほげほげまんでくらくして」
-「Alexa、ほげほげまんできって」
-「Alexa、ほげほげまんでけして」

Lambda関数で使ったnode.jsは詳しくないのでもっと良い方法があると思う。スキルの設定ももっとスマートな方法があるはず。でもこれで動くことは動く。

作成して時間が経っているので、記憶があやふやになり、あまり詳しく書けなかったのが反省。

[^1]: amazon.comのアカウントでログインしたら、Echoと関連づけられていなかった。

