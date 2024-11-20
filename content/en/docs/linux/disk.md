---
title: "Disks"
linkTitle: "Disks"
# weight: 1000
# description:
---

## Partition a usb, then add a filesystem on it

<!-- xlinux-old/storage/#formatting-a-usb-drive -->

1. locate the disk that you want to partition. This naming scheme varies from one device to another

```bash
# dmesg to see kernel messages for the usb
sudo dmesg | tail
[  886.725353] usb-storage 2-1:1.0: USB Mass Storage device detected
[  886.725732] scsi host3: usb-storage 2-1:1.0
[  887.816989] scsi 3:0:0:0: Direct-Access      USB      SanDisk 3.2Gen1 1.00 PQ: 0 ANSI: 6
[  887.819866] sd 3:0:0:0: Attached scsi generic sg2 type 0
[  887.826848] sd 3:0:0:0: [sdb] 120164352 512-byte logical blocks: (61.5 GB/57.3 GiB)
[  887.829822] sd 3:0:0:0: [sdb] Write Protect is off
[  887.829834] sd 3:0:0:0: [sdb] Mode Sense: 43 00 00 00
[  887.831965] sd 3:0:0:0: [sdb] Write cache: disabled, read cache: enabled, doesn\'t support DPO or FUA
[  887.846908]  sdb: sdb1
[  887.847344] sd 3:0:0:0: [sdb] Attached SCSI removable disk

# list block devices. If no path under MOUNTPOINTS, its not mounted:
lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   25G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   23G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:0    0 11.5G  0 lvm  /
sdb                         8:16   1 57.3G  0 disk # usb drive
└─sdb1                      8:17   1 57.3G  0 part # partition (takes up entire disk)
sr0                        11:0    1 1024M  0 rom

# view mountpoint in filesystem
ll /dev/sdb*
brw-rw---- 1 root disk 8, 16 Nov 10 23:41 /dev/sdb
brw-rw---- 1 root disk 8, 17 Nov 10 23:41 /dev/sdb1

# detailed information with fdisk
> sudo fdisk -l
Disk /dev/sda: 25 GiB, 26843545600 bytes, 52428800 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: E709F353-19E4-4540-AA41-D8CA695DB004

Device       Start      End  Sectors Size Type
/dev/sda1     2048     4095     2048   1M BIOS boot
/dev/sda2     4096  4198399  4194304   2G Linux filesystem
/dev/sda3  4198400 52426751 48228352  23G Linux filesystem


Disk /dev/mapper/ubuntu--vg-ubuntu--lv: 11.5 GiB, 12343836672 bytes, 24109056 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdb: 57.3 GiB, 61524148224 bytes, 120164352 sectors  # usb drive
Disk model:  SanDisk 3.2Gen1
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00000000

Device     Boot Start       End   Sectors  Size Id Type
/dev/sdb1          32 120164351 120164320 57.3G  c W95 FAT32 (LBA)

```

2. Verify that your disk is not mounted. View all mounted devices, filter for your device (skip if not mounted):

```bash
mount
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
udev on /dev type devtmpfs (rw,nosuid,relatime,size=4030056k,nr_inodes=1007514,mode=755,inode64)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,nodev,noexec,relatime,size=813272k,mode=755,inode64)
/dev/mapper/ubuntu--vg-ubuntu--lv on / type ext4 (rw,relatime)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev,inode64)
tmpfs on /run/lock type tmpfs (rw,nosuid,nodev,noexec,relatime,size=5120k,inode64)
cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime,nsdelegate,memory_recursiveprot)
...
```

3. Unmount with `umount` if mounted: 
```bash
sudo umount
```
13:40 in https://www.youtube.com/watch?v=2Z6ouBYfZr8&t=183s

4. Run `fdisk` to start partitioning:

```bash
sudo fdisk /dev/sdb

Welcome to fdisk (util-linux 2.39.3).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

# p to view existing partitions
Command (m for help): p
Disk /dev/sdb: 57.3 GiB, 61524148224 bytes, 120164352 sectors
Disk model:  SanDisk 3.2Gen1
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00000000

Device     Boot Start       End   Sectors  Size Id Type
/dev/sdb1          32 120164351 120164320 57.3G  c W95 FAT32 (LBA)

# create new GPT partition, which wipes out the existing partition table
Command (m for help): g
Created a new GPT disklabel (GUID: 8AE83933-1443-47D3-BADB-EFADD54FD659).
The device contains 'dos' signature and it will be removed by a write command. See fdisk(8) man page and --wipe option for more details.

# partition table is empty
Command (m for help): p

Disk /dev/sdb: 57.3 GiB, 61524148224 bytes, 120164352 sectors
Disk model:  SanDisk 3.2Gen1
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 8AE83933-1443-47D3-BADB-EFADD54FD659

# create new partition
Command (m for help): n
Partition number (1-128, default 1): 
First sector (2048-120164318, default 2048): 
# create a 10G partition
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-120164318, default 120162303): +10G

Created a new partition 1 of type 'Linux filesystem' and of size 10 GiB.

# write the partition
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

# command exits, so check partitions again to verify
sudo fdisk /dev/sdb

Welcome to fdisk (util-linux 2.39.3).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): p
Disk /dev/sdb: 57.3 GiB, 61524148224 bytes, 120164352 sectors
Disk model:  SanDisk 3.2Gen1
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 8AE83933-1443-47D3-BADB-EFADD54FD659

Device     Start      End  Sectors Size Type
/dev/sdb1   2048 20973567 20971520  10G Linux filesystem

Command (m for help): q

# view partition with lsblk
lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   25G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   23G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:0    0 11.5G  0 lvm  /
sdb                         8:16   1 57.3G  0 disk 
└─sdb1                      8:17   1   10G  0 part # new partition 
sr0                        11:0    1 1024M  0 rom 
```

5. Format the disk with a filesystem with `mkfs`. Use exfat :

```bash
sudo apt install exfatprogs exfat-fuse
```

6. Format the filesystem:

```bash
sudo mkfs.exfat /dev/sdb1 
exfatprogs version : 1.2.2
Creating exFAT filesystem(/dev/sdb1, cluster size=32768)

Writing volume boot record: done
Writing backup volume boot record: done
Fat table creation: done
Allocation bitmap creation: done
Upcase table creation: done
Writing root directory entry: done
Synchronizing...

exFAT format complete!
```

7. Mount the disk drive. This means that you mount the filesystem into a directory on your current filesystem.
   
   There are a few locations that linux suggests that you mount storage:
   - `/mnt`: permanent filesystems that you want available at all times
   - `/media`: temporary storage volume that will not be attached all the time

```bash
# make a new directory in /mnt
ls -l /mnt
total 0
sudo mkdir /mnt/disk1

# mount the disk in the new directory
sudo mount /dev/sdb1 /mnt/disk1/

# verify it was mounted
lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   25G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   23G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:0    0 11.5G  0 lvm  /
sdb                         8:16   1 57.3G  0 disk 
└─sdb1                      8:17   1   10G  0 part /mnt/disk1 # it was mounted
sr0                        11:0    1 1024M  0 rom 

# verify with df
df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              795M  1.1M  794M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   12G  2.7G  8.0G  25% /
tmpfs                              3.9G     0  3.9G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/sda2                          2.0G   95M  1.7G   6% /boot
tmpfs                              795M   12K  795M   1% /run/user/1000
/dev/sdb1                           10G  128K   10G   1% /mnt/disk1 # it was mounted
```

## ncdu

Scan your filesystem and show report on which directories are using the most space:
- Use the arrow and **Enter** key to drill down into directories in the output.

```bash
# run from $HOME
sudo ncdu
ncdu 1.15.1 ~ Use the arrow keys to navigate, press ? for help                                                          
--- /home/ryanseymour --------------------------------------------------------------------------------------------------
   47.0 GiB [##########] /isos                                                                                          
   34.8 GiB [#######   ] /VirtualBox VMs
   32.7 GiB [######    ] /Development
   14.4 GiB [###       ] /Desktop
    8.6 GiB [#         ] /Downloads
    4.9 GiB [#         ] /.local
    4.7 GiB [          ] /.config

# run from root dir
sudo ncdu /
ncdu 1.19 ~ Use the arrow keys to navigate, press ? for help
--- / -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    2.0 GiB [##############################] /usr
  604.8 MiB [########                      ] /var
   94.3 MiB [#                             ] /boot
    6.2 MiB [                              ] /etc
    1.1 MiB [                              ] /run
  436.0 KiB [                              ] /home

# search only the local filesystem (no mounts)
sudo ncdu / -x
```

## fstab

`/etc/fstab` lets you mount storage automatically.

## df

Displays disk usage on all mounted filesystems:

```bash
# display each partition with disk usage and location
$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              795M  1.1M  794M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   12G  2.6G  8.1G  25% /
tmpfs                              3.9G     0  3.9G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/sda2                          2.0G   95M  1.7G   6% /boot
tmpfs                              795M   12K  795M   1% /run/user/1000
```

## dd

[Complete guide to dd](https://blog.kubesimplify.com/the-complete-guide-to-the-dd-command-in-linux)

Disk/data duplicator, also called the "disk destroyer":
- Create backups of entire partitions and hard drives
- Wipe disks


```bash
# backup a partition
dd if=/dev/sdb1 of=/dev/sdb2 status=progress

# overwrite with 0s
dd if=/dev/zero of=/dev/sdb2 status=progress

# overwrite with random chars
dd if=/dev/urandom of=/dev/sdb2 status=progress
```