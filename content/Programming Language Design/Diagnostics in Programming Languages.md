---
title: Diagnostics in Programming Languages
tags:
  - topic-prog-lang
  - topic-pl-architecture
  - exploration
date: 2026-03-31
---
**Focus question:** How do other programming languages manage diagnostics (errors, warnings, etc.)

## Gleam
Gleam's `Diagnostic` struct:
```rust
// TODO: split this into locationed diagnostics and locationless diagnostics
#[derive(Debug, Clone, PartialEq, Eq)]
pub struct Diagnostic {
    pub title: String,
    pub text: String,
    pub level: Level,
    pub location: Option<Location>,
    pub hint: Option<String>,
}
```
- `title` - used as the message
- `text` - can be used as a "help" message
- `level` - two levels: error and warning
- `location` - the location of the diagnostic
- `hint` - an optional hint for how to fix the diagnostic



### Other random notes
- Gleam uses `ecow::EcoString` so that each diagnostic can store a reference to the source
- Gleam uses `thiserror` for its errors
- Gleam uses ANSI codes in its diagnostic displays (using `colorterm`)