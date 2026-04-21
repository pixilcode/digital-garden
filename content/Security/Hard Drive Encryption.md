---
title: Hard Drive Encryption
tags:
  - reference
  - topic-linux
  - topic-security
date: 2026-04-16
---
See [dm-crypt/Device encryption](https://wiki.archlinux.org/title/Dm-crypt/Device_encryption)

## List devices on computer

```bash
sudo fdisk -l
# or
lsblk
```

## Benchmarking different encryption algorithms

Since changing an encryption cipher of a block device after setup is difficult, it is important to check _dm-crypt_ performance for the individual parameters in advance:

```bash
cryptsetup benchmark
```

can give guidance on deciding for an algorithm and key-size prior to installation. If certain AES ciphers excel with a considerable higher throughput, these are probably the ones with hardware support in the CPU.

Quoted from [here](https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Cryptsetup_usage)
## Encrypting a device with LUKS

See [here](https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Encrypting_devices_with_LUKS_mode)

```bash
# format a LUKS partition, this will prompt you for a password
sudo cryptsetup luksFormat <device>

# check the results
sudo cryptsetup luksDump <device>
```

## Accessing a LUKS-encrypted device

See [here](https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Unlocking/Mapping_LUKS_partitions_with_the_device_mapper)

```bash
# this creates a device at `/dev/mapper/<device_mapped_name>`
sudo cryptsetup open <device> <device_mapped_name>
# example
sudo cryptsetup open /dev/sda1 backup
```

All access must be through `/dev/mapper/backup` for the encryption to work. This includes creating a filesystem such as [[Btrfs]].

```bash
sudo mkfs.btrfs /dev/mapper/backup
```

`/dev/mapper/backup` can be mounted like any regular partition.

To close the LUKS encryption, **unmount the partition**, then run

```bash
sudo cryptsetup close backup
```