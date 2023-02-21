<!--
# Declarative Macros
-->
# 宣言的マクロ(Declarative Macros)

<!--
This chapter will introduce Rust's declarative macro system: [`macro_rules!`][mbe].
-->
本章ではRustの宣言的マクロシステム: [`macro_rules!`][mbe]について説明します。

<!--
There are two different introductions in this chapter, a [methodical] and a [practical].
-->
本章では[体系的説明]と[実践的説明]の2つを行います。

<!--
The former will attempt to give you a complete and thorough explanation of *how* the system works, while the latter one will cover more practical examples.
As such, the [methodical introduction][methodical] is intended for people who just want the system as a whole explained, while the [practical introduction][practical] guides one through the implementation of a single macro.
-->
前者はこのマクロシステムがどのように動作するのかについて完全かつ徹底的に説明する試みです。一方、後者ではより実践的な例を取り上げます。
[体系的説明]はこのシステムに関する全ての説明を求める読者向けとなっている一方で、[実践的説明]はひとつのマクロを実装する体験を通して読者を導く構成となっています。

<!--
Following up the two introductions it offers some generally very useful [patterns] and [building blocks] for creating feature-rich macros.
-->
2種類の説明の補足として、多機能なマクロを実装する際に一般的に非常に役立ついくつかの[パターン]と[構成要素] (building blocks) を紹介します。

<!--
Other resources about declarative macros include the [Macros chapter of the Rust Book] which is a more approachable, high-level explanation as well as the reference [chapter](https://doc.rust-lang.org/reference/macros-by-example.html) which goes more into the precise details of things.
-->
宣言的マクロに関する他の資料としては、より敷居が低く高水準な視点からの説明である[The Rust Book のマクロの章][^rust-book-ja]や、より厳密な詳細に深入りするリファレンス[^reference]の[マクロの章](https://doc.rust-lang.org/reference/macros-by-example.html) が挙げられます。

<!--
> **Note**: This book will usually use the term *mbe*(**M**acro-**B**y-**E**xample), *mbe macro* or `macro_rules!` macro when talking about `macro_rules!` macros.
-->
> **Note**: 本書では、`macro-rules!`によって定義されるマクロを指して *MBE*(**M**acro-**B**y-**E**xample), *MBEマクロ* あるいは `macro-rules!`マクロという用語を用います。

[mbe]: https://doc.rust-lang.org/reference/macros-by-example.html
<!--
[Macros chapter of the Rust Book]: https://doc.rust-lang.org/book/ch19-06-macros.html
-->
[The Rust Book のマクロの章]: https://doc.rust-lang.org/book/ch19-06-macros.html
<!--
[practical]: ./decl-macros/macros-practical.md
-->
[実践的説明]: ./decl-macros/macros-practical.md
<!--
[methodical]: ./decl-macros/macros-methodical.md
-->
[体系的説明]: ./decl-macros/macros-methodical.md
<!--
[patterns]: ./decl-macros/patterns.md
-->
[パターン]: ./decl-macros/patterns.md
<!--
[building blocks]: ./decl-macros/building-blocks.md
-->
[構成要素]: ./decl-macros/building-blocks.md

[^rust-book-ja]: *訳注*: [日本語版はこちら](https://doc.rust-jp.rs/book-ja/ch19-06-macros.html)

[^reference]: *訳注*: [The Rust Reference](https://doc.rust-lang.org/reference/)のこと
