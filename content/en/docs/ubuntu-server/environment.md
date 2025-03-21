---
title: "Environment"
linkTitle: "Environment"
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

```bash
#-------------------------------
# TEXT
i # insert mode
w # write
u # undo
wq # write and quit
q! # quit without saving

a # begin typing after the cursor
o # insert new line below and enter text
O # insert new line on current line enter text - existing line moves down
x # delete current character
r # replace current char with next char you enter
dd # delete lines
dw # delete word to the right
3wd # delete the 3 words to the right
db # delete word to the left
3bd # delete the 3 words to the left
99dd # delete 99 lines
d0 # delete to the beginning of the sentence
d$ # delete to the end of the sentence
das # delete current sentence

33dd # delete 33 lines
u # undo

#-------------------------------
# NAVIGATION
gg # go to top of file
G # go to bottom of file
w # skip to next wordj
dw # delete word to the rightkk
3w # move to the 3rd word to the right
3wd # delete the 3 words to the right
b # skip to previous word
db # delete word to the left
3b # move to the 3rd word to the left
3bd # delete the 3 words to the left
( # next sentence beginning
) # prev sentence beginning

#-------------------------------
# SEARCHING
/<pattern> # search file for pattern
n # after search, step forward through <pattern> occurrences
N # after search, step backward through <pattern> occurrences

#-------------------------------
# CUT and PASTE
yy # yank text (copy)
p # after delete, paste the
5p # paste 5 times
yy # copy the current line, including the newline character.
3yy # copy three lines, starting from the line where the cursor is positioned.
y$ # copy everything from the cursor to the end of the line.
y^ # copy everything from the cursor to the start of the line.
yw # copy to the start of the next word.
yiw # copy the current word.
y% # copy to the matching character. By default supported pairs are (), {}, and []. Useful to copy text between matching brackets.

#-------------------------------
# VISUAL MODE
v # highlight current line
V # visual mode, select text by line
CTRL + V # visual mode, select text by char
vjjj # highlight current line and 3 lines below (number of js)
vjjjyy # hightlight and copy three lines
```

### .`vimrc`

Config file for `vim`:
- In `/home/.vimrc`, but there is also a system-wide `/etc/vimrc`

```bash
# Start vim without reading .vimrc
vim -u NONE


```