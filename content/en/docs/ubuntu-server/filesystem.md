---
title: "Navigating the filesystem"
linkTitle: "Filesystem"
weight: 30
# description:
---

The Linux filesystem directory structure follows the Filesystem Hierarchy Standard (FHS), where every directory has a specific purpose.

## Common directories

You can also run `man hier` to get a detailed description of the filesystem:

| Directory | Description |
|:--|:--|
| / | Beginning of the fs |
| /etc | System-wide application configuration |
| /home | User home directories |
| /root | Home directory for `root` |
| /media | Removable media, i.e. flash drives |
| /mnt | Volumes that will be mounted for a while |
| /opt | Additional software packages |
| /bin | Essential user libraries such as `cp`, `ls`, etc... |
| /proc | Virtual filesystem for OS-level components such as running processes |
| /usr/bin | Common user commands that are not needed to boot or repair the system |
| /usr/lib | Object libraries |
| /var/log | Log files |


## Distro info

```bash
# --- Get distro info --- #
cat /etc/os-release 
PRETTY_NAME="Ubuntu 24.04.1 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.1 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo

# --- Get shorter distro info --- #
lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 24.04.1 LTS
Release:	24.04
Codename:	noble
```

## Viewing log files

`/var/log/syslog` - Contains information about processes happening in the background as your server runs, including warnings and errors.
- If you have an error, look in `syslog` and then google for a resolution

```bash
head -n 100 /var/log/syslog             # view first 100 lines
head -100 /var/log/syslog

tail -n 100 /var/log/syslog             # view last 100 lines
tail -100 /var/log/syslog

tail -f /var/log/syslog                 # watch syslog in real time
```