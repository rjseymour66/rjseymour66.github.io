---
title: "Cruising the filesystem"
linkTitle: "Filesystem"
weight: 40
# description:
---

## CDPATH

`CDPATH` holds a list of directories in memory that the shell searches when changing directories. It works like `PATH`, but for the `cd` command rather than searching for executables:
- define `CDPATH` in your `.bashrc` file
- when you change to a dir using `CDPATH`, the shell prints the absolute path to stdout
- add `..` to add every parent directory. This lets you change to sibling directories without needing to enter `..`. For example, `cd ../sibling` becomes `cd sibling`.
- works best if you don't have directories with duplicate names

```bash
CDPATH=$CDPATH:$HOME/core/first/second/third:..                     # define CDPATH var
echo $CDPATH
/home/elinux:/home/elinux/core:/home/elinux/core/first/second/third

~/core
cd fourth/
/home/elinux/core/first/second/third/fourth
```