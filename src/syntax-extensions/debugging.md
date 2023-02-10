<!--
# Debugging
-->
# デバッグ

<!--
`rustc` provides a number of tools to debug general syntax extensions, as well as some more specific ones tailored towards declarative and procedural macros respectively.
-->
`rustc` は構文拡張全般をデバッグするためのツールをいくつか提供しています。さらに、宣言的マクロと手続き的マクロのそれぞれに合わせたより特化したツールも提供します。

<!--
Sometimes, it is what the extension *expands to* that proves problematic as you do not usually see the expanded code.
Fortunately `rustc` offers the ability to look at the expanded code via the unstable `-Zunpretty=expanded` argument.
Given the following code:
-->
普段は展開後のコードを見ることはないため、構文拡張の*展開結果*がよく分からなくなることがあります。
ありがたいことに、`rustc` のunstableな `-Zunpretty=expanded` 引数を使って展開後のコードを見ることができます。
次のようなコードがあるとします:

```rust,ignore
// Shorthand for initializing a `String`.
// `String` 初期化の略記法
macro_rules! S {
    ($e:expr) => {String::from($e)};
}

fn main() {
    let world = S!("World");
    println!("Hello, {}!", world);
}
```

<!--
compiled with the following command:
-->
これを次のコマンドでコンパイルすると:

```shell
rustc +nightly -Zunpretty=expanded hello.rs
```

<!--
produces the following output (modified for formatting):
-->
次のような結果が得られます(結果を整形しています):

```rust,ignore
#![feature(prelude_import)]
#[prelude_import]
use std::prelude::rust_2018::*;
#[macro_use]
extern crate std;
// Shorthand for initializing a `String`.
// `String` 初期化の略記法
macro_rules! S { ($e : expr) => { String :: from($e) } ; }

fn main() {
    let world = String::from("World");
    {
        ::std::io::_print(
            ::core::fmt::Arguments::new_v1(
                &["Hello, ", "!\n"],
                &match (&world,) {
                    (arg0,) => [
                        ::core::fmt::ArgumentV1::new(arg0, ::core::fmt::Display::fmt)
                    ],
                }
            )
        );
    };
}
```
<!--
But not just `rustc` exposes means to aid in debugging syntax extensions.
For the aforementioned `-Zunpretty=expanded` option, there exists a nice `cargo` plugin called [`cargo-expand`](https://github.com/dtolnay/cargo-expand) made by [`dtolnay`](https://github.com/dtolnay) which is basically just a wrapper around it.
-->
構文拡張のデバッグを支援する手段を提供しているのは `rustc` だけではありません。
[dtolnay氏](https://github.com/dtolnay)が[`cargo-expand`](https://github.com/dtolnay/cargo-expand)という名前の素晴らしい `cargo` プラグインを制作しています。これは基本的には前述した `-Zunpretty=expanded` オプションの単なるラッパーです。

<!--
You can also use the [playground](https://play.rust-lang.org/), clicking on its `TOOLS` button in the top right gives you the option to expand syntax extensions as well!
-->
[Playground](https://play.rust-lang.org/)を利用することもできます。右上にある `TOOLS` ボタンをクリックすると、構文拡張を展開するオプションが選択できます。
