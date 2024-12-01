---
title: "Memory"
linkTitle: "Memory"
# weight: 1000
# description:
---

## free

Parses the `/proc/meminfo` file and shows total available memory:

```bash
# shared - tmpfs to maintain pseudo fs like /sys and /dev
# buff/cache - kernel block I/O 
free -h
               total        used        free      shared  buff/cache   available
Mem:           7.8Gi       631Mi       7.0Gi       3.8Mi       373Mi       7.1Gi
Swap:             0B          0B          0B

# with high and low stats
free -lm
               total        used        free      shared  buff/cache   available
Mem:           31704        6752       19689        1023        5263       23472
Low:           31704       12015       19689
High:              0           0           0
Swap:          65535           0       65535

```

## vmstat

Checks swap usage. You can run it over an extended period of time. This gives you 4 readings at 30-second intervals:

```bash
# si - swapped into memory
# so - swapped out of memory
vmstat 30 4
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 4996868 1537492 18966428    0    0    18    61   23    1  4  2 95  0  0
 0  0      0 5006424 1537504 18956112    0    0     0    61 6112 7490  4  4 92  0  0
```

## Swap

> From [All about linux swap space](https://www.linux.com/news/all-about-linux-swap-space/)
>
> Linux divides its physical RAM (random access memory) into chunks of memory called pages. Swapping is the process whereby a page of memory is copied to the preconfigured space on the hard disk, called swap space, to free up that page of memory. The combined sizes of the physical memory and the swap space is the amount of virtual memory available.

Virtual memory is RAM (physical memory) and swap space.
- Need swap space when there isn't enough available physical memory on your machine for all the active processes
- Inactive pages from physical memory are moved into swap to free up physical memory
- Swap space can be a partition, file, or combo of both
  - Swap files do not use contiguous disk blocks, so may have performance impact
- If you use swap, make it equal to your physical memory

```bash
# check for swap partition (no swap here)
sudo parted -l
Model: KXG60ZNV512G NVMe KIOXIA 512GB (nvme)
Disk /dev/nvme0n1: 512GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name                  Flags
 1      1049kB  829MB   828MB   fat32        EFI system partition  boot, esp
 2      829MB   6198MB  5369MB  fat32        Basic data partition  msftres
 3      6198MB  512GB   506GB   ext4


# see swap
cat /proc/swaps 
Filename				Type		Size		Used		Priority
/swapfile                               file		67108860	0		-2

# in fstab
cat /etc/fstab 
# /etc/fstab: static file system information.
...
/swapfile                                 none            swap    sw              0       0
```
### Swappiness

```bash
# 0 - 100. 0 is avoid swapping, 100 is aggressive swappiness
cat /proc/sys/vm/swappiness 
60
```

## OOM Killer