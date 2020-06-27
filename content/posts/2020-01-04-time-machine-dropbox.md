---
title: "Time Machine と Dropbox のスマート シンク"
slug: "time-machine-dropbox"
date:   2020-01-04 00:00:00 +0900
tags: 
  - dropbox
  - mac
  - time machine
notoc: true
---

Time Machineの初回バックアップ時で表示されるバックアップの容量がバックアップ対象の容量よりも大きなっていた。気になってログを調べるとDropboxのスマート シンクが原因であった。

スマート シンクはローカルにはファイルを保存せず、Dropboxのサーバー上にのみファイルを保存し、必要な時にサーバーからダウンロードする仕組みである。ローカルのディスク容量が少ない時には重宝する機能だが、Time Machineでのバックアップ時にはこの機能が邪魔をするようで、ログに以下のようなエラーを出力する。

```
Error: (-36) SrcErr:NO Copying /Volumes/com.apple.TimeMachine.localsnapshots/Backups.backupdb/<マシン名>/2020-01-03-083400/Macintosh HD - Data/Users/<ユーザー名>/Dropbox/<フォルダ名>/<スマートシンク対象のファイル名> to /Volumes/Time Machineバックアップ(以下、略)
```

対応としては、Time Machineのオプションで、バックアップ対象から場外する項目に、Dropboxのフォルダを追加すればよい。

