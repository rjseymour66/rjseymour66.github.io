---
title: "Users and groups"
linkTitle: "Users and groups"
# weight: 1000
# description:
---
There are three types of Linux users:
- root
- sudoers
- regular users

## sudo

[How to edit the sudoers file](https://www.digitalocean.com/community/tutorials/how-to-edit-the-sudoers-file)

Stands for "substitute user". Do not log in as `root`. Assign superuser privileges to a regular user and perform root operations with `sudo`.

sudoers are maintained in the `/etc/sudoers` file. Edit it with `visudo`:

```bash
sudo visudo
```

### Common commands

```bash
# interactive mode - root shell env
sudo -i

# sudo shell - regular user shell env and bashrc
sudo -s

# add user to sudoers
usermod -aG sudo <username>

# check sudo usage history
journalctl -e /usr/bin/sudo

# failed root login attempts
sudo lastb
```

### Set time between sudo password

Set how long (in seconds) before you have to reenter the sudo password to execute sudo commands. Add `timestamp_timeout=<seconds>` to the `env_reset` line.

Set to `0` to prompt every time, and `-1` to never prompt:

```bash
sudo visudo

#
# This file MUST be edited with the 'visudo' command as root.
...
Defaults        env_reset,timestamp_timeout=90      # ask for passwd after 90s
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
...

## Change password

```bash
passwd
Changing password for <username>.
Current password: 
New password: 
Retype new password: 
passwd: password updated successfully

```

## Delete user

```bash
userdel <username>
```