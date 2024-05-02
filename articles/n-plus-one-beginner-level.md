---
title: "【Ruby on Rails】N+1問題の基本と対策【初級編】"
emoji: "☔️"
type: "tech"
topics:
  - "Rails"
  - "SQL"
  - "DB"
published: true
published_at: "2024-01-27 18:30"
publication_name: "irsc"
---

# はじめに
Ruby on Rails における　N+1問題と対応方法の基本を解説します。

## TL;DR
- N+1問題の概要と基本の回避方法を解説します
- N+1問題とは、余分なクエリが大量に発生し、パフォーマンスに影響が出る状態
- `preload` や `joins` などのレコードをキャッシュする関数やテーブルを結合する関数を用いて回避する

## 解説しないこと
ActiveRecord のロード関数や結合関数がテーマですが、ActiveRecord::Relation のキャッシュやロードなど、詳細な仕様については解説しません。


## 例示用のモデル
今回例示用に用いるデータモデルの関連性は下記の通り。

`author.rb`
```ruby
class Author < ApplicationRecord
  has_many :books
end
```

`book.rb`
```ruby
class Book < ApplicationRecord
  belongs_to :author
end
```

:::message
Book レコードが 5つ以上存在するとする。
:::

# N+1問題とは
N+1問題とは、簡単にいうと「余分なクエリが大量に発生し、パフォーマンスに影響が出ている状態にあること」

例えばカレーを作る際にテーブルに材料を並べて行くとして、
必要な材料を集めるために、材料1つを手に入れる度にテーブルと冷蔵庫を毎回1往復する作業をするような非効率さを指す。
解決策としては、必要な材料を一括で把握し、冷蔵庫に一度アクセスして全部の材料を一度に取ってくればいい。

## N+1問題の例
N+1問題は、あるレコードの関連レコードの値を参照する処理を繰り返し実行した際に、
関連レコードを取得するためのSQLクエリが繰り返し発行されることで発生する。

例えば下記の処理を実行する。

```ruby
Book.limit(5).each do |book|
  book.author.id
end
```

発行されるSQLクエリは下記のようになる。

```ruby
  Book Load (0.7ms)  SELECT `books`.* FROM `books` LIMIT 5
  Author Load (0.4ms)  SELECT `authors`.* FROM `authors` WHERE `authors`.`id` = 1 LIMIT 1
  Author Load (0.3ms)  SELECT `authors`.* FROM `authors` WHERE `authors`.`id` = 2 LIMIT 1
  Author Load (0.3ms)  SELECT `authors`.* FROM `authors` WHERE `authors`.`id` = 3 LIMIT 1
  Author Load (0.3ms)  SELECT `authors`.* FROM `authors` WHERE `authors`.`id` = 4 LIMIT 1
  Author Load (0.3ms)  SELECT `authors`.* FROM `authors` WHERE `authors`.`id` = 5 LIMIT 1
```

5つの Book を取得し、それぞれの Book にひもづく Author テーブルのレコードを一つずつ参照して処理している。
Book 取得のクエリ1回と、それぞれの Author 取得のクエリが5回呼ばれている（N+1回）。

N+1問題は、関連レコードの値を参照したり、値を元に絞り込みを行うタイミングで起きる。

# 基本の回避方法
ActiveRecord にはこのN+1問題を回避するためのメソッドが用意されている。

基本的には前述した通り、「必要な材料を一括で把握し、冷蔵庫に一度アクセスして全部の材料を一度に取って」くるために、「関連データを結合して一括で取得」したり、「先に読み込んで（キャッシュして）おく」関数である。

:::message
**効率化の方法**
- 「キャッシュ」しておいた値を参照する
- 「テーブル結合」してリレーション間の絞り込みをする
:::

- `joins`
- `left_outer_joins`
- `preload`
- `eager_load`
- `includes`

使い方と特徴を解説する。

## joins
[joins | Railsドキュメント](https://railsdoc.com/page/joins)

関連レコードのテーブルを `INNER JOIN` で結合して取得する。
`INNER JOIN` によってテーブルが結合されるので、 `where` （WHERE句）による絞り込みが可能。

例えば下記の処理を実行する。

```ruby
Book.limit(5).joins(:author).where(author: { id: 1 })
```

発行されるクエリは下記のようになる。

```ruby
  Book Load (2.6ms)  SELECT `books`.* FROM `books` INNER JOIN `authors` `author` ON `author`.`id` = `books`.`author_id` WHERE `author`.`id` = 1 LIMIT 5
```

結合したテーブルを利用した絞り込みが行われるので、効率よくデータにアクセスできている。

一方、取得したデータの保持は行わないため、テーブルを跨いだレコードの値の参照は効率化できない。

```ruby
Book.limit(5).joins(:author).each {|book| puts book.author.id}
```

```ruby
  Book Load (0.8ms)  SELECT `books`.* FROM `books` INNER JOIN `authors` ON `authors`.`id` = `books`.`author_id` LIMIT 5
  Author Load (0.7ms)  SELECT `authors`.* FROM `authors` WHERE `authors`.`id` = 1 LIMIT 1
1
  Author Load (0.6ms)  SELECT `authors`.* FROM `authors` WHERE `authors`.`id` = 2 LIMIT 1
2
  Author Load (0.5ms)  SELECT `authors`.* FROM `authors` WHERE `authors`.`id` = 3 LIMIT 1
3
  Author Load (0.6ms)  SELECT `authors`.* FROM `authors` WHERE `authors`.`id` = 4 LIMIT 1
4
  Author Load (0.8ms)  SELECT `authors`.* FROM `authors` WHERE `authors`.`id` = 5 LIMIT 1
5
```

## left_outer_joins

[left_outer_joins | Railsドキュメント](https://railsdoc.com/page/left_outer_joins) 

関連レコードのテーブルを `LEFT OUTER JOIN` で結合して取得する。

`joins` の 左外部結合バージョンなので、説明は割愛。

## preload
[preload | Railsドキュメント](https://railsdoc.com/page/preload)

関連レコードを事前に取得し、キャッシュする。

例えば下記の処理を実行する。

```ruby
Book.limit(5).preload(:author).each { |book| puts book.author.id }
```

発行されるクエリは下記のようになる。

```ruby
  Book Load (0.7ms)  SELECT `books`.* FROM `books` LIMIT 5
  Author Load (0.5ms)  SELECT `authors`.* FROM `authors` WHERE `authors`.`id` IN (1, 2, 3, 4, 5)
1
2
3
4
5
```

1回目のクエリで取得した Book 情報をもとに、 2回目のクエリで Author テーブルから必要な情報を取得している。
事前に取得してキャッシュした情報をもとに処理を行うので、効率よく値を参照できている。

一方、テーブルの結合は行わないので、 `where` による絞り込みは不可。

```ruby
Book.limit(5).preload(:author).where(author: { id: 1 }).count
```

```ruby
(0.7ms)  SELECT COUNT(*) FROM (SELECT 1 AS one FROM `books` WHERE `author`.`id` = 1 LIMIT 5) subquery_for_count
ActiveRecord::StatementInvalid: Mysql2::Error: Unknown column 'author.id' in 'where clause'
```

取得した値に対して実行される `select (filter)` を用いれば絞り込みが可能。

```ruby
Book.limit(5).preload(:author).select { |book| book.author.id == 1 }.count
```

```ruby
  Book Load (1.1ms)  SELECT `books`.* FROM `books` LIMIT 5
  Author Load (0.8ms)  SELECT `authors`.* FROM `authors` WHERE `authors`.`id` IN (1, 2, 3, 4, 5)
1
```

## eager_load
[eager_load | Railsドキュメント](https://railsdoc.com/page/eager_load)

関連レコードのテーブルを `LEFT OUTER JOIN`` で結合して取得し、キャッシュする。値参照処理と絞込み処理におけるN+1問題の回避に用いる。

例えば下記の処理を実行する。

```ruby
Book.limit(5).eager_load(:author).where(author: { id: 1 })
```

発行されるクエリは下記のようになる。

```ruby
  SQL (0.9ms)  SELECT `books`.`id` AS t0_r0, `books`.`author_id` AS t0_r1, `books`.`title` AS t0_r2, `books`.`created_at` AS t0_r3, `books`.`updated_at` AS t0_r4, `author`.`id` AS t1_r0, `author`.`name` AS t1_r1, `author`.`created_at` AS t1_r2, `author`.`updated_at` AS t1_r3 FROM `books` LEFT OUTER JOIN `authors` `author` ON `author`.`id` = `books`.`author_id` WHERE `author`.`id` = 1 LIMIT 5
1
2
3
4
5
```

各カラムにはエイリアスが割り当てられ、 `LEFT OUTER JOIN` によってテーブルが結合されていることがわかる。テーブルが結合されているので `where` による絞り込みが可能。

さらに、取得した情報がキャッシュされているので、関連レコードの値を参する照処理も効率化される。

```ruby
Book.limit(5).eager_load(:author).each { |book| puts book.author.id }
```

```ruby
  SQL (0.9ms)  SELECT `books`.`id` AS t0_r0, `books`.`author_id` AS t0_r1, `books`.`title` AS t0_r2, `books`.`created_at` AS t0_r3, `books`.`updated_at` AS t0_r4, `author`.`id` AS t1_r0, `author`.`name` AS t1_r1, `author`.`created_at` AS t1_r2, `author`.`updated_at` AS t1_r3 FROM `books` LEFT OUTER JOIN `authors` `author` ON `author`.`id` = `books`.`author_id` WHERE `author`.`id` = 1 LIMIT 5
1
2
3
4
5
```

## includes
[includes | Railsドキュメント](https://railsdoc.com/page/includes) 

処理内容に応じて `preload` と `eager_load` の挙動を切り替える便利関数。

例えば単純な値参照処理の場合、

```ruby
Book.limit(5).includes(:author).each { |book| puts book.author.id }
```

発行されるクエリは下記のようになる。

```ruby
  Book Load (0.8ms)  SELECT `books`.* FROM `books` LIMIT 5
  Author Load (0.5ms)  SELECT `authors`.* FROM `authors` WHERE `authors`.`id` IN (1, 2, 3, 4, 5)
1
2
3
4
5
```

これは `preload` の挙動と同じである。データのキャッシュがされるが、テーブル結合はされない。

一方、 `where` による絞込みを行う場合、

```ruby
Book.limit(5).includes(:author).where(author: { id: 1 })
```

発行されるクエリは下記のようになる。

```ruby
  SQL (0.9ms)  SELECT `books`.`id` AS t0_r0, `books`.`author_id` AS t0_r1, `books`.`title` AS t0_r2, `books`.`created_at` AS t0_r3, `books`.`updated_at` AS t0_r4, `author`.`id` AS t1_r0, `author`.`name` AS t1_r1, `author`.`created_at` AS t1_r2, `author`.`updated_at` AS t1_r3 FROM `books` LEFT OUTER JOIN `authors` `author` ON `author`.`id` = `books`.`author_id` WHERE `author`.`id` = 1 LIMIT 5
```

これは `eager_load` の挙動と同じになり、テーブル結合とデータのキャッシュがされる。

# 比較と見極め
まとめると下記の通り。

| メソッド名 | 発行SQL | キャッシュ | 結合 |
| -------- | ------- | -------- | --- |
| `joins` | `INNER JOIN` | × | ○ |
| `left_outer_joins` | `LEFT OUTER JOIN` | × | ○ |
| `preload` | `SELECT` 句をモデル毎に1回ずつ | ○ | × |
| `eager_load` | `LEFT OUTER JOIN` | ○ | ○ |
| `includes` | 場合によって `preload` / `eager_load` 切り替え | ○ | 場合による |

キャッシュされるということはループ内で値を参照が効率化されるということ。
結合されるということはリレーション間の絞り込みをSQLクエリを発行する `where` 句が使えるということ。

:::message
**テーブル結合のコスト**

大量のデータに対して関連テーブルを結合すると、処理に時間がかかる。
結合前にできるだけデータを絞り込むなど工夫が必要。
:::

## `joins` vs `left_outer_joins`
内部結合か左外部結合か。
関連モデルのレコードが必ず存在するか、存在しない場合は結果から除外したい場合は、内部結合なので `joins` を用いる。
関連モデルのレコードが存在しないケースがあり得て、存在しないも結合元を含めたい場合は、左外部結合なので `left_outer_joins` を用いる。

## `joins (left_outer_joins)` vs `eager_load`
単純な絞り込み用途であれば `joins` を使う。`eager_load` はクエリが長くなる。
絞り込み結果の関連レコードの値を参照する必要がある場合は `eager_load` を使う。

## 使用する関数の見極め
`includes` は状況に応じて `eager_load` と `preload` を切り替えるものなので、次のように方針を決めておく。
- 明確に目的を持って実装するスタンスに立って、基本的に使わない
- 必要とする処理が変わる中で柔軟に挙動を変えていきたいので、積極的に使っていく

その上で、どの関数を使うかは、下記のフローチャートで考える。

![関数見極めフローチャート](https://storage.googleapis.com/zenn-user-upload/08aae7f0c609-20240127.png)

# まとめ
- N+1問題とは、余分なクエリが大量に発生し、パフォーマンスに影響が出る状態
- `preload` や `joins` などのレコードをキャッシュする関数やテーブルをJOINする関数を用いて回避する

次回は ActiveRecord:: Relation のソースコードを読みながら、キャッシュやロードについてより詳しく見ていく予定です！

# 参考
[Active Record クエリインターフェイス - Railsガイド](https://railsguides.jp/active_record_querying.html)
[Railsアプリケーションのパフォーマンス改善をしながらN+1問題を解決するスキルを身に付けよう！ | Techpit](https://www.techpit.jp/courses/79)

 