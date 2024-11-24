---
title: "rails-new と Dev Container で環境に依存しないプロジェクトを実現"
emoji: "💎"
type: "tech"
topics:
  - "rails"
  - "devcontainer"
  - "docker"
  - "rails-new"
published: true
published_at: "2024-11-24 18:30"
publication_name: "irsc"
---

# はじめに
今回は `devcontainer` と `rails-new` を使って rails プロジェクトを作成します。
`rails-new` は、rails のインストールやプロジェクトの作成をコンテナ内で行うことができるツールです。
`devcontainer` は、コンテナ内で開発環境を作成することができる機能です。
この二つにより、ローカルに rails をインストールしなくても rails プロジェクトを作成することができます。
複数環境下での環境構築や開発がスムーズに行えるようになり、開発体験の向上につながります。

# 準備
## Docker Desktop をインストール
コンテナの技術を使うので、Docker Desktop をインストールします。
[公式ドキュメント](https://docs.docker.com/desktop/) に記載されている、各環境向けの手順に従ってインストールしてください。

## VSCode をインストール
`devcontainer` は VSCode の拡張機能なので、VSCode をインストールします。
[公式サイト](https://code.visualstudio.com/)からダウンロードしてインストールしてください。

`devcontainer` 拡張機能を使えるエディタであれば、他のものでも大丈夫かと思いますが、本記事は VSCode での手順を説明します。

## Dev Containers 拡張機能をインストール
VSCode に Dev Containers 拡張機能を取得します。
[こちらのマーケットプレイス](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)からダウンロードしてください。

## rails-new をインストール
rails をインストールするには、ruby やDBクライアントをインストールする必要があります。
windows や macOS など環境によって手順に違いが出てきます。

`rails-new` を使えば、ローカルに ruby や rails をインストールしなくても `rails new` できます。

https://github.com/rails/rails-new

[最新のリリースノート](https://github.com/rails/rails-new/releases)から、パッケージをダウンロードします。

M1 製の Mac では、下記パッケージを選びましょう。
`rails-new-aarch64-apple-darwin.tar.gz`

Intel 製の Mac では、下記パッケージをダウンロードしました。
`rails-new-x86_64-apple-darwin.tar.gz`

任意のディレクトリにパッケージを配置して、PATH を通します。

下記は MacOS の場合の手順です。`Downloads` フォルダで解凍したパッケージを `usr/local/bin` に配置します。
```sh
mv ~/Downloads/rails-new /usr/local/bin/rails-new
which rails-new
# /usr/local/bin/rails-new
```

# rails-new を使ってプロジェクトを作成

`--devcontainer` オプションをつけて作成します。
今回は `myapp` というプロジェクト名で作成しますが、任意のプロジェクト名で良いです。

```sh
rails-new myapp --devcontainer
```

:::message
`rails-new` では `rails new` のオプションをそのまま指定できます。`--devcontainer` オプションもその一つです。
他にどのようなオプションがあるかは、下記コマンドで確認できます。
```sh
rails-new rails-help
```
:::

VSCode で `myapp` プロジェクトを開きます。
```sh
code myapp
```

# DevContainer を起動
VSCode で `myapp` プロジェクトを開くと、ポップアップが表示されるので、`Reopen in Container` を押下します。
![reopen-in-container](/images/open-in-devcontainer.png)

DevContainer が起動するのを待ちます。

# 動作確認
DevContainer が起動したら、VSCode内のターミナルで下記コマンドを実行して rails のバージョンを確認します。
```sh
rails --version
```
`Rails 7.2.2` と表示されました。
2024/11/20 時点では rails-new は Rails 8.0 に対応していないようです。

rails 8.0 に対応する差分は下記のコミットをご参考ください。

https://github.com/tyamap/rails-new/commit/3c6dc71ae8e003682ac62ca83783599d1ce8c1e2

ローカルサーバも起動してみます。
```sh
rails s
```

ブラウザで http://localhost:3000/ にアクセスしてみます
![rails-welcome](/images/rails-welcome.png)

無事アプリケーションを起動できました！

# まとめ
`rails-new` と `devcontainer` を使うことで、ローカルに rails をインストールしなくても rails プロジェクトを作成することができました。

# 参考
https://railsguides.jp/getting_started_with_devcontainer.html
