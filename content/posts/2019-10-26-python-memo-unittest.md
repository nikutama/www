---
title: "Python メモ – unittest"
slug: "python-memo-unittest"
date:   2019-10-26 00:00:00 +0900
tags: 
  - python
  - test
  - unittest
notoc: true
---

Pythonでのテストコードについて調べた。サードパーティ製の`pytest`が人気のようだけど、まずは、標準ライブラリの`unittest`についてのメモ。

## プロジェクト構造

まずはこんな感じで、ファイルを配置する。

```
プロジェクトルート
|- app
|  |-- __init__.py (空ファイル）
|  |-- calculator.py (テスト対象のコード）
|
|-tests
  |-- __init__.py (空ファイル）
  |-- test_calculator.py（テストコード）
```

## テストケース実装

addメソッドで足し算が正しくできるか検証する。

### テスト失敗

テストコードを最初に書く。`assertEqual()`に期待値と実行した結果を渡して検証する。

```python
# test_calculator.py
import unittest
from app.calculator import Calculator

class TestCalculator(unittest.TestCase):
     def test_add_integers(self):
         expected = 3
         actual = Calculator.add(1, 2)
         self.assertEqual(expected, actual)
```

この時点ではテスト対象のコードのメソッドには処理を記述せず、テストが失敗することを確認する。

```python
# calculator.py
class Calculator:
    @classmethod
    def add(cls, x, y):
        pass
```

テストコードを実行[^1]すると、addメソッドを実装してないので当然エラーになる。

```bash
$ python -m unittest discover
F
======================================================================
FAIL: test_add_integers (tests.test_calculator.TestCalculator)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/Users/nikutama/unittest/tests/test_calculator.py", line 8, in test_add_integers
    self.assertEqual(expected, actual)
AssertionError: 3 != None

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
```

### テスト成功

テストが成功するようにaddメソッドを実装する。

```python
# calculator.py
class Calculator:
    @classmethod
    def add(cls, x, y):
        return x + y
```

今度はテストが成功する。

```bash
$ python -m unittest discover
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
```

## エラーテストケース

### テスト失敗

addの引数に文字列が渡された場合は例外を発生する仕様[^2]とし、それを検証するテストコードを追加する。

例外が発生することを検証するには、`with self.assertRaises(想定するExceptionを指定):`を書きブロック内に例外が発生する処理を書く。

```python
# test_calculator.py
    def test_add_method_throws_type_error_when_argument_is_string(self):
        with self.assertRaises(TypeError):
            Calculator.add(1, "y")
        with self.assertRaises(TypeError):
            Calculator.add("x", 2)
        with self.assertRaises(TypeError):
            Calculator.add("x", "y")
```

テストコードを実行すると、引数の片方のみ文字列の場合は、`x + y`の実行時に`TypeError`が発生するが、どちらも文字列の場合は文字列結合になるので、`TypeError`が発生していない。


```bash
$ python -m unittest discover
test_add_integers (tests.test_calculator.TestCalculator) ... ok
test_add_method_throws_type_error_when_argument_is_string (tests.test_calculator.TestCalculator) ... FAIL

======================================================================
FAIL: test_add_method_throws_type_error_when_argument_is_string (tests.test_calculator.TestCalculator)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/Users/nikutama/unittest/tests/test_calculator.py", line 16, in test_add_method_throws_type_error_when_argument_is_string
    Calculator.add("x", "y")
AssertionError: TypeError not raised

----------------------------------------------------------------------
Ran 2 tests in 0.001s

FAILED (failures=1)
```

### テスト成功

両方の引数が文字列の場合もTypeErrorが発生するようにaddメソッドを修正する。[^3]

```python
# test_calculator.py
    def add(cls, x, y):
        if isinstance(x, str) and isinstance(y, str):
            raise TypeError 
        return x + y
```

これでテストは成功する。

```bash
$ python -m unittest discover
test_add_integers (tests.test_calculator.TestCalculator) ... ok
test_add_method_throws_type_error_when_argument_is_string (tests.test_calculator.TestCalculator) ... ok

----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
```

[^1]: `unittest`では`python -m unittest discover`でカレントディレクトリを起点に`test*.py`にマッチするテストコードを再帰的に探して実行してくれる。
[^2]: 本当は文字列以外にbooleanなども例外にする必要があるが、説明を簡単にするため文字列のみとした。
[^3]: 実際にはif文では、`x`しか型をチェックなくても、`x+y`のところで、`TypeError`は発生する。

