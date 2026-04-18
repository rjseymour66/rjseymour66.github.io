+++
title = 'Setup'
date = '2025-09-07T18:56:55-04:00'
weight = 10
draft = false
+++

## Shell environment

When you log in, bash configures your environment before displaying the prompt. It reads a series of configuration files called environment files or startup files. Bash starts in one of three ways:

- Default login: a text-only login with no GUI
- Interactive shell: a subshell spawned from a GUI terminal
- Non-interactive shell: a shell started to run a script

### Environment variables

Environment variables store information about the current shell session, such as the current user, home directory, and default editor. View them with any of the following commands:

| Command    | Description                                                   |
| :--------- | :------------------------------------------------------------ |
| `set`      | Display all shell variables and functions (uses `less` pager) |
| `env`      | Display all exported environment variables                    |
| `printenv` | Display all or specific environment variables                 |
| `$VARNAME` | Print the value of a specific variable                        |
### Environment files

User environment files are copied from `/etc/skel` at account creation. Users can edit them afterward. Bash reads these files in order and stops at the first one it finds:

1. `.bash_profile`
2. `.bash_login`
3. `.profile`

`.bashrc` is not read directly by bash at login. Instead, one of the files above sources it explicitly. For example, a typical `.bash_profile` contains:

```bash
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi
```

This means `.bashrc` runs at every interactive login. It also runs automatically when a non-interactive shell starts, making it the right place for aliases, functions, and environment settings you want available in every session.

### Global files

Global environment files apply to all users on the system:

- `/etc/profile`
- `/etc/profile.d/` (individual `.sh` files, run during login)
- `/etc/bash` or `/etc/bash.bashrc` (varies by distribution)

{{< admonition "Do not edit global files directly" warning >}}
Editing `/etc/profile` or `/etc/bash.bashrc` directly risks breaking the environment for all users if you introduce a syntax error. Instead, create a custom file with a `.sh` extension in `/etc/profile.d/`. All `.sh` files in that directory are sourced automatically during bash login, so your settings apply system-wide without touching the base files. This also makes it easy to remove your customizations by deleting the file.

For example, create a file to set a custom PATH for all users:

```bash
sudo vim /etc/profile.d/custom-path.sh

export PATH="$PATH:/opt/myapp/bin"
```
{{< /admonition >}}


## Set default text editor

Ubuntu sets `nano` as the default editor. Run `update-alternatives` to select a different one. The command presents a numbered list of installed editors. Enter the number for your preferred editor and press Enter:

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

Bash completion is often disabled by default in `/etc/bash.bashrc`. To enable it for all users, uncomment the following block in that file:

```bash
sudo vim /etc/bash.bashrc

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

`hostnamectl` sets the system hostname. The change takes effect immediately and persists across reboots:

```bash
hostnamectl set-hostname mylittlecloudbox
```

## Set timezone

The system timezone affects scheduled tasks and timestamps in `/var/log`. Use `timedatectl` to view and change it.

View the current timezone:

```bash
timedatectl
               Local time: Sun 2024-12-01 17:15:21 UTC
           Universal time: Sun 2024-12-01 17:15:21 UTC
                 RTC time: Sun 2024-12-01 17:15:21
                Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

List available timezones:

```bash
timedatectl list-timezones
```

Set a new timezone and verify:

```bash
sudo timedatectl set-timezone America/New_York

timedatectl
               Local time: Sun 2024-12-01 12:15:57 EST
           Universal time: Sun 2024-12-01 17:15:57 UTC
                 RTC time: Sun 2024-12-01 17:15:57
                Time zone: America/New_York (EST, -0500)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

## Dot files

Dot files are hidden configuration files stored in your home directory. Their names begin with `.`, which causes them to be hidden from standard directory listings. Run `ls -a` to view them. Common dot files:

| File         | Description                                             |
| :----------- | :------------------------------------------------------ |
| `.bashrc`    | Runs every time you open a bash terminal                |
| `.profile`   | Loads OS-specific settings and other dot files at login |
| `.aliases`   | Dedicated file for shell aliases                        |
| `.exports`   | Sets environment variables                              |
| `.vimrc`     | Configuration file for vim                              |
| `.gitignore` | Specifies files to exclude from git tracking            |

### `.bashrc`

`.bashrc` runs every time you open an interactive bash terminal. A common pattern is to source `.profile` from `.bashrc`, then load other dot files from `.profile` to keep each file focused on one concern.

Source `.profile` only when an interactive terminal is present:

```bash
[ -n "$PS1" ] && source ~/.profile
```

### `.profile`

`.profile` runs at login and is a good place to load OS-specific settings or source other dot files. The following pattern sources each dot file only if it exists and is readable:

```bash
for file in ~/.{exports,aliases,functions,extra}; do
    [ -r "$file" ] && source "$file"
done
unset file
```

### `.aliases`

`.aliases` keeps all shell aliases in one place, separate from `.bashrc`. It is sourced by `.profile` at login. For example, a set of git shorthand aliases:

```bash
alias gs="git status"
alias gb="git branch"
alias gc="git commit -am"
alias gp="git push"
```

### `.exports`

`.exports` stores environment variable definitions, keeping them separate from other configuration. It is sourced by `.profile` at login. For example, set the Go workspace path:

```bash
export GOPATH="$HOME/go-workspace"
```

### `.vimrc`

`.vimrc` stores your personal vim configuration. Settings here apply every time you open vim. For a full reference, open vim and run `:help vimrc-intro`. See the [vim](../vim) page for common settings.

### `.gitignore`

`.gitignore` lists file patterns that git should not track. Place it in the root of a repository. For example, ignore compiled binaries and environment files:

```bash
*.out
.env
```

### Chain dot files together

At login, bash reads `.bashrc`, which sources `.profile`. `.profile` then sources each specialized dot file if it exists. This keeps each file focused on one concern.

`.bashrc` sources `.profile`:

```bash
# ~/.bashrc
[ -n "$PS1" ] && source ~/.profile
```

`.profile` sources the specialized dot files:

```bash
# ~/.profile
for file in ~/.{exports,aliases,functions,extra}; do
    [ -r "$file" ] && source "$file"
done
unset file
```

`.exports` defines environment variables:

```bash
# ~/.exports
export GOPATH="$HOME/go-workspace"
export PATH="$PATH:$GOPATH/bin"
```

`.aliases` defines shorthand commands:

```bash
# ~/.aliases
alias gs="git status"
alias gb="git branch"
alias gc="git commit -am"
alias gp="git push"
```

## Prompt Statement (PS)

The Prompt Statement variables control how bash displays prompts. Add customizations to `.bashrc` to make them permanent across sessions.

### PS1

`PS1` is the default interactive prompt displayed before every command. It supports escape sequences for dynamic content such as the username, hostname, and current directory. For a complete list of options, see the [phoenixNap bash prompt guide](https://phoenixnap.com/kb/change-bash-prompt-linux).

### PS2

`PS2` is the continuation prompt displayed when a command spans multiple lines. The default value is `>`. Set a custom value to make multi-line input easier to read:

```bash
export PS2="continue-> "
ls /etc/ \
continue-> line 2 \
continue-> line 3 \
continue-> end;
```

### PS3

`PS3` customizes the prompt shown by the `select` command, replacing the default `#?`. For a practical example, see the [Geek Stuff bash prompt guide](https://www.thegeekstuff.com/2008/09/bash-shell-take-control-of-ps1-ps2-ps3-ps4-and-prompt_command/).


### PS4

`PS4` customizes the prefix printed by debug mode. Debug mode runs a script with `set -x`, which causes bash to print each command before executing it. This is useful for tracing what a script does line by line. The default prefix is `+`. Set `PS4` to include the line number to make the output easier to follow.

Default output:

```bash
+ echo 'PS4 demo script'
PS4 demo script
+ ls -l /etc
+ wc -l
229
```

Set a custom `PS4` and compare the output:

```bash
export PS4='Line: $LINENO++ '

Line: 7++ echo 'PS4 demo script'
PS4 demo script
Line: 8++ ls -l /etc
Line: 8++ wc -l
229
```

### PROMPT_COMMAND

`PROMPT_COMMAND` holds a command that bash runs before displaying `PS1`. Use it to show dynamic information such as the current time before every prompt.

Print the time before each prompt on its own line:

```bash
export PROMPT_COMMAND="date +%k:%M:%S"
11:12:35
user@machine:~$ ls
...
11:12:43
```

Print the time on the same line as `PS1`:

```bash
export PROMPT_COMMAND="echo -n [$(date +%k:%M:%S)]"
[11:12:35]user@machine:~$
```

