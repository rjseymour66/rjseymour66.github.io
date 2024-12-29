---
title: "Users and permissions"
linkTitle: "Users and permissions"
weight: 10
# description:
---



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


## Groups

- Groups let you manage in bulk what files and directories specific users have access to

## root

- `root` is the most powerful user and can perform any task or access any file or directory in the system
- After installation, you have the `root` user and the administrator user account you created
  - The admin should have their own account and use `sudo` to switch to `root` when needed
  - The admin user is really just a regular user that has `sudo` access by default
  - You have to explicitly grant other user accounts access to `sudo`
- Ubuntu locks out the `root` account after installation. You should run individual commands that require elevated privileges with `sudo`