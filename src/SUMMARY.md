# Summary

[はじめに](./introduction.md)

- [構文拡張](./syntax-extensions.md)
    - [ソースコード解析](./syntax-extensions/source-analysis.md)
    - [ASTにおけるマクロ](./syntax-extensions/ast.md)
    - [展開](./syntax-extensions/expansion.md)
    - [衛生性](./syntax-extensions/hygiene.md)
    - [デバッグ](./syntax-extensions/debugging.md)
- [宣言的マクロ](./decl-macros.md)
    - [体系的説明](./decl-macros/macros-methodical.md)
    - [実践的説明](./decl-macros/macros-practical.md)
    - [枝葉末節](./decl-macros/minutiae.md)
        - [フラグメント指定子](./decl-macros/minutiae/fragment-specifiers.md)
        - [Metavariables and Expansion Redux](./decl-macros/minutiae/metavar-and-expansion.md)
        - [Metavariable Expressions](./decl-macros/minutiae/metavar-expr.md)
        - [Hygiene](./decl-macros/minutiae/hygiene.md)
        - [Non-Identifier Identifiers](./decl-macros/minutiae/identifiers.md)
        - [Debugging](./decl-macros/minutiae/debugging.md)
        - [Scoping](./decl-macros/minutiae/scoping.md)
        - [Import and Export](./decl-macros/minutiae/import-export.md)
    - [パターン](./decl-macros/patterns.md)
        - [Callbacks](./decl-macros/patterns/callbacks.md)
        - [Incremental TT Munchers](./decl-macros/patterns/tt-muncher.md)
        - [Internal Rules](./decl-macros/patterns/internal-rules.md)
        - [Push-down Accumulation](./decl-macros/patterns/push-down-acc.md)
        - [Repetition Replacement](./decl-macros/patterns/repetition-replacement.md)
        - [TT Bundling](./decl-macros/patterns/tt-bundling.md)
    - [構成要素](./decl-macros/building-blocks.md)
        - [AST Coercion](./decl-macros/building-blocks/ast-coercion.md)
        - [Counting](./decl-macros/building-blocks/counting.md)
            - [Abacus Counting](./decl-macros/building-blocks/abacus-counting.md)
        - [Parsing Rust](./decl-macros/building-blocks/parsing.md)
    - [マクロ2.0](./decl-macros/macros2.md)
 - [Procedural Macros](./proc-macros.md)
    - [A Methodical Introduction](./proc-macros/methodical.md)
        - [Function-like](./proc-macros/methodical/function-like.md)
        - [Attribute](./proc-macros/methodical/attr.md)
        - [Derive](./proc-macros/methodical/derive.md)
    - [A Practical Introduction]()<!-- ./proc-macros/practical.md -->
        - [Function-like]()<!-- ./proc-macros/practical/function-like.md -->
        - [Attribute]()<!-- ./proc-macros/practical/attr.md -->
        - [Derive]()<!-- ./proc-macros/practical/derive.md -->
    - [Third-Party Crates](./proc-macros/third-party-crates.md)
    - [Hygiene and Spans](./proc-macros/hygiene.md)
    - [Techniques]()<!-- ./proc-macros/techniques.md -->
        - [Testing]()<!-- ./proc-macros/techniques/testing.md -->

 [用語集](./glossary.md)
