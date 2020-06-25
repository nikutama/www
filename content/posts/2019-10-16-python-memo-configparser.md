---
title: "Python メモ – 設定ファイル"
slug: "python-memo-configparser"
date:   2019-10-16 00:00:00 +0900
tags: 
  - python
notoc: true
---

設定ファイルから値を読み込む方法について調べた。

## 設定ファイル

```ini
# config.ini
[section1]
key1 = value1
key2 = value3

[section2]
keyA = valueA
keyB = valueB
```

## プログラム

設定ファイルに定義した項目に`key - value`形式でアクセスできる。

```python
# app.py
import configparser

config = configparser.ConfigParser()
config.read('config.ini')

for section in config.sections():
    for key, value in config.items(section):
        print(('{} : {}').format(key, value))

print(config['section1']['key1'])
```

## 実行結果

```bash
$ python3 app.py                    
key1 : value1
key2 : value3
keya : valueA
keyb : valueB
value1
```

keyは小文字で格納されるようです。

## 参考情報

https://docs.python.org/3/library/configparser.html

