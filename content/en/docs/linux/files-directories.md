---
title: "Files and directories"
weight: 30
description: >
  How to handle files and directories.
---

Files are stored within a structure called the _virtual directory_, a directory that contains files from all the computer's storage devices and merges them into a single directory structure. The _root_ directory is at the base of the virtual directory.

- A directory is a special file used to locate other files. This directory 'file' stores file name and inode number.
- 

## Common commands

### Tips

- `alias` to see all your aliases
- `tree` for pseudo-UI view
- `mkdir -p` make all subdirs in path
- `mkdir -v` verbal

### ls

View files' and subdirectories' names and metadata.

```bash
ls [OPTION]... [FILE]...
ll # list long
-a # all files, incl hidden
-d # current dir own metadata
-F # classify 
-h # human readable byte size
-i # inodes
-l # long (all metadata)
-o # no group info
-R # recursive list
```

### cp

Copy a file or directory locally.

```bash
cp [OPTION]... SOURCE DEST
-a # recursive, keep file attributes
-f # force overwrite the DEST
-i # interactive, ask before overwrite
-n # no-clobber, no overwrite DEST
-R,r # recursive (for directories)
-u # update, only overwrite DEST files with same name
-v # verbose
```

### mv

Move or rename a file.

```bash
mv [OPTION]... SOURCE DEST
-f # force overwrite files in DEST
-i # interactive
-n # -n # no-clobber, no overwrite DEST
-u # update, only overwrite DEST files with same name
-v # verbose
```

### rsync

Remote sync. Copies large files or a group of large files quickly, including:
- copy files over a network
- create backups

```bash
rsync [OPTION]... SOURCE DEST
-a # archive mode, equivalent to -rlptgoD (dir tree backups)
-D # retain Device and special files
-g # retain file group
-h # human-readable numeric output
-l # copy symbolic links
-o # retain file owner
-p # retain file perms
--progress # display progress of file copy
-r # recursive
--stats # display file transfer stats
-t # retain file's modification time
-v # verbose
```

### rm

Removes files and directories.

```bash
rm [OPTION]... FILE
-d # delete empty directories
-f # force
-i # interactive
-I # ask before deleting more than 3 files
-R,r # recursive
-v # verbose
```

## Linking files and directories

There are two types of links: hard and soft.

### Hard links

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

### Soft links

Also called _symbolic links_.

- Typically a soft link is a pointer to a file that might be on another filesystem
- Different inodes bc they point to different data
- If a soft link points to a file that was deleted or removed, that is a security risk in the event that a malicious file is put in the original file's place.

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

## Reading entire text files

### cat

Also check out `bat`?

```bash
cat [OPTION]... [FILE]...
-n # add numbers to output line
```

### pr

Display file with special formatting.

```bash
pr [OPTION]... [FILE]...
-n # display file in n columns
-l <n> # default lines displayed
-m # merge multiple files into one
-s <c> # change default column separator to c
-t # no file header or trailer
-w <n> # change default page width to n

pr -tl 15 filename.txt

# great for comparing files (merging output)
pr -mtl 15 file1.txt file2.txt
```

## Reading portions of text files

### grep

Search for text patterns or a single string in a file.

```bash
grep [OPTIONS] PATTERN [FILE...]

grep -i ryan /etc/passwd

# search for 'hosts:' in all /etc/ files, skipping dirs (-d skip)
grep -d skip hosts: /etc/*
```

### head

Display first lines of a file (10 by default).

```bash
head [OPTION]... [FILE]...

# these two are equivalent
head -n 2 file.txt
head -2 file.txt

# display all file lines except the last 40
head -n -40 file.txt
```

### tail 

Display last lines of a file (10 by default) and watching log files:

```bash
tail [OPTION]... [FILE]...

# these are equivalent
tail -n 2 file.txt
tail -2 file.txt

# begin display at line 40 of file
tail -n +40 file.txt

# Very useful to watch log files with `-f` (follow) option:
tail -f /var/log/auth.log
```

## Reading text file pages

### pagers (more and less)

`more` won't let you go back, but `less` will (`less` is more):

```bash
less file.txt
# up and down arrows
# spacebar to move forward
# ? to search backward for word in file
# / to search forward for word in file
# q to exit
```

## Viewing file information

### file and stat

```bash
# file provides basic info about the file type
file testscript.sh 
testscript.sh: Bourne-Again shell script, ASCII text executable

# stat shows the file and history
  stat testscript.sh 
  File: testscript.sh
  Size: 75        	Blocks: 8          IO Block: 4096   regular file
Device: 10303h/66307d	Inode: 4873930     Links: 1
Access: (0755/-rwxr-xr-x)  Uid: ( 1001/ryanseymour)   Gid: ( 1001/ryanseymour)
Access: 2024-03-17 15:05:57.739663588 -0400
Modify: 2024-03-17 15:05:56.439662366 -0400
Change: 2024-03-17 15:05:56.443662370 -0400
 Birth: 2024-03-17 15:05:56.439662366 -0400
```

### diff

Compare two files, line by line.

```bash
diff [OPTION]... FILES
-e # create ed script
-q # brief message if files are different
-r # recursive comparisons
-s # report identical files, brief message if files are same
-W n # width max of n chars for output
-y # display output in 2 columns

# no args
diff numbers.txt random.txt 
2,3c2,3 # to make numbers match random, change lines 2 and 3 (2,3 changes to 2,3)
< 2A
< 52
---
> Flat Land
> Schrodinger\'s Cat
5c5 # line 5 needs to be changed
< *
---
> 0000 0010


# see difference between files (with and w/o -W)
diff -yW 35 numbers.txt random.txt 
42		    42
2A	      |	Flat Land
52	      |	Schrodinger's
0010 1010	0010 1010
*	      |	0000 0010

# w/o -W
diff -y numbers.txt random.txt 
42								    42
2A							      |	Flat Land
52							      |	Schrodinger's Cat
0010 1010							0010 1010
*							      |	0000 0010
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

