---
title: "Users and groups"
weight: 90
description: >
  Linux and localization.
---

_Authentication_ is when you determine that a user is authentic - they are who they claim to be.

## Managing user accounts

### Adding accounts

A few files are used to create accounts (files on the same line are not linked):

```
/etc/default/useradd    -->                             --> /home/userid

/etc/login.defs         -->                             --> /etc/passwd

                                User account created    

/etc/skel               -->                             --> /etc/shadow

admin input             -->                             --> /etc/group
```
#### `/etc/login.defs`

Config file that contains directives for various shadow password suite commands. These settings are created after the OS installation.
- _shadow password suite_: commands that deal with account credentials, including:
  - `useradd`
  - `userdel`
  - `passwd`


Directives include passwd length, how long before you have to change passwd, whether home dir is created by default, etc. Common directives include:

| Name | Description |
|------|-------------|
| `PASS_MAX_DAYS` | Num of days before passwd change is required. |
| `PASS_MIN_DAYS` | Num of days after a passwd is changed that it can be changed again. |
| `PASS_MIN_LENGTH` | Min chars requried in password. |
| `PASS_WARN_AGE` | Num of days prior to passwd expiration that a warning is issued. |
| `CREATE_HOME` | Create user acct home directory. Default is 'no'. |
| `ENCRYPT_METHOD` | passwd hash method. |

```bash
# lines that do not begin (-v) with $ or # (comment and blanks)
grep -v ^$ /etc/login.defs | grep -v ^\#
MAIL_DIR        /var/mail
...
UID_MIN			 1000   # lowest UID allowed for user accts (sometimes 500)
UID_MAX			60000
GID_MIN			 1000   # lowest GID allowed for group accts
GID_MAX			60000
LOGIN_RETRIES		5
LOGIN_TIMEOUT		60
...

# root is always 0
gawk -F: '{print $3, $1}' /etc/passwd | sort -n
0 root
1 daemon
2 bin
```
- UID: User Identification Number (UID) is assigned to a _user account_ or _normal account_.
  - Humans use account names, Linux uses UIDs.
- System accounts proide services (daemons) or perform special tasks.
- root always has UID = 0.