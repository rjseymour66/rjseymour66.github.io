---
title: "Files and directories"
weight: 30
description: >
  How to handle files and directories.
---

Files are stored within a structure called the _virtual directory_, a directory that contains files from all the computer's storage devices and merges them into a single directory structure. The _root_ directory is at the base of the virtual directory.

- A directory is a special file used to locate other files. This directory 'file' stores file name and inode number.
- 

## Tips

- `alias` to see all your aliases
- `tree` for pseudo-UI view
- `mkdir -p` make all subdirs in path
- `mkdir -v` verbal

## ls

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
-R # recursive list
```

## cp

Copy a file or directory locally.

```bash
cp [OPTION]... SOURCE DEST
-a # recursive, keep file attributes
-f # force overwrite the DEST
-i # interactive, ask before overwrite
-n # no-clobber, no overwrite DEST
-r,R # recursive (for directories)
-u # update, only overwrite DEST files with same name
-v # verbose
```

## mv

Move or rename a file.

```bash
mv [OPTION]... SOURCE DEST
-f # force overwrite files in DEST
-i # interactive
-n # -n # no-clobber, no overwrite DEST
-u # update, only overwrite DEST files with same name
-v # verbose
```

## rsync

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