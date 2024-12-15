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


Stands for "substitute user". Do not log in as `root`. Assign superuser privileges to a regular user and perform root operations with `sudo`.

sudoers are maintained in the `/etc/sudoers` file. Edit it with `visudo`:

```bash
sudo visudo

# only uncommented line in sudoers:
root ALL=(ALL) ALL      # USER PLACES=(AS_USER) [NOPASSWD:] COMMAND

# root:  users that this rule applies to
# ALL:   hosts where the sudo command can be executed - envs can share a sudoers file
# (ALL): which users the user (root) can act as
# ALL:   commands that the user (root) can run
# In plain English: "root can execute all commands as all suers from all places"


# Allow user "fred" to run "sudo reboot"
# ...and don't prompt for a password
#
fred ALL = NOPASSWD:/sbin/reboot
fred ALL=(ALL) ALL                                  # run all commands from anywhere as root
fred ALL=(ALL) NOPASSWD:ALL                         # run all commands w/o passwd prompt
fred ALL=(ALL) /usr/bin/apt,/sbin/mount             # run specified commands anywhere as root
fred ALL=(ALL) /usr/bin/apt, NOPASSWD:/sbin/mount   # run apt anywhere, mount anywhere w/o passwd
```

### Common commands

```bash
sudo -i                         # interactive mode - root shell env
sudo -s                         # sudo shell - regular user shell env and bashrc
usermod -aG sudo <username>     # add user to sudoers
journalctl -e /usr/bin/sudo     # check sudo usage history
sudo lastb                      # failed root login attempts
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

### References

- [How to edit the sudoers file](https://www.digitalocean.com/community/tutorials/how-to-edit-the-sudoers-file)
- [Sudo - An Advanced Howto](https://centoshelp.org/security/sudo-an-advanced-howto/)

## Account files

A few files are used to create accounts (files on the same line are not linked):

| Account creation files | Build or mod after acct creation |
|---|---|
| `/etc/default/useradd`  | `/home/userid`  |
| `/etc/login.defs`  | `/etc/passwd`  |
| -**user acct created**-  | -**user acct created**-  |
| `/etc/skel`  | `/etc/shadow`  |
|  admin input | `/etc/group`  |


### Creation files

#### /etc/login.defs

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

#### /etc/default/useradd

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

#### /etc/skel

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

### Built or modified when acct created

These files are built or modified after a user account is created. Does not include group files.

#### /etc/passwd

Stores account information. When account is created, it is added to this file.
- Security riskb bc its world-readable bc other commands use it. For example, `ls` uses it to display ownership

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


#### /etc/shadow

Contains info about account's password, even if one does not yet exist. Root accessible only:

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

### Restrict shell access

Accounts for mail servers (POP3 or SMTP) or FTP/SFTP don't need shell access. You can assign them the `nologin` shell to restrict access.

```bash
# existing users
cat /etc/shells                                             # 1. Check /etc/shells for /usr/sbin/nologin (ubuntu)
sudo vim /etc/shells                                        # 2. Add if necessary
cat /etc/shells                                             # 3. Verify
...
/usr/sbin/nologin

grep '^neverlogin' /etc/passwd                              # 4. Check current shell
neverlogin:x:1003:1003:,,,:/home/neverlogin:/bin/bash

usermod -s /usr/sbin/nologin neverlogin                     # 5. Modify user shell
usermod -s /usr/sbin/nologin neverlogin                     # 6. Verify changes

# new users - must have /usr/sbin/nologin available in /etc/shells
useradd -s /usr/sbin/nologin neverlogin                     # 1. Create user

getent passwd neverlogin                                    # 2. Verify
neverlogin:x:1003:1003::/home/neverlogin:/usr/sbin/nologin

# assign shell to nologin user
chsh -s /usr/bin/bash neverlogin                            # 1. Change shell
getent passwd neverlogin                                    # 2. Verify
neverlogin:x:1003:1003::/home/neverlogin:/usr/bin/bash

```

## Create accounts

### adduser vs useradd

- `adduser` is easier to use because it has an interactive menu that helps create users.
- `useradd` is a low-level utility with more options during account creation.

```bash
# create user and add to group
adduser --ingroup <group-name> <username>
```

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

Get entries from Name Service switch libraries which are configured in `/etc/nsswitch.conf`:

```bash
getent <filename> <username>

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

## Troubleshooting

### Check password

```bash
# check password
sudo getent passwd linuxuser 
linuxuser:x:1001:1001::/home/linuxuser:/bin/bash

# check shadow file, make sure no '!'
sudo getent shadow linuxuser
linuxuser:$y$j9T$f4O7G12v7ecON6j8SAfiQ.$7kRt7qYpxngkDPaHATTNBlDGc6hHQc7sPuAW9iMKAJ.:19813:0:99999:7:::
```


### last, lastlog, and lastb

`last` and `lastlog` check when an account was last accessed:
- `last` maintains more login in `/var/log/wtmp` and `/var/log/wtmp.*` files
- `lastlog` maintains only most recent login in `/var/log/lastlog`
- `lastb` searches for failed login attempts in `/var/log/btmp`

```bash
# all logins, specify additional files with -f option
sudo last -f /var/log/wtmp | grep linuxus
linuxuse pts/1        10.0.2.2         Mon Apr 22 08:31 - 08:34  (00:03)
linuxuse pts/0        10.0.2.2         Fri Apr 19 20:44    gone - no logout
linuxuse pts/0        10.0.2.2         Thu Apr 18 23:06 - 23:49  (00:42)
linuxuse pts/0        10.0.2.2         Thu Apr 18 22:13 - 23:06  (00:53)
linuxuse pts/0        10.0.2.2         Thu Apr 18 22:09 - 22:09  (00:00)
linuxuse tty2         tty2             Thu Apr 18 22:00 - down   (00:05)
linuxuse tty2         tty2             Thu Apr 18 21:55 - down   (00:03)
linuxuse tty2         tty2             Sun Apr 14 10:39 - crash (4+11:15)
linuxuse tty2         tty2             Sun Apr 14 10:34 - 10:35  (00:01)
linuxuse tty2         tty2             Thu Apr 11 22:02 - 10:31 (2+12:28)
linuxuse tty2         tty2             Thu Apr 11 21:53 - 22:01  (00:08)

# most recent login
lastlog
Username         Port     From             Latest
root                                       **Never logged in**
...
linuxuser        pts/1    10.0.2.2         Mon Apr 22 08:31:02 -0400 2024   # in last output
...

# failed login attempts
sudo lastb -f /var/log/btmp 
linuxuse ssh:notty    127.0.0.1        Thu Apr 18 23:04 - 23:04  (00:00)
linuxuse ssh:notty    127.0.0.1        Thu Apr 18 23:04 - 23:04  (00:00)
linuxuse ssh:notty    127.0.0.1        Thu Apr 18 23:04 - 23:04  (00:00)
linuxuse ssh:notty    127.0.0.1        Thu Apr 18 23:04 - 23:04  (00:00)

btmp begins Thu Apr 18 23:04:27 2024
```

### Privilege issues

Check to make sure the users have the correct `su` privileges:

```bash
# members of admin can become su
# members of sudoers can execute any command as any user
sudo cat /etc/sudoers
...
# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL
```

Check which groups the user belongs to with `id`:

```bash
id linuxuser
uid=1001(linuxuser) gid=1001(linuxuser) groups=1001(linuxuser),27(sudo)
```
### GUI status

If a user can log into the terminal but cannot log into the GUI, check the GUI with `systemctl`:

```bash
# check GUI status
sudo systemctl status graphical.target
● graphical.target - Graphical Interface
     Loaded: loaded (/lib/systemd/system/graphical.target; static)
     Active: active since Thu 2024-04-18 22:12:54 EDT; 5 days ago
       Docs: man:systemd.special(7)

Apr 18 22:12:54 ubuntu22 systemd[1]: Reached target Graphical Interface.
```

### Terminal status

You should check:
- multiple users can log into the terminal
- whether the terminal is corrupted
- getty services are running. These services provide the login prompts for text-based terminals

```bash
# verify multiple users can log in
sudo systemctl status multi-user.target
● multi-user.target - Multi-User System
     Loaded: loaded (/lib/systemd/system/multi-user.target; static)
     Active: active since Thu 2024-04-18 22:12:54 EDT; 5 days ago
       Docs: man:systemd.special(7)

Apr 18 22:12:54 ubuntu22 systemd[1]: Reached target Multi-User System.

# check whether terminal is corrupted
c # means its a character file
- # means terminal is corrupted, rebuild with mknod command
ls -l /dev/tty?
crw--w---- 1 root tty 4, 0 Apr 21 09:44 /dev/tty0
crw--w---- 1 gdm  tty 4, 1 Apr 21 09:44 /dev/tty1
crw--w---- 1 root tty 4, 2 Apr 21 09:44 /dev/tty2
crw--w---- 1 root tty 4, 3 Apr 21 09:44 /dev/tty3
crw--w---- 1 root tty 4, 4 Apr 21 09:44 /dev/tty4
crw--w---- 1 root tty 4, 5 Apr 21 09:44 /dev/tty5
crw--w---- 1 root tty 4, 6 Apr 21 09:44 /dev/tty6
crw--w---- 1 root tty 4, 7 Apr 21 09:44 /dev/tty7
crw--w---- 1 root tty 4, 8 Apr 21 09:44 /dev/tty8
crw--w---- 1 root tty 4, 9 Apr 21 09:44 /dev/tty9

# check getty services
sudo systemctl status getty.target
● getty.target - Login Prompts
     Loaded: loaded (/lib/systemd/system/getty.target; static)
     Active: active since Thu 2024-04-18 22:12:29 EDT; 5 days ago
       Docs: man:systemd.special(7)
             man:systemd-getty-generator(8)
             http://0pointer.de/blog/projects/serial-console.html

Apr 18 22:12:29 ubuntu22 systemd[1]: Reached target Login Prompts.
```

### Locked accounts

```bash
# check status of linuxuser. 'L' means 'locked'
sudo passwd -S linuxuser 
linuxuser L 03/31/2024 0 99999 7 -1

# verify locked w getent. '!' before passwd means its locked
sudo getent shadow linuxuser
linuxuser:!$y$j9T$f4O7G12v7ecON6j8SAfiQ.$7kRt7qYpxngkDPaHATTNBlDGc6hHQc7sPuAW9iMKAJ.:19813:0:99999:7:::

# unlock user
sudo usermod -U linuxuser 

# verify unlocked
sudo passwd -S linuxuser 
linuxuser P 03/31/2024 0 99999 7 -1
```

### Expired accounts

Check account expiration with `chage`:

```bash
date
Wed Apr 24 02:51:34 PM EDT 2024

# view account status
sudo chage -l linuxuser
Last password change					: Mar 31, 2024
Password expires					: never
Password inactive					: never
Account expires						: never                           # never expires
Minimum number of days between password change		: 0
Maximum number of days between password change		: 99999
Number of days of warning before password expires	: 7
```

### Authentication

Check or run the following:
- PAM modules
- `pam_tally2`
- `faillock`
- IdP config (LDAP, Kerberos)
- SELinux or AppArmor

You can use `sealert` for SELinux machines:

```bash
# checks for policy violations
sealert -a /var/log/audit/audit.log
100% done
found 0 alerts in /var/log/audit/audit.log
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
# view groups
groups
linuxuser adm cdrom sudo dip plugdev lxd

# view groups with id
id
uid=1000(linuxuser) gid=1000(linuxuser) groups=1000(linuxuser),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),101(lxd)

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

# view with members utility
apt install members
members <group-name>
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

### newgrp

Logs you into a new group.
- You can only be logged into one group at a time during a session
- When you switch groups, all files and dirs that you create belong to that group

```bash
newgrp [GROUP]

# logged in group is listed first
id
uid=1000(linuxuser) gid=1000(linuxuser) groups=1000(linuxuser),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),101(lxd)

newgrp plugdev      # change group
id                  # view groups
uid=1000(linuxuser) gid=46(plugdev) groups=46(plugdev),4(adm),24(cdrom),27(sudo),30(dip),101(lxd),1000(linuxuser)
touch plugdevfile   # create file
ls -l plugdevfile   # view group
-rw-r--r-- 1 linuxuser plugdev 0 Dec 14 19:28 plugdevfile
```


## Querying users

You can audit user access history and other account information.

### whoami

Displays current user name:

```bash
whoami
```

### who

Displays info about every account on the system:

```bash
who
<username> tty2         2024-03-24 16:48 (tty2)
```

### w

Like `who`, but more verbose:

```bash
# load average: 1m, 5m, 15m
# JCPU: total CPU time acct has used
# PCPU: CPU time current command has used
# WHAT: what the acct is currently running
w
 08:59:08 up 14:54,  1 user,  load average: 0.19, 0.13, 0.10
USER       TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
<username> tty2     tty2             24Mar24 23:24m  0.01s  0.01s /usr/libexec/gnome-session-binary --session=ubuntu
```

### id

Useful in shell scripts.

Gather various data about the current user process or info about the provided user id:

```bash
id [username|UID]
-g # Display acct's current group's GID
-G # Display all grou membership GIDs
-n # Display acct name instead of UID
-u # Display acct UID

# current user
id
uid=1000(<current-user>) gid=1000(<current-user>) groups=1000(<current-user>),27(sudo),999(vboxsf)
# specified user
id linuxuser 
uid=1001(linuxuser) gid=1001(linuxuser) groups=1001(linuxuser)
# specified user
id -un 1001
linuxuser

# script example
grep USER /etc/profile
USER="`/usr/bin/id -un`"
```

### last

Pulls info from the `/var/log/wtmp` file and shows a list of accounts and the last login/logout times:

```bash
# pull from /var/log/wtmp
last
<username> tty2         tty2             Sun Mar 24 16:48    gone - no logout
reboot     system boot  6.5.0-26-generic Sun Mar 24 16:47   still running
<username> tty2         tty2             Sat Mar 23 09:19 - crash (1+07:27)
reboot     system boot  6.5.0-26-generic Sat Mar 23 09:04   still running
<username> tty2         tty2             Thu Mar 21 23:09 - down   (00:05)
reboot     system boot  6.5.0-26-generic Thu Mar 21 23:07 - 23:14  (00:06)
reboot     system boot  6.5.0-26-generic Thu Mar 21 23:07 - 23:07  (00:00)
<username> tty2         tty2             Thu Mar 21 23:06 - down   (00:00)
reboot     system boot  6.5.0-26-generic Thu Mar 21 23:05 - 23:06  (00:01)
...

# pull from specific file (wtmp.1):
last -f /var/log/wtmp.1
```

## Disk quotas (NEED TO FIGURE OUT)

https://www.linode.com/docs/guides/file-system-quotas/

You can limit the number of files a user can create and restrict the total fs space available to them to prevent users from filling up the hard drive with files.

Has four steps:
1. Modify `/etc/fstab` to enable fs quotas
2. Mount the fs. If it was already mounted, unmount it and mount it again.
3. Create the file quota.
4. Establish user or group quota limits and grace periods.
