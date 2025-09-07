---
title: "Users, groups, and permissions"
linkTitle: "xUsers/groups"
weight: 10
# description:
---

There are three types of Linux users:
- root
- sudoers
- regular users

## sudo

Stands for "substitute user"
- Do not log in as `root` - assign superuser privileges to a regular user and perform root operations with `sudo`.
- `sudo` lets you restrict which commands a user can run as `root`

### Common commands

```bash
usermod -aG sudo <username>     # add user to sudoers (Ubuntu)
sudo !!                         # run previous command as sudo
sudo -i                         # interactive mode - root shell env
sudo -s                         # sudo shell - regular user shell env and bashrc
journalctl -e /usr/bin/sudo     # check sudo usage history
sudo lastb                      # failed root login attempts
```

### visudo

Command that lets you configure `sudo` privileges per user
- sudoers are maintained in the `/etc/sudoers` file. Edit it with `visudo`
- `visudo` ensures that there are no errors in the `/etc/sudoers` file
- group names are preceded by `%`, user names are not
- use groups to manage `sudo` because you can add/remove users from a group rather than the `/etc/sudoers` file
- use full path (`/usr/bin/apt`) in command rules to prevent malicious scripting

```bash
sudo visudo

# group names start with %, users do not
root ALL=(ALL:ALL) ALL
%admin ALL=(ALL) ALL


# --- Explanation --- #
root ALL=(ALL:ALL) ALL
# root  - users that this rule applies to
# ALL=  - hosts where the sudo command can be executed - root can use sudo from any terminal
# (ALL: - which users the user (root) can act as
# ALL)  - which groupss the user (root) can act as
# ALL   - commands that the user (root) can run
# In plain English: "root can execute all commands as all suers from all places"


# Allow user "fred" to run "sudo reboot" from any terminal
# ...and don't prompt for a password
#
fred ALL = NOPASSWD:/sbin/reboot
fred ALL=(ALL) ALL                                  # run all commands from any terminal as root
fred ALL=(ALL) NOPASSWD:ALL                         # run all commands w/o passwd prompt
fred ALL=(ALL) /usr/bin/apt,/sbin/mount             # run specified commands any terminal as root
fred ALL=(ALL) /usr/bin/apt, NOPASSWD:/sbin/mount   # run apt any terminal, mount anywhere w/o passwd
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

| Account creation files  | Build or mod after acct creation |
| ----------------------- | -------------------------------- |
| `/etc/default/useradd`  | `/home/userid`                   |
| `/etc/login.defs`       | `/etc/passwd`                    |
| -**user acct created**- | -**user acct created**-          |
| `/etc/skel`             | `/etc/shadow`                    |
| admin input             | `/etc/group`                     |


### Creation files

#### /etc/login.defs

Config file that contains directives for various shadow password suite commands. These settings are created after the OS installation.

_shadow password suite_: commands that deal with account credentials, including:
- `useradd`
- `userdel`
- `passwd`


Directives include passwd length, how long before you have to change passwd, whether home dir is created by default, etc. Common directives include:

| Name              | Description                                                         |
| ----------------- | ------------------------------------------------------------------- |
| `PASS_MAX_DAYS`   | Num of days before passwd change is required.                       |
| `PASS_MIN_DAYS`   | Num of days after a passwd is changed that it can be changed again. |
| `PASS_MIN_LENGTH` | Min chars requried in password.                                     |
| `PASS_WARN_AGE`   | Num of days prior to passwd expiration that a warning is issued.    |
| `CREATE_HOME`     | Create user acct home directory. Default is 'no'.                   |
| `ENCRYPT_METHOD`  | passwd hash method.                                                 |

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

| Field num | Description                                                                                                                                                                                                                                                                                                      |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1         | username                                                                                                                                                                                                                                                                                                         |
| 2         | password field. `x` indicates there is a passwd in `/etc/shadow`                                                                                                                                                                                                                                                 |
| 3         | UID                                                                                                                                                                                                                                                                                                              |
| 4         | GID                                                                                                                                                                                                                                                                                                              |
| 5         | Comment field, traditionally contains user's full name                                                                                                                                                                                                                                                           |
| 6         | user HOME dir                                                                                                                                                                                                                                                                                                    |
| 7         | user default shell. `/sbin/nologin` and `/bin/false` prevents an acct from logging in to shell, usually for system service accounts. If you attempt to login, you get a message and the session terminates. Edit/create `/etc/nologin.txt` to customize the message. `/bin/false` just logs you out, no message. |


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
| Field num | Description                                                                                                                                                        |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1         | username. Only field shared with `/etc/passwd`                                                                                                                     |
| 2         | passwd, salted and hashed. `!` or `!!` indicates no password. `*` indicates account cannot log in with a password. `!<password-hash>` indicates account is locked. |
| 3         | Date of passwd change in Unix Epoch time                                                                                                                           |
| 4         | Num of days after a passwd is changed that it can be changed again.                                                                                                |
| 5         | Num of days before passwd change is required. Password expiration date                                                                                             |
| 6         | Num of days prior to passwd expiration that a warning is issued.                                                                                                   |
| 7         | Num of days after a passwd expires that the account is deactivated                                                                                                 |
| 8         | Date account expired in Unix Epoch time                                                                                                                            |
| 9         | _special flag_ field for a special future use. It is blank.                                                                                                        |

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

## Users

- User accounts represent the actual people that use the server and have access to what they need the server for
  - Users do not have to be people, they can be "system users" that run system tasks
- To create multiple users on the same system, create all their home directories in `/home`:
  ```bash
  la -l /home/
  total 8
  drwxr-x--- 2 jdoe       jdoe       4096 Dec 28 23:51 jdoe
  drwxr-x--- 4 normaluser normaluser 4096 Dec 27 21:44 normaluser
  ```

### root

- `root` is the most powerful user and can perform any task or access any file or directory in the system
- After installation, you have the `root` user and the administrator user account you created
  - The admin should have their own account and use `sudo` to switch to `root` when needed
  - The admin user is really just a regular user that has `sudo` access by default
  - You have to explicitly grant other user accounts access to `sudo`
- Ubuntu locks out the `root` account after installation. You should run individual commands that require elevated privileges with `sudo`
- You must create the `root` password after installation

```bash
sudo su -           # switch to root from account that has root access
```
### adduser vs useradd

- `adduser` is easier to use because it has an interactive menu that helps create users.
- `useradd` is a low-level utility with more options during account creation.

```bash
# create user and add to group
adduser --ingroup <group-name> <username>
```

### useradd

Lets you add user accounts only. You need to create a password, specify home dir, etc.:

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

# create user with home directory in /home
useradd -d /home/jdoe -m jdoe      
```

### adduser

Interactive alternative to `useradd` that lets you assign a password upon creation and provides verbose output.
- A Perl script that calls `useradd` under the hood.
- Not available on all Linux distros, so learn `useradd` and its options as well

```bash
sudo adduser seconduser
info: Adding user `seconduser' ...
info: Selecting UID/GID from range 1000 to 59999 ...
info: Adding new group `seconduser' (1002) ...
info: Adding new user `seconduser' (1002) with group `seconduser (1002)' ...
info: Creating home directory `/home/seconduser' ...
info: Copying files from `/etc/skel' ...
New password: 
Retype new password: 
passwd: password updated successfully
Changing the user information for seconduser
Enter the new value, or press ENTER for the default
	Full Name []: 
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 
Is the information correct? [Y/n] Y
info: Adding new user `seconduser' to supplemental / extra groups `users' ...
info: Adding user `seconduser' to group `users' ...
```

### usermod

Utility to modify accounts, useful when you forget to check the distro’s account creation directives:

```bash
-a # append user to group
-c # modify comment field
-d # set new home dir. use -m option to move contents of current /home to new /home
-e # modify acct expiration date
-f # modify num of days for acct to become inactive after passwd expiration
-g # modify default group
-G # update additional group membership
-l # modify account username (login)
-L # lock acct, (!) in shadow file
-m # move home dir to new location
-s # change acct shell
-u # change UID
-U # Unlock acct, rm (!) in shadow file

usermod -aG <group> <username>          # add <username> to <group> as secondary group
usermod -g <group> <username>           # add <username> to <group> as primary group


# --- Lock user account --- #
sudo usermod -L linuxuser               # lock account
sudo passwd -S linuxuser                # display status
linuxuser L 03/31/2024 0 99999 7 -1

sudo getent shadow linuxuser            # display shadow file entry- (!)
linuxuser:!$y$...:19813:0:99999:7:::

sudo usermod -U linuxuser               # unlock
sudo passwd -S linuxuser                # display status again
linuxuser P 03/31/2024 0 99999 7 -1
```

### userdel

Deletes a user account:
- Verify that you do not need any of the user's files before you delete it
- Does not remove contents of user's `/home` directory by default

```bash
userdel <username>      # delete user only
userdel -r <username>   # delete user, /home dir, and mail spool
```

### /etc/skel default files and configs

Files in this dir are copied into all new user `/home` directories:
- Create default files for recommended text editor or a 'welcome' file w important info
- Add files or edit existing as needed

```bash
ls -la /etc/skel/
total 20
drwxr-xr-x   2 root root 4096 Aug 27 14:18 .
drwxr-xr-x 108 root root 4096 Dec 29 00:49 ..
-rw-r--r--   1 root root  220 Mar 31  2024 .bash_logout
-rw-r--r--   1 root root 3771 Mar 31  2024 .bashrc
-rw-r--r--   1 root root  807 Mar 31  2024 .profile

```

### chsh
> Maybe `usermod` instead?


Change the user's default shell:

```bash
cat /etc/shells                         # 1. Get available shells
# /etc/shells: valid login shells
/bin/sh
/usr/bin/sh
/bin/bash
/usr/bin/bash
/bin/rbash
/usr/bin/rbash
/usr/bin/dash
/usr/bin/screen
/usr/bin/tmux

grep ^jdoe /etc/passwd                  # 2. Check current shell
jdoe:x:1001:1001::/home/jdoe:/bin/sh

sudo chsh -s /bin/bash jdoe             # 3. Change shell to /bin/bash

grep ^jdoe /etc/passwd                  # 4. Verify change
jdoe:x:1001:1001::/home/jdoe:/bin/bash
```

Restrict user's shell access with `/usr/sbin/nologin`:

```bash
cat /etc/shells             # 1. Check if /usr/sbin/nologin is available
...
sudo vim /etc/shells        # 2. Add to file, if needed
```

### Switching users

Switch user accounts with `su`:

```bash
sudo su -               # switch to root from acct that has root access
su - <username>         # switch to <username> acct with <username> passwd
sudo su - <username>    # switch to <username> acct with your passwd 
```

### Lock and unlock accts

Use `passwd` to lock and unlock accounts:

```bash
passwd -l jdoe                      # lock acct
passwd: password changed.

sudo passwd -S jdoe                 # verify locked (L)
jdoe L 2024-12-29 0 99999 7 -1

passwd -u jdoe                      # unlock acct
passwd: password changed.

sudo passwd -S jdoe                 # verify unlocked (P)
jdoe P 2024-12-29 0 99999 7 -1
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


## Passwords

### passwd

Set password for a user:

```bash
password <username>
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


#### get passwd status


Fields:
1. Username
2. Password status
   - `P`: set and usable
   - `NP`: no password
   - `L`: locked
3. Date of last password change
4. Min number of days that must pass before the user can change password again. Prevents users from changing password back to previous passwords too quickly.
5. Max number of days that can pass between password changes.
6. Num days before expiration date that the user is warned to change password
7. Num days after expiration that account is disabled if password not changed
8. Num days since Unix epoch that will elapse before acct is disabled

```bash
# get info for each page
man shadow
man passwd

sudo passwd -S linuxuser
linuxuser P 03/31/2024 0 99999 7 -1

passwd                  # change password for current logged in user
sudo passwd <username>  # change password for <username>
```

### chage

Same as `passwd -S <username>` but more human readable:

```bash
# view acct passwd status
sudo chage -l <username>
# interactively update passwd settings
sudo chage <username>
```

### Set password expiration

`chage` - change user password account information.

```bash
# --- View password info (/etc/shadow) --- #
chage -l normaluser
Last password change					: Dec 27, 2024
Password expires					: never
Password inactive					: never
Account expires						: never
Minimum number of days between password change		: 0
Maximum number of days between password change		: 99999
Number of days of warning before password expires	: 7


sudo chage -d 0 <username>          # force passwd change on first login
sudo chage -M 90 <username>         # require passwd change after 90 days
sudo chage -m 10 <username>         # can't change password within 10 days of change
```

### Set password policy

> When making authentication changes, always keep a root shell open where you make the changes, and test the changes in another shell with the privilges or user that you are making the changes for.


A password policy determines the password requirements:
- Pluggable Authentication Modules (PAM) - packages that you can install that give more control over auth and lets us extend auth features
- Set in `/etc/pam.d/common-password`

```bash
sudo apt install libpam-passwdqc

sudo vim /etc/pam.d/common-password
...
# system remembers previous passwords
password	required			pam_pwhistory.so remember=99 use_authok
```
### /etc/passwd

Stores account information:
- When you create an account, it gets an entry in `/etc/passwd`
- World-readable because commands like `ls` use it for info
- System assigns the next available UID, which it uses to reference the account
- When the second field has an `x`, that means there is a password stored for that user in the `/etc/shadow` file

```bash
ls -l /etc/passwd
-rw-r--r-- 1 root root 1826 Dec 29 00:14 /etc/passwd    # world readable

cat /etc/passwd
...
# username:password:UID:GID:username:home dir:default shell
normaluser:x:1000:1000:ryan:/home/normaluser:/bin/bash
```

### /etc/shadow

Contains account password information, requires `root` privileges:
- UID is stored in `/etc/passwd`, so no need to duplicate here
- Hashed password is second field, after username
- `*` or `!` in second field means account is locked. You can't log into this account through shell or network, you must log in as normal user and switch users
- Third col is number of days since Unix epoch (Jan 1, 1970) that the password was last changed

```bash
sudo cat /etc/shadow
...
normaluser:$6$nYwtntKtT8/Cngzp$lIY0yPSFcc/tofKENzxGsFDry7DDvorh5eoLvVSSu.c63ammowy.rthykd7bX5vpehIR.BUa7MVgJRwchpb501:20084:0:99999:7:::
jdoe:$y$j9T$8LuhOxanK6eCw78R.yFK50$D3VM0AEQ7dNzIM9J.X8ap0Sl7uA28JDwE7fgkhGELI9:20085:0:99999:7:::

# root is locked
grep root /etc/shadow
root:*:19962:0:99999:7:::
```


## Groups

Groups let you categorize and manage users in bulk:
- What resources a user can access
- Every file and directory has an assigned user and group
- Each user is a member of a group named with their username
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
groups                  # which groups logged in user belongs to
groups <username>       # which groups <username> belongs to
cat /etc/group          # view all groups on the system
root:x:0:               #   1. group name
...                     #   2. group passwd (do not set bc security)
plugdev:x:46:normaluser #   3. GID
...                     #   4. Comma-separated list of users in group
normaluser:x:1000:
testuser:x:1002:
```

### groupadd

Create group:
- Tracked in `/etc/group`
- Group must exist before you add user to it

```bash
groupadd admins                 # add group
grep admins /etc/group          # verify
admins:x:1003:

sudo groupadd -g 1966 cream     # -g specifies group number

getent group cream              # view group
cream:x:1966:

grep cream /etc/group           # <groupname>:<password-in-/etc/gshadow>:GID:<group-members>
cream:x:1966:

sudo getent gshadow cream       # check for passwd
cream:!::
```

### gpasswd

Lets you administer the `/etc/group` and `/etc/gshadow` files:

```bash
gpasswd -d <username> <group>           # Remove user from group
Removing user jdoe from group admins

gpasswd -a <username> <group>           # Add user to group
Adding user jdoe to group admins
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
groupdel <group>                    # deletes <group>
getent group <group>                # check /etc/group cleanup
sudo find / -gid <GID> 2>/dev/null   # check fs for files associated with deleted group
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

## Permissions

| Permission | File                              | Directory                  |
| :--------: | :-------------------------------- | :------------------------- |
|    `r`     | File can be read                  | Can view contents of dir   |
|    `w`     | File can be written to            | Can change contents of dir |
|    `x`     | File can be executed as a program | Can `cd` into dir          |


```bash
ls -l file.1
-rw-rw-r-- 1 linuxuser linuxuser 23 Nov  9 14:50 file.1
 |   |  |       |         |     \_____________/   |
owner| world  owner     group         |        filename
   group                         Most recent edit

# first dash is object type
ls -logh
drwxrwxr-x 2 4.0K Dec 30 00:59 directory
-rw-rw-r-- 1    0 Dec 30 00:59 file
lrwxrwxrwx 1    4 Dec 30 01:00 link -> file

# view permissions in octal form
stat -c %a chownSample.txt
```

### chmod

Change file or directory permissions based on the specified mode:
- for greater control, use `find` to recursively set file and dir permissions

Set permissions with one of the following systems:
- string 
- numeric (octal)
- umask

#### Shortcuts

| Permission | Character | Number |
| :--------: | :-------: | :----: |
|    read    |    `r`    |   4    |
|   write    |    `w`    |   2    |
|  execute   |    `x`    |   1    |

#### String actions

|         Action         | Operator |
| :--------------------: | :------: |
|          add           |   `+`    |
|         remove         |   `-`    |
| set as only permission |   `=`    |

```bash
# --- numeric perms --- #
chmod 771 perms.sh
ll perms.sh 
-rwxrwx--x 1 linuxuser linuxuser 0 Nov 10 22:41 perms.sh*

# --- string --- #
chmod u-x perms.sh 
ll perms.sh 
-rw-rwx--x 1 linuxuser linuxuser 0 Nov 10 22:41 perms.sh*

chmod g-x perms.sh 
ll perms.sh 
-rw-rw---x 1 linuxuser linuxuser 0 Nov 10 22:41 perms.sh*

# --- string set as only --- #
chmod o=r perms.sh 
ll perms.sh 
-rw-rw-r-- 1 linuxuser linuxuser 0 Nov 10 22:41 perms.sh

# --- Recursive with find --- #

```

### chown

Change owner or group of file or directory:

```bash
chown [options] newowner[:newgroup] [ filenames | directories ]
-h # change symlink owner (not original object owner)
-R # recursively changes the owner for all files in a directory

chown newowner:newgroup filenames...                           # change owner and group
chown helen chownDir/                                          # change owner
chown :accounting chownSample.txt                              # change group only
chown linuxuser:testgroup cat.txt counts.txt tar_tests/        # change multiple files and dirs
chown -R helen:accounting TestPermissions/                     # recursively change all files and dirs
chown -h helen symlinkname                                     # change symlink owner
```

### chgrp

Changes the group because originally, `chown` could not set the group, only the user.
- `chown :group` is not portable or standard
- `chown user:` is not standard

```bash
chgrp [options] newgroup filenames

chgrp testgroup alphabet.txt        # change group on file
chgrp -h accounting mysymlink       # change group on symlink
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

