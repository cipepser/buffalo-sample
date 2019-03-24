# GoBuffalo

[Buffalo – Rapid Web Development in Go](https://gobuffalo.io/en)を触ってみる。

## TODOリスト（bussiness-card）

[Business\-card in GoBuffalo \- Part 1 \- My Code Smells\!](https://mycodesmells.com/post/business-card-in-gobuffalo---part-1)をやってみる。

`buffalo dev`でエラーがあった。

```sh
❯ buffalo dev
buffalo: 2019/03/23 07:51:25 === Rebuild on: :start: ===
buffalo: 2019/03/23 07:51:25 === Running: go build -v -i -tags development -o tmp/business-card-build  (PID: 33723) ===
actions/app.go:13:2: cannot find package "github.com/gobuffalo/mw-i18n" in any of:
	/usr/local/Cellar/go/1.12/libexec/src/github.com/gobuffalo/mw-i18n (from $GOROOT)
	/Users/cipepser/.go/src/github.com/gobuffalo/mw-i18n (from $GOPATH)
buffalo: 2019/03/23 07:51:26 === Error! ===
buffalo: 2019/03/23 07:51:26 === exit status 1
actions/app.go:13:2: cannot find package "github.com/gobuffalo/mw-i18n" in any of:
	/usr/local/Cellar/go/1.12/libexec/src/github.com/gobuffalo/mw-i18n (from $GOROOT)
	/Users/cipepser/.go/src/github.com/gobuffalo/mw-i18n (from $GOPATH)
 ===
```

`go get`したらエラーは消えた。

```sh
❯ go get -u github.com/gobuffalo/mw-i18n
```

チュートリアル中の以下コマンドで`createdb command not found`がでた。

```sh
$ createdb -U postgres -h 127.0.0.1 -p 15432 business-card_development
```

dockerで作るのでどうしようか迷ったけど、入れてしまうことにした。

```sh
❯ brew install postgresql
```

ちなみにpasswordは`mysecretpassword`だった。これいつ設定されたんだ。

この時点で`http://localhost:3000/`にアクセスすると`500 - ERROR`にはなるけど、buffaloの画面は見れる。

`home.html`、`resume.html`、`contact.html`を作ってbuffaloを起動して`curl`とブラウザからアクセスしたけど、だめ。
よく見ると`couldn't start a new transaction: could not create new transaction: dial tcp 127.0.0.1:5432: connect: connection refused`になっているので、docker動いていない？

```sh
❯ docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                     NAMES
8f330a5ee55f        postgres               "docker-entrypoint.s…"   7 hours ago         Up 7 hours          0.0.0.0:15432->5432/tcp   some-postgres
```

なので、`15432`で待受している。  
⇛`database.yaml`がデフォルトじゃなくて、少し変わっていた。

`port`フィールドを追加したのと、開発・本番のURLのところを`15432`に変えたらエラーメッセージが変わった。

`couldn't start a new transaction: could not create new transaction: pq: password authentication failed for user "postgres"`

postgresのパスワードは`postgres`だめだったかも？
`database.yml`の`password`フィールドを`mysecretpassword`に変えたらエラーメッセージが変わった。

`couldn't start a new transaction: could not create new transaction: pq: database "business_card_development" does not exist`

`database.yml`の`database`フィールドを`business-card_development`に変えたら(`_`を`-`に変更)エラーメッセージが変わった。

`runtime error: invalid memory address or nil pointer dereference`

`app.go`の以下も`_`を`-`に変更する必要があった。

```go
app = buffalo.New(buffalo.Options{
  Env:         ENV,
  SessionName: "_business-card_session",
})
```

`database.yml`も`-`に修正。パスワードも直した。

それでもうまく動かないので一度消してやり直す。

`createdb -U postgres -h 127.0.0.1 -p 15432 business-card_development`
の`-`を`_`にしてDBを作り直したらうまくいきそう。
（作り直したのでもろもろファイルが消えてしまったけど、デフォルトでちゃんと表示されるようになった）


## References
- [Buffalo – Rapid Web Development in Go](https://gobuffalo.io/en)
- [Go製WebToolKit Buffalo\[概要編\] \- Qiita](https://qiita.com/k-kurikuri/items/f46356b70fe3e7e8da7d)
- [Business\-card in GoBuffalo \- Part 1 \- My Code Smells\!](https://mycodesmells.com/post/business-card-in-gobuffalo---part-1)