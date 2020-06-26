---
title: "Pythonメモ – セイウチ演算子"
slug: "python-memo-walrus-operator"
date:   2019-12-08 00:00:00 +0900
tags: 
  - python
notoc: true
---

Python 3.8で導入されたセイウチ演算子についてのメモ。演算子`:=`がセイウチに似ているからこう呼ばれるらしい。

## 3.7までの書き方

```python
>>> hoge = "abc"
>>> n = len(hoge)
>>> if n > 2:
...     print(f"hogeの値は2文字より長い。{n}文字です。")
... 
hogeの値は2文字より長い。3文字です。
```

このコードでは変数`hoge`の文字数が2より大きければメッセージを表示している。

1. 変数`hoge`に入っている文字数を変数`n`に入れる
2. if文で変数`n`が2より大きいか判定
3. 真であれば、変数`n`の値と共にメッセージを表示

## 3.8での書き方

3.8からはセイウチ演算子`:=`を使うことで、条件式の中で変数`n`への代入も同時に可能になる。

```python
>>> hoge = "abc"
>>> if (n := len(hoge)) > 2:
...     print(f"hogeの値は2文字より長い。{n}文字です。")
... 
hogeの値は2文字より長い。3文字です
```

少しだけ簡潔になりますね。

## 参考情報

https://docs.python.org/ja/3/whatsnew/3.8.html
