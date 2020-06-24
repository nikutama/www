---
title:  "Python メモ – プライベート メソッド"
slug: "python-memo-private"
date:   2019-10-13 00:00:00 +0900
tags: 
  - python
---

Pythonのプライベートメソッドの書き方について調べたのでメモ。Pythonでは、名前が`__`で始まり`__`で終わらないメソッドがプライベートとして扱われる。次の例だと、`__fullname()`がプライベートになり、外から呼び出すとエラーになる。

```python
>>> class Person:
...     def __init__(self, firstname, lastname):
...         self.firstname = firstname
...         self.lastname = lastname
...
...     def print_fullname(self):
...         print(self.__fullname())
...
...     def __fullname(self):
...         return self.firstname + ' ' + self.lastname
...
...

>>> p = Person('Donald', 'Trump')
>>>
>>> p.print_fullname()
Donald Trump

>>> p.__fullname()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Person' object has no attribute '__fullname'
```

しかし、プライベートメソッドの前に`_クラス`をつけると呼び出せる。

```python
>>> p._Person__fullname()
'Donald Trump'
```

よって、厳密には外から呼び出せないわけではないので気をつける必要がある。

