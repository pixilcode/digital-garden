---
title: Nim - Experience Report
tags:
  - experience-report
  - topic-pl-experience
date: 2025-11-13
---
**Projects:**
- [Interval Arithmetic Analysis](https://github.com/pixilcode/interval-arithmetic-analysis)
- [Nap: an automatic screen locker](https://github.com/pixilcode/nap)
# Positive

- Using `distinct` makes it really easy to create newtypes
- I can easily compile a simple script, no need to make a whole workspace
- On top of that, I could easily turn a script into a multi-file project
- I like that the first argument of a function can be passed with `.` syntax. It creates behavior similar to monkey-patching by just writing a function that accepts the "monkey-patched" type as the first argument.
    - `proc foo(bar: Bar, baz: Baz)` can be called as `bar.foo(baz)`
    - This also made function calls read nicer (ex. `interval.min.isNaN` versus `isNaN(min(interval)))`
- The macros available are powerful
    - I especially like the `collect` macro that allows for "list comprehension"-like behavior
- Nim templates are simplified macros that allow for me to define substitution macros without having to worry about the AST at all
    - Arguments to templates are checked, though, so we don't run into problems that we see with C macros, where they can produce invalid code if not used correctly
- Their "batteries included" library makes it really easy to find a lot of the tools that I need without having to figure out package management at all
- Tuples with field names were really handy!
    - I could use a tuple with field names in one or two spots if I didn't want to create a full object type for them. Then, if I found I was using it in a lot of different places, I could create a full-fledged object type.

# Negative

- The LSP support wasn't great
  - incorrect syntax highlighting
  - loaded too slowly
  - sometimes would get stuck
  - renaming
  - automatic imports
  - NOTE: I don't want to complain _too_ much, LSPs can be tough to implement
- I wasn't sure how function overloading worked
  - how do I use a "trait-like" value? (ex. I want an `Interval` trait that says that all intervals implement x, y, and z operations)
- The distinction between `proc` and `func` doesn't tend to be a useful, and it sometimes makes for more work when I need to add something that turns something from a `func` into a `proc`.
    - If a `func` is deeply nested in other `func`s, then changing it to a `proc` requires changing all of its parents to `proc`s, too
- There are too many ways to do something
  - different naming cases are valid and interchangeable
  - `echo "foo"` vs `echo("foo")` vs `"foo".echo` - can make sense, but no convention?
  - different ways of returning (`result = foo`, `return foo`, or just returning the last expression) can make it hard to identify what is being returned
- Implicit defaults made refactoring harder
    - If I add a field to an object, the compiler doesn't tell me where to update that object
- `case` statement can't be an expression
    - I like `match` expressions
- Exporting with `*` means that it can be hard to find exported values
    - It isn't specifically greppable either, since `*` can mean multiplication as well
- The language is almost _too_ powerful
    - Macros can make it hard to understand what is going on and can add extra syntax to learn
    - Function overloading can make it hard to find the definition of the function that you're calling - is it from module A or module B?
    - The amount of freedom that is allowed when writing code can make it hard to develop a mental model of code structure and organization


## Overall Impression

Nim was a cool language! It has a lot of cool features that I enjoyed using, including the `distinct` keyword and tuples with fields. It's approach to macros was very useful, and it's standard library had everything I needed to build a lot of the basic projects I was envisioning.

However, in some ways, Nim tried to be _too_ useful. It allowed for many different conventions and even converted between them when it came to naming. This forced me to spend time deciding what _my_ convention would be, rather than spending time working on the code.

Overall, I think I probably wouldn't use Nim for very many projects. It has a lot of amazing features, but because it allows too many different ways of doing things, it makes code feel unmanageable after it gets too big. It has some great design choices that I think other languages could learn from.