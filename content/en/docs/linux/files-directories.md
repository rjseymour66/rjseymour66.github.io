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