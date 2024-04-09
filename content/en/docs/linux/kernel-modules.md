---
title: "Kernel modules"
weight: 130
description: >
  "Tending kernel modules"
---

A kernel module is a self-contained driver library file.
- modules keep the kernel lighter, as opposed to compiling them into the kernel
- load or unload them dynamically without needing to reboot
- module files use `.ko` extension

Types of kernel modules:
- Device driver: Communicates with hardware device
- Filesystem driver: Required for fs I/O
- Network driver: Implement network protocols
- System calls: Additional functions for adding/modifying system services
- Executable loader: Allows additional executable formats to load

## Module and config files

Stored in `/lib/modules/` directory:

```bash
# view kernel versions
ls -F /lib/modules 
6.5.0-18-generic/  6.5.0-26-generic/
# config files generated during install or by sys admin
ls -hogl /etc/modprobe.d/
total 40K
-rw-r--r-- 1 2.5K Feb 22  2021 alsa-base.conf
-rw-r--r-- 1  154 Aug 20  2023 amd64-microcode-blacklist.conf
-rw-r--r-- 1  325 Aug 17  2021 blacklist-ath_pci.conf
-rw-r--r-- 1 1.5K Aug 17  2021 blacklist.conf
-rw-r--r-- 1  210 Aug 17  2021 blacklist-firewire.conf
-rw-r--r-- 1  677 Aug 17  2021 blacklist-framebuffer.conf
-rw-r--r-- 1  156 Feb 22  2021 blacklist-modem.conf
lrwxrwxrwx 1   41 Mar 21 20:12 blacklist-oss.conf -> /lib/linux-sound-base/noOSS.modprobe.conf
-rw-r--r-- 1  583 Aug 17  2021 blacklist-rare-network.conf
-rw-r--r-- 1  154 Nov 14 20:03 intel-microcode-blacklist.conf
-rw-r--r-- 1  347 Aug 17  2021 iwlwifi.conf
# config files for 3rd party packages
ls -hogl /etc/modules-load.d/
total 4.0K
-rw-r--r-- 1 119 May 15  2023 cups-filters.conf
lrwxrwxrwx 1  10 Mar 21 20:12 modules.conf -> ../modules
# same as /lib/modprobe.d/
ls -hogl /usr/lib/modprobe.d/
total 20K
-rw-r--r-- 1  655 Aug 17  2021 aliases.conf
-rw-r--r-- 1 1.6K Feb  7 04:12 blacklist_linux-hwe-6.5_6.5.0-18-generic.conf
-rw-r--r-- 1 1.6K Mar 12 04:50 blacklist_linux-hwe-6.5_6.5.0-26-generic.conf
-rw-r--r-- 1  390 Sep 26  2023 fbdev-blacklist.conf
-rw-r--r-- 1  773 Mar 11  2022 systemd.conf

# shows current kernel ring buffer
sudo dmesg | grep -i driver
[501249.099477] usbcore: registered new interface driver usb-storage
[501249.101243] usbcore: registered new interface driver uas
# status of modules currently within linux kernel
# Used by is number of processes or modules currently using that Module
lsmod
Module                  Size  Used by
dm_crypt               53248  1
uas                    28672  0
usb_storage            77824  2 uas
tcp_diag               16384  0
inet_diag              24576  1 tcp_diag
sctp                  393216  10
...
# get detailed info about specific module
sudo modinfo bridge
filename:       /lib/modules/5.15.0-91-generic/kernel/net/bridge/bridge.ko
alias:          rtnl-link-bridge
version:        2.3
license:        GPL
srcversion:     65DA8B280548959EDB1E02D
depends:        stp,llc
retpoline:      Y
intree:         Y
name:           bridge
vermagic:       5.15.0-91-generic SMP mod_unload modversions 
...

```

## Installing kernel modules

Also called inserting or loading a module.

### insmod

Insert a single module:
- Very basic, so you have to provide an absolute directory reference to the module file.
- Doesn't load module dependencies

### modprobe

Insert module by module name:
- Inserts unloaded dependencies too

```bash
# install dm_mirror and dependencies with verbose flag
# calls insmod to do work
sudo modprobe -v dm_mirror
insmod /lib/modules/6.5.0-26-generic/kernel/drivers/md/dm-log.ko 
insmod /lib/modules/6.5.0-26-generic/kernel/drivers/md/dm-region-hash.ko 
insmod /lib/modules/6.5.0-26-generic/kernel/drivers/md/dm-mirror.ko 
# get status of dm_mirror
lsmod | grep -i dm_mirror
dm_mirror              24576  0
dm_region_hash         24576  1 dm_mirror
dm_log                 20480  2 dm_region_hash,dm_mirror
# get infor about dm_mirror
sudo modinfo dm_mirror
filename:       /lib/modules/6.5.0-26-generic/kernel/drivers/md/dm-mirror.ko
license:        GPL
author:         Joe Thornber
description:    device-mapper mirror target
srcversion:     E16D5E7307354AFA415433B
depends:        dm-region-hash,dm-log
retpoline:      Y
intree:         Y
name:           dm_mirror
vermagic:       6.5.0-26-generic SMP preempt mod_unload modversions 
...

# module dependencies are listed in /lib/modules/modules.deb
cat /lib/modules/6.5.0-26-generic/modules.dep | head
kernel/arch/x86/events/amd/amd-uncore.ko:
kernel/arch/x86/events/intel/intel-cstate.ko:
kernel/arch/x86/events/rapl.ko:
kernel/arch/x86/kernel/cpu/mce/mce-inject.ko:

# view module and dependencies
# dependencies are listed after the colon (:)
grep -i mirror /lib/modules/6.5.0-26-generic/modules.dep
kernel/drivers/md/dm-mirror.ko: kernel/drivers/md/dm-region-hash.ko kernel/drivers/md/dm-log.ko

# update modules.dep file
sudo depmod -v
```

## Removing kernel modules

```bash
# rm module but not its dependencies
sudo rmmod -v <module-name>
# remove module and its dependencies
# -r recursive
# -v verbose
sudo modprobe -rv dm_mirror
rmmod dm_mirror
rmmod dm_region_hash
rmmod dm_log

```