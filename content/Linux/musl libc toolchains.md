---
title: musl libc toolchains
tags:
  - link
  - tool
  - topic-linux
date: 2026-08-31
---
If you want to build a `musl` Rust project, you will need to install a `musl` toolchain. This can be found here:

[musl.cc](https://musl.cc/)

You can also get a list of the available `musl` toolchains by running

```bash
curl -s musl.cc
```

Once you have selected the toolchain you want, you can get it by running

```bash
curl -O https://musl.cc/x86_64-linux-musl-cross.tgz
```