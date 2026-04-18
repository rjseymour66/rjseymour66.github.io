---
title: "Shell efficiency"
weight: 50
description: >
  History, job control, command substitution, xargs, one-liners, and time-saving techniques.
---

The shell rewards investment in learning its efficiency features. Most of what makes an experienced administrator fast is not typing speed -- it is knowing which tools to reach for and how to combine them. History navigation lets you recall and fix commands in seconds instead of retyping them. Job control lets you run long tasks in the background while you continue working. Process substitution and `xargs` let you feed the output of one command into another in ways that pipes alone cannot.

This page covers the techniques that experienced practitioners reach for every day.

## Command history

Bash records every command you run to a history file.

*HISTSIZE* controls how many commands are saved to memory. Set it to `-1` for unlimited history.

*HISTFILE* is the path where bash writes history when a shell exits. The default is `~/.bash_history`.

*HISTCONTROL* controls which commands are saved. Setting it to `ignoredups` prevents saving identical consecutive commands:

```bash
HISTSIZE=10000
HISTCONTROL=ignoredups
```

Add both lines to `~/.bashrc` to make them permanent. Each shell maintains its own history and writes it to `HISTFILE` when it exits.

Basic history commands:

```bash
history                         # print all shell history
history 5                       # print the last 5 commands
history | less                  # browse history
history | grep -w ssh           # find all SSH commands
history -c                      # clear history for the current shell
```

### History expansion

History expansion lets you recall and rerun commands without retyping them. The shell expands these expressions before executing the command:

```bash
!!                              # run the most recent command
sudo !!                         # re-run the last command with sudo

!grep                           # run the most recent command that began with grep
!?deploy?                       # run the most recent command that contained "deploy"
!42                             # run command number 42 from history
!-2                             # run the command executed 2 commands ago
!-2:p                           # print (but do not run) the command 2 commands ago

!$                              # final argument of the previous command
!*                              # all arguments of the previous command
```

A common pattern for safe deletions: list the files you want to remove, verify the output, then delete using `!$`:

```bash
ls /tmp/*.log                   # confirm which files match
rm !$                           # delete them using the previous command's arguments
```

Fix a typo in the previous command with caret syntax:

```bash
git commit -m "fix tyop"
^tyop^typo                      # corrects the typo and reruns the command
```

Alternatively, apply a substitution with history expansion:

```bash
!!:s/tyop/typo/
```

### Incremental history search

Press `Ctrl+R` to start an interactive reverse search through history. Type a few characters and bash shows the most recent matching command. Press `Ctrl+R` again to cycle through older matches. Press `Enter` to run the command or `Ctrl+G` to cancel:

```bash
(reverse-i-search)`deploy': ./deploy.sh --env production
```

### vim- and emacs-style command editing

By default, bash uses emacs-style key bindings for command-line editing. Press `Esc` to enter editing mode. Switch to vi-style editing with `set -o vi`. Add the setting to `.bashrc` to make it permanent:

```bash
set -o vi       # vi-style editing
set -o emacs    # emacs-style editing (default)
```

## Command lists

### Conditional lists

Conditional lists run each command based on the success or failure of the previous one. An exit code of `0` means success and any non-zero value means failure.

`&&` runs the next command only if the previous one succeeded. `||` runs the next command only if the previous one failed:

```bash
git add . && git commit -m "update config" && git push     # chain: stop on first failure
cd /app || mkdir /app                                       # if cd fails, create the directory
cd /app || exit 1                                           # exit the script if the directory is inaccessible
```

Combine both operators to handle a create-or-navigate pattern:

```bash
cd /app/logs || mkdir /app/logs && cd /app/logs || echo "Failed to set up log directory"
```

### Unconditional lists

Separate commands with `;` to run them one after another regardless of whether each succeeds. Only the exit status of the last command is captured in `$?`:

```bash
sleep 7200; cp -a ~/important-files /mnt/backup-drive      # wait 2 hours, then back up
```

## Substitution techniques

### Command substitution

Command substitution runs a command and replaces the expression with its output. The preferred syntax is `$()`:

```bash
$(command)                              # preferred syntax
`command`                               # old syntax -- do not use

mv $(grep -l "draft" *.txt) /archive/                           # move all draft files
echo "Today is $(echo $(date +%A) | tr a-z A-Z)!"               # Today is MONDAY!
DISK_FREE=$(df -h / | awk 'NR==2 {print $4}')                   # capture free disk space
```

In scripts, store command output in a variable rather than running the command twice:

```bash
UPTIME=$(uptime -p)
LOG_COUNT=$(ls /var/log/*.log | wc -l)
echo "System $UPTIME -- $LOG_COUNT log files found"
```

### Process substitution

Process substitution runs a command and presents its output as if it were a file. This is useful when a command requires file arguments but you want to feed it the output of another command:

```bash
<(command)          # read the output of a command as a file
```

`diff` requires two file arguments, but with process substitution you can compare the output of two commands directly:

```bash
diff <(ls /etc | sort) <(ls /etc.backup | sort)         # compare two directory listings
diff <(ssh host1 ls /etc) <(ssh host2 ls /etc)          # compare remote directories
```

Check which expected files are missing from a directory:

```bash
diff <(ls *.jpg | sort -n) <(seq 1 1000 | sed 's/$/.jpg/') \
| grep '^>' \
| cut -c3-
```

Enable process substitution if it is not already active:

```bash
set -o posix        # if process substitution is unavailable, this may resolve it
```

## xargs

`xargs` builds and runs commands from stdin. It reads whitespace-separated strings from stdin and appends them to a command template:

```bash
find . -type f -name "*.py" -print0 | xargs -0 wc -l       # count lines in all Python files
```

Pass `-n` to control how many arguments are appended per command:

```bash
ls | xargs -n1 echo       # run echo once per file
ls | xargs -n2 echo       # run echo with two filenames at a time
```

Pass `-I {}` to control where the input appears in the command. The `{}` placeholder is replaced with each input string:

```bash
cat server_list.txt | xargs -I {} ssh {} uptime             # run uptime on each server
find . -name "*.log" | xargs -I {} cp {} /backup/           # back up each log file
```

Pass `-0` to change the input separator from whitespace to the null character. Combine this with `find -print0` to handle filenames that contain spaces:

```bash
find . -type f -name "*.txt" -print0 | xargs -0 grep -l "TODO"
```

## Job control

### Background jobs

Append `&` to a command to run it in the background. The shell prints the job ID and process ID, then immediately returns the prompt:

```bash
./run_report.sh &                               # [1] 12345
./compress_logs.sh & ./sync_files.sh &          # run two jobs simultaneously
```

When running a background job, redirect both stdout and stderr so output does not interrupt your terminal:

```bash
./long_backup.sh &> /var/log/backup.log &
```

### Job control commands

The following table shows commands for managing jobs:

| Command    | Effect |
|:-----------|:-------|
| `jobs`     | List all jobs in the current shell |
| `Ctrl+Z`   | Suspend the running foreground job |
| `bg`       | Resume the most recently suspended job in the background |
| `bg %n`    | Resume job number n in the background |
| `fg %n`    | Bring background job number n to the foreground |
| `kill %n`  | Send SIGTERM to background job number n |

A common workflow: start a command, realize it will take a long time, suspend it, and resume it in the background:

```bash
./large_data_import.sh
# realize this will take 30 minutes
Ctrl+Z
# [1]+  Stopped    ./large_data_import.sh
bg %1
# [1]+ ./large_data_import.sh &
# now you can continue working
```

Bring a background job back when you want to check on it:

```bash
jobs                            # list jobs and their numbers
fg %1                           # bring job 1 to the foreground
```

### Subshells for pipeline isolation

Enclose commands in parentheses to run them in a subshell. Changes to the working directory or variables inside the subshell do not affect the parent shell:

```bash
(cd /var/log && tar -czf /tmp/logs.tar.gz .)    # changes directory without affecting your shell
```

This is useful in pipeline steps where you need to change directories temporarily.

## Running commands as strings

### Piping to bash

`bash` reads commands from stdin, so you can construct commands as text and pipe them to bash for execution. This is useful for running multiple similar commands:

```bash
ls -1 ??* | sed 's/^\(.\)\(.*\)$/mv \1\2 \1/' | bash
```

The previous command moves every file in the current directory into a subdirectory named after the first letter of the filename. For example, `apple` moves to `a/`, and `banana` moves to `b/`. The `??*` pattern ensures files with names of at least two characters are processed.

Steps to build it:

1. Create the subdirectories:
   ```bash
   mkdir {a..z}
   ```

2. Build the `mv` commands with `sed` and preview them:
   ```bash
   ls -1 ??* | sed 's/^\(.\)\(.*\)$/mv \1\2 \1/' | less
   ```

3. When satisfied, execute:
   ```bash
   ls -1 ??* | sed 's/^\(.\)\(.*\)$/mv \1\2 \1/' | bash
   ```

### bash -c for sudo redirection

When you run `sudo` with output redirection, the shell evaluates the redirection before applying sudo, so the redirection runs as the current user rather than root. Pass the entire command to `bash -c` to apply sudo to the whole expression:

```bash
sudo bash -c 'echo "new config" > /etc/app/config.conf'
```

Without `bash -c`, the file write would fail with a permission error even though `sudo` was specified.

## Building one-liners

A one-liner is a complex bash command that solves a complete problem in a single pipeline. The recommended process for building one:

1. Write a command that produces the initial data you want to work on.
2. Run it and inspect the output.
3. Recall the command with history (`!!` or `Ctrl+R`) and extend it.
4. Repeat until the full pipeline produces the correct result.

For testing intermediate steps, insert `tee /tmp/checkpoint.txt` into the pipeline to see what the data looks like at that point without breaking the pipeline.

A one-liner that renames a set of image files by shifting their numbers:

```bash
paste <(echo {1..10}.jpg | sed 's/ /\n/g') \
      <(echo {0..9}.jpg | sed 's/ /\n/g') \
| sed 's/^/mv /' \
| bash
```

What each step does:

- `echo {1..10}.jpg | sed 's/ /\n/g'` generates `1.jpg` through `10.jpg`, one per line
- `echo {0..9}.jpg | sed 's/ /\n/g'` generates `0.jpg` through `9.jpg`, one per line
- `paste` prints both lists side by side
- `sed 's/^/mv /'` prepends `mv` to each line
- `bash` executes the resulting `mv` commands

### Checking matched pairs of files

Confirm that every `.jpg` in a directory has a corresponding `.txt` file:

```bash
diff <(ls *.jpg | sed 's/\.[^.]*$//') <(ls *.txt | sed 's/\.[^.]*$//') \
| grep '^[<>]' \
| awk '/^</{print $2 ".jpg"} /^>/{print $2 ".txt"}'
```

### Generating test files

Generate 1,000 text files with random content drawn from the system dictionary:

```bash
yes 'shuf -n $RANDOM -o $(pwgen -N1 10).txt /usr/share/dict/words' \
| head -n 1000 \
| bash
```

Generate 1,000 empty files with random lowercase names:

```bash
grep '^[a-z]*$' /usr/share/dict/words \
| shuf \
| head -n 1000 \
| xargs -I {} touch {}.txt
```

## Time-saving techniques

### Open a file in vim from less

When you are viewing a file with `less`, press `v` to open it immediately in your default editor. When you exit the editor, you return to `less`.

Set your default editor in `.bashrc`:

```bash
export VISUAL=vim
export EDITOR=vim
```

### Edit all files that match a grep search

Open every file that contains a given string in vim in one command. Cycle through the files with `:bn` (next) and `:bp` (previous):

```bash
vim $(grep -l "TODO" *.go)                                              # files in current directory
vim $(grep -lr "deprecated" src/)                                       # search a directory tree
vim $(find . -type f -print0 | xargs -0 grep -l "fixme")               # larger directory trees
```

### Process a file one line at a time

Read a file line by line with `cat` and a `while` loop:

```bash
cat server_list.txt | while read -r line; do
    echo "Processing: $line"
    ssh "$line" uptime
done
```

### Clipboard control

In Linux, the clipboard system is part of X selections. There are two selection types:

- *Clipboard*: The standard clipboard. Content is copied explicitly (Ctrl+C) and pasted explicitly (Ctrl+V).
- *Primary selection*: Automatically populated when you highlight text in a window. Paste it with the middle mouse button or Shift+Insert.

`Shift+Insert` pastes the primary selection directly into the terminal.

### Browser keyboard shortcuts

The following table shows common browser shortcuts useful when working alongside the command line:

| Shortcut    | Action |
|:------------|:-------|
| `Ctrl+N`    | Open new window |
| `Ctrl+T`    | Open new tab |
| `Ctrl+W`    | Close current tab |
| `Ctrl+Tab`  | Cycle through tabs |
| `Ctrl+L`    | Jump to address bar |

Open a browser from the terminal and redirect all diagnostic output to suppress clutter:

```bash
firefox &> /dev/null &
google-chrome &> /dev/null &

firefox --new-window https://docs.example.com
firefox --private-window https://example.com
```

### Download files with curl and wget

`curl` prints the response body to stdout. `wget` saves the response to a file:

```bash
curl https://example.com/api/status                 # print response to terminal
wget https://example.com/releases/app-1.0.tar.gz    # save to a file
curl -O https://example.com/file.tar.gz             # save to a file using the remote filename
```
