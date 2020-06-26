---
title: "Python メモ – Celery"
slug: "python-memo-celery"
date:   2019-11-10 00:00:00 +0900
tags: 
  - celery
  - python
notoc: true
---

実行する処理が重い場合、処理を受け付けるプログラムと処理を実行するプログラムを分離して、非同期にバックグラウンドで処理をすることで、リクエスト元のプログラムには素早く応答を返すことができる。[^1]

Pythonでは[Celery](http://docs.celeryproject.org/en/latest/)を使うケースが多いようなので、調べたことをメモする。

## セットアップ

まずは、pipでインストール

```bash
$ pip install -U Celery
```

Brokerはいくつかあるが、Docker版のRabbitMQを使った。

```bash
$ docker run -d --hostname my-rabbit --name some-rabbit-with_management -p 8080:15672 -p 5672:5672 rabbitmq:3-management
```

## 簡単な実行例

```python
# tasks.py
from celery import Celery
import time

app = Celery('tasks', broker='pyamqp://guest@localhost//')

@app.task()
def add(x, y):
    time.sleep(60)  # 本来はいらない処理だが後で実行中のステータスを確認したいため記述している
    return x + y
```

### ワーカー起動

処理を実行するワーカーを起動する。デフォルトではワーカーは4つ起動するが、ここでは１つにした。

```bash
$ celery -A tasks worker --loglevel=info --concurrency=1
```

### タスク実行

タスクを非同期で実行する。

```python
>>> from tasks import add
>>> add.delay(2, 3)
<AsyncResult: d4393779-6ed9-4172-b605-696f9e8fac8b>
```

ワーカーのコンソールに処理結果が出力されていることがわかる。

```
[2019-11-10 17:02:03,800: INFO/MainProcess] Received task: tasks.add[d4393779-6ed9-4172-b605-696f9e8fac8b]  
[2019-11-10 17:03:06,358: INFO/ForkPoolWorker-1] Task tasks.add[d4393779-6ed9-4172-b605-696f9e8fac8b] succeeded in 60.00033328399999s: 5
```

デフォルトでは、実行結果のステータスや戻り値をリクエスト元が知ることができない。知りたい場合は`result_backend`を使用するとよい。

## 処理結果の取得

`result_backend`にはDatabaseやRedisなども使えるが、ここではRabbitMQを使う。

まず、設定ファイルのRabbitMQを使う設定を書く。

```python
# celeryconfig.py
CELERY_RESULT_BACKEND = 'rpc://'
CELERY_RESULT_PERSISTENT = True
```

設定ファイルをプログラムから読み込むように修正して、ワーカーを起動しなおす。

```python
# tasks.py
from celery import Celery
import time

app = Celery('tasks', broker='pyamqp://guest@localhost//')
app.config_from_object('celeryconfig') # この行を追加

@app.task()
def add(x, y):
    time.sleep(60)
    return x + y
```

タスクを実行する。

```python
>>> from tasks import add
>>> result =  add.delay(2, 3)
>>> result.ready()
False
```

`add.delay()`の後すぐに`result.ready()`すると、まだ実行中なので`False`が返る。60秒経過後に実行すると`True`になり、`get()`で戻り値を取得できる。

```python
>>> result.ready()
True
>>> result.get()
5
```

## タスクの中止

タスクを止めたい場合は`revoke()`で止めることができる。引数がない場合は、ワーカーが処理を始めているタスクは中止できない。実行中のタスクを中止するには`terminate=True`をオプションに指定する。

[^1]: 非同期なので、処理結果を知りたい場合は、別途、結果を問い合わせる必要がある。

```python
>>> a = add.delay(2,3)
>>> b = add.delay(2,3)
>>> b.revoke()
>>> a.revoke(terminate=True)

>>> result =  add.delay(2, 3)
>>> app.control.revoke(result.id)
```

```python
# b.revoke()のログ 
[2019-11-10 17:33:50,049: INFO/MainProcess] Tasks flagged as revoked: 74cf7c4e-8b37-4894-8e67-8e7436e85a7f

# a.revoke(terminate=True)のログ 
[2019-11-10 17:34:02,328: INFO/MainProcess] Terminating 1ce4462d-77cb-4f54-a76a-7635a0d35f7f (Signals.SIGTERM)
[2019-11-10 17:34:02,348: ERROR/MainProcess] Task handler raised error: Terminated(15)
Traceback (most recent call last):
  File "/Users/nikutama/study/python/venv/lib/python3.7/site-packages/billiard/pool.py", line 1769, in _set_terminated
    raise Terminated(-(signum or 0))
billiard.exceptions.Terminated: 15
[2019-11-10 17:34:02,406: INFO/MainProcess] Discarding revoked task: tasks.add[74cf7c4e-8b37-4894-8e67-8e7436e85a7f]
```

## タスクのモニター

### ステータスの変化によってハンドリング

ステータスが変わったら、それを検知して処理したい場合、ステータスを監視するプログラムを作るとよい。

次の例では、`RECEIVED`, `STARTED`, `SUCCESS`, `FAILURE`, `REVOKED`の時に、監視プログラムがコンソールにメッセージを出力する。

```python
# monitor.py
from celery import Celery

def my_monitor(app):
    state = app.events.State()

    def announce_task_status(event):
        state.event(event)
        task = state.tasks.get(event['uuid'])
        print('Status: %s[%s] %s %s' % (
            task.state, task.name, task.uuid, task.info()))
        
        state.event(event)

      task.state, task.name, task.uuid, task.info()))

    with app.connection() as connection:
        recv = app.events.Receiver(connection, handlers={
            'task-received': announce_task_status,
            'task-started': announce_task_status,
            'task-succeeded': announce_task_status,
            'task-failed': announce_task_status,
            'task-revoked': announce_task_status,
        })
        recv.capture(limit=None, timeout=None, wakeup=True)

if __name__ == '__main__':
    app = Celery(broker='amqp://guest@localhost//')
    app.config_from_object('celeryconfig')
    my_monitor(app)
```

監視プログラムを起動して、別コンソールでワーカーにタスクを実行すると、監視プログラムにメッセージが出力されていることがわかる。

### Flowerによる監視

[Flower](https://flower.readthedocs.io/en/latest/)を使うと、Webから状況を確認できる。

pipでインストールして起動後に、ブラウザで<http://localhost:5555>にアクセスすると、タスクの状況を確認できる。この画面からタスクを終了することも可能。

```bash
$ pip install flower
$ celery -A tasks flower
```

![Flowerの画面](/img/20191110/01.png)


#### Flowerで注意するポイント

Flowerを起動した後のの状況のみ確認できる。起動前の状況は確認できない。  
Flowerはデフォルトではパーシステントではないので、Flowerを再起動すると、前に表示されていた情報はクリアされる。再起動後も保持したい場合は、パーシステントを有効にして起動する。

```bash
$ export FLOWER_PERSISTENT=True
$ celery -A tasks flower
```

パーシステントを有効にすると、デフォルトではカレントディレクトリに`flower`という名前のファイルが作成されて永続化される。

## 参考情報

http://docs.celeryproject.org/en/latest/

