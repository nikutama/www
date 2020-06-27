---
title: "Python メモ – Azure App Serviceへmasterではないブランチをデプロイする方法"
slug: "python-memo-azure-app-service-deploy"
date:   2020-02-16 00:00:00 +0900
tags: 
  - azure
  - python
notoc: true
---

Azure App ServiceではローカルのGitリポジトリからAzure App Service内のGitリポジトリにプッシュすることでデプロイできる。通常の手順ではローカルのmasterブランチからAzure App Serviceのmasterへのpushになるが、masterではないブランチをデプロイする方法を調べたのでメモ。

## 事前準備

ローカルのGitリポジトリとApp Serviceのリポジトリを紐づける。

**デプロイ用のユーザー設定**

```bash
az webapp deployment user set --user-name <username> --password <password>
```

**デプロイ用のURLを取得**

```bash
az webapp deployment source config-local-git --name <app-name> --resource-group <group-name>
```

**取得したURLをリモートリポジトリとして追加**

```bash
git remote add azure <url>
```

## masterブランチのデプロイ

ローカルにあるmasterブランチをApp Serviceに反映（デプロイ）するには、次のコマンドだけでOK。

```bash
git push azure master
```

## master以外のブランチをデプロイ

App Serviceでは、masterブランチ以外のブランチをpushできるが、App Service上のアプリケーションとしては反映されないように見える。
masterブランチとは別のブランチを反映したい場合は、App ServiceのGitリポジトリはmasterブランチを指定する必要がある。例えば、ローカルにあるdevブランチをApp Serviceに反映するには次のようにする。

```bash
git push azure dev:master
```

App Serviceを開発環境、ステージング環境、本番環境のように分けている場合に、リリース際の環境別にブランチを分けているケースでは、この手段は使えるのではないだろうか。もちろん、push先のリポジトリはURLによって異なる。

```bash
# 開発環境へのリリース
git push azure dev:master
# ステージング環境へのリリース
git push azure staging:master
# 本番環境へのリリース
git push azure production:master
```

## 参考情報

https://docs.microsoft.com/en-us/azure/app-service/deploy-local-git?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json

