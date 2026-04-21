---
title: Managing removable storage
tags:
  - reference
  - topic-linux
date: 2026-04-16
---
## Mounting/unmounting removable storage

```bash
sudo mount /dev/<device> /run/media/<USER>/<name>
```

Note that the second directory (`/run/media/...`) must exist _before_ mounting.

```bash
sudo umount /run/media/<USER>/<name>
```

## Working with file systems

- [[Btrfs]]
## LUKS encryption

- [[Hard Drive Encryption]]