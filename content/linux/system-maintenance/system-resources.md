+++
title = 'System Resources'
date = '2025-09-07T18:48:56-04:00'
weight = 70
draft = false
+++


## Disk space

When a disk fills up, start with `df -h` to identify the affected volume, then drill down with `du` and `ncdu` to find what is consuming space.

1. Run `df -h` to identify which volume is running low.
2. Run `du -hsc *` in the affected volume to find the directory consuming the most space.
3. Run `ncdu -x` for an interactive breakdown of that directory.

### df

`df` (disk free) shows file system usage across all mounted volumes. By default it reports sizes in bytes. Pass `-h` for human-readable output. Device names reflect the underlying storage hardware. For example, `sda` for SATA/SCSI disks and `nvme0n1` for NVMe drives.

Disk space is not the only resource to monitor. Each file also consumes an inode, a data structure the kernel uses to store file metadata: owner, permissions, size, and timestamps. A system can run out of inodes before running out of disk space, typically when a process generates large numbers of small files such as logs or mail messages:

```bash
df -h               # list disk space in human-readable format
df -i               # list inode usage
```

### ncdu

`ncdu` (NCurses Disk Usage) provides an interactive view of which directories consume the most space. It can only scan directories the current user has permission to read. Navigate the output with the arrow keys and press **Enter** to drill into a directory.

Install it before first use:

```bash
sudo apt install ncdu               # install ncdu
ncdu -x                             # scan the current filesystem only
```

## Disk usage by directory

### du

`du` shows how much space a directory and its subdirectories are consuming. It scans only directories the current user can read. Run it as `root` for a complete picture. Once you identify the problem directory, `cd` into it and run `du` again to narrow down further:

```bash
du -hsc *       # human-readable, summary, total usage
```

## Memory usage

### free

`free` reports current memory usage. To assess memory pressure, compare the `available` column against `total`. When `available` drops close to zero, the system is under memory pressure.

The `free` column shows memory not in use by anything. The `available` column is more useful. It includes memory currently held in the kernel cache that the system can reclaim immediately if an application needs it. This is by design: idle RAM is used as a filesystem cache to reduce disk reads, which makes the system faster.

`tmpfs` entries in the output represent temporary filesystems stored in RAM rather than on disk.

| Column     | Description                                                                     |
| :--------- | :------------------------------------------------------------------------------ |
| total      | Total installed memory                                                          |
| used       | Memory in use. Calculated as: total - free - buff/cache                         |
| free       | Memory not in use by anything                                                   |
| shared     | Memory used by `tmpfs` and other shared resources                               |
| buff/cache | Memory used by kernel buffers and the filesystem cache                          |
| available  | Memory available for application use, including reclaimable cache               |

```bash
free                # memory usage in KB
free -m             # memory usage in MB (recommended)

# --- Example to understand columns --- #
free -m
               total        used        free      shared  buff/cache   available
Mem:            3915         533        2474           1        1198        3381
Swap:           2335           0        2335
```

## Swap

Swap is a disk partition or file the kernel uses as overflow when physical memory is full. Because it lives on disk, it is much slower than RAM, but it prevents the out-of-memory (OOM) killer from terminating processes under pressure. Modern Ubuntu systems use a swap file rather than a dedicated partition, which is easier to resize. A minimum of 2 GB is recommended on most servers.

Some applications, such as Kubernetes, require swap to be disabled.

Swap is listed in `/etc/fstab`. Only delete the swap file if you are replacing it with a larger one.

`swappiness` controls how aggressively the kernel moves memory pages to swap. It defaults to `60`. Higher values cause the kernel to swap more frequently. To persist a change across reboots, set the value in `/etc/sysctl.conf`.

These commands manage swap at runtime and inspect swappiness:

```bash
grep swap /etc/fstab            # confirm swap entry in /etc/fstab
/swap.img	none	swap	sw	0	0

swapon -a                       # activate all swap listed in /etc/fstab
swapoff -a                      # deactivate all swap
cat /proc/sys/vm/swappiness     # view current swappiness value
sysctl vm.swappiness=30         # change swappiness until next reboot
```

### Create a swap file

A new swap file must be allocated, formatted, registered in `/etc/fstab`, and activated.

1. Create the file at the desired size.

   ```bash
   fallocate -l 2G /swapfile
   ```

2. Restrict access to root only.

   ```bash
   chmod 0600 /swapfile
   ```

3. Format the file as swap.

   ```bash
   mkswap /swapfile
   ```

4. Add an entry to `/etc/fstab` so the swap persists across reboots.

   ```bash
   /swapfile   none    swap    0   0
   ```

5. Activate the new swap file.

   ```bash
   swapon -a
   ```

To change swappiness and persist it after reboot, add or update the following line in `/etc/sysctl.conf`:

```bash
vm.swappiness = 30
```

### fallocate

`fallocate` creates a file with a preallocated size. Pass `-l` with a size value such as `2G` or `500M`:

```bash
fallocate -l <size> <filename>
fallocate -l 4G /swapfile
```

## Load average

Load average tracks CPU demand over time as three values: the 1-minute, 5-minute, and 15-minute averages. Each value represents the average number of tasks waiting for CPU time in that window. The raw data is in `/proc/loadavg`, but `uptime` displays it in a more readable format.

To interpret the values, compare them against the number of logical CPU cores on the system (get this with `nproc`). A load average equal to the core count means all cores are fully utilized. A value above the core count means tasks are queuing for CPU time and the system is overloaded.

Load average gives a better picture of CPU pressure than a point-in-time tool like `htop`, because a brief spike to 100% may not reflect sustained demand. Establish a baseline for your system under normal workloads so you can recognize when the trend changes:

```bash
cat /proc/loadavg                   # view load average in /proc
0.00 0.00 0.00 1/234 14112

uptime                              # view load average, resets at reboot
nproc                               # get the number of logical CPU cores
```

## View resource usage

### htop

`htop` provides an interactive view of system performance and is more capable than `top`. Run it as `root` to enable process management actions such as killing processes. You can navigate with the mouse or keyboard.

To add a CPU average meter for all cores, open **F2** (Setup), go to **Meters**, and select CPU average:

```bash
F2                  # open setup to configure display and colors
u                   # filter processes by user
F5                  # toggle tree view
F9                  # kill the selected process (prompts for signal)
htop -d 70          # refresh every 7 seconds (-d value is in tenths of a second)
```

## Devices

### lsusb

`lsusb` lists all USB devices currently attached to the system:

```bash
lsusb
Bus 004 Device 033: ID 0bda:8153 Realtek Semiconductor Corp. RTL8153 Gigabit Ethernet Adapter
Bus 004 Device 032: ID 2109:0817 VIA Labs, Inc. USB3.0 Hub             
Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
...
```

### lshw

`lshw` lists detailed hardware information for all devices on the system. Filter by class to narrow the output:

```bash
lshw -class network
  *-network                 
       description: Ethernet interface
       product: Wi-Fi 6 AX200
       ...
  *-network
       description: Ethernet interface
       physical id: e
       ...

lshw -html > lshw-output.html      # export full hardware report as HTML
lshw -c memory                     # list memory devices
lshw -c storage                    # list storage devices
lshw -c multimedia                 # list multimedia devices
lshw -c cpu                        # list CPU information
```