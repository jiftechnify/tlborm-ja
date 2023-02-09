<!--
# Expansion
-->
# 展開

<!--
Expansion is a relatively simple affair.
At some point *after* the construction of the AST, but before the compiler begins constructing its semantic understanding of the program, it will expand all syntax extensions.
-->
マクロの展開(expansion)は比較的単純な作業です。
抽象構文木を構築*し終えてから*コンパイラがプログラムの意味を理解しようとし始めるまでの間のどこかで、コンパイラはすべての構文拡張を展開します。

<!--
This involves traversing the AST, locating syntax extension invocations and replacing them with their expansion.
-->
これは、抽象構文木を走査し、構文拡張の呼び出し箇所を見つけ、展開形で置き換えるという処理を伴います。

<!--
Once the compiler has run a syntax extension, it expects the result to be parsable as one of a limited set of syntax elements, based on context.
For example, if you invoke a syntax extension at module scope, the compiler will parse the result into an AST node that represents an item.
If you invoke a syntax extension in expression position, the compiler will parse the result into an expression AST node.
-->
コンパイラが構文拡張を実行する際、コンパイラは呼び出し結果がその文脈に合致する構文要素のいずれかとしてパースできることを期待します。
例えば、構文拡張をモジュールスコープで呼び出したならば、コンパイラは呼び出し結果をアイテムを表す抽象構文木のノードとしてパースすることになります。
構文拡張を式が来るべき位置で呼び出したならば、コンパイラは結果を式の抽象構文木ノードとしてパースします。

<!--
In fact, it can turn a syntax extension result into any of the following:
-->
実のところ、コンパイラは構文拡張の呼び出し結果を以下のいずれかに変換できます:

<!--
* an expression,
* a pattern,
* a type,
* zero or more items, or
* zero or more statements.
-->
* 式
* パターン
* 型
* 0個以上のアイテム
* 0個以上の文

<!--
In other words, *where* you can invoke a syntax extension determines what its result will be interpreted as.
-->
言いかえれば、構文拡張を*どこで*呼び出したかによって、その結果がどう解釈されるかが決まるということです。

<!--
The compiler will take this AST node and completely replace the syntax extension's invocation node with the output node.
*This is a structural operation*, not a textual one!
-->
コンパイラは、構文拡張を展開した結果の抽象構文木ノードで構文拡張の呼び出しに対応するノードをそっくり置き換えます。
*これは構造を考慮した操作であり*、テキスト上の操作ではありません!

<!--
For example, consider the following:
-->
例えば、以下のコードを考えてみましょう:

```rust,ignore
let eight = 2 * four!();
```

<!--
We can visualize this partial AST as follows:
-->
これに対応する部分抽象構文木を図解すると次のようになります:

```text
┌─────────────┐
│ Let         │
│ name: eight │   ┌─────────┐
│ init: ◌     │╶─╴│ BinOp   │
└─────────────┘   │ op: Mul │
                ┌╴│ lhs: ◌  │
     ┌────────┐ │ │ rhs: ◌  │╶┐ ┌────────────┐
     │ LitInt │╶┘ └─────────┘ └╴│ Macro      │
     │ val: 2 │                 │ name: four │
     └────────┘                 │ body: ()   │
                                └────────────┘
```

<!--
From context, `four!()` *must* expand to an expression (the initializer can *only* be an expression).
Thus, whatever the actual expansion is, it will be interpreted as a complete expression.
In this case, we will assume `four!` is defined such that it expands to the expression `1 + 3`.
As a result, expanding this invocation will result in the AST changing to:
-->
文脈より、`four!()`は式として展開*されなければなりません*(初期化子(initializer)[^initializer]には式*しか*来ないため)。
よって、実際の展開形が何であれ、それは完全な式として解釈されることになります。
この場合、`four!()`は`1 + 3`のような式に展開されるものとして定義されていると仮定できます。
結果として、この呼び出しを展開すると抽象構文木は次のように変化します:

```text
┌─────────────┐
│ Let         │
│ name: eight │   ┌─────────┐
│ init: ◌     │╶─╴│ BinOp   │
└─────────────┘   │ op: Mul │
                ┌╴│ lhs: ◌  │
     ┌────────┐ │ │ rhs: ◌  │╶┐ ┌─────────┐
     │ LitInt │╶┘ └─────────┘ └╴│ BinOp   │
     │ val: 2 │                 │ op: Add │
     └────────┘               ┌╴│ lhs: ◌  │
                   ┌────────┐ │ │ rhs: ◌  │╶┐ ┌────────┐
                   │ LitInt │╶┘ └─────────┘ └╴│ LitInt │
                   │ val: 1 │                 │ val: 3 │
                   └────────┘                 └────────┘
```

[^initializer]: *訳注*: 初期化子(initializer)とは、変数の初期化文の右辺のこと。

<!--
This can be written out like so:
-->
これは次のように書き下すことができます:

```rust,ignore
let eight = 2 * (1 + 3);
```
<!--
Note that we added parentheses *despite* them not being in the expansion.
Remember that the compiler always treats the expansion of a syntax extension as a complete AST node, **not** as a mere sequence of tokens.
To put it another way, even if you don't explicitly wrap a complex expression in parentheses, there is no way for the compiler to "misinterpret" the result, or change the order of evaluation.
-->
展開形に含まれていない*にもかかわらず*、括弧が付け足されていることに注意してください。
コンパイラは常に構文拡張の展開形を完全な抽象構文木のノードとして扱うのであって、ただのトークンの列として扱うのでは**ない**ことを思い出してください。
別の言い方をすれば、複雑な式を明示的に括弧で囲まなくても、コンパイラが展開の結果を「誤解」したり、評価の順序を入れ替えたりすることはないということです。

<!--
It is important to understand that syntax extension expansions are treated as AST nodes, as this design has two further implications:
-->
構文拡張の展開形が抽象構文木のノードとして扱われるということをよく理解しておきましょう。この設計はさらに2つの意味を持ちます:

<!--
* In addition to there being a limited number of invocation *positions*, syntax extension can *only* expand to the kind of AST node the parser *expects* at that position.
* As a consequence of the above, syntax extension  *absolutely cannot* expand to incomplete or syntactically invalid constructs.
-->
* 構文拡張は、呼び出し*位置*の制約に加えて、その位置においてパーサが期待する種類の抽象構文木ノードに*しか*展開できないという制約を受ける。
* 上記の制約の帰結として、構文拡張は不完全な、あるいは構文的に不正な構造には*決して*展開*されない*。

<!--Add
There is one further thing to note about expansion: what happens when a syntax extension expands to something that contains *another* syntax extension invocation.
For example, consider an alternative definition of `four!`; what happens if it expands to `1 + three!()`?
-->
構文拡張の展開について、さらにもう一つ注意すべきことがあります。ある構文拡張が別の構文拡張の呼び出しを含む何かに展開されたらどうなるのでしょうか。
例えば、`four!`の別定義を考えてみましょう。それが`1 + three!()`に展開されるとしたら、どうなるのでしょうか?

```rust,ignore
let x = four!();
```
<!--
Expands to:
-->
これは次のように展開されます:

```rust,ignore
let x = 1 + three!();
```

<!--
This is resolved by the compiler checking the result of expansions for additional syntax extension invocations, and expanding them.
Thus, a second expansion step turns the above into:
-->
これはコンパイラが追加の構文拡張呼び出しの展開結果を確認し、展開することで解決されます。
したがって、2段階めの展開ステップにより上記のコードは次のように変換されます:

```rust,ignore
let x = 1 + 3;
```

<!--
The takeaway here is that expansion happens in "passes";
as many as is needed to completely expand all invocations.
-->
ここから得られる結論は、構文拡張の展開はすべての呼び出しが完全に展開されるのに必要なだけの「パス」にわたって行われるということです。

<!--
Well, not *quite*.
In fact, the compiler imposes an upper limit on the number of such recursive passes it is willing to run before giving up.
This is known as the syntax extension recursion limit and defaults to 128.
If the 128th expansion contains a syntax extension invocation, the compiler will abort with an error indicating that the recursion limit was exceeded.
-->
いや、これには*語弊があります*。
実際には、コンパイラは断念するまでに実行を試みる再帰的パスの数に上限を設けています。
これは構文拡張の再帰制限(recursion limit)として知られており、デフォルト値は128となっています。
もし128回めの展開が構文拡張呼び出しを含んでいたら、コンパイラは再帰制限を超過したことを示すエラーとともに実行を中断します。

<!--
This limit can be raised using the `#![recursion_limit="…"]` [attribute][recursion_limit], though it *must* be done crate-wide.
Generally, it is recommended to try and keep syntax extension below this limit wherever possible as it may impact compilation times.
-->
この制限は`#![recursion_limit="…"]`[属性][recursion_limit]を用いて引き上げることができるものの、クレート単位でしか設定できません。
上限の引き上げはコンパイル時間に影響を与える可能性があるため、基本的にはできる限り構文拡張が再帰制限を超えないように努めることをおすすめします。

[recursion_limit]: https://doc.rust-lang.org/reference/attributes/limits.html#the-recursion_limit-attribute
