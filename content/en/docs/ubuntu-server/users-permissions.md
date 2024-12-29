---
title: "Users and permissions"
linkTitle: "Users and permissions"
weight: 10
# description:
---

There are three types of Linux users:
- root
- sudoers
- regular users

## sudo

Stands for "substitute user". Do not log in as `root`. Assign superuser privileges to a regular user and perform root operations with `sudo`.


### Common commands

```bash
sudo -i                         # interactive mode - root shell env
sudo -s                         # sudo shell - regular user shell env and bashrc
usermod -aG sudo <username>     # add user to sudoers
journalctl -e /usr/bin/sudo     # check sudo usage history
sudo lastb                      # failed root login attempts
```

### visudo

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

```bash
sudo apt install libpam-passwdqc

# system remembers previous X passwords
# system remembers previous passwords
password	required			pam_pwhistory.so remember=99 use_authok
```

## Groups

Groups let you categorize and manage users in bulk:
- What resources a user can access
- Every file and directory has an assigned user and group
- Each user is a member of a group named with their username

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

### groupdel

Deletes a group:

```bash
groupdel <group>                    # deletes <group>
getent group <group>                # check /etc/group cleanup
sudo find / -gid <GID> 2>/dev/null   # check fs for files associated with deleted group
```