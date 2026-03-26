---
title: Git Hooks
tags:
  - software-development
  - git
date: 2026-03-24
---
If you want to perform actions on commit, add scripts to `.git/hooks/`. Examples of scripts that can be added are already found in that directory.

For example, in Oneil, I added the following scripts to `.git/hooks/pre-push` and `.git/hooks/pre-commit`:

```bash
#!/bin/bash
# in .git/hooks/pre-push

cargo build --all-targets --all-features
cargo test --all-features
cargo clippy --all-targets --all-features
cargo fmt --check
```

```bash
#!/bin/bash
#  in .git/hooks/pre-commit
cargo fmt
```