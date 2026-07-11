---
title: Link - Demystifying monads in Rust through property-based testing
tags:
  - link
  - blog-article
  - topic-software-development
  - topic-rust
  - topic-testing
date: 2026-04-07
---
**Link: [Demystifying monads in Rust through property-based testing](https://sunshowers.io/posts/monads-through-pbt/)**

A description of how monads work in software development in general (rather than just in functional programming). Also a great introduction to property-based testing and the `proptest` library in Rust.

Defining

## Insightful quotes

### Property testing

> Nothing quite exemplifies testing-as-modeling like property-based testing—an approach where instead of specifying exact examples, you define models in terms of **properties**, or invariants, that your code should satisfy. Then, you test your models against randomly generated inputs.

> To recap—property-based testing consists of two components:
>
> - Test case **generation** using a source of randomness.
> - On failing a test, **shrinking** it down to a smaller, more understandable size.

### Monads and using powerful tools

> The common thread through all of these examples is that **within a framework, monadic composition is not just from value to _value_. It is from value to a further instance of that _framework_.**

> This is part of a general observation throughout programming, whenever there’s an interaction between two parties or two sides. **The more restrictions there are on one side, the freer the other side is to do what it likes.**

> Whether you’re a user or a library designer, **pay close attention to situations where your operations are monadic.** They can provide a great deal of power, perhaps too much in some circumstances. **If non-monadic operations are sufficient to help you achieve your goal, prefer them.**

