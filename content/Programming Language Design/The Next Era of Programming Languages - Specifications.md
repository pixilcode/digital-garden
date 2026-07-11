---
title: The Next Era of Programming Languages - Specifications
tags:
  - topic-prog-lang
  - blog-post-draft
  - topic-ai
  - topic-specifications
date: 2026-03-30
---
Code is just a precise way of specification (see meme)
- related [comic strip](https://www.commitstrip.com/en/2016/08/25/a-very-comprehensive-and-precise-spec/?)
- interesting related articles:
    - [related article](https://haskellforall.com/2026/03/a-sufficiently-detailed-spec-is-code)
    - [related article](https://codemanship.wordpress.com/2026/01/15/yeah-about-your-precise-specification/)
    - [related article](https://medium.com/@olmedo2408/code-is-not-dead-why-specifications-arent-the-full-story-40a654736856)


Specification-driven development

Specially-trained LLMs are the new compilers

Cross-compilation = compiling to different programming language targets

Different levels/kinds of specification
- plain English for throwaway applications (Python of specification languages)
- general purpose specifications
- domain-specific specifications
- tests (property, unit, integration)
- types
- post-/pre-req specs
- dependently typing

Different levels of specification give different evaluation guarantees
- `Add two and two` - no real guarantees
- `2 + 2` - guarantees that addition will be mathematically accurate
- Guarantees can be made for memory, security, resources, etc.

Specifications get us even further from the "how" and closer to the "what".

There will still be a need for engineers that delve deeper into the actual code, just like there are still times that we need to delve into the generated assembly.
- computer engineers
- security analysts
- penetration testers
- researchers
- verification for actual specifications

Grow in two ways:
- specification languages (higher level)
- stricter typing (more statically-checked specifications) Rust, dependent typing, Dafny

Higher level = we care less about the details of the implementation

LLMs also need to get better, this is a prediction of the future, not the present

Non-deterministic? Yes, but we gave up non-determinism with memory management years ago when we started using garbage collection. All we care about is that the memory _is_ collected, not _how_ it is collected. For the times when we care about how memory is managed, we drop down to a lower-level programming language.

Basically, other languages may not go away, but they will be less frequently used.

This is also an approach for _producing_, not for _learning_. Because we are not actively involved in the code, we don't learn about the edge-cases or the processes that the AI is using. We will need to make a decision about what we want to learn.

Also, maybe experiment by writing a prototype by hand, then get rid of it and write specifications that an AI can then implement.