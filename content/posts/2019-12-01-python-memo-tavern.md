---
title: "Python メモ – REST APIのテスト"
slug: "python-memo-tavern"
date:   2019-12-01 00:00:00 +0900
tags: 
  - python
  - rest
  - tavern
  - test
notoc: true
---

PythonでREST APIをテストするツールを探していたら、`Tavern`なるものを見つけたのでメモ。


## REST API

まず、テスト対象として、ユーザーを追加とユーザー取得のAPIを用意。

```python
# app.py
from flask import Flask, request, jsonify
import json

from dataclasses import dataclass

app = Flask(__name__)

users = []

@app.route('/user/<int:id>', methods=['GET'])
def index(id):
    return jsonify(users[id-1])

@app.route('/user', methods=['POST'])
def create():
    users.append(User(request.json['name'], request.json['age']))
    return jsonify({'id': len(users)})


@dataclass
class User:
    name: str
    age: str
```

APIサーバーを起動しておく。

```bash
$ export FLASK_APP=app.py
$ export FLASK_ENV=development
$ flask run
```

## Tavernインストール

`Tavern`とテスフレームワークの`Pytest`をインストール

```bash
$ pip install pytest
$ pip install tavern
```

## テストケース作成

テストケースをyamlで記述する。ここではテストケースで共通で使う変数を`common.yaml`に、テストケースを`test_user.tavern.yaml`に書いた。

### common.yaml

このファイルにAPIの接続先を設定した。最初は、`name`と`description`を書いていなかったが、これらは必須であったためエラーになった。忘れずに書きましょう。

```yaml
# common.yaml
name: Common test information
description: information for test server

variables:
  protocol: http
  host: localhost
  port: 5000
```

### test_user.tavern.yaml

このテストケースでは、最初にPOSTでユーザーを作成して、次に作成したユーザーをGETできるか検証している。

```yaml
# tests/integration/test_user.tavern.yaml
test_name: add/get User 

strict: 
  - body

includes:
  - !include common.yaml
    
stages:
  - name: add user
    request:
      url: "{protocol:s}://{host:s}:{port:d}/user"
      method: POST
      json:
        name: test taro
        age: 7
    response:
      status_code: 200
      headers:
        content-type: application/json
      verify_response_with:
        function: tavern.testutils.helpers:validate_pykwalify
        extra_kwargs:
          schema:
            type: map
            mapping:
              id:
                type: int
                required: True
      save:
        body:
          userid: id

  - name: get user
    request:
      url: "{protocol:s}://{host:s}:{port:d}/user/{userid}"
      method: GET
    response:
      status_code: 200
      headers:
        content-type: application/json
      body:
        name: test taro
        age: 7
```

#### POST /user

1. POSTでユーザー作成
2. レスポンスの検証
    - ステータスコードが`200`であること
    - HTTPヘッダーの`Content-Type`が`application/json`であること
    - 応答メッセージの`id`が`int`であること
3. 応答メッセージの`id`の値を変数`userid`に格納

## テスト実行

```bash
$ pytest
=================================== test session starts ===================================
platform darwin -- Python 3.8.0, pytest-4.5.0, py-1.8.0, pluggy-0.13.1
rootdir: /Users/nikutama/study/python-tavern
plugins: tavern-0.33.0
collected 1 item                                                                          

tests/integration/test_user.tavern.yaml .                                           [100%]

================================ 1 passed in 0.93 seconds =================================
```

今まではPostmanで手動で確認していたが、これでAPIの検証も自動化でき楽になる。

## 参考情報

https://tavern.readthedocs.io/en/latest/index.html

