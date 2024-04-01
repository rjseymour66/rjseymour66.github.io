---
title: "Storage"
weight: 100
description: >
  Handling storage.
---


Help:

  DOS (MBR)
   a   toggle a bootable flag
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