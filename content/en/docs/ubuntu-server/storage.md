---
title: "Managing storage volumes"
linkTitle: "Storage"
weight: 80
# description:
---

## Add storage volumes

Things to consider before you add storage:
- How much storage do you need
- What name did the system assign the new device? `/dev/sd[a..z]`, `/dev/vd[a..z]`
  - For example, the disk/device is named `/dev/sdb` and the first partition is named `/dev/sdb1`
- How to format the disk - what filesystem?
- Where to mount the disk


### Find devices

```bash
# --- Options to locate attached devices ---#
fdisk -l            # view device info
dmesg --follow      # watch kernel ring buffer as you attach new device
dmesg | tail        # view end of kernel ring buffer for new device
lsblk               # list block devices
```

### Format and partition storage

After you find your device, create a partition with `fdisk`:
- Linux stores partition configurations in partition tables. There are two types of partition tables:
  - Master Boot Record (MBR): Legacy version
    - `fdisk` might refer to this as DOS (Disk Operating System)
    - Only lets you create up to 4 partitions
    - Can only partition up to 2TB of disk
  - GUID Partition Table (GPT): New standard
    - Create up to 128 partitions

```bash
# --- Create partition on disk --- #
fdisk /dev/sdb          # 1. Run fdisk on the device
p                       # 2. Verify that you are working on the correct device
g                       # 3. Create a GPT partition table
n                       # 4. Create a new partition
ENTER                   # 5. Accept default partition number
ENTER                   # 6. Accept default for first sector of partition
+10G                    # 7. Set the partition size - this is 10 gigabytes (set the last partition sector)
p                       # 8. View changes
w                       # 9. Write changes to the partition table

# --- Format new partition --- #
mkfs.ext4 /dev/sdb1

# --- Delete a partition --- #
fdisk /dev/sdb          # 1. Run fdisk on the device
p                       # 2. Verify that you are working on the correct device
d                       # 3. Delete command
[1..]                   # 4. Select the partition number (if more than one)
```

### fdisk

```bash

fdisk -l            # view device info
fdisk <device>      # start partitioning on <device>
```

## Mount and unmount volumes

When you mount a volume, you attach a storage device or network share to a local directory on your server:
- Local dir must be empty.
- Volume is another way to say "filesystem" or "partition"
- Filesystem Hierarchy Standard recommends these dirs for mountpoints:
  - `/mnt` for mounted fs you plan to use regularly - hard drives, network-attached storage
  - `/media`: removable media - flash drives, external hard drives
  - To mount a USB drive, create a `usb1/` dir. Ex:  `/media/usb1/`

```bash
# --- Mount a volume --- #
mkdir /media/usb1                       # 1. Create empty dir to mount
df -h                                   # 2. View currently mounted filesystems
mount /dev/sdb1 /media/usb1             # 3. Mount volume to new empty directory
mount /dev/sdb1 -t ext4 /media/usb1     #    If there is an error about fs type
df -h                                   # 4. Confirm volume was mounted correctly

# --- Unmount a volume --- #
umount /media/usb1                      # 1. Unmount (no "n") to disconnect the volume from the fs
                                        #    Volume cannot be in use, or returns an error
```

### mount

Attaches a storage device or network share to a local directory on your server:

```bash
mount           # list all mounted filesystems
```


## /etc/fstab

At boot, the system looks at this file to mount your main fs, swap, and any other file:
- Each partition is ID'd with a universally unique identifier (UUID)
  - Can list regular partition name--`/dev/sdb1`-- but UUID is preferred bc partiton names change depending on where you mount them or the order in which they are attached to the system
  - UUID does not change unless you reformat your volume

### Column descriptions

| Column no. | Description |
|:--|:--|
| 1 | UUID or label for the device |
| 2 | fs location where you want to mount the device |
| 3 | fs type |
| 4 | options for each mount. `defaults` is a good default. Ex: `errors=remount-ro` for root to mount in read only mode if error occurs |
| 5 | Determines whether the the fs should be backed up. `0` for no, `1` for yes. Rarely used nowadays. |
| 6 | Order that `fsck` checks the filesystems. Recommend using `1` for main fs and `2` for all others. Some cloud providers set additional disks to `0` too. |

Options included in `defaults`:
- `rw`: Device is mounted as read/write
- `exec`: Allow files on volume to be executed as scripts
- `auto`: Automatically mount device at boot time or when you run `sudo mount -a`
- `nouser`: Only `root` can mount the fs
- `async`: Output to the device is asynchronous

Other options:
- `ro`: Device is mounted as read-only
- `noexec`: Prevent storing executable scripts on mounted volume
- `noauto`: Must mount manually. Cannot mount at boot or with `sudo mount -a`. This is good for drives used for backups that don't need to be mounted all the time.
- `users`: Give regular users access without `sudo` or using `root` privileges

### Adding volume to /etc/fstab

```bash
blkid /dev/sdb1                     # 1. Find UUID of volume
mkdir /mnt/extra_storage_fstab      # 2. Create mountpoint dir

# --------------------------------- # 3. Add info to /etc/fstag
# Extra storage
UUID=8609c662-378d-4df0-be90-f35d1e1fdccc /mnt/extra_storage_fstab ext4 defaults 0 0

sudo mount -a                       # 4. Mount without reboot
df -h                               # 5. Verify mounted volume

# --- Manually mount w/fstab --- #
# read/write, manually mount
UUID=8609c662-378d-4df0-be90-f35d1e1fdccc /mnt/ext_disk ext4 rw,noauto 0 0

mount /mtn/ext_disk                  # just list dest path for mount
```

### blkid

List UUIDs of all volumes known to your system:
- might have to run as `root`

```bash
blkid                   # list all blkid
blkid <partition>       # list UUID for <partition>
```

## RAID

Lets you join multiple disks in a variety of configurations to prevent data loss:
- Has different levels, where the higher the level, the more disks that can fail before you have a data loss problem
- RAID level 1: two hard disks always have the same data, one can fail
  - If a disk fails, you can rebuild the array
- RAID level 5: same as level 1 but more disks (?)
- RAID level 6: two disks can fail before 
- NOT A BACKUP SOLUTION - you can still lose data if disks are fried or stolen

## Backups

Backups duplicate data:
- Should be offsite or not in the same physical location as your data
- Should be resilient and let you get your data back up and running quickly
  - You should test your backups regularly
- Have three layers - NAS, cloud, offsiet, mirroring data
- Consider encrypting data, wherever it is

## LVM

Linux Volume Manager.

You can resize your filesystem/partitions without rebooting or shutting down your server:
- ALWAYS use LVM on storage volumes for a virtual server, when possible
- For virtual (cloud) disks, you can grow a filesystem without a server reboot
  - Likely require reboot if you need to attach physical disk to physical server
- _hot-plugging_ is adding or removing hardware components to a running server without having to shut it down or reboot it


### Terminology

Volume group
: Namespace that includes all physical and logical volumes for an implementation of LVM.
  - Use a common naming convention, such as `vg-<volume-name>`.

Physical volume
: Physical or virtual hard disk that is a member of a volume group.

Logical volume
: A flexible, resizable partition that acts as a single unit whose size can span multiple physical volumes.

### Implementation

Create a volume group, assign physical disk space to that group, then split up the physical volumes into logical volumes:
- LVM expects a block device, so make sure there are no partitions or partition signatures on the disk
  - `sudo wipefs -a /dev/sdb` deletes the partition signatures
- When you add physical volumes to a volume group with `vgextend`, do not assign the PVs to a logical volume until the space is needed
- View LVs, PVs, and VGs with `xxdisplay` command. For example, `lvdisplay`, `pvdisplay`, `vgdisplay`

```bash
apt install lvm2                    # Install package
lvs                                 # Monitor logical volume size
# --- Set up Logical Volume --- #

pvcreate /dev/sdb                               # 1. Designate each disk as a physical volume
pvdisplay                                       # 2. View phyical volumes and confirm changes
vgcreate vg-test /dev/sdb                       # 3. Create the volume group and assign a physical disk
vgdisplay                                       # 4. View volume groups and confirm changes
lvcreate -n myvol1 -L 5g vg-test                # 5. Create 5G logical volume named myvol1 and assign to vg-test
lvdisplay                                       # 6. View logical volumes and confirm changes

# --- Format Logical Volume --- #

lvdisplay                                       # 1. Get device name (LV Path)
mkfs.ext4 /dev/vg-test/myvol1                   # 2. Format logical volume
mkdir -p /mnt/lvm/myvol1                        # 3. Create empty dir for mountpoint
mount /dev/vg-test/myvol1 /mnt/lvm/myvol1/      # 4. Mount the logical volume
df -h                                           # 5. Verify volume was mounted

# --- Resizing Logical Volume --- #

lvextend -n /dev/vg-test/myvol1 -l +100%FREE    # 1. Extend LV to take up remaining PV disk space
df -h                                           # 2. Get name of filesystem on LV
resize2fs /dev/mapper/vg--test-myvol1           # 3. Resize filesystem to span entire LV

# --- Add PVs to volume group --- #

vgextend vg-test /dev/sdc                       # 1. Add PV to volume group
lvextend -L+10g /dev/vg-test/myvol1             # 2. Add 10G from the new PV to the logical volume
resize2fs /dev/vg-test/myvol1                   # 3. Resize the filesystem to span entire LV

# --- Removing LVMs --- #

umount /mnt/lvm/myvol1                          # 1. Unmount the logical volume
lvremove vg-test/myvol1                         # 2. Remove LV myvol1 from VG vg-test
vgremove vg-test                                # 3. Remove the volume group
```

### LVM Snapshots

Captures a logical volume at a specific point in time and preserves it:
- You can mount a snapshot or use it to revert the snapshot in case of failure
  - Ex: you want to test how security updates will affect your server and might need to revert back
  - NOT A FORM OF BACKUP - not stored in different location, can become corrupt if disk space gets low
- Requires unallocated space in volume group
- Is a clone of the original LV
- Use case: Take a snapshot of the root filesystem before installing all available updates. Rollback if there is an issue, or delete if everything is OK.

```bash
lvcreate -s -n mysnapshot -L 4g vg-test/myvol1  # Create snapshot mysnapshot of LV myvol1
lvconvert --merge vg-test/mysnapshot            # Roll back to mysnapshot (must unmount first)
lvremove vg-test/mysnapshot                     # Delete the snapshot
```

## wipefs

Deletes partition signatures from a disk:

```bash
wipefs -a /dev/sdb
```