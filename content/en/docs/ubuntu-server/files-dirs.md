---
title: "Files and directories"
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

| Operator | Description | Example |
|----------|:------------|---------|
| `>` | Sends the output of the value on the left to the value on the right. | `ls -la > listing.out` |
| `<` | Sends the value on the right to the STDIN of the value on the left. | `program < input.txt` |
| `2>` | Redirect STDERR messages to the value on the right. | `cp -r /etc/a /etc/b 2> err.msg` |
| `2>&1`, `&>`| Send output to STDOUT and STDERR. Place this  | `XXXXXXXXXXX` |
| `1> file 2>` | Redirect more than one stream | `find / -name 'syslog' > stdout.txt 2> stderr.txt` |

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

### Symbolic link

A symbolic link is a pointer to the original file's path, not a clone:
- Linked files have different inodes
- Cannot move original file, because symlinks just point to a path location
- Can link across filesystems
- Can reference a directory