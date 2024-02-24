---
title: "【Ruby on Rails】N+1問題の基本と対策【中級編】"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - "Rails"
  - "SQL"
  - "DB"
published: false
---


# はじめに
Ruby on Rails における　N+1問題と対応方法の基本を解説します。
初級編はこちらをご覧ください。
https://zenn.dev/tyamap/articles/n-plus-one-beginner-level

## TL;DR

- 扱っているオブジェクトが何者なのかを意識しよう
- 特にループ処理や View 内の処理は気をつけよう
- ActiveRecord はロードとキャッシュを意識しよう
- ActiveRecord のソースコードを読もう

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

## ActiveRecord とは
Ruby on Rails における O/Rマッパー。MVC でいうところの M担当。
DBMS に依存しない形で、DB関連の下記機能を提供してくれる。

- レコードの作成・更新・削除
- レコードの取得
- マイグレーション
- バリデーション
- アトリビュート管理

:::message
**O/Rマッパーとは**
DBのデータと、各プログラムのオブジェクトとの互換性を上手いことやってくれるツール・ライブラリ・フレームワーク。
:::

## ActiveRecord::Relation とは
指定の条件に沿ってSQL文を作ってDBから読み取ってくる処理（レコードの取得）を担当するモジュール。
DB のレコードをオブジェクト指向的に扱えるものにしてくれる。

```ruby
books = Book.all
=> Book Load (0.9ms)  SELECT `books`.* FROM `books`

puts books.class
=> Book::ActiveRecord_Relation

puts books.class.superclass
=> ActiveRecord::Relation
```

様々なモジュールが組み込まれてる

- `ActiveRecord::FinderMethods`
- `ActiveRecord::Calculations`
- `ActiveRecord::SpawnMethods`
- `ActiveRecord::QueryMethods`
- `ActiveRecord::Batches`
- `ActiveRecord::Explain`
- `ActiveRecord::Delegation`

## とりあえず押さえておきたい ActiveRecord::Relation のアトリビュート
### `@loaded`
SQLクエリを発行して オブジェクトを取得したことがあるかのフラグ

```ruby
Book.all.loaded? => false
books = Book.all => Book Load (0.9ms)  SELECT `books`.* FROM `books`
books.loaded? => true
```

### `@records`
SQLクエリを発行して取得した条件に合うオブジェクトをメモ化した配列

```ruby
Book.all.records => [#<Book:...>, #<Book:...>, ...]
Book.all.records.class => Array
```

:::message
**Point: ロードされてるかされていないか。キャッシュを使うか使わないか。**

`records` メソッドは、
`loaded?` が false のときはクエリを発行して `@records` を取得して返し、
`loaded?` が true のときは `@records` をそのまま返す。

つまり `@records` はクエリしたオブジェクトのキャッシュとして扱われる。

ActiveRecord::Relation の関数の中には、 `records` を使う（キャッシュを使う）関数と、使わない関数がある。
:::

## キャッシュを使う関数

### records に delegate されているメソッド

[delegation.rb](https://github.com/rails/rails/blob/main/activerecord/lib/active_record/relation/delegation.rb)

```ruby
delegate :to_xml, :encode_with, :length, :each, :uniq, :to_ary, :join,
         :[], :&, :|, :+, :-, :sample, :reverse, :compact, :in_groups, :in_groups_of,
         :to_sentence, :to_formatted_s, :as_json,
         :shuffle, :split, :index, to: :records
```
これらは `records` に委譲されているので、 Array の関数として実行される。

### Enumerable のメソッド
[batch_enumerator.rb](https://github.com/rails/rails/blob/80bce4aad3b288a92745abb7c3d15e5bb8aa2c82/activerecord/lib/active_record/relation/batches/batch_enumerator.rb)

ActiveRecord::Relation は Enumerableをインクルードしているため、 `#map`, `#collect` などは、 Enumerable の関数として実行される。
つまり、全て `each` を用いた実装の関数なので、 Array の振る舞い。

#### #pluck
[calculations.rb](https://github.com/rails/rails/blob/80bce4aad3b288a92745abb7c3d15e5bb8aa2c82/activerecord/lib/active_record/relation/calculations.rb)

指定したカラムのみ取得して配列を返す関数。 指定したカラムを含んだ `records` があればそちらを使う。キャッシュがない場合、ロードによる更新をしない。

```ruby
# キャッシュがない場合の例
Book.limit(10).pluck(:id)
=>   (0.7ms)  SELECT `books`.`id` FROM `books` LIMIT 10

# キャッシュがある場合の例
books = Book.limit(10)
=>  Book Load (0.7ms)  SELECT `books`.* FROM `books` LIMIT 10
books.pluck(:id)
=> [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

#### #size
[relation.rb](https://github.com/rails/rails/blob/80bce4aad3b288a92745abb7c3d15e5bb8aa2c82/activerecord/lib/active_record/relation.rb)

```ruby
# Returns size of the records.
def size
  if loaded?
    records.length
  else
    count(:all)
  end
end
```

詳細は後述。

#### #first など
[finder_methods.rb](https://github.com/rails/rails/blob/main/activerecord/lib/active_record/relation/finder_methods.rb)

キャッシュがあればその配列に対してインデックスを指定して取得する。

:::message
ソースコード読むとたまにジョークも見つかるよ
[#forty_two](https://github.com/rails/rails/blob/c7551d08fc572655105da38394d8672b8259c22f/activerecord/lib/active_record/relation/finder_methods.rb#L287)
:::

## キャッシュを使わない関数
### #count
[calculations.rb](https://github.com/rails/rails/blob/80bce4aad3b288a92745abb7c3d15e5bb8aa2c82/activerecord/lib/active_record/relation/calculations.rb)

入力:
```ruby
books = Book.limit(10)
5.times do
  puts books.count
end
```
結果:

```ruby
  Book Load (0.8ms)  SELECT `books`.* FROM `books` LIMIT 10
   (0.5ms)  SELECT COUNT(*) FROM (SELECT 1 AS one FROM `books` LIMIT 10) subquery_for_count
10
   (0.3ms)  SELECT COUNT(*) FROM (SELECT 1 AS one FROM `books` LIMIT 10) subquery_for_count
10
   (0.2ms)  SELECT COUNT(*) FROM (SELECT 1 AS one FROM `books` LIMIT 10) subquery_for_count
10
   (0.3ms)  SELECT COUNT(*) FROM (SELECT 1 AS one FROM `books` LIMIT 10) subquery_for_count
10
   (0.3ms)  SELECT COUNT(*) FROM (SELECT 1 AS one FROM `books` LIMIT 10) subquery_for_count
10
```

全ての `count` 処理でSQLクエリが叩かれている。
これは、 `#count` 関数が `@records` キャッシュを使用しない 関数であるからである。

取得した `@records` を利用する関数である、 `#size` や `#length` を使うことで、無駄なクエリを防ぐことができる。

入力:
```ruby
books = Book.limit(10)
5.times do
  puts books.count
end
```
結果:
```ruby
  Book Load (0.8ms)  SELECT `books`.* FROM `books` LIMIT 10
10
10
10
10
10
```

`#size` と `#length` も仕様が異なるので注意する。

| メソッド名 | 内容 | キャッシュ |
| -------- | ------- | -------- |
| `count` | `COUNT` クエリで要素数を取得する | 使わない |
| `length` | キャッシュがなければ `SELECT *` で取得して   `Array#length` で要素を数える | 使う |
| `size` | キャッシュがなければ COUNT クエリで要素数を取得する  キャッシュがあれば `Array#length` で要素を数える | 使う |

:::message
Array `#count/#size/#length` と ActiveRecord `#count/#size/#length` は別物

`Array#size` と `Array#length` は同じ（エイリアス）。
`Array#count` は引数で条件を指定できる。

ActiveRecord における `#count`, `#size`, `#length` はそれぞれ別物。

`Book.all.method(:count).owner=> ActiveRecord::Calculations`
`Book.all.method(:size).owner => ActiveRecord::Relation`
`Book.all.method(:length).owner => ActiveRecord::Delegation`
:::

### 計算関数
[Active Record クエリインターフェイス - Railsガイド](https://railsguides.jp/active_record_querying.html#%E8%A8%88%E7%AE%97)

### クエリやロードするための関数
下記のようなSQLクエリを作る関数や、ロードするための関数は、キャッシュを更新するためのものなのでキャッシュを使わない。
`where`, `find`, `find_by`, `order`, …
`load`, `reload`, …

:::message
配列としての `@records` を使いたい場合は、 `records` や `to_a` でキャッシュを明示的に取得して使えば良い。
:::

## 気をつけたいこと、意識したいこと
### キャッシュを使うということは
情報が古くないか気を遣う必要あり
入力: 
```ruby
books = Book.all
books.each_with_index do |u,i|
  puts "count: #{books.count}, length: #{books.length}, size: #{books.size}"
  if i === 2
    Book.create!(author_id: 1, name: 'hoge')
  end
  if i === 4
    break
  end
end
```
結果:
```ruby
  Book Load (1.1ms)  SELECT `books`.* FROM `books`
  
   (0.5ms)  SELECT COUNT(*) FROM `books`
count: 60, length: 60, size: 60
   (0.3ms)  SELECT COUNT(*) FROM `books`
count: 60, length: 60, size: 60
   (0.2ms)  SELECT COUNT(*) FROM `books`
count: 60, length: 60, size: 60
  TRANSACTION (0.2ms)  BEGIN
  Book Create (1.5ms)  INSERT INTO `books` (`author_id`, `name`) VALUES (1, 'hoge')
  TRANSACTION (1.3ms)  COMMIT
   (0.4ms)  SELECT COUNT(*) FROM `books`
count: 61, length: 60, size: 60
   (0.5ms)  SELECT COUNT(*) FROM `books`
count: 61, length: 60, size: 60
```

### ループ処理で ActiveRecord 関数を使う際は注意
キャッシュ取得した ActiveRecord::Relation をループで回すのは（ループ内でキャッシュを使用する関数を使う限り）問題ないが、キャッシュを使わない関数や、アソシエーションをロードする処理（基礎編参照）をすると N+1 が発生する。ループ処理内で ActiveRecord::Relation 関数を使う際は気をつけよう。

```ruby
authors = Author.limit(10)
authors.each do |l|
  puts authors.size     # Author キャッシュされてるので問題なし
  puts l.books.size  # Book キャッシュされてないので毎回クエリ叩かれる
end 
```

[初級編](https://zenn.dev/articles/n-plus-one-beginner-level/)でも扱ったように、ちゃんと preload しようねという話

```ruby
authors = Author.limit(10).includes(:books)
authors.each do |l|
  puts authors.size     # Author キャッシュされてるので問題なし
  puts l.books.size  # Book キャッシュされてるので問題なし
end 
```

### ActiveRecord 関数と Array 関数の対応
レコードを更新する必要のない処理に関しては、 ActiveRecord の関数をつかわずに `#records` や `#to_a` で Array にした上で Array の関数で代替できないか考える。
今まで見てきたように ActiveRecord::Relation と Array の仕様は全く異なるので注意は必要だが、的確に使えれば N+1 を防げる。

| ActiveRecord | Array (Enumerable) |
| --- | --- |
| `where` | `filter`, `select`, `reject` |
| `find`, `find_by` | `find`, `detect` |
| `first`, `second`, `last` | `first`, `second`, `last` |
| `count` | `count`, `length`, `size` |

などなど…

`select` や `find` は ActiveRecord の関数としても定義されていて、引数にブロックを渡すかどうかで挙動が変わり、紛らわしいので注意。

:::message
扱っているオブジェクトが ActiveRecord かどうかを意識すると、N+1 を防げるだけでなく、例えば View 内で Model 的な処理を書いてしまうアンチパターンを防げますね。
:::

## まとめ
- 扱っているオブジェクトが何者なのかを意識しよう
- 特にループ処理や View 内の処理は気をつけよう
- ActiveRecord はロードとキャッシュを意識しよう
- ActiveRecord のソースコードを読もう

## 参考
- [Active Record クエリインターフェイス - Railsガイド](https://railsguides.jp/active_record_querying.html)
- [ActiveRecord::Relationとは一体なんなのか | ⬢ Appirits spirits](https://spirits.appirits.com/doruby/8831/?cn-reloaded=1)