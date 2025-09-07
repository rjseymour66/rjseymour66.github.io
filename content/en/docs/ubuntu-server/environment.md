---
title: "Environment"
linkTitle: "xEnvironment"
# weight: 1000
# description:
---

## Environment setup

- After user authenticates to the system and before the bash shell prompt displays, the user environment is configured.
- When you start a bash shell, bash checks several files for configuration, called environment files (also called startup files)
- Can start a shell in 3 ways:
  - Default login (login to server w no GUI)
  - Interactive shell spawned as subshell (such as from a GUI)
  - non-interactive shell, such as when running a script

### Environment variables

Store info about the current shell session

```bash
# user less pager
set
env
printenv
# view what is stored in env var
$ENVVARNAME
```
### Environment files

- Generally populated from the `/etc/skel` file. Users can edit these files after they are in their user account.
- The first file found in the following order is ran, and the rest are ignored:
  - `.bash_profile`
  - `.bash_login`
  - `.profile`
    > `.bashrc` is run from a file in the preceding list. It is also always run when there is a non-interactive shell started

### Global files

Global files:
- `/etc/profile`
- `/etc/profile.d` files
- `/etc/bash` or `/etc/bash.bashrc` file (depends on distro)

> Do not change global files. You can create a custom env file with an `.sh` extension and place it in `/etc/profile.d`. Files in this directory are run during bash login.


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

## Enable bash completion globally

Uncomment some lines in the global `bash.bashrc` file:

```bash
sudo vim /etc/bash.bashrc 

# opened file - uncomment if lines
# enable bash completion in interactive shells
if ! shopt -oq posix; then
  if [ -f /usr/share/bash-completion/bash_completion ]; then
    . /usr/share/bash-completion/bash_completion
  elif [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
  fi
fi

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

## dot files

Dot files are hidden config files. The personal config files are hidden in `/home/`:

### `.bashrc`

Runs every time you open a bash terminal.

You can source `.profile` from this file. In `.profile`, you can load other dot config files to keep your env clean.

```bash
# if you have a terminal prompt, load profile
[ -n "$PS1" ] && source ~/.profile
```

### `.profile`

Load OS-specific settings or other dot files:

```bash
# if file exists and is readable, source it
for file in ~/.{exports,aliases,functions,extra}; do
    [ -r "$file" ] && source "$file"
done
unset file
```

### `.aliases`

File dedicated to aliases only:

```bash
# git aliases
alias gs="git status"
alias gb="git branch"
alias gc="git commit -am"
alias gp="git push"
```

### `.exports`

Set environment variables:

```bash
# set GOPATH
export GOPATH="$HOME/go-workspace"
```

### `.vimrc`

vim configuration file. For more info, open vim and enter `:help vimrc-intro`:

### `.gitignore`

Ignore files from git tracking.

## Prompt Statement (PS)

Add customizations to `.bashrc` to make them permanent.

### PS1

Default interactive prompt. See this [phoenixNap](https://phoenixnap.com/kb/change-bash-prompt-linux) page for a complete list of options.

### PS2

Continuation interactive prompt. This is the prompt displayed when you enter a multi-line command:

```bash
export PS2="continue-> "
ls /etc/ \
continue -> line 2 \
continue -> line 3 \
continue -> end;
```

### PS3

For `select` command prompts, it lets you replace the `#?`. I don't see a good reason to use this often.

Read this [Geek Stuff](https://www.thegeekstuff.com/2008/09/bash-shell-take-control-of-ps1-ps2-ps3-ps4-and-prompt_command/) article for an example.


### PS4

Customize the debug mode (`set -x`) output for shell scripts. By default, it outputs `++` beside each line of the script. You can customize it to output the script name and line number:

```bash
# default PS4
+ echo 'PS4 demo script'
PS4 demo script
+ ls -l /etc
+ wc -l
229
+ du -sh /home/linuxuser
1.4G	/home/linuxuser

# custom PS4: export PS4='Line: $LINENO++ '
Line: 7++ echo 'PS4 demo script'
PS4 demo script
Line: 8++ ls -l /etc
Line: 8++ wc -l
229
Line: 9++ du -sh /home/linuxuser
1.4G	/home/linuxuser
```

### PROMPT_COMMAND

Executed right before displaying `PS1`:

```bash
export PROMPT_COMMAND="date +%k:%m:%S"
11:12:35
# next command
ls -1
find-files
...
11:12:43

# display on same line as PS1
export PROMPT_COMMAND="echo -n [$(date +%k:%m:%S)]"k:%m:%S)]"
[11:12:35]user@machine:~$ 

```

## vim

There are two modes: "normal" and "insert".
- Pres ESC twice to return to normal mode

### Editing

| Command | Description         |
| ------- | ------------------- |
| `i`     | insert mode         |
| `w`     | write               |
| `u`     | undo                |
| `wq`    | write and quit      |
| `q`!    | quit without saving |

### Basic Navigation

| Command    | Description                                |
| ---------- | ------------------------------------------ |
| `h`        | Move left                                  |
| `j`        | Move down                                  |
| `k`        | Move up                                    |
| `l`        | Move right                                 |
| `w`        | Move to the beginning of the next word     |
| `3w`       | move to the 3rd word to the right          |
| `3wd`      | delete the 3 words to the right            |
| `b`        | Move to the beginning of the previous word |
| `3b`       | move to the 3rd word to the left           |
| `e`        | Move to the end of the current word        |
| `0`        | Move to the beginning of the line          |
| `$`        | Move to the end of the line                |
| `gg`       | Go to the first line of the file           |
| `G`        | Go to the last line of the file            |
| `Ctrl + u` | Move up half a screen                      |
| `Ctrl + d` | Move down half a screen                    |
| `(`        | next sentence beginning                    |
| `)`        | prev sentence beginning                    |

### Editing

| Command    | Description                                                                                                                   |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `i`        | Insert mode at the cursor                                                                                                     |
| `I`        | Insert mode at the beginning of the line                                                                                      |
| `a`        | Append mode after the cursor                                                                                                  |
| `A`        | Append mode at the end of the line                                                                                            |
| `o`        | Open a new line below the current line                                                                                        |
| `O`        | Open a new line above the current line                                                                                        |
| `r`        | replace current char with next char you enter                                                                                 |
| `x`        | Delete the character under the cursor                                                                                         |
| `X`        | Delete the character before the cursor                                                                                        |
| `dd`       | Delete the current line                                                                                                       |
| `99dd`     | Delete 99 lines                                                                                                               |
| `dw`       | Delete from the cursor to the end of the word                                                                                 |
| `3wd`      | Delete the 3 words to the right                                                                                               |
| `db`       | Delete word to the left                                                                                                       |
| `3db`      | Delete the 3 words to the left                                                                                                |
| `d$`       | Delete from the cursor to the end of the line                                                                                 |
| `d0`       | Delete from the cursor to the beginning of the line                                                                           |
| `das`      | Delete current sentence                                                                                                       |
| `yy`       | Copy (yank) the current line                                                                                                  |
| `3yy`      | copy three lines, starting from the line where the cursor is positioned.                                                      |
| `yw`       | Copy (yank) the current word                                                                                                  |
| `yiw`      | copy the current word.                                                                                                        |
| `y$`       | Copy (yank) to the end of the line                                                                                            |
| `y%`       | copy to the matching character. By default supported pairs are (), {}, and []. Useful to copy text between matching brackets. |
| `y^`       | copy everything from the cursor to the start of the line.                                                                     |
| `p`        | Paste after the cursor                                                                                                        |
| `5p`       | paste 5 times                                                                                                                 |
| `P`        | Paste before the cursor                                                                                                       |
| `u`        | Undo                                                                                                                          |
| `Ctrl + r` | Redo                                                                                                                          |


### Visual Mode

| Command    | Description                                             |
| ---------- | ------------------------------------------------------- |
| `v`        | Start visual mode                                       |
| `V`        | Start visual line mode                                  |
| `vjjj`     | highlight current line and 3 lines below (number of js) |
| `vjjjyy`   | hightlight and copy three lines                         |
| `Ctrl + v` | Start visual block mode                                 |
| `y`        | Yank (copy) the selected text                           |
| `d`        | Delete the selected text                                |
| `>`        | Indent the selected text                                |
| `<`        | Un-indent the selected text                             |


### Search and Replace

| Command          | Description                                                |
| ---------------- | ---------------------------------------------------------- |
| `/pattern`       | Search for pattern                                         |
| `?pattern`       | Search backward for pattern                                |
| `n`              | Repeat the search in the same direction                    |
| `N`              | Repeat the search in the opposite direction                |
| `:%s/old/new/g`  | Replace all occurrences of old with new in the entire file |
| `:%s/old/new/gc` | Replace all occurrences with confirmation                  |

### Working with Multiple Files

| Command       | Description               |
| ------------- | ------------------------- |
| `:e filename` | Open a file               |
| `:w`          | Save the current file     |
| `:w filename` | Save as filename          |
| `:q`          | Quit Vim                  |
| `:wq`         | Save and quit             |
| `:q!`         | Quit without saving       |
| `:bn`         | Go to the next buffer     |
| `:bp`         | Go to the previous buffer |
| `:bd`         | Delete a buffer           |

### Useful Commands

| Command             | Description                       |
| ------------------- | --------------------------------- |
| `:set nu`           | Show line numbers                 |
| `:set nonu`         | Hide line numbers                 |
| `:syntax on`        | Enable syntax highlighting        |
| `:syntax off`       | Disable syntax highlighting       |
| `:set paste`        | Enable paste mode                 |
| `:set nopaste`      | Disable paste mode                |
| `:set tabstop=4`    | Set tab width to 4 spaces         |
| `:set expandtab`    | Convert tabs to spaces            |
| `:set shiftwidth=4` | Set indentation width to 4 spaces |

### Exiting Vim

| Command       | Description         |
| ------------- | ------------------- |
| `:w`          | Save the file       |
| `:q`          | Quit Vim            |
| `:wq` or `ZZ` | Save and quit       |
| `:q!`         | Quit without saving |


### .`vimrc`

Config file for `vim`:
- In `/home/.vimrc`, but there is also a system-wide `/etc/vimrc`

```bash
# Start vim without reading .vimrc
vim -u NONE
```