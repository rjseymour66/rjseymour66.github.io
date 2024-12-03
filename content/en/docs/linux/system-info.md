---
title: "System information"
linkTitle: "System info"
# weight: 1000
# description:
---

## Distro info

```bash
lsb_release -a

cat /etc/os-release

uname -a

# kernel version
uname -r
```

## Who are you?

```bash
# who you are logged in as
whoami
linuxuser

# who is logged into the system
who
linuxuser tty1         2024-11-30 15:11
linuxuser pts/0        2024-11-30 15:11 (192.168.56.1)

# what logged in users are doing
w
 17:04:00 up  3:32,  2 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU  WHAT
linuxuse tty1     -                Sat15   25:52m  0.08s  0.03s -bash
linuxuse          192.168.20.10    Sat15   25:53m  0.00s  0.03s sshd: linuxuser [priv]

```

## How long system running

```bash
uptime
 04:02:12 up 45 min,  2 users,  load average: 0.00, 0.00, 0.00
```

## Hardware info

```bash
lscpu   # CPU architecture
lsblk   # block devices
lspci   # PCI devices
lsusb   # Attached USB devices
```

## Memory and CPU usage

### free

Parses the `/proc/meminfo` file and shows total available memory:

```bash
# shared - tmpfs to maintain pseudo fs like /sys and /dev
# buff/cache - kernel block I/O 
free -h
               total        used        free      shared  buff/cache   available
Mem:           7.8Gi       631Mi       7.0Gi       3.8Mi       373Mi       7.1Gi
Swap:             0B          0B          0B

# available column is free memory without swapping (in Mi)
free -m
               total        used        free      shared  buff/cache   available
Mem:            7942         714        6593           3         925        7227
Swap:              0           0           0

```

### vmstat

Checks swap usage. You can run it over an extended period of time. This gives you 4 readings at 30-second intervals:

```bash
# si - swapped into memory
# so - swapped out of memory
vmstat 30 4
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 4996868 1537492 18966428    0    0    18    61   23    1  4  2 95  0  0
 0  0      0 5006424 1537504 18956112    0    0     0    61 6112 7490  4  4 92  0  0
```

## Processes

### top

### htop

## Disk usage

### df

Displays disk usage by partition. Use the -i option to see the number of inodes left in the fs. ext3 and ext4 filesystems allocate a specific number of inodes, and you can’t make more files when you run out of inodes.

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
# -d - directory depth
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