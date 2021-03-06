---
title:  "OpenWhiskで天気予報サービスを使ってみる"
slug: "open-whisk-weather-news"
date:   2017-03-06 00:00:00 +0900
tags: 
  - openwhisk
  - serverless
  - watson
---
**※ [はてなブログ](https://tatata.hatenablog.jp/entry/2017/03/06/004027)から引っ越しました（2019/10/21）**

Bluemix上のOpenWhiskから天気予報(英語)を取得して、Watsonに翻訳させるサービスを作ってみました。

## OpenWhiskでの開発

メニューから[OpenWhisk]を選択します。

![メニュー](/img/20170306/01.png)

２通りの開発方法がありますが、今回は[ブラウザーで開発]を選択しました。

![選択画面](/img/20170306/02.png)

## 経度、緯度を受付けるアクションを作成

[アクションの作成]を選択して、天気予報サービスに渡す経度と緯度を取得するアクションを作成します。


![アクションの作成](/img/20170306/03.png)

次のコードを入力して、[ライブにする]をクリックします。  
このコードでは、緯度と経度を受け取り、そのまま返却します。緯度、経度の入力がない場合は、デフォルト値として東京の緯度、経度を返却します。

```javascript
function main(params) {
    var msg={};
    msg.latitude=params.latitude || "35.66";
    msg.longitude=params.longitude || "139.73";
    return msg;
}
```

[シーケンスにリンク]をクリックして、天気予報を取得する処理を次に追加します。

![シーケンスにリンク](/img/20170306/04.png)

## 天気予報を取得する処理の追加

[WEATHER]を選択します。

![新規アクション・シーケンスの構成](/img/20170306/05.png)

[NEW BINDING]をクリックします。

![NEW BINDING](/img/20170306/06.png)

[サービス・インスタンスのプロビジョニング]をクリックします。

![サービス・インスタンスのプロビジョニング](/img/20170306/07.png)

[OK、準備ができました]をクリックします。 

![OK、準備ができました](/img/20170306/08.png)

[Weather Company Data]を選択します。

![Weather Company Data](/img/20170306/09.png)

画面下の[作成]をクリックします。

![作成](/img/20170306/10.png)

この画面が表示されたらサービス・インスタンスの作成は完了です。元の画面に戻ります。

![Weather Company Dataの表示](/img/20170306/11.png)

[OK、新規サービス・インスタンス]をクリックします。

![サービス・インスタンスのプロビジョニング](/img/20170306/12.png)

作成したサービス・インスタンスが選択されていることを確認して、[構成の保存]をクリックします。画面上の名前の入力(今回はgetWeatherDataとしました)も忘れずに！

![構成の保存](/img/20170306/13.png)

[シーケンスに追加]をクリックします。

![シーケンスに追加](/img/20170306/14.png)

[これは適切なようです]をクリックします。

![これは適切なようです](/img/20170306/15.png)

[シーケンス名]を入力して、[アクション・シーケンスの保存]をクリックします。

![アクション・シーケンスの保存](/img/20170306/16.png)

[完了]をクリックします。

## 取得したデータを加工するアクションを作成

次に取得した天気予報データのうち、当日の”narrative”を抽出するアクションを作成します。

[アクションの作成]を選択します。

![アクションの作成](/img/20170306/17.png)

アクションの名前をconvertMessageとして作成しました。コードは次のようになります。取得したデータをWatsonに渡すため、メッセージを加工しています。

```javascript
function main(params) {
      var msg={};
        // 翻訳先の言語は日本語
      msg.translateTo="ja";
        // 翻訳元の言語は英語
      msg.translateFrom="en";
　　　　// 天気予報サービスから取得した当日の"narrative"を取得
      msg.payload=params.forecasts[0].narrative;
        // ログ出力用
      console.log(msg.payload);
    return msg;
}
```

作成した[マイ・シーケンス]を選び、[拡張]をクリックします。

![アクション・シーケンス・ビューワー](/img/20170306/18.png)

[MY ACTIONS]を選択します。

![新規アクション・シーケンスの構成](/img/20170306/19.png)

[convertMessage]を選択して、[シーケンスに追加]をクリックします。

![convertMessageを選択](/img/20170306/20.png)

[変更の保存]をクリックします。

## 翻訳サービス(WATSON TRANSLATOR)の追加

カタログに戻り[Watsonサービス]からサービスを作成します。

![Watsonサービスを選択](/img/20170306/21.png)

[Language Translator]を選択し、[作成]をクリックします。

![Language Translatorを選択](/img/20170306/22.png)

OpenWhiskのマイ・シーケンスに戻り、[拡張]をクリックします。

[WATSON TRANSLATOR]を選択します。

![WATSON TRANSLATORを選択](/img/20170306/23.png)

[translator]を選択します。 

![translatorを選択](/img/20170306/24.png)

[NEW BINDING]の横の青色のマークを選択して、[シーケンスに追加]をクリックします。

[変更の保存]をクリックします。

## シーケンスの実行

[このシーケンスを実行]をクリックします。

![このシーケンスを実行](/img/20170306/25.png)

[この値で実行]をクリックして、シーケンスを実行します。

![アクションの呼び出し中](/img/20170306/27.png)

実行結果です。

![呼び出しコンソール](/img/20170306/28.png)

**元の英語**

```
Partly cloudy. Low 7C.
```

**翻訳した日本語**

```
一つには曇り。 低7C。
```

翻訳精度がいまいちですね。ちなみにGoogle翻訳だと

```
晴れときどき曇り。 低い7C。
```

こちらの方がマシですね。

## 感想

サーバーレスで天気予報や翻訳サービスなどを簡単に呼び出せるのはとても便利ですね。アイデア次第でサービスを組み合わせて、面白いサービスが作れそうです。本当は取得した結果をTwitterにつぶやくようにしたかったのですが、時間がなくできませんでした。また今度挑戦してみます。それにしてもBluemixのコンソールは遅いです。もう少し改善しないものでしょうか。。。

