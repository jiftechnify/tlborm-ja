<!--
# Macros, A Methodical Introduction
-->
# マクロ: 体系的説明

<!--
This chapter will introduce Rust's declarative [Macro-By-Example][mbe] system by explaining the system as a whole.
It will do so by first going into the construct's syntax and its key parts and then following it up with more general information that one should at least be aware of.
-->
本章ではRustの宣言的な[Macros-By-Example][mbe]のシステムについて、全体の概略を説明することによって紹介していきます。
まず構成要素の文法と鍵となるパーツについて説明したあと、最低限知っておくべきより全般的な情報を補足します。

[mbe]: https://doc.rust-lang.org/reference/macros-by-example.html
[Macros chapter of the Rust Book]: https://doc.rust-lang.org/book/ch19-06-macros.html

# `macro_rules!`

<!--
With all that in mind, we can introduce `macro_rules!` itself.
As noted previously, `macro_rules!` is *itself* a syntax extension, meaning it is *technically* not part of the Rust syntax.
It uses the following forms:
-->
以上のことを念頭に置いて、`macro_rules!`自体の説明に入ります。
先述したとおり、`macro_rules!`は*それ自体*が構文拡張のひとつであり、これは`macro_rules!`が*技術的には*Rustの文法に含まれないということを意味します。

```rust,ignore
macro_rules! $name {
    $rule0 ;
    $rule1 ;
    // …
    $ruleN ;
}
```

<!--
There must be *at least* one rule, and you can omit the semicolon after the last rule.
You can use brackets(`[]`), parentheses(`()`) or braces(`{}`).
-->
少なくとも1つのルールが必要で、最後のルールの後ろのセミコロンは省略できます。
かっこは、角かっこ(`[]`)、丸かっこ(`()`)、波かっこ(`{}`)のどれを使ってもかまいません。

<!--
Each *"rule"* looks like the following:
-->
それぞれの「ルール」は次のような見た目になります:

```ignore
    ($マッチパターン) => {$展開形}
```

<!--
Like before, the types of parentheses used can be any kind, but parentheses around the matcher and braces around the expansion are somewhat conventional.
The expansion part of a rule is also called its *transcriber*.
-->
先程と同様かっこの種類はどれでも構いませんが、マッチパターンは丸かっこで、展開形は波かっこで囲む慣習があります。

<!--
Note that the choice of the parentheses does not matter in regards to how the mbe macro may be invoked.
In fact, function-like macros can be invoked with any kind of parentheses as well, but invocations with `{ .. }` and `( ... );`, notice the trailing semicolon, are special in that their expansion will *always* be parsed as an *item*.
-->
ここでどのかっこを選ぶかは、MBEマクロの呼び出し方には影響しないということを指摘しておきます。
実際のところ、関数形式マクロはどの種類のかっこを使っても呼び出せます。ただし、`{ .. }`や`( ... );`(末尾のセミコロンに注意)という形の呼び出しは、*常にアイテム*としてパースされるという点で特別です。

<!--
If you are wondering, the `macro_rules!` invocation expands to... *nothing*.
At least, nothing that appears in the AST; rather, it manipulates compiler-internal structures to register the mbe macro.
As such, you can *technically* use `macro_rules!` in any position where an empty expansion is valid.
-->
不思議に思われるかもしれませんが、`macro_rules!`の呼び出しが何に展開されるかというと… *何にも*展開されません。
少なくとも、抽象構文木には何の変化も起きません。むしろ、その呼び出しはMBEマクロを登録するためにコンパイラ内部のデータ構造を操作します。
そういうわけで、*技術的には* `macro_rules!`は空の展開が許される全ての場所で利用できます。

## Matching

When a `macro_rules!` macro is invoked, the `macro_rules!` interpreter goes through the rules one by one, in declaration order.
For each rule, it tries to match the contents of the input token tree against that rule's `matcher`.
A matcher must match the *entirety* of the input to be considered a match.

If the input matches the matcher, the invocation is replaced by the `expansion`; otherwise, the next rule is tried.
If all rules fail to match, the expansion fails with an error.

The simplest example is of an empty matcher:

```rust,ignore
macro_rules! four {
    () => { 1 + 3 };
}
```

This matches if and only if the input is also empty (*i.e.* `four!()`, `four![]` or `four!{}`).

Note that the specific grouping tokens you use when you invoke the function-like macro *are not* matched, they are in fact not passed to the invocation at all.
That is, you can invoke the above macro as `four![]` and it will still match.
Only the *contents* of the input token tree are considered.

Matchers can also contain literal token trees, which must be matched exactly.
This is done by simply writing the token trees normally.
For example, to match the sequence `4 fn ['spang "whammo"] @_@`, you would write:

```rust,ignore
macro_rules! gibberish {
    (4 fn ['spang "whammo"] @_@) => {...};
}
```

You can use any token tree that you can write.

## Metavariables

Matchers can also contain captures.
These allow input to be matched based on some general grammar category, with the result captured to a metavariable which can then be substituted into the output.

Captures are written as a dollar (`$`) followed by an identifier, a colon (`:`), and finally the kind of capture which is also called the fragment-specifier, which must be one of the following:

* [`block`](./minutiae/fragment-specifiers.md#block): a block (i.e. a block of statements and/or an expression, surrounded by braces)
* [`expr`](./minutiae/fragment-specifiers.md#expr): an expression
* [`ident`](./minutiae/fragment-specifiers.md#ident): an identifier (this includes keywords)
* [`item`](./minutiae/fragment-specifiers.md#item): an item, like a function, struct, module, impl, etc.
* [`lifetime`](./minutiae/fragment-specifiers.md#lifetime): a lifetime (e.g. `'foo`, `'static`, ...)
* [`literal`](./minutiae/fragment-specifiers.md#literal): a literal (e.g. `"Hello World!"`, `3.14`, `'🦀'`, ...)
* [`meta`](./minutiae/fragment-specifiers.md#meta): a meta item; the things that go inside the `#[...]` and `#![...]` attributes
* [`pat`](./minutiae/fragment-specifiers.md#pat): a pattern
* [`path`](./minutiae/fragment-specifiers.md#path): a path (e.g. `foo`, `::std::mem::replace`, `transmute::<_, int>`, …)
* [`stmt`](./minutiae/fragment-specifiers.md#stmt): a statement
* [`tt`](./minutiae/fragment-specifiers.md#tt): a single token tree
* [`ty`](./minutiae/fragment-specifiers.md#ty): a type
* [`vis`](./minutiae/fragment-specifiers.md#vis): a possible empty visibility qualifier (e.g. `pub`, `pub(in crate)`, ...)

For more in-depth description of the fragment specifiers, check out the [Fragment Specifiers](./minutiae/fragment-specifiers.md) chapter.

For example, here is a `macro_rules!` macro which captures its input as an expression under the metavariable `$e`:

```rust,ignore
macro_rules! one_expression {
    ($e:expr) => {...};
}
```

These metavariables leverage the Rust compiler's parser, ensuring that they are always "correct".
An `expr` metavariables will *always* capture a complete, valid expression for the version of Rust being compiled.

You can mix literal token trees and metavariables, within limits (explained in [Metavariables and Expansion Redux]).

To refer to a metavariable you simply write `$name`, as the type of the variable is already specified in the matcher. For example:

```rust,ignore
macro_rules! times_five {
    ($e:expr) => { 5 * $e };
}
```

Much like macro expansion, metavariables are substituted as complete AST nodes.
This means that no matter what sequence of tokens is captured by `$e`, it will be interpreted as a single, complete expression.

You can also have multiple metavariables in a single matcher:

```rust,ignore
macro_rules! multiply_add {
    ($a:expr, $b:expr, $c:expr) => { $a * ($b + $c) };
}
```

And use them as often as you like in the expansion:

```rust,ignore
macro_rules! discard {
    ($e:expr) => {};
}
macro_rules! repeat {
    ($e:expr) => { $e; $e; $e; };
}
```

There is also a special metavariable called [`$crate`] which can be used to refer to the current crate.

[Metavariables and Expansion Redux]: ./minutiae/metavar-and-expansion.md
[`$crate`]: ./minutiae/hygiene.md#crate

## Repetitions

Matchers can contain repetitions. These allow a sequence of tokens to be matched.
These have the general form `$ ( ... ) sep rep`.

* `$` is a literal dollar token.
* `( ... )` is the paren-grouped matcher being repeated.
* `sep` is an *optional* separator token. It may not be a delimiter or one
    of the repetition operators. Common examples are `,` and `;`.
* `rep` is the *required* repeat operator. Currently, this can be:
    * `?`: indicating at most one repetition
    * `*`: indicating zero or more repetitions
    * `+`: indicating one or more repetitions

    Since `?` represents at most one occurrence, it cannot be used with a separator.

Repetitions can contain any other valid matcher, including literal token trees, metavariables, and other repetitions allowing arbitrary nesting.

Repetitions use the same syntax in the expansion and repeated metavariables can only be accessed inside of repetitions in the expansion.

For example, below is a mbe macro which formats each element as a string.
It matches zero or more comma-separated expressions and expands to an expression that constructs a vector.

```rust
macro_rules! vec_strs {
    (
        // Start a repetition:
        $(
            // Each repeat must contain an expression...
            $element:expr
        )
        // ...separated by commas...
        ,
        // ...zero or more times.
        *
    ) => {
        // Enclose the expansion in a block so that we can use
        // multiple statements.
        {
            let mut v = Vec::new();

            // Start a repetition:
            $(
                // Each repeat will contain the following statement, with
                // $element replaced with the corresponding expression.
                v.push(format!("{}", $element));
            )*

            v
        }
    };
}

fn main() {
    let s = vec_strs![1, "a", true, 3.14159f32];
    assert_eq!(s, &["1", "a", "true", "3.14159"]);
}
```

You can repeat multiple metavariables in a single repetition as long as all metavariables repeat equally often.
So this invocation of the following macro works:

```rust
macro_rules! repeat_two {
    ($($i:ident)*, $($i2:ident)*) => {
        $( let $i: (); let $i2: (); )*
    }
}

repeat_two!( a b c d e f, u v w x y z );
```

But this does not:

```rust
# macro_rules! repeat_two {
#     ($($i:ident)*, $($i2:ident)*) => {
#         $( let $i: (); let $i2: (); )*
#     }
# }

repeat_two!( a b c d e f, x y z );
```

failing with the following error

```
error: meta-variable `i` repeats 6 times, but `i2` repeats 3 times
 --> src/main.rs:6:10
  |
6 |         $( let $i: (); let $i2: (); )*
  |          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

## Metavariable Expressions

> *RFC*: [rfcs#1584](https://github.com/rust-lang/rfcs/blob/master/text/3086-macro-metavar-expr.md)\
> *Tracking Issue*: [rust#83527](https://github.com/rust-lang/rust/issues/83527)\
> *Feature*: `#![feature(macro_metavar_expr)]`

Transcriber can contain what is called metavariable expressions.
Metavariable expressions provide transcribers with information about metavariables that are otherwise not easily obtainable.
With the exception of the `$$` expression, these have the general form `$ { op(...) }`.
Currently all metavariable expressions but `$$` deal with repetitions.

The following expressions are available with `ident` being the name of a bound metavariable and `depth` being an integer literal:

- `${count(ident)}`: The number of times `$ident` repeats in the inner-most repetition in total. This is equivalent to `${count(ident, 0)}`.
- `${count(ident, depth)}`: The number of times `$ident` repeats in the repetition at `depth`.
- `${index()}`: The current repetition index of the inner-most repetition. This is equivalent to `${index(0)}`.
- `${index(depth)}`: The current index of the repetition at `depth`, counting outwards.
- `${length()}`: The number of times the inner-most repetition will repeat for. This is equivalent to `${length(0)}`.
- `${length(depth)}`: The number of times the repetition at `depth` will repeat for, counting outwards.
- `${ignore(ident)}`: Binds `$ident` for repetition, while expanding to nothing.
- `$$`:	Expands to a single `$`, effectively escaping the `$` token so it won't be transcribed.

&nbsp;

For the complete grammar definition you may want to consult the [Macros By Example](https://doc.rust-lang.org/reference/macros-by-example.html#macros-by-example) chapter of the Rust reference.
