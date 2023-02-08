<!--
# Macros in the AST
-->
# ASTにおけるマクロ

<!--
As previously mentioned, macro processing in Rust happens *after* the construction of the AST.
As such, the syntax used to invoke a macro *must* be a proper part of the language's syntax.
In fact, there are several "syntax extension" forms which are part of Rust's syntax.
Specifically, the following 4 forms (by way of examples):
-->

先述した通り、Rustにおけるマクロの処理は抽象構文木が構築された*後に*行われます。
したがって、マクロを呼び出すのに使う構文は言語の構文上正しいもの*でなければなりません*。
実際、いくつかの「構文拡張(syntax extension)」形式がRustの構文に組み込まれています。
具体的には、以下の4つの形式があります:

<!--
1. `# [ $arg ]`; *e.g.* `#[derive(Clone)]`, `#[no_mangle]`, …
2. `# ! [ $arg ]`; *e.g.* `#![allow(dead_code)]`, `#![crate_name="blang"]`, …
3. `$name ! $arg`; *e.g.* `println!("Hi!")`, `concat!("a", "b")`, …
4. `$name ! $arg0 $arg1`; *e.g.* `macro_rules! dummy { () => {}; }`.
-->

1. `# [ $arg ]` *例:* `#[derive(Clone)]`, `#[no_mangle]`, …
2. `# ! [ $arg ]` *例:* `#![allow(dead_code)]`, `#![crate_name="blang"]`, …
3. `$name ! $arg` *例:* `println!("Hi!")`, `concat!("a", "b")`, …
4. `name ! $arg0 $arg1` *例:* `macro_rules! dummy { () => {}; }`

<!--
The first two are [attributes] which annotate items, expressions and statements. They can be
classified into different kinds, [built-in attributes], [proc-macro attributes] and [derive attributes].
[proc-macro attributes] and [derive attributes] can be implemented with the second macro system that Rust
offers, [procedural macros]. [built-in attributes] on the other hand are attributes implemented by
the compiler.
-->

最初の2つは[属性]で、アイテム、式、文にアノテーションをつけるものです。
属性は[組み込み属性]、[proc-macro属性]、[derive属性]に分類できます。
[proc-macro属性]と[derive属性]はRustが提供する第二のマクロシステムである[手続き的マクロ]によって実装できます。
一方、[組み込み属性]はコンパイラが実装している属性です。

<!--
The third form `$name ! $arg` are function-like macros. It is the form available for use with `macro_rules!`, `macro` and also procedural macros.
Note that this form is not *limited* to `macro_rules!` macros: it is a generic syntax extension form.
For example, whilst [`format!`] is a `macro_rules!` macro, [`format_args!`] (which is used to *implement* [`format!`]) is *not* as it is a compiler builtin.
-->

3つめの形式 `$name ! $arg` は関数形式マクロです。これは`macro_rules!`によるマクロ、`macro`によるマクロ、そして手続き的マクロを呼び出すのに使えます。
`macro_rules!`で定義されたマクロだけに限定されているわけではないことに注意してください: これは一般的な構文拡張の形式なのです。
例えば、[`format!`]は`macro_rules!`によるマクロですが、([`format!`]を*実装*するのに用いられている)[`format_args!`]はそうでは*なく*、コンパイラ組み込みのマクロです。

<!--
The fourth form is essentially a variation which is *not* available to macros.
In fact, the only case where this form is used *at all* is with the `macro_rules!` construct itself.
-->
4つめの形式は原則としてマクロに対して使える形式では*ありません*。
実際、この形式が使われるのは`macro_rules!`構文そのものに対して*のみ*に限られます。

<!--
So, starting with the third form, how does the Rust parser know what the `$arg` in (`$name ! $arg`) looks like for every possible syntax extension?
The answer is that it doesn't *have to*.
Instead, the argument of a syntax extension invocation is a *single* token tree.
More specifically, it is a single, *non-leaf* token tree; `(...)`, `[...]`, or `{...}`. With that
knowledge, it should become apparent how the parser can understand all of the following invocation
forms:
-->
さて、3番めの形式について、Rustのパーサはいかにして考えうるすべての構文拡張に対して `($name ! $arg)`の`$arg`の中身がどうなっているのかを知るのでしょうか?
知る*必要がない*、というのが答えです。
代わりに、構文拡張の呼び出しにおける引数部分は*単一の*トークン木になります。
より具体的にいえば、単一の*葉でない*トークン木、すなわち`(...)`, `[...]` または `{...}`になります。
この知識があれば、パーサが以下の呼び出し形式をどのように理解するのかがはっきりわかるはずです:


```rust,ignore
bitflags! {
    struct Color: u8 {
        const RED    = 0b0001,
        const GREEN  = 0b0010,
        const BLUE   = 0b0100,
        const BRIGHT = 0b1000,
    }
}

lazy_static! {
    static ref FIB_100: u32 = {
        fn fib(a: u32) -> u32 {
            match a {
                0 => 0,
                1 => 1,
                a => fib(a-1) + fib(a-2)
            }
        }

        fib(100)
    };
}

fn main() {
    use Color::*;
    let colors = vec![RED, GREEN, BLUE];
    println!("Hello, World!");
}
```

<!--
Although the above invocations may *look* like they contain various kinds of Rust code, the parser simply sees a collection of meaningless token trees.
To make this clearer, we can replace all these syntactic "black boxes" with ⬚, leaving us with:
-->
上記の呼び出し形式はありとあらゆる種類のRustコードを含んでいるように*見える*かもしれませんが、パーサが見ているのはただの無意味なトークン木の集まりにすぎません。
このことをよりはっきりさせるために、すべての構文上の「ブラックボックス」を⬚に置き換えてみると、次のようになります:

```text
bitflags! ⬚

lazy_static! ⬚

fn main() {
    let colors = vec! ⬚;
    println! ⬚;
}
```

<!--
Just to reiterate: the parser does not assume *anything* about ⬚;
it remembers the tokens it contains, but doesn't try to *understand* them.
This means ⬚ can be anything, even invalid Rust!
As to why this is a good thing, we will come back to that at a later point.
-->
繰り返しになりますが、パーサは⬚に関して*何の*仮定も置きません。⬚に含まれるトークンを覚えておくだけで、それを*理解*しようとすることはありません。
すなわち⬚に入るのは何であっても、無効なRustコードであってもかまわないのです!
どうしてこれがいいことなのかについては、あとで説明します。

<!--
So, does this also apply to `$arg` in form 1 and 2, and to the two args in form 4? Kind of.
The `$arg` for form 1 and 2 is a bit different in that it is not directly a token tree, but a *simple path* that is either followed by an `=` token and a literal expression, or a token tree.
We will explore this more in-depth in the appropriate proc-macro chapter.
The important part here is that this form as well, makes use of token trees to describe the input.
The 4th form in general is more special and accepts a very specific grammar that also makes use of token trees though.
The specifics of this form do not matter at this point so we will skip them until they become relevant.
-->
さて、このことは形式1と2における`$arg`、そして形式4における2つの引数にも当てはまるのでしょうか? 大体はあっています。
形式1と2の`$arg`は少し違っていて、そのままトークン木になるわけではなく、後に`=`とリテラル式かトークン木が続く*単純パス(simple path)* になります。
これについては、手続き的マクロについてのしかるべき章で深堀りしていきます。
ここで重要なのは、この形式においても入力を表現するのにトークン木を用いているということです。
4つめの形式は概してより特別で、非常に限定された構文(とはいえ、これもトークン木を用いる)のみを受け付けます。
この形式の詳細は現時点では重要ではないので、重要になるまで脇においておきましょう。

<!--
The important takeaways from this are:
-->
以上のことから得られる結論は次の通りです:

<!--
* There are multiple kinds of syntax extensions in Rust.
* Just seeing something of the form `$name! $arg`, doesn't tell you what kind of syntax extension it might be.
    It could be a `macro_rules!` macro, a `proc-macro` or maybe even a builtin.
* The input to every `!` macro invocation, that is form 3, is a single non-leaf token tree.
* Syntax extensions are parsed as *part* of the abstract syntax tree.
-->
* Rustには複数の種類の構文拡張が存在する。
* ただ`$name! $arg`という形をした何かを見るだけでは、それがどの種類の構文拡張なのかを知ることはできない。それは`macro_rules!`によるマクロかもしれないし、手続き的マクロ、さらには組み込みのマクロでさえありうる。
* すべての`!`がつくマクロ呼び出し (形式3) の入力は単一の葉でないトークン木である。
* 構文拡張は抽象構文木の*一部*としてパースされる。

<!--
The last point is the most important, as it has *significant* implications.
Because syntax extensions are parsed into the AST, they can **only** appear in positions where they are explicitly supported.
Specifically syntax extensions can appear in place of the following:
-->
最後の点が最も重要で、それはこのことが*重大な*意味合いを含むためです。
構文拡張は抽象構文木にパースされるため、明示的にサポートされた場所に**しか**現れ得ないのです。
具体的には、構文拡張は次の場所に書くことができます:

<!--
* Patterns
* Statements
* Expressions
* Items(this includes `impl` items)
* Types
-->
* パターン
* 文
* 式
* アイテム (`impl`アイテムを含む)
* 型

<!--
Some things *not* on this list:
-->
このリストに含まれないものの例:

<!--
* Identifiers
* Match arms
* Struct fields
-->
* 識別子
* match式のアーム
* 構造体のフィールド

<!--
There is absolutely, definitely *no way* to use syntax extensions in any position *not* on the first list.
-->
前者のリストに*含まれない*場所で構文拡張を使うことは、絶対に、どう足掻こうとも*不可能*です。

<!--
[attributes]: https://doc.rust-lang.org/reference/attributes.html
-->
[属性]: https://doc.rust-lang.org/reference/attributes.html
<!--
[built-in attributes]: https://doc.rust-lang.org/reference/attributes.html#built-in-attributes-index
-->
[組み込み属性]: https://doc.rust-lang.org/reference/attributes.html#built-in-attributes-index
<!--
[proc-macro attributes]: https://doc.rust-lang.org/reference/procedural-macros.html#attribute-macros
-->
[proc-macro属性]: https://doc.rust-lang.org/reference/procedural-macros.html#attribute-macros
<!--
[derive attributes]: https://doc.rust-lang.org/reference/procedural-macros.html#derive-macro-helper-attributes
-->
[derive属性]: https://doc.rust-lang.org/reference/procedural-macros.html#derive-macro-helper-attributes
<!--
[procedural macros]: https://doc.rust-lang.org/reference/procedural-macros.html
-->
[手続き的マクロ]: https://doc.rust-lang.org/reference/procedural-macros.html
[`format!`]: https://doc.rust-lang.org/std/macro.format.html
[`format_args!`]: https://doc.rust-lang.org/std/macro.format_args.html
