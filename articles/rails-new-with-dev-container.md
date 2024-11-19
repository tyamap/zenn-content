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
- https://github.com/rails/rails-new?tab=readme-ov-file
- ローカルにrubyやrailsをインストールしなくても `rails new` できる
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
`Rails 7.2.2` と表示されました

localhost も起動してみます
```sh
rails s
```

ブラウザで http://localhost:3000/ にアクセスしてみます
![rails-welcome](./images/rails-welcome.png)

無事アプリケーションを起動できました
次回はこのプロジェクトを Rails 8.0.0 にアップグレードしてみます


# 参考
- https://railsguides.jp/getting_started_with_devcontainer.html