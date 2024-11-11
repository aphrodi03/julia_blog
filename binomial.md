@def prepath = "julia_blog"

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
