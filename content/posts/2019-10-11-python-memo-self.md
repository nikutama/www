---
title:  "Python メモ – __init__() と self"
slug: "python-memo-self"
date:   2019-10-11 00:00:00 +0900
tags: 
  - python
---
クラスに定義されている`__init__()`はインスタンス生成時に呼び出される。
例えば次のコードだと、Personクラスのインスタンス生成時に引数として`Yamada`を渡すと、`name`に`Yamada`がセットされて、`p.name`で名前を参照できる。厳密には、インスタンス生成時に`__new__()`が呼ばれて、`__new__()`が`__init__()`を呼び出しているらしい。

```python
>>> class Person:
...     def __init__(self, name):
...         self.name = name
... 
>>> p = Person('Yamada')
>>> 
>>> p.name
'Yamada'
```

ここで疑問が出てくる。`Person('Yamada')`のように、引数は１つしか渡していない。しかし、`__init__(self, name)`と引数は2つになっている。それは何故か？この`self`とは何なのか。
実は内部の処理では、`__new__(インスタンス生成するクラス, インスタンス生成時に渡した引数)`でインスタンスを生成して、生成されたインスタンスが`self`になる。そして、`__new__()`が`__init(self, name)__`を呼び出している。

上の例だと、`__new__(Person, 'Yamada')`が`__init__(self, 'Yamada')`を呼び出し、`self.name`に`'Yamada'`をセットしている。

Pythonでは`__init__()`だけでなく、クラスに定義するメソッドの第一引数には`self`が必須である。
例えば、次の例だと`get_name_wit_title`で`self`を定義している。これによりインスタンス生成時に設定した`self.name`を参照できる

```python
>>> class Person:
...     def __init__(self, name):
...         self.name = name
...     
...     def get_name_with_title(self):
...         return 'Mr. ' + self.name
... 
>>> p = Person('Yamada')
>>> p.get_name_with_title()
'Mr. Yamada'
```

引数が不要なメソッドであればselfは不要に思えるが、指定しないとエラーになる。

```python
>>> class Person:
...     def __init__(self, name):
...         self.name = name
...         
...     def hello():
...         return 'Hello!'
... 
>>> p = Person('Yamada')
>>> p.hello()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: hello() takes 0 positional arguments but 1 was given
```

不要でもselfを指定しましょう。

```python
>>> class Person:
...     def __init__(self, name):
...         self.name = name
...     
...     def hello(self):
...         return 'Hello!'
... 
>>> 
>>> p = Person('Yamada')
>>> p.hello()
'Hello!'
```

実は、`self`は別の名前でも問題ないが、慣習として`self`を使うことになっているので、`self`を指定しましょう。

参考情報  
https://docs.python.org/ja/3/reference/datamodel.html#object.__new__

