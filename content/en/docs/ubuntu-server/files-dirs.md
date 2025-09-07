---
title: "xFiles and directories"
# linkTitle: ""
weight: 40
# description:
---

## vim

`vim.nox` is a minimal flavor of `vim` for server environments that has scripting language support.
- command mode lets you move around the file
- insert mode lets you edit text
- visual mode lets you select text for copy/paste
- Add default settings to `.vimrc` in `/home` dir. Ex: `set number`

```bash
CTRL + z        # put vim session in background
fg              # bring vim session to foreground
A               # insert mode at the end of the current line
gg              # go to beginning of first line of file
G               # go to beginning of last line of file
:%s/original string/new string/g    # global replace
:! <shell command>                  # run <shell command> while vim is open
:set number     # add line numbers
:set nonumber   # remove line numbers

# --- Split files --- #
:sp /path/to/file                   # split current window with file window
:vs /path/to/file                   # vertical split window and file
CTRL + w, CTRL + w                  # switch between open files
```

## Streams

### STDIN, STDOUT, STDERR

Every program is a process, and every process has 3 distinct file descriptors:
- `stdin`: File descriptor 0. Default source for input to a program
- `stdout`: File descriptor 1. Default place for sending output (the console)
- `stderr`: File descriptor 2. Where error messages are written


### Redirection and piping

Redirection is when you change the input and outputs of a program without modifying the program.

| Operator     | Description                                                          | Example                                            |
| ------------ | :------------------------------------------------------------------- | -------------------------------------------------- |
| `>`          | Sends the output of the value on the left to the value on the right. | `ls -la > listing.out`                             |
| `<`          | Sends the value on the right to the STDIN of the value on the left.  | `program < input.txt`                              |
| `2>`         | Redirect STDERR messages to the value on the right.                  | `cp -r /etc/a /etc/b 2> err.msg`                   |
| `2>&1`, `&>` | Send output to STDOUT and STDERR. Place this                         | `XXXXXXXXXXX`                                      |
| `1> file 2>` | Redirect more than one stream                                        | `find / -name 'syslog' > stdout.txt 2> stderr.txt` |

The following command sends `input.txt` to the STDIN of `program`, and sends the output to `output.out`:
```bash
program < input.txt > output.out
```

To distinguish between STDOUT and STDERR, use the file descriptor `2>`. The following command redirects error messages to `err.msgs`:
```bash
program 2> err.msgs
```
Or, combine all redirection methods:
```bash
program < input.txt > output.out 2> err.msgs
```

Combined redirect: To send STDERR to the same location as STDOUT, combine the error messages with the standard output:
```bash
program < input.txt > output.out 2>&1
# shorthand
program < input.txt &> output.out
```
The previous command sends input.txt to program, then sends the STDOUT and STDERR to output.out. The shorthand is more clear.

Commonly, you can discard STDOUT by sending it to `/dev/null`:
```bash
program < input.txt > /dev/null
```

The `tee` command sends output to STDOUT and the file that follows the command:
```bash
program < input.txt | tee results.out # the -a option allows tee to append to the file
```
To append to a file, use the `>>` operator:
```bash
program < input.txt >> appended.file
```
To append STDOUT and STDERR, use `&>>`:
```bash
program < input.txt &>> appended.file
```

### Running commands in the background

Use the `&` operator at the end of the command to run it in the background:
```bash
ping 10.20.30.40 > ping.log &
```

When you run a task in the background, send both STDOUT and STDERR so the task doesn't log everything to the console:
```bash
ping 10.20.30.40 &> ping.log &
```

To bring a job back to the foreground, use `jobs` to list running tasks, then use `fg` with the corresponding task number:
```bash
ping 192.168.10.56 &> ping.log &
[1] 7452
jobs
[1]+  Running                 ping 192.168.10.56 &> ping.log &
fg 1
ping 192.168.10.56 &> ping.log
^C
```
Stop a running job with `Ctrl+z`, and send it to the background with `bg`:
```bash
ping 192.168.10.56 &> ping.log &
[1] 7572
fg 1
ping 192.168.10.56 &> ping.log
^Z
[1]+  Stopped                 ping 192.168.10.56 &> ping.log
bg 1
[1]+ ping 192.168.10.56 &> ping.log &
fg 1
ping 192.168.10.56 &> ping.log
^C
```

## Links

Links let you reference a file somewhere else on the system to create shortcuts:
- `inode` is a database object that contains metadata about a file or directory, such as the woner, permissions, last modified date, type,...

### Hard link

Hard links are duplicates of the original file:
- Both files point to the same inode
- Because they share an inode, you cannot reference files on another filesystem
- Can only create hard links to file
- You can move either file anywhere in the fs and it will not break the link bc hard links use inodes

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


### Symbolic (soft) link

A symbolic link is a pointer to the original file's path, not a clone. It's an object that points to an object somewhere else on the system:
- Linked files have different inodes
- Cannot move original file, because symlinks just point to a path location
- Can link across filesystems
- Has unique inode
- Reference abstractions, not physical place on disk
- Can point to file or dir
- Can reference an object on another disk or volume
- Is not deleted if you delete the original file, but won't reference anything
- Typically a soft link is a pointer to a file or directory that might be on another filesystem
- Different inodes bc they point to different data
- If a soft link points to a file that was deleted or removed, that is a security risk in the event that a malicious file is put in the original file's place.
- View or execute a resource from the local dir

#### Use case

For apache, you can delete the symlink in `/sites-enabled/` instead of the config in `/sites-avaiable/`, then readd the link when you want to reactivate the site.

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