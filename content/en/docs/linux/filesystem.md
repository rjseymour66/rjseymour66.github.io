---
title: "Filesystem"
# linkTitle: ""
# weight: 1000
# description:
---

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



## Symbolic links

### Soft link
An object that points to a file or directory somewhere else on the filesystem:

## Hard links

- A hard link is a file or directory that has one inode number but at least two different filenames.
  - One inode means its a single data file on the filesystem (single filesystem location on a disk partition)
  - Two filenames means that it can be accessed in multiple ways
- Use case: file backup when there is not enough space to backup the file. If someone deletes one of the files, its not permanently deleted.
- Original file must exist, linked file cannot exist
- Both files share the same data, exist on same filesystem in any directory
- Unlink the linked file with `unlink LINKED-FILE`
  
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