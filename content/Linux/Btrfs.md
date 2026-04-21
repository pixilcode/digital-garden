---
title: Btrfs
tags:
  - reference
  - topic-linux
date: 2026-04-16
---
See [the Arch Linux wiki](https://wiki.archlinux.org/title/Btrfs)

## Creating the file system on a single device

```bash
sudo mkfs.btrfs -L <mylabel> /dev/<partition>
```

Increasing the node size decreases fragmentation but it increases operation times.

```bash
sudo mkfs.btrfs -L <mylabel> -n 32k /dev/<partition>
```


## Enabling compression

See [here](https://wiki.archlinux.org/title/Btrfs#Enabling_compression)

### For all files going forward

```bash
btrfs property set <fs-root> compression <alg>
```

`<alg>` can be
- `zlib`
- `lzo`
- `zstd`
- `no` (for no compression)

### For existing files

See [here](https://wiki.archlinux.org/title/Btrfs#For_existing_files).

## Encrypting Btrfs

Btrfs doesn't support encryption itself, but it can be placed on a hard drive with encryption. See [the Arch Linux wiki](https://wiki.archlinux.org/title/Btrfs#Encryption) and [[Hard Drive Encryption]] for more details.