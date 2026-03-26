---
title: Link - How to print floating-point numbers
tags:
  - link
  - blog-article
  - algorithm
  - reference
date: 2026-03-24
---
**Link: [How to print floating-point numbers](https://michael-koller-91.github.io/blog/2023-03-26-how-to-print-floating-point-numbers.html)**

A description of how _Dragon 2_ (the floating point printing algorithm that Python uses) works.

It can be summarized as follows:
1. find $x$ such that $v' = v \cdot 10^{-x} < 2^p$ holds
2. print the integer part $k$ of $v'$
3. print a decimal point
4. compute the remaining precision $n = p - \text{ceil}(\log_2(k))$
5. print the fractional part with a precision of $n$ (this prints the integer $F$)
6. print $\cdot 10$
7. print $x$ as an exponent