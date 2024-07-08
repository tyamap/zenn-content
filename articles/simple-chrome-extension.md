---
title: "Chrome拡張機能の開発をサクッと理解する"
emoji: "⚙"
type: "tech"
topics:
  - "chrome-extension"
  - "chrome"
  - "web"
published: false
---
# 基本を理解する
下記の基本的な構造を紹介します。
- マニフェスト
- 3つのコンポーネント
- 権限とAPI
- その他主要な概念
## マニフェスト
すべてのChrome拡張機能が持っている拡張機能の設定や権限を定義するためのファイル。
ここに下記のテンプレートやスクリプトにどのファイルを使用するかを記載する。

`manifest.json`
```json
{
  "version": "0.0.1",
  "author": "Author",
  "name": "Extension Name",
  "description": "Extension Desc",
  "icons": {
    "128": "icon128.png",
    ...
  },
  "manifest_version": 3,
  "action": {
    "default_popup": "popup.html"
  },
  "options_ui": {
    "page": "options.html",
  },
  "permissions": [
    "storage",
    "activeTab"
  ],
  "content_scripts": [
    {
      "matches": [
        "<all_urls>"
      ],
      "js": [
        "content-script.js"
      ],
    }
  ],
}
```
## コンポーネント
### Content Script
アクティブなタブに対して挿入されるスクリプト。
特定のURLパターンに一致するページが開かれるたびに実行される。
### Background Script
Chromeを起動している間ずっと動いているスクリプト。
後述のChromeのAPIを利用できる。
### Action (Browser Action, Page Action)
Chrome右上に表示される拡張機能アイコンをクリックした時に実行する処理。
アイコンクリックをトリガーにして、ポップアップを表示したり、スクリプトを実行したり。
BackgroundScript と同じく各種Chrome APIを利用できる。
#### Popup
アイコンをクリックしたときに表示されるHTML
![](/images/chrome_extension_image.png)
## 権限とAPI
Chrome上のAPIや機能にアクセスするために、拡張機能に権限を付与する必要がある。
使用する権限は manifest.json に記載する。
下記のような機能・情報にアクセスできる
- タブ
- ストレージ
- 履歴
- クリップボードなど

https://developer.chrome.com/docs/extensions/reference/permissions-list?hl=ja
## その他主要な概念
### Message Passing
Content Script, Background Script, Browser Action の3つのコンポーネント間でのやり取り。
https://developer.chrome.com/docs/extensions/develop/concepts/messaging?hl=ja
### Options
右上のアイコンを右クリックして、行事される「オプション」をクリックした時に遷移する画面のHTML
# フレームワークで開発する
## Plasmo
https://docs.plasmo.com/
- TypeScript
- React.js
- tailwind
- Stripe 連携など
# 公開する
## パッケージ化されていない拡張機能として読み込む
ストアに公開せずに、個人使用する場合に使う方法です。
1. chrome://extensions/ にアクセス
2. 右上の デベロッパー モード をオンにする
3. **パッケージ化されていない拡張機能を読み込む** をクリック
4. 拡張機能のフォルダを選択

更新のたびに繰り返す必要があります。
## ChromeWebStore で公開する
### 1. ChromeWebStore に登録
1. 規約とプラポリに同意
2. 登録料を支払う（5ドル）
### 2. 拡張機能ファイルのアップロード
基本的には manifest.json が入った ZIP があれば良い
### 3. 必要事項の記入
- 商品の詳細
- 画像アセット

など
### 4. 審査
初回の審査はだいたい数営業日で完了する
その後の更新は数時間で完了する印象