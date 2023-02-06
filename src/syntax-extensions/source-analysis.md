<!--
# Source Analysis
-->
# ソースコード解析

<!--
### Tokenization
-->
### トークン化

<!--
The first stage of compilation for a Rust program is [tokenization]. This is where the source text is
transformed into a sequence of tokens (*i.e.* indivisible lexical units; the programming language
equivalent of "words"). Rust has various kinds of tokens, such as:
-->

Rustプログラムのコンパイルの最初のステップは[トークン化]です。
これはソースコードをトークン(これ以上分割できない字句単位。プログラミング言語の「単語」)の列に変換するステップです。
Rustにはさまざまな種類のトークンがあります。例えば:

<!--
* Identifiers: `foo`, `Bambous`, `self`, `we_can_dance`, `LaCaravane`, …
* Literals: `42`, `72u32`, `0_______0`, `1.0e-40`, `"ferris was here"`, …
* Keywords: `_`, `fn`, `self`, `match`, `yield`, `macro`, …
* Symbols: `[`, `:`, `::`, `?`, `~`, `@`[^wither-at], …
-->

* 識別子: `foo`, `Bambous`, `self`, `we_can_dance`, `LaCaravane`, …
* リテラル: `42`, `72u32`, `0_______0`, `1.0e-40`, `"ferris was here"`, …
* 予約語: `_`, `fn`, `self`, `match`, `yield`, `macro`, …
* 記号: `[`, `:`, `::`, `?`, `~`, `@`[^wither-at], …

<!--
…among others. There are some things to note about the above: first, `self` is both an identifier
*and* a keyword. In almost all cases, `self` is a keyword, but it *is* possible for it to be
*treated* as an identifier, which will come up later (along with much cursing). Secondly, the list
of keywords includes some suspicious entries such as `yield` and `macro` that aren't *actually* in
the language, but *are* parsed by the compiler—these are [reserved] for future use. Third, the list
of symbols *also* includes entries that aren't used by the language. In the case of `<-`, it is
vestigial: it was removed from the grammar, but not from the lexer. As a final point, note that
`::` is a distinct token; it is not simply two adjacent `:` tokens. The same is true of all
mult-character symbol tokens in Rust, as of Rust 1.2.[^two-lexers]
-->
…など。上記において注意すべき点がいくつかあります。まず、`self`は識別子であり、*かつ*予約語でもあるということです。
ほとんどすべての状況で`self`は予約語ですが、識別子として*扱われる*ことも*確かに*あるのです。これについては後述します(たくさんの呪詛とともに)。
次に、予約語のリストは`yield`や`macro`といった怪しげな要素を含んでいます。これらは*実際*言語の一部ではないのですが、コンパイラは予約語として認識します—将来の利用のために[予約されている]のです。
`<-`は「名残」です。文法からは削除されたものの、字句解析器には残っているのです。
最後に、`::`が区別されたトークンであることに注意してください。`:`というトークンが2つ連続したものとは異なります。複数文字からなるすべての記号トークンについて同じことがいえます(Rust 1.2以降)。[^two-lexers]
<!--
[^wither-at]: `@` has a purpose, though most people seem to forget about it completely: it is used
in patterns to bind a non-terminal part of the pattern to a name.
-->
[^wither-at]: `@`には用途があるのですが、ほとんどの人は完全に忘れてしまっているようです: `@`はパターンの中で使い、パターンの非終端部分を名前に束縛します (*訳注*: 「パターンの非終端部分(non-terminal part of the pattern)」は、[@束縛](https://doc.rust-jp.rs/book-ja/ch18-03-pattern-syntax.html#%E6%9D%9F%E7%B8%9B)においてマッチ条件を表すパターンの部分を指すものと思われる)。

<!--
[^two-lexers]: Technically rust currently(1.46) has two lexers, [`rustc_lexer`] which only emits
single character symbols as tokens and the [lexer] in [`rustc_parse`] which sees multi-character
symbols as distinct tokens.
-->
[^two-lexers]: 技術的には、Rustは現時点(1.46)で2つの字句解析器を持ちます。単一文字の記号のみをトークンとして出力する[`rustc_lexer`]と、複数文字の記号を個別のトークンとみなす[`rustc_parse`]内の[lexer]の2つです(*訳注*: 翻訳時点の最新版(1.67)においても同様)。

<!--
As a point of comparison, it is at *this* stage that some languages have their macro layer, though
Rust does *not*. For example, C/C++ macros are *effectively* processed at this point.[^lies-damn-lies-cpp]
-->
比較点として、まさにこの段階でマクロの展開を行う言語が存在する一方で、Rustはそうではありません。
例えば、C/C++のマクロは*事実上*この時点で処理されます。[^lies-damn-lies-cpp]

<!--
This is why the following code works: [^cpp-it-seemed-like-a-good-idea-at-the-time]
-->
次のコードが動作するのはこのためです。[^cpp-it-seemed-like-a-good-idea-at-the-time]

```c
#define SUB void
#define BEGIN {
#define END }

SUB main() BEGIN
    printf("Oh, the horror!\n");
END
```

<!--
[^lies-damn-lies-cpp]: In fact, the C preprocessor uses a different lexical structure to C itself,
but the distinction is *broadly* irrelevant.
-->
[^lies-damn-lies-cpp]: 実際のところ、CのプリプロセッサはC言語自体とは異なる字句構造を用いていますが、その違いは*ほとんど*重要ではありません。

<!--
[^cpp-it-seemed-like-a-good-idea-at-the-time]: *Whether* it should work is an entirely *different*
question.
-->
[^cpp-it-seemed-like-a-good-idea-at-the-time]: これが動作*すべきか否か*というのは全く*別の*問題です。

<!--
### Parsing
-->
### 構文解析

<!--
The next stage is parsing, where the stream of tokens is turned into an [Abstract Syntax Tree] (AST).
This involves building up the syntactic structure of the program in memory. For example, the token
sequence `1 + 2` is transformed into the equivalent of:
-->
次の段階は構文解析で、これはトークンの列を[抽象構文木](Abstract Syntax Tree, AST)に変換します。
これはプログラムの構文構造をメモリ上に構築する処理を伴います。例えば、トークン列`1 + 2`は次のようなものに変換されます:

```text
┌─────────┐   ┌─────────┐
│ BinOp   │ ┌╴│ LitInt  │
│ op: Add │ │ │ val: 1  │
│ lhs: ◌  │╶┘ └─────────┘
│ rhs: ◌  │╶┐ ┌─────────┐
└─────────┘ └╴│ LitInt  │
              │ val: 2  │
              └─────────┘
```

<!--
The AST contains the structure of the *entire* program, though it is based on purely *lexical*
information. For example, although the compiler may know that a particular expression is referring
to a variable called `a`, at this stage, it has *no way* of knowing what `a` is, or even *where* it
comes from.
-->
抽象構文木はプログラム*全体*の構造を含みますが、それは*字句上の*情報のみに基づくものです。
例えば、特定の式が`a`と呼ばれる変数を参照しているのをコンパイラが知っているとしても、`a`が何なのか、ひいてはそれが*どこから*やってきたのかを知る*術はありません*。

<!--
It is *after* the AST has been constructed that macros are processed. However, before we can discuss
that, we have to talk about token trees.
-->
抽象構文木が構築された*後*、ついにマクロが処理されます。ですが、マクロの処理の考察に入る前に、トークン木について考えておかねばなりません。

<!--
## Token trees
-->
## トークン木 (Token trees)

<!--
Token trees are somewhere between tokens and the AST. Firstly, *almost* all tokens are also token
trees; more specifically, they are *leaves*. There is one other kind of thing that can be a token
tree leaf, but we will come back to that later.
-->
トークン木は、トークンと抽象構文木の中間に位置するものです。
第一に、*ほとんど*すべてのトークンはトークン木でもあります。より正確にいえば、トークンは*葉*にあたります。
もう一種類トークン木の葉になりうるものが存在しますが、それについては後述します。

<!--
The only basic tokens that are *not* leaves are the "grouping" tokens: `(...)`, `[...]`, and `{...}`.
These three are the *interior nodes* of token trees, and what give them their structure. To give a
concrete example, this sequence of tokens:
-->
基本的なトークンのうち唯一葉ではないのが、`(...)`, `[...]`, `{...}`といった「グループ化」トークンです。
これらはトークン木の内部ノードにあたり、トークン木に構造をもたらすものです。具体的な例を挙げると、次のトークン列は…


```text
a + b + (c + d[0]) + e
```

<!--
would be parsed into the following token trees:
-->
次のようなトークン木にパースされます:

```text
«a» «+» «b» «+» «(   )» «+» «e»
          ╭────────┴──────────╮
           «c» «+» «d» «[   ]»
                        ╭─┴─╮
                         «0»
```

<!--
Note that this has *no relationship* to the AST the expression would produce; instead of a single
root node, there are *seven* token trees at the root level. For reference, the AST would be:
-->
これは、この式が生成する抽象構文木とは*何の関連もない*ということに注意してください。
1つの根ノードがあるのではなく、根のレベルに*7つの*トークン木があるのです。
参考までに、この式の抽象構文木を載せておきます:

```text
              ┌─────────┐
              │ BinOp   │
              │ op: Add │
            ┌╴│ lhs: ◌  │
┌─────────┐ │ │ rhs: ◌  │╶┐ ┌─────────┐
│ Var     │╶┘ └─────────┘ └╴│ BinOp   │
│ name: a │                 │ op: Add │
└─────────┘               ┌╴│ lhs: ◌  │
              ┌─────────┐ │ │ rhs: ◌  │╶┐ ┌─────────┐
              │ Var     │╶┘ └─────────┘ └╴│ BinOp   │
              │ name: b │                 │ op: Add │
              └─────────┘               ┌╴│ lhs: ◌  │
                            ┌─────────┐ │ │ rhs: ◌  │╶┐ ┌─────────┐
                            │ BinOp   │╶┘ └─────────┘ └╴│ Var     │
                            │ op: Add │                 │ name: e │
                          ┌╴│ lhs: ◌  │                 └─────────┘
              ┌─────────┐ │ │ rhs: ◌  │╶┐ ┌─────────┐
              │ Var     │╶┘ └─────────┘ └╴│ Index   │
              │ name: c │               ┌╴│ arr: ◌  │
              └─────────┘   ┌─────────┐ │ │ ind: ◌  │╶┐ ┌─────────┐
                            │ Var     │╶┘ └─────────┘ └╴│ LitInt  │
                            │ name: d │                 │ val: 0  │
                            └─────────┘                 └─────────┘
```

<!--
It is important to understand the distinction between the AST and token trees. When writing macros,
you have to deal with *both* as distinct things.
-->
抽象構文木とトークン木の違いはよく理解しておいてください。マクロを書く際は、これらを別々のものとして、*両方とも*扱う必要があるのです。

<!--
One other aspect of this to note: it is *impossible* to have an unpaired parenthesis, bracket or brace;
nor is it possible to have incorrectly nested groups in a token tree.
-->
もう一つ注意すべき点は、トークン木が組になっていない括弧、あるいはネスト構造がおかしいグループを含むことはないということです。

<!--
[tokenization]: https://en.wikipedia.org/wiki/Lexical_analysis#Tokenization
-->
[トークン化]: https://en.wikipedia.org/wiki/Lexical_analysis#Tokenization

<!--
[reserved]: https://doc.rust-lang.org/reference/keywords.html#reserved-keywords
-->
[予約されている]: https://doc.rust-lang.org/reference/keywords.html#reserved-keywords

[`rustc_lexer`]: https://github.com/rust-lang/rust/tree/master/compiler/rustc_lexer
[`rustc_parse`]: https://github.com/rust-lang/rust/tree/master/compiler/rustc_parse
[lexer]: https://github.com/rust-lang/rust/tree/master/compiler/rustc_parse/src/lexer

<!-- 
[Abstract Syntax Tree] https://en.wikipedia.org/wiki/Abstract_syntax_tree
-->
[抽象構文木]: https://en.wikipedia.org/wiki/Abstract_syntax_tree
