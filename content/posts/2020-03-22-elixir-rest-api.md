---
title: "Elixirで簡単なREST APIを作る"
slug: "elixir-rest-api"
date:   2020-03-22 00:00:00 +0900
tags: 
  - elixir
  - phoenix
  - rest
notoc: true
---

Elixirで簡単なREST APIを作る方法のメモ

# プロジェクト作成

webpackは不要なので`--no-webpack`を付けて実行

```bash
$ mix phx.new hello --no-webpack
$ cd hello
```

## データベース作成

デフォルトではPostgreSQLを使うので事前にPostgreSQLを起動しておき、次のコマンドを実行する。なお、 PostgreSQLへの接続情報は`config/dev.exs`(devモードの場合)に記述してあるので必要に応じて変更する。

```bash
$ mix ecto.create
```
## API作成

ここでは`name`と`age`を持つUser APIを作る。

```bash
$ mix phx.gen.json Accounts User users name:string age:integer
```

usersテーブルを作成するためマイグレートする。

```bash
$ mix ecto.migrate
```

`lib/hello_web/router.ex`の以下の部分のコメント削除し、`resources "/users"...`の１行を追加する。これにより`/api/users`でアクセス可能となる。

```elixir
# Other scopes may use custom stacks.
scope "/api", HelloWeb do
  pipe_through :api
  # この行を追加
  resources "/users", UserController, only: [:index, :show, :create, :update] 
end
```

## API起動

次のコマンドで起動する。

```bash
$ iex -S mix phx.server                                  
Erlang/OTP 22 [erts-10.6.4] [source] [64-bit] [smp:4:4] [ds:4:4:10] [async-threads:1] [hipe] [dtrace]
[info] Running HelloWeb.Endpoint with cowboy 2.7.0 at 0.0.0.0:4000 (http)
[info] Access HelloWeb.Endpoint at http://localhost:4000
Interactive Elixir (1.10.1) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)>
```

## 確認

ユーザーを登録してみる。

```bash
$ curl -X POST -H "Content-type: application/json" -d '{"user":{"name":"tom","age": 20}}' http://localhost:4000/api/users
```

登録したユーザーを取得してみる。

```bash
$ curl  http://localhost:4000/api/users/1
{"data":{"age":20,"id":1,"name":"tom"}}
```

## カスタマイズ

デフォルトではユーザー登録のリクエストメッセージは`{"user":{"name":"tom","age": 20}}`となり、`"user"`をつける必要がある。これを`{"name":"tom","age": 20}`で登録できるようにしたい。

そのためには、`lib/hello_web/controllers/user_controller.ex`を次のように書き換えればよい。

```elixir
# 変更前
def create(conn, %{"user" => user_params})
# 変更後
def create(conn, user_params)
```

またユーザー取得時には、`{"data":{"age":20,"id":1,"name":"tom"}}`となり、`"data"`がつくがこれを`{"age":20,"id":1,"name":"tom"}`にしたい。

そのためには、`lib/hello_web/views/user_view.ex`を次のように修正すればよい。

```elixir
def render("index.json", %{users: users}) do
  # %{data: render_many(users, UserView, "user.json")} # コメントアウト
  render_many(users, UserView, "user.json") # このように書き換える
end

def render("show.json", %{user: user}) do
  # %{data: render_one(user, UserView, "user.json")} # コメントアウト
  render_one(user, UserView, "user.json") # このように書き換える
end
```

リクエストを投げて確認する。

```bash
$ curl -X POST -H "Content-type: application/json" -d '{"name":"joe","age": 30}' http://localhost:4000/api/users
$ curl http://localhost:4000/api/users/2
{"age":30,"id":2,"name":"joe"}
```

ちゃんと変わってますね。

## 参考情報

https://hexdocs.pm/phoenix/up_and_running.html#content  
https://hexdocs.pm/phoenix/Mix.Tasks.Phx.Gen.Json.html  
https://hexdocs.pm/phoenix/1.4.17/phoenix_mix_tasks.html#content  

