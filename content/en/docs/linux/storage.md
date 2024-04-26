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
- Creates links to the `/dev` storage device files to the `/dev/disk` folder. This way if the `/dev/` name changes as you add or remove devices, you can still access the correct storage.
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
| `/sbin` | Executable programs that the system requires |
| `/sys` | Virtual filesystem providind device, driver, and some kernel information as files, updated in real time |
| `/tmp` | Contains temporary files created by system users |
| `/usr` | Data for standard Linux programs |
| `/usr/bin` | Local user programs and data |
| `/usr/local` | Data for programs unique to the local installation |
| `/usr/sbin` | Data for system programs and data |
| `/var` | Files whose contents are expected to change frequently, such as log files |

## Formatting filesystems

To assign a drive partition to a mount point in the virtual filesystem, you have to format it using a filesystem

### Linux filesystems

Most linux systems use ext4 as default, but RHEL uses XFS. 
- Both use journaling, which tracks data not yet written to disk in a log file, called the journal
- If system crashes, journal can be recovered and stored on next system boot

Common linux filesystems:
- btrfs: Newer, high-performance.
  - Supports files up to 16 exbibytes (EiB) and total fs size of 16 EiB.
  - Can perform it own Redundant Array of Inexpensive Disks (RAID) and logical volume management (LVM).
  - Built-in snapshots improved fault tolerance, on demand data compression
- eCryptfs: Enterproce Cryptographic File System
  - Applies POSIX-compliant encryption to data before it is saved on the device.
  - Only the OS that created the fs can read data from it
- ext3: Descendant of original Linux ext fs.
  - Supports files up to 2 tebibites (TiB) and total fs of 16 TiB
  - Supports journaling, faster startup and recovery
- ext4: Current version of original Linux fs
  - Supports files up to 16 TiB and total fs of 1 EiB
  - Supports journaling, performance features
- XFS: 64-bit high-performance journaling fs
  - Supports fs up to 8 exibibytes
- swap: Create virtual memory for your system using space on physical disk
  - System swaps data out of normal memory into swap space, which adds additional memory to your system
  - Not for persistent data

### Non-linux filesystems

Linux can read data off filesystems created by other OSes:
- CIFS: Common Internet Filesystem.
- HFS: Hierarchical File System
- ISO-9660: Staandard used for creating filesystms on CD-ROM devices.
- NFS: Network File System. Open-source standard for reading and writing data across a network using a network storage device.
- NTFS: New Technology File System (NTFS). Used by Microsoft NT operating system.
- SMB: Server Message Block fs, created by MS
- UDF: Universal Disc Format, used on DVD-ROM devices for storing data
- VFAT: Virtual File Allocation Table, extension of original MS FAT system.
  - Commonly used on removable storage like USB memory sticks
- ZFS: Zettabyte File Stystem, created by Sun for Unix workstations and servers. 
  - Similar to btrfs

## Creating filesystems

### mkfs

```bash
mkfs -t <fs-type> <drive-partition>

# format partition sda1 on device sda
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
```
## Mounting filesystems

After you format a drive partition, you add it to the virtual filesystem. This is called _mounting_.

### mount (manual mounting)

Temporarily mounts the device in the virtual directory.
- Good for removable/USB devices

```bash
mount -t <fstype> <device> <mountpoint>

# create mountpoint dir
mkdir mount_here

# mount /dev/sda1 to mountpoint
sudo mount -t ext4 /dev/sda1 mount_here/

# by itself, mount displays all devices currently mounted in the fs
mount
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
udev on /dev type devtmpfs (rw,nosuid,relatime,size=16150204k,nr_inodes=4037551,mode=755,inode64)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
...
```

### umount

Remove device from the virtual filesystem:

```bash
umount [<device-filename> | <mount-point-dir>]
sudo umount /dev/sda1
```

## Automatically mounting devices

`/etc/fstab` indicates which devices should be mounted at boot.
- Drive device file is either the raw file or one of its permanent `udev` filenames:
  ```bash
  <drive-device-file> <mount point> <fstype> <options>
  ```
- `<drive-device-file>` is usually the `udev` UUID value to ensure that the correct drive partition is accessed, regardless of the order that it appears in the raw device table.
- `systemd` manages mounted filesystems. All mount points in `/etc/fstab` are converted into native units when server is booted or systemd reloads.

> If you add a device to `/etc/fstab` and it is not available at boot time, you get a boot error.

```bash
cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/nvme0n1p3 during installation
UUID=9b2aa8ce-a32d-44eb-9178-18f77a7d9039 /               ext4    errors=remount-ro 0       1
# /boot/efi was on /dev/nvme0n1p1 during installation
UUID=AEAA-7165  /boot/efi       vfat    umask=0077      0       1
/swapfile                                 none            swap    sw              0       0
```

## Retrieving filesystem stats

### df

Displays disk usage by partition. Use the `-i` option to see the number of inodes left in the fs. ext3 and ext4 filesystems allocate a specific number of inodes, and you can't make more files when you run out of inodes.

```bash
df -t <fstype> -i -h 

# all disk partitions, human-readable
df -ih
Filesystem     Inodes IUsed IFree IUse% Mounted on
tmpfs            3.9M  1.6K  3.9M    1% /run
/dev/nvme0n1p3    30M  1.7M   28M    6% /
tmpfs            3.9M   323  3.9M    1% /dev/shm
tmpfs            3.9M     6  3.9M    1% /run/lock
/dev/nvme0n1p1      0     0     0     - /boot/efi
tmpfs            793K   178  793K    1% /run/user/1001

# ext4 disk partitions
df -it ext4 -h
Filesystem     Inodes IUsed IFree IUse% Mounted on
/dev/nvme0n1p3    30M  1.7M   28M    6% /

# tmpfs partitions
df -it tmpfs -h
Filesystem     Inodes IUsed IFree IUse% Mounted on
tmpfs            3.9M  1.6K  3.9M    1% /run
tmpfs            3.9M   323  3.9M    1% /dev/shm
tmpfs            3.9M     6  3.9M    1% /run/lock
tmpfs            793K   178  793K    1% /run/user/1001
```

### du

Displays disk usage by directory. Good for finding users or apps that take up the most disk space:

```bash
# -d is directory depth
du -d 1
12	./assets
8	./layouts
788	./resources
6460	./.git
1288	./content
11012	./node_modules
12	./.github
8	./themes
19700	.
```

### iostat

Real-time chart of disk statistics by partition:

```bash
iostat
Linux 5.15.0-91-generic (precision-5540) 	04/03/2024 	_x86_64_	(12 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
          24.94    0.26    9.50    0.03    0.00   65.27

Device             tps    kB_read/s    kB_wrtn/s    kB_dscd/s    kB_read    kB_wrtn    kB_dscd
loop0             0.00         0.00         0.00         0.00         34          0          0
loop1             0.00         0.00         0.00         0.00       3708          0          0

```

### lsblk

Current partition sizes and mount points:

```bash
lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0         7:0    0     4K  1 loop /snap/bare/5
loop1         7:1    0  12.9M  1 loop /snap/snap-store/1113
...
loop36        7:36   0 504.2M  1 loop /snap/gnome-42-2204/172
nvme0n1     259:0    0 476.9G  0 disk 
├─nvme0n1p1 259:1    0   790M  0 part /boot/efi
├─nvme0n1p2 259:2    0     5G  0 part 
└─nvme0n1p3 259:3    0 471.2G  0 part /var/snap/firefox/common/host-hunspell
```

### /proc/partitions, /proc/mounts

The kernel uses `/proc` to record process statistics:
- `/proc/partitions`: info on system partitions
- `/proc/mounts`: info on system mount points


### /sys/block

The kernel uses `/sys` to record system statistics. `/sys/block` contains separate folders for eah mounted drive, showing partitions and kernel-level stats.


### e2fsprogs package

This pacakge provides utilities for working with ext3 and ext4 filesystems:
- `blkid`: Displays info about block devices
- `chattr`: Changes file attributes on the fs
- `debugfs`: Manually views and modifies the fs structure, such as undeleting a file
- `dumpe2fs`: Displays block and superblock group information
- `e2lable`: Changes the fs lable
- `resize2fs`: Expands or shrinks the fs
- `tune2fs`: modifies fs params

### fsck

Checks and repairs an fs. The fs must be unmounted:

```bash
sudo fsck -f /dev/sda1
```
If it returns an error, run it in repair mode.


## Storage alternatives

Standard parition layouts have limitations: you can't resize them, andn they are susceptible to failures. Linux provides more storage options that are more dynamic.

### Multipath

- Device Mapper Multipathing (DM-multipathing) lets you configure multiple paths between Linux and network storage devices.
- Uses `/dev/mapper` device folder
  - `/dev/mapper/mpath`_`N`_ for each multipath drive (N is the number that represents the drive)
  - Device file acts as normal device file: you can partition it 
- Increases throughput when all paths are active
- Provides fault tolerance when one or more paths are inactive
- Tools:
  - `dm-multipath`: Kernel module that supports multipathing
  - `multipath`: command line command for viewing multipath devices
  - `multipathd`: daemon for monitoring, activating, and deactivating paths
  - `kpartx`: command line tool for creating device entries for multipath storage

### Logical Volume Manager (LVM)

- Aggregates multiple physical drive partitions into virtual volumes that you can treat as a single partition on your system
  - For example, if you have physical drives `/dev/sda` and `/dev/sdb`, and each volume has 3 partitions, you can create 2 logical partitions by mixing the partitions:
    - `/dev/sda1` and `/dev/sda2`
    - `/dev/sda3`, `/dev/sdb1`, `/dev/sdb2`, `/dev/sdb3`
  - You can add or remove phusical partitions from the logical volume as needed
- Creates virtual drive devices
- Uses `/dev/mapper` device folder
  - Format these with a filesystem like a normal partition

#### Creating an LVM

[LVM commands](https://man7.org/linux/man-pages/man8/lvm.8.html#COMMANDS)

```bash
# create a lvm partition from a USB drive
sudo gdisk /dev/sda
GPT fdisk (gdisk) version 1.0.8

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): n   # new partition
Partition number (1-128, default 1): 1
First sector (34-120176606, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-120176606, default = 120176606) or {+-}size{KMGTP}: 
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 8e00    # code for LVM
Changed type of partition to 'Linux LVM'

Command (? for help): w     # write it

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): Y
OK; writing new GUID partition table (GPT) to /dev/sda.
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot or after you
run partprobe(8) or kpartx(8)
The operation has completed successfully.

# create the physical volume
sudo pvcreate /dev/sda1
WARNING: ext4 signature detected on /dev/sda1 at offset 1080. Wipe it? [y/n]: y
  Wiping ext4 signature on /dev/sda1.
  Physical volume "/dev/sda1" successfully created.

# combine physical volume into volume group
sudo vgcreate newvol /dev/sda1
  Volume group "newvol" successfully created

# create a logical volume
sudo lvcreate -l 100%FREE -n lvdisk newvol
  Logical volume "lvdisk" created.

# format the logical volume w ext4
sudo mkfs -t ext4 /dev/mapper/newvol-lvdisk 
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 15021056 4k blocks and 3760128 inodes
Filesystem UUID: 0b2b523d-b54f-4c0c-b74b-2a6267b83b05
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (65536 blocks): done
Writing superblocks and filesystem accounting information: done   

# make mount point 
sudo mkdir /media/newdisk

# mount the logical volume to mount point
sudo mount /dev/mapper/newvol-lvdisk /media/newdisk/

# view mount point contents
cd /media/newdisk/
ll -a
total 24
drwxr-xr-x 3 root root  4096 Apr  3 23:02 ./
drwxr-xr-x 4 root root  4096 Apr  3 23:03 ../
drwx------ 2 root root 16384 Apr  3 23:02 lost+found/
```

## RAID technology

[Digital Ocean](https://www.digitalocean.com/community/tutorials/an-introduction-to-raid-terminology-and-concepts)

- Redundant Array of Inexpensive Disks (RAID) lets you combine multiple drives into a single virtual drive to improve data access performance and implement data redundancy for fault tolerance
- [Common versions of RAID](https://en.wikipedia.org/wiki/Standard_RAID_levels):
  - RAID-0
  - RAID-1
  - RAID-10
  - RAID-4
  - RAID-5
  - RAID-6
- Despite its name, RAID can be expensive and impractical

### mdadm

[Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-create-raid-arrays-with-mdadm-on-ubuntu-22-04)

- Lets you specify multiple partitions to use in RAID environment.
- RAID device appears in `/dev/mapper`
  - You can partition and format with an fs
- `/proc/mdstat` contains current status of kernel RAID state

## Encrypting partitions

- File encryption can be tedius, so you can encrypt entire partitions.
- Linux Unified Key Setup (LUKS) has utility `cryptsetup`
  - Create encrypted partitions, then open them to format them with an fs and mount

```bash
# format a partition for encryption
sudo cryptsetup -y -v luksFormat /dev/sda1

WARNING!
========
This will overwrite data on /dev/sda1 irrevocably.

Are you sure? (Type 'yes' in capital letters): YES
Enter passphrase for /dev/sda1: 
Verify passphrase: 
Key slot 0 created.
Command successful.

# make it avialable for use with luksOpen
sudo cryptsetup -v luksOpen /dev/sda1 safedata
No usable token is available.
Enter passphrase for /dev/sda1: 
Key slot 0 unlocked.
Command successful.

# safedata references opened encrypted partition
ls /dev/mapper/ -l
total 0
crw------- 1 root root 10, 236 Mar 16 09:39 control
lrwxrwxrwx 1 root root       7 Apr  3 23:32 safedata -> ../dm-0

# format with an fs
sudo mkfs -t ext4 /dev/mapper/safedata
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 15017723 4k blocks and 3760128 inodes
Filesystem UUID: fa3f9078-2873-4d67-b199-63b6f3236ed5
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (65536 blocks): done
Writing superblocks and filesystem accounting information: done   

# mount it 
sudo mount /dev/mapper/safedata /mnt/mydata

# close the encrypted partition and remove it from /dev/mapper
sudo cryptsetup -v luksClose /dev/mapper/safedata
Device /dev/mapper/safedata is still in use.
Command failed with code -5 (device already exists or device is busy).

# to access the partition again, remount it with luksOpen 
```

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

## Troubleshooting

### Filesystem space

You need filesystem space. If you run out, add more with LVM.

This section uses these commands:
- `df` displays overall space usage
- `du` displays usage by directory

```bash
# view ext4 usage, human-readable
df -ht ext4
Filesystem      Size  Used Avail Use% Mounted on
/dev/nvme0n1p3  463G  256G  184G  59% /

# view disk usage by directory, starting at root
sudo du -d 1 /
19804038	/snap
31676	/root
...
2728	/run
...
0	/proc
1034748	/opt
420	/tmp
4	/cdrom
13588708	/usr
20	/target
377328	/boot
60	/dev
16	/lost+found
168366840	/home
0	/sys
8	/mnt
4	/srv
21828	/etc
17593112	/var
8	/media
287930426	/

# drill down into a specific subdir
sudo du -d 1 /snap
85815	/snap/snap-store
742578	/snap/gtk-common-themes
10178	/snap/gnome-system-monitor
507308	/snap/core22
1459713	/snap/firefox
396587	/snap/cups
4210	/snap/snapd-desktop-integration
248417	/snap/ffmpeg
596618	/snap/core
4	/snap/bin
2568216	/snap/gnome-42-2204
1746599	/snap/gnome-3-34-1804
428069	/snap/core20
1847998	/snap/gnome-3-38-2004
1151578	/snap/gnome-3-26-1604
6933472	/snap/intellij-idea-ultimate
729770	/snap/chromium
5	/snap/bare
346899	/snap/core18
19804038	/snap
```

### I/O wait with iostat

I/O wait is a performance statistic that shows the amount of time a processor must wait on disk I/O:

> `iostat` is available in the `sysstat` package.

```bash
iostat [OPTION] [INTERVAL] [COUNT]
-y # exclude 'since system booted' stats
-N # display registered device mapper names for logical vols
-z # exclude devices w/no activity
-p device # 

# by itself, displays summary stats
iostat
Linux 5.15.0-105-generic (precision-5540) 	04/22/2024 	_x86_64_	(12 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           3.02    0.12    1.56    0.03    0.00   95.26

Device             tps    kB_read/s    kB_wrtn/s    kB_dscd/s    kB_read    kB_wrtn    kB_dscd
loop0             0.00         0.00         0.00         0.00         17          0          0
...

# view two iostat calls, five seconds apart
iostat -yNz 5 2
Linux 5.15.0-105-generic (precision-5540) 	04/22/2024 	_x86_64_	(12 CPU)


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.59    0.05    0.79    0.02    0.00   98.56

Device             tps    kB_read/s    kB_wrtn/s    kB_dscd/s    kB_read    kB_wrtn    kB_dscd
nvme0n1           5.00         0.00       281.60         0.00          0       1408          0


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.82    0.03    0.69    0.02    0.00   98.44

Device             tps    kB_read/s    kB_wrtn/s    kB_dscd/s    kB_read    kB_wrtn    kB_dscd
nvme0n1           4.20         0.00        20.00         0.00          0        100          0
```

## I/O scheduling

For high I/O issues, you need to investigate the kernel's defined I/O scheduling. I/O scheduling is a series of kernel actions that handle I/O requests and their related activities. 

| Scheduler | Description |
|---|---|
| `cfq` | Creates queues for each process and handles the various queues in a loop. Prioritizes read requests over write requests. Good for more balance I/O is needed and/or a multiprocessor. |
| `deadline` | Batches disk I/O requests and attempts to handle each request by a specified time. Good where increased db I/O and overall reduced I/O is needed, and/or SSD is employed, and/or real-time apps are in use. |
| `noop` | Puts all I/O requests in FIFO queue and handles them in order. Good where less CPU usage is needed and/or SSD is in use. |

The configuration file for the disk scheduler is in a directory associated with the disk:

```bash
# view disks
ls /sys/block/
loop0  loop10  loop12  loop2  loop4  loop6  loop8  sda
loop1  loop11  loop13  loop3  loop5  loop7  loop9  sr0

# view scheduler file
cat /sys/block/sda/queue/scheduler 
none [mq-deadline]
```

### fio

I/O operations per second (IOPS) is the number of input or output operations that a storage device can perform in a second. Measure this with `fio`:

```bash
# look for read and write vals and compare to other disks. Writes are always significantly lower than reads.
\# fio \
> --randrepeat=1 \
> --ioengine=libaio \
> --direct=1 \
> --gtod_reduce=1 \
> --name=fiotest \
> --filename=fiotest \
> --bs=4k \
> --iodepth=64 \
> -size=1G \
> -readwrite=randrw \
> -rwmixread=75
fiotest: (g=0): rw=randrw, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=64
fio-3.28
Starting 1 process
fiotest: Laying out IO file (1 file / 1024MiB)
Jobs: 1 (f=1): [m(1)][100.0%][r=102MiB/s,w=34.4MiB/s][r=26.1k,w=8803 IOPS][eta 00m:00s]
fiotest: (groupid=0, jobs=1): err= 0: pid=14101: Mon Apr 22 14:06:40 2024
  read: IOPS=25.8k, BW=101MiB/s (106MB/s)(768MiB/7616msec)    # important
   bw (  KiB/s): min=75760, max=118968, per=100.00%, avg=103337.53, stdev=14946.47, samples=15
   iops        : min=18940, max=29742, avg=25834.20, stdev=3736.69, samples=15
  write: IOPS=8619, BW=33.7MiB/s (35.3MB/s)(256MiB/7616msec); 0 zone resets     # important
   bw (  KiB/s): min=25373, max=40440, per=100.00%, avg=34524.20, stdev=4956.99, samples=15
   iops        : min= 6343, max=10110, avg=8630.87, stdev=1239.25, samples=15
  cpu          : usr=2.43%, sys=87.50%, ctx=54501, majf=0, minf=8
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.1%, >=64=100.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.1%, >=64=0.0%
     issued rwts: total=196498,65646,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=64

Run status group 0 (all jobs):
   READ: bw=101MiB/s (106MB/s), 101MiB/s-101MiB/s (106MB/s-106MB/s), io=768MiB (805MB), run=7616-7616msec
  WRITE: bw=33.7MiB/s (35.3MB/s), 33.7MiB/s-33.7MiB/s (35.3MB/s-35.3MB/s), io=256MiB (269MB), run=7616-7616msec

Disk stats (read/write):
  sda: ios=193366/64635, merge=5/16, ticks=67138/8912, in_queue=76056, util=98.89%
```

### fstrim

`fstrim` recovers memory locations for solid-state devices (SSDs). When you delete data on an SSD, the system needs to recover these locations that are now empty because they can slow down the read and write access to the SSD device.

### Failing disk

If a chunk of HDD or SSD does not respond to I/O requests, then it is marked as a bad sector, and the firmware will move any data from it to a new location. If you see bad sectors, replace the disk.

Unmount the disk, then use `fsck` to check and repair the disk.

## Troubleshooting

### Common issues

Degraded storage/mode
: _Degraded storage_ is a storage's gradual decay due to time or misuse. SSD degrades if you erase it a lot, which means that you should not use it for your swap space.

  _Degraded mode_ is when one or more disks in a RAID array failed. Use `mdadm -D` to view the array's detailed status, and look for the word `degraded`.

Missing devices
: If you can't find a device, check the following:
  - `lsblk` to view all block devices
  - `lspci -M` to perform a detailed scan of all PCI-attached devices
  - Look at the device file in `/dev/`. For example, `/dev/sda`.
    - Whole disk is referred to by the device filename with no numbers. Ex: `/dev/sda`
    - Disk partition is device filename and number. Ex: `/dev/sda1`
    - NvME SSD includes a namespace. Ex:
      ```bash
      nvme0n1     259:0    0 476.9G  0 disk             # device name is 'nvme0', namespace 1
      ├─nvme0n1p1 259:1    0   790M  0 part /boot/efi   # namespace 1, partition 1
      ├─nvme0n1p2 259:2    0     5G  0 part 
      └─nvme0n1p3 259:3    0 471.2G  0 part /var/snap/firefox/common/host-hunspell
      ```
 
Missing volumes
: If you can't ID a physical volume that makes up part of the logical volume:
  - `pvscan` to verify whether it was lost
  - `pvcreate` to replace the volume
  - `vgcfgrestore` to restore the group's metadata
  - `vgscan` to recover the group
  - `vgchange` to activate the new volume

Missing mount points
: If you see 'Mount point does not exist', create the mount point with `mkdir`.

Storage integrity
: A _bad block_ is a small chunk of a disk drive that does not respond to I/O requests due to corruption or physical damage. Run `badblocks -nsv <partition-device-file>` to monitor the drive. It focuses on a partition and does not perfom repairs.

Performance issues
: Poor storage performance can affect your apps.
  - `hdparm` determines read speeds.
  - `dstat` is similar to `iostat` but provides additional helpful data for troubleshooting performance problems.
  - `dmstats` allows the setup and management of stats for any devices charted by the device mapper
  - `lsblk -p` to determine device mapper filenames associated with logical volumes

Resource exhaustion
: When you run out of resources such as disk space or inode numbers.

### Check device driver

```bash
# THIS DID NOT WORK, TODO
# find the device in the kernel buffer ring
dmesg | grep sda
[91914.548960] sd 3:0:0:0: [sda] 120176640 512-byte logical blocks: (61.5 GB/57.3 GiB)
[91914.550065] sd 3:0:0:0: [sda] Write Protect is off
[91914.550074] sd 3:0:0:0: [sda] Mode Sense: 43 00 00 00
[91914.550545] sd 3:0:0:0: [sda] Write cache: disabled, read cache: enabled, doesn't support DPO or FUA
[91914.587927]  sda:
[91914.590460] sd 3:0:0:0: [sda] Attached SCSI removable disk
[95002.730995] sd 3:0:0:0: [sda] 120176640 512-byte logical blocks: (61.5 GB/57.3 GiB)
[95002.731675] sd 3:0:0:0: [sda] Write Protect is off
[95002.731676] sd 3:0:0:0: [sda] Mode Sense: 43 00 00 00
[95002.732082] sd 3:0:0:0: [sda] Write cache: disabled, read cache: enabled, doesn't support DPO or FUA

# find the driver
ls /sys/bus/scsi/drivers
sd  sr

# confirm which driver is used for sd devices
udevadm info -an /dev/sda | grep DRIVERS | grep sd
    DRIVERS=="sd"

# determine if the module is loaded 
lsmod | grep sd
snd_intel_sdw_acpi     20480  2 snd_sof_intel_hda_common,snd_intel_dspcfg
rtsx_pci_sdmmc         32768  0
rtsx_pci               98304  1 rtsx_pci_sdmmc
```

## SATA drives

SATA drives are self-configuring and are typically connected to the SCSI bus and denoted by `/dev/sd*` files.
- can fail earlier than others due to frequent head loads and unloads
- check on these drives with the self-monitoring analysis and reporting technology (SMART)