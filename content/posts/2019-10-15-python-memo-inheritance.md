---
title:  "Python メモ – 継承"
slug: "python-memo-inheritance"
date:   2019-10-15 00:00:00 +0900
tags: 
  - python
---

Pythonではクラス名の後ろに`()`が書いてあることがある。最初にこの記述を見たときは意味がわからず、リファクタリングのときに不要だと思い、削除してエラーになったことがあった。実は、クラス名の後ろに`(親クラス)`と記述することでクラスを継承できるということであった。

例えば、次のコードではPersonクラスを、JapaneseクラスとGermanクラスが継承している

```python
# person.py
class Person:
    def say_hello(self, name):
        print('Hello, ' + name + '!')

class Japanese(Person):
    def say_hello(self, name):
        print('こんにちは、' + name + 'さん')

class German(Person):
    def say_hello(self, name):
        print('Hallo, ' + name + '!')

p = Person()
p.say_hello('Mike')

j = Japanese()
j.say_hello('太郎')

g = German()
g.say_hello('Ben')

$ python3 person.py
Hello, Mike!
こんにちは、太郎さん
Hallo, Ben!
```

