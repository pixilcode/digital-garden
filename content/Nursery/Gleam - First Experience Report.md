---
title: Gleam - First Experience Report
tags:
  - prog-lang
  - pl-experience
  - gleam
date: 2026-03-03
---
**Project:** [`oncmp`](https://github.com/pixilcode/oncmp)

## Liked

- Simple
    - Not a lot of "magic" going on behind the scenes
    - Code does what it says it does
    - Pretty easy to reason about
    - Structural equals/debug printing is always implemented
        - this is _usually_ what I want, and when it isn't, I can just make my own equality/printing function
- Functional
    - Immutability enforced, including for data structures
    - ADTs are a must for me
- Labelled function parameters
    - Helps function chaining to read well
    - Makes it clear what an argument is for
- Function chaining with `|>`
    - It's nice how a structure can be piped through a series of transformations
    - It almost feels like specialized currying
- Very clean, familiar project structure (especially when coming from Rust)
    - Bonus: initializing even comes with a Github workflow out of the box!
- Dicts don't require any sort of "hashing" or "comparison" for the key type
    - A dict can use anything as the key
    - Dicts are easy to use because of this (in comparison to OCaml and even Rust to some extent)
    - Downside is I don't exactly know how key equivalence is calculated underneath or how performance is
- `use` expression
    - Really well designed
    - I like using `use <- bool.guard(when: ..., return: ...)` for early returns
    - `result.try` is helpful too
    - Clever way to introduce a monad-like tool, but also more general purpose

## Interesting
- Parentheses are `{}`
    - A bit odd at first, especially when using it for arithmetic
    - However, it makes multi-line expressions read more like blocks in other languages, which feels kind of elegant
    - I might pick this up for a language of my own
- An `if` statement is missing from the language
    - `case` on a `Boolean` works about as well

## Disliked
- No quick way to do `let assert` with a message for unwrapping `Result`s
    - Hoping for something similar to Rust's `Result::expect`
    - Could be a benefit, though, since it encourages the programmer to think about how they should be handling the error, rather than just skipping over it
- Standard library is lacking a bit
    - No builtin support for regex or parsing, which would have been very useful for me
    - No builtin file support, although this one makes sense if they want it to be compatible with both Erlang and Javascript
    - This can be a bit of a benefit, though, since it's easier to peruse and look for the tools that I need
- Importing all of the individual enum variants is somewhat annoying
    - I can't think of any situations where only importing some is helpful
    - However, it reflects an interesting separation between the "compile time" type (`use foo.{type Foo}`) and the "runtime" variants (`use foo.{Bar, Baz})
- I miss `let Ok(foo) = foo_result else { return x; }` from Rust
    - There isn't really a `use` equivalent that I could find, either
    - Current pattern is `use <- bool.guard(foo_result |> result.is_error(), return: x)` `let assert Ok(foo) = foo_result`
- Nested patterns would be nice, although not essential (`["foo" | "bar", ..rest]` as opposed to `["foo", ..rest] | ["bar", ..rest]`)
- It's difficult to figure out how to install a project as a CLI
    - Originally, I tried `gleam export erlang-shipment` and tried to install that, but I ran into some trouble
    - After a bit of time, I found the instructions to use `gleescript` at the end of the [_Writing Gleam_](https://gleam.run/writing-gleam/) page

## Overall Impression
- Simple language
- Easy to pickup quickly
- Not as "powerful" = more useful for the task that I needed it for
- It's missing a couple of things in the core language (especially regex) that I'd like
- Holds to functional programming principles well
- Note that I didn't take advantage of any of the features that set it apart, specifically running on the BEAM