---
title: "Python メモ – ログ"
slug: "python-memo-log"
date:   2019-10-20 00:00:00 +0900
tags: 
  - python
---

Pythonでのログ出力について調べた。

## ログレベル

ログレベルは、`debug`、`info`、`warning`、`error`、`critical`の順にレベルが高くなり、`logger.ログベル('出力するメッセージ')`で出力できる。

```python
import logging

logging.debug('debug message')
logging.info('info message')
logging.warning('warning message')
logging.error('error message')
logging.critical('critical message')
```

デフォルトでは、`warning`以上のログが出力される。

```bash
WARNING:root:warning message
ERROR:root:error message
CRITICAL:root:critical message
```

`logging.basicConfig(level=logging.ログレベル)`でログレベルを変更できる。以下は`info`に変更した例。

```python
import logging

logging.basicConfig(level=logging.INFO)
logging.debug('debug message')
logging.info('info message')
logging.warning('warning message')
logging.error('error message')
logging.critical('critical message')
```

ログレベルを`info`にしたので、`info`以上が出力されている。

```bash
INFO:root:info message
WARNING:root:warning message
ERROR:root:error message
CRITICAL:root:critical message
```

## フォーマット

ログのフォーマットは`logging.basicConfig(format=FORMAT)`で指定できる。

```python
import logging

FORMAT = '%(asctime)s [%(levelname)s] %(name)s %(message)s'
logging.basicConfig(level=logging.INFO, format=FORMAT)
logging.info('info message')
```

出力結果

```bash
2019-10-20 18:41:14,862 [INFO] root info message
```

## ファイル出力

ログの出力先ファイル名は`logging.basicConfig(filename='ファイル名')`で指定できる。

```python
import logging

FORMAT = '%(asctime)s [%(levelname)s] %(name)s %(message)s'
logging.basicConfig(filename='application.log', level=logging.INFO, format=FORMAT)

logging.info('message 1')
logging.warning('message 2')
logging.error('message 3')
```

出力結果

```bash
$ cat application.log
2019-10-20 18:47:38,210 [INFO] root message 1
2019-10-20 18:47:38,210 [WARNING] root message 2
2019-10-20 18:47:38,210 [ERROR] root message 3
```

## Logger

Loggerを使うと、どこでログが出力されたかがわかりやすくなる。

```python
# application.py
import logging
import mylib

FORMAT = '%(asctime)s [%(levelname)s] %(name)s %(message)s'
logging.basicConfig(level=logging.INFO, format=FORMAT)

logger = logging.getLogger(__name__)
logger.info('call do_something()')
mylib.do_something()
```

```python
# mylib.py
import logging

logger = logging.getLogger(__name__)

def do_something():
    logger.info("something...")
```

`application.py`から`mylib.do_somehitng()`を呼び出しており、ログには`__main__`または`mylib`が記録されている。`__main__`なら`application.py`、`mylib`なら`mylib.py`が出力していることを意味する。

```bash
$ python application.py
2019-10-20 19:14:15,167 [INFO] __main__ call do_something()
2019-10-20 19:14:15,168 [INFO] mylib something...
```

## Handler

ハンドラを使うことで、ログをコンソールやファイルなど複数箇所に出力することができる。
次の例では、`StreamHandler`でコンソール、`FileHandler`でファイルにログを出力している。

```python
# application.py
import logging
import mylib

logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

formatter = logging.Formatter(
    '%(asctime)s - %(name)s - %(levelname)s - %(message)s')

ch = logging.StreamHandler()
ch.setLevel(logging.ERROR)
ch.setFormatter(formatter)

fh = logging.FileHandler('application.log')
fh.setLevel(logging.INFO)
fh.setFormatter(formatter)

logger.addHandler(ch)
logger.addHandler(fh)

logger.debug('debug message')
logger.info('info message')
logger.warning('warn message')
logger.error('error message')
logger.critical('critical message')
```

ログレベルをコンソール(ERROR)とファイル(INFO)で変えているので出力内容が異なっている。

コンソールへの出力結果
```bash
$ python application.py
2019-10-20 19:28:58,770 - __main__ - ERROR - error message
2019-10-20 19:28:58,770 - __main__ - CRITICAL - critical message
```

ファイルへの出力結果

```bash
$ cat application.log
2019-10-20 19:28:58,769 - __main__ - INFO - info message
2019-10-20 19:28:58,769 - __main__ - WARNING - warn message
2019-10-20 19:28:58,770 - __main__ - ERROR - error message
2019-10-20 19:28:58,770 - __main__ - CRITICAL - critical message
```

## ファイルのローテート

`logging.handlers.RotatingFileHandler`を使うと、`maxBytes`で指定したサイズを超えるとファイルが切り替えられる。`backupCount`で指定した数だけファイルがバックアップされる。

```python
import logging
import logging.handlers
import mylib

logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

formatter = logging.Formatter(
    '%(asctime)s - %(name)s - %(levelname)s - %(message)s')

fh = logging.handlers.RotatingFileHandler('application.log' , maxBytes=20, backupCount=3)

fh.setLevel(logging.INFO)
fh.setFormatter(formatter)

logger.addHandler(fh)

logger.info('info message')
logger.warning('warn message')
logger.error('error message')
logger.critical('critical message')
```

ログが4つ出力されている。

```bash
$ ls application.log*
application.log         application.log.1       application.log.2
application.log.3
```

## ログの設定のファイル化

設定をコードの中に直接書くのではなく、外部の設定ファイルに記述する。

```ini
# log.conf
[loggers]
keys=root

[handlers]
keys=consoleHandler, fileHandler

[formatters]
keys=simpleFormatter

[logger_root]
level=DEBUG
handlers=consoleHandler, fileHandler

[handler_consoleHandler]
class=StreamHandler
level=ERROR
formatter=simpleFormatter
args=(sys.stdout,)

[handler_fileHandler]
class=FileHandler
level=INFO
formatter=simpleFormatter
args=('application.log','a')

[formatter_simpleFormatter]
format=%(asctime)s - %(name)s - %(levelname)s - %(message)s

# application.py
import logging
import logging.config
import mylib

logging.config.fileConfig('log.conf')
logger = logging.getLogger(__name__)

logger.debug('debug message')
logger.info('info message')
logger.warning('warn message')
logger.error('error message')
logger.critical('critical message')
```

実行結果は、[Handlerのセクション](#handler)での実行結果と同じになる。

コンソールへの出力結果
```bash
$ python application.py
2019-10-20 20:21:58,076 - __main__ - ERROR - error message
2019-10-20 20:21:58,076 - __main__ - CRITICAL - critical message
```

ファイルへの出力結果

```bash
$ cat application.log
2019-10-20 20:21:58,074 - __main__ - INFO - info message
2019-10-20 20:21:58,075 - __main__ - WARNING - warn message
2019-10-20 20:21:58,076 - __main__ - ERROR - error message
2019-10-20 20:21:58,076 - __main__ - CRITICAL - critical message
```

## 参考情報

https://realpython.com/python-logging/  
https://docs.python.org/ja/3/howto/logging.html  
https://docs.python.org/ja/3/library/logging.handlers.html
