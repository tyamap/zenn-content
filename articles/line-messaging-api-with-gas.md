---
title: "LINE Messaging API へのメッセージ を GAS で受け取って処理する"
emoji: "💬"
type: "tech"
topics:
  - "LINE"
  - "bot"
  - "line-messaging-api"
  - "gas"
  - "google-script-app"
published: false
published_at: "2023-08-300 20:00"
---

## 何をするか
- LINE Messaging API の webhookイベントを GAS で受け付けて処理したい
- 今回は、処理日時とメッセージ送信者のUserID、メッセージ本文をスプレッドシートに記録する処理とする
## 前提
- LINE Messaging API を作成済み
- GAS をウェブアプリとしてデプロイ済み
- Messaging API の webhook エンドポイントをGASのデプロイURLに設定済み
## 流れ
- GAS の doPost 関数で POST を受け付ける
- イベントの種類を判別し、必要な情報を取り出す
- スプレッドシートに記述する
## スクリプト
```
const SPREAD_SHEET_ID = '{スプレッドシートのID}'
const SHEET_NAME = '{シート名}'

// 一行分のデータを受け取って、シート末尾に記録する
function addRecord(records = []) {
    const ss = SpreadsheetApp.openById(SPREAD_SHEET_ID); 
    const sheet = ss.getSheetByName(SHEET_NAME);
    // 最終行の一行下に追記
    const lastRow = sheet.getLastRow() + 1;
    const range = sheet.getRange(lastRow, 1, 1, records.length)
    range.setValues([records])
}

// POSTリクエストに対する処理
function doPost(e) {
  if (e == null || e.postData == null || e.postData.contents == null) return;

  // リクエストを受け取ってオブジェクト化
  const requestJSON = e.postData.contents;
  const requestObj = JSON.parse(requestJSON);

  // 以降は LINE Messaging API の仕様に準じた処理
  const events = requestObj.events;
  // events は配列で渡ってくるので、繰り返し処理する
  events.forEach((event) => {
    // メッセージイベントのみ受け付ける
    if (event.type !== 'message') return;
    const message = event.message;
    // テキスト入力のみ受け付ける
    if (message.type !== 'text') return;

    // 記録するデータを取得
    const datetime = new Date();
    const userId = event.source.userId;
    const text = message.text;

    const records = [datetime, userId, text];

    // スプレッドシートに記載
    addRecord(records);
  })
}
```
## 解説
基本的にはコメントにある通り。
下記は捕捉として。
### addRecord(records) 関数について
独自で定義する関数。
デバッグ目的で多用するため、関数として定義しておくと便利。
`range.setValues` は二次元配列を複数行のセルに記述する仕様なので、配列の形に注意。
今回は`record` 引数に一行分のデータとして一次元配列を受け取り、 `setValues` 実行時に二次元配列として渡すことにしている。
### doPost(e) 関数について
GAS をウェブアプリとしてデプロイしている場合、公開URLにPOSTリクエストが発生した際に実行される関数。
`e` 引数はリクエストのイベント情報。 
`e.postData.contents` でリクエストのコンテンツを取得できる。
JSONテキスト形式なので、 `JSON.parse()` でオブジェクト化しておく。
ウェブアプリとしてのGASや `e` のデータ構造の仕様については下記リンクを参照。
[ウェブアプリ  |  Apps Script  |  Google for Developers](https://developers.google.com/apps-script/guides/web?hl=ja)
### Messaging API webhook の JSON について
`requestJSON.events` が今回取り扱うデータ。
`requestJSON` の仕様については下記のリンクを参照。
[Messaging APIリファレンス](https://developers.line.biz/ja/reference/messaging-api/)
上記リファレンスにもある通り、 `events` は配列として渡ってくるので注意する。
## 動作確認
### 定数の設定
スクリプト冒頭の定数定義箇所に下記文字列を記述しておく。
`SPREAD_SHEET_ID`
データを記録したいスプレッドシートのURL末尾の文字列。
`SHEET_NAME`
データを記録したいシート名。スプレッドシート作成時の初期値は「シート1」
### LINE でメッセージを送信
LINE bot (Messaging API）アカウントにメッセージを送信する。
指定したシートの末尾に情報が記述されれば成功