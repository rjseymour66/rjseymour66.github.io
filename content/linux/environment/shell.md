+++
title = 'Shell'
date = '2025-09-07T18:57:00-04:00'
weight = 30
draft = false
+++


The shell is the command-line interface between you and the operating system. Each user account is assigned a default shell, stored in the last field of `/etc/passwd`. System accounts are typically assigned `/bin/false` or `/usr/sbin/nologin` to prevent interactive login:

```bash
cat /etc/passwd | cut -d: -f 1,7
...
dhcpcd:/bin/false
sshd:/usr/sbin/nologin
testuser:/bin/bash
```

## General command efficiency

Common keyboard shortcuts for navigating and editing commands at the prompt:

| Shortcut       | Description                                |
| :------------- | :----------------------------------------- |
| `Tab` `Tab`    | Cycle through tab completion options       |
| `Ctrl + a`     | Move cursor to beginning of line           |
| `Ctrl + e`     | Move cursor to end of line                 |
| `Ctrl + l`     | Clear the screen                           |
| `Ctrl + k`     | Delete from cursor to end of line          |
| `Ctrl + u`     | Delete entire line (also clears passwords) |
| `Ctrl + w`     | Delete word to the left of cursor          |
| `Ctrl + r`     | Search command history                     |
| `Ctrl + x` `e` | Open current command in editor             |
| `echo $?`      | View exit code of previous command         |


## History

The shell records every command you run in `~/.bash_history`. To prevent a command from being saved, prefix it with a space. On some distributions, add `HISTCONTROL=ignoreboth` to `~/.bashrc` to enable this behavior:

| Command                  | Description                                        |
| :----------------------- | :------------------------------------------------- |
| `history`                | View bash history                                  |
| `!!`                     | Repeat the previous command                        |
| `!<num>`                 | Run command number `<num>` from history            |
| `history -d <num>`       | Delete command number `<num>` from history         |
| `<space><cmd>`           | Run a command without saving it to history         |
| `HISTCONTROL=ignoreboth` | Add to `~/.bashrc` to enable space-prefix behavior |

## Aliases

An alias substitutes a short command for a longer one. Add aliases to `~/.bashrc` to make them permanent across sessions.

Manage aliases in the current session:

| Command          | Description                               |
| :--------------- | :---------------------------------------- |
| `alias`          | List all active aliases                   |
| `unalias <name>` | Remove a specific alias                   |
| `unalias -a`     | Remove all aliases in the current session |

For example, define aliases for common tasks:

```bash
alias cpu10='ps -L aux | sort -nr -k 3 | head -10'   # list top 10 CPU-consuming processes
alias mem10='ps -L aux | sort -nr -k 4 | head -10'   # list top 10 memory-consuming processes
alias lsmount='mount | column -t'                    # list mounted filesystems in columns
```


## Scripts

A shell script is an executable text file containing commands that the shell runs in sequence. Scripts are useful for tasks you perform more than once. Two common shell constructs to know:

- `(( ))`: Arithmetic evaluation
- `$()`: Run a command in a subshell and return its output to the current shell

#### Check for a file

Use `[ ! -f <path> ]` to test whether a file exists before running commands. For example, install Apache only if it is not already present:

```bash
#!/bin/bash
if [ ! -f /usr/sbin/apache2 ]; then
    sudo apt install -y apache2
    sudo apt install -y libapache2-mod-php8.1
    sudo a2enmod php8.1
    sudo systemctl restart apache2
fi
```

#### Check for a directory

Use `[ ! -d <path> ]` to test whether a directory exists:

```bash
#!/bin/bash
if [ ! -d /path/to/dir ]; then
    ...
fi
```

### Variables

When bash sees a string followed by `=`, it assigns the value on the right to a variable. Variable names are case-sensitive and contain no spaces around `=`.

| Command | Description |
| :------ | :---------- |
| `env` | View all current shell variables |
| `read <var>` | Read a line from STDIN and store it in `<var>` |

### Looping

Two loop types are available in bash:

- `while`: Executes a block of commands until a condition is no longer true
- `for`: Executes a block of commands once for every item in a set

A `while` loop that counts from 1 to 15:

```bash
#!/bin/bash
myvar=1
while [ $myvar -le 15 ]; do
    echo $myvar
    ((myvar++))
done
```

A `for` loop that iterates over a list of names:

```bash
#!/bin/bash
turtles="Donatello Leonardo Michelangelo Raphael"
for t in $turtles; do
    echo $t
done
```

### Backup script

This script uses `rsync` to perform an incremental backup. It installs `rsync` if it is not present, then syncs `/src` to `/target`. Deleted files are moved to a dated subdirectory rather than removed permanently. Key options:

| Option | Description |
| :----- | :---------- |
| `-a` | Archive mode: retains file metadata |
| `-v` | Verbose output |
| `-b` | Backup mode: renames duplicate files in the target to prevent overwrites |
| `--delete` | Remove files from the target that no longer exist in the source |
| `--backup-dir` | Directory where deleted or overwritten files are moved |

```bash
#!/bin/bash
curdate=$(date +%m-%d-%Y)

if [ ! -f /usr/bin/rsync ]; then
    sudo apt install -y rsync
fi

rsync -avb --delete --backup-dir=/backup/incremental/$curdate /src /target
```