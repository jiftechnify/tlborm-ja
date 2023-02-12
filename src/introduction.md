# Rustマクロの薄い本

> **訳注**: 本書は[The Little Book of Rust Macros](https://github.com/veykril/tlborm)(Lukas Wirth氏による改訂版)の非公式日本語訳です🦀🇯🇵

<!--
> **Note**: This is a continuation of [Daniel Keep's Book](https://github.com/DanielKeep/tlborm) which has not been updated since the early summer of 2016, adapted to make use of [mdBook](https://github.com/rust-lang/mdBook).
-->

> **Note**: 翻訳元は[Daniel Keep氏による原著](https://github.com/DanielKeep/tlborm)(2016年の中頃より更新停止)を引き継いだもので、[mdBook](https://github.com/rust-lang/mdBook)を利用するように改変してあります。

<!--
View the [rendered version here](https://veykril.github.io/tlborm/) and the [repository here](https://github.com/veykril/tlborm).
-->

本書のHTML版は[こちら](https://jiftechnify.github.io/tlborm-ja/)から読めます。リポジトリは[こちら](https://github.com/jiftechnify/tlborm-ja)にあります。

<!--
A chinese version of this book can be found [here](https://zjp-cn.github.io/tlborm/).
-->

中国語訳は[こちら](https://zjp-cn.github.io/tlborm/)。

<!--
This book is an attempt to distill the Rust community's collective knowledge of Rust macros, the `Macros by Example` ones as well as procedural macros(WIP).
As such, both additions (in the form of pull requests) and requests (in the form of issues) are very much welcome.
If something's unclear, opens up questions or is not understandable as written down, fear not to make an issue asking for clarification.
The goal is for this book to become the best learning resource possible.
-->

本書の目標は、Rustのマクロ(宣言的マクロ(`Macros by Example`)と手続き的マクロ(WIP)の両方)に関するRustコミュニティの集合知を蒸留することです。
ですから、(プルリクエストという形での)内容の追加も(issueという形の)リクエストも大歓迎です。
もし不明な点があれば、気軽に質問してください。よくわからない記述があれば、わかりやすくしてほしい旨を恐れずissueにぶつけてください。
本書をできる限り最高の学習資料にするのが目標です。

<!-- The [original Little Book of Rust Macros](https://github.com/DanielKeep/tlborm) has helped me immensely with understanding ***Macros by Example*** style macros while I was still learning the language.
Unfortunately, the original book hasn't been updated since April of 2016, while the Rust language as well as its macro-system keeps evolving.
Which is why I took up the task to update the book and keep it updated as well as I can while also adding newfound things to it.
In hopes that it will help out all the fresh faces coming to Rust understanding its macro systems, a part of the language a people tend to have trouble with.
-->

この本の[原著](https://github.com/DanielKeep/tlborm)は、私がRust言語を学んでいた頃に***宣言的マクロ*** (Macros by Example)スタイルのマクロについて理解する大いなる助けになりました。
残念なことに、原著は2016年4月を最後に更新が途絶えてしまいました。その一方で、Rust言語自体もマクロシステムも進化し続けています。
そんなわけで私は、原著をアップデートし、最新に保ち、さらには新たな発見を書き加えるという仕事を引き受けたのです。
Rustに入門する方々がマクロシステムという手こずりがちなRust言語の一機能について理解を深めようとするにあたり、本書が助けになれば幸いです。

<!--
> This book expects you to have basic knowledge of Rust, it will not explain language features or constructs that are irrelevant to macros.
> No prior knowledge of macros is assumed.
> Having read and understood the first seven chapters of the [Rust Book](https://doc.rust-lang.org/stable/book/) is a must, though having read the majority of the book is recommended.
-->

> 本書は読者がRustの基本的な知識を持っていることを仮定して書かれています。マクロに関係ない言語機能や構成要素については説明しません。
> マクロに関する事前知識は不要です。
> [The Book](https://doc.rust-lang.org/stable/book/)[^the-book-ja] の7章までを読んで理解しておくのが必須事項です。とはいえ、大部分を読んでおくことをおすすめします。

[^the-book-ja]: *訳注*: [日本語版はこちら](https://doc.rust-jp.rs/book-ja/title-page.html)

<!--
## Thanks
-->

## 謝辞

<!--
A big thank you to Daniel Keep for the original work as well as all the contributors that added to the original which can be found [here](https://github.com/DanielKeep/tlborm).
-->

原著の著者であるDaniel Keep氏、および原著へのコントリビューターの皆様([こちら](https://github.com/DanielKeep/tlborm)で確認できる)に深く感謝申し上げます。

<!--
## License
-->

## ライセンス

<!--
This work inherits the licenses of the original, hence it is licensed under both the [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/) and the [MIT license](http://opensource.org/licenses/MIT).
-->
本書は原著のライセンスを継承します。よって、本書の使用は[Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/)と[MIT license](http://opensource.org/licenses/MIT)の両方の下で許諾されます[^license-ja]。

[^license-ja]: *訳注*: 日本語版についても同様の扱いとします。詳しくは本プロジェクトのリポジトリの[README](https://github.com/jiftechnify/tlborm-ja#readme)をご覧ください。
