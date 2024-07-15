---
title: "Chrome拡張機能の開発をサクッと理解する"
emoji: "⚙"
type: "tech"
topics:
  - "chromeextension"
  - "chrome"
  - "web"
published_at: "2024-07-15 18:00"
publication_name: "irsc"
---
# はじめに
Chrome拡張機能、便利ですよね。日々ブラウザを使って行う作業が効率化されます。
今回はそんなChrome拡張機能を自分で作ってみたいとなった時に、知っておくと良い基礎知識を、ざっくりと紹介します。
詳細を知りたい方はコメントやSNSでお声がけください！

# 基本を理解する
下記の基本的な構造を紹介します。
- マニフェスト
- 3つのコンポーネント
- 権限とAPI
- その他主要な概念
## マニフェスト
すべてのChrome拡張機能が持っている拡張機能の設定や権限を定義するためのファイルです。
ここに下記のテンプレートやスクリプトにどのファイルを使用するかを記載します。

`manifest.json`
```json
{
  "version": "0.0.1",
  "author": "Author",
  "name": "Extension Name",
  "description": "Extension Desc",
  "icons": {
    "128": "icon128.png"
  },
  "manifest_version": 3,
  "action": {
    "default_popup": "popup.html"
  },
  "options_ui": {
    "page": "options.html"
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
主に3つのコンポーネントが拡張機能を構成しています。
- Content Script
- Background Script
- Action (Browser Action, Page Action)
![](/images/chrome_extension_image.png)
### Content Script
アクティブなタブ（画面）に対して挿入されるスクリプトです。
特定のURLパターンに一致するページが開かれるたびに実行されます。

特定のWebページでDOMを操作するなど、JSを実行したい場合は、コンテンツスクリプトを使用します。

```json
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
```

### Background Script
Chromeを起動している間ずっと動いているスクリプトです。
後述する Chrome API を利用できます。

拡張機能全体の管理とブラウザのAPIを扱うために仕様します。
例えば、ユーザーの操作に応じたイベント処理やデータの保存といった、持続的なタスクを担当します。

```json
"background": {
  "service_worker": "background.js",
},
```

### Action
Chrome右上に表示される拡張機能アイコンをクリックした時に実行する処理です。
アイコンクリックをトリガーにして、ポップアップを表示したり、スクリプトを実行したりします。
各種 Chrome API を利用できます。

例えば `popup.html` は、拡張機能のアイコンをクリックしたときに表示されるポップアップのHTMLです。
```hson
  "action": {
    "default_popup": "popup.html"
  },
```
## 権限とAPI
Chrome上のAPIや機能にアクセスするために、拡張機能に権限を付与する必要があります。
使用する権限は manifest.json に記載します。
下記のような機能・情報にアクセスできます。
- タブ
- ストレージ
- 履歴
- クリップボードなど

https://developer.chrome.com/docs/extensions/reference/permissions-list?hl=ja
## その他主要な概念
### Message Passing
Content Script, Background Script, Action の3つのコンポーネント間でのやり取りは、 Message によって行われます。
例えば下記のように実装します。

`ContentScript から BackgroundScript へメッセージを送信:`
```js
chrome.runtime.sendMessage({greeting: "hello"}, function(response) {
  console.log(response.farewell);
});
```

`BackgroundScript でメッセージを受信:`
```js
chrome.runtime.onMessage.addListener(function(request, sender, sendResponse) {
  console.log(request.greeting);
  sendResponse({farewell: "goodbye"});
});
```

挙動は以下のようになります。
1. ContentScript からのメッセージを受けて、 BackgroundScript 実行環境で `hello` と出力
2. BackgroundScript からのレスポンスを受けて、 ContentScript 実行環境（アクティブなページ）で `goodbye` と出力

詳細は下記をご参照ください。
https://developer.chrome.com/docs/extensions/develop/concepts/messaging?hl=ja

### Options
右上のアイコンを右クリックして、表示されるコンテキストメニュー内の「オプション」をクリックした時に遷移する画面のHTMLです。

```json
"options_ui": {
  "page": "options.html"
},
```

# フレームワークを用いて開発する
ここまでで `manifest.json` の仕様通りにスクリプトファイルを用意して実装していけば良いことがお分かりいただけたかと思います。
フルスクラッチで開発を始めることも簡単ではありますが、フレームワークライブラリを用いると、より開発体験は向上すると思います。
今回は Plasmo ライブラリをご紹介します。

## Plasmo
下記のような機能・サポートに魅力を感じる方におすすめです。

- No-Config
- JSX
- TypeScript
- tailwind
- Stripe 連携など

https://docs.plasmo.com/

他にも様々なライブラリやツールキットが公開されていますので、探してみてください！

# 配布する
[Chrome Web Store](https://chromewebstore.google.com/) に公開する方法と、自分のブラウザに直接取り込む方法があります。

## パッケージ化されていない拡張機能として読み込む
ストアに公開せずに、個人使用や社内利用する場合に使う方法です。
1. chrome://extensions/ にアクセス
2. 右上の デベロッパー モード をオンにする
3. **パッケージ化されていない拡張機能を読み込む** をクリック
4. 拡張機能のフォルダを選択

読み込んだ拡張機能のフォルダ内のファイルが更新されれば、その内容が反映されます。
個人でのデバッグや利用には向いていますが、
複数人に配布したい場合は、手間がかかります。

## ChromeWebStore で公開する
ストアに公開すれば、自動でアップデート配布されるなどのメリットがあります。
限定公開やグループ公開もできるので、社内利用であればこちらで問題ないと思います。
### 1. ChromeWebStore に登録
1. 規約とプラポリに同意
2. 登録料 $5 (2023年現在) を支払う
### 2. 拡張機能ファイルのアップロード
manifest.json がルートにある拡張機能のフォルダを、ZIP化してアップロードします。
### 3. 必要事項の記入
- 商品の詳細
- 画像アセット

など
### 4. 審査
初回の審査は数営業日で完了します。
その後のアップデート審査は数時間で完了する印象です

# おわりに
Chrome拡張機能は、基本的な HTML/CSS/JavaScript の知識と、上記のような概念の知識があれば簡単に作れそうだということがわかっていただけたかと思います。
まずは社内向けのツールの開発などで、拡張機能開発に挑戦してみてはいかがでしょうか！
