---
title: "Ruby で HMAC-SHA256"
emoji: "🔏"
type: "tech"
topics:
  - "rails"
  - "ruby"
  - "認証"
  - "暗号化"
  - "hmac"
published: true
published_at: "2023-06-26 20:00"
---

# やりたいこと
Ruby で `HMAC-SHA256` のハッシュ値を生成したい

# 変数
> シークレットキー: "secret"
> ハッシュ化対象文字列: "payload"

# コード
```ruby
require 'openssl'

secret = 'secret'
payload = 'payload'

hash = OpenSSL::HMAC.hexdigest(
	OpenSSL::Digest.new('sha256'),
	[secret].pack('H*'),
	payload
       )
```

Rails の場合、 require 'openssl' は不要

# 解説
1. `OpenSSL::Digest.new('sha256')` を使用して `SHA-256` ハッシュオブジェクトを作成
2. `OpenSSL::HMAC.hexdigest` 関数に渡された秘密鍵とメッセージを使用して、`HMAC` を計算
3. 計算された `HMAC` は、16進数表現の文字列として返される

## `HMAC-SHA256` とは

`HMAC-SHA256` （`Hash-based Message Authentication Code with SHA-256`）は、暗号学的ハッシュ関数である `SHA-256` と、秘密鍵を使用してメッセージの認証情報（`MAC`）を生成するための認証プロトコル。

**`HMAC`:** ハッシュ情報を用いて認証情報を生成する認証方式
**`SHA-256`:** 256ビット（32バイト）の長さのハッシュ値を生成する関数

- 送信者は、メッセージに対して `HMAC-SHA256` 関数を適用し、その結果をメッセージと一緒に送信する
- 受信者は、同じ秘密鍵を使用して受け取ったメッセージに `HMAC-SHA256` 関数を適用し、生成された `MAC` と送信者から受け取った `MAC` を比較する
- もし `MAC` が一致しない場合、メッセージが改ざんされている可能性があるため、認証に失敗したと判断する

`HMAC-SHA256` は、`SHA-256` ハッシュ関数と秘密鍵を使用しているため、安全性が高いとされている。

### 参考
https://learn.microsoft.com/ja-jp/dotnet/api/system.security.cryptography.hmacsha256?view=net-7.0

## シークレットキーを16進数文字列にする理由

秘密鍵 (secret_key) をバイト配列として表現するため。
一般的に秘密鍵はバイトのシーケンスとして表現されるが、文字列として直接扱う場合には、16進数表現を使用する。
16進数表現に変換することで、バイト配列をより扱いやすい形式に変換することができる。