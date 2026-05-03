---
title: Archiving with tar
tags:
  - tips
  - tool
  - reference
  - topic-linux
date: 2026-05-02
---
`tar` is a command line utility that comes with most Linux installations. It's generally useful for zipping a bunch of files together, but I also use it specifically for backing up hard drives and individual directories. Here are some commands that I use frequently.

## Creating an archive from a directory

```bash
tar --create --gzip --file <path-to-archive> <directory>
# or for short
tar -czf <path-to-archive> <directory>
```

