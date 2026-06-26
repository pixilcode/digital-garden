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

## Implementation questions

### What is the output of running &inator?

### How do I run &inator?

### What libraries does &inator depend on?

- LLVM 17 (`llvm-17` and `llvm-17-dev`) - LLVM development libraries
- Clang 17 (`clang-17` and `libclang-17-dev`) - clang compiler and dev libraries
- `pkgconfig` - helps for finding libraries for building code
- `libpolly-17-dev` - high level optimizer used by LLVM
- `libz3-dev` - libraries for the Z3 theorem prover
- `zlib1g-dev` - library for zlib data compression
- `libzstd-dev` - library for zstandard data compression
- `libssl-dev` - library for TLS

### Why does &inator use two data compression libraries?

### Why does &inator need TLS? Does it access the network?

## Z3

- [Z3 Prover Paper](https://z3prover.github.io/papers/programmingz3.html)

Z3 is a Satisfiability Modulo Theories (SMT) solver.