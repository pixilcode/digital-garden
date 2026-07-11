---
title: What if writing tests was a joyful experience?
tags:
  - link
  - blog-article
  - inspiration
  - topic-ocaml
  - topic-software-development
  - topic-testing
  - topic-gleam
date: 2026-07-09
---
**Link: [What if writing tests was a joyful experience?](https://blog.janestreet.com/the-joy-of-expect-tests/)**

Often, when testing code, you write a test, then test the output structure of that code. This is the essence of a unit test. However, sometimes it can be hard to read those tests to figure out what they do. One way to help with this is to write snapshot tests (referred to in this article as "expect tests"). This can improve the readability of a test and make it much more useful.

**Link: [Testing can be fun, actually](https://giacomocavalieri.me/writing/testing-can-be-fun-actually)**

Writing unit tests can be tedious. For example, having to update a string every time the output changes is annoying. This can be changed with snapshot testing. There are libraries, such as Gleam's [`birdie`](https://birdie.hexdocs.pm/index.html) that allow you to update snapshots easily and efficiently.

In addition, it can be helpful to see the results of a function in an easy-to-read string.

**Link: [Markdown is great for encoding test snapshots](https://jordaneldredge.com/markdown-snapshots/)**

Markdown can be a great way to do snapshot testing! It allows you to keep all of your outputs together in one file, and you get builtin syntax highlighting.