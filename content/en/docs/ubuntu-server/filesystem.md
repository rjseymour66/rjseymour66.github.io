---
title: "Navigating the filesystem"
linkTitle: "xFilesystem"
weight: 30
# description:
---

The Linux filesystem directory structure follows the Filesystem Hierarchy Standard (FHS), where every directory has a specific purpose.

## Common directories

You can also run `man hier` to get a detailed description of the filesystem:

| Directory | Description                                                           |
| :-------- | :-------------------------------------------------------------------- |
| /         | Beginning of the fs                                                   |
| /etc      | System-wide application configuration                                 |
| /home     | User home directories                                                 |
| /root     | Home directory for `root`                                             |
| /media    | Removable media, i.e. flash drives                                    |
| /mnt      | Volumes that will be mounted for a while                              |
| /opt      | Additional software packages                                          |
| /bin      | Essential user libraries such as `cp`, `ls`, etc...                   |
| /proc     | Virtual filesystem for OS-level components such as running processes  |
| /usr/bin  | Common user commands that are not needed to boot or repair the system |
| /usr/lib  | Object libraries                                                      |
| /var/log  | Log files                                                             |


## Distro info

```bash
# --- Get distro info --- #
cat /etc/os-release 
PRETTY_NAME="Ubuntu 24.04.1 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.1 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo

# --- Get shorter distro info --- #
lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 24.04.1 LTS
Release:	24.04
Codename:	noble
```

## Viewing log files

`/var/log/syslog` - Contains information about processes happening in the background as your server runs, including warnings and errors.
- If you have an error, look in `syslog` and then google for a resolution

```bash
head -n 100 /var/log/syslog             # view first 100 lines
head -100 /var/log/syslog

tail -n 100 /var/log/syslog             # view last 100 lines
tail -100 /var/log/syslog

tail -f /var/log/syslog                 # watch syslog in real time
```

## View supported filesystems

```bash
cat /proc/filesystems
nodev	sysfs
nodev	tmpfs
...
nodev	devpts
	ext3
	ext2
	ext4
	squashfs
	vfat
...
```

## Elements of a file system

A filesystem is an organization of data and metadata on a storage device.
- Linux filesystems use a common API that lets the kernel work with any filesystem type (ext4) on any storage medium (SATA).
- The linux filesystem is implemented in layers to separate the user interface from the implementation, from the drivers that work with the storage devices.
- ext4 creates one inode for each 16KB of fs capacity

### Mounting

You mount a filesystem that is on a partition, not a hard drive or a partition. The file system is an abstraction on top of the data stored on the hard drive's partition.

When you mount a filesystem, you attach a separate filesystem to the current filesystem hierarchy.

The `losetup` command associates a loop device with a file, which makes it appear to the system as if it is a block device:

```bash
losetup /dev/loop0 file.img
```
After you make the file look like a block device, you can format it with a filesystem.

### High-level architecture

- GNU C lib (libc) is in user space and provides the user interface for the system calls (open, read, write, close), and the system calls send any calls from user space to the appropriate areas (endpoints) in kernel space.
- The Virtual File System (VFS) takes system calls and abstracts them—makes them compatible—to the underlying filesystem. Each filesystem has an inode cache and a dentry (directory entry?) cache that stores recently-used fs objects.
- Each filesystem attached to the system has a set of interfaces that the VFS expects, and it has a buffer cache that stores recent requests so that the system can access the data without going all the way back to the physical device, but you can flush the cache with the `sync` command so that all data unwritten to disk goes to the device driver and then the device.

### Major structures

superblock
: describes and maintains state of the fs

inode
: an inode for each fs object, it contains metadata to manage the object

dentries
: uses the directory cache to map names to inodes, and maintains relationships between dirs and files so you can traverse the fs

files
: VFS maintains the state of open files such as the write offset

### VFS layer

The VFS tracks the currently supported fss and fss that are currently mounted
- You can see registered filesystems in the /proc/filesystems fs, and an fs is registered with the register_filesystem struct function.
- The VFS also tracks the mounted fss with the vfsmount struct

#### Superblock

Structure that is typically stored on the storage medium and stores all the information about the file system, including:

- fs name (ext4)
- size
- reference to the block device
- metadata

The superblock includes the superblock operations, which is a struct that holds the functions to manage inodes, which satisfies the interface to the VFS layer.

### inode and dentry

An inode is an object in the fs with a unique identifier that has methods to work on the inode and methods for system calls that work with files and dirs.
- There is an inode cache and a dentry cache that stores the most recently used inodes and dentries, and each inode in the cache has a corresponding dentry in the cache.


### Buffer cache

Located at the bottom of the fs layer, and it tracks read and write reqs from the individual filesystems and the devices (through the device drivers), and it stores the most recently used pages so the data can be sent to the requesting filesystem quickly.

## inodes

A file is just a filename and an inode number. The inode points to file data on disk.
- A directory struct contains a file's inode, name, and length of name

The inode is in the VFS between the filename and the files data on disk
- Every file points to an inode, and every inode points to a disk block that contains the associated data on disk
- Hard links are when two files have the same inode and point to the same data on disk
- inodes track perms, ownership, and dates metadata
- You cannot add more inodes after installing the fs

```bash
# -i option to view inode
ls -i greptest.txt 
131551 greptest.txt

# stat to view file metadate, including inode
stat greptest.txt 
  File: greptest.txt
  Size: 82        	Blocks: 8          IO Block: 4096   regular file
Device: 252,0	Inode: 131551      Links: 1
Access: (0664/-rw-rw-r--)  Uid: ( 1000/linuxuser)   Gid: ( 1000/linuxuser)
Access: 2024-12-13 11:25:45.069716490 -0500
Modify: 2024-12-13 11:25:39.722707145 -0500
Change: 2024-12-14 16:59:53.434398357 -0500
 Birth: 2024-12-13 11:25:39.721707144 -0500

# view available inodes on system
df -hi
Filesystem                        Inodes IUsed IFree IUse% Mounted on
tmpfs                               993K   823  992K    1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   736K  139K  598K   19% /
tmpfs                               993K     1  993K    1% /dev/shm
tmpfs                               993K     4  993K    1% /run/lock
/dev/sda2                           128K   321  128K    1% /boot
tmpfs                               199K    32  199K    1% /run/user/1000

# inodes on a partition
df -i /dev/mapper/ubuntu--vg-ubuntu--lv 
Filesystem                        Inodes  IUsed  IFree IUse% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv 753664 141940 611724   19% /

# get system block size
blockdev --getbsz /dev/mapper/ubuntu--vg-ubuntu--lv
4096

# check block size usage
df -B 4096 /dev/mapper/ubuntu--vg-ubuntu--lv 
Filesystem                        4K-blocks   Used Available Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   2939690 892182   1892731  33% /

# does 
debugfs -R "stat <inode>" <filesystem>
debugfs -R "stat <263700>" /dev/mapper/ubuntu--vg-ubuntu--lv
Inode: 263700   Type: regular    Mode:  0644   Flags: 0x80000
Generation: 1397416079    Version: 0x00000000:00000001
User:  1000   Group:  1000   Project:     0   Size: 18
File ACL: 0
Links: 1   Blockcount: 8
Fragment:  Address: 0    Number: 0    Size: 0
 ctime: 0x67644365:e3260cc4 -- Thu Dec 19 11:01:41 2024
 atime: 0x67644365:e3260cc4 -- Thu Dec 19 11:01:41 2024
 mtime: 0x67644365:e3260cc4 -- Thu Dec 19 11:01:41 2024
crtime: 0x67644365:e3260cc4 -- Thu Dec 19 11:01:41 2024
Size of extra inode fields: 32
Inode checksum: 0x0a68b9d2
EXTENTS:
(0):2189145                                                     # disk block offset
```
## Pinpoint commands

Quickly locate files to see if they're installed, locate a config file, find docs, etc.

### which 

Shows full path name of a shell command or if it is using an alias:

```bash
# utility
which diff
/usr/bin/diff

# system binary
which shutdown
/usr/sbin/shutdown

# not in system
which line
```

### whereis

Locate the binaries, source code files, and man pages:

```bash
# utility
whereis diff
diff: /usr/bin/diff /usr/share/man/man1/diff.1.gz

# system binary
whereis shutdown
shutdown: /usr/sbin/shutdown /usr/share/man/man8/shutdown.8.gz /usr/share/man/man2/shutdown.2.gz

# not in system
whereis line
line:
```

### locate

Searches the `mlocate.db` database in `/var/lib/mlocate/` to see if the file exists on the system.
- Faster than `find`
- Uses an index that you might need to update before running

> `locate` might not find newly created files because `mlocate.db` is usually updated once a day with a cron job:

```bash
locate [OPTION]... PATTERN...
-A # matches all patterns in pattern list
-b # only filenames that match pattern, not directory names
-c # display count of files that match patterns
-i # ignore case
-q # quiet, do not display error messages
-r # use regex arg, not pattern list
-w # wholename, display filenames and direcories

# separate patterns with a space
locate -b '\passwd' '\group'
/snap/core22/1033/usr/share/doc-base/base-passwd.users-and-groups
/usr/share/doc-base/base-passwd.users-and-groups

# locate a file
locate file.1

# update index (setup cron job to run nightly)
updatedb
```

### find 

Locates files using data and metadata, such as:
- file owner
- last modified
- permissions

Specify the starting path for the command, and it will look through that tree.

| Option    | Expression | Description                                      |
| --------- | ---------- | ------------------------------------------------ |
| -cmin     | _n_        | Display names of files that changed _n_ mins ago |
| -empty    |            | Display empty files or dirs                      |
| -gid      | _n_        | Display files whos group id is _n_               |
| -group    | _name_     | Display files in group _name_                    |
| -inum     | _n_        | Display files with _inode_                       |
| -maxdepth | _n_        | Search _n_ directory levels                      |
| -mmin     | _n_        | Files whose data changed _n_ mins ago            |
| -name     | _pattern_  | Names of files that match _pattern_              |
| -nogroup  |            | No group name exists for the file's group ID     |
| -nouser   |            | No username exists for the file's user ID        |
| -perm     | _mode_     | Files whose permissions match _mode_             |
| -size     | _n_        | Files whose size matches _n_                     |
| -user     | _name_     | Files whose owner is _name_                      |

```bash
find [PATH...] [FILE OR dir] [OPTIONS] [ACTION ON RESULT]

# ACTIONS ON RESULT:
-delete                 # delete the files or dirs
-exec <command> {} \;   # run <command> on result from find
-ok <command>           # same as exec but prompts before execution

find . -name "*.txt"
./numbers.txt
./random.txt

find . -maxdepth 2 -name "*.txt"

# find binaries with SUID settings
$ find /usr/bin/ -perm /4000
/usr/bin/fusermount3
/usr/bin/passwd
/usr/bin/mount
...

# ----------------------------
# Examples

# type
find . -type d              # find all dirs in cwd
find . -type f              # find all files in cwd
find ~/Downloads/           # find all files and dirs in /Downloads
find ~/Downloads/ -type f   # find all files in /Downloads
find ~/Downloads/ -type d   # find all dirs in /Downloads
find $HOME -type f,d        # find all files and dirs in $HOME

# name and iname
find Development/ -type f -name *.go  # find all files with names that end in '.go'
find /var/log -type f -name *.log     # find all files with names that end in '.log'
find /var/log -type f -iname '*s.LoG' # case insensitive files with names that end in '.log'

# search multiple dirs
find Development/ /var/log -type f -name '*.log'

# exclude files or dirs
find test-files/ -type f -not -name '*.1' 
find Development/ -type f -not -path '*/lets-go/*'          # exclude files in -path <path>

find $HOME -regex <regex>                                   # find file with regex
find $HOME/test-files/ -name "*.gz" -o -name "*.tar"        # name <name> OR name <name>
find /home -type f -perm 0644                               # search by permissions
find $HOME -type f -name ".*"                               # hidden files
find / -maxdepth 2 -type d -name bin                        # find dir no deeper than 2 levels

find / -perm /g=s                                           # SGID files
find / -perm -2000                                          # SGID files
find / -perm /u=s                                           # SUID files
find / -perm -4000                                          # SUID files
find / -type f -perm -110                                   # x bit set for user and group

find $HOME -user linuxuser                                  # user ownership
find / -group linuxuser                                     # group ownership
find / -nouser                                              # find orphaned files
find $HOME -user u1 -o -user u2                             # owned by user u1 or u2
find / -not -user linuxuser                                 # not specified user

find $HOME -size 10M                                        # exact file size
find $HOME -size +10M                                       # greater than file size
find $HOME -size -10M                                       # less than file size

find / -xdev -size +100M 2>/dev/null                        # restrict search to specified fs

find / -mtime 4 2>/dev/null                                 # modified 4 days ago
find / -atime 5 2>/dev/null                                 # accessed 5 days ago

find $HOME -type f -empty                                   # empty files
find $HOME -type f -size 0                                  # empty files
find $HOME -type d -empty                                   # empty dir

find find-files/ -type f -name '*.mp4' -delete              # find and delete file
find find-files/ -type f -empty                             # find empty files



# exec commands
find $HOME -type f -exec ls -s {} \; | sort -rn | head -5     # find 5 largest files
find $HOME -type f -exec ls -s {} \; | sort -n | head -5      # find 5 smallest files
find find-files/ -type f -perm 777 -exec chmod 644 {} \;      # find files w 777 perms and change to 644
find /var/ -type f -name '*.log' -exec grep -i 'error' {} \;  # find files with text string
```

## lsof

Lists open files using info from the `/proc` virtual filesystem:

```bash
lsof /run           # what process is using /run
lsof -p 932         # what process is using process 932
lsof -u linuxuser   # what files linuxuser has open
lsof -i TCP:22      # what TCP process is using port 22
```

## fuser

```bash
# find user and PID for process
fuser -v /var/log/bad.log 
                     USER        PID ACCESS COMMAND
/var/log/bad.log:    admin       586 F.... badlog.py
```

## losetup

Sets up and controls loop devices
- Associate a file as a block device for testing

```bash
losetup /dev/loop0 file.img         # associate file with loop device
losetup -d /dev/loop0               # detach loop device from file

# list all attached loop devices
losetup -a
/dev/loop0: []: (/home/linuxuser/links/file.img)
```

## find

Searches the file system directly, looking for matches to the options you provide and outputs to STDOUT.
- Uses permissions of executing user

| Option      | Expression | Description                                          |
| ----------- | ---------- | ---------------------------------------------------- |
| `-cmin`     | _n_        | Display names of files that changed _n_ mins ago     |
| `-empty`    |            | Display empty files or dirs                          |
| `-gid`      | _n_        | Display files whos group id is _n_                   |
| `-group`    | _name_     | Display files in group _name_                        |
| `-inum`     | _n_        | Display files with _inode_                           |
| `-maxdepth` | _n_        | Search _n_ directory levels                          |
| `-mmin`     | _n_        | Files whose data changed _n_ mins ago                |
| `-mtime`    | _n_        | Files whose data changed _n_ days ago                |
| `-iname`    | _pattern_  | Case insensitive names of files that match _pattern_ |
| `-name`     | _pattern_  | Names of files that match _pattern_                  |
| `-nogroup`  |            | No group name exists for the file's group ID         |
| `-nouser`   |            | No username exists for the file's user ID            |
| `-perm`     | _mode_     | Files whose permissions match _mode_                 |
| `-size`     | _n_        | Files whose size matches _n_                         |
| `-user`     | _name_     | Files whose owner is _name_                          |
| `-xdev`     |            | Search within a single filesystem                    |

```bash
find [PATH...] [OPTION] [EXPRESSION]

find . -name "*.txt"
./numbers.txt
./random.txt

find . -maxdepth 2 -name "*.txt"

# find binaries with SUID settings
$ find /usr/bin/ -perm /4000
/usr/bin/fusermount3
/usr/bin/passwd
/usr/bin/mount
...
```

### -exec

Use `find` with the `-exec` option to execute a command on every file that `find` finds:

```bash
# {} represents the file that find located and sent to STDOUT
find [PATH...] [OPTION] [EXPRESSION] -exec [COMMAND] {} \;

# find user files and copy to dir
find $HOME -iname "*.1*" -exec cp {} find-files/ \;

# find files and create tar file
find $HOME -iname "*.1*" -exec tar -rvf find-files/find.tar {}  \;
```
