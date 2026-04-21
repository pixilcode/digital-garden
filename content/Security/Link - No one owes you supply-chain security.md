---
title: Link - No one owes you supply-chain security
tags:
  - link
  - blog-article
  - topic-rust
  - topic-security
  - topic-software-development
date: 2026-04-21
---
**Link: [No one owes you supply-chain security](https://purplesyringa.moe/blog/no-one-owes-you-supply-chain-security/)**

Supply-chain attacks are becoming more common in today's world, and many people are blaming [crates.io](https://crates.io) for problems with the supply chain. However, many of the proposed solutions are infeasible.

The best solution is that **everyone audits their own supply chain**. In other words, any time you choose to add a crate to your project, _you_ need to make sure that the project is not malicious.

It doesn't take a security expert to identify sketchy code in a build script or in the code itself. You just need to browse the source for a second and look for anything that seems off. Plus, Rust provides many tools like [`cargo vet`](https://mozilla.github.io/cargo-vet/) that allow you to check your dependencies for security.