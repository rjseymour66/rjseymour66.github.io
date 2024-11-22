---
title: "Filesystem"
# linkTitle: ""
# weight: 1000
# description:
---



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

Searches the file system looking for matches to the options you provide and outputs to STDOUT.

| Option | Expression | Description |
|--------|------------|-------------|
| -cmin  | _n_ | Display names of files that changed _n_ mins ago |
| -empty |  | Display empty files or dirs |
| -gid   | _n_ | Display files whos group id is _n_ |
| -group | _name_ | Display files in group _name_ |
| -inum | _n_ | Display files with _inode_ |
| -maxdepth | _n_ | Search _n_ directory levels |
| -mmin | _n_ | Files whose data changed _n_ mins ago |
| -iname | _pattern_ | Case insensitive names of files that match _pattern_ |
| -name | _pattern_ | Names of files that match _pattern_ |
| -nogroup |  | No group name exists for the file's group ID |
| -nouser |  | No username exists for the file's user ID |
| -perm | _mode_ | Files whose permissions match _mode_ |
| -size | _n_ | Files whose size matches _n_ |
| -user | _name_ | Files whose owner is _name_ |

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

## gzip

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

## locate

Searches entire file system for files matching string pattern:
- Faster than `find`
- Uses an index that you might need to update before running

```bash
# locate a file
locate file.1

# update index
updatedb
```

## Symbolic links


### Soft link
An object that points to a file or directory somewhere else on the filesystem:
- 



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