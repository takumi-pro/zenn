---
title: "Go言語でOpenAIを活用したCLIアプリケーションの雛形をつくる"
emoji: "📦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## はじめに
最近Go言語を学習していて、何か簡単で面白いアプリケーション作りたいなーと思っていたところ、WebアプリよりもCLIアプリの方が手軽に作れるのでは（規模による）と思い、まずはアプリの雛形をつくってみることにしました。何か捻りを加えたかったので、OpenAIのAPIも活用して組み込んでみました。

## Cobraの導入
今回はCobraを使用してCLIアプリを開発していきます。
まずはインストール
```bash
go get -u github.com/spf13/cobra@latest
```

cobra-cliを使用する
```bash
go install github.com/spf13/cobra-cli@latest
```

初期化
```bash
cobra-cli init
```
以下のような構成が出来上がる
```txt:tree
cli-training
├── LICENSE
├── REDME.md
├── cmd
│   └── root.go
├── go.mod
├── go.sum
└── main.go
```

実行する
```bash
go run ./main.go

A longer description that spans multiple lines and likely contains
examples and usage of using your application. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.
```

## go-openaiのセットアップ
https://github.com/sashabaranov/go-openai

Go言語でOpenAIを使用するために`go-openai`を使用する。

インストール
```bash
go get github.com/sashabaranov/go-openai
```

## api keyの設定
以下にアクセスしてapi keyを生成しておく
https://platform.openai.com/account/api-keys

`Billing`の設定もしておく


## 参考
https://qiita.com/kotattsu3/items/d6533adc785ee8509e2c
https://zenn.dev/kou_pg_0131/articles/go-cli-packages