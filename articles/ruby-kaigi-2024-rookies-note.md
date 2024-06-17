---
title: "RubyKaigi 2024 初参加の記録"
emoji: "💎"
type: "tech"
topics: ['ruby', 'rubykaigi', 'rubykaigi2024', 'rails']
published: false
---

沖縄で行われた [RubyKaigi 2024](https://rubykaigi.org/2024/) が、私の初 RubyKaigi でした！  
初参加かつ、単身での参加かつ、他の技術カンファレンスにまともに参加したこともなかったので、右も左もワカラナイ状態でしたが、そんな中で学んだことや、各セッションのメモを記していきます。  
ほぼ誰の役にも立たないものですが……  
RubyKaigi未参加の方や初参加の方の参考になると嬉しいです  
間違ってたり補足が必要な点が多いと思うので、ぜひご意見ご指摘あればコメントください！

# Tips
今回のカンファレンス参加で学んだことを書いていきます。
他の技術カンファレンスにも適用できるTipsだと思います。
## 早起きは三文の徳
Early bird チケットは半額くらいで購入できます。
イベントの参加登録もすぐ埋まるので早めに済ませておきましょう。早く行動するために、次の「常にアンテナを張る」が大事です。
## 常にアンテナを張る
チケットの販売、追加情報、イベントの開催情、ブースの情報、ランチの場所やおすすめ観光スポットに至るまで、 随時アナウンスされます。X（旧twitter）やメルマガ、Discord をウォッチしておきましょう。公式ノベルティも地味に配布されるし、休憩時間の弁当や軽食の配布もシレッと行われる。初日の公式パーティへはバスが出たけど気づかない人もいたらしい。周りをよく見て、人の流れを読む
## 前提知識が命
何よりもこれが一番大切でした。事前勉強、予習必須です。
Keynote以外のセッションは短く、前提となる話は省略されます。セッション後の質問なども受け付けていません。直近のバージョンアップにより追加された機能や、前回のKaigiで話題になったテーマなどをキャッチアップしておきましょう
## 気になるコミッターをウォッチ
スピーカーや、コミッターの情報もチェックしておきましょう。好きなライブラリのコミッターの顔と名前を覚えておくと良いです。会場やパーティの場にいるので、直接話ができるかも。なおスピーカーは発表が終わるまでは非常に緊張していらっしゃる
## 身軽さが正義
セッションやブースを巡るために会場を何往復もします。会場では仕事用の16-incMacBookProは開けず、私物のiPadが大活躍。大きいPCは飛行機の荷物の重量制限も圧迫する。MacbookAirがほしい。ノベルティも大量にもらうので、荷物には余裕を持って行きましょう。
## 英語力は必要
3分の2が英語セッション、日本語スピーカーもスライドは英語表記です。
リアルタイム自動翻訳のツールを準備しておくか、リスニング力で乗り切ろう

---
# 1日目
## Writing Weird Code
[Writing Weird Code - RubyKaigi 2024](https://rubykaigi.org/2024/presentations/tompng.html#day1)

1日目の Keynote
短くてトリッキーなコードの紹介
Rubyにおける文字リテラルや演算子のフシギ
- `%?+-`
- `<<HEREDOC`
- UTF-8 の絵文字
	- #️⃣ はコメントとして使える、など

**ひとりTRICK 開催！**
https://github.com/tompng/selftrick2024

### 感想
- 6作品もの素晴らしいアートプログラムに圧巻……
- それぞれで使用している技術が RubyKaigi の各セッションのテーマの布石になっていて、初日の Keynote として非常に完成度が高い……
- 言ってることはわかる（？）けどやってることがすごすぎる……

### キーワード
- ./#DSL
- ./#Quine
- ./#TRICK

### 資料
https://drive.google.com/file/d/1Dkx15u_5UAGoFqJHCeAuj2FXS-z_U7EE/view

https://x.com/tompng/status/1792482001452425444


## Unlocking Potential of Property Based Testing with Ractor
https://rubykaigi.org/2024/presentations/ohbarye.html#day1
### PBT gem の紹介
https://github.com/ohbarye/pbt
#### 設計
Arbitrary
Predicater テストの性質
#### 特徴
- シード値を持つことで、再現性を持たせる
- verbosely Mode で実行ログを出力する
	- どのような値でテストしてShrinkしたかわかるので楽しい
### Ractor を使用してPBT処理を改善しようとした話
#### 学び
- gem開発の中で苦戦した点を説明
- Ractorを使うことの難しさを感じた
---
### 感想
- PBTの概念は面白そう。実務で使えるだろうか。どのような実装のテストに向いているだろうか
- Ractor を使いこなすのは難しそう
- TODO: 結局 Ractor では効果が出なかった原因は何にあるのだろう（聞き逃している）
### キーワード
- #PBT
- #Ractor
### 資料
https://speakerdeck.com/ohbarye/unlocking-potential-of-property-based-testing-with-ractor

https://x.com/ohbarye/status/1790618995172979025

## Long journey of Ruby standard library
https://rubykaigi.org/2024/presentations/hsbt.html#day1
### Ruby Standard Library
C拡張で実装されているライブラリも多い
今は rubygems.org が gem のホスティングサービス
#### default gems
stdgems.org 
`loaded_specs["rss"].default_gems?`
defaultgems の変更はruby 本体にコミットされる
#### bundled gems
https://github.com/ruby/ruby/blob/master/gems/bundled_gems
defaultgemsをこちらに移していく作業をしている
- 可能な限りソフトウェアのライブラリに対する依存度を低くして、脆弱性を低くしたい
- gems のコミット、更新を促進したい
### Ruby 3.3.0 の警告機能
default gems は gemfile に書かなくても使える
ruby のアップデートによって、default gems から外れるケースがある
その警告を出してくれる
依存関係も抽出して警告。依存関係は少ない方がいい
ruby バージョンアップによってDGから外れることをPR出して修正する活動をしてほしい
### 今後
bunled gems を多くしていきたい
default gems は少なくなる
C拡張が難しい
#### Cross platform Problem
WASM対応で苦戦
C拡張の区別、棲み分け
namsespace が待ち望まれている
C拡張をrubyで書き直す作業をやっている
https://github.com/ruby/ruby/blob/master/tool/sync_default_gems.rb

---
### 感想
- Gem の種類を意識したことがなかった
- すごく骨が折れそうな作業。少しでも何か貢献できるといいな……
- 警告機能を活用して、OSSライブラリに Issue や PR 出したい
### キーワード
- ./#Ruby Standard Library
- ./#C拡張
### 資料
https://speakerdeck.com/andpad/long-journey-of-ruby-standard-library-rubykaigi-2024

https://x.com/hsbt/status/1790936604108111900

## Namespace, What and Why
https://rubykaigi.org/2024/presentations/tagomoris.html#day1
app/libを隔離する仕組み
ライブラリを複数のバージョンで使用すると競合する
https://bugs.ruby-lang.org/issues/19744
Namespace#requireがライブラリを分離された領域にロードする
### Why
名前と定義とバージョンの衝突を差ける
ライブラリの依存するライブラリのバージョンが異なる問題も解決
### Approach
- OnWrite
	- 定義する側で、名前空間を宣言する
- OnRead
	- 標準機能としての名前空間
### How
ごにょごにょ
TODO: 理解して補足する
### Where to go
以下はあくまで構想
- ひとつのruby環境で複数のアプリケーションを動かす
- 依存関係地獄からの脱却
- "Packages" APIの実現
---
### 感想
- 複数のバージョン取得できたり依存関係地獄から脱却できるのはいいな
- 他のパッケージシステムはどうなってるかな
- この辺りから、スライドの内容に頭が追いついていない……
### キーワード
- ./#名前空間
- ./#ライブラリとパッケージシステム
### 資料
https://speakerdeck.com/tagomoris/namespace-what-and-why

https://x.com/tagomoris/status/1790914708696170986


## Exploring Reline: Enhancing Command Line Usability
https://rubykaigi.org/2024/presentations/ima1zumi.html#day1
### IRBとReline
IRB: カラーリング、補完、ドキュメント表示
Reline: 表示を担当
### Reline
コマンドラインエディタ
ない場合、カーソル移動などが効かない
ダイアログ機能、マルチライン対応、Rubyで拡張可能
GNU Readline との互換性を目指している
いくつか機能が不足している
### GNU Readline 
-  行編集
- 履歴機能
- 自動補完
- 編集コマンド
- マクロなど
### undo 機能を作りました
操作の履歴を保持
Undo時に履歴を取り出す
https://github.com/ruby/reline/pull/701
### 今後
- Redo機能
- ドキュメント充実
---
### 感想
- わかりやすい！便利！
- 自分が使っているツールの話を聞けて、やっと緊張が解ける
- 自分も頑張りたいなとコントリビューションのモチベが湧いた
### キーワード
- ./#IRB
- ./#Reline
- ./#Readline
### 資料
https://speakerdeck.com/ima1zumi/exploring-reline-enhancing-command-line-usability

https://x.com/ima1zumi/status/1790650187108765911

## Refactoring with ASTs and Pattern Matching
https://rubykaigi.org/2024/presentations/keystonelemur.html#day1
### AST
抽象構文木
ソースコードの構造を表現する木構造
### Parser
それぞれの特徴を解説……
- Ripper
- WhitequarkParser 
- PrismParser

Ease of use is Critical
Precision / Close enough
### PatternMatching
特定の構造やパターンをコード内で検出し、それに基づいて処理を行う
### Future
ASTとパターンマッチングを用いてこんなことができるよ〜
 - WebTool Rebulor
 - Upgrading Gem
 - A to B Refactor
---
### 感想
- 英語だ……Parserだ……ナンモワカラン
- ASTとか初めて聞いた。文系ファジーエンジニアには難しい……勉強しよ
- このあと、いろんなセッションで Parser の話を聞くことになる、その布石
### キーワード
- ./#AST
- ./#Parser
- ./#PatternMatching
### 資料
https://github.com/baweaver/kaigi24_refactoring_talk
---
## Leveraging Falcon and Rails for Real-Time Interactivity
https://rubykaigi.org/2024/presentations/ioquatix.html#day2

### 通信の歴史
1972 BBSの誕生
https://ja.wikipedia.org/wiki/電子掲示板
1996: Earth2025 webブラウザゲーム
2005: Railsの誕生
「設定よりも規約」「リクエストレスポンスもえる」「リアルタイムのインタラクティブ性の欠如」
リアルタイムの対話を実現する WebSocket＋HTML5
2007：Asyncの誕生
Async以前: でかいリクエストが来ると停滞してしまう
遅いリクエストがあっても、他のFiberが応答してくれるので安心
Async vs  Ractor
https://rubygems.org/gems/async-container
ActionCable
https://gigazine.net/news/20120328-mozilla-browser-quest/

https://bugs.ruby-lang.org/issues/19078
### Falcon
async gem を使った web server
https://rubygems.org/gems/falcon
Live
https://rubygems.org/gems/live
インタラクティブな通信を実現
railsはfiber safeじゃなかったので、　falconで使えるようにするのが大変だったよという話

ブラウザ上でJSライクな動作をするゲームをRubyで作れる
live.forwardEventで、ブラウザ内のイベントをサーバに送る
https://github.com/socketry/flappy-bird

rails7.2 と100％互換
### lively
https://github.com/socketry/lively

---

### 感想
- 昨日 Official Party で少し話をした人が KeyNote のすごい人でした
- え、これサーバサイドでゲームを動かしてるってこと？合ってる？
- ゲームで遊ぶ Matz を見れた
- Falcon 使ってみたい

### キーワード
- ./#async
- ./#ActionCable
### 資料
TODO: 探す


## Does Ruby Parser dream of highly expressive grammar?
https://rubykaigi.org/2024/presentations/ydah_.html#day2

### Lrama
https://github.com/ruby/lrama
competer: Bison
parse.y 

Bisonはプリミティブな文法規則しかサポートしていなかった
Lramaはそれを解決する

Parser and Lexer
https://yui-knk.hatenablog.com/entry/2023/12/06/082203

---
### 感想
- なるほどワカラン…
- parse.y がとても難解なこと、Lrama という Parser がそれを解く一助になることがわかった
- この後も Lrama based Parser の話を何度も耳にすることになる
- Parser を扱っている人たちの熱量と凄さを知った
### キーワード
- ./#parse.y
- ./#Lrama
### 資料
https://speakerdeck.com/ydah/does-ruby-parser-dream-of-highly-expressive-grammar

https://x.com/ydah_/status/1791032088462041297


## RubyGems on ruby.wasm
https://rubykaigi.org/2024/presentations/kateinoigakukun.html#day2

### ruby.wasm
2022RubyKaigiで発表
packaging system 
### rubygems support
installer
runtime loader `require `
when to install gems
	pre-installation をサポートしていない
WASMでブラウザ内でマストドンを動かすデモ

Serverless Staging を提供できる
ローコスト、ローリスク

#### Package Gem
#### Cross-Compile C-extensions
Static Linking は複雑すぎる
DynamicLinking

---
### 感想
- 気になってた wasm の話
- まだ本番運用できる感じではなさそう
- でも完全サーバレスなマストドンが動いてるすごい
- ローコストでポータブルなステージング環境を作れるということ？
### キーワード
- ./#wasm
- ./#require
- ./#pre-installation
### 資料
https://speakerdeck.com/kateinoigakukun/rubygems-on-ruby-dot-wasm

## Unlock The Universal Parsers: A New PicoRuby Compiler
https://rubykaigi.org/2024/presentations/hasumikin.html#day2

### Prism
Portability
Small footprint
rubocopなどもこちらを採用
### Lrama 
AST
PicoRubyではこちらを採用したい

依存関係を減らしていくことで、メモリ消費を減らすことに成功

### Picoruby
電子工作を ruby で書ける

---
### 感想
- Parser は Prism がつよいらしい
- Lrama based Parser もポテンシャルがあるらしい
- リファクタリングやパフォーマンス改良のステップがわかった
- Picoruby も触ってみたい
### キーワード
- ./#Prism parser
- ./#Picoruby
### 資料
https://slide.rabbit-shocker.org/authors/hasumikin/RubyKaigi2024/UnlockTheUniversalParsers.pdf

## Breaking the Ruby Performance Barrier
https://rubykaigi.org/2024/presentations/maximecb.html#day2

### YJITプロジェクト
速度改善
C拡張に頼らずにRubyのパフォーマンスを上げようという取り組み

なぜC？
- I/O機能
- パフォーマンス
- ネイティブライブラリを使うため
	- rmagic
	- redis
一部のライブラリはpure ruby 互換を持ち始めている
Protoboeuf
	Google Protobuf
Ruby の String method は "Perf Killer"

YJIIT3.4 

---
### 感想
- YJITについては寡聞にして知らず…Ruby2系までしか追いついていない……
- この後もYJITのセッションを何度か聴講
- 早くruby3系にあげて、パフォーマンス計測したい
### キーワード
- ./#JIT
- ./#MJIT
- ./#YJIT
### 資料
https://www.slideshare.net/slideshow/breaking-the-ruby-performance-barrier-with-yjit/269367259

## Good first issues of TypeProf
https://rubykaigi.org/2024/presentations/mametter.html#day2
### TypeProf
型推論を、補完、定義ジャンプ、GoToリファレンス
型定義を書かなくても、型推論する

コントリビューター募集中！

`service.rb`
`ast/` is easy
`env/`, `graph/` is difficult

### Good first Issue
1. Play
	1. scenario/known-issue
2. ruby / rbs constructs
3. エラー表示改善
4. GoTo Definition の拡張
### 今後
今年中に Ruby/RBS 全構文サポート
来年中に Rails サポートしたい

---
### 感想
- Good First Issue と言うが簡単ではない
- Ruby の型システムも全然追いついていないから、勉強せねば
- とにかくruby3系にあげたい欲と、取り残されてる焦り
### キーワード
- ./#RBS
- ./#TypeProf
### 資料
https://speakerdeck.com/mame/good-first-issues-of-typeprof

## Squeezing Unicode Names into Ruby Regular Expressions
https://rubykaigi.org/2024/presentations/duerst.html#day2

```
"abにna" ~= /\p{Hiragana}/ #=>1
```

Ruby supports Unicode "Name" Property
探索木で name property を探索し、ユニコードを特定する

---
### 感想
- とても便利そうな正規表現が出てきた
- でも何話しているのかはワカラナイ
- 文系だからとか逃げずに、コンピュータサイエンスの基礎だけでも勉強したい
- アルゴリズムやデータ構造はしっかり身につけよう
### キーワード
- ./#Ruby と 正規表現
- ./#Name property
### 資料
TODO: 探す
## Lightning Talks
https://rubykaigi.org/2024/presentations/lt/
### Visualize the internal state of ruby processes in Real-Time
metrics_monitor
CPU, Memory を可視化
VisualVM for ruby を作りたかった
- require なしでやりたい
- Visuallizer 充実
### The Frontend Rubyist
WASM
10MBzip
1 Thread
RubyKoans
Electron
### Enjoy Creative Coding with Ruby
クリエイティブコーディング
コードでお絵描き、アニメーション
ruby & Processing, WASM
業務コーディングは常に同じ答えを返す確実性が大事だが、
クリエイティブコーディングでは偶然性を取り入れる
息抜きや趣味として
### Rearchitect Ripper
パーサーの話題がすごく多い、Praserルネサンスや
ripper は Parser のスーパークラス
parse.y
Lrama によりバグが治った！
### The Journey of rubocop-daemon into RuboCop
`require rubocop`が遅いから、デーモンで起動しておく `rubocop-aemon`
IDEプラグインに対応するために、ドキュメントを整える
パッチを取り込む際はオーナーである自分が責務を持つ必要がある
daemon が rubocop 本体に取り込まれた in 2022
OSSは素晴らしい。とはいえオーナーにとっては結構大変なこともある
頑張るといいことある
### The test code generator using static analysis and LLM
Omochi is CLI for supporting ruby development
LLM使ってる
Railsの機能開発支援
Rspec の雛形を作成する
AST, DFS
今後: request spec に対応、テストが書けていない（苦笑）
### Contributing to the Ruby Parser
parse.y の難しさ
コードベースが大きく、どこから読めばいいかわからない
コードを短くする挑戦
C依存を減らす挑戦
もっと相談をしていきたい
### Improved REXML XML parsing performance using StringScanner
RBPDFの作者
REXML 、 SAX parser
YJITによるパフォ改善
Regexp より StringScanner により早く
### Hotspot on Coverage
 APP上でCoverageを動かす
 akainaa gem 
 コードのホットスポットを表示
 SimpleCovと違い、実行されているコードを強調表示する
### Drive Your Code: Building an RC Car by Writing Only Ruby
電子工作で大変な環境構築が、Rubyならできる。PicoRuby
ラズパイPico
R2P2
Rubyを書くだけでラジコンが作れる
### An anthropological view of the Ruby community
Ruby emblaces the Chaos
readable, maintainable, Ideal
CODESPEAK free reading

---
### 感想
- ちゃんとしたLT会（制限時間ありでテンポよく回していくやつ）を初めて見た
- 業務にも取り入れられそうな話がたくさん聞けた
- 会場の雰囲気もとても良くて楽しめた
- 二日目も終わり、楽しいけど疲労感を感じる……
- イベントもあるけれど、今日は一人で美味しい沖縄料理食べて、ゆっくり休もう……

---
## Ruby Committers and the World
https://rubykaigi.org/2024/presentations/rubylangorg.html#day3
Ruby3.4.0 リリース
### Literal string will be frozen in the future
3.4.0 から段階的になる
- 反対派: 
    - 動的性がRubyの強み
    - Quine で使いたい
- 賛成派:
    - 可読性
     - Ractor との相性はいい
     - `+` を書けばfrozenではないものとして扱われる
- その他観点
    - パフォーマンス
    - GC
 
参考になりそう
https://zenn.dev/mamayukawaii/articles/20240404003435
### Embeded RBS
Matz「私は型を書きたくない。でもコメントで書く分には許容する」

①
```
## @sig siza: Integer
```
vs
```
## @rbs size: Integer
```

②
```
## @rbs returns void
@ rbs yields (String) -> void
```
vs
```
## @rbs return: void
@ @rbs &: (String) -> void
```

③
長く書く、説明もつけたい
vs
短く書く

YARD

### Replacing Ruby's build system GNUAutotools → 
https://github.com/ruby/ruby/blob/master/tool/missing-baseruby.bat
ビルドシステムを移植する
LMake RubyMake Rake 
「悪いのは最初にextconfにしたMatz」
「Perlを参考にしたらそれは有罪でしょう」
https://chatgpt.com/share/73d07e10-ae60-4f07-9afd-40c9eddcfe57
### Do you want to remove the GVL
早くなるかもしれないけど、早くなる前に壊れる
出来るけど、すごく大変。だし、その世界でプログラミングするのもすごく大変
https://en.wikipedia.org/wiki/Giant_lock
ロックを取るやつと、ロックを取らないやつがいたら、ロックの意味がない
https://peps.python.org/pep-0703/
### Improving Ruby's async usability "async...await..."
Matz「async await で読みやすくなったと感じる人はどうかしてる」
何かコードを書いたときに、それがasyncなのかどうかを判断することができない
async の間 RubyGem 以外全て止まっているという構造は都合がいい
### `defer` for ruby ?
需要はある
実装するための仕組みもある

---
### 感想
- すごい人たちの議論を見れる貴重な経験
- 白熱して面白い発言もちらほら
- でも内容の半分は理解できてない
    - ビルドシステムとかLockとか
- Rubiest コミュニティの良さが詰まってる Keynote でした
### キーワード
- ./#frozen string
- ./#GNU Autotool
- ./#GVL
## YJIT Makes Rails 1.7x Faster
YJIT Makes Rails ~~1.7x~~ 1.8x Faster
https://rubykaigi.org/2024/presentations/k0kubun.html#day3
### YJIT
production ready JIT for CRuby
主にShopifyが主導

Rails7.2~ Ruby3.3~ でデフォルトに

メモリサイズが心配
`--yjit-exec-mem-size` で調整する
	32, 24, 16MiB
	3~4x をメタデータのために使用する

3.3ではメモリ上ではなくレジストリ上で上手いことやるようにした
スタックをレジスタに割り当てるようにした
レジスタはCPUにくっついているのではやい

---
### 感想
- YJIT、なんと素晴らしい取り組み
- Ruby と Rails のバージョンあげたい
### キーワード
- ./#YJIT
### 資料
https://speakerdeck.com/k0kubun/rubykaigi-2024

## Speeding up Instance Variables with Red-Black Trees
https://rubykaigi.org/2024/presentations/tenderlove.html#day3

### why InstanceVariable is slow
Red-Black tree
kin of binary tree, self balancing
いくつかのルールに則って定義される木
https://ja.wikipedia.org/wiki/平衡二分探索木

https://cgi.cse.unsw.edu.au/~eptcs/paper.cgi?MSFP2014.8
https://www.microsoft.com/en-us/research/project/koka/
 Okasaki Style  Tree
参考
http://ppl.jssst.or.jp/?ss2021#jb6258cb
再平衡（リバランス）、パターンマッチによって探索が容易になる
https://www.cs.tufts.edu/comp/150FP/archive/chris-okasaki/redblack99.pdf
https://dev.to/baweaver/definitive-pattern-matching-array-like-structures-1298
### Object Shape
インスタンス変数は名前で管理。木構造で管理している ShapeTree
名前と設定する順番のみ
定義の確認をするために O(n) 時間がかかる

### まとめ
データ構造を学ぼう
Ruby3.3にあげよう
Ruby3.4はもっと早いよ

---
### 感想
- インスタンス変数のパフォーマンスとか気にしたことなかった
- いろんな木構造があるんだな
- ObjectShape というのも初耳
- データ構造を学ぼう
- パフォーマンスの話が続いている
### キーワード
- ./#インスタンス変数
- ./#赤黒木
- ./#Object Shape
### 資料
TODO: 探す

## ERB, ancient and future
https://rubykaigi.org/2024/presentations/m_seki.html#day3
Sekiさんが作ったもの
- dRuby
- Rinda
- ERB
https://toruby.connpass.com/
### 古代編
CGI: HTML記述のためのロジックが大半を占める
→ eRuby
→ ERb, ERbLight
→ ERBがRuby標準ライブラリに 
Proc問題
### 未来編
正規表現でブロックの始まりを検知するしくみがあった→難しい
解決策は見えたので、誰か手伝ってほしい

---
### 感想
- 大変お世話になっているお方だ
- まだまだ進化していくんだな
- gemの歴史に興味を持った
### キーワード
- ./#CGI
- ./#ERB
- ./#Proc in ERB
### 資料
https://speakerdeck.com/m_seki/erb-ancient-and-future

https://x.com/m_seki/status/1791779124014670056

## Finding and fixing memory safety bugs in C with ASAN
https://rubykaigi.org/2024/presentations/KJTsanaktsidis.html#day3

ASAN can help debug crush
### ASAN
Adress sanitizer
https://en.m.wikipedia.org/wiki/Code_sanitizer
https://github.com/google/sanitizers/wiki/AddressSanitizer

Cはルールが多く、破ると "undefined behavior"
ASAN はこれらのルールを強制する

2〜5x overhead 
本番では使えないが、開発やテストで有用

https://bugs.ruby-lang.org/issues/20387

---
### 感想
- 英語のセッションは理解度がガクンと落ちる
    - 次回はリアルタイム翻訳の仕組みを整えていこう
### キーワード
- ./#ASAN
### 資料
TODO: 探す

## Using Ruby in the browser is wonderful.
https://rubykaigi.org/2024/presentations/ledsun.html#day3

Ruby をブラウザで動かすには
- JSとRubyとは差分があるので、 Glue が必要
- 外部のリソースを参照できない
- OS依存のメソッドを使用できない

Ruby.wasm
CRubyをブラウザ上で動かすVMとなる
Ruby.wasm  がJSとRubyの架け橋になる
- JS.global
- JS.eval
- JS.object

JA::RequirwRemote
何も手を加えずとも、コンパーチブルに動くようになる
### 次に
Ruby.wasm でフロントエンドフレームワークを作りたい
> 世にフロントエンドフレームワークが乱立している理由は、作ると楽しいからでしょう

→ 作った
抽象的な動きをするフレームワークを開発するのに120行でできた
https://github.com/ledsun/orbital_ring

---
### 感想
- WASM がだんだん実用的なものに進化している、ありがたい
- フロントエンドフレームワーク作れちゃうのすごい
- Orbital Ring かっこいい……
### キーワード
- ./#wasm
### 資料
https://speakerdeck.com/ledsun/using-ruby-in-the-browser-is-wonderful

## Matz keynote
Better Ruby
「RubyKaigiは価値観を破壊されるカンファレンス」
「私は普通のおじさんなのに」
「私が使いたいと思う言語を作ったら、多くの人に刺さった。特に意思決定者に多くマッチした」
Rubyの良さは、コード書いてて楽しい「Programmer's best friend」
- 制約の少ない言語
- Gems/Tools, Community
- 高い生産性につながる(理不尽さにつながることが少ない）

Rails 20周年
- WAFフレームワークの寿命は短い中、長寿

https://toprubycompanies.info/
FastGlowingCompany の上位にも
Rubyが社会を動かしている

Ruby was slow
make it Better

Rubyの未来を4つの要素で語る
### 1. Performance
- ベンチマーク大好き
- 必要以上にスピードが大好き
- YARV 
- MJIT 2014
- YJIT 2022
- VMを改善していく
### 2. Performance
- ObjectHeap
- Varibalbe width
- Object shape
- GC, Memory allocation

メモリはボトルネックになりやすい。VMを改善することで、金銭的に助かる。メモリは高いから
### 3. Performance
Concurrency for System Architecture
- Thread
- GVL
- Processes
- Fiber for I/O
- Ractor for CPU

↓
- NxM Threwads
- Lightweight Ractor
- Ractor local GC
- Fiber Scheduler
### 4. Performance
Software Performance is Important
Developer, Programer Performance is Important
Better Experience by Tools Support
- Rubocop
- Profiler
- devtool

Need better Parser
Parser for Ruby
Parser for Tools
We need the Universal Parser
- Prism
- Parser by Lrama

API will using Prism
Universal Parser might be  Lrama-Parser
- Prism: Hand-written Parser
- Lrama based Parser: Parser from Parser Generator

Syntax Moratorium
- **don't change Syntax for at least a year (for Prism)**
- (Prismsの開発者であるKevin次第)

We need Comunity
- Ruby Association Grant
- Google Summer of Code
- Independent Community Effort
- Conferences

Rubyist を支援していきます
### The future of Ruby
2004 Ruby2 (not Ruby2.0(2012))
https://matz.rubyist.net/20031115.html#p01
I once tried to restart Ruby
Just like Perl6 or Python3000
It turned out to be a bad idea...コミュニティの分断を生んだ
	Perl6 → Raku 
	Python3000 → Pytho3.0
構想はしたが面倒くさくて実装はしなかった。Matz の怠惰癖がRubyの分断を防いだ
Ruby2の構想のほとんどは実現できてきた
- Namespace
- Packages
**Namespace の導入ができればRuby4.0と呼べる**
### Dream
SDGs, GX(GreenTransformation)
地球にやさしいRuby
名前も決めたけどいいや
メモリ消費を抑える、パフォーマンス
シングルバイナリ
AOT Compiler

---
### 感想
- Rubyのパフォーマンスの重要性と今後を強調。実際たくさんのセッションを通してパフォーマンスの成長を感じていたので、非常に納得感がある
- Rubyの歴史を知れて面白かった
- 実際Rubyは書いていて楽しいし、だからこそこんなにもたくさんの人・コミュニティに愛されている
- Ruby のコミュニティに貢献できるように、もっともっと成長したい！

---
# RubyKaigi 全体を通しての感想
- 最初は知識不足を痛感し、疎外感を感じていた
    - 単身で初の参加というのもあり、孤独感
    - 実際業務で使っているRuby環境は、非常に遅れをとってしまっている
- でも会場の雰囲気や、イベント・ブースでのコミュニケーションを通して、少しずつ自信を持てた
- ここで得た知識をもとに、社内の環境を整えていきたい
- 素晴らしい言語とそれを支えるコミュニティ、 Rubyist の一人であることを自負できるように、もっともっと成長したい
- とにかくまずはテストを書いて、アップデートのための地盤を固めよう
- 来年もぜひ参加したい！
---

# キーワード
## MRI ruby
Matz Ruby Interpreter
CRuby いわゆる普通の Ruby
## Ruby の種類
- jruby
  - Java実装
- mruby
- picoruby

などなど
## DSL
Domain Specifitic Language
あるドメイン（タスク）をうまく記述することに集中した言語や実装。メタプログラミング。
外部DSLと内部DSL。Rubyは内部DSLの実装に長けている。Rails や Rspec がその代表。
https://ja.wikipedia.org/wiki/%E3%83%89%E3%83%A1%E3%82%A4%E3%83%B3%E5%9B%BA%E6%9C%89%E8%A8%80%E8%AA%9E
## Quine
実行すると自身と同じ文字列を出力するコード。百聞は一見にしかず。
https://ja.m.wikipedia.org/wiki/%E3%82%AF%E3%83%AF%E3%82%A4%E3%83%B3_(%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0)
https://github.com/tric/trick2022/blob/master/01-tompng/entry.rb
## TRICK
Transcendental Ruby Imbroglio Contest for rubyKaigi
超絶技巧なRubyコードを書いて競い合うコンテスト。
https://github.com/tric/trick2022
## PBT
PropertyBasedTesting
ランダムな値でテストを行う。
- Generator
- Property
- Shrinking
失敗したケースに対してShrinkしてバグが再現する最小構成を特定する
←→ ExampleBasedTesting
https://tech.findy.co.jp/entry/2024/05/20/164206
## Ractor
並列処理を実現するためのインターフェース
CPUの処理能力を超えて実行する
https://atdot.net/~ko1/activities/2023_rubykaigi2023.pdf
## Ruby Standard Library
Ruby の標準ライブラリ。
https://docs.ruby-lang.org/en/master/standard_library_rdoc.html
## C拡張
Rubyの拡張ライブラリ。
C 言語で書かれたコードを Ruby のコードと統合して、高速な処理や低レベルのアクセスを実現できる
https://docs.ruby-lang.org/en/master/extension_ja_rdoc.html
## 名前空間
各要素に一意の名前をつけて識別できるようにした範囲。Ruby では `::` で区分する。
例えば拡張ライブラリが他のライブラリのシンボルを使用している場合、名前が衝突する。
https://tagomoris.hatenablog.com/entry/2023/05/15/174652
## ライブラリとパッケージシステム
gem が Ruby ライブラリ。ライブラリ ≒ パッケージ。
Ruby におけるパッケージ管理システムが Rubygems。
Bundler はプロジェクトごとに gem を管理する gem。Gemfile や Gemfile.lock の記述。
https://qiita.com/3no3_tw/items/8c1e3e95c75edae1036d
## IRB
Interactive Ruby
対話的に Ruby を実行するシェル。
Ruby 環境の `irb` コマンドや、Rails の `rails console` コマンドで起動する
## Reline
IRBにおける、CLIでのテキスト入力のためのライブラリ
CLIでのテキストの読み込みや編集、履歴などを管理する
Ruby2.7にバンドルされた
## Readline
UNIX系システムで使用される、CLIでのテキスト入力のためのライブラリ
C依存なので、Relineに置き換える取り組みが進められている
Ruby3.3からサポートを削除する
## AST
## DFS
## Parser
## PatternMatching
## async
## ActionCable
## parse.y
## Lrama
## wasm
## require
## pre-installation
## Prism parser
## Picoruby
## JIT
## YJIT
## RBS
## TypeProf
## Ruby と 正規表現
## StringScanner
## Name property
## frozen string
## GNU Autotool
## GVL
## インスタンス変数
## 赤黒木
## Object Shape
## Profiler
## GC
ガベージコレクションのこと
[module GC (Ruby 3.3 リファレンスマニュアル)](https://docs.ruby-lang.org/ja/latest/class/GC.html)
## CGI
## ERB
## Proc in ERB
## ASAN
