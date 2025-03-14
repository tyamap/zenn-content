---
title: "私と Quine の出会いと別れ、そして再会"
emoji: "🪛"
type: "tech"
topics:
  - "Ruby"
  - "Quine"
  - "メタプログラミング"
published: true
published_at: "2025-03-14 09:00"
---

[Roppongi.rb](https://roppongirb.connpass.com/event/346623/) で発表した LT をもとに記事を書きました

# Quine とは

> 自身のソースコードと完全に同じ文字列を出力するプログラム
> [wikipedia](https://ja.wikipedia.org/wiki/%E3%82%AF%E3%83%AF%E3%82%A4%E3%83%B3_(%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0))

# Quine との出会いと別れ

私のクワインとの出会いとしては、昨年の RubyKaigi が初めてでした。
初めてかつ一人で参加して非常に心細かったのですが、Quineだけでなくさまざまな超絶技巧が紹介され、rubyの世界にグッと引き込まれる素晴らしい Keynote でした。
https://rubykaigi.org/2024/presentations/tompng.html

ただ、自己出力プログラムをどのように書くのか、全く分かりませんでした。
出力といえば `puts`, `puts` と書けば `puts` という文字列を出力する必要があるので `puts` を続けて…… `puts "puts 'puts'..."` と、無限ループしてしまい、脳みそがオーバフローして、そのまま考えるのをやめました。

と、ここまでが出会いと別れパートです。

そして一年ほど、quineとは縁のない生活を送っていたある日、嬉しいことがありました。

# そして、感動の再会

前職を辞めるタイミングで、上司からパーカーをプレゼントされたのです。
しかもそのパーカーにはサービスロゴのアスキーアートが Quine で描かれてました。
先ほどの RubyKaigi のお土産話を聞いて、書いてみたくなったそうです。

まさに感動の再会です。

https://zenn.dev/irsc/articles/977e8355acafd3

![](/images/quine_1.png)

これはとても嬉しいプレゼントでした。
そしてその感動は、僕もクワインを書きたいと思い立たせてくれました。
今回はその挑戦で学んだことを共有したいと思います。

今回のゴールとして、このパーカーのようにアスキーアートのクワインを書きたい、ということにします

# シンプルな Quine

まずクワインの基本を知る必要があるので、 ChatGPT にシンプルなクワインを書いてもらいました。

```rb
eval$s=%q(puts "eval$s=%q(#{$s})")
```
これは実際に Quine として動作します。

見慣れない記号が連なってて気が滅入りますが、わからないとこを一つ一つ紐解いてみます。

## `eval`

- 引数の文字列をそのまま ruby コードとして実行する関数
- 今回の Quine の主役

## `$s`

- 変数のプレフィクスに `$` をつけることで、グローバル変数になる
- $ によってパーサーが変数であると認識してくれるので、関数あとの括弧やスペースを省略できる

## `%q`

- `%q()`, `%q''`, `%q{}` で囲むと、シングルクォートで囲むのと同じ役割
- シングルクォートが使えない場合に便利

## `"#{}"`
- これは実務でもよく使う
- 文字列の中に式を展開する役割

---

逆にわかりづらいうえにQuineでもなくなってしまいますが、分解してみます。
```rb
# eval$s=%q(puts "eval$s=%q(#{$s})")
$s = 'puts "eval$s=%q(#{$s})"'
eval($s)
```

`$s` に文字列を代入しながら実行し、そのなかで `$s` が展開されることで、 Quine が実現するわけですね。面白いです。

# 　アスキーアート Quine

なかなか難しいですが、だいたい理解できたので、 AA Quine に挑戦します。

## 参考
Findy さんの記事や mickey24 さんの記事が大変参考になりました

https://tech.findy.co.jp/entry/2024/05/23/093756

https://mickey24.hatenablog.com/entry/20100915/ruby_udonge_quine

## 使用するAA
本来は弊社ロゴとか使いたかったのですが、ガイドラインの確認などをする時間がなかったので日和ってこのようなアスキーアートを用意しました。

```
AAAAAAAAAAAAAAAAAAA                                                                                                                                                                           AAAAAA
AAAAAAAAAAAAAAAAAAAAAA                                                                                                                                                                        AAAAAA
AAAAAAAAAAAAAAAAAAAAAAAA                                                                                                                                                                      AAAAAA
AAAAAA          AAAAAAA                                                                                                                                                                             
AAAAAA             AAAAAA         AAAAAAAAAAA           AAAAA AAAAAAAAAAA          AAAAA AAAAAAAAAAA               AAAAAAAAAAA           AAAAA   AAAAAAAAAA             AAAAAAAAAAAAAAAAAA AAAAAAAAA
AAAAAA             AAAAAA      AAAAAAAAAAAAAAAAA        AAAAAAAAAAAAAAAAAAAA      AAAAAAAAAAAAAAAAAAAAA         AAAAAAAAAAAAAAAAA        AAAAAAAAAAAAAAAAAAAAA      AAAAAAAAAAAAAAAAAAAAA  AAAAAAAAA
AAAAAA             AAAAAA    AAAAAAAAAAAAAAAAAAAAA      AAAAAAAAAAAAAAAAAAAAAA    AAAAAAAAAAAAAAAAAAAAAA      AAAAAAAAAAAAAAAAAAAAA      AAAAAAAAAAAAAAAAAAAAA      AAAAAAA     AAAAAAAAA  AAAAAAAAA
AAAAAA           AAAAAAA    AAAAAAA         AAAAAAA     AAAAAA          AAAAAA     AAAAAA         AAAAAAA    AAAAAAA         AAAAAAA     AAAAAA          AAAAAA    AAAAAA         AAAAAA      AAAAAA
AAAAAAAAAAAAAAAAAAAAAAA     AAAAAA           AAAAAA     AAAAAA           AAAAAA    AAAAAA          AAAAAA    AAAAAA           AAAAAA     AAAAAA          AAAAAA    AAAAAA          AAAAA      AAAAAA
AAAAAAAAAAAAAAAAAAAAA      AAAAAA            AAAAAA     AAAAAA           AAAAAA    AAAAAA          AAAAAA   AAAAAA            AAAAAA     AAAAAA          AAAAAA    AAAAAA         AAAAAA      AAAAAA
AAAAAAAAAAAAAAAAAAAA       AAAAAA             AAAAA     AAAAAA           AAAAAA    AAAAAA          AAAAAA   AAAAAA            AAAAAA     AAAAAA          AAAAAA     AAAAAAAAAAAAAAAAAAA       AAAAAA
AAAAAA         AAAAAA       AAAAAA           AAAAAA     AAAAAA           AAAAAA    AAAAAA          AAAAAA    AAAAAA           AAAAAA     AAAAAA          AAAAAA      AAAAAAAAAAAAAAAAA        AAAAAA
AAAAAA          AAAAAA      AAAAAAA         AAAAAAA     AAAAAA          AAAAAA     AAAAAA         AAAAAAA    AAAAAAA         AAAAAAA     AAAAAA          AAAAAA     AAAAAAA   AA              AAAAAA
AAAAAA           AAAAAA      AAAAAAAAAAAAAAAAAAAAA      AAAAAAA       AAAAAAAA     AAAAAAAAA    AAAAAAAA      AAAAAAAAAAAAAAAAAAAAA      AAAAAA          AAAAAA    AAAAAA                     AAAAAA
AAAAAA           AAAAAAA      AAAAAAAAAAAAAAAAAAA       AAAAAAAAAAAAAAAAAAAA      AAAAAAAAAAAAAAAAAAAAA        AAAAAAAAAAAAAAAAAAA       AAAAAA          AAAAAA    AAAAAAAAAAAAAAAAAAA        AAAAAA
AAAAAA            AAAAAAA       AAAAAAAAAAAAAA          AAAAAAAAAAAAAAAAAA        AAAAAAAAAAAAAAAAAA             AAAAAAAAAAAAAA          AAAAAA          AAAAAA     AAAAAAAAAAAAAAAAAAA       AAAAAA
                                                        AAAAAA                    AAAAAA                                                                                         AAAAAA            
                                                        AAAAAA                    AAAAAA                                                                          AAAAAAA     AAAAAAAA             
                                                        AAAAAA                    AAAAAA                                                                          AAAAAAAAAAAAAAAAAAAA             
                                                        AAAAAA                     AAAAAA                                                                            AAAAAAAAAAAAAAAAA                      
```

これをQuineで表現します。

# AA Quine のポイント

## `%w` と `join`

まずアーキーアートは、記号やスペースの連続で、ほとんどがスペースですね。

必要な箇所にスペースを入れる必要があるので、文字列配列を扱う `%w` 記法と `join` を使います。
`%w` の中ではスペースは要素区切りとして認識され、連続して入力できます。
文字列配列を `join` して `eval` に渡すことで、任意の箇所にスペースを入れても文字列として実行できるというわけですね。

```rb
eval %w(
pu  ts
  "h   e   ll  o"
).join
# => hello
```

## データ量圧縮

コードでAAを表現するので、当然ですがロジックはAAの文字より短くする必要があります。
なのでAAをそのまま eval に渡すことはできないわけです。

ですので、AAをいかに圧縮して別の形で表現するかがポイントになるわけです。

## バイナリで表現

AAを扱いやすいデータ構造に変換します。スペースを0、それ以外を1に置き換えます。

```
aa = <<~EOM
111111111111111111100000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000111111
111111111111111111111100000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000111111
111111111111111111111111000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000111111
111111000000000001111111000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
111111000000000000011111100000000011111111111000000000001111101111111111100000000001111100111111111110000000000000001111111111100000000000111110001111111111000000000000011111111111111111100111111111
111111000000000000011111100000011111111111111111000000001111111111111111111100000001111111111111111111110000000001111111111111111100000000111111111111111111111000000011111111111111111111100111111111
111111000000000000011111100001111111111111111111110000001111111111111111111111000001111111111111111111111000000111111111111111111111000000111111111111111111111000000111111100000011111111100111111111
111111000000000001111111000011111110000000001111111000001111110000000000111111000001111110000000000111111100001111111000000000111111100000111111000000000011111100001111110000000001111110000000111111
111111111111111111111110000011111100000000000111111000001111110000000000011111100001111110000000000011111100001111110000000000011111100000111111000000000011111100001111110000000000111110000000111111
111111111111111111111000000111111000000000000111111000001111110000000000011111100001111110000000000011111100011111100000000000011111100000111111000000000011111100001111110000000001111110000000111111
111111111111111111110000000111111000000000000011111000001111110000000000011111100001111110000000000011111100011111100000000000011111100000111111000000000011111100000111111111111111111100000000111111
111111000000000111111000000011111100000000000111111000001111110000000000011111100001111110000000000011111100001111110000000000011111100000111111000000000011111100000011111111111111111000000000111111
111111000000000011111100000011111110000000001111111000001111110000000000111111000001111110000000000111111100001111111000000000111111100000111111000000000011111100000111111100011000000000000000111111
111111000000000001111110000001111111111111111111110000001111111000000011111111000001111111110000011111111000000111111111111111111111000000111111000000000011111100001111110000000000000000000000111111
111111000000000001111111000000111111111111111111100000001111111111111111111100000001111111111111111111110000000011111111111111111110000000111111000000000011111100000111111111111111111100000000111111
111111000000000000111111100000001111111111111100000000001111111111111111110000000001111111111111111110000000000000111111111111110000000000111111000000000011111100000011111111111111111110000000111111
000000000000000000000000000000000000000000000000000000001111110000000000000000000001111110000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000111111000000000000
000000000000000000000000000000000000000000000000000000001111110000000000000000000001111110000000000000000000000000000000000000000000000000000000000000000000000000000111111100000111111110000000000000
000000000000000000000000000000000000000000000000000000001111110000000000000000000001111110000000000000000000000000000000000000000000000000000000000000000000000000000111111111111111111110000000000000
000000000000000000000000000000000000000000000000000000001111110000000000000000000001111110000000000000000000000000000000000000000000000000000000000000000000000000000111111111111111110000000000000000
EOM
```

これでAAが二進数のデータになりました。　　
これを整数として保持します。

```rb
# `先頭の 0 の連続数 < 末尾の 0 の連続数` であれば `reverse` すると節約できる
bits = aa.gsub("\n", "").reverse.to_i(2)
puts bits.to_s.length # => 1188
```

全文字数 `3980`, コード部分（1 で表現した部分）文字数 `1531` の AA が 1188 桁の整数になりました。

## AA構造データをさらに圧縮

さらに `Marshal.dump` でシリアライズし、 `Array#pack` で Base64 エンコードすることで圧縮します

```rb
bin = [Marshal.dump(bits)].pack("m").gsub("\n", "")
puts bin.length # => 668
```

`668` 文字のバイト文字列になりました。  
これで処理部分に書けるコードが増えるわけです。

これをデコードして、2進数にして、適切に改行を挟めば、AAが再現できるというわけですね.

## 復元ロジック

なので、そのようなロジックを書いていきます。

### デコード

```rb
b = Marshal.load(b.unpack("m")[0])
```

### Quine 部分

```rb
e = "eval $s = %w" << 39 << ($s*3)
```
`39` はシングルクォートの ASCII コードです。
文字列に ASCIIコード整数を `concat` することで、対応する文字列として変換されるんですね。
今回は `%w' '`でコード部分を囲むので、コードにシングルクォートを生で書けないのです。
また、AA中にスペースや改行も直接書けないので、ASCIIコードを使います。

```rb
puts 'hoge' << 39 << 10 << 32 << 'huga'
#=> hoge'
#=>  huga
 ```

`$s` の値はこの時は定義されていませんが、これまでのコードと後述するコードが結合されて、Quine ロジックになります。
`$s*3` と、コードを3回続けて出力することで、AAを埋めるのに十分な文字列を出力します。
1回目のコードの後はコメントアウトするので、それ以降はダミーのコードとして機能します。

### 構造データから AA を復元してコードで置き換え
`1` をコード、`0` をスペースに置き換え、 改行を適切な位置に挟む処理を書いていきます。

```rb
o = ""
j = -1
0.upto(20 * 198 - 1) { |i|                 # 20, 198: AAの縦・横の長さ
   o << ((n[i] == 1) ? e[j += 1] : 32)
   o << ((i % 198 == (198 - 1)) ? 10 : "") 
}
o[-17, 6] = "" << 39 << ".join"            # 最後の6文字を '.join に置き換え
```
`32`, `10` はそれぞれ スペース, `\n` のASCIIコードです

### これまでのロジックを繋げて出力

最後にこれらをevalで実行できるような文字列にして、繋げて出力するロジックを書きます。
変数も使ってわかりやすく描き直します。

```rb
START_TEXT = 'eval$s=%w'
END_TEXT = '.join'
SINGLE_QUOTE = 39
SPACE = 32
LINE_BREAK = 10

aa_data = aa.split("\n")
x_length = aa_data.first.length
y_length = aa_data.length
last_point = aa_data.last.split('1').last.length

code = <<~CODE
  b="#{bin}"
  n=Marshal.load(b.unpack("m")[0])
  e="#{START_TEXT}"<<#{SINGLE_QUOTE}<<($s*3)
  o=""
  j=-1
  0.upto(#{y_length}*#{x_length}-1){|i|
    o<<((n[i]==1)?e[j+=1]:#{SPACE}$)
    o<<((i%#{x_length}==(#{x_length - 1}))?#{LINE_BREAK}:"")
  }
  o[-#{last_point + END_TEXT.length + 2},6]=""<<#{SINGLE_QUOTE}<<"#{END_TEXT}"
  puts(o)
CODE

code = code.split("\n").join(';')
code << '#'

puts "#{START_TEXT}'#{code}'#{END_TEXT}"
```

このような出力が得られました

```rb
eval$s=%w'b="BAhsKwH3//8HAAAAAAAAAAAAAAAAAAAAAAAAAAAA////DwAAAAAAAAAAAAAAAAAAAAAAAAAAwP///w8AAAAAAAAAAAAAAAAAAAAAAAAAAPD/APgDAAAAAAAAAAAAAAAAAAAAAAAAAAAAPwD4AfwfAN//Afj8HwDwfwB8/A8A/v/n/w8AfuD/P8D//wP+/z+A//8A//8f8P//+f8DgB/+/z/w//+D//8f+P//wP//B/7Af/7/APjDH8Af/AD84AfgD38Af/AD8MMP4Af8//9/8APgBz8AfvgB8MMPgB/8APzwA/AB////B34A+MEPgB9+APz4AeAHPwA//AB+wP///4AfAHzwA+CHHwA/fgD4wQ/AD/7/D/D/AH7AD4Af/AD44QfADz8AfvAD8AP//wH8PwA/8AfwBz8AP/gB+MMfwB/8APzgjwEA/w+AH/j//8Af8A/+g3/g//8DPwA//AAAwP8D4A/8/x/w//+A//8P8P9/wA/AD/7/D/D/APAH/P8A/P8P4P9/APD/A/AD8AP//wf8AAAAAAAAAD8AAPgBAAAAAAAAAAAAAPADAAAAAAAAAMAPAAB+AAAAAAAAAAAA+IN/AAAAAAAAAADwAwCAHwAAAAAAAAAAAP7/HwAAAAAAAAAA/AAA4AcAAAAAAAAAAID//wA=";n=Marshal.load(b.unpack("m")[0]);e="eval$s=%w"<<39<<($s*3);o="";j=-1;0.upto(20*198-1){|i|;o<<((n[i]==1)?e[j+=1]:32);o<<((i%198==(197))?10:"");};o[-23,6]=""<<39<<".join";puts(o)#'.join
```

これを実行すると……

### 👏

```
eval$s=%w'b="BAhsKw                                                                                                                                                                             H3//8H
AAAAAAAAAAAAAAAAAAAAAA                                                                                                                                                                          AAAAAA
////DwAAAAAAAAAAAAAAAAAA                                                                                                                                                                        AAAAAA
AAwP//           /w8AAAA
AAAAAA             AAAAAA         AAAAAAAAAAP           D/APg DAAAAAAAAAA          AAAAA  AAAAAAAAAAA               AAPwD4AfwfA           N//Af   j8HwDwfwB8             /A8A/v/n/w8AfuD/P8  D//wP+/z+
A//8A/             /8f8P/      /+f8DgB/+/z/w//+D        //8f+P//wP//B/7Af/7/       APjDH8Af/AD84AfgD38Af         /AD8MMP4Af8//9/8A        PgBz8AfvgB8MMPgB/8APz       wA/AB////B34A+MEPgB9+  APz4AeAHP
wA//AB             +wP///    4AfAHzwA+CHHwA/fgD4wQ      /AD/7/D/D/AH7AD4Af/AD4     4QfADz8AfvAD8AP//wH8Pw      A/8AfwBz8AP/gB+MMfwB/      8APzgjwEA/w+AH/j//8Af      8A/+g3/      g//8DPwA/  /AAAwP8D4
A/8/x/           w//+A//    8P8P9/w         A/AD/7/     D/D/AP          AH/P8A     /P8P4P          9/APD/A    /AD8AP/         /wf8AAA     AAAAAA          D8AAPg    BAAAAA         AAAAAA       AAPADA
AAAAAAAAMAPAAB+AAAAAAAA     AAAA+I           N/AAAA     AAAAAA           DwAwCA    HwAAAA           AAAAAA    AP7/Hw           AAAAAA     AAAA/A          AA4AcA    AAAAAA          AAAID       //wA="
;n=Marshal.load(b.unp      ack("m            ")[0])     ;e="ev           al$s=%    w"<<39           <<($s*   3);o="            ";j=-1     ;0.upt          o(20*1    98-1){         |i|;o<       <((n[i
]==1)?e[j+=1]:32);o<       <((i%1             98==(     197))?           10:"")    ;};o[-           23,6]=   ""<<39            <<".jo     in";pu          ts(o)#     b="BAhsKwH3//8HAAAA        AAAAAA
AAAAAA         AAAAAA       AAAAAA           ////Dw     AAAAAA           AAAAAA    AAAAAA           AAAAAA    AAwP//           /w8AAA     AAAAAA          AAAAAA      AAAAAAAAAAAPD/APg         DAAAAA
AAAAAA          AAAAAA      AAAAAAA         AAAAPwD     4AfwfA          N//Afj     8HwDwf          wB8/A8A    /v/n/w8         AfuD/P8     D//wP+          /z+A//     8A//8f8   P/               /+f8Dg
B/+/z/           w//+D/      /8f+P//wP//B/7Af/7/AP      jDH8Af/       AD84AfgD     38Af/AD8M     MP4Af8//      9/8APgBz8AfvgB8MMPgB/      8APzwA          /AB///    /B34A+                      MEPgB9
+APz4A           eAHPwA/      /AB+wP///4AfAHzwA+C       HHwA/fgD4wQ/AD/7/D/D       /AH7AD4Af/AD44QfADz8A        fvAD8AP//wH8PwA/8Af       wBz8AP          /gB+MM     fwB/8APzgjwEA/w+AH/        j//8Af
8A/+g3            /g//8DP       wA//AAAwP8D4A/          8/x/w//+A//8P8P9/w         A/AD/7/D/D/APAH/P8             A/P8P4P9/APD/A          /AD8AP          //wf8A      AAAAAAAAD8AAPgBAAAA       AAAAAA
                                                        AAAPAD                     AAAAAA                                                                                           AAAMAP
                                                        AAB+AA                     AAAAAA                                                                            AAAA+IN     /AAAAAAA
                                                        AAADwA                     wCAHwA                                                                            AAAAAAAAAAP7/HwAAAAA
                                                        AAAAA/                     AAA4Ac                                                                            AAAAAAAAAAI'.join
```

# コードの構成

Quine コードの構成はこんな感じです。

![](/images/quine_2.png)

ほとんどがコメントアウトして繰り返し出力しているダミー文字列になってますね。

この部分はまだロジックを書けるので、例えば色をつけたり、アニメーションをつけたりできるかもしれません。

---

# 学び
- さまざまな記法を知れた
- バイナリやエンコードの活用方法を学べた
- 短いコードにたくさんの知恵が詰まってる
- 複雑に見えても一つずつ紐解いていけば案外理解できる
- Ruby 面白い！

---

# Next

- 『あなたの知らない超絶技巧プログラミングの世界』を読む
- 色をつけたい
- 企業・サービスロゴで作りたい
- TRICK参加？？

---

# ご提案

エンジニアへのプレゼントやノベルティに Quine はいかがでしょう

---

# 感謝 🙌


