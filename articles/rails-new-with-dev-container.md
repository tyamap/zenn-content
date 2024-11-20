---
title: "Dev Container で rails new"
emoji: "💎"
type: "tech"
topics:
  - "rails"
  - "devcontainer"
  - "docker"
  - "rails-new"
published: false
---

# 準備
## Docker Desktop をインストール
## VSCode をインストール
## Dev Containers 拡張機能をインストール
## rails-new をインストール
rails をインストールするには、ruby やDBクライアントをインストールする必要があります。
windows や macOS など環境によって手順に違いが出てきます。

rails-new を使えば、ローカルに ruby や rails をインストールしなくても `rails new` できます。
devcontainer を使うモチベーションとしては、環境依存をなくしたいというものがあるので、
せっかくなのでrails-new を使いましょう。

- https://github.com/rails/rails-new?tab=readme-ov-file
- PATH を通す

# rails-new を使ってプロジェクトを作成

devcontainer オプションをつけて作成します
rails-new には `rails new` のオプションをそのまま指定できます

```sh
rails-new myapp --devcontainer
```

VSCode で myapp プロジェクトを開きます
```sh
code myapp
```

# DevContainer を起動
ポップアップが表示されるので、`Reopen in Container` を選択します
![reopen-in-container](./images/reopen-in-container.png)

DevContainerが起動するのを待ちます

# 動作確認
```sh
rails --version
```
`Rails 7.2.2` と表示されました。
2024/11/20 時点では rails-new は Rails 8.0.0 に対応していないようです。

rails 8.0.0 に対応する差分は下記のコミットをご参考ください。

https://github.com/tyamap/rails-new/commit/3c6dc71ae8e003682ac62ca83783599d1ce8c1e2

localhost も起動してみます
```sh
rails s
```

ブラウザで http://localhost:3000/ にアクセスしてみます
![rails-welcome](./images/rails-welcome.png)

無事アプリケーションを起動できました

# 参考
- https://railsguides.jp/getting_started_with_devcontainer.html