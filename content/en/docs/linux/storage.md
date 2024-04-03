---
title: "Storage"
weight: 100
description: >
  Handling storage.
---

## Storage basics

- Hard drive disk (HDD): Physical devices that store data on disk platters that spin and store data magnetically on the platters with a movable r/w head. Increasingly rare.
- Solid-state drives (SDD): Integrated circuts that store data electronically. There are no moving parts.

### Drive connections

Four main types of drive connections:
- PATA (Parallel Advanced Technology Attachment): Parallel interface, supports 2 devices per adapter.
  - Drive device file is `/dev/hd<x>`, where `<x>` is the letter that represents the drive, starting with `a`.
- SATA (Serial Advanced Technology Attachment): Serial interface, faster than PATA. Supports up to 4 devices per adapter.
  - Drive device file is `/dev/sd<x>`, where `<x>` is the letter that represents the drive, starting with `a`. So, `/dev/sda`, `/dev/sdb`.
- SCSI (Small Computer System Interface): Parallel interface, as fast as SATA. Supports up to 8 devices per adapter.
  - Drive device file is `/dev/sd<x>`, where `<x>` is the letter that represents the drive, starting with `a`.
- NVMe (Nonvolatile Memory Express): connects SSDs w/parallel interface. Supports up to 12 devices per adapter.
  - Drive device file is `/dev/nvme<x>`, where `<x>` is the letter that represents the drive, starting with `a`.

### Partitioning drives

A partition is a self-contained section in the drive that the OS treats as a separate storage space.
- Can help you organize data

Must be tracked by a indexing system on the drive
- BIOS systems use the MBR to manage disk partitions
  - Only supports up to 4 partitions on a drive
  - Can split each primary into an extended partition. Extended partitions are numbered starting at `5`.
- UEFI use GUID Partition Table (GPT) to manage partitions
  - Supports up to 128 partitons on a drive

### Automatic drive detection

Linux systems used to detect new drives at boot, but now you can insert or remove USB drives all the time. Now, `udev` application runs in the background and detects new hardware and assigns device filenames in `/dev`
- Createslinks to the `/dev` storage device files to the `/dev/disk` folder. This way if the `/dev/` name changes as you add or remove devices, you can still access the correct storage.
- `/dev/disk` has four folders:
  - `/dev/disk/by-id`: manufacturer details
  - `/dev/disk/by-label`: label assigned to them
  - `/dev/disk/by-path`: port
  - `/dev/disk/by-uuid`: UUID assigned to device.

## Partitioning tools

### fdisk

Most common partitioning tool. `fdisk` works on any drie that uses the MBR method of partition indexing.
- Can't alter size of existing partitions - must delete and build from scratch


```bash
DOS (MBR)
  a   toggle a bootable flag              # boot the system from the partition
  b   edit nested BSD disklabel
  c   toggle the dos compatibility flag

Generic
  d   delete a partition
  F   list free unpartitioned space
  l   list known partition types
  n   add a new partition
  p   print the partition table
  t   change a partition type
  v   verify the partition table
  i   print information about a partition

Misc
  m   print this menu
  u   change display/entry units
  x   extra functionality (experts only)

Script
  I   load disk layout from sfdisk script file
  O   dump disk layout to sfdisk script file

Save & Exit
  w   write table to disk and exit
  q   quit without saving changes

Create a new label
  g   create a new empty GPT partition table
  G   create a new empty SGI (IRIX) partition table
  o   create a new empty DOS partition table
  s   create a new empty Sun partition table
```

### gdisk

For drives that use the GPT indexing method:

```bash
sudo gdisk /dev/<drive-file>
# example
sudo gdisk /dev/sda

b	back up GPT data to a file
c	change a partition's name
d	delete a partition
i	show detailed information on a partition
l	list known partition types
n	add a new partition
o	create a new empty GUID partition table (GPT)
p	print the partition table
q	quit without saving changes
r	recovery and transformation options (experts only)
s	sort partitions
t	change a partition's type code
v	verify disk
w	write table to disk and exit
x	extra functionality (experts only)
?	print this menu

# i option
Command (? for help): i
Using 1
Partition GUID code: 0FC63DAF-8483-4772-8E79-3D69D8477DE4 (Linux filesystem)
Partition unique GUID: CC310D9F-5FEB-4642-AE6B-115522E2266C
First sector: 2048 (at 1024.0 KiB)
Last sector: 120176639 (at 57.3 GiB)
Partition size: 120174592 sectors (57.3 GiB)
Attribute flags: 0000000000000000
Partition name: 'Linux filesystem'
```
### parted

GNU parted lets you modify existing partition sizes:

```bash
sudo parted
GNU Parted 3.4
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print                                                            
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sda: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name                  Flags
 1      1049kB  2097kB  1049kB                                     bios_grub
 2      2097kB  540MB   538MB   fat32        EFI System Partition  boot, esp
 3      540MB   21.5GB  20.9GB  ext4

(parted) 
```
## Filesystems

- Filesystems manage data stored on storage devices.
- Linux uses _virtual directory_, which contains file paths from all storage devices consolidated into a single directory structure

### Virtual directory

- Root directory (not `/root`) is base directory
- Linux places physical devices in the virtual directory using mount points
- _Mount point_ is a folder placeholder in the virtual directory that points to a specific device

#### Linux filesystem hierarchy standard (FHS)

| Folder | Description |
|--------|-------------|
| `/bin` | Executable programs that the system needs to run in single-user mode |
| `/boot` | Contains bootlader files that boot the system |
| `/dev` | Device files |
| `/etc` | System service config files |
| `/home` | User data files |
| `/lib` | Library files required by executable programs |
| `/media` | Mount point for removable devices |
| `/mnt` | Mount point for removable devices |
| `/opt` | Data for optional third-party programs |
| `/proc` | Virtual filesystem that provides kernel and processing information as files, updated in real time |
| `/root` | Home dir for root user |
| `/sbin` | Executable programs that the systm requires |
| `/sys` | Virtual filesystem providind device, driver, and some kernel information as files, updated in real time |
| `/tmp` | Contains temporary files created by system users |
| `/usr` | Data for standard Linux programs |
| `/usr/bin` | Local user programs and data |
| `/usr/local` | Data for programs unique to the local installation |
| `/usr/sbin` | Data for system programs and data |
| `/var` | Files whose contents are expected to change frequently, such as log files |

## Formatting filesystems




## Formatting a USB drive

```bash
# 1. Insert a USB drive into the computer

# 2. check system console for
sudo dmesg | tail
[501249.099477] usbcore: registered new interface driver usb-storage
[501249.101243] usbcore: registered new interface driver uas
[501250.129224] scsi 3:0:0:0: Direct-Access      USB      SanDisk 3.2Gen1 1.00 PQ: 0 ANSI: 6
[501250.129751] sd 3:0:0:0: Attached scsi generic sg0 type 0
[501250.130299] sd 3:0:0:0: [sda] 120176640 512-byte logical blocks: (61.5 GB/57.3 GiB)
[501250.131154] sd 3:0:0:0: [sda] Write Protect is off
[501250.131161] sd 3:0:0:0: [sda] Mode Sense: 43 00 00 00
[501250.131545] sd 3:0:0:0: [sda] Write cache: disabled, read cache: enabled, doesn\'t support DPO or FUA
[501250.144239]  sda: sda1  # device: partition

# 3. Unmount the device
sudo umount /dev/sda1

# 4. partition the disk with fdisk
sudo fdisk /dev/sda

Welcome to fdisk (util-linux 2.37.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): p
Disk /dev/sda: 57.3 GiB, 61530439680 bytes, 120176640 sectors
Disk model:  SanDisk 3.2Gen1
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00000000

Device     Boot Start       End   Sectors  Size Id Type
/dev/sda1          32 120176639 120176608 57.3G  c W95 FAT32 (LBA)

# 5. Delete the existing partition
Command (m for help): d
Selected partition 1
Partition 1 has been deleted.

# 6. View and notice that there is no partition
Command (m for help): p
Disk /dev/sda: 57.3 GiB, 61530439680 bytes, 120176640 sectors
Disk model:  SanDisk 3.2Gen1
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00000000

# 7. Create new partition
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)

# 8. Create primary partition
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-120176639, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-120176639, default 120176639): 

Created a new partition 1 of type 'Linux' and of size 57.3 GiB.

# 9. Save the new partition layout
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

# 10. Create new fs on new partition
sudo mkfs -t ext4 /dev/sda1
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 15021824 4k blocks and 3760128 inodes
Filesystem UUID: 66fe4850-1158-4305-ac07-7c13e9f6736a
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (65536 blocks): done
Writing superblocks and filesystem accounting information: done   

# 11. Create dir to use as a new mount point
mkdir mediatest1

# 12. Mount the new fs to the mount point
sudo mount -t ext4 /dev/sda1 mediatest1/

# 13. Unmount the USB stick
sudo umount /dev/sda1

```