@def title = "Franklin Example"
@def tags = ["syntax", "code"]

# Juliaに演算子を追加してみた

[演算子](/operators/)

# Franklin.jlのadmonitionを機能拡張してみた




# admonitionsの実装

!!! Info!!!
!!! Note メモ!!!
!!! Warning 警告!!!
!!! Bug バグ!!!
!!! Summary 要約!!!
!!! Success 成功!!!
!!! Failure!!!
!!! Danger 危険!!!
!!! Tip ヒント（ヒント）!!!
!!! Question かわいいだけじゃだめですか？!!!
!!! Example 例!!!
!!! Quote
title="words you should bear in mind" 
厳しいって -ジョージ!!!


# How to use Franklin

\tableofcontents <!-- you can use \toc as well -->

or have tables:

| English         | Mandarin   |
| --------------- | ---------- |
| winnie the pooh | 维尼熊      |

## Basic Franklin extensions

### Divs

@@colbox-blue
Here we go! (this is styled in the css sheet with name "colbox-blue").
@@

### LaTeX and Maths

$$  \varphi(\E{X}) \le \E{\varphi(X)}. \label{equation blah} $$

since we've given it the label `\label{equation blah}`, we can refer it like so: \eqref{equation blah} which can be convenient for pages that are math-heavy.

### A quick note on whitespaces

For most commands you will use `#k` to refer to the $k$-th argument as in LaTeX.

\newcommand{\pathwith}[1]{`/usr/local/bin/#1`}
\newcommand{\pathwithout}[1]{`/usr/local/bin/!#1`}

* with: \pathwith{script.jl}, there's a whitespace you don't want 🚫
* without: \pathwithout{script.jl} here there isn't ✅

### Raw HTML

You can include raw HTML by just surrounding a block with `~~~`.
Not much more to add.

~~~
<div class="row">
  <div class="container">
    <img class="left" src="/assets/rndimg.jpg">
    <p>
    Marine iguanas are truly splendid creatures. They're found on the Gálapagos islands, have skin that basically acts as a solar panel, can swim and may have the ability to adapt their body size depending on whether there's food or not.
    </p>
    <p>
    Evolution is cool.
    </p>
    <div style="clear: both"></div>      
  </div>
</div>
~~~

## Pages and structure

* [menu 1](/menu1/)
* [menu 2](/menu2/)
* [menu 3](/menu3/)

## References (not really)

* \biblabel{noether15}{Noether (1915)} **Noether**,  Körper und Systeme rationaler Funktionen, 1915.
* \biblabel{bezanson17}{Bezanson et al. (2017)} **Bezanson**, **Edelman**, **Karpinski** and **Shah**, [Julia: a fresh approach to numerical computing](https://julialang.org/research/julia-fresh-approach-BEKS.pdf), SIAM review 2017.

## Header and Footer

```html
Last modified: {{ fill fd_mtime }}.
```