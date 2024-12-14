---
title: "Basics"
linkTitle: "Basics"
# weight: 1000
# description:
---

## Command information

### tldr

Shortened version of man pages:

```bash
tldr cd

  cd

  Change the current working directory.
  More information: https://manned.org/cd.

  - Go to the specified directory:
    cd path/to/directory

  - Go up to the parent of the current directory:
    cd ..

  - Go to the home directory of the current user:
    cd

  - Go to the home directory of the specified user:
    cd ~username

  - Go to the previously chosen directory:
    cd -

  - Go to the root directory:
    cd /
```

### apropos and man -k

Search `man` pages by keyword:

```bash
# search by keyword
apropos 'rebase'
git-rebase (1)       - Reapply commits on top of another base tip

# same search with man -k
man -k 'rebase'
git-rebase (1)       - Reapply commits on top of another base tip
```


### help

If `man <command>` doesn't work, use `help`:

```bash
man export
No manual entry for export

# search with help
$ help export
export: export [-fn] [name[=value] ...] or export -p
    Set export attribute for shell variables.
    
    Marks each NAME for automatic export to the environment of subsequently
    executed commands.  If VALUE is supplied, assign VALUE before exporting.
    ...
```

### info

Provides more detail than the `man` pages:

```bash
info ls
```

### type

The type command is used to find out if command is builtin or external binary file. It also indicate how it would be interpreted if used as a command name.

```bash
type ls
ls is aliased to `ls --color=auto'
```

## Directory info

### man hier

Describes the Linux directory hierarchy:

```bash
HIER(7)                                       Linux Programmer's Manual                                       HIER(7)

NAME
       hier - description of the filesystem hierarchy

DESCRIPTION
       A typical Linux system has, among others, the following directories:

       /      This is the root directory.  This is where the whole tree starts.

       /bin   This  directory contains executable programs which are needed in single user mode and to bring the sys‐
              tem up or repair it.
       ...
```

### ls

```bash
ls [OPTION]... [FILE]...
ll # list long
la # list all except . and ..
l  # list in columns
-1 # one entry per line
-a # all files, incl hidden
-d # current dir own metadata
-F # classify 
-h # human readable byte size
-i # inodes
-l # long (all metadata)
-o # no group info
-r # reverse order
-R # recursive list
-t # list by time file was modified

# common usage to see oldest modified object
ls -ltra
```

### pushd and popd

Tracks your directory history. `pushd` puts your dir in a stack, and `popd` pops it off so you can retrace your steps:

```bash
~$ pushd /var/log
/var/log ~

/var/log$ pushd /dev/
/dev /var/log ~

/dev$ pushd /etc/
/etc /dev /var/log ~

/etc$ popd
/dev /var/log ~

/dev$ popd
/var/log ~

/var/log$ popd
~

~$ 

```
## history

Get your command history to rerun commands:
- Ctrl + r starts an autocomplete program that matches what you type with anything in your history

```bash
# view all history
history

# view previous 10 commands
history 10

# grep history
history | grep less

# execute a command from history
!<history-number>
!98

# execute command relative to current history
!-2

# execute last command
!!

# execute last command with sudo
sudo !!
# example:
cat /etc/shadow
cat: /etc/shadow: Permission denied

sudo !!
sudo cat /etc/shadow
...

# replace incorrect string in previous command
cat /etc/shosst
cat: /etc/shosst: No such file or directory
# ^incorrect^corrected^
^shosst^hosts^
cat /etc/hosts
127.0.0.1 localhost
...

# word designator - access arg from prev command
less .profile 
# - replace less with cat
cat !!:1
cat .profile
# ~/.profile: executed by the command interpreter for login shells.
...

# word designator - get all args from previous command
!!:1*
!!:*


```

## Pagers

You can use less or more:
- more has forward nav and limited backward nav
- less: forward and backward nav, search options, switch to an editor, faster for large files


### less

```bash

# forward search
/ # start search - enter pattern
n # go to next match (forwards)
N # go to previous match (backwards)


# backward search
? # start search - enter pattern
n # go to next match (backwards)
N # go to previous match (forwards)

f # forward one window
b # backward one window
d # forward 1/2 window
u # backward 1/2 window

j # forward one line
10j # forward 10 lines
k # backward one line
10k # backward 10 lines

G # end of file
g # start of file
q or ZZ # exit

F # waiting for data mode (file is being written to)

v # use default editor to edit
h # help
&<pattern> # display only pattern matches

# mark file and return to it
m<letter> # mark a line in a file - ex: ma
'<letter> # go back to the mark - ex: 'a
```