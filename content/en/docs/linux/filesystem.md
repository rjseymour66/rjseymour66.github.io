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