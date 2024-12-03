---
title: "Basics"
linkTitle: "Basics"
# weight: 1000
# description:
---

## Set default text editor

Ubuntu uses `nano` by default, change it to `vim`:

```bash
sudo update-alternatives --config editor
There are 4 choices for the alternative editor (providing /usr/bin/editor).

  Selection    Path                Priority   Status
------------------------------------------------------------
  0            /bin/nano            40        auto mode
  1            /bin/ed             -100       manual mode
  2            /bin/nano            40        manual mode
* 3            /usr/bin/vim.basic   30        manual mode
  4            /usr/bin/vim.tiny    15        manual mode

Press <enter> to keep the current choice[*], or type selection number: 
```
## Change hostname

```bash
hostnamectl set-hostname mylittlecloudbox
```

## Set timezone

This is important for scheduling tasks and the timestamps for logs under `/var/log`:

```bash
# view current timezone
timedatectl
               Local time: Sun 2024-12-01 17:15:21 UTC
           Universal time: Sun 2024-12-01 17:15:21 UTC
                 RTC time: Sun 2024-12-01 17:15:21
                Time zone: Etc/UTC (UTC, +0000)             # current tz
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no

# get list of available timezones
timedatectl list-timezones

# set new timezone
sudo timedatectl set-timezone America/New_York

# verify
timedatectl
               Local time: Sun 2024-12-01 12:15:57 EST
           Universal time: Sun 2024-12-01 17:15:57 UTC
                 RTC time: Sun 2024-12-01 17:15:57
                Time zone: America/New_York (EST, -0500)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no


```

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

Check what kind of command:

```bash

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
### popd

```bash

```