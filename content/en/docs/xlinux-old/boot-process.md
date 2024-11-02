---
title: "Boot process"
weight: 50
description: >
  How a Linux machine boots up.
---

## Quick overview

1. Workstation starts and performs a quick check of the hardware with a Power-On Self Test (POST), then looks for a bootloader program to run from a bootable device.
2. Bootloader runs and determins which Linux kernel program to load.
3. Kernel program loads into memory and starts background programs required for the system to operate.

## View the boot process

Boot kernel messages are copied into the _kernel ring buffer_, a special ring buffer in memory. The buffer has limited space, so old messages are rotated out as new ones are rotated in. View them with `dmesg`:

```bash
sudo dmesg
[299027.516117] OOM killer disabled.
[299027.516119] Freezing remaining freezable tasks ... (elapsed 0.001 seconds) done.
[299027.517873] printk: Suspending console(s) (use no_console_suspend to debug)
[299030.501486] ACPI: EC: interrupt blocked
[299030.603894] ACPI: PM: Preparing to enter system sleep state S3
[299030.611125] ACPI: EC: event blocked
[299030.611126] ACPI: EC: EC stopped
[299030.611126] ACPI: PM: Saving platform NVS memory
[299030.611183] Disabling non-boot CPUs ...
...
```

Boot messages are also stored in a log file, usually /var/log/boot.log:

```bash
ls -lhost /var/log/boot*
  0 -rw------- 1 root   0 Feb 12 17:03 /var/log/boot.log
12K -rw------- 1 root 11K Feb 12 17:03 /var/log/boot.log.1
12K -rw------- 1 root 10K Nov 29 00:00 /var/log/boot.log.2
12K -rw------- 1 root 11K Oct 22 18:06 /var/log/boot.log.3
12K -rw------- 1 root 11K Oct 16 00:00 /var/log/boot.log.4
12K -rw------- 1 root 11K Aug 29  2023 /var/log/boot.log.5
24K -rw------- 1 root 24K Aug 28  2023 /var/log/boot.log.6
12K -rw------- 1 root 12K Jul 14  2023 /var/log/boot.log.7
64K -rw-r--r-- 1 root 61K Jun  8  2018 /var/log/bootstrap.log
```

## Firmware startup

Firmware controls how the installed OS starts:
- Basic Input/Output System (BIOS) is older
- Unified Extensible Firmware Interface (UEFI) is for newer systems

### BIOS

Could only read one sector's worth of data from a hard drive of memory, so it could not load an entire OS

- BIOS runs a bootloader program, a small program that initializes the necessary hardware to find and run the full OS program.
  - often found on same hard drive, but sometimes on external location like memory stick or ISO file
- When booting from a a hard drive, you have to define a Master Boot Record (MBR) that designates the hard drive and partition on that hard drive where BIOS can find the bootloader
- MBR is the first sector on the first HD partition on the system.
- Bootloader program points to location of the OS kernel file

### UEFI

- Uses special disk partition called the EFI System Partition (ESP) to store boot loader programs.
  - Allows any size program
  - Can store multiple bootloader programs for multiple OSs
- Uses Microsoft File Allocation Table (FAT) fs to store bootloader programs
  - Usually mounted in `/boot/efi`
- UEFI uses a mini-bootloader (boot manager) that lets you confi which bootloader program file to launch.
- You have to register each bootloader file you want to appear in the bootloader interface menu


## Linux bootloaders

Main bootloaders in Linux history:
- Linux loader (LILO)
- Grand Unified Bootloader (GRUB) legacy
- GRUB2

You can actually load the entire kernel without a bootloader since version 3.3.0, but everyone still uses GRUB2:
- developed in 2005
- can load hardware driver modules and use logic statements to dynamically alter boot menu options depending on system conditions

### Grub2

To view the GRUB menu, press the right SHIFT button on the keyboard during the boot process.

Config file is `/boot/grub/grub.cfg`, but you never modify it. You can update it with:

- Modifiable config files are in the `/etc/grub.d/`
  - create individual config files for each boot option
```bash
ls -lh /etc/grub.d/
total 144K
-rwxr-xr-x 1 root root  11K Dec  2  2022 00_header
-rwxr-xr-x 1 root root 6.2K Dec  2  2022 05_debian_theme
-rwxr-xr-x 1 root root  19K Dec  2  2022 10_linux
-rwxr-xr-x 1 root root  43K Dec  2  2022 10_linux_zfs
-rwxr-xr-x 1 root root  15K Dec 18  2022 20_linux_xen
-rwxr-xr-x 1 root root 2.9K Feb  6  2022 20_memtest86+
-rwxr-xr-x 1 root root  14K Dec  2  2022 30_os-prober
-rwxr-xr-x 1 root root 1.4K Dec  2  2022 30_uefi-firmware
-rwxr-xr-x 1 root root  700 Jul  2  2022 35_fwupd
-rwxr-xr-x 1 root root  214 Mar  4  2018 40_custom
-rwxr-xr-x 1 root root  215 Dec  2  2022 41_custom
-rwxr-xr-x 1 root root 1.3K Nov 11  2020 99_dell_recovery
-rw-r--r-- 1 root root  483 Mar  4  2018 README
```
Global commands are in `/etc/default/grub` config file:

```bash
$ cat /etc/default/grub
# If you change this file, run 'update-grub' afterwards to update
# /boot/grub/grub.cfg.
# For full documentation of the options in this file, see:
#   info -f grub -n 'Simple configuration'

GRUB_DEFAULT=0
GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=0
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
GRUB_CMDLINE_LINUX="nvidia-drm.modeset=1"

# Uncomment to enable BadRAM filtering, modify to suit your needs
# This works with Linux (no patch required) and with any kernel that obtains
# the memory map information from GRUB (GNU Mach, kernel of FreeBSD ...)
#GRUB_BADRAM="0x01234567,0xfefefefe,0x89abcdef,0xefefefef"

# Uncomment to disable graphical terminal (grub-pc only)
#GRUB_TERMINAL=console

# The resolution used on graphical terminal
# note that you can use only modes which your graphic card supports via VBE
# you can see them in real GRUB with the command `vbeinfo'
#GRUB_GFXMODE=640x480

# Uncomment if you don't want GRUB to pass "root=UUID=xxx" parameter to Linux
#GRUB_DISABLE_LINUX_UUID=true

# Uncomment to disable generation of recovery mode menu entries
#GRUB_DISABLE_RECOVERY="true"

# Uncomment to get a beep at grub start
#GRUB_INIT_TUNE="480 440 1"
GRUB_DISABLE_OS_PROBER=true
```

```bash
# update config (debian). -o option writes to a file, stdout by default
grub-mkconfig -o /boot/grub/grub.cfg

# find grub version (debian)
grub-mkconfig -V

# reinstall onto the primary hard disk
grub2-install /dev/sda
```

## System recovery

Most common issues are:
- Kernel failures
- Drive failures


### Kernel failure

- Kernel panic: when the kernel stops running in memory and crashes
- When you install a new kernel, you should leave the old kernel file in place and create an entry for it in the GRUB boot menu to point to it.

### Single-user mode

- Single-user mode is good when you want to perform maintenance, like adding a new module. You want to prevent other users from logging in
- Press `E` key on boot option in GRUB boot menu, then add `single` command to the end of the `linux` line in the boot menu commands. press CTRL + X to save.
  - Enter the root password when prompted.

### Passing kernel parameters

- Press `E` key on boot option in GRUB boot menu, then add your command to the `linux` line in the boot menu commands (no good example right now)

## Root drive failure

- Happens when the bootloader can't read the root drive device.