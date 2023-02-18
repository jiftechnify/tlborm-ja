<!--
# Macros, A Practical Introduction
-->
# マクロ: 実践的説明

<!--
This chapter will introduce Rust's declarative [Macro-By-Example](https://doc.rust-lang.org/reference/macros-by-example.html) system using a relatively simple, practical example.
It does *not* attempt to explain all of the intricacies of the system; its goal is to get you comfortable with how and why macros are written.
-->
本章では、Rustの宣言的な[Macro-By-Example](https://doc.rust-lang.org/reference/macros-by-example.html)のシステムについて、比較的シンプルで実践的な例を通して説明していきます。

<!--
There is also the [Macros chapter of the Rust Book](https://doc.rust-lang.org/book/ch19-06-macros.html) which is another high-level explanation, and the [methodical introduction](../decl-macros.md) chapter of this book, which explains the macro system in detail.
-->
高水準な視点からの説明としては、他にも[The Rust Bookのマクロの章](https://doc.rust-lang.org/book/ch19-06-macros.html)があります。
また本書の[形式的説明](../decl-macros.md)の章では、このマクロシステムについて詳細に説明しています。

<!--
## A Little Context
-->
## 背景を少し

<!--
> **Note**: don't panic! What follows is the only math that will be talked about.
> You can quite safely skip this section if you just want to get to the meat of the article.
-->

> **Note**: 落ち着いて！ これに続くのはマクロの説明に関係するちょっとした数学の話です。
> 早くこの章の本題に入りたいのであれば、この節を飛ばして読んでも大丈夫です。

<!--
If you aren't familiar, a recurrence relation is a sequence where each value is defined in terms of one or more *previous* values, with one or more initial values to get the whole thing started.
For example, the [Fibonacci sequence](https://en.wikipedia.org/wiki/Fibonacci_number) can be defined by the relation:
-->
詳しくない方向けに説明すると、漸化式とは、各値が1つ以上*前の*値に基づいて定まる数列で、全ての始まりである1つ以上の初期値を伴います。
例えば、[フィボナッチ数列](https://en.wikipedia.org/wiki/Fibonacci_number)[^fib-wikipedia-ja]は次の漸化式により定義されます:

\\[F_{n} = 0, 1, ..., F_{n-2} + F_{n-1}\\]

[^fib-wikipedia-ja]: *訳注*: 日本語版は[こちら](https://en.wikipedia.org/wiki/Fibonacci_number)。

<!--
Thus, the first two numbers in the sequence are 0 and 1, with the third being \\( F_{0} + F_{1} = 0 + 1 = 1\\), the fourth \\( F_{1} + F_{2} = 1 + 1 = 2\\), and so on forever.
-->
したがって、数列の最初の2つの数は 0 と 1、3番めは \\( F_{0} + F_{1} = 0 + 1 = 1\\)、 4番めは \\( F_{1} + F_{2} = 1 + 1 = 2\\)、という具合に無限に続きます。

<!--
Now, *because* such a sequence can go on forever, that makes defining a `fibonacci` function a little tricky, since you obviously don't want to try returning a complete vector.
What you *want* is to return something which will lazily compute elements of the sequence as needed.
-->
さて、このような数列は無限に続く*ため*、`fibonacci` 関数を定義するのは少しややこしい作業になります。というのも、明らかに完全なベクタを返すべきではないからです。
ここで*すべき*ことは、必要に応じて数列の要素を遅延的に計算する何かを返すことです。

<!--
In Rust, that means producing an [`Iterator`].
This is not especially *hard*, but there is a fair amount of boilerplate involved: you need to define a custom type, work out what state needs to be stored in it, then implement the [`Iterator`] trait for it.
-->
Rustにおいて、これは[`Iterator`]を生成せよ、ということです。
これは特別難しいことではありませんが、かなりの量のボイラープレートを必要とします。独自の型を定義し、その型に保存すべき状態を考え出し、[`Iterator`]トレイトを実装する必要があります。

<!--
However, recurrence relations are simple enough that almost all of these details can be abstracted out with a little `macro_rules!` macro-based code generation.
-->
ですが、小さな`macro_rules!`マクロに基づくコード生成だけでこれらの詳細のほとんどを括りだすことができるくらい、漸化式はシンプルです。

<!--
So, with all that having been said, let's get started.
-->
それでは、以上のことを踏まえて、早速始めていきましょう。

[`Iterator`]:https://doc.rust-lang.org/std/iter/trait.Iterator.html

<!--
## Construction
-->
## 構成要素

<!--
Usually, when working on a new `macro_rules!` macro, the first thing I do is decide what the invocation should look like.
In this specific case, my first attempt looked like this:
-->
たいてい、新しい `macro_rules!` マクロの実装に取りかかるとき、私が初めにするのはその呼び出し方を決めることです。
今回のケースでは、最初の試行は次のようなものになりました:

```rust,ignore
let fib = recurrence![a[n] = 0, 1, ..., a[n-2] + a[n-1]];

for e in fib.take(10) { println!("{}", e) }
```
<!--
From that, we can take a stab at how the `macro_rules!` macro should be defined, even if we aren't sure of the actual expansion.
This is useful because if you can't figure out how to parse the input syntax, then *maybe* you need to change it.
-->
これをもとに、実際の展開形について確信は持てなくとも、`macro_rules!` マクロがどのように定義されるべきかを考えてみることはできます。
入力の構文をパースする方法を思いつけないのであれば、構文を変更する必要があるかもしれないということなので、これは有用な考え方です。

```rust,ignore
macro_rules! recurrence {
    ( a[n] = $($inits:expr),+ , ... , $recur:expr ) => { /* ... */ };
}
# fn main() {}
```

<!--
Assuming you aren't familiar with the syntax, allow me to elucidate.
This is defining a syntax extension, using the [`macro_rules!`] system, called `recurrence!`.
This `macro_rules!` macro has a single parsing rule.
That rule says the input to the invocation must match:
-->
この構文は見慣れないものだと思いますので、少し説明させてください。
これは `recurrence!` という名前の、[`macro_rules!`]のシステムを使った構文拡張の定義になります。
この `macro_rules!` マクロはただ一つの構文ルールを持っています。
そのルールは、呼び出しの入力が次のものに一致しなければならないというものです:

<!--
- the literal token sequence `a` `[` `n` `]` `=`,
- a [repeating] (the `$( ... )`) sequence, using `,` as a separator, and one or more (`+`) repeats of:
    - a valid *expression* captured into the [metavariable] `inits` (`$inits:expr`)
- the literal token sequence `,` `...` `,`,
- a valid *expression* captured into the [metavariable] `recur` (`$recur:expr`).
-->
- リテラルトークンの列 `a` `[` `n` `]` `=`
- `,` を区切りとする、1回以上 (`+`) の妥当な*式*の[繰り返し] (`$( ... )`)。この式は[メタ変数] `inits` に捕捉される (`$inits:expr`)
- リテラルトークンの列 `,` `...` `,`
- 妥当な*式*。この式は[メタ変数] `recur` に捕捉される (`$recur:expr`)

[繰り返し]: ./macros-methodical.md#repetitions
[メタ変数]: ./macros-methodical.md#metavariables

<!--
Finally, the rule says that *if* the input matches this rule, then the invocation should be replaced by the token sequence `/* ... */`.
-->
結局、このルールは、*もし*入力がこのルールに一致したら、マクロの呼び出しを `/* ... */` というトークンの列で置き換えよ、ということを表しています。

<!--
It's worth noting that `inits`, as implied by the name, actually contains *all* the expressions that match in this position, not just the first or last.
What's more, it captures them *as a sequence* as opposed to, say, irreversibly pasting them all together.
Also note that you can do "zero or more" with a repetition by using `*` instead of `+` and even optional, "zero or one" with `?`.
-->
`inits` は、その名前が示唆するように、最初や最後だけではなく、その位置にある*すべての*式を含むことに注意してください。
さらにいえば、`inits` は、それらの式を不可逆的にまとめてペーストするような形ではなく、*列として*捕捉します。
また、`+` の代わりに `*` を使えば「0回以上」の繰り返しを、`?`を使えば「任意」、つまり「0回か1回」の繰り返しを表せます。

<!--
As an exercise, let's take the proposed input and feed it through the rule, to see how it is processed.
The "Position" column will show which part of the syntax pattern needs to be matched against next, denoted by a "⌂".
Note that in some cases, there might be more than one possible "next" element to match against.
"Input" will contain all of the tokens that have *not* been consumed yet.
`inits` and `recur` will contain the contents of those bindings.
-->
練習として、先に挙げた入力をこのルールに与えてみて、どのように処理されるか見てみましょう。
「位置」欄では、次に構文パターンのどの部分がマッチングされるかを「 ⌂ 」で示しています。
ある状況では、マッチング対象となる「次」の要素の候補が複数存在することがあるのに注意してください。
「入力」欄は、まだ消費されていないトークンです。
`inits`・`recur` 欄はそれらに捕捉されている内容です。

{{#include macros-practical-table.html}}

<!--
The key take-away from this is that the macro system will *try* to incrementally match the tokens provided as input to the macro against the provided rules.
We'll come back to the "try" part.
-->
ここで重要なのは、マクロシステムが、入力として与えられたトークンたちを所与のルールに対してインクリメンタルにマッチングを*試みる*ということです。
「試みる」という部分については後で補足します。

<!--
Now, let's begin writing the final, fully expanded form.
For this expansion, I was looking for something like:
-->
さて、最後の、完全に展開された形を書きはじめましょう。
この展開に対しては、次のようなものが求められています:

```rust,ignore
let fib = {
    struct Recurrence {
        mem: [u64; 2],
        pos: usize,
    }
```
<!--
This will be the actual iterator type.
`mem` will be the memo buffer to hold the last few values so the recurrence can be computed.
`pos` is to keep track of the value of `n`.
-->
これは実際のイテレータ型になるべきものです。
`mem` は漸化式を計算するのに必要となる、直近の数個の値を保持するメモバッファになります。
`pos` は `n` の値を追跡するための変数です。

<!--
> **Aside**: I've chosen `u64` as a "sufficiently large" type for the elements of this sequence.
> Don't worry about how this will work out for *other* sequences; we'll come to it.
-->
> **余談**: `u64` は、数列の要素を表すのに「十分大きな」型として選びました。
> これが*他の*数列に対して上手くいくかを心配する必要はありません。きっと上手くいきますよ。

```rust,ignore
    impl Iterator for Recurrence {
        type Item = u64;

        fn next(&mut self) -> Option<Self::Item> {
            if self.pos < 2 {
                let next_val = self.mem[self.pos];
                self.pos += 1;
                Some(next_val)
```

<!--
We need a branch to yield the initial values of the sequence; nothing tricky.
-->
数列の初期値を生成する分岐が必要です。難しいところはないでしょう。

```rust,ignore
            } else {
                let a = /* something */;
                let n = self.pos;
                let next_val = a[n-2] + a[n-1];

                self.mem.TODO_shuffle_down_and_append(next_val);

                self.pos += 1;
                Some(next_val)
            }
        }
    }
```

<!--
This is a bit harder; we'll come back and look at *how* exactly to define `a`.
Also, `TODO_shuffle_down_and_append` is another placeholder;
I want something that places `next_val` on the end of the array, shuffling the rest down by one space, dropping the 0th element.
-->
こちらはちょっと難しいです。`a`を厳密にどう定義するかについては、あとで見ていきます。
`TODO_shuffle_down_and_append` も仮実装になっています。
ここには、`next_val` を配列の末尾に配置し、残りの要素を1つずつずらし、最初の要素を削除するものが必要です。

```rust,ignore

    Recurrence { mem: [0, 1], pos: 0 }
};

for e in fib.take(10) { println!("{}", e) }
```

<!--
Lastly, return an instance of our new structure, which can then be iterated over.
To summarize, the complete expansion is:
-->
最後に、この新しい構造体のインスタンスを返します。これに対して反復処理を行うことができます。
まとめると、展開形の全容は以下のようになります:

```rust,ignore
let fib = {
    struct Recurrence {
        mem: [u64; 2],
        pos: usize,
    }

    impl Iterator for Recurrence {
        type Item = u64;

        fn next(&mut self) -> Option<u64> {
            if self.pos < 2 {
                let next_val = self.mem[self.pos];
                self.pos += 1;
                Some(next_val)
            } else {
                let a = /* something */;
                let n = self.pos;
                let next_val = (a[n-2] + a[n-1]);

                self.mem.TODO_shuffle_down_and_append(next_val.clone());

                self.pos += 1;
                Some(next_val)
            }
        }
    }

    Recurrence { mem: [0, 1], pos: 0 }
};

for e in fib.take(10) { println!("{}", e) }
```

<!--
> **Aside**: Yes, this *does* mean we're defining a different `Recurrence` struct and its implementation for each invocation.
> Most of this will optimise away in the final binary.
-->
> **余談**: そう、これはマクロの呼び出しのたびに別の `Recurrence` 構造体とその実装を定義することを意味します。
> ほとんどの部分は最終的なバイナリ上では最適化されるでしょう。

<!--
It's also useful to check your expansion as you're writing it.
If you see anything in the expansion that needs to vary with the invocation, but *isn't* in the actual accepted syntax of our macro, you should work out where to introduce it.
In this case, we've added `u64`, but that's not necessarily what the user wants, nor is it in the macro syntax. So let's fix that.
-->
展開形を書きながら、それを見直すのも有用です。
展開形の中に、呼び出しのたびに異なるべき何かがあって、それがマクロが実際に受け入れる構文の中に*ない*のであれば、それをどこに導入するか考える必要があります。
今回の例では、`u64`を追加しましたが、それはユーザにとってもマクロ構文中にも必ずしも必要なものではありません。修正しましょう。

```rust
macro_rules! recurrence {
    ( a[n]: $sty:ty = $($inits:expr),+ , ... , $recur:expr ) => { /* ... */ };
}

/*
let fib = recurrence![a[n]: u64 = 0, 1, ..., a[n-2] + a[n-1]];

for e in fib.take(10) { println!("{}", e) }
*/
# fn main() {}
```

<!--
Here, I've added a new metavariable: `sty` which should be a type.
-->
新たにメタ変数 `sty` を追加しました。これは型にマッチします。

<!--
> **Aside**: if you're wondering, the bit after the colon in a metavariable can be one of several kinds of syntax matchers.
> The most common ones are `item`, `expr`, and `ty`.
> A complete explanation can be found in [Macros, A Methodical Introduction; `macro_rules!` (Matchers)](./macros-methodical.md#metavariables).
>
> There's one other thing to be aware of: in the interests of future-proofing the language, the compiler restricts what tokens you're allowed to put *after* a matcher, depending on what kind it is.
> Typically, this comes up when trying to match expressions or statements;
> those can *only* be followed by one of `=>`, `,`, and `;`.
>
> A complete list can be found in [Macros, A Methodical Introduction; Minutiae; Metavariables and Expansion Redux](./minutiae/metavar-and-expansion.md).
-->
> **余談**: メタ変数のコロン以降の部分は、マッチする構文の種類を表します。
> よく使われるのは `item`, `expr`, そして `ty` です。
> 詳しい説明は [「マクロ: 形式的説明」 の章の 「メタ変数」の項目](./macros-methodical.md#metavariables)をご覧ください。
>
> もう一つ知っておくべきことがあります。言語の将来の変化に備える意味で、コンパイラはマッチパターンの種類に応じて、その*あとに*続けられるトークンの種類に制限を設けています。
> 概して、これは式や文にマッチングさせようとしたときに問題になります。
> これらのあとに続けられるのは `=>`, `,`, `;` *のみ*となります。
>
> 完全なリストは[「枝葉末節」の章の「メタ変数と展開・再考」の節](./minutiae/metavar-and-expansion.md)にあります。

## Indexing and Shuffling

I will skim a bit over this part, since it's effectively tangential to the macro-related stuff.
We want to make it so that the user can access previous values in the sequence by indexing `a`;
we want it to act as a sliding window keeping the last few (in this case, 2) elements of the sequence.

We can do this pretty easily with a wrapper type:

```rust,ignore
struct IndexOffset<'a> {
    slice: &'a [u64; 2],
    offset: usize,
}

impl<'a> Index<usize> for IndexOffset<'a> {
    type Output = u64;

    fn index<'b>(&'b self, index: usize) -> &'b u64 {
        use std::num::Wrapping;

        let index = Wrapping(index);
        let offset = Wrapping(self.offset);
        let window = Wrapping(2);

        let real_index = index - offset + window;
        &self.slice[real_index.0]
    }
}
```

> **Aside**: since lifetimes come up *a lot* with people new to Rust, a quick explanation: `'a` and `'b` are lifetime parameters that are used to track where a reference (*i.e.* a borrowed pointer to some data) is valid.
> In this case, `IndexOffset` borrows a reference to our iterator's data, so it needs to keep track of how long it's allowed to hold that reference for, using `'a`.
>
> `'b` is used because the `Index::index` function (which is how subscript syntax is actually implemented) is *also* parameterized on a lifetime, on account of returning a borrowed reference.
> `'a` and `'b` are not necessarily the same thing in all cases.
> The borrow checker will make sure that even though we don't explicitly relate `'a` and `'b` to one another, we don't accidentally violate memory safety.

This changes the definition of `a` to:

```rust,ignore
let a = IndexOffset { slice: &self.mem, offset: n };
```

The only remaining question is what to do about `TODO_shuffle_down_and_append`.
I wasn't able to find a method in the standard library with exactly the semantics I wanted, but it isn't hard to do by hand.

```rust,ignore
{
    use std::mem::swap;

    let mut swap_tmp = next_val;
    for i in (0..2).rev() {
        swap(&mut swap_tmp, &mut self.mem[i]);
    }
}
```

This swaps the new value into the end of the array, swapping the other elements down one space.

> **Aside**: doing it this way means that this code will work for non-copyable types, as well.

The working code thus far now looks like this:

```rust
macro_rules! recurrence {
    ( a[n]: $sty:ty = $($inits:expr),+ , ... , $recur:expr ) => { /* ... */ };
}

fn main() {
    /*
    let fib = recurrence![a[n]: u64 = 0, 1, ..., a[n-2] + a[n-1]];

    for e in fib.take(10) { println!("{}", e) }
    */
    let fib = {
        use std::ops::Index;

        struct Recurrence {
            mem: [u64; 2],
            pos: usize,
        }

        struct IndexOffset<'a> {
            slice: &'a [u64; 2],
            offset: usize,
        }

        impl<'a> Index<usize> for IndexOffset<'a> {
            type Output = u64;

            #[inline(always)]
            fn index<'b>(&'b self, index: usize) -> &'b u64 {
                use std::num::Wrapping;

                let index = Wrapping(index);
                let offset = Wrapping(self.offset);
                let window = Wrapping(2);

                let real_index = index - offset + window;
                &self.slice[real_index.0]
            }
        }

        impl Iterator for Recurrence {
            type Item = u64;

            #[inline]
            fn next(&mut self) -> Option<u64> {
                if self.pos < 2 {
                    let next_val = self.mem[self.pos];
                    self.pos += 1;
                    Some(next_val)
                } else {
                    let next_val = {
                        let n = self.pos;
                        let a = IndexOffset { slice: &self.mem, offset: n };
                        a[n-2] + a[n-1]
                    };

                    {
                        use std::mem::swap;

                        let mut swap_tmp = next_val;
                        for i in [1,0] {
                            swap(&mut swap_tmp, &mut self.mem[i]);
                        }
                    }

                    self.pos += 1;
                    Some(next_val)
                }
            }
        }

        Recurrence { mem: [0, 1], pos: 0 }
    };

    for e in fib.take(10) { println!("{}", e) }
}
```

Note that I've changed the order of the declarations of `n` and `a`, as well as wrapped them(along with the recurrence expression) in a block.
The reason for the first should be obvious(`n` needs to be defined first so I can use it for `a`).
The reason for the second is that the borrowed reference `&self.mem` will prevent the swaps later on from happening (you cannot mutate something that is aliased elsewhere). The block ensures that the `&self.mem` borrow expires before then.

Incidentally, the only reason the code that does the `mem` swaps is in a block is to narrow the scope in which `std::mem::swap` is available, for the sake of being tidy.

If we take this code and run it, we get:

```text
0
1
1
2
3
5
8
13
21
34
```

Success!
Now, let's copy & paste this into the macro expansion, and replace the expanded code with an invocation.
This gives us:

```rust
macro_rules! recurrence {
    ( a[n]: $sty:ty = $($inits:expr),+ , ... , $recur:expr ) => {
        {
            /*
                What follows here is *literally* the code from before,
                cut and pasted into a new position. No other changes
                have been made.
            */

            use std::ops::Index;

            struct Recurrence {
                mem: [u64; 2],
                pos: usize,
            }

            struct IndexOffset<'a> {
                slice: &'a [u64; 2],
                offset: usize,
            }

            impl<'a> Index<usize> for IndexOffset<'a> {
                type Output = u64;

                fn index<'b>(&'b self, index: usize) -> &'b u64 {
                    use std::num::Wrapping;

                    let index = Wrapping(index);
                    let offset = Wrapping(self.offset);
                    let window = Wrapping(2);

                    let real_index = index - offset + window;
                    &self.slice[real_index.0]
                }
            }

            impl Iterator for Recurrence {
                type Item = u64;

                fn next(&mut self) -> Option<u64> {
                    if self.pos < 2 {
                        let next_val = self.mem[self.pos];
                        self.pos += 1;
                        Some(next_val)
                    } else {
                        let next_val = {
                            let n = self.pos;
                            let a = IndexOffset { slice: &self.mem, offset: n };
                            (a[n-2] + a[n-1])
                        };

                        {
                            use std::mem::swap;

                            let mut swap_tmp = next_val;
                            for i in (0..2).rev() {
                                swap(&mut swap_tmp, &mut self.mem[i]);
                            }
                        }

                        self.pos += 1;
                        Some(next_val)
                    }
                }
            }

            Recurrence { mem: [0, 1], pos: 0 }
        }
    };
}

fn main() {
    let fib = recurrence![a[n]: u64 = 0, 1, ..., a[n-2] + a[n-1]];

    for e in fib.take(10) { println!("{}", e) }
}
```

Obviously, we aren't *using* the metavariables yet, but we can change that fairly easily.
However, if we try to compile this, `rustc` aborts, telling us:

```text
error: local ambiguity: multiple parsing options: built-in NTs expr ('inits') or 1 other option.
  --> src/main.rs:75:45
   |
75 |     let fib = recurrence![a[n]: u64 = 0, 1, ..., a[n-2] + a[n-1]];
   |
```

Here, we've run into a limitation of the `macro_rules` system.
The problem is that second comma.
When it sees it during expansion, `macro_rules` can't decide if it's supposed to parse *another* expression for `inits`, or `...`.
Sadly, it isn't quite clever enough to realise that `...` isn't a valid expression, so it gives up.
Theoretically, this *should* work as desired, but currently doesn't.

> **Aside**: I *did* fib a little about how our rule would be interpreted by the macro system.
> In general, it *should* work as described, but doesn't in this case.
> The `macro_rules` machinery, as it stands, has its foibles, and its worthwhile remembering that on occasion, you'll need to contort a little to get it to work.
>
> In this *particular* case, there are two issues.
> First, the macro system doesn't know what does and does not constitute the various grammar elements (*e.g.* an expression); that's the parser's job.
> As such, it doesn't know that `...` isn't an expression.
> Secondly, it has no way of trying to capture a compound grammar element (like an expression) without 100% committing to that capture.
>
> In other words, it can ask the parser to try and parse some input as an expression, but the parser will respond to any problems by aborting.
> The only way the macro system can currently deal with this is to just try to forbid situations where this could be a problem.
>
> On the bright side, this is a state of affairs that exactly *no one* is enthusiastic about.
> The `macro` keyword has already been reserved for a more rigorously-defined future [macro system](https://github.com/rust-lang/rust/issues/39412).
> Until then, needs must.

Thankfully, the fix is relatively simple: we remove the comma from the syntax.
To keep things balanced, we'll remove *both* commas around `...`:

```rust
macro_rules! recurrence {
    ( a[n]: $sty:ty = $($inits:expr),+ ... $recur:expr ) => {
//                                     ^~~ changed
        /* ... */
#         // Cheat :D
#         (vec![0u64, 1, 2, 3, 5, 8, 13, 21, 34]).into_iter()
    };
}

fn main() {
    let fib = recurrence![a[n]: u64 = 0, 1 ... a[n-2] + a[n-1]];
//                                         ^~~ changed

    for e in fib.take(10) { println!("{}", e) }
}
```

Success! ... or so we thought.
Turns out this is being rejected by the compiler nowadays, while it was fine back when this was written.
The reason for this is that the compiler now recognizes the `...` as a token, and as we know we may only use `=>`, `,` or `;` after an expression fragment.
So unfortunately we are now out of luck as our dreamed up syntax will not work out this way, so let us just choose one that looks the most befitting that we are allowed to use instead, I'd say replacing `,` with `;` works.

```rust
macro_rules! recurrence {
    ( a[n]: $sty:ty = $($inits:expr),+ ; ... ; $recur:expr ) => {
//                                     ^~~~~~^ changed
        /* ... */
#         // Cheat :D
#         (vec![0u64, 1, 2, 3, 5, 8, 13, 21, 34]).into_iter()
    };
}

fn main() {
    let fib = recurrence![a[n]: u64 = 0, 1; ...; a[n-2] + a[n-1]];
//                                        ^~~~~^ changed

    for e in fib.take(10) { println!("{}", e) }
}
```

Success! But for real this time.


### Substitution

Substituting something you've captured in a macro is quite simple; you can insert the contents of a metavariable `$sty:ty` by using `$sty`.
So, let's go through and fix the `u64`s:

```rust
macro_rules! recurrence {
    ( a[n]: $sty:ty = $($inits:expr),+ ; ... ; $recur:expr ) => {
        {
            use std::ops::Index;

            struct Recurrence {
                mem: [$sty; 2],
//                    ^~~~ changed
                pos: usize,
            }

            struct IndexOffset<'a> {
                slice: &'a [$sty; 2],
//                          ^~~~ changed
                offset: usize,
            }

            impl<'a> Index<usize> for IndexOffset<'a> {
                type Output = $sty;
//                            ^~~~ changed

                #[inline(always)]
                fn index<'b>(&'b self, index: usize) -> &'b $sty {
//                                                          ^~~~ changed
                    use std::num::Wrapping;

                    let index = Wrapping(index);
                    let offset = Wrapping(self.offset);
                    let window = Wrapping(2);

                    let real_index = index - offset + window;
                    &self.slice[real_index.0]
                }
            }

            impl Iterator for Recurrence {
                type Item = $sty;
//                          ^~~~ changed

                #[inline]
                fn next(&mut self) -> Option<$sty> {
//                                           ^~~~ changed
                    /* ... */
#                     if self.pos < 2 {
#                         let next_val = self.mem[self.pos];
#                         self.pos += 1;
#                         Some(next_val)
#                     } else {
#                         let next_val = {
#                             let n = self.pos;
#                             let a = IndexOffset { slice: &self.mem, offset: n };
#                             (a[n-2] + a[n-1])
#                         };
#
#                         {
#                             use std::mem::swap;
#
#                             let mut swap_tmp = next_val;
#                             for i in (0..2).rev() {
#                                 swap(&mut swap_tmp, &mut self.mem[i]);
#                             }
#                         }
#
#                         self.pos += 1;
#                         Some(next_val)
#                     }
                }
            }

            Recurrence { mem: [0, 1], pos: 0 }
        }
    };
}

fn main() {
    let fib = recurrence![a[n]: u64 = 0, 1; ...; a[n-2] + a[n-1]];

    for e in fib.take(10) { println!("{}", e) }
}
```

Let's tackle a harder one: how to turn `inits` into both the array literal `[0, 1]` *and* the array type, `[$sty; 2]`.
The first one we can do like so:

```rust,ignore
            Recurrence { mem: [$($inits),+], pos: 0 }
//                             ^~~~~~~~~~~ changed
```

This effectively does the opposite of the capture: repeat `inits` one or more times, separating each with a comma.
This expands to the expected sequence of tokens: `0, 1`.

Somehow turning `inits` into a literal `2` is a little trickier.
It turns out that there's no direct way to do this, but we *can* do it by using a second `macro_rules!` macro.
Let's take this one step at a time.

```rust
macro_rules! count_exprs {
    /* ??? */
#     () => {}
}
# fn main() {}
```

The obvious case is: given zero expressions, you would expect `count_exprs` to expand to a literal
`0`.

```rust
macro_rules! count_exprs {
    () => (0);
//  ^~~~~~~~~~ added
}
# fn main() {
#     const _0: usize = count_exprs!();
#     assert_eq!(_0, 0);
# }
```

> **Aside**: You may have noticed I used parentheses here instead of curly braces for the expansion.
> `macro_rules` really doesn't care *what* you use, so long as it's one of the "matcher" pairs: `( )`, `{ }` or `[ ]`.
> In fact, you can switch out the matchers on the macro itself(*i.e.* the matchers right after the macro name), the matchers around the syntax rule, and the matchers around the corresponding expansion.
>
> You can also switch out the matchers used when you *invoke* a macro, but in a more limited fashion: a macro invoked as `{ ... }` or `( ... );` will *always* be parsed as an *item* (*i.e.* like a `struct` or `fn` declaration).
> This is important when using macros in a function body; it helps disambiguate between "parse like an expression" and "parse like a statement".

What if you have *one* expression?
That should be a literal `1`.

```rust
macro_rules! count_exprs {
    () => (0);
    ($e:expr) => (1);
//  ^~~~~~~~~~~~~~~~~ added
}
# fn main() {
#     const _0: usize = count_exprs!();
#     const _1: usize = count_exprs!(x);
#     assert_eq!(_0, 0);
#     assert_eq!(_1, 1);
# }
```

Two?

```rust
macro_rules! count_exprs {
    () => (0);
    ($e:expr) => (1);
    ($e0:expr, $e1:expr) => (2);
//  ^~~~~~~~~~~~~~~~~~~~~~~~~~~~ added
}
# fn main() {
#     const _0: usize = count_exprs!();
#     const _1: usize = count_exprs!(x);
#     const _2: usize = count_exprs!(x, y);
#     assert_eq!(_0, 0);
#     assert_eq!(_1, 1);
#     assert_eq!(_2, 2);
# }
```

We can "simplify" this a little by re-expressing the case of two expressions recursively.

```rust
macro_rules! count_exprs {
    () => (0);
    ($e:expr) => (1);
    ($e0:expr, $e1:expr) => (1 + count_exprs!($e1));
//                           ^~~~~~~~~~~~~~~~~~~~~ changed
}
# fn main() {
#     const _0: usize = count_exprs!();
#     const _1: usize = count_exprs!(x);
#     const _2: usize = count_exprs!(x, y);
#     assert_eq!(_0, 0);
#     assert_eq!(_1, 1);
#     assert_eq!(_2, 2);
# }
```

This is fine since Rust can fold `1 + 1` into a constant value.
What if we have three expressions?

```rust
macro_rules! count_exprs {
    () => (0);
    ($e:expr) => (1);
    ($e0:expr, $e1:expr) => (1 + count_exprs!($e1));
    ($e0:expr, $e1:expr, $e2:expr) => (1 + count_exprs!($e1, $e2));
//  ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ added
}
# fn main() {
#     const _0: usize = count_exprs!();
#     const _1: usize = count_exprs!(x);
#     const _2: usize = count_exprs!(x, y);
#     const _3: usize = count_exprs!(x, y, z);
#     assert_eq!(_0, 0);
#     assert_eq!(_1, 1);
#     assert_eq!(_2, 2);
#     assert_eq!(_3, 3);
# }
```

> **Aside**: You might be wondering if we could reverse the order of these rules.
> In this particular case, *yes*, but the macro system can sometimes be picky about what it is and is not willing to recover from.
> If you ever find yourself with a multi-rule macro that you *swear* should work, but gives you errors about unexpected tokens, try changing the order of the rules.

Hopefully, you can see the pattern here.
We can always reduce the list of expressions by matching one expression, followed by zero or more expressions, expanding that into 1 + a count.

```rust
macro_rules! count_exprs {
    () => (0);
    ($head:expr) => (1);
    ($head:expr, $($tail:expr),*) => (1 + count_exprs!($($tail),*));
//  ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ changed
}
# fn main() {
#     const _0: usize = count_exprs!();
#     const _1: usize = count_exprs!(x);
#     const _2: usize = count_exprs!(x, y);
#     const _3: usize = count_exprs!(x, y, z);
#     assert_eq!(_0, 0);
#     assert_eq!(_1, 1);
#     assert_eq!(_2, 2);
#     assert_eq!(_3, 3);
# }
```

> **<abbr title="Just for this example">JFTE</abbr>**: this is not the *only*, or even the *best* way of counting things.
> You may wish to peruse the [Counting](./building-blocks/counting.md) section later for a more efficient way.

With this, we can now modify `recurrence` to determine the necessary size of `mem`.

```rust
// added:
macro_rules! count_exprs {
    () => (0);
    ($head:expr) => (1);
    ($head:expr, $($tail:expr),*) => (1 + count_exprs!($($tail),*));
}

macro_rules! recurrence {
    ( a[n]: $sty:ty = $($inits:expr),+ ; ... ; $recur:expr ) => {
        {
            use std::ops::Index;

            const MEM_SIZE: usize = count_exprs!($($inits),+);
//          ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ added

            struct Recurrence {
                mem: [$sty; MEM_SIZE],
//                          ^~~~~~~~ changed
                pos: usize,
            }

            struct IndexOffset<'a> {
                slice: &'a [$sty; MEM_SIZE],
//                                ^~~~~~~~ changed
                offset: usize,
            }

            impl<'a> Index<usize> for IndexOffset<'a> {
                type Output = $sty;

                #[inline(always)]
                fn index<'b>(&'b self, index: usize) -> &'b $sty {
                    use std::num::Wrapping;

                    let index = Wrapping(index);
                    let offset = Wrapping(self.offset);
                    let window = Wrapping(MEM_SIZE);
//                                        ^~~~~~~~ changed

                    let real_index = index - offset + window;
                    &self.slice[real_index.0]
                }
            }

            impl Iterator for Recurrence {
                type Item = $sty;

                #[inline]
                fn next(&mut self) -> Option<$sty> {
                    if self.pos < MEM_SIZE {
//                                ^~~~~~~~ changed
                        let next_val = self.mem[self.pos];
                        self.pos += 1;
                        Some(next_val)
                    } else {
                        let next_val = {
                            let n = self.pos;
                            let a = IndexOffset { slice: &self.mem, offset: n };
                            (a[n-2] + a[n-1])
                        };

                        {
                            use std::mem::swap;

                            let mut swap_tmp = next_val;
                            for i in (0..MEM_SIZE).rev() {
//                                       ^~~~~~~~ changed
                                swap(&mut swap_tmp, &mut self.mem[i]);
                            }
                        }

                        self.pos += 1;
                        Some(next_val)
                    }
                }
            }

            Recurrence { mem: [$($inits),+], pos: 0 }
        }
    };
}
/* ... */
#
# fn main() {
#     let fib = recurrence![a[n]: u64 = 0, 1; ...; a[n-2] + a[n-1]];
#
#     for e in fib.take(10) { println!("{}", e) }
# }
```

With that done, we can now substitute the last thing: the `recur` expression.

```rust
# macro_rules! count_exprs {
#     () => (0);
#     ($head:expr $(, $tail:expr)*) => (1 + count_exprs!($($tail),*));
# }
# macro_rules! recurrence {
#     ( a[n]: $sty:ty = $($inits:expr),+ ; ... ; $recur:expr ) => {
#         {
#             use std::ops::Index;
#
#             const MEM_SIZE: usize = count_exprs!($($inits),+);
#             struct Recurrence {
#                 mem: [$sty; MEM_SIZE],
#                 pos: usize,
#             }
#             struct IndexOffset<'a> {
#                 slice: &'a [$sty; MEM_SIZE],
#                 offset: usize,
#             }
#             impl<'a> Index<usize> for IndexOffset<'a> {
#                 type Output = $sty;
#
#                 #[inline(always)]
#                 fn index<'b>(&'b self, index: usize) -> &'b $sty {
#                     use std::num::Wrapping;
#
#                     let index = Wrapping(index);
#                     let offset = Wrapping(self.offset);
#                     let window = Wrapping(MEM_SIZE);
#
#                     let real_index = index - offset + window;
#                     &self.slice[real_index.0]
#                 }
#             }
#             impl Iterator for Recurrence {
#               type Item = $sty;
/* ... */
                #[inline]
                fn next(&mut self) -> Option<u64> {
                    if self.pos < MEM_SIZE {
                        let next_val = self.mem[self.pos];
                        self.pos += 1;
                        Some(next_val)
                    } else {
                        let next_val = {
                            let n = self.pos;
                            let a = IndexOffset { slice: &self.mem, offset: n };
                            $recur
//                          ^~~~~~ changed
                        };
                        {
                            use std::mem::swap;
                            let mut swap_tmp = next_val;
                            for i in (0..MEM_SIZE).rev() {
                                swap(&mut swap_tmp, &mut self.mem[i]);
                            }
                        }
                        self.pos += 1;
                        Some(next_val)
                    }
                }
/* ... */
#             }
#             Recurrence { mem: [$($inits),+], pos: 0 }
#         }
#     };
# }
# fn main() {
#     let fib = recurrence![a[n]: u64 = 1, 1; ...; a[n-2] + a[n-1]];
#     for e in fib.take(10) { println!("{}", e) }
# }
```

And, when we compile our finished `macro_rules!` macro...

```text
error[E0425]: cannot find value `a` in this scope
  --> src/main.rs:68:50
   |
68 |     let fib = recurrence![a[n]: u64 = 1, 1; ...; a[n-2] + a[n-1]];
   |                                                  ^ not found in this scope

error[E0425]: cannot find value `n` in this scope
  --> src/main.rs:68:52
   |
68 |     let fib = recurrence![a[n]: u64 = 1, 1; ...; a[n-2] + a[n-1]];
   |                                                    ^ not found in this scope

error[E0425]: cannot find value `a` in this scope
  --> src/main.rs:68:59
   |
68 |     let fib = recurrence![a[n]: u64 = 1, 1; ...; a[n-2] + a[n-1]];
   |                                                           ^ not found in this scope

error[E0425]: cannot find value `n` in this scope
  --> src/main.rs:68:61
   |
68 |     let fib = recurrence![a[n]: u64 = 1, 1; ...; a[n-2] + a[n-1]];
   |                                                             ^ not found in this scope
```

... wait, what?
That can't be right... let's check what the macro is expanding to.

```shell
$ rustc +nightly -Zunpretty=expanded recurrence.rs
```

The `-Zunpretty=expanded` argument tells `rustc` to perform macro expansion, then turn the resulting AST back into source code.
The output (after cleaning up some formatting) is shown below;
in particular, note the place in the code where `$recur` was substituted:

```rust,ignore
#![feature(no_std)]
#![no_std]
#[prelude_import]
use std::prelude::v1::*;
#[macro_use]
extern crate std as std;
fn main() {
    let fib = {
        use std::ops::Index;
        const MEM_SIZE: usize = 1 + 1;
        struct Recurrence {
            mem: [u64; MEM_SIZE],
            pos: usize,
        }
        struct IndexOffset<'a> {
            slice: &'a [u64; MEM_SIZE],
            offset: usize,
        }
        impl <'a> Index<usize> for IndexOffset<'a> {
            type Output = u64;
            #[inline(always)]
            fn index<'b>(&'b self, index: usize) -> &'b u64 {
                use std::num::Wrapping;
                let index = Wrapping(index);
                let offset = Wrapping(self.offset);
                let window = Wrapping(MEM_SIZE);
                let real_index = index - offset + window;
                &self.slice[real_index.0]
            }
        }
        impl Iterator for Recurrence {
            type Item = u64;
            #[inline]
            fn next(&mut self) -> Option<u64> {
                if self.pos < MEM_SIZE {
                    let next_val = self.mem[self.pos];
                    self.pos += 1;
                    Some(next_val)
                } else {
                    let next_val = {
                        let n = self.pos;
                        let a = IndexOffset{slice: &self.mem, offset: n,};
                        a[n - 1] + a[n - 2]
                    };
                    {
                        use std::mem::swap;
                        let mut swap_tmp = next_val;
                        {
                            let result =
                                match ::std::iter::IntoIterator::into_iter((0..MEM_SIZE).rev()) {
                                    mut iter => loop {
                                        match ::std::iter::Iterator::next(&mut iter) {
                                            ::std::option::Option::Some(i) => {
                                                swap(&mut swap_tmp, &mut self.mem[i]);
                                            }
                                            ::std::option::Option::None => break,
                                        }
                                    },
                                };
                            result
                        }
                    }
                    self.pos += 1;
                    Some(next_val)
                }
            }
        }
        Recurrence{mem: [0, 1], pos: 0,}
    };
    {
        let result =
            match ::std::iter::IntoIterator::into_iter(fib.take(10)) {
                mut iter => loop {
                    match ::std::iter::Iterator::next(&mut iter) {
                        ::std::option::Option::Some(e) => {
                            ::std::io::_print(::std::fmt::Arguments::new_v1(
                                {
                                    static __STATIC_FMTSTR: &'static [&'static str] = &["", "\n"];
                                    __STATIC_FMTSTR
                                },
                                &match (&e,) {
                                    (__arg0,) => [::std::fmt::ArgumentV1::new(__arg0, ::std::fmt::Display::fmt)],
                                }
                            ))
                        }
                        ::std::option::Option::None => break,
                    }
                },
            };
        result
    }
}
```

But that looks fine!
If we add a few missing `#![feature(...)]` attributes and feed it to a nightly build of `rustc`, it even compiles!  ... *what?!*

> **Aside**: You can't compile the above with a non-nightly build of `rustc`.
> This is because the expansion of the `println!` macro depends on internal compiler details which are *not* publicly stabilized.

### Being Hygienic

The issue here is that identifiers in Rust syntax extensions are *hygienic*.
That is, identifiers from two different contexts *cannot* collide.
To show the difference, let's take a simpler example.

```rust,ignore
macro_rules! using_a {
    ($e:expr) => {
        {
            let a = 42i;
            $e
        }
    }
}

let four = using_a!(a / 10);
# fn main() {}
```

This macro simply takes an expression, then wraps it in a block with a variable `a` defined.
We then use this as a round-about way of computing `4`.
There are actually *two* syntax contexts involved in this example, but they're invisible.
So, to help with this, let's give each context a different colour.
Let's start with the unexpanded code, where there is only a single context:

<pre class="rust rust-example-rendered"><span class="synctx-0"><span class="macro">macro_rules</span><span class="macro">!</span> <span class="ident">using_a</span> {&#xa;    (<span class="macro-nonterminal">$</span><span class="macro-nonterminal">e</span>:<span class="ident">expr</span>) <span class="op">=&gt;</span> {&#xa;        {&#xa;            <span class="kw">let</span> <span class="ident">a</span> <span class="op">=</span> <span class="number">42</span>;&#xa;            <span class="macro-nonterminal">$</span><span class="macro-nonterminal">e</span>&#xa;        }&#xa;    }&#xa;}&#xa;&#xa;<span class="kw">let</span> <span class="ident">four</span> <span class="op">=</span> <span class="macro">using_a</span><span class="macro">!</span>(<span class="ident">a</span> <span class="op">/</span> <span class="number">10</span>);</span></pre>

Now, let's expand the invocation.

<pre class="rust rust-example-rendered"><span class="synctx-0"><span class="kw">let</span> <span class="ident">four</span> <span class="op">=</span> </span><span class="synctx-1">{&#xa;    <span class="kw">let</span> <span class="ident">a</span> <span class="op">=</span> <span class="number">42</span>;&#xa;    </span><span class="synctx-0"><span class="ident">a</span> <span class="op">/</span> <span class="number">10</span></span><span class="synctx-1">&#xa;}</span><span class="synctx-0">;</span></pre>

As you can see, the <code><span class="synctx-1">a</span></code> that's defined by the macro invocation is in a different context to the <code><span class="synctx-0">a</span></code> we provided in our invocation.
As such, the compiler treats them as completely different identifiers, *even though they have the same lexical appearance*.

This is something to be *really* careful of when working on `macro_rules!` macros, syntax extensions in general even: they can produce ASTs which
will not compile, but which *will* compile if written out by hand, or dumped using `-Zunpretty=expanded`.

The solution to this is to capture the identifier *with the appropriate syntax context*.
To do that, we need to again adjust our macro syntax.
To continue with our simpler example:

<pre class="rust rust-example-rendered"><span class="synctx-0"><span class="macro">macro_rules</span><span class="macro">!</span> <span class="ident">using_a</span> {&#xa;    (<span class="macro-nonterminal">$</span><span class="macro-nonterminal">a</span>:<span class="ident">ident</span>, <span class="macro-nonterminal">$</span><span class="macro-nonterminal">e</span>:<span class="ident">expr</span>) <span class="op">=&gt;</span> {&#xa;        {&#xa;            <span class="kw">let</span> <span class="macro-nonterminal">$</span><span class="macro-nonterminal">a</span> <span class="op">=</span> <span class="number">42</span>;&#xa;            <span class="macro-nonterminal">$</span><span class="macro-nonterminal">e</span>&#xa;        }&#xa;    }&#xa;}&#xa;&#xa;<span class="kw">let</span> <span class="ident">four</span> <span class="op">=</span> <span class="macro">using_a</span><span class="macro">!</span>(<span class="ident">a</span>, <span class="ident">a</span> <span class="op">/</span> <span class="number">10</span>);</span></pre>

This now expands to:

<pre class="rust rust-example-rendered"><span class="synctx-0"><span class="kw">let</span> <span class="ident">four</span> <span class="op">=</span> </span><span class="synctx-1">{&#xa;    <span class="kw">let</span> </span><span class="synctx-0"><span class="ident">a</span></span><span class="synctx-1"> <span class="op">=</span> <span class="number">42</span>;&#xa;    </span><span class="synctx-0"><span class="ident">a</span> <span class="op">/</span> <span class="number">10</span></span><span class="synctx-1">&#xa;}</span><span class="synctx-0">;</span></pre>

Now, the contexts match, and the code will compile.
We can make this adjustment to our `recurrence!` macro by explicitly capturing `a` and `n`.
After making the necessary changes, we have:

```rust
macro_rules! count_exprs {
    () => (0);
    ($head:expr) => (1);
    ($head:expr, $($tail:expr),*) => (1 + count_exprs!($($tail),*));
}

macro_rules! recurrence {
    ( $seq:ident [ $ind:ident ]: $sty:ty = $($inits:expr),+ ; ... ; $recur:expr ) => {
//    ^~~~~~~~~~   ^~~~~~~~~~ changed
        {
            use std::ops::Index;

            const MEM_SIZE: usize = count_exprs!($($inits),+);

            struct Recurrence {
                mem: [$sty; MEM_SIZE],
                pos: usize,
            }

            struct IndexOffset<'a> {
                slice: &'a [$sty; MEM_SIZE],
                offset: usize,
            }

            impl<'a> Index<usize> for IndexOffset<'a> {
                type Output = $sty;

                #[inline(always)]
                fn index<'b>(&'b self, index: usize) -> &'b $sty {
                    use std::num::Wrapping;

                    let index = Wrapping(index);
                    let offset = Wrapping(self.offset);
                    let window = Wrapping(MEM_SIZE);

                    let real_index = index - offset + window;
                    &self.slice[real_index.0]
                }
            }

            impl Iterator for Recurrence {
                type Item = $sty;

                #[inline]
                fn next(&mut self) -> Option<$sty> {
                    if self.pos < MEM_SIZE {
                        let next_val = self.mem[self.pos];
                        self.pos += 1;
                        Some(next_val)
                    } else {
                        let next_val = {
                            let $ind = self.pos;
//                              ^~~~ changed
                            let $seq = IndexOffset { slice: &self.mem, offset: $ind };
//                              ^~~~ changed
                            $recur
                        };

                        {
                            use std::mem::swap;

                            let mut swap_tmp = next_val;
                            for i in (0..MEM_SIZE).rev() {
                                swap(&mut swap_tmp, &mut self.mem[i]);
                            }
                        }

                        self.pos += 1;
                        Some(next_val)
                    }
                }
            }

            Recurrence { mem: [$($inits),+], pos: 0 }
        }
    };
}

fn main() {
    let fib = recurrence![a[n]: u64 = 0, 1; ...; a[n-2] + a[n-1]];

    for e in fib.take(10) { println!("{}", e) }
}
```

And it compiles!
Now, let's try with a different sequence.

```rust
# macro_rules! count_exprs {
#     () => (0);
#     ($head:expr) => (1);
#     ($head:expr, $($tail:expr),*) => (1 + count_exprs!($($tail),*));
# }
#
# macro_rules! recurrence {
#     ( $seq:ident [ $ind:ident ]: $sty:ty = $($inits:expr),+ ; ... ; $recur:expr ) => {
#         {
#             use std::ops::Index;
#
#             const MEM_SIZE: usize = count_exprs!($($inits),+);
#
#             struct Recurrence {
#                 mem: [$sty; MEM_SIZE],
#                 pos: usize,
#             }
#
#             struct IndexOffset<'a> {
#                 slice: &'a [$sty; MEM_SIZE],
#                 offset: usize,
#             }
#
#             impl<'a> Index<usize> for IndexOffset<'a> {
#                 type Output = $sty;
#
#                 #[inline(always)]
#                 fn index<'b>(&'b self, index: usize) -> &'b $sty {
#                     use std::num::Wrapping;
#
#                     let index = Wrapping(index);
#                     let offset = Wrapping(self.offset);
#                     let window = Wrapping(MEM_SIZE);
#
#                     let real_index = index - offset + window;
#                     &self.slice[real_index.0]
#                 }
#             }
#
#             impl Iterator for Recurrence {
#                 type Item = $sty;
#
#                 #[inline]
#                 fn next(&mut self) -> Option<$sty> {
#                     if self.pos < MEM_SIZE {
#                         let next_val = self.mem[self.pos];
#                         self.pos += 1;
#                         Some(next_val)
#                     } else {
#                         let next_val = {
#                             let $ind = self.pos;
#                             let $seq = IndexOffset { slice: &self.mem, offset: $ind };
#                             $recur
#                         };
#
#                         {
#                             use std::mem::swap;
#
#                             let mut swap_tmp = next_val;
#                             for i in (0..MEM_SIZE).rev() {
#                                 swap(&mut swap_tmp, &mut self.mem[i]);
#                             }
#                         }
#
#                         self.pos += 1;
#                         Some(next_val)
#                     }
#                 }
#             }
#
#             Recurrence { mem: [$($inits),+], pos: 0 }
#         }
#     };
# }
#
# fn main() {
for e in recurrence!(f[i]: f64 = 1.0; ...; f[i-1] * i as f64).take(10) {
    println!("{}", e)
}
# }
```

Which gives us:

```text
1
1
2
6
24
120
720
5040
40320
362880
```

Success!

[`macro_rules!`]: ./macros-methodical.md
