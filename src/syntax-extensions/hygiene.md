<!--
# Hygiene
-->
# 衛生性(Hygiene)

<!--
Hygiene is an important concept for macros.
It describes the ability for a macro to work in its own syntax context, not affecting nor being affected by its surroundings.
In other words this means that a syntax extension should be invocable anywhere without interfering with its surrounding context.
-->
衛生性(Hygiene)はマクロに関する重要な概念です。
衛生性とは、マクロが周辺のコンテキストに影響を与えたり逆に影響受けたりすることなく、自身の構文コンテキストの中で動作する能力を指します。
言いかえれば、任意の構文拡張の呼び出しは周辺のコンテキストに干渉するべきではないということを意味します。

<!--
In a perfect world all syntax extensions in Rust would be fully hygienic, unfortunately this isn't the case, so care should be taken to avoid writing syntax extensions that aren't fully hygienic.
We will go into general hygiene concepts here which will be touched upon in the corresponding hygiene chapters for the different syntax extensions Rust has to offer.
-->
理想的には、Rustのすべての構文拡張は完全に衛生的であってほしいところですが、残念ながらそうではないので、十分衛生的ではない構文拡張を書くことがないよう注意を払うべきです。
ここから一般的な衛生性の概念の説明に入ります。なお、これについてはRustが提供する別の種類の構文拡張の衛生性に関する章でも言及します。

&nbsp;

<!--
Hygiene mainly affects identifiers and paths emitted by syntax extensions.
In short, if an identifier created by a syntax extension cannot be accessed by the environment where the syntax extension has been invoked it is hygienic in regards to that identifier.
Likewise, if an identifier used in a syntax extension cannot reference something defined outside of a syntax extension it is considered hygienic.
-->
衛生性は主に構文拡張が出力する識別子とパス(path)に影響します。
簡単にいうと、構文拡張が生成した識別子を、構文拡張を呼び出した環境から参照できないとき、構文拡張はその識別子に関して衛生的であるといえます。
同様に、構文拡張の内部で利用されている識別子が、構文拡張の外部で定義されたものを参照できないときも、構文拡張は衛生的であるといえます。

<!--
> **Note**: The terms `create` and `use` refer to the position the identifier is in.
> That is the `Foo` in `struct Foo {}` or the `foo` in `let foo = …;` are created in the sense that they introduce something new under the name,
> but the `Foo` in `fn foo(_: Foo) {}` or the `foo` in `foo + 3` are usages in the sense that they are referring to something existing.
-->
> **Note**: 「生成」「利用」という用語は、識別子がどの位置に出現しているかを表すものです。
> `struct Foo {}` における `Foo` や `let foo = …;` における `foo` は、その名前のもとで新しい何かを導入したという意味で「生成」されたといえます。
> 一方で `fn foo(_: Foo) {}` における `Foo` や `foo + 3` における `foo` は、既存の何かを参照するという意味で「利用」だといえます。

<!--
This is best shown by example.
-->
例を挙げて説明するのが一番でしょう。

<!--
Let's assume we have some syntax extension `make_local` that expands to `let local = 0;`, that is it *creates* the identifier `local`.
Then given the following snippet:
-->
`make_local` という構文拡張があって、`let local = 0;` に展開されるとしましょう。これは `local` という識別子を*生成*します。
次のようなコード片を考えます:


```rust,ignore
make_local!();
assert_eq!(local, 0);
```

<!--
If the `local` in `assert_eq!(local, 0);` resolves to the local defined by the syntax extension, the syntax extension is not hygienic (at least in regards to local names/bindings).
-->
もし `assert_eq!(local, 0);` における `local` が、この構文拡張が定義したローカル変数に解決されるならば、この構文拡張は(少なくともローカルな名前・束縛(bindings)に関していえば)衛生的ではないということになります。

<!--
Now let's assume we have some syntax extension `use_local` that expands to `local = 42;`, that is it makes *use* of the identifier `local`.
Then given the following snippet:
-->
今度は `use_local` という構文拡張があって、`local = 42;` に展開されるとしましょう。これは `local`という識別子を*利用*します。
次のようなコード片を考えます:

```rust,ignore
let mut local = 0;
use_local!();
```

<!--
If the `local` inside of the syntax extension for the given invocation resolves to the local defined before its invocation, the syntax extension is not hygienic either.
-->
もし呼び出された構文拡張の内部の `local` が、その呼び出しの前に定義されたローカル変数に解決されるのであれば、この構文拡張はやはり衛生的ではないということになります。

<!--
This is a rather short introduction to the general concept of hygiene.
It will be explained in more depth in the corresponding [`macro_rules!` `hygiene`] and [proc-macro `hygiene`] chapters, with their specific peculiarities.
-->
以上は衛生性の一般概念への比較的短い導入です。
衛生性については、 [`macro_rules!` の衛生性]と[手続き的マクロの衛生性]の章で、各々に固有の性質に言及しながら、より深いところまで説明します。

<!--
[`macro_rules!` `hygiene`]: ../decl-macros/minutiae/hygiene.md
-->
[`macro_rules!` の衛生性]: ../decl-macros/minutiae/hygiene.md
<!--
[proc-macro `hygiene`]: ../proc-macros/hygiene.md
-->
[手続き的マクロの衛生性]: ../proc-macros/hygiene.md
