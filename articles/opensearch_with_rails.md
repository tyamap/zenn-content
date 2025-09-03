---
title: "全文検索導入に向けた設計と実践ガイド【Rails×OpenSearch】"
emoji: "🔍"
type: "tech"
topics:
  - "rails"
  - "ruby"
  - "全文検索"
  - "opensearch"
publication_name: "medley"
published: true
published_at: "2025-09-04 10:30"
---

こんにちは！ 株式会社メドレーでエンジニアをしています、山下です。
本日は医療介護の求人サイト「ジョブメドレー」での全文検索エンジン導入に際して、調べたことや学んだことを紹介したいと思います！

この記事は [MEDLEY Summer Tech Blog Relay](https://developer.medley.jp/entry/2025/08/15/20250815/) 9日目の記事です。
https://developer.medley.jp/entry/2025/08/15/20250815/

# はじめに

webアプリケーションの開発をしていると、「検索機能を充実させたい」という要求に直面することがよくあると思います。

「ジョブメドレー」での全文検索機能導入経験をもとに、全文検索エンジンの導入に際して、段階的な選択肢と判断基準をまとめました。

Rails と OpenSearch の技術スタックを前提にしていますが、他の構成でも通用すると思いますので、ぜひ参考にしていただければと思います！

# 検索機能の段階的実装

## 1. 基本的なLIKE検索

まずはミニマムに取り入れられる、SQLのLIKE検索について見てみます。

```ruby
class Post < ApplicationRecord
  scope :search, ->(query) { where("title LIKE ?", "%#{query}%") }
end

# 生成されるSQL
SELECT * FROM posts WHERE title LIKE '%キーワード%';

# 正規表現での検索も可能
SELECT * FROM posts WHERE title REGEXP '^[A-Za-z]+$';
```

この手法は実装が簡単で、小規模なデータであれば十分に機能します。ただ、スケールにあたっては課題があります。

- **インデックスが効かない**: 基本的にフル探索となってインデックスを活用できない
- **パフォーマンス**: データ量が増えると顕著に遅くなる
- **機能に限界がある**: 関連度順の並び替えや複雑な検索条件に対応できない

### 適用場面

- 単純な部分一致検索で十分な場合
- データ量・レコードあたりの文字数が少ない場合
- 既存システムに最小限のコストで検索機能を追加したい場合

## 2. PostgreSQLの全文検索機能

PostgreSQLには強力な組み込み全文検索機能があります。RDB内で完結できるため、システム構成をシンプルに保てるのが大きなメリットです。

https://www.postgresql.org/docs/current/textsearch.html

```sql
-- textsearch モジュールを利用してインデックスを作成
CREATE INDEX idx_posts_content 
ON posts USING gin(to_tsvector('english', content));

SELECT * FROM posts 
WHERE to_tsvector('english', content) 
  @@ to_tsquery('english', 'Rails');
```

標準で `tsvector/tsquery` が使用できるようですが、日本語の検索に対応する場合は、以下のようなプラグインが必要になるようです。

- **pg_trgm**: トライグラム（3文字）ベースの検索
- **pg_bigm**: バイグラム（2文字）ベースの検索
- **pgroonga**: Groongaベースの高機能検索

詳しくは下記の Zenn @dyoshikawa さんの事例をご参照ください！

https://zenn.dev/team_zenn/articles/zenn-search-tuning-story

### 適用場面

- PostgreSQLを既に使用しているプロジェクト
- システム構成をシンプルに保ちたい

## 3. 全文検索エンジンの導入

より高度な検索要件や大規模データに対応するには、全文検索エンジンが威力を発揮します。
今回の記事のメインの内容になるので、詳しく見てみましょう。
全文検索エンジンにもいろいろありますが、今回は Elasticsearch や OpenSearch を例に見てみましょう。

### 基本的な仕組みと機能

#### 転置インデックス（Inverted Index）

従来のDBが「レコード → 内容」でデータを保持するのに対し、全文検索エンジンは「トークン → 出現するレコード一覧」という転置した構造でインデックスを作成します。

- 通常のDBインデックス:
    - レコード1: "Rails アプリケーション開発"
    - レコード2: "Ruby on Rails の基礎"
    - レコード3: "JavaScript フロントエンド"
- 転置インデックス:
    - "Rails": `[レコード1, レコード2]`
    - "アプリケーション": `[レコード1]`
    - "Ruby": → `[レコード2]`
    - "JavaScript": → `[レコード3]`

この構造により、例えば「Rails」を含む全てのドキュメントをすぐに特定できます。

#### トークナイザーとアナライザー

文書を検索可能な単位に変換する処理です。

- **トークナイザー**: 文書を単語（トークン）に分割
    - `"Railsアプリケーション開発"`
        - → `["Rails", "アプリケーション", "開発"]`
    - 単語の判別には辞書ファイルを用いる
    - 要件に応じて固有名詞や専門用語に特化した辞書を選択する
- **アナライザー**: トークナイザー + 各種フィルター
    - 大文字小文字統一、全角半角統一
    - 「の」「は」「を」などの除去
    - 「running」→「run」のような語幹抽出
    - 「サーバー」「サーバ」「鯖」を同じ語として扱う 

英語であればスペース区切りで単語単位に分けられるのでシンプルですが、日本語は単語間にスペースがないため、形態素解析やN-gramという手法を用います。

- **形態素解析**:
文を解析し、辞書に定義された単語で分割する方法です

```
"Rails アプリケーション開発" → ["Rails", "アプリケーション", "開発"]
```
- **N-gram**:
文字数単位で機械的に分割する方法です

```
バイグラム（2文字）: "アプリケーション" → ["アプ", "プリ", "リケ", "ケー", "ーシ", "ショ", "ョン"]
```
#### 分散処理

大規模なデータを効率的に処理するため、データを複数のシャードに分割し、複数のノードに分散配置します。

**検索処理の分散実行**

複数のシャードで並列で検索を行うことで、大量のデータを効率よく検索できます。

1. クエリ受信 → コーディネートノードが全シャードに転送
2. 各シャードで並行検索実行
3. 各シャードの結果をマージ・ソート
4. 最終結果をクライアントに返却

**レプリカによる冗長化**

データの格納場所を検索時にメインで使用するプライマリシャードと、コピーを保持するレプリカシャードに持つことで、障害時のデータ破損やダウンタイムを最小化します。

これらの分散アーキテクチャの仕組みにより、下記が実現します。

- **負荷分散**: 検索リクエストを複数ノードで分担
- **高可用性**: ノード障害時も他ノードやレプリカでサービス継続
- **スケーラビリティ**: ノード追加で性能向上

### 適用場面

他にもあいまい検索やスコアリングなど、検索体験を向上させる様々な機能が提供されています。

- 複雑な検索要件
- 大量データを扱う
- 検索体験を重視するプロダクト

# Elasticsearch と OpenSearch

少し前まで、全文検索といえば [Solr](https://solr.apache.org/) や [Elasticsearch](https://www.elastic.co/jp/elasticsearch) がデファクトスタンダードだったと思います。
2021年に、[OpenSearch](https://opensearch.org/) という OSS プロジェクトが、Elasticsearch からフォークして誕生しました。
少しこの辺りの経緯もさらっておきましょう。

```
Apache Lucene（Java製全文検索ライブラリ）
├── Apache Solr
└── Elasticsearch
    └── OpenSearch
```

**2021年**

- Elasticsearch がライセンスを SSPL + Elastic License に変更
- 実質的な非OSS化により、AWS等のクラウドベンダーが影響を受ける
- AWS が Elasticsearch 7.10.2 からフォークして OpenSearch を開始

**2024年**

- Elasticsearch が AGPL v3 ライセンスを追加し、再びOSS化
- OpenSearch は Linux Foundation の管轄下に移行、オープンガバナンス体制を確立

このような経緯は Redis → Valkey の件とも似ており、オープンソースソフトウェアと大手クラウドベンダーの関係について考えさせられる出来事でした。

#### OpenSearchを選ぶべきケース

- **オープンガバナンス**: コミュニティ主導の開発を重視
- **エコシステム**: Amazon OpenSearch Service/Serverless との親和性

#### Elasticsearchを選ぶべきケース

- **エコシステム**: Kibana、Logstash等のElastic Stack, Elastic Cloud との統合

# OpenSearch クライアントGem比較

弊社ではAWSを利用しており、他プロダクトで OpenSearch Service の運用実績もあったことから、OpenSearchを採用しました。
アプリケーションは Rails で実装していますので、Rails アプリケーションに OpenSearch を導入するにあたり、どの Gem を使用するか、比較しました。

## 1. opensearch-ruby

https://github.com/opensearch-project/opensearch-ruby

公式のクライアントで、最も低レベルなAPIを提供しており細かい制御が可能です。

```ruby
client = OpenSearch::Client.new(host: 'localhost:9200')

# インデックス作成
client.index(index: 'posts', body: { title: 'Hello World' })

# 設定・マッピング更新
client.indices.put_mapping(
  index: 'posts',
  body: { properties: { title: { type: 'text' } } }
)

query = { 'size': 5, 'query': { "match": { "title": "Rails" } } }
response = client.search(
  body: query,
  index: 'posts'
)
```

#### メリット

- コントロールできる範囲が広い

#### デメリット

- 実装コストが高い
- ActiveRecordとの連携等は自前で構築

## 2. elasticsearch-rails

https://github.com/elastic/elasticsearch-rails

Rails統合でActiveRecordと連携するインターフェースを提供します。
ElasticSearch公式のRails統合Gemですが、OpenSearchでも今のところ動作します。

```ruby
# Model
class Post < ApplicationRecord
  include Elasticsearch::Model
  
  settings index: { number_of_shards: 1 } do
    mappings dynamic: 'false' do
      indexes :title, type: 'text'
    end
  end
end

# インデックス作成・更新
Post.create_index! force: true
Post.first.import

Post.search("Hello")
```

#### メリット

- ActiveRecordライフサイクルとの自動連携
- Elasticsearchからの移行コストが低い

#### デメリット

- OpenSearchとの将来的な互換性に不安
- Elasticsearch前提の設計

## 3. Searchkick

簡単に全文検索を導入できる統合Gemです。

https://github.com/ankane/searchkick

```rb
# Model
class Post < ApplicationRecord
  searchkick mappings: {
    properties: { title: { type: 'text' } }
  }
end

# インデックス作成・更新
Post.reindex
Post.first.reindex

Post.search("Hello")
```

#### メリット

- 最小の設定で高度なActiveRecord連携ができる
- Railsらしい直感的なAPI
- 便利な機能を提供

#### デメリット

- 暗黙的な挙動による予期しない動作
- 細かいカスタマイズが難しい場合がある

# 構成と運用の検討

下記のAWSのドキュメントを参考に、サーバスペックや運用の方法について考えていきましょう。

https://docs.aws.amazon.com/ja_jp/opensearch-service/latest/developerguide/bp.html

## 1. クラスタ構成とスペック選定

### ノード構成

コストとパフォーマンスに大きく影響します。専用マスターノードを使用するかなど、要件に合わせてノードの数やそれぞれのスペックを検討しましょう。
また、シャード数とその配置方法に応じて必要なノード数は異なります。

https://dev.classmethod.jp/articles/opensearch-design-pattern/

### シャード数

データサイズやvCPU数、メモリ容量に応じて、適切なシャード数を設計する必要があります。
検索レイテンシーを意識する場合、1シャードのサイズを 10～30 GiBに保ちます。
また、可用性の要件に応じてレプリカの個数も検討しましょう。

#### シャード数が多すぎる場合

- オーバーヘッドでパフォーマンス低下
- メモリ使用量増加

#### シャード数が少なすぎる場合

- 単一ノードがボトルネックに
- 分散処理の恩恵を受けにくい
### スペック選定の指針

データサイズや可用性の要件を踏まえ、ハードウェアリソースの選定をしていきます。
ストレージ要件の容量 100 GiB ごとに vCPU x 2 コア、メモリ 8 GiB に近い構成を目安に考えていきますが、実際のリクエスト数や処理要件によって、適切なテストが必要になります。

https://aws.amazon.com/opensearch-service/pricing/

#### CPU

- コア数は一度に制御できるクエリ数に影響
- 分散処理の性能にも影響
- インデックス更新が多い場合は多めに確保

#### メモリ

- シャード数に応じて容量が必要
- JVMメモリの負荷が一定割合を超えるとGCにより速度が落ちる

#### ストレージ

- **インデックスサイズ**: 元データの20-50%程度
- **I/O性能**: データの更新/検索頻度に応じて設定
- **レプリカ**: プライマリ + レプリカでサイズは2倍


## 2. インデックス管理

### データ同期方法の検討

既存のwebアプリケーションに全文検索を導入する場合、メインのデータはRDBで管理しているケースがほとんどだと思います。
RDBと全文検索エンジンのデータをどのように同期するかは重要になると思います。

検索機能の要件やデータの種類・使用頻度に応じて、同期の方法やタイミングを設計しましょう。

- ActiveRecord のコールバックで更新
- 一定のペースでバッチ処理

### リインデックス方法の検討

インデックスの設定を変更したり、マッピングを変更した場合、それを全てのドキュメントに反映するために全てのデータのリインデックスをする必要があります。

インデックスにエイリアスを設定して、新しいエイリアスでデータをインデックスし、参照先を切り替えることでダウンタイムを抑えたリインデックスが可能です。

## 3. モニタリングとアラート

#### 主要メトリクス

- **Cluster Health**: Green/Yellow/Red
- **Node負荷**: CPU・メモリ・ディスク使用量
- **検索性能**: レスポンス時間・スループット
- **インデックス**: サイズ・ドキュメント数

#### アラート設定例

- Cluster状態がYellowになった場合
- 検索レスポンスが3秒を超える場合
- JVM負荷平均が75%を超える場合

## 4. その他気をつけたこと

- **要件駆動**: 現在の要件に対してオーバーエンジニアリングしない
- **段階的移行**: 一気に移行せず、段階的に機能を拡張

# 最後に

検索機能はユーザー体験を大きく左右する重要な機能です。
全文検索機能の実装にあたっては複雑な構成や設計で考慮すべきことも多いですが、実現できると高いパフォーマンスと機能により、検索体験が向上する恩恵も大きくなると思います。
ビジネス要件と技術的制約のバランスを取りながら、最適な選択をしていきましょう！

明日の [MEDLEY Summer Tech Blog Relay](https://developer.medley.jp/entry/2025/08/15/20250815/) 10日目は医療プラットフォームの井津さんです！ お楽しみに！

# We’re hiring
最後まで読んでいただきありがとうございます！
メドレーでは、一緒に働く仲間を大募集中です
少しでも興味を持っていただけましたら、ぜひカジュアル面談でお待ちしています！

https://www.medley.jp/jobs
https://pitta.me/uratotsu/medley
