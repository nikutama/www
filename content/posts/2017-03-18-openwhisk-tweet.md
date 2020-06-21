---
title:  "OpenWhiskからつぶやけないorz"
slug: "openwhisk-tweet"
date:   2017-03-18 00:00:00 +0900
tags: 
  - node.js
  - openwhisk
  - serverless
  - twitter
---
**※ はてなブログから引っ越しました（2019/10/21）**


[前回]作成した天気予報を取得する仕組みからTwitterにつぶやくプログラムを作ろうとしました。

## Twitterの開発者用API取得

Twitterの開発者用のAPIを取得します。

[Twitter](https://apps.twitter.com/)にアクセスして、[Create New App]をクリックします。

[Name]、[Description]、[Website]への入力し、[Deveopper Aggreement]に同意して、[Create your Twiiter applicaiton] をクリックします。

[Cretae my access toke] をクリックして、アクセストークンを生成したら、[Consumer Key (API Key) ]、[Consumer Secret (API Secret)]、[Access Token]、[Access Token Secret]を控えておきます。

## 必要なモジュールのインストール

node.js、npmを事前にインストールしておきます。

TwitterのAPIをコールするモジュールtwittを導入します。

```bash
$ npm install twitt
```

## プログラム作成

index.jsという名前で以下のファイルを作成します。

```javascript
var Twit = require('twit');
var client = new Twit({
    consumer_key: '先ほど控えた値を指定',
    consumer_secret: '先ほど控えた値を指定',
    access_token: '先ほど控えた値を指定',
    access_token_secret: '先ほど控えた値を指定'
});

function main(params) {
  msg=params.payload;
  //msg='ローカルでバッグ用メッセージ';
  client.post('statuses/update', { status: msg }, function(error, tweet, response) {
    if(error) throw error;
    console.log(tweet);     // Tweet body.
    console.log(response);  // Raw response object.
    return tweet;
  });
}

module.exports.main = main;
// ローカルで実行する場合は以下のコメントを外す
//main(params);
```

## package.jsonの作成

package.jsonという名前で以下のファイルを作成します。

```javascript
{
  "name": "TweetWeather",
  "main": "index.js",
  "dependencies" : {
    "twit" : "2.2.5"
  }
}
```

## Bluemixへアクションを登録

作成したファイルをZIP化します。

```bash
$ zip -r action.zip *
```

[https://console.ng.bluemix.net/openwhisk/learn/cli](https://console.ng.bluemix.net/openwhisk/learn/cli)より、OpenWhiskのコマンドラインツールをダウンロードします。

プログラムをアップロードして、Bluemix上にアクションを作成します。ここではアクション名をTweetWeatherにしました。

先ほどダウンロードしたツールを次のように実行することでアクションを登録できます。

```bash
$ wsk action create TweetWeather --kind nodejs:6 action.zip
```

[前回]作成したシーケンスの最後にこのアクションをつなげると、日本語に翻訳した天気予報をtweetできます。と言いたいのですが、OpenWhiskからはなぜかTweetできませんでした。。。ローカルからだとつぶやけるのですが。

次のように意図的にエラーになる行を追加するとつぶやけるので、プログラムの問題だと思います。node.jsをよくわかっていないために正しいプログラムを書けていないようです。

```javascript
    return tweet;
  });
  throw error;  //意図的にエラーになる行を追加
}
```

{{< tweet  842934821429100544 >}}

node.jsの勉強が必要ですね。

[前回]: /2017/03/06/open-whisk-weather-news
