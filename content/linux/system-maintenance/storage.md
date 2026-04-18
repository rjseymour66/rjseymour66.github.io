+++
title = 'Storage'
date = '2025-09-07T18:49:17-04:00'
weight = 40
draft = false
+++


## Add storage volumes

Before adding storage, determine how much space you need, which filesystem to format it with, and where to mount it. Linux assigns new devices a name from the `/dev/sd[a..z]` or `/dev/vd[a..z]` scheme. For example, a disk named `/dev/sdb` has its first partition at `/dev/sdb1`.


### Find devices

Before partitioning, identify the device name the system assigned to your new disk. The following commands each accomplish this in a different way:

| Command | Description |
| :--- | :--- |
| `fdisk -l` | View detailed info for all attached devices. |
| `lsblk` | List block devices in a tree view. |
| `dmesg \| tail` | View the end of the kernel ring buffer for recently attached devices. |
| `dmesg --follow` | Watch the kernel ring buffer in real time as you attach a new device. |

### Format and partition storage

After you find your device, create a partition with `fdisk`. Linux stores partition configurations in partition tables. For new disks, use GPT. There are two types:

| Type | Description |
| :--- | :--- |
| GUID Partition Table (GPT) | The current standard. Supports up to 128 partitions. |
| Master Boot Record (MBR) | The legacy standard, sometimes called DOS in `fdisk`. Limited to 4 partitions and disks up to 2 TB. |

#### Create a partition

Run `fdisk` on the device and follow the interactive prompts:

```bash
fdisk /dev/sdb          # 1. run fdisk on the device
p                       # 2. verify that you are working on the correct device
g                       # 3. create a GPT partition table
n                       # 4. create a new partition
ENTER                   # 5. accept default partition number
ENTER                   # 6. accept default for first sector of partition
+10G                    # 7. set the partition size (10 gigabytes)
p                       # 8. view changes
w                       # 9. write changes to the partition table
```

#### Format a partition

Run `mkfs` on the partition, not the disk:

```bash
mkfs.ext4 /dev/sdb1
```

#### Delete a partition

Run `fdisk` on the device and select the partition to remove:

```bash
fdisk /dev/sdb          # 1. run fdisk on the device
p                       # 2. verify that you are working on the correct device
d                       # 3. delete command
[1..]                   # 4. select the partition number (if more than one)
```

### fdisk

`fdisk` is the primary tool for managing disk partitions:

```bash
fdisk -l            # view device info
fdisk <device>      # start partitioning on <device>
```

### Partition and format USB

{{< admonition "Example video" "tip" >}}
For a walkthrough of this process, see [Learning Linux TV](https://www.youtube.com/watch?v=2Z6ouBYfZr8&t=183s).
{{< /admonition >}}

1. Locate the disk. Device naming varies, so use the following commands to identify it:

    ```bash
    sudo dmesg | tail       # view kernel messages for the USB
    lsblk                   # list block devices; no path under MOUNTPOINTS means not mounted
    ll /dev/sdb*            # view mountpoint in filesystem
    sudo fdisk -l           # detailed information with fdisk
    ```

2. Verify that your disk is not mounted. View all mounted devices and filter for your device:

    ```bash
    mount | grep <disk>
    ```

3. Unmount the disk if it is mounted:

    ```bash
    sudo umount /path/to/disk
    ```

4. Run `fdisk` to start partitioning:

    ```bash
    sudo fdisk /dev/sdb

    # 1. view existing partitions
    Command (m for help): p
    Disk /dev/sdb: 57.3 GiB, 61524148224 bytes, 120164352 sectors
    ...

    # 2. create new GPT partition table, which wipes the existing one
    Command (m for help): g

    # 3. verify partition table is empty
    Command (m for help): p

    # 4. create new partition
    Command (m for help): n
    Partition number (1-128, default 1):
    First sector (2048-120164318, default 2048):
    Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-120164318, default 120162303): +10G

    Created a new partition 1 of type 'Linux filesystem' and of size 10 GiB.

    # 5. write the partition
    Command (m for help): w

    # 6. verify the new partition
    lsblk
    ...
    sdb                         8:16   1 57.3G  0 disk
    └─sdb1                      8:17   1   10G  0 part # new partition
    ...
    ```

5. Install the exFAT tools and format the partition:

    ```bash
    sudo apt install exfatprogs exfat-fuse
    sudo mkfs.exfat /dev/sdb1
    ```

6. Mount the disk. Linux recommends `/mnt` for permanent storage and `/media` for removable storage:

    ```bash
    sudo mkdir /mnt/disk1
    sudo mount /dev/sdb1 /mnt/disk1/

    # verify with lsblk
    lsblk
    ...
    sdb                         8:16   1 57.3G  0 disk
    └─sdb1                      8:17   1   10G  0 part /mnt/disk1
    ...

    # verify with df
    df -h
    ...
    /dev/sdb1                           10G  128K   10G   1% /mnt/disk1
    ```

## Mount and unmount volumes

Mounting a volume attaches a storage device or network share to a local directory on your server. The local directory must be empty. A volume is another term for a filesystem or partition. The Filesystem Hierarchy Standard recommends two locations for mountpoints:

- `/mnt` for permanent storage you plan to use regularly, such as hard drives or network-attached storage.
- `/media` for removable media such as flash drives and external hard drives. For example, mount a USB drive at `/media/usb1/`.

To mount and unmount a volume:

1. List all currently mounted filesystems to see what is already attached:

    ```bash
    mount
    ```

2. Create an empty directory for the mountpoint and attach the device to it:

    ```bash
    mkdir /media/usb1                       # create empty directory for the mountpoint
    mount /dev/sdb1 /media/usb1             # mount volume to the empty directory
    mount /dev/sdb1 -t ext4 /media/usb1     # specify filesystem type if you get an error
    df -h                                   # confirm volume was mounted correctly
    ```

3. When finished, unmount the volume. It must not be in use:

    ```bash
    umount /media/usb1                      # note: no "n"
    ```


## /etc/fstab

At boot, the system reads `/etc/fstab` to mount the main filesystem, swap, and any other configured volumes. Each partition is identified by a universally unique identifier (UUID). You can reference a partition by name such as `/dev/sdb1`, but UUID is preferred because partition names can change depending on where a device is mounted or the order in which devices are attached. A UUID only changes if you reformat the volume.

### Column descriptions

| Column no. | Description                                                                                                                                             |
| :--------- | :------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1          | UUID or label for the device                                                                                                                            |
| 2          | fs location where you want to mount the device                                                                                                          |
| 3          | fs type                                                                                                                                                 |
| 4          | options for each mount. `defaults` is a good default. Ex: `errors=remount-ro` for root to mount in read only mode if error occurs                       |
| 5          | Determines whether the the fs should be backed up. `0` for no, `1` for yes. Rarely used nowadays.                                                       |
| 6          | Order that `fsck` checks the filesystems. Recommend using `1` for main fs and `2` for all others. Some cloud providers set additional disks to `0` too. |

The `defaults` option includes the following settings:

- `rw`: Device is mounted as read/write.
- `exec`: Allow files on the volume to be executed as scripts.
- `auto`: Automatically mount at boot or when running `sudo mount -a`.
- `nouser`: Only `root` can mount the filesystem.
- `async`: Write output to the device asynchronously.

Other commonly used options:

- `ro`: Device is mounted as read-only.
- `noexec`: Prevent executable scripts from being stored on the volume.
- `noauto`: Require manual mounting. Cannot mount at boot or with `sudo mount -a`. Useful for backup drives that do not need to be attached all the time.
- `users`: Allow regular users to mount the filesystem without `sudo`.

### Adding volume to /etc/fstab

Adding a volume to `/etc/fstab` makes it mount automatically at boot. To add a volume:

1. Find the UUID of the volume:

    ```bash
    blkid /dev/sdb1
    ```

2. Create the mountpoint directory:

    ```bash
    mkdir /mnt/extra_storage_fstab
    ```

3. Add an entry to `/etc/fstab`:

    ```bash
    # Extra storage
    UUID=8609c662-378d-4df0-be90-f35d1e1fdccc /mnt/extra_storage_fstab ext4 defaults 0 0
    ```

4. Mount all entries in `/etc/fstab` without rebooting:

    ```bash
    sudo mount -a
    ```

5. Verify the volume was mounted:

    ```bash
    df -h
    ```

To configure a volume for manual mounting only, set `noauto` in the options column. Then mount it by specifying only the destination path:

```bash
UUID=8609c662-378d-4df0-be90-f35d1e1fdccc /mnt/ext_disk ext4 rw,noauto 0 0

mount /mnt/ext_disk
```

### blkid

`blkid` lists the UUIDs of all volumes known to the system. It may require `root` privileges:

```bash
blkid                   # list all UUIDs
blkid <partition>       # list UUID for a specific partition
```

### Disk quotas

Disk quotas let you limit the number of files a user can create and restrict the total filesystem space available to them, preventing users from filling up the disk. For a full walkthrough, see the [Linode guide to file system quotas](https://www.linode.com/docs/guides/file-system-quotas/).

Setting up quotas requires four steps:

1. Modify `/etc/fstab` to enable filesystem quotas.
2. Mount the filesystem. If it was already mounted, unmount and remount it.
3. Create the quota file.
4. Establish user or group quota limits and grace periods.

## RAID

{{< admonition "RAID is not a backup solution" "warning" >}}
RAID does not protect against data loss from damaged or stolen disks. Always maintain separate backups.
{{< /admonition >}}

RAID joins multiple disks in a variety of configurations to prevent data loss. Different RAID levels determine how many disks can fail before data is lost.

| Level | Description |
| :--- | :--- |
| RAID 1 | Two disks mirror each other. One can fail and the array can be rebuilt. |
| RAID 5 | Distributes data and parity across three or more disks. One disk can fail. |
| RAID 6 | Like RAID 5 but with two parity blocks. Two disks can fail. |

## Backups

Backups duplicate your data so you can recover from loss or corruption. Store them offsite or in a different physical location from your primary data. Test your backups regularly to confirm they can be restored quickly. A robust strategy has three layers:

- Local storage such as NAS
- Cloud storage
- Offsite or mirrored data

Encrypt your backups wherever they are stored.

## LVM

Linux Volume Manager (LVM) lets you resize filesystems and partitions without rebooting. For virtual servers, always use LVM on storage volumes when possible. Cloud disks can be grown without a reboot, but attaching a physical disk to a physical server typically requires one. Adding or removing hardware components to a running server is called hot-plugging.


### Terminology

Volume group
: Namespace that includes all physical and logical volumes for an LVM implementation. By convention, name volume groups with the `vg-<volume-name>` prefix.

Physical volume
: Physical or virtual hard disk that is a member of a volume group.

Logical volume
: A flexible, resizable partition that acts as a single unit whose size can span multiple physical volumes.

### Implementation

Create a volume group, assign physical disk space to that group, then divide it into logical volumes. LVM expects a block device with no existing partitions or partition signatures. Run `sudo wipefs -a /dev/sdb` to remove partition signatures before starting.

When adding physical volumes to a volume group with `vgextend`, do not assign them to a logical volume until the space is needed. View logical volumes, physical volumes, and volume groups with the `lvdisplay`, `pvdisplay`, and `vgdisplay` commands respectively.

1. Install the LVM package:

    ```bash
    apt install lvm2
    ```

2. Monitor logical volume size:

    ```bash
    lvs
    ```

3. Set up a logical volume by designating a physical volume, creating a volume group, and creating the logical volume:

    ```bash
    pvcreate /dev/sdb                               # 1. designate disk as a physical volume
    pvdisplay                                       # 2. confirm physical volume
    vgcreate vg-test /dev/sdb                       # 3. create volume group and assign disk
    vgdisplay                                       # 4. confirm volume group
    lvcreate -n myvol1 -L 5g vg-test                # 5. create 5G logical volume in vg-test
    lvdisplay                                       # 6. confirm logical volume
    ```

4. Format the logical volume, create a mountpoint, and mount it:

    ```bash
    lvdisplay                                       # 1. get device name (LV Path)
    mkfs.ext4 /dev/vg-test/myvol1                   # 2. format logical volume
    mkdir -p /mnt/lvm/myvol1                        # 3. create empty directory for mountpoint
    mount /dev/vg-test/myvol1 /mnt/lvm/myvol1/      # 4. mount the logical volume
    df -h                                           # 5. verify volume was mounted
    ```

5. Resize the logical volume to consume all remaining space in the volume group:

    ```bash
    lvextend -n /dev/vg-test/myvol1 -l +100%FREE    # 1. extend LV to remaining PV space
    df -h                                           # 2. get filesystem name on LV
    resize2fs /dev/mapper/vg--test-myvol1           # 3. resize filesystem to span entire LV
    ```

6. Add physical volumes to the volume group to expand available space:

    ```bash
    vgextend vg-test /dev/sdc                       # 1. add PV to volume group
    lvextend -L+10g /dev/vg-test/myvol1             # 2. add 10G from new PV to logical volume
    resize2fs /dev/vg-test/myvol1                   # 3. resize filesystem to span entire LV
    ```

7. Remove logical volumes, the volume group, and clean up:

    ```bash
    umount /mnt/lvm/myvol1                          # 1. unmount the logical volume
    lvremove vg-test/myvol1                         # 2. remove LV from VG
    vgremove vg-test                                # 3. remove the volume group
    ```

### LVM Snapshots

{{< admonition "Snapshots are not backups" "warning" >}}
Snapshots are stored in the same volume group as the original data. They can become corrupt if disk space runs low and do not protect against disk failure or theft.
{{< /admonition >}}

An LVM snapshot captures a logical volume at a specific point in time. It is a clone of the original logical volume and requires unallocated space in the volume group. You can mount a snapshot or roll back to it if something goes wrong.

A common use case is taking a snapshot of the root filesystem before installing updates. If the updates cause problems, roll back to the snapshot. If everything is fine, delete it.

The following examples show how to create, roll back to, and delete a snapshot:

```bash
lvcreate -s -n mysnapshot -L 4g vg-test/myvol1  # create snapshot of LV myvol1
lvconvert --merge vg-test/mysnapshot             # roll back to snapshot (must unmount first)
lvremove vg-test/mysnapshot                      # delete the snapshot
```

## wipefs

`wipefs` removes partition signatures from a disk. Run it before designating a disk as an LVM physical volume:

```bash
wipefs -a /dev/sdb
```

## dd

`dd` is the disk and data duplicator, sometimes called the "disk destroyer" for its ability to overwrite data irreversibly. Use it to back up entire partitions or wipe disks.

For a full reference, see the [complete guide to dd](https://blog.kubesimplify.com/the-complete-guide-to-the-dd-command-in-linux).


The following examples show how to back up a partition and wipe a disk:

```bash
# backup a partition
dd if=/dev/sdb1 of=/dev/sdb2 status=progress

# overwrite with 0s
dd if=/dev/zero of=/dev/sdb2 status=progress

# overwrite with random chars
dd if=/dev/urandom of=/dev/sdb2 status=progress
```