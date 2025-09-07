---
title: "Bash shell"
linkTitle: "xShell"
weight: 50
# description:
---

View each user shell in the last field of the `/etc/passwd` file:
- system users are often set to `/bin/false/` or `/usr/sbin/nologin` to prevent someone from logging in as that user

```bash
cat /etc/passwd | cut -d: -f 1,7
...
dhcpcd:/bin/false
...
sshd:/usr/sbin/nologin
testuser:/bin/bash
```

## General command efficiency

```bash
<tab><tab>          # cycle through tab completion options
CTRL + a            # move cursor to beginning of line
CTRL + e            # move cursor to end of line
CTRL + l            # clears screen
CTRL + k            # deletes chars from cursor to end of line
CTRL + u            # deletes everything you typed and clears passwords
CTRL + w            # deletes word to left of cursor

CTRL + r            # cycle through command history for command
CTRL + xe           # open command in editor to change and run at exit

echo $?             # view exit code for previous command
```


## History

Shell history for previously run commands:
- If you do not want a command to be saved in history, prefix it with a space:
- History is stored in `.bash_history`

```bash
history                 # view bash history
!!                      # repeat previous command
!<num>                  # run command from history
history -d <num>        # delete <num> command from history
<space><cmd>            # do not save command in shell history (Ubuntu default)
HISTCONTROL=ignoreboth  # add to .bashrc to enable <space><cmd> feature on some distros
```

## aliases

Substitute commands for simpler commands:
- add to `.bashrc` to make permanent

```bash
alias                                               # view all aliases in current shell
unalias <alias-name>                                # remove <alias-name> from aliases
unalias -a                                          # remove all aliases in current session
alias cpu10='ps -L aux | sort -nr -k 3 | head -10'  # top 10 cpu-consuming processes
alias mem10='ps -L aux | sort -nr -k 4 | head -10'  # top 10 memory-consuming processes
alias lsmount='mount | column -t'                   # list mounted fs in column format
```


## Scripts

Write a script for something that you plan to do more than once:
- Executable text file that contains commands that your shell will execute one by one
- Double shell is for arithmetic: `(( ))`
- Run in a subshell and get output in current shell: `$()`

```bash
# check to see if file is present
#!/bin/bash
# Install Apache if it isn't already present
if [ ! -f /usr/sbin/apache2 ]; then
        sudo apt install -y apache2
        sudo apt install -y libapache2-mod-php8.1
        sudo a2enmod php8.1
        sudo systemctl restart apache2
fi

# check for dir instead
if [ ! -d /path/to/dir ]; then
        ...
fi

```

### Variables

When bash sees a string followed by an equals sign, it 

```bash
env                 # view default shell variables
read <var>          # store next STDIN in <var>
```

### Looping

`while` loop
: Executes a statement until a condition is met

`for` loop
: Executes a statement for every item in a set

```bash
# --- WHILE LOOP --- #
#!/bin/bash

myvar=1
while [ $myvar -le 15 ]; do
	echo $myvar
	((myvar++))                 # INCREMENT VAR
done


# --- FOR LOOP --- #
#!/bin/bash
turtles="Donatello Leonardo Michelangelo Raphael"
for t in $turtles; do
	echo $t
done

```

### Backup script

```bash
rsync -avb --delete --backup-dir=/backup/incremental/08-17-2022 /src /target
-a # archive - retain metadata for copied file
-v # verbose
-b # backup mode, rename duplicate files in target so no overwrites
--delete # delete any files in /target that are not present in /src
--backup-dir # if a file is deleted, send to this dir

#!/bin/bash
curdate=$(date +%m-%d-%Y)

if [ ! -f /usr/bin/rsync ];then
	sudo apt install -y rsync
fi

rsync -avb --delete --backup-dir=/backup/incremental/$curdate /src /target
```