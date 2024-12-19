---
title: "Filesystem"
# linkTitle: ""
# weight: 1000
# description:
---

View supported filesystems:

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

```bash
# superblock struct in fs.h
struct super_block {
        struct list_head        s_list;         /* Keep this first */
        dev_t                   s_dev;          /* search index; _not_ kdev_t */
        unsigned char           s_blocksize_bits;
        unsigned long           s_blocksize;
        loff_t                  s_maxbytes;     /* Max file size */
        struct file_system_type *s_type;
        const struct super_operations   *s_op;
        const struct dquot_operations   *dq_op;
        const struct quotactl_ops       *s_qcop;
        const struct export_operations *s_export_op;
        unsigned long           s_flags;
        unsigned long           s_iflags;       /* internal SB_I_* flags */
        unsigned long           s_magic;
        struct dentry           *s_root;
        struct rw_semaphore     s_umount;
        int                     s_count;
        atomic_t                s_active;
#ifdef CONFIG_SECURITY
        void                    *s_security;
#endif
        const struct xattr_handler * const *s_xattr;
#ifdef CONFIG_FS_ENCRYPTION
        const struct fscrypt_operations *s_cop;
        struct fscrypt_keyring  *s_master_keys; /* master crypto keys in use */
#endif
#ifdef CONFIG_FS_VERITY
        const struct fsverity_operations *s_vop;
#endif
#if IS_ENABLED(CONFIG_UNICODE)
        struct unicode_map *s_encoding;
        __u16 s_encoding_flags;
#endif
        struct hlist_bl_head    s_roots;        /* alternate root dentries for NFS */
        struct list_head        s_mounts;       /* list of mounts; _not_ for fs use */
        struct block_device     *s_bdev;
        struct bdev_handle      *s_bdev_handle;
        struct backing_dev_info *s_bdi;
        struct mtd_info         *s_mtd;
        struct hlist_node       s_instances;
        unsigned int            s_quota_types;  /* Bitmask of supported quota types */
        struct quota_info       s_dquot;        /* Diskquota specific options */

        struct sb_writers       s_writers;

        /*
         * Keep s_fs_info, s_time_gran, s_fsnotify_mask, and
         * s_fsnotify_marks together for cache efficiency. They are frequently
         * accessed and rarely modified.
         */
        void                    *s_fs_info;     /* Filesystem private info */

        /* Granularity of c/m/atime in ns (cannot be worse than a second) */
        u32                     s_time_gran;
        /* Time limits for c/m/atime in seconds */
        time64_t                   s_time_min;
        time64_t                   s_time_max;
#ifdef CONFIG_FSNOTIFY
        __u32                   s_fsnotify_mask;
        struct fsnotify_mark_connector __rcu    *s_fsnotify_marks;
#endif

        char                    s_id[32];       /* Informational name */
        uuid_t                  s_uuid;         /* UUID */

        unsigned int            s_max_links;

        /*
         * The next field is for VFS *only*. No filesystems have any business
         * even looking at it. You had been warned.
         */
        struct mutex s_vfs_rename_mutex;        /* Kludge */

        /*
         * Filesystem subtype.  If non-empty the filesystem type field
         * in /proc/mounts will be "type.subtype"
         */
        const char *s_subtype;

        const struct dentry_operations *s_d_op; /* default d_op for dentries */

        struct shrinker *s_shrink;      /* per-sb shrinker handle */

        /* Number of inodes with nlink == 0 but still referenced */
        atomic_long_t s_remove_count;

        /*
         * Number of inode/mount/sb objects that are being watched, note that
         * inodes objects are currently double-accounted.
         */
        atomic_long_t s_fsnotify_connectors;

        /* Read-only state of the superblock is being changed */
        int s_readonly_remount;

        /* per-sb errseq_t for reporting writeback errors via syncfs */
        errseq_t s_wb_err;

        /* AIO completions deferred from interrupt context */
        struct workqueue_struct *s_dio_done_wq;
        struct hlist_head s_pins;

        /*
         * Owning user namespace and default context in which to
         * interpret filesystem uids, gids, quotas, device nodes,
         * xattrs and security labels.
         */
        struct user_namespace *s_user_ns;

        /*
         * The list_lru structure is essentially just a pointer to a table
         * of per-node lru lists, each of which has its own spinlock.
         * There is no need to put them into separate cachelines.
         */
        struct list_lru         s_dentry_lru;
        struct list_lru         s_inode_lru;
        struct rcu_head         rcu;
        struct work_struct      destroy_work;

        struct mutex            s_sync_lock;    /* sync serialisation lock */

        /*
         * Indicates how deep in a filesystem stack this SB is
         */
        int s_stack_depth;

        /* s_inode_list_lock protects s_inodes */
        spinlock_t              s_inode_list_lock ____cacheline_aligned_in_smp;
        struct list_head        s_inodes;       /* all inodes */

        spinlock_t              s_inode_wblist_lock;
        struct list_head        s_inodes_wb;    /* writeback inodes */
} __randomize_layout;
```

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

### inode struct

```bash
# inode struct in fs.h
struct inode {
        umode_t                 i_mode;
        unsigned short          i_opflags;
        kuid_t                  i_uid;
        struct list_head        i_lru;          /* inode LRU list */
        kgid_t                  i_gid;
        unsigned int            i_flags;

#ifdef CONFIG_FS_POSIX_ACL
        struct posix_acl        *i_acl;
        struct posix_acl        *i_default_acl;
#endif

        const struct inode_operations   *i_op;
        struct super_block      *i_sb;
        struct address_space    *i_mapping;

#ifdef CONFIG_SECURITY
        void                    *i_security;
#endif

        /* Stat data, not accessed from path walking */
        unsigned long           i_ino;
        /*
         * Filesystems may only read i_nlink directly.  They shall use the
         * following functions for modification:
         *
         *    (set|clear|inc|drop)_nlink
         *    inode_(inc|dec)_link_count
         */
        union {
                const unsigned int i_nlink;
                unsigned int __i_nlink;
        };
        dev_t                   i_rdev;
        loff_t                  i_size;
        struct timespec64       __i_atime;
        struct timespec64       __i_mtime;
        struct timespec64       __i_ctime; /* use inode_*_ctime accessors! */
        spinlock_t              i_lock; /* i_blocks, i_bytes, maybe i_size */
        unsigned short          i_bytes;
        u8                      i_blkbits;
        u8                      i_write_hint;
        blkcnt_t                i_blocks;

#ifdef __NEED_I_SIZE_ORDERED
        seqcount_t              i_size_seqcount;
#endif

        /* Misc */
        unsigned long           i_state;
        struct rw_semaphore     i_rwsem;

        unsigned long           dirtied_when;   /* jiffies of first dirtying */
        unsigned long           dirtied_time_when;

        struct hlist_node       i_hash;
        struct list_head        i_io_list;      /* backing dev IO list */
#ifdef CONFIG_CGROUP_WRITEBACK
        struct bdi_writeback    *i_wb;          /* the associated cgroup wb */

        /* foreign inode detection, see wbc_detach_inode() */
        int                     i_wb_frn_winner;
        u16                     i_wb_frn_avg_time;
        u16                     i_wb_frn_history;
#endif
        struct list_head        i_sb_list;
        struct list_head        i_wb_list;      /* backing dev writeback list */
        union {
                struct hlist_head       i_dentry;
                struct rcu_head         i_rcu;
        };
        atomic64_t              i_version;
        atomic64_t              i_sequence; /* see futex */
        atomic_t                i_count;
        atomic_t                i_dio_count;
        atomic_t                i_writecount;
#if defined(CONFIG_IMA) || defined(CONFIG_FILE_LOCKING)
        atomic_t                i_readcount; /* struct files open RO */
#endif
        union {
                const struct file_operations    *i_fop; /* former ->i_op->default_file_ops */
                void (*free_inode)(struct inode *);
        };
        struct file_lock_context        *i_flctx;
        struct address_space    i_data;
        struct list_head        i_devices;
        union {
                struct pipe_inode_info  *i_pipe;
                struct cdev             *i_cdev;
                char                    *i_link;
                unsigned                i_dir_seq;
        };

        __u32                   i_generation;

#ifdef CONFIG_FSNOTIFY
        __u32                   i_fsnotify_mask; /* all events this inode cares about */
        struct fsnotify_mark_connector __rcu    *i_fsnotify_marks;
#endif

#ifdef CONFIG_FS_ENCRYPTION
        struct fscrypt_inode_info       *i_crypt_info;
#endif

#ifdef CONFIG_FS_VERITY
        struct fsverity_info    *i_verity_info;
#endif

        void                    *i_private; /* fs or device private pointer */
} __randomize_layout;
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

| Option | Expression | Description |
|--------|------------|-------------|
| -cmin  | _n_ | Display names of files that changed _n_ mins ago |
| -empty |  | Display empty files or dirs |
| -gid   | _n_ | Display files whos group id is _n_ |
| -group | _name_ | Display files in group _name_ |
| -inum | _n_ | Display files with _inode_ |
| -maxdepth | _n_ | Search _n_ directory levels |
| -mmin | _n_ | Files whose data changed _n_ mins ago |
| -name | _pattern_ | Names of files that match _pattern_ |
| -nogroup |  | No group name exists for the file's group ID |
| -nouser |  | No username exists for the file's user ID |
| -perm | _mode_ | Files whose permissions match _mode_ |
| -size | _n_ | Files whose size matches _n_ |
| -user | _name_ | Files whose owner is _name_ |

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

## Format a partition

### mkfs

```bash
mkfs.<fs-type> <partition>

# create an exfat filesystem
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

## find

Searches the file system directly, looking for matches to the options you provide and outputs to STDOUT.
- Uses permissions of executing user

| Option | Expression | Description |
|--------|------------|-------------|
| `-cmin`  | _n_ | Display names of files that changed _n_ mins ago |
| `-empty` |  | Display empty files or dirs |
| `-gid`   | _n_ | Display files whos group id is _n_ |
| `-group` | _name_ | Display files in group _name_ |
| `-inum` | _n_ | Display files with _inode_ |
| `-maxdepth` | _n_ | Search _n_ directory levels |
| `-mmin` | _n_ | Files whose data changed _n_ mins ago |
| `-mtime` | _n_ | Files whose data changed _n_ days ago |
| `-iname` | _pattern_ | Case insensitive names of files that match _pattern_ |
| `-name` | _pattern_ | Names of files that match _pattern_ |
| `-nogroup` |  | No group name exists for the file's group ID |
| `-nouser` |  | No username exists for the file's user ID |
| `-perm` | _mode_ | Files whose permissions match _mode_ |
| `-size` | _n_ | Files whose size matches _n_ |
| `-user` | _name_ | Files whose owner is _name_ |
| `-xdev` | | Search within a single filesystem |

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

## Soft (symbolic) links
An object that points to an object somewhere else on the system:
- Have unique inode
- Reference abstractions, not physical place on disk
- Can point to file or dir
- Can reference an object on another disk or volume
- Is not deleted if you delete the original file, but won't reference anything
  
## Hard links

An object that points to another file on the same disk or volume
- A hard link is a file that has one inode number but at least two different filenames.
  - One inode means its a single data file on the filesystem (single filesystem location on a disk partition)
  - Two filenames means that it can be accessed in multiple ways
- References physical location on disk
- Shares inode with original file
- Is not deleted if you delete original file and references the same data
- Original file must exist, linked file cannot exist
- Both files share the same data, exist on same filesystem in any directory
- Unlink the linked file with `unlink LINKED-FILE`

Use case
: file backup when there is not enough space to backup the file. If someone deletes one of the files, its not permanently deleted.
  
```bash
ln ORIGINAL LINKED-FILE
unlink LINKED-FILE

# create original
touch original-file.txt
original-file.txt

# link files
ln original-file.txt hard-link-file.txt 

# different file for comparison
touch single-file.txt

# view inode and links
ls -iog
total 0           (*)
4868127 -rw-rw-r-- 2 0 Mar 17 09:32 hard-link-file.txt
4868127 -rw-rw-r-- 2 0 Mar 17 09:32 original-file.txt
4868128 -rw-rw-r-- 1 0 Mar 17 09:33 single-file.txt

# unlink
unlink hard-link-file.txt 

# linked file is gone
ls -iog
total 0           (*)
4868127 -rw-rw-r-- 1 0 Mar 17 09:32 original-file.txt
4868128 -rw-rw-r-- 1 0 Mar 17 09:33 single-file.txt
```

## Soft links

Also called _symbolic links_.

- Typically a soft link is a pointer to a file or directory that might be on another filesystem
- Different inodes bc they point to different data
- If a soft link points to a file that was deleted or removed, that is a security risk in the event that a malicious file is put in the original file's place.
- View or execute a resource from the local dir
- For apache, you can delete the symlink in `/sites-enabled/` instead of the config in `/sites-avaiable/`, then readd the link when you want to reactivate the site.

```bash
ln -s ORIGINAL LINKED-FILE

# current contents
ls -iog
total 0
4868127 -rw-rw-r-- 1 0 Mar 17 09:32 original-file.txt

# create soft link
ln -s original-file.txt soft-link-file.txt

# linked, but different inode and link numbers
ls -iog
total 0           (*)
4868127 -rw-rw-r-- 1  0 Mar 17 09:32 original-file.txt
4868128 lrwxrwxrwx 1 17 Mar 17 09:45 soft-link-file.txt -> original-file.txt

# rm link
unlink soft-link-file.txt

# linked file is deleted
ls -iog
total 0
4868127 -rw-rw-r-- 1 0 Mar 17 09:32 original-file.txt
```

## Compression

The following compression methods all provide lossless compression: 
_gzip_
: Replaced old `compress` program. Replaces original file with a compressed version and .gz file extension. `gunzip` to reverse the operation.

_bzip2_
: Compressed Linux kernel until 2013. Replaces original file.

_xz_
: Popular with Linux admins. Compresses Linux Kernel since 2013. Replaces original file

_zip_
: Can operate on multiple files, packs them together in 'folder' or 'archive'. Does not replace original file--places copy of the file into an archive.



```bash
# use the -<n> option with all but zip to control compression.
# -1 is fast but lowest compression
# -9 is slow but highest compression
# -6 is default 
$ ls -lh wtmp?
-rw-rw-r-- 1 137K Apr  4 22:08 wtmp1
-rw-rw-r-- 1 137K Apr  4 22:08 wtmp2
-rw-rw-r-- 1 137K Apr  4 22:08 wtmp3
-rw-rw-r-- 1 137K Apr  4 22:08 wtmp4

# compress each file
gzip wtmp1      # gunzip <compressed-file> to reverse
bzip2 wtmp2     # bunzip2 <compressed-file> to reverse
xz wtmp3        # unxz <compressed-file> to reverse 
zip wtmp4.zip wtmp4     # unzip <compressed-file> to reverse
  adding: wtmp4 (deflated 96%)

# xz compresses the best
ls -lh wtmp?.*
-rw-rw-r-- 1 5.6K Apr  4 22:08 wtmp1.gz
-rw-rw-r-- 1 4.4K Apr  4 22:08 wtmp2.bz2
-rw-rw-r-- 1 3.9K Apr  4 22:08 wtmp3.xz
-rw-rw-r-- 1 5.7K Apr  4 22:18 wtmp4.zip

# zip did not replace the file, still there
$ ls wtmp?
wtmp4
```

### gzip

Compresses a file:
- Best and most common compression tool.

```bash
# compress and delete original
gzip find.tar

# -c option to compress file and preserve original
gzip -c find.tar > find.tar.gz

# unzip with gunzip
gunzip find.tar.gz
```