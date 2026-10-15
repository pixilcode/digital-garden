---
title: Call-by-value is dual to call-by-name
tags:
  - paper-review
  - topic-prog-lang
date: 2026-10-07
---
**Paper: [Call-by-value is dual to call-by-name](https://dl.acm.org/doi/10.1145/944705.944723)**

This paper describes a calculus that corresponds to **Gentzen's sequent calculus**.

## Notes

### Variables and Covariables

This paper talks about _variables_ and _covariables_. Based on what I'm reading, I think these can be viewed from the following perspective: _variables_ are _value producers_, while _covariables_ are value consumers.

_Variables_ could be considered to have a thunk-like type `() -> a`, where the thunk is automatically evaluated when read from. _Covariables_ could be considered to have a sort of inverse type `a -> !`.

>[!note]
>I think it's `a -> !`, since that feels like the natural inverse of `() -> a`. It could also be `a -> ()`, though.

Given this representation, I think _covariables_ would essentially be continuations with one argument.