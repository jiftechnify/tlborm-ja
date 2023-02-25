<!--
# Fragment Specifiers
-->
# フラグメント指定子 (Frangment Specifiers)

<!--
As mentioned in the [`methodical introduction`](../macros-methodical.md) chapter, Rust, as of 1.60, has 14 fragment specifiers.
This section will go a bit more into detail for some of them and shows a few example inputs of what each matcher matches.
-->
[体系的説明](../macros-methodical.md)の章で触れたように、Rust(1.60時点)には14種類のフラグメント指定子があります。
本節ではその一部についてもう少し詳しく説明し、各マッチパターンに一致する入力例を示します。

<!--
> **Note**: Capturing with anything but the `ident`, `lifetime` and `tt` fragments will render the captured AST opaque, making it impossible to further match it with other fragment specifiers in future macro invocations.
-->
> **Note**: `ident`, `lifetime`, `tt` 以外のすべてのフラグメントによる捕捉は、抽象構文木を不透明(opaque)にします。これにより、その抽象構文木は後のマクロ呼び出しにおいてそれ以上フラグメント指定子にマッチしなくなります。

* [`block`](#block)
* [`expr`](#expr)
* [`ident`](#ident)
* [`item`](#item)
* [`lifetime`](#lifetime)
* [`literal`](#literal)
* [`meta`](#meta)
* [`pat`](#pat)
* [`pat_param`](#pat_param)
* [`path`](#path)
* [`stmt`](#stmt)
* [`tt`](#tt)
* [`ty`](#ty)
* [`vis`](#vis)

## `block`

<!--
The `block` fragment solely matches a [block expression](https://doc.rust-lang.org/reference/expressions/block-expr.html), which consists of an opening `{` brace, followed by any number of statements and finally followed by a closing `}` brace.
-->
`block` フラグメントは、開き波かっこ `{`・任意の数の文・閉じ波かっこ `}` の並びからなる[ブロック式](https://doc.rust-lang.org/reference/expressions/block-expr.html)のみにマッチします。

```rust
macro_rules! blocks {
    ($($block:block)*) => ();
}

blocks! {
    {}
    {
        let zig;
    }
    { 2 }
}
# fn main() {}
```

## `expr`

<!--
The `expr` fragment matches any kind of [expression](https://doc.rust-lang.org/reference/expressions.html) (Rust has a lot of them, given it *is* an expression orientated language).
-->
`expr` フラグメントは、任意の種類の[式](https://doc.rust-lang.org/reference/expressions.html)にマッチします (Rustは「式指向 (expression oriented)」の言語なので、たくさんの種類の式があります)。

```rust
macro_rules! expressions {
    ($($expr:expr)*) => ();
}

expressions! {
    "literal"
    funcall()
    future.await
    break 'foo bar
}
# fn main() {}
```

## `ident`

<!--
The `ident` fragment matches an [identifier](https://doc.rust-lang.org/reference/identifiers.html) or *keyword*.
-->
`ident` フラグメントは、[識別子](https://doc.rust-lang.org/reference/identifiers.html)または*予約語*にマッチします。

```rust
macro_rules! idents {
    ($($ident:ident)*) => ();
}

idents! {
    // _ <- This is not an ident, it is a pattern
    // _ <- これは識別子ではなくパターン
    foo
    async
    O_________O
    _____O_____
}
# fn main() {}
```

## `item`

<!--
The `item` fragment simply matches any of Rust's [item](https://doc.rust-lang.org/reference/items.html) *definitions*, not identifiers that refer to items.
This includes visibility modifiers.
-->
`item` フラグメントは、任意のRustの[アイテム](https://doc.rust-lang.org/reference/items.html)の*定義*のみにマッチし、アイテムを参照する識別子にはマッチしません。
これには可視性修飾子を含みます。

```rust
macro_rules! items {
    ($($item:item)*) => ();
}

items! {
    struct Foo;
    enum Bar {
        Baz
    }
    impl Foo {}
    pub use crate::foo;
    /*...*/
}
# fn main() {}
```

## `lifetime`

<!--
The `lifetime` fragment matches a [lifetime or label](https://doc.rust-lang.org/reference/tokens.html#lifetimes-and-loop-labels).
It's quite similar to [`ident`](#ident) but with a prepended `'`.
-->
`lifetime` フラグメントは、[ライフタイムとラベル](https://doc.rust-lang.org/reference/tokens.html#lifetimes-and-loop-labels)にマッチします。
ライフタイムは、`'` が先頭につくことを除き[`ident`](#ident)と非常に似ています。

```rust
macro_rules! lifetimes {
    ($($lifetime:lifetime)*) => ();
}

lifetimes! {
    'static
    'shiv
    '_
}
# fn main() {}
```

## `literal`

<!--
The `literal` fragment matches any [literal expression](https://doc.rust-lang.org/reference/expressions/literal-expr.html).
-->
`literal` フラグメントは、任意の[リテラル式](https://doc.rust-lang.org/reference/expressions/literal-expr.html)にマッチします。


```rust
macro_rules! literals {
    ($($literal:literal)*) => ();
}

literals! {
    -1
    "hello world"
    2.3
    b'b'
    true
}
# fn main() {}
```

## `meta`

<!--
The `meta` fragment matches the contents of an [attribute](https://doc.rust-lang.org/reference/attributes.html).
That is, it will match a simple path, one without generic arguments followed by a delimited token tree or an `=` followed by a literal expression.
-->
`meta` フラグメントは、[属性](https://doc.rust-lang.org/reference/attributes.html)の中身にマッチします。
すなわち、それは単純パス(simple path)と呼ばれるジェネリック引数を持たないパスの後ろに、かっこで括られたトークン木または `=`とリテラル式の並びが続いたものにマッチします

<!--
> **Note**: You will usually see this fragment being used in a matcher like `#[$meta:meta]` or `#![$meta:meta]` to actually capture an attribute.
-->
**Note**: 通常、このフラグメントは、属性を捕捉するための `#[$meta:meta]` または`#![$meta:meta]` といったマッチパターンの中で使われます。

```rust
macro_rules! metas {
    ($($meta:meta)*) => ();
}

metas! {
    ASimplePath
    super::man
    path = "home"
    foo(bar)
}
# fn main() {}
```

<!--
> **Doc-Comment Fact**: Doc-Comments like `/// ...` and `!// ...` are actually syntax sugar for attributes! They desugar to `#[doc="..."]` and `#![doc="..."]` respectively, meaning you can match on them like with attributes!
-->
> **Docコメントの真実**: `/// ...` や `!// ...` のようなDocコメントは、実は属性の構文糖衣なのです！ これらはそれぞれ `#[doc="..."]` や `#![doc="..."]` という形に脱糖されます。
> つまり、Docコメントに対して属性と同様のマッチングが行えるということです！

## `pat`

<!--
The `pat` fragment matches any kind of [pattern](https://doc.rust-lang.org/reference/patterns.html), including or-patterns starting with the 2021 edition.
-->
`pat` フラグメントは、任意の種類の[パターン](https://doc.rust-lang.org/reference/patterns.html)にマッチします。これには、Rust 2021 エディションで追加されたorパターンを含みます。

```rust
macro_rules! patterns {
    ($($pat:pat)*) => ();
}

patterns! {
    "literal"
    _
    0..5
    ref mut PatternsAreNice
    0 | 1 | 2 | 3
}
# fn main() {}
```

## `pat_param`

<!--
In the 2021 edition, the behavior for the `pat` fragment type has been changed to allow or-patterns to be parsed.
This changes the follow list of the fragment, preventing such fragment from being followed by a `|` token.
To avoid this problem or to get the old fragment behavior back one can use the `pat_param` fragment which allows `|` to follow it, as it disallows top level or-patterns.
-->
Rust 2021 エディションにおいて、orパターンのパースを可能にするために `pat `フラグメントの動作が変更されました。
`pat` フラグメントに続けられるもののリストが変更され、`|` トークンを続けることができなくなりました。
この問題を回避する、または従来の動作を取り戻すのに、`pat_param` フラグメントが使えます。これには `|` を続けることができますが、一方でトップレベルのorパターンには使えません。

```rust
macro_rules! patterns {
    ($( $( $pat:pat_param )|+ )*) => ();
}

patterns! {
    "literal"
    _
    0..5
    ref mut PatternsAreNice
    0 | 1 | 2 | 3
}
# fn main() {}
```

## `path`

<!--
The `path` fragment matches a so called [TypePath](https://doc.rust-lang.org/reference/paths.html#paths-in-types) style path.
This includes the function style trait forms, `Fn() -> ()`.
-->
`path` フラグメントは、[TypePath](https://doc.rust-lang.org/reference/paths.html#paths-in-types)と呼ばれるスタイルのパスにマッチします。
これには関数スタイルのトレイト `Fn() -> ()` を含みます。

```rust
macro_rules! paths {
    ($($path:path)*) => ();
}

paths! {
    ASimplePath
    ::A::B::C::D
    G::<eneri>::C
    FnMut(u32) -> ()
}
# fn main() {}
```

## `stmt`

<!--
The `statement` fragment solely matches a [statement](https://doc.rust-lang.org/reference/statements.html) without its trailing semicolon, unless it is an item statement that requires one (such as a Unit-Struct).
-->
`stmt` フラグメントは[文](https://doc.rust-lang.org/reference/statements.html)のみにマッチします。(Unit構造体のような)セミコロンが必須のアイテム文の場合を除き、末尾のセミコロンにはマッチしません。

<!--
Let's use a simple example to show exactly what is meant with this.
We use a macro that merely emits what it captures:
-->
簡単な例によって、これが正確には何を意図しているのかを示します。
捕捉したものをそのまま出力するだけのマクロを使います:

```rust,ignore
macro_rules! statements {
    ($($stmt:stmt)*) => ($($stmt)*);
}

fn main() {
    statements! {
        struct Foo;
        fn foo() {}
        let zig = 3
        let zig = 3;
        3
        3;
        if true {} else {}
        {}
    }
}

```

<!--
Expanding this, via the [playground](https://play.rust-lang.org/) for example[^debugging], gives us roughly the following:
-->
例えば[playground](https://play.rust-lang.org/)を使って[^debugging]、これを展開すると、おおよそ次のようなものが得られます:

```rust,ignore
/* snip */

fn main() {
    struct Foo;
    fn foo() { }
    let zig = 3;
    let zig = 3;
    ;
    3;
    3;
    ;
    if true { } else { }
    { }
}
```
<!--
From this we can tell a few things.
-->
この結果からいくつかのことがいえます。

<!--
The first you should be able to see immediately is that while the `stmt` fragment doesn't capture trailing semicolons, it still emits them when required, even if the statement is already followed by one.
The simple reason for that is that semicolons on their own are already valid statements which the fragment captures eagerly.
So our macro isn't capturing 8 times, but 10!
This can be important when doing multiples repetitions and expanding these in one repetition expansion, as the repetition numbers have to match in those cases.
-->
まず、すぐにわかることは、`stmt` フラグメントが末尾のセミコロンを捕捉しないにもかかわらず、必要なところにセミコロンが出力されているということです。これは、すでに文の末尾にセミコロンがあったとしても同様です。
この理由は単純で、セミコロンはそれ自体がすでに、`stmt` フラグメントが捕捉する妥当な文だからです。
よって、このマクロは8回ではなく10回捕捉を行っているのです！
これは複数回の繰り返しを行い、それらを一つの繰り返し展開形として展開する際に重要となりえます。そのようなケースでは繰り返しの回数が一致している必要があるためです。

<!--
Another thing you should be able to notice here is that the trailing semicolon of the `struct Foo;` item statement is being matched, otherwise we would've seen an extra one like in the other cases.
This makes sense as we already said, that for item statements that require one, the trailing semicolon will be matched with.
-->
もう一つわかることは、アイテム文`struct Foo;` の末尾のセミコロンにマッチしているということです。そうでなければ、他の場合のように余計なセミコロンが出てくるはずです。
前述したように、セミコロン必須のアイテム文に関しては末尾のセミコロンがマッチ対象になるため、筋は通っています。

<!--
A last observation is that expressions get emitted back with a trailing semicolon, unless the expression solely consists of only a block expression or control flow expression.
-->
最後に、単独のブロック式や制御フロー式のみから構成される場合を除き、式が末尾のセミコロンつきで出力され直していることが観察できます。

<!--
The fine details of what was just mentioned here can be looked up in the [reference](https://doc.rust-lang.org/reference/statements.html).
-->
ここで言及したことの詳細を知るには、[リファレンス](https://doc.rust-lang.org/reference/statements.html)をあたってください。

<!--
Fortunately, these fine details here are usually not of importance whatsoever, with the small exception that was mentioned earlier in regards to repetitions which by itself shouldn't be a common problem to run into.
-->
幸い、これらの詳細は通常まったく重要ではありません。先述したように繰り返しに関する小さな例外はありますが、それだけではそこまで頻繁には問題にならないはずです。

<!--
[^debugging]:See the [debugging chapter](./debugging.md) for tips on how to do this.
-->
[^debugging]: デバッグを行う際のコツについては[デバッグの章](./debugging.md)を参照。

## `tt`

<!--
The `tt` fragment matches a TokenTree.
If you need a refresher on what exactly a TokenTree was you may want to revisit the [TokenTree chapter](../../syntax-extensions/source-analysis.md#token-trees) of this book.
The `tt` fragment is one of the most powerful fragments, as it can match nearly anything while still allowing you to inspect the contents of it at a later state in the macro.
-->
`tt` フラグメントは1つのトークン木にマッチします。
トークン木が正確には何なのかについて復習したければ、本書の[トークン木の章](../../syntax-extensions/source-analysis.md#token-trees)を読み返すとよいでしょう。
`tt `フラグメントは最も強力なフラグメントの一角です。ほぼすべてのものにマッチさせられるうえに、あとでその中身を調べる余地を残すこともできるためです。

<!--
This allows one to make use of very powerful patterns like the [tt-muncher](../patterns/tt-muncher.md) or the [push-down-accumulator](../patterns/push-down-acc.md).
-->
このフラグメントは、[tt-muncher](../patterns/tt-muncher.md)や[プッシュダウン累算器](../patterns/push-down-acc.md) (push-down-accumulator) といった非常に強力なパターンを可能にします。

## `ty`

<!--
The `ty` fragment matches any kind of [type expression](https://doc.rust-lang.org/reference/types.html#type-expressions).
-->
`ty` フラグメントは、任意の種類の[型の式](https://doc.rust-lang.org/reference/types.html#type-expressions) (type expression) にマッチします。

```rust
macro_rules! types {
    ($($type:ty)*) => ();
}

types! {
    foo::bar
    bool
    [u8]
    impl IntoIterator<Item = u32>
}
# fn main() {}
```

## `vis`

<!--
The `vis` fragment matches a *possibly empty* [Visibility qualifier].
-->
`vis` フラグメントは*空かもしれない*[可視性修飾子] (Visibility qualifier) にマッチします。

```rust
macro_rules! visibilities {
    //         ∨~~Note this comma, since we cannot repeat a `vis` fragment on its own
    //         v~~このコンマが必要なことに注意。`vis`フラグメント単体を繰り返すことはできないため。
    ($($vis:vis,)*) => ();
}

visibilities! {
    , // no vis is fine, due to the implicit `?`
      // 暗黙の `?` により、可視性なしでも問題ない
    pub,
    pub(crate),
    pub(in super),
    pub(in some_path),
}
```

<!--
While able to match empty sequences of tokens, the fragment specifier still acts quite different from [optional repetitions](../macros-methodical.md#repetitions) which is described in the following:
-->
空のトークン列にマッチさせられるとはいえ、以下に説明するように、`vis` フラグメントは[選択的繰り返し](../macros-methodical.md#repetitions)とは大きく異なった動作をします。

<!--
If it is being matched against no left over tokens the entire macro matching fails.
-->
`vis` に空トークン列をマッチさせたあとに余るトークンがない場合、マクロのマッチング全体が失敗します。

```rust
macro_rules! non_optional_vis {
    ($vis:vis) => ();
}
non_optional_vis!();
// ^^^^^^^^^^^^^^^^ error: missing tokens in macro arguments
```

<!--
`$vis:vis $ident:ident` matches fine, unlike `$(pub)? $ident:ident` which is ambiguous, as `pub` denotes a valid identifier.
-->
`$vis:vis $ident:ident` は問題なくマッチしますが、`$(pub)? $ident:ident` は `pub` が妥当な識別子であるために曖昧なのでマッチしません。

```rust
macro_rules! vis_ident {
    ($vis:vis $ident:ident) => ();
}
vis_ident!(pub foo); // this works fine
                     // これは問題なく動く

macro_rules! pub_ident {
    ($(pub)? $ident:ident) => ();
}
pub_ident!(pub foo);
        // ^^^ error: local ambiguity when calling macro `pub_ident`: multiple parsing options: built-in NTs ident ('ident') or 1 other option.
```

<!--
Being a fragment that matches the empty token sequence also gives it a very interesting quirk in combination with `tt` fragments and recursive expansions.
-->
空のトークン列にマッチするフラグメントは、`tt` フラグメントや再帰的展開と組み合わせることでとても興味深い奇妙な動作を生み出します。

<!--
When matching the empty token sequence, the metavariable will still count as a capture and since it is not a `tt`, `ident` or `lifetime` fragment it will become opaque to further expansions.
This means if this capture is passed onto another macro invocation that captures it as a `tt` you effectively end up with token tree that contains nothing!
-->
空のトークン列にマッチする場合であっても、`vis` 指定のメタ変数はやはりキャプチャとみなされ、さらに `tt`, `ident`, `lifetime`フラグメントでないため以降の展開に対して不透明になります。
このキャプチャを別のマクロを呼び出す際に `tt` として渡すと、実質的に何も含まないトークン木が手に入ることになります。

```rust
macro_rules! it_is_opaque {
    (()) => { "()" };
    (($tt:tt)) => { concat!("$tt is ", stringify!($tt)) };
    ($vis:vis ,) => { it_is_opaque!( ($vis) ); }
}
fn main() {
    // this prints "$tt is ", as the recursive calls hits the second branch with
    // an empty tt, opposed to matching with the first branch!
    // "$tt is " と表示される。再帰呼び出しが1番めの分岐にマッチしようとせず、
    // 空のトークン木(tt)を伴って2番めの分岐に到達するため
    println!("{}", it_is_opaque!(,));
}
```

[`macro_rules`]: ../macro_rules.md
[可視性修飾子]: https://doc.rust-lang.org/reference/visibility-and-privacy.html
