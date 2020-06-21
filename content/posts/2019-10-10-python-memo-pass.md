---
title:  "Python メモ – pass 文"
slug: "python-memo-pass"
date:   2019-10-10 00:00:00 +0900
tags: 
  - python
---
`pass`だけ書いてあるクラスやメソッドがあった場合、それは何もしない。TDDでテストコードを先に書いて、実装を後でする場合に使うと便利ですね。

```python
# 空のクラス
class Foo:
    pass
```

```python
# 空のメソッド
def foo:
    pass
```

参考情報
[Python チュートリアル – pass文](https://docs.python.org/ja/3/tutorial/controlflow.html#pass-statements)

