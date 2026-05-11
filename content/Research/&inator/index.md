---
title: "&inator: Correct, Precise C-to-Rust Interface Translation"
tags:
  - research
  - topic-rust
  - topic-software-development
  - paper-review
date: 2026-05-07
---
[**Link to paper**](https://arxiv.org/abs/2604.17261)

[**Link to implementation**](https://github.com/PLaSSticity/refinator-impl)

_&inator_ (pronounced _ref-inator_) is a tool for performing _interface translation_ from C to Rust.

- **interface translation**: assigning Rust types to top-level C declarations such as structs, constants, function parameters, and function return types

- Why is interface translation useful? Getting the interface types right makes it a lot easier to get the rest of the translation right.

