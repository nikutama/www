---
title: "Python メモ – データクラス"
slug: "python-memo-dataclass"
date:   2019-11-17 00:00:00 +0900
tags: 
  - python
notoc: true
---

単にデータを格納するだけのクラスを作成していて、毎回、`__init__()`を書くのを面倒に感じていたが、Python 3.7からは`@dataclass`を使うと簡単にできることがわかったのでメモ。

## 従来の方法

普通にクラスを作るとこんな感じになる。

```python
# person.py
class Person:
    def __init__(self, first_name, last_name):
        self.first_name = first_name
        self.last_name = last_name

    def full_name(self):
        return self.first_name + ' ' + self.last_name
```

実行するとこうなる。

```python
# person.py
>>> from person import Person
>>> p = Person('Taro', 'Yamada')
>>> p.last_name
'Yamada'
>>> p.full_name()
'Taro Yamada'
```

## データクラスを使う方法

`@dataclass`を使うと、クラスの前にデコレータ`@dataclass`をつけて、型を指定して変数を列挙すると、`__init__()`に書いたのと同じになるので簡単。

```python
from dataclasses import dataclass

@dataclass
class Person: 
    first_name: str
    last_name: str

    @property
    def full_name(self) -> str:
        return self.first_name + ' ' + self.last_name
```

各変数へのアクセス方法は同じ。また、full_nameにはデコレータ`@property`を使ってみた。`@property`をメソッドにつけると、他の変数と同じようにアクセスできる。ちなみに、`@property`はデータクラスの機能ではない。そのため、データクラスとは関係なく単独でも利用できるようだ。

```python
>>> from person import Person
>>> p = Person('Taro', 'Yamada')
>>> p.last_name
'Yamada'
>>> p.full_name
'Taro Yamada'
```

## 参考情報

https://docs.python.org/3/library/dataclasses.html  
https://docs.python.org/3/library/functions.html?highlight=property#property

