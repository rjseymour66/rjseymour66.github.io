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

