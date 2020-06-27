---
title: "Python メモ – Azure App ServiceでのFlask起動コマンドカスタマイズ"
slug: "python-azure-app-service"
date:   2020-02-22 00:00:00 +0900
tags: 
  - azure
  - flask
  - python
notoc: true
---

Azure App ServiceにFlaskアプリケーションをデプロイしても、デフォルトの画面から更新されずにハマったのでメモ。

## デフォルトの挙動

Azure App Serviceでは、flaskアプリケーションは`application.py`か`app.py`を探して起動しているようだ。起動に実行されるコマンドは以下の通り。

```bash
# application.pyの場合
gunicorn --bind=0.0.0.0 --timeout 600 application:app
# app.pyの場合
gunicorn --bind=0.0.0.0 --timeout 600 app:app
```

ファイル名が異なる場合は、起動コマンドをカスタマイズする必要がある。

## 起動コマンドのカスタマイズ

```python
# hello.py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'
```

上のファイル`hello.py`を起動するには、次のコマンドを実行して起動コマンドをカスタマイズする。

```bash
az webapp config set --name <アプリ名> --resource-group <リソースグループ名> --startup-file "gunicorn --bind=0.0.0.0 --timeout 600 hello:app"
```

## 参考情報

https://docs.microsoft.com/en-us/azure/app-service/containers/how-to-configure-python

