---
title: "Python メモ – JSON(非ASCII文字)"
slug: "python-memo-json"
date:   2019-12-26 00:00:00 +0900
tags: 
  - python
notoc: true
---

PythonでJSONを扱っていて、JSONにASCIIでない文字が含まれていたら文字化けて困った。その時の解決方法をメモ。

## 文字化けするケース

```python
>>> import json
>>>
>>> person = { "name": "にくたま" }
>>> json.dumps(person)
'{"name": "\\u306b\\u304f\\u305f\\u307e"}'
```

## 解決方法

`json.dumps`の引数に`ensure_ascii=False`をつけると正しく表示される。

```python
>>> json.dumps(person, ensure_ascii=False)
'{"name": "にくたま"}'
```

## 参考情報

https://docs.python.org/3/library/json.html

