---
title: Safety in an unsafe world
tags:
  - link
  - blog-article
  - topic-rust
  - topic-software-development
  - topic-software-verification
  - topic-type-systems
date: 2026-07-31
---
**Link: [Safety in an `unsafe { world }`](https://joshlf.com/posts/safety-unsafe-world/)**

On its own, Rust guarantees memory safety and thread safety. However, we can take safety to the next level using three steps.

1. **Define** an object that the Rust type system can reason about, such as a struct. Also define a safety property that Rust _can't_ reason about and attach it to the object through an invariant.
2. **Enforce** that the safety property is upheld. This responsibility is on us as the programmer.
3. **Consume** the safety property as a precondition. This is where we benefit most from our efforts.

Using these steps, we can ensure safety for any property we'd like.

## Further Reading
- [Newtypes and Contracts](https://research.texttotypes.com/newtypes-and-contracts/)
