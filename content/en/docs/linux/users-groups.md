---
title: "Users and groups"
weight: 90
description: >
  Linux and localization.
---

_Authentication_ is when you determine that a user is authentic - they are who they claim to be.

## Managing user accounts

## Adding accounts

A few files are used to create accounts (files on the same line are not linked):

```
Account creation files:                                 Built or modified after acct creation

/etc/default/useradd    -->                             --> /home/userid

/etc/login.defs         -->                             --> /etc/passwd

                                User account created    

/etc/skel               -->                             --> /etc/shadow

admin input             -->                             --> /etc/group
```

## Account creation files

### /etc/login.defs

Config file that contains directives for various shadow password suite commands. These settings are created after the OS installation.

_shadow password suite_: commands that deal with account credentials, including:
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

### /etc/default/useradd

Contains default values for `useradd` command:

```bash
# list all active directives
useradd -D
GROUP=100
HOME=/home              # Must set CREATE_HOME in /etc/login.defs to 'yes'
INACTIVE=-1             # Num of days after passwd expiration that the acct is deactivated
EXPIRE=
SHELL=/bin/sh           # default shell program
SKEL=/etc/skel          # skeleton directory
CREATE_MAIL_SPOOL=no
```

### /etc/skel

Contains files. If you set up users to have a HOME directory, these files are copied to `/user/home/`:

```bash
ls -laog /etc/skel/
total 44
drwxr-xr-x   2  4096 Mar 20 20:51 .
drwxr-xr-x 168 12288 Mar 20 20:55 ..
-rw-r--r--   1   220 Apr  4  2018 .bash_logout
-rw-r--r--   1  3771 Apr  4  2018 .bashrc
-rw-r--r--   1  8980 Apr 16  2018 examples.desktop
-rw-r--r--   1  2078 Dec  6  2021 .kshrc
-rw-r--r--   1   807 Apr  4  2018 .profile
```

## Files built or modified when acct created

These files are built or modified after a user account is created. Does not include group files.

### /etc/passwd

Stores account information. When account is created, it is added to this file.

> This file does not hold passwords due to security concerns. If it does, you can migrate passwords to the correct `/etc/shadow` file with the `pwconv` command.


```bash
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
```

| Field num | Description |
|-----------|-------------|
| 1 | username |
| 2 | password field. `x` indicates there is a passwd in `/etc/shadow` |
| 3 | UID |
| 4 | GID |
| 5 | Comment field, traditionally contains user's full name |
| 6 | user HOME dir |
| 7 | user default shell. `/sbin/nologin` and `/bin/false` prevents an acct from logging in to shell, usually for system service accounts. If you attempt to login, you get a message and the session terminates. Edit/create `/etc/nologin.txt` to customize the message. `/bin/false` just logs you out, no message. |


### /etc/shadow

Contains info about account's password, even if one does not yet exist.

```bash
sudo cat /etc/shadow
root:$y$j9T$EDErVNwiCEqd1Nm5W26d..$aFG6ZaS4.iri2wuCLwuzcexksaNNYDpm1ntYDOGTb03:19804:0:99999:7:::
daemon:*:19773:0:99999:7:::
bin:*:19773:0:99999:7:::
sys:*:19773:0:99999:7:::
...
<username>:$y$j9T$qQNRlSwnLVRW/liifd7HW/$w22t73yRHnNvpqsSgDChfEucV825MollmauAvtbeYp9:19804:0:99999:7:::
```
| Field num | Description |
|-----------|-------------|
| 1 | username. Only field shared with `/etc/passwd` |
| 2 | passwd, salted and hashed. `!` or `!!` indicates no password. `*` indicates account cannot log in with a password. `!<password-hash>` indicates account is locked. |
| 3 | Date of passwd change in Unix Epoch time |
| 4 | Num of days after a passwd is changed that it can be changed again. |
| 5 | Num of days before passwd change is required. Password expiration date |
| 6 | Num of days prior to passwd expiration that a warning is issued. |
| 7 | Num of days after a passwd expires that the account is deactivated |
| 8 | Date account expired in Unix Epoch time |
| 9 | _special flag_ field for a special future use. It is blank. |

> Unix Epoch time is the number of seconds since Jan 1, 1970. `etc/shadow` expresses it in days, not seconds.

## Create accounts

### useradd

```bash
useradd <username>
-c # comment field, traditionally includes user's full name
-d # home dir specification
-D # display directives in /etc/default/useradd
-e # acct expirtaion date, set by EXPIRE directive
-f # num of days passwd expired before acct deactivated. -1 means never deactivate. INACTIVE directive
-g # group membership
-G # additional group membership
-m # create user acct /home. CREATE_HOME directive
-M # do NOT create /home
-s # acct shell. SHELL directive
-u # UID
-r # create system acct, not user acct
```

### Rocky

```bash
# check directive settings
grep CREATE_HOME /etc/login.defs 
CREATE_HOME     yes
useradd -D | grep -i shell
SHELL=/bin/bash
# create user
sudo useradd <username>
```

### Ubuntu

You have to add more options in Ubunutu bc `/etc/login.defs` does not have the same defaults as other distros:

```bash
# check directive settings
grep CREATE_HOME /etc/login.defs 
useradd -D | grep -i shell
SHELL=/bin/sh
# create user with home dir and bash shell
sudo useradd -md /home/linuxuser -s /bin/bash linuxuser
# check user files were created
grep ^linuxuser /etc/passwd
linuxuser:x:1001:1001::/home/linuxuser:/bin/bash
sudo grep ^linuxuser /etc/shadow
linuxuser:!:19813:0:99999:7:::  # no password yet
sudo ls -a /home/linuxuser/
.  ..  .bash_logout  .bashrc  .profile
sudo ls -a /etc/skel/
.  ..  .bash_logout  .bashrc  .profile
```

### getent

View account records:

```bash
getend <filename> <username>

getent passwd linuxuser
linuxuser:x:1001:1001::/home/linuxuser:/bin/bash
# honors security settings on /etc/shadow
getent shadow linuxuser
sudo getent shadow linuxuser
linuxuser:!:19813:0:99999:7:::
```

## Passwords

### passwd

Set or modify the user password with `passwd`:

```bash
sudo password <username>
-d # rm acct passwd
-e # set passwd as expired, must change it at next login
-i # sets num of days for acct to become inactive after passwd expiration
-l # lock, places '!' in front of passwd in /etc/shadow
-n # num of days after passwd change that user can change passwd again
-S # display password status
-u # unlock, rm '!' in front of passwd in /etc/shadow
-w # num of days to issue warning before password expiration
-x # num of days until a passwd change is required

# get passwd status
# P usable
# NP no password
# L locked
sudo passwd -S linuxuser
linuxuser P 03/31/2024 0 99999 7 -1
```

### chage

Same as `passwd -S <username>` but more human readable:

```bash
# view acct passwd status
sudo chage -l <username>
# interactively update passwd settings
sudo chage <username>
```

## Modifying accounts

### usermod

Utility to modify accounts, useful when you forget to check the distro's account creation directives:

```bash
usermod [FLAG...] <username>
-c # modify comment field
-d # set new home dir. use -m option to move contents of current /home to new /home
-e # modify acct expiration date
-f # modify num of days for acct to become inactive after passwd expiration
-g # modify default group
-G # update additional group membership
-l # modify account username (login)
-L # lock acct, (!) in shadow file
-s # change acct shell
-u # change UID
-U # Unlock acct, rm (!) in shadow file

# lock account
sudo usermod -L linuxuser 
# display status
sudo passwd -S linuxuser 
linuxuser L 03/31/2024 0 99999 7 -1
# display shadow file entry- (!)
sudo getent shadow linuxuser
linuxuser:!$y$j9T$f4O7G12v7ecON6j8SAfiQ.$7kRt7qYpxngkDPaHATTNBlDGc6hHQc7sPuAW9iMKAJ.:19813:0:99999:7:::
# unlock and display status again
sudo usermod -U linuxuser 
sudo passwd -S linuxuser 
linuxuser P 03/31/2024 0 99999 7 -1
```

## Deleting accounts

### userdel

Deletes an account and all account files:

```bash
# -r deletes user /home dir and its contents
sudo userdel -r <username>
```

## Groups

- Groups are part of Linux's discretionary access control (DAC).
- DAC: Traditional Linux security control where access to a file or any object is based on the user's identity and current group membership.
- _Default group_: When you create a user, it is given a _default group_.
  - A process can have only one group at a time
  - Users can be members of many groups but have only one default
  - Default group is the acct's current group when first logged in
  - If default group is not designated when a user acct is created, a new group is created with same name as username and new GID
- Have name and group identification number (GID). Names are for humans to read, GIDs are for linux to read
- tracked in `/etc/group`
- Group passwds stored in `/etc/gshadow` file

> DO NOT allow access to groups with group passwords. Set user acct passwords, and grant access to groups via membership, not passwords.

### View group membership

```bash
# GID is 4th field
grep linuxuser /etc/passwd | gawk -F : '{print $4}'
1001
# GID, but not group name
getent passwd linuxuser 
linuxuser:x:1001:1001::/home/linuxuser:/bin/bash
# display account name
sudo groups linuxuser 
linuxuser : linuxuser
# display GID
getent group linuxuser
linuxuser:x:1001:
# confirm GID
grep 1001 /etc/group
linuxuser:x:1001:
```

### groupadd

Creates a group, tracked in `/etc/group`. Group must exist before you can add a user to it.

> Debian prefers that you use `addgroup` 

```bash
# -g specifies group number
sudo groupadd -g 1966 cream
# view group
getent group cream
cream:x:1966:
# <groupname>:<password-in-/etc/gshadow>:GID:<group-members>
grep cream /etc/group
cream:x:1966:
# check for passwd
sudo getent gshadow cream
cream:!::
```

### Add user to group

Add users to groups with `usermod`:

```bash
# -a preserves existing group membership
# -G add the user acct to the group
sudo usermod -aG <groupname>
# view groups
sudo groups linuxuser 
linuxuser : linuxuser
# add user to group
sudo usermod -aG cream linuxuser 
# view user groups
sudo groups linuxuser 
linuxuser : linuxuser cream
# view user group membership
getent group cream
cream:x:1966:linuxuser
```

### groupmod

Modifies a group:

```bash
groupmod [OPTIONS...] <groupname>
-g # modify GID
-n # modify name

# view group info
getent group cream
cream:x:1966:linuxuser
# change group name
sudo groupmod -n Cream cream
# confirm /etc/group was updated
getent group Cream
Cream:x:1966:linuxuser
```

### groupdel

Deletes a group:

```bash
# delete group
sudo groupdel Cream
# check /etc/group cleanup
getent group Cream
# check user was removed from group
sudo groups linuxuser 
linuxuser : linuxuser
# check fs for files associated with deleted group
sudo find / -gid 1966 2>/dev/null
```

## Environment setup

