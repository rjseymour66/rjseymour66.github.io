+++
title = 'Set up a sysadmin user'
date = '2026-04-18T00:00:00-04:00'
weight = 70
draft = false
+++

This page walks through creating a dedicated administrator account on Ubuntu, granting it the right level of privilege, securing SSH access, and configuring a shell environment suited to day-to-day admin work. Complete every step before closing your current session.

{{< admonition "Keep a root session open" warning >}}
Before making authentication or SSH changes, open a second terminal with an active root session. Test each change from a third terminal before closing anything. Locking yourself out of a remote server is difficult to recover from without console access.
{{< /admonition >}}

## Create the account

Use `adduser` rather than `useradd`. It runs interactively, sets a password, creates the home directory, and copies `/etc/skel` in one step:

```bash
sudo adduser adminuser
```

`adduser` prompts for a password and optional contact fields. Fill in the full name; leave the rest blank.

Verify the account:

```bash
getent passwd adminuser
adminuser:x:1001:1001:Admin User,,,:/home/adminuser:/bin/bash
```

## Grant sudo privileges

Add the account to the `sudo` group. Members of this group can run any command as root:

```bash
sudo usermod -aG sudo adminuser
```

Confirm the membership:

```bash
groups adminuser
adminuser : adminuser sudo
```

Group membership takes effect at the next login. The change does not apply to any sessions already running under that account.

## Restrict sudo with a drop-in file

The `sudo` group grants full access by default. For accounts that need a narrower scope, create a drop-in file in `/etc/sudoers.d/` instead of editing `/etc/sudoers` directly. Drop-in files are easier to audit and less risky to edit.

Always use `visudo` to edit sudoers files. It validates syntax before saving and prevents you from writing a broken file:

```bash
sudo visudo -f /etc/sudoers.d/adminuser
```

For a full-access admin who must enter a password every time, add:

```bash
adminuser ALL=(ALL:ALL) ALL
```

For a deployment account that only needs to restart a specific service without a password prompt, use:

```bash
deployuser ALL=(ALL) NOPASSWD:/usr/bin/systemctl restart myapp.service
```

Set the drop-in file to read-only after saving:

```bash
sudo chmod 440 /etc/sudoers.d/adminuser
```

Verify that `sudo` works before continuing:

```bash
su - adminuser
sudo -v
```

## Set up SSH key authentication

Password-based SSH logins are vulnerable to brute-force attacks. Use key-based authentication instead.

### Add the public key

On the admin account, create the `.ssh` directory and set strict permissions:

```bash
sudo -u adminuser mkdir -p /home/adminuser/.ssh
sudo chmod 700 /home/adminuser/.ssh
```

Add the administrator's public key to the `authorized_keys` file:

```bash
sudo -u adminuser tee /home/adminuser/.ssh/authorized_keys <<'EOF'
ssh-ed25519 AAAA... adminuser@workstation
EOF
sudo chmod 600 /home/adminuser/.ssh/authorized_keys
```

Confirm ownership on both files:

```bash
ls -la /home/adminuser/.ssh/
drwx------ 2 adminuser adminuser 4096 Apr 18 14:00 .
drwxr-x--- 4 adminuser adminuser 4096 Apr 18 14:00 ..
-rw------- 1 adminuser adminuser  100 Apr 18 14:00 authorized_keys
```

### Verify key login

From the administrator's workstation, test key-based login before disabling passwords:

```bash
ssh adminuser@server
```

Confirm you land in the right home directory and can run `sudo`:

```bash
pwd              # /home/adminuser
sudo -v          # should succeed without an SSH password prompt
```

### Harden sshd

After confirming key login works, edit `/etc/ssh/sshd_config` to disable root login and password authentication:

```bash
sudo vim /etc/ssh/sshd_config
```

Set or confirm these directives:

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```

Validate the configuration before restarting:

```bash
sudo sshd -t
```

If `sshd -t` returns no errors, restart the service:

```bash
sudo systemctl restart ssh
```

Test login again from a new terminal to confirm the changes did not break access.

## Configure the shell environment

A useful admin prompt shows the time, account, host, and working directory. The timestamp helps correlate terminal activity with log entries.

### Set the prompt

Add the following to `/home/adminuser/.bashrc`:

```bash
PS1='[\[\e[33m\]\t\[\e[0m\]] \[\e[32m\]\u@\h\[\e[0m\]:\[\e[34m\]\w\[\e[0m\]\$ '
```

This produces a prompt like:

```
[14:32:01] adminuser@webserver:~/configs$
```

| Color  | Element |
| :----- | :------ |
| Yellow | Timestamp (`\t`) |
| Green  | `user@host` (`\u@\h`) |
| Blue   | Working directory (`\w`) |
| White  | `$` (or `#` when root) |

### Extend history

By default, bash keeps a short history and overwrites it between sessions. Extend it and record timestamps so you can reconstruct what happened and when:

```bash
HISTSIZE=10000
HISTFILESIZE=20000
HISTTIMEFORMAT="%F %T "
HISTCONTROL=ignoredups:erasedups
shopt -s histappend
```

`histappend` appends to the history file rather than overwriting it when the shell exits. `erasedups` removes duplicate entries while preserving the most recent occurrence.

### Set the default editor and add aliases

```bash
export EDITOR=vim
export VISUAL=vim

alias ll='ls -lh'
alias la='ls -lah'
alias grep='grep --color=auto'
alias df='df -h'
alias free='free -h'
alias ports='ss -tlnp'
alias syslog='sudo journalctl -f'
```

`ports` and `syslog` are shortcuts for the two most common quick checks during troubleshooting.

### Apply the changes

Reload `.bashrc` to apply the changes to the current session:

```bash
source ~/.bashrc
```

## Verify the setup

Before closing any existing sessions, confirm everything works from a clean login:

1. Open a new terminal and connect as the admin account.
2. Confirm the prompt shows the timestamp, username, and hostname.
3. Run `sudo -v` to verify `sudo` access with a password prompt.
4. Run `id` to confirm `sudo` group membership.
5. Run `ssh adminuser@server` from the workstation to confirm key login works.

If any step fails, investigate from the open root session rather than logging out.
