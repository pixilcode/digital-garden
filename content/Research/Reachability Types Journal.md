---
title: Reachability Types Journal
tags:
  - research
  - literature-review
  - topic-prog-lang
  - topic-type-systems
date: 2026-10-28
---
## 2026-10-28

> [!question] What are reachability types?
> Reachability types are a way to statically determine what resources are reachable from any given position in a program. They are similar to Rust's _shared XOR mutable_ model, but much more flexible.
> 
> > [!quote]
> > \[Reachability types\] enable **safe sharing** and **escape of mutable data** by **tracking which resources each expression's result can reach**.
> > 
> > ― [[Escape with Your Self - Sound and Expressive Bidirectional Typing with Avoidance for Reachability Types]]

> [!question] What does the ♦ symbol mean in reachability types?
> They indicate a _fresh_ value that is _unreachable_ from existing variables

### Discovered resources
- [Modeling Reachability Types with Logical Relations](https://dl.acm.org/doi/epdf/10.1145/3763116)
- [[Reachability types - tracking aliasing and seperation in higher-order functional programs]]
- [Complete the Cycle: Reachability Types with Expressive Cyclic References](https://dl.acm.org/doi/epdf/10.1145/3763172)
- [Capture Now, Consume Later: Reachability Types with Flow-Sensitive Effects for Higher-Order Ownership Transfer](https://arxiv.org/pdf/2510.08939)
- [Graph IRs for Impure Higher-Order Languages](https://dl.acm.org/doi/epdf/10.1145/3622813)
- [The simple essence of algebraic subtyping: principal type inference with subtyping made easy (functional pearl)](https://dl.acm.org/doi/epdf/10.1145/3409006)