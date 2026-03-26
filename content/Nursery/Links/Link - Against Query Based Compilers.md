---
title: Link - Against Query Based Compilers
tags:
  - link
  - blog-article
  - prog-lang
  - query-compiler
date: 2026-03-04
---
**Link: [Against Query Based Compilers](https://matklad.github.io/2026/02/25/against-query-based-compilers.html)**

Query-based compilers are quite popular for languages these days. However, while query-based compiling is a useful tool, a language's design should encourage as few query dependencies as possible.

For example, parsing in Rust might have dependencies on other files being compiled since macros can change the resulting AST. These dependencies are not ideal.

Meanwhile, in Zig, a file can be parsed with zero dependencies, which means all files can be parsed in parallel, which is a helpful property.

## Further Reading
- [Three Architectures For Responsive IDE](https://rust-analyzer.github.io/blog/2020/07/20/three-architectures-for-responsive-ide.html)
- [Zig AstGen: AST => ZIR](https://mitchellh.com/zig/astgen)
- Links at the bottom of the article