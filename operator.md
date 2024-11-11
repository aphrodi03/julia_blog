# Juliaに演算子を追加してみた

\toc

今回は、言語処理系のOSSの一つである**JULIA**言語について、ソースコードを手探り、演算子を追加することを目指しました。

あらかじめ`2024-05-julia`ディレクトリに移動しておきましょう。

## 要件

組み合わせの総数を計算する*Combination*演算は、現状
```julia
binomial(4,2)
> 6
```
と、関数として定義されている。

この`binomial(M,N)`を`M~~N`という短縮された**二項演算子**にする。

演算子には優先順位がある。

!!! Example title="演算子の優先順位の代表例"
3+5×2に対して、3+5よりも5×2が先に行われる。
!!!

既存の演算子の優先順位は、

```scheme
(define prec-assignment
  (append! (add-dots '(= += -= −= *= /= //= |\\=| ^= ÷= %= <<= >>= >>>= |\|=| &= ⊻= ≔ ⩴ ≕))
           (add-dots '(~))
           '(:= $=)))
;; comma - higher than assignment outside parentheses, lower when inside
(define prec-pair (add-dots '(=>)))
(define prec-conditional '(?))
(define prec-arrow       (add-dots '(← → ↔ ↚ ↛ ↞ ↠ ↢ ↣ ↦ ↤ ↮ ⇎ ⇍ ⇏ ⇐ ⇒ ⇔ ⇴ ⇶ ⇷ ⇸ ⇹ ⇺ ⇻ ⇼ ⇽ ⇾ ⇿ ⟵ ⟶ ⟷ ⟹ ⟺ ⟻ ⟼ ⟽ ⟾ ⟿ ⤀ ⤁ ⤂ ⤃ ⤄ ⤅ ⤆ ⤇ ⤌ ⤍ ⤎ ⤏ ⤐ ⤑ ⤔ ⤕ ⤖ ⤗ ⤘ ⤝ ⤞ ⤟ ⤠ ⥄ ⥅ ⥆ ⥇ ⥈ ⥊ ⥋ ⥎ ⥐ ⥒ ⥓ ⥖ ⥗ ⥚ ⥛ ⥞ ⥟ ⥢ ⥤ ⥦ ⥧ ⥨ ⥩ ⥪ ⥫ ⥬ ⥭ ⥰ ⧴ ⬱ ⬰ ⬲ ⬳ ⬴ ⬵ ⬶ ⬷ ⬸ ⬹ ⬺ ⬻ ⬼ ⬽ ⬾ ⬿ ⭀ ⭁ ⭂ ⭃ ⥷ ⭄ ⥺ ⭇ ⭈ ⭉ ⭊ ⭋ ⭌ ￩ ￫ ⇜ ⇝ ↜ ↝ ↩ ↪ ↫ ↬ ↼ ↽ ⇀ ⇁ ⇄ ⇆ ⇇ ⇉ ⇋ ⇌ ⇚ ⇛ ⇠ ⇢ ↷ ↶ ↺ ↻ --> <-- <-->)))
(define prec-lazy-or     (add-dots '(|\|\||)))
(define prec-lazy-and    (add-dots '(&&)))
(define prec-comparison
  (append! '(in isa)
           (add-dots '(> < >= ≥ <= ≤ == === ≡ != ≠ !== ≢ ∈ ∉ ∋ ∌ ⊆ ⊈ ⊂ ⊄ ⊊ ∝ ∊ ∍ ∥ ∦ ∷ ∺ ∻ ∽ ∾ ≁ ≃ ≂ ≄ ≅ ≆ ≇ ≈ ≉ ≊ ≋ ≌ ≍ ≎ ≐ ≑ ≒ ≓ ≖ ≗ ≘ ≙ ≚ ≛ ≜ ≝ ≞ ≟ ≣ ≦ ≧ ≨ ≩ ≪ ≫ ≬ ≭ ≮ ≯ ≰ ≱ ≲ ≳ ≴ ≵ ≶ ≷ ≸ ≹ ≺ ≻ ≼ ≽ ≾ ≿ ⊀ ⊁ ⊃ ⊅ ⊇ ⊉ ⊋ ⊏ ⊐ ⊑ ⊒ ⊜ ⊩ ⊬ ⊮ ⊰ ⊱ ⊲ ⊳ ⊴ ⊵ ⊶ ⊷ ⋍ ⋐ ⋑ ⋕ ⋖ ⋗ ⋘ ⋙ ⋚ ⋛ ⋜ ⋝ ⋞ ⋟ ⋠ ⋡ ⋢ ⋣ ⋤ ⋥ ⋦ ⋧ ⋨ ⋩ ⋪ ⋫ ⋬ ⋭ ⋲ ⋳ ⋴ ⋵ ⋶ ⋷ ⋸ ⋹ ⋺ ⋻ ⋼ ⋽ ⋾ ⋿ ⟈ ⟉ ⟒ ⦷ ⧀ ⧁ ⧡ ⧣ ⧤ ⧥ ⩦ ⩧ ⩪ ⩫ ⩬ ⩭ ⩮ ⩯ ⩰ ⩱ ⩲ ⩳ ⩵ ⩶ ⩷ ⩸ ⩹ ⩺ ⩻ ⩼ ⩽ ⩾ ⩿ ⪀ ⪁ ⪂ ⪃ ⪄ ⪅ ⪆ ⪇ ⪈ ⪉ ⪊ ⪋ ⪌ ⪍ ⪎ ⪏ ⪐ ⪑ ⪒ ⪓ ⪔ ⪕ ⪖ ⪗ ⪘ ⪙ ⪚ ⪛ ⪜ ⪝ ⪞ ⪟ ⪠ ⪡ ⪢ ⪣ ⪤ ⪥ ⪦ ⪧ ⪨ ⪩ ⪪ ⪫ ⪬ ⪭ ⪮ ⪯ ⪰ ⪱ ⪲ ⪳ ⪴ ⪵ ⪶ ⪷ ⪸ ⪹ ⪺ ⪻ ⪼ ⪽ ⪾ ⪿ ⫀ ⫁ ⫂ ⫃ ⫄ ⫅ ⫆ ⫇ ⫈ ⫉ ⫊ ⫋ ⫌ ⫍ ⫎ ⫏ ⫐ ⫑ ⫒ ⫓ ⫔ ⫕ ⫖ ⫗ ⫘ ⫙ ⫷ ⫸ ⫹ ⫺ ⊢ ⊣ ⟂ ⫪ ⫫ <: >:))))
(define prec-pipe<       '(|.<\|| |<\||))
(define prec-pipe>       '(|.\|>| |\|>|))
(define prec-colon       (append! '(: |..|) (add-dots '(… ⁝ ⋮ ⋱ ⋰ ⋯))))
(define prec-plus        (append! '($)
                          (add-dots '(+ - − ¦ |\|| ⊕ ⊖ ⊞ ⊟ |++| ∪ ∨ ⊔ ± ∓ ∔ ∸ ≏ ⊎ ⊻ ⊽ ⋎ ⋓ ⟇ ⧺ ⧻ ⨈ ⨢ ⨣ ⨤ ⨥ ⨦ ⨧ ⨨ ⨩ ⨪ ⨫ ⨬ ⨭ ⨮ ⨹ ⨺ ⩁ ⩂ ⩅ ⩊ ⩌ ⩏ ⩐ ⩒ ⩔ ⩖ ⩗ ⩛ ⩝ ⩡ ⩢ ⩣))))
(define prec-times       (add-dots '(* / ⌿ ÷ % & · · ⋅ ∘ × |\\| ∩ ∧ ⊗ ⊘ ⊙ ⊚ ⊛ ⊠ ⊡ ⊓ ∗ ∙ ∤ ⅋ ≀ ⊼ ⋄ ⋆ ⋇ ⋉ ⋊ ⋋ ⋌ ⋏ ⋒ ⟑ ⦸ ⦼ ⦾ ⦿ ⧶ ⧷ ⨇ ⨰ ⨱ ⨲ ⨳ ⨴ ⨵ ⨶ ⨷ ⨸ ⨻ ⨼ ⨽ ⩀ ⩃ ⩄ ⩋ ⩍ ⩎ ⩑ ⩓ ⩕ ⩘ ⩚ ⩜ ⩞ ⩟ ⩠ ⫛ ⊍ ▷ ⨝ ⟕ ⟖ ⟗ ⨟)))
(define prec-rational    (add-dots '(//)))
(define prec-bitshift    (add-dots '(<< >> >>>)))
;; `where`
;; implicit multiplication (juxtaposition)
;; unary
(define prec-power       (add-dots '(^ ↑ ↓ ⇵ ⟰ ⟱ ⤈ ⤉ ⤊ ⤋ ⤒ ⤓ ⥉ ⥌ ⥍ ⥏ ⥑ ⥔ ⥕ ⥘ ⥙ ⥜ ⥝ ⥠ ⥡ ⥣ ⥥ ⥮ ⥯ ￪ ￬)))
(define prec-decl        '(|::|))
;; `where` occurring after `::`
(define prec-dot         '(|.|))

(define prec-names '(prec-assignment
                     prec-pair prec-conditional prec-arrow prec-lazy-or prec-lazy-and prec-comparison
                     prec-pipe< prec-pipe> prec-colon prec-plus prec-times prec-rational prec-bitshift
                     prec-power prec-decl prec-dot))
```

などと定義されている。(`src/julia-parser.scm`)

今回は、binomial演算子の優先順位を、
* bitshift演算子`<< >> >>>`
* power演算子`^`

の間に設定する。

!!! Note title="既存の~演算子"
すでに~は単項演算子として実装されており、これはビット反転を表す。
!!!

以上を踏まえて、ブラックボックステストのテストケースを作成した。
```julia
# examples/unicode.jl
println(3~~2)       # 一般的なもの
println(1~~1)       # binomial(1,1)=1
println(1~~0)       # binomial(1,0)=1
println(-3~~2)      # -binomial(3,2)=3
println((-3)~~2)    # binomial(-3,2)=6
println((-3)~~3)    # binomial(-3,3)=10
println(2~~(-2))    # binomial(2,-2)=0
println((-3)~~(-1)) # binomial(-3,-1)=0
println(2+6~~2)     # 2+binomial(6,2)=17
println(2-6~~2)     # 2-binomial(6,2)=-13
println(3*6~~2)     # 3*binomial(6,2)=45
println((3*6)~~2)   # binomial(3*6,2)=153
println(6~~2<4)     # binomial(6,2)<4=false
println(4<6~~2)     # 4<binomial(6,2)=true
println(4<6~~2 ? "TRUE" : "FALSE") # 4<binomial(6,2) ? "TRUE" : "FALSE" = "TRUE"
println(4<6~~2?"TRUE" : "FALSE")   # 4<binomial(6,2)?"TRUE" : "FALSE" -> ERROR: ParseError: space required before `?` operator
println(4<6~~2&&6~~2<7 ? "TRUE" : "FALSE")  # 4<binomial(6,2)&&binomial(6,2)<7 ? "TRUE" : "FALSE" = "FALSE"
println(4<6~~2&&6~~2<16 ? "TRUE" : "FALSE") # 4<binomial(6,2)&&binomial(6,2)<16 ? "TRUE" : "FALSE" = "TRUE"
println(4<6~~2||6~~2<7 ? "TRUE" : "FALSE")  # 4<binomial(6,2)||binomial(6,2)<7 ? "TRUE" : "FALSE" = "TRUE"
println(4<6~~2||6~~2<16 ? "TRUE" : "FALSE") # 4<binomial(6,2)||binomial(6,2)<16 ? "TRUE" : "FALSE" = "TRUE"
x=2
x+=6~~2
println(x)              # x=2, x+=6~~2 = 17
println(6~~2/3)         # binomial(6,2)/3 = 5.0
println(90/6~~2)        # 90/binomial(6,2) = 6.0
println(6~~2//3)        # binomial(6,2)//3 = 5//1
println(90//6~~2)       # 90//binomial(6,2) = 6//1
println(1<<5~~2)        # 1<<binomial(5,2) = 1024
println(1024>>4~~2)     # 1024>>binomial(4,2) = 16
println((1024>>4)~~2)   # binomial((1024>>4),2) = 2016
println(2^6~~2)         # binomial(2^6,2) = 2016
println(6~~2^2)         # binomial(6,2^2) = 15

# ~と~~
println(~(-7)~~2)       # binomial(~(-7),2) = 15
println(6~~~(-3))       # binomial(6,~(-3)) = 15
println(6~~3~~2)        # binomial(binomial(6,3),2) = 190
println(~6~~3~~2)        # binomial(binomial(~6,3),2) = 3570
println((~6)~~3~~2)        # binomial(binomial(~6,3),2) = 3570
println(6~~(~3)~~2)        # binomial(binomial(6,~3),2) = 0
```

## `./julia`が生成される過程

演算子を追加する上では、**julia**言語の動作原理をうっすら理解しておく必要がある。

まずは、**julia**言語のソフトウェアとしてのディレクトリ/ファイル構造を示す。

![ディレクトリ構造](/assets/directory.png)

### JULIAコードで基本的な構文や機能を実装

`base/`ディレクトリには、無数の`.jl`ファイルが含まれている。

このディレクトリでは、JULIAのコードにより
* 様々な演算子・便利関数の定義
* 字句規則の制定
* 構文規則の制定
* 字句解析の関数定義
* 構文解析の関数定義

などが行われている。特に、字句解析・構文解析関連の実装は、全て、`base/JuliaSyntax/`で行われている。

### C言語で実行の枠組みを作成

我々が普段書くようなJULIAのプログラムを実行する
```sh
julia ./helloworld.jl
```
といったコマンドは、実際には
* 環境変数に指定されたPATHの先にある、実行ファイル（ここまで`./julia`と呼んできたものに相当）を実行する
* 実行時のコマンドライン引数として、プログラムのPATH`./helloworld.jl`を渡す

ことを意味している。

逆に言えば、実行ファイル`./julia`は、`.jl`形式のJULIAで書かれたプログラムに対し、

* 字句解析: 文字列としてのプログラムコード（`helloworld.jl`の中身）を、文->Tokenと分解する
* 構文解析: Tokenの並びが構文規則に従うか判定する
* 意味解析: 解析結果をもとに中間コード（例: 逆ポーランド記法）に変換する
* 型推論: 各変数や式の型を推論
* 最適化: レジスタの割付や、不要な演算の省略などを行う（LLVM; Low-Level-Virtual-Machine コンパイラフレームワークが行う`deps/`のコード）
* アセンブリに変換
* 実行: アセンブリを順に実行する

といった機能を全て包含していることになる。

!!! Note title="字句解析で文に分解する方法"
改行文字\nを読み取るか、文の区切り文字;を読み取る度に区切る。
!!!

実は、これらの機能の司会進行をしているのは、**`src/`ディレクトリに配置されたC言語ないしはC++のコード**から生成された部分である。

### 構文解析器の作成

さて、これまでの説明を聞いて、ふとある疑問が浮かぶのではないか。

!!! Question title="JULIAの文法をJULIAで定義?"
JULIAの字句規則や構文規則ですらJULIAで書かれていては、それを読みようがないのでは、という疑問が生じますよね。至極まともな疑問なので以下で解説します。
!!!

最終的な目標は、**`./julia`に、`base/`の構文規則等を理解させ、構文解析器を含めること**である。そのための手段として、**bootstrap**というものがある。

!!! Tip title="Bootstrapとは"
コンピュータシステムやソフトウェアが、自身を立ち上げるために必要な基盤を自動的に用意しながら段階的に起動する過程や手法である。

プログラミング言語におけるブートストラップとは、言語処理系（コンパイラやインタープリタ）を新しく作る際、最初はその言語自身ではなく別の言語で基礎を構築することを指す。
!!!

JULIAの構文解析も、最初は`julia-parser.scm`のように**Scheme**などの異なる言語で行い(1)、**徐々にJulia自体が自分のコードを処理できるようになる**(2)。
最終的に、自己ホスティングの準備が整うと、以後はJuliaだけでさらに拡張可能になる(3)。

![ブートストラップイメージ](/assets/bootstrap.png)

!!! Tip title="コンピュータのブートストラップ"
コンピュータの電源を入れると、まず最小限のプログラム（BIOSやUEFI）が起動し、ストレージからオペレーティングシステムをロードする。これが進むにつれて、より多くの機能が動作可能となり、最終的にOSが完全に立ち上がるとユーザーが使用できる状態になる。
!!!
