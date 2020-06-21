---
title:  "DropboxへAPIを使ってアクセスしてみる"
slug: "dropbox-file-upload"
date:   2017-02-19 00:00:00 +0900
tags: 
  - dropbox
  - python
---
**※ [はてなブログ](https://tatata.hatenablog.jp/entry/2017/02/19/213408)から引っ越しました（2019/10/21）**

DropboxのAPIを使ってみました。言語はPythonを選びました。

## Dropbox for Pythonをインストール

pipコマンドでインストール。

```bash
$ pip install dropbox
```

## Dropboxへアクセスするアプリを作成

Dropboxにログインして、[https://www.dropbox.com/developers/apps/create](https://www.dropbox.com/developers/apps/create) にアクセスします。

以下の3項目を入力して、[Create app]をクリックします。

[1. Choose an API] -> [Dropbox API]を選択しました。

[2. Choose the type of access you need] -> [App folder] [^1]を選択しました。

[3. Name your app] -> [AppSample1] 名前は好きな名前にしましょう。[^2]

メニューの[My apps]-[Settings]を表示して、[Generate access token]の[Generate]をクリックし、表示された値を控えます。

## アクセスするコードを作成

```python
# coding=utf-8
import dropbox

dbx = dropbox.Dropbox('<先ほど控えたトークンを指定>')
dbx.users_get_current_account()
f = open('<アップロードするローカルファイルのパスを指定>', 'rb')
dbx.files_upload(f.read(),'<Dropbox上でのファイルパスを指定>')
f.close()

# アプリディレクトリにあるファイルの一覧を表示
for entry in dbx.files_list_folder('').entries:
    print(entry.name)
```

## アップロード実行

作成したファイルを実行すると、Dropboxのルートに[アプリ]というフォルダが作成され、その下にデフォルトだとアプリ名と同じフォルダが作成され、その直下にファイルがアップロードされます。

[^1]: folderを選択するとアプリ専用のフォルダ以外にはアクセスできません。
[^2]: アプリ名は他の人が使っていると駄目なようなので、他とかぶらないようにしましょう。
