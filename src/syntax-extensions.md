<!--
# Syntax Extensions
-->
# 構文拡張 (Syntax Extensions)

<!--
Before talking about Rust's different macro systems it is worthwhile to discuss the general mechanism they are built on: *syntax extensions*.
-->
Rustのさまざまなマクロシステムの話に入る前に、それらの基礎となるより全般的な機構: *構文拡張* (syntax extensions) について考察するのは有意義でしょう。

<!--
To do that, we must first discuss how Rust source is processed by the compiler, and the general mechanisms on which user-defined macros and proc-macros are built upon.
-->
そのためには、まずどのようにしてRustのソースコードがコンパイラ、ひいてはユーザ定義のマクロや手続き的マクロの基礎となる機構によって処理されるのかについて考えていかねばなりません。

<!--
> **Note**: This book will use the term *syntax extension* from now on when talking about all of rust's different macro kinds in general to reduce potential confusion with the upcoming [declarative macro 2.0](https://github.com/rust-lang/rust/issues/39412) proposal which uses the `macro` keyword.
-->
> **Note**: 本書では以後、*構文拡張* (syntax extension) という用語をすべての種類のRustマクロに通ずる一般論を語る際に用います。これは、今後導入されるであろう、`macro`キーワードを用いる[宣言的マクロ2.0](https://github.com/rust-lang/rust/issues/39412)の提案に伴って生じうる混乱を軽減するためです[^translation-note]。

[^translation-note]: *訳注*: 言いたいことがよく分からないが、おそらく「Rustのマクロ全般を指して "macro" と呼ぶことにすると、(宣言的マクロ2.0で導入される予定の)`macro`キーワードを用いたマクロ*のみ*を指すように見えてしまう。それを避けるために、Rustのマクロ全般を指す用語として "syntax extension" という言葉を導入するよ」ということだと思われる。
