---
title: "Kernel"
linkTitle: "Kernel"
# weight: 1000
# description:
---


## Find kernel version

```bash
uname -r
6.8.0-49-generic
```

## Kernel modules

A _loadable kernel module_ (LKM) is software that tells the kernel where to find a device and what to do with it.

The modules available for each kernel release are in `/lib/modules`:

```bash
ls -l /lib/modules
total 8
drwxr-xr-x. 5 root root 4096 Nov  2 17:14 6.8.0-48-generic
drwxr-xr-x. 5 root root 4096 Nov 20 00:51 6.8.0-49-generic

# get available modules for your kernel version
ls -l /lib/modules/$(uname -r)
total 7168
lrwxrwxrwx.  1 root root      39 Nov  1 10:56 build -> /usr/src/linux-headers-6.8.0-49-generic
drwxr-xr-x.  2 root root    4096 Nov  1 10:56 initrd
drwxr-xr-x. 17 root root    4096 Nov 20 00:50 kernel
-rw-r--r--.  1 root root 1663327 Nov 20 00:51 modules.alias
-rw-r--r--.  1 root root 1620199 Nov 20 00:51 modules.alias.bin
-rw-r--r--.  1 root root    9714 Nov  1 10:56 modules.builtin
-rw-r--r--.  1 root root   10690 Nov 20 00:51 modules.builtin.alias.bin
-rw-r--r--.  1 root root   11907 Nov 20 00:51 modules.builtin.bin
-rw-r--r--.  1 root root   87700 Nov  1 10:56 modules.builtin.modinfo
-rw-r--r--.  1 root root  863202 Nov 20 00:51 modules.dep
-rw-r--r--.  1 root root 1144982 Nov 20 00:51 modules.dep.bin
-rw-r--r--.  1 root root     353 Nov 20 00:51 modules.devname
-rw-r--r--.  1 root root  262876 Nov  1 10:56 modules.order
-rw-r--r--.  1 root root    2755 Nov 20 00:51 modules.softdep
-rw-r--r--.  1 root root  731722 Nov 20 00:51 modules.symbols
-rw-r--r--.  1 root root  889131 Nov 20 00:51 modules.symbols.bin
drwxr-xr-x.  3 root root    4096 Nov 20 00:50 vdso
```

### lsmod

Get loaded kernel modules:

```bash
lsmod
Module                  Size  Used by
sch_netem              24576  0
ib_core               507904  0
tls                   155648  0
...
```

### modprobe

Load kernel modules, and get all available kernel modules:

```bash
# get total count
modprobe -c | wc -l
51317

# load a module
modprobe <module-name>
```

### rmmod

Remove a kernel module:

```bash
rmmod <module-name>
```