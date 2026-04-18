+++
title = 'Users and groups'
date = '2025-09-07T18:57:06-04:00'
weight = 40
draft = false
+++


Linux organizes users into three types based on the level of access they have:
- `root`: Has unrestricted access to the entire system
- Sudo user: A regular user granted permission to run commands with root privileges
- Regular user: Has access only to their own files and explicitly granted resources

## sudo

`sudo` stands for "superuser do." It lets you run individual commands with elevated privileges without logging in as `root`.

Best practices for using `sudo`:
- Do not log in as `root`. Assign superuser privileges to a regular user and run root operations with `sudo` instead.
- Restrict which commands each user can run as `root` to limit the blast radius of mistakes or compromised accounts.

### Common commands

Common `sudo` commands:

| Command                       | Description                                          |
| :---------------------------- | :--------------------------------------------------- |
| `usermod -aG sudo <username>` | Add a user to the sudo group (Ubuntu)                |
| `sudo !!`                     | Run the previous command as sudo                     |
| `sudo -i`                     | Open a root shell with the root environment          |
| `sudo -s`                     | Open a shell with your own environment and `.bashrc` |
| `journalctl -e /usr/bin/sudo` | View sudo usage history                              |
| `sudo lastb`                  | View failed root login attempts                      |

### visudo

References:
- [How to edit the sudoers file](https://www.digitalocean.com/community/tutorials/how-to-edit-the-sudoers-file)
- [Sudo - An advanced how-to](https://centoshelp.org/security/sudo-an-advanced-howto/)

`visudo` opens the `/etc/sudoers` file in a safe editing mode that validates syntax before saving. Always use `visudo` to edit `/etc/sudoers`. Editing it directly risks locking yourself out if you introduce a syntax error.

Key rules for `/etc/sudoers`:
- Group names are preceded by `%`. Usernames are not.
- Use groups to manage `sudo` access so you can add or remove users from a group instead of editing `/etc/sudoers` directly
- Use the full path to commands (for example, `/usr/bin/apt`) in command rules to prevent privilege escalation through malicious scripts

#### Syntax

Run `sudo visudo` to open the file:

```bash
sudo visudo
```

Each rule follows this format:

```bash
<user_or_group> <hosts>=(<run_as_user>:<run_as_group>) <commands>
```

For example, `root ALL=(ALL:ALL) ALL` breaks down as:

| Field        | Value  | Meaning                     |
| :----------- | :----- | :-------------------------- |
| User         | `root` | The user this rule applies to |
| Hosts        | `ALL`  | Any terminal or host        |
| Run as user  | `ALL`  | Can act as any user         |
| Run as group | `ALL`  | Can act as any group        |
| Commands     | `ALL`  | Can run any command         |

#### Examples

```bash
fred ALL=(ALL) ALL                                 # run all commands as root from any terminal
fred ALL=(ALL) NOPASSWD:ALL                        # run all commands without a password prompt
fred ALL=(ALL) /usr/bin/apt,/sbin/mount            # run only apt and mount as root
fred ALL=(ALL) /usr/bin/apt, NOPASSWD:/sbin/mount  # run apt as root; mount without a password prompt
fred ALL = NOPASSWD:/sbin/reboot                   # run reboot without a password prompt
%admins ALL=(ALL) ALL                              # all members of the admins group can run any command
```

### Set the sudo password timeout

By default, `sudo` caches your password for a period of time before prompting again. Add `timestamp_timeout=<seconds>` to the `Defaults env_reset` line in `/etc/sudoers` to control this interval.

Special values:
- `0`: Prompt for a password every time
- `-1`: Never prompt again after the first authentication

```bash
sudo visudo

Defaults        env_reset,timestamp_timeout=90      # re-prompt after 90 seconds
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
```

## Account files

Linux uses several files during and after account creation. The files in each phase are independent of one another.

**Before account creation:**
- `/etc/default/useradd`
- `/etc/login.defs`
- `/etc/skel`
- Admin input

**After account creation:**
- `/home/<username>`
- `/etc/passwd`
- `/etc/shadow`
- `/etc/group`


### Creation files

#### /etc/login.defs

`/etc/login.defs` is the configuration file for the shadow password suite. It defines default settings for account-related commands such as `useradd`, `userdel`, and `passwd`. Linux applies these settings after the OS installation.

Common directives:

| Directive         | Description                                                           |
| :---------------- | :-------------------------------------------------------------------- |
| `PASS_MAX_DAYS`   | Number of days before a password change is required                   |
| `PASS_MIN_DAYS`   | Number of days after a password change before it can be changed again |
| `PASS_MIN_LENGTH` | Minimum number of characters required in a password                   |
| `PASS_WARN_AGE`   | Number of days before password expiration that a warning is issued    |
| `CREATE_HOME`     | Whether to create a home directory for new accounts. Default is `no`. |
| `ENCRYPT_METHOD`  | Password hashing method                                               |

Linux assigns a User Identification Number (UID) to every account. Humans use account names. Linux uses UIDs internally. System accounts provide services (daemons) or perform special tasks. The `root` account always has UID `0`.

View active directives in `/etc/login.defs`:

```bash
grep -v ^$ /etc/login.defs | grep -v ^\#
MAIL_DIR        /var/mail
...
UID_MIN          1000   # lowest UID allowed for user accounts (sometimes 500)
UID_MAX         60000
GID_MIN          1000   # lowest GID allowed for group accounts
GID_MAX         60000
LOGIN_RETRIES       5
LOGIN_TIMEOUT      60
...
```

View all UIDs sorted numerically:

```bash
gawk -F: '{print $3, $1}' /etc/passwd | sort -n
0 root
1 daemon
2 bin
```

#### /etc/default/useradd

`/etc/default/useradd` stores the default values applied when you run `useradd`. View all active directives with `useradd -D`:

```bash
useradd -D
GROUP=100
HOME=/home              # Must set CREATE_HOME in /etc/login.defs to 'yes'
INACTIVE=-1             # Number of days after password expiration before the account is deactivated
EXPIRE=
SHELL=/bin/sh           # Default shell
SKEL=/etc/skel          # Skeleton directory
CREATE_MAIL_SPOOL=no
```

#### /etc/skel

`/etc/skel` contains template files that Linux copies into every new user's home directory at account creation. Add files to `/etc/skel` to distribute default configurations, such as an editor config or a welcome file, to every new user at account creation.

View the default contents:

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

{{< admonition "Passwords in /etc/passwd" warning >}}
`/etc/passwd` should not contain passwords. If it does, run `pwconv` to migrate them to `/etc/shadow`.
{{< /admonition >}}

`/etc/passwd` stores account information. When you create an account, Linux adds an entry to this file. The file is world-readable because commands like `ls` use it to display ownership information:

```bash
ls -l /etc/passwd
-rw-r--r-- 1 root root 1826 Dec 29 00:14 /etc/passwd
```

Each entry follows the format `username:password:UID:GID:comment:home:shell`:

```bash
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
```

| Field | Description                                                                                                                                                                      |
| :---- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1     | Username                                                                                                                                                                         |
| 2     | Password field. `x` indicates the password is stored in `/etc/shadow`                                                                                                           |
| 3     | UID                                                                                                                                                                              |
| 4     | GID                                                                                                                                                                              |
| 5     | Comment field, traditionally the user's full name                                                                                                                                |
| 6     | Home directory                                                                                                                                                                   |
| 7     | Default shell. `/sbin/nologin` displays a message and ends the session. `/bin/false` ends the session silently. Both prevent interactive log in, typically for service accounts. |


#### /etc/shadow

`/etc/shadow` stores password information for every account, even accounts that have no password set. Only `root` can read this file.

```bash
sudo cat /etc/shadow
root:$y$j9T$EDErVNwiCEqd1Nm5W26d..$aFG6ZaS4.iri2wuCLwuzcexksaNNYDpm1ntYDOGTb03:19804:0:99999:7:::
daemon:*:19773:0:99999:7:::
bin:*:19773:0:99999:7:::
sys:*:19773:0:99999:7:::
...
<username>:$y$j9T$qQNRlSwnLVRW/liifd7HW/$w22t73yRHnNvpqsSgDChfEucV825MollmauAvtbeYp9:19804:0:99999:7:::
```

Date fields use Unix Epoch time expressed in days since January 1, 1970 (not seconds):

| Field | Description                                                                                                                                                              |
| :---- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1     | Username. The only field shared with `/etc/passwd`                                                                                                                       |
| 2     | Password, salted and hashed. `!` or `!!` means no password. `*` means the account cannot log in with a password. `!<hash>` means the account is locked.                 |
| 3     | Date of last password change in Unix Epoch days                                                                                                                          |
| 4     | Number of days after a password change before it can be changed again                                                                                                    |
| 5     | Number of days before a password change is required                                                                                                                      |
| 6     | Number of days before expiration that a warning is issued                                                                                                                |
| 7     | Number of days after expiration before the account is deactivated                                                                                                        |
| 8     | Date the account expires in Unix Epoch days                                                                                                                              |
| 9     | Reserved for future use                                                                                                                                                  |

### Restrict shell access

Service accounts for mail servers (POP3 or SMTP) or FTP/SFTP do not need shell access. Assign them `/usr/sbin/nologin` to prevent interactive log in.

#### Restrict an existing user

1. Check that `/usr/sbin/nologin` is listed in `/etc/shells`:

   ```bash
   cat /etc/shells
   ```

2. If it is not listed, add it:

   ```bash
   sudo vim /etc/shells
   ```

3. Check the user's current shell:

   ```bash
   grep '^neverlogin' /etc/passwd
   neverlogin:x:1003:1003:,,,:/home/neverlogin:/bin/bash
   ```

4. Set the shell to `nologin`:

   ```bash
   usermod -s /usr/sbin/nologin neverlogin
   ```

5. Verify the change:

   ```bash
   getent passwd neverlogin
   neverlogin:x:1003:1003::/home/neverlogin:/usr/sbin/nologin
   ```

#### Create a new user with no shell access

1. Create the user with `nologin` as the shell:

   ```bash
   useradd -s /usr/sbin/nologin neverlogin
   ```

2. Verify:

   ```bash
   getent passwd neverlogin
   neverlogin:x:1003:1003::/home/neverlogin:/usr/sbin/nologin
   ```

#### Restore shell access

1. Assign a valid shell:

   ```bash
   chsh -s /usr/bin/bash neverlogin
   ```

2. Verify:

   ```bash
   getent passwd neverlogin
   neverlogin:x:1003:1003::/home/neverlogin:/usr/bin/bash
   ```

## Users

A user account represents an identity on the system. Users can be people or service accounts that run automated tasks. All user home directories are stored under `/home`:

```bash
ls -l /home/
total 8
drwxr-x--- 2 jdoe       jdoe       4096 Dec 28 23:51 jdoe
drwxr-x--- 4 normaluser normaluser 4096 Dec 27 21:44 normaluser
```

### root

The `root` account has unrestricted access to every file and command on the system. After installation, you have the `root` user and the administrator account you created during setup. The administrator is a regular user with `sudo` access. You must explicitly grant `sudo` access to other accounts.

Ubuntu locks the `root` account after installation. Run commands that require elevated privileges with `sudo` instead of logging in as `root`. You must set the `root` password after installation.

Switch to `root` from an account that has root access:

```bash
sudo su -
```
### adduser vs useradd

Use either command to create a user account:

| Command   | Description |
| :-------- | :---------- |
| `adduser` | Interactive prompt that guides you through account creation, including setting a password. Easier to use but not available on all distributions. |
| `useradd` | Low-level utility with more flags and options. Available on all Linux distributions. |

Create a user and assign them to a group at creation:

```bash
adduser --ingroup <group-name> <username>
```

### useradd

`useradd` creates a user account. It does not set a password or create a home directory by default. You must specify those explicitly. Common options:

| Option | Description |
| :----- | :---------- |
| `-c` | Comment field, traditionally the user's full name |
| `-d` | Home directory path |
| `-D` | Display directives from `/etc/default/useradd` |
| `-e` | Account expiration date (`EXPIRE` directive) |
| `-f` | Days after password expiration before the account is deactivated. `-1` means never deactivate (`INACTIVE` directive) |
| `-g` | Primary group |
| `-G` | Additional group memberships |
| `-m` | Create a home directory (`CREATE_HOME` directive) |
| `-M` | Do not create a home directory |
| `-r` | Create a system account instead of a user account |
| `-s` | Default shell (`SHELL` directive) |
| `-u` | UID |

Create a user with a home directory:

```bash
useradd -d /home/jdoe -m jdoe
```

### adduser

`adduser` is a Perl script that calls `useradd` under the hood. It prompts you for a password and user details at creation and prints verbose output. It is not available on all distributions.

Create a new user:

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

`usermod` modifies an existing user account. Use it to update group membership, shell, home directory, or lock and unlock accounts. Common options:

| Option | Description |
| :----- | :---------- |
| `-a` | Append user to a group (use with `-G`) |
| `-c` | Modify the comment field |
| `-d` | Set a new home directory |
| `-e` | Modify account expiration date |
| `-f` | Days after password expiration before account is deactivated |
| `-g` | Change primary group |
| `-G` | Update additional group memberships |
| `-l` | Change the account username |
| `-L` | Lock the account |
| `-m` | Move home directory contents to new location (use with `-d`) |
| `-s` | Change the default shell |
| `-u` | Change the UID |
| `-U` | Unlock the account |

#### Add a user to a group

```bash
usermod -aG <group> <username>    # add as secondary group
usermod -g <group> <username>     # set as primary group
```

#### Lock and unlock an account

```bash
sudo usermod -L linuxuser               # lock account
sudo passwd -S linuxuser                # display status
linuxuser L 03/31/2024 0 99999 7 -1

sudo getent shadow linuxuser            # verify lock in shadow file
linuxuser:!$y$...:19813:0:99999:7:::

sudo usermod -U linuxuser               # unlock
sudo passwd -S linuxuser                # display status
linuxuser P 03/31/2024 0 99999 7 -1
```

### userdel

`userdel` removes a user account. Verify that you do not need any of the user's files before deleting the account. By default, it does not remove the user's home directory. Use `-r` when you are certain the account and its data are no longer needed:

| Command | Description |
| :------ | :---------- |
| `userdel <username>` | Delete the user account only |
| `userdel -r <username>` | Delete the user account, home directory, and mail spool |


### chsh

`chsh` changes the default shell for a user account. Run `cat /etc/shells` first to see which shells are available on the system.

Change a user's shell:

```bash
cat /etc/shells                 # list available shells
grep ^jdoe /etc/passwd          # check current shell
jdoe:x:1001:1001::/home/jdoe:/bin/sh

sudo chsh -s /bin/bash jdoe     # change shell

grep ^jdoe /etc/passwd          # verify
jdoe:x:1001:1001::/home/jdoe:/bin/bash
```

### Switching users

`su` switches your session to another user account. The `-` flag loads the target user's environment, including their shell profile and home directory:

| Command | Description |
| :------ | :---------- |
| `sudo su -` | Switch to root using your own password |
| `su - <username>` | Switch to another account using that account's password |
| `sudo su - <username>` | Switch to another account using your own password |

### Lock and unlock accounts

`passwd` can lock and unlock a user account. A locked account has `L` in the status output. An unlocked account has `P`.

Lock an account:

```bash
passwd -l jdoe
passwd: password changed.

sudo passwd -S jdoe
jdoe L 2024-12-29 0 99999 7 -1
```

Unlock an account:

```bash
passwd -u jdoe
passwd: password changed.

sudo passwd -S jdoe
jdoe P 2024-12-29 0 99999 7 -1
```

### Rocky Linux

On Rocky Linux, `/etc/login.defs` creates home directories by default and sets the shell to `/bin/bash`. Create a user with:

```bash
grep CREATE_HOME /etc/login.defs
CREATE_HOME     yes
useradd -D | grep -i shell
SHELL=/bin/bash

sudo useradd <username>
```

### Ubuntu

On Ubuntu, `/etc/login.defs` does not create home directories by default and sets the shell to `/bin/sh`. Specify both options explicitly:

```bash
grep CREATE_HOME /etc/login.defs
useradd -D | grep -i shell
SHELL=/bin/sh

sudo useradd -md /home/linuxuser -s /bin/bash linuxuser

grep ^linuxuser /etc/passwd
linuxuser:x:1001:1001::/home/linuxuser:/bin/bash

sudo grep ^linuxuser /etc/shadow
linuxuser:!:19813:0:99999:7:::

sudo ls -a /home/linuxuser/
.  ..  .bash_logout  .bashrc  .profile
```

### getent

`getent` retrieves entries from Name Service Switch (NSS) libraries, which are configured in `/etc/nsswitch.conf`. Unlike `cat /etc/passwd`, it honors security settings. For example, reading `/etc/shadow` requires root privileges even through `getent`.

Look up a user entry:

```bash
getent passwd linuxuser
linuxuser:x:1001:1001::/home/linuxuser:/bin/bash
```

Look up a shadow entry (requires root):

```bash
sudo getent shadow linuxuser
linuxuser:!:19813:0:99999:7:::
```


## Querying users

These commands let you audit active sessions, account details, and login history.

### whoami

`whoami` prints the username of the currently logged-in user:

```bash
whoami
```

### who

`who` lists every user currently logged into the system, along with their terminal and login time:

```bash
who
<username> tty2         2024-03-24 16:48 (tty2)
```

### w

`w` extends `who` with additional detail about each logged-in user. The header line shows the current time, system uptime, number of users, and load averages for the past 1, 5, and 15 minutes. Key output columns:

- `JCPU`: Total CPU time the account has used
- `PCPU`: CPU time the current command has used
- `WHAT`: The command the account is currently running

```bash
w
 08:59:08 up 14:54,  1 user,  load average: 0.19, 0.13, 0.10
USER       TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
<username> tty2     tty2             24Mar24 23:24m  0.01s  0.01s /usr/libexec/gnome-session-binary --session=ubuntu
```

### id

`id` displays the UID, primary group GID, and all group memberships for a user. It is commonly used in shell scripts to check identity or group membership. Common options:

| Option | Description |
| :----- | :---------- |
| `-g` | Display the current group's GID |
| `-G` | Display all group membership GIDs |
| `-n` | Display the account name instead of UID |
| `-u` | Display the account UID |

Look up the current user:

```bash
id
uid=1000(<current-user>) gid=1000(<current-user>) groups=1000(<current-user>),27(sudo),999(vboxsf)
```

Look up a specific user:

```bash
id linuxuser
uid=1001(linuxuser) gid=1001(linuxuser) groups=1001(linuxuser)
```

Look up a user by UID:

```bash
id -un 1001
linuxuser
```

`id` is also used in shell scripts to resolve the current username. For example, `/etc/profile` uses it to set the `USER` variable:

```bash
grep USER /etc/profile
USER="`/usr/bin/id -un`"
```

### last

`last` reads `/var/log/wtmp` and displays a list of user logins, logouts, and system reboots. Use `-f` to read from a specific log file:

```bash
last
<username> tty2         tty2             Sun Mar 24 16:48    gone - no logout
reboot     system boot  6.5.0-26-generic Sun Mar 24 16:47   still running
<username> tty2         tty2             Sat Mar 23 09:19 - crash (1+07:27)
reboot     system boot  6.5.0-26-generic Sat Mar 23 09:04   still running
...
```

Read from a specific log file:

```bash
last -f /var/log/wtmp.1
```


## Passwords

Password data is stored in `/etc/passwd` and `/etc/shadow`. Both files are described in [Account files](#account-files).

### passwd

`passwd` sets or updates a user's password. Common options:

| Option | Description |
| :----- | :---------- |
| `-d` | Remove the account password |
| `-e` | Expire the password immediately, forcing a change at next login |
| `-i` | Days after password expiration before the account is deactivated |
| `-l` | Lock the account by placing `!` before the password in `/etc/shadow` |
| `-n` | Minimum days before the user can change the password again |
| `-S` | Display password status (`P` = usable, `NP` = no password, `L` = locked) |
| `-u` | Unlock the account by removing `!` from `/etc/shadow` |
| `-w` | Days before expiration that a warning is issued |
| `-x` | Days until a password change is required |

Change the current user's password:

```bash
passwd
```

Change another user's password (requires root):

```bash
sudo passwd <username>
```

Display password status:

```bash
sudo passwd -S linuxuser
linuxuser P 03/31/2024 0 99999 7 -1
```



### chage

`chage` displays and updates password aging information. Its output is more readable than `passwd -S`:

| Command | Description |
| :------ | :---------- |
| `sudo chage -l <username>` | Display password aging details |
| `sudo chage <username>` | Interactively update password settings |

### Set password expiration

View the current password aging settings for an account:

```bash
chage -l normaluser
Last password change          : Dec 27, 2024
Password expires              : never
Password inactive             : never
Account expires               : never
Minimum number of days between password change    : 0
Maximum number of days between password change    : 99999
Number of days of warning before password expires : 7
```

Common `chage` commands for setting expiration policy:

| Command | Description |
| :------ | :---------- |
| `sudo chage -d 0 <username>` | Force a password change at next login |
| `sudo chage -M 90 <username>` | Require a password change every 90 days |
| `sudo chage -m 10 <username>` | Prevent password changes within 10 days of the last change |

### Set password policy

{{< admonition "Test authentication changes safely" warning >}}
When making authentication changes, keep a root shell open in a separate terminal. Test your changes from a second shell before closing the root session.
{{< /admonition >}}

Password policy is managed through Pluggable Authentication Modules (PAM). PAM packages extend authentication features and are configured in `/etc/pam.d/common-password`.

Install the `libpam-passwdqc` package to enforce password quality requirements:

```bash
sudo apt install libpam-passwdqc
```

Edit `/etc/pam.d/common-password` to configure policy. For example, configure the system to remember previous passwords:

```bash
sudo vim /etc/pam.d/common-password
...
password    required    pam_pwhistory.so remember=99 use_authok
```

## Groups

{{< admonition "Do not use group passwords" warning >}}
Grant group access through membership, not passwords. Set passwords on user accounts instead.
{{< /admonition >}}

Groups let you control which resources users can access. Every file and directory has an assigned user and group. Groups are part of Linux's discretionary access control (DAC), where access to an object is determined by the user's identity and group membership.

Each user belongs to a default group, which is their active group at login. A process can have only one active group at a time, but a user can be a member of many groups. If no default group is specified at account creation, Linux creates a new group with the same name as the username. Groups are tracked in `/etc/group`. Group passwords are stored in `/etc/gshadow`.

### View group membership

`/etc/group` stores all group definitions. Each entry follows the format `name:password:GID:members`, where `members` is a comma-separated list of users in the group:

```bash
cat /etc/group
root:x:0:
...
plugdev:x:46:normaluser
...
normaluser:x:1000:
testuser:x:1002:
```

Check group membership for a user:

| Command | Description |
| :------ | :---------- |
| `groups` | List groups the current user belongs to |
| `groups <username>` | List groups a specific user belongs to |

### groupadd

`groupadd` creates a new group and adds it to `/etc/group`. A group must exist before you can add users to it. Use `-g` to specify a GID:

```bash
sudo groupadd admins
grep admins /etc/group
admins:x:1003:

sudo groupadd -g 1966 cream
getent group cream
cream:x:1966:
```

Verify that no group password is set:

```bash
sudo getent gshadow cream
cream:!::
```

### gpasswd

`gpasswd` administers `/etc/group` and `/etc/gshadow`. Use it to add and remove users from groups:

| Command | Description |
| :------ | :---------- |
| `gpasswd -a <username> <group>` | Add a user to a group |
| `gpasswd -d <username> <group>` | Remove a user from a group |

For example, add and remove `jdoe` from the `admins` group:

```bash
gpasswd -a jdoe admins
Adding user jdoe to group admins

gpasswd -d jdoe admins
Removing user jdoe from group admins
```

### groupmod

`groupmod` modifies an existing group. Common options:

| Option | Description |
| :----- | :---------- |
| `-g` | Change the GID |
| `-n` | Rename the group |

Rename a group:

1. View the current group entry:

   ```bash
   getent group cream
   cream:x:1966:linuxuser
   ```

2. Rename the group:

   ```bash
   sudo groupmod -n Cream cream
   ```

3. Confirm the change in `/etc/group`:

   ```bash
   getent group Cream
   Cream:x:1966:linuxuser
   ```

### groupdel

`groupdel` removes a group. After deleting a group, check the filesystem for files that were owned by that group and reassign them:

1. Delete the group:

   ```bash
   groupdel <group>
   ```

2. Confirm the group was removed from `/etc/group`. [`getent`](#getent) queries the Name Service Switch libraries and returns no output if the group no longer exists:

   ```bash
   getent group <group>
   ```

3. Find any files still associated with the deleted GID:

   ```bash
   sudo find / -gid <GID> 2>/dev/null
   ```
### newgrp

`newgrp` switches your active group for the current session. You can only have one active group at a time. Files you create after switching belong to the new group.

Switch to the `plugdev` group and create a file:

1. Check your current active group (listed first in `id` output):

   ```bash
   id
   uid=1000(linuxuser) gid=1000(linuxuser) groups=1000(linuxuser),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),101(lxd)
   ```

2. Switch to `plugdev`:

   ```bash
   newgrp plugdev
   ```

3. Verify the active group has changed:

   ```bash
   id
   uid=1000(linuxuser) gid=46(plugdev) groups=46(plugdev),4(adm),24(cdrom),27(sudo),30(dip),101(lxd),1000(linuxuser)
   ```

4. Create a file and confirm group ownership:

   ```bash
   touch plugdevfile
   ls -l plugdevfile
   -rw-r--r-- 1 linuxuser plugdev 0 Dec 14 19:28 plugdevfile
   ```

## Permissions

Linux permissions control what users and groups can do with files and directories. Three permission types apply to three categories: owner, group, and world (everyone else):

| Permission | File | Directory |
| :--------: | :--- | :-------- |
| `r` | File can be read | Can view directory contents |
| `w` | File can be written to | Can change directory contents |
| `x` | File can be executed as a program | Can `cd` into the directory |

Each file listing shows the permission string, link count, owner, group, size, date, and filename:

```bash
ls -l file.1
-rw-rw-r-- 1 linuxuser linuxuser 23 Nov  9 14:50 file.1
 |   |  |       |         |
owner| world  owner     group
   group
```

The first character of the permission string indicates the object type:

- `-`: regular file
- `d`: directory
- `l`: symbolic link

The `-logh` flags combine long format (`-l`) with human-readable sizes (`-h`) while omitting the owner (`-o`) and group (`-g`) columns, which is useful for comparing object types at a glance:

```bash
ls -logh
drwxrwxr-x 2 4.0K Dec 30 00:59 directory
-rw-rw-r-- 1    0 Dec 30 00:59 file
lrwxrwxrwx 1    4 Dec 30 01:00 link -> file
```

View permissions in octal form:

```bash
stat -c %a chownSample.txt
```

### chmod

`chmod` changes the permissions on a file or directory. Specify permissions using numeric (octal) or string notation. For recursive changes across mixed file and directory trees, use `find` to apply different permissions to each type.

Permissions are represented as:

| Permission | Character | Octal |
| :--------- | :-------: | :---: |
| read | `r` | `4` |
| write | `w` | `2` |
| execute | `x` | `1` |

String notation uses operators to modify permissions:

| Action | Operator |
| :----- | :------: |
| Add | `+` |
| Remove | `-` |
| Set as only permission | `=` |

Set permissions using octal notation:

```bash
chmod 771 perms.sh
ll perms.sh
-rwxrwx--x 1 linuxuser linuxuser 0 Nov 10 22:41 perms.sh*
```

Remove execute permission from the owner using string notation:

```bash
chmod u-x perms.sh
ll perms.sh
-rw-rwx--x 1 linuxuser linuxuser 0 Nov 10 22:41 perms.sh*
```

Set world permission to read-only:

```bash
chmod o=r perms.sh
ll perms.sh
-rw-rw-r-- 1 linuxuser linuxuser 0 Nov 10 22:41 perms.sh
```

Apply permissions recursively with `find`:

```bash
find /path/to/dir -type f -exec chmod 644 {} \;
find /path/to/dir -type d -exec chmod 755 {} \;
```

### chown

`chown` changes the owner or group of a file or directory. Common options:

| Option | Description |
| :----- | :---------- |
| `-h` | Change the symlink owner instead of the target file |
| `-R` | Recursively change ownership for all files in a directory |

Common usage:

| Command | Description |
| :------ | :---------- |
| `chown <owner>:<group> <file>` | Change owner and group |
| `chown <owner> <dir>/` | Change owner only |
| `chown :<group> <file>` | Change group only |
| `chown -R <owner>:<group> <dir>/` | Recursively change owner and group |
| `chown -h <owner> <symlink>` | Change symlink owner |

### chgrp

`chgrp` changes the group ownership of a file or symlink. It predates group support in `chown`, which is not portable across all systems. Use `chgrp` when you need to change only the group:

| Command | Description |
| :------ | :---------- |
| `chgrp <group> <file>` | Change the group on a file |
| `chgrp -h <group> <symlink>` | Change the group on a symlink |



## Troubleshooting

### Check password

Verify that a user account is configured correctly and has a valid password. A `!` at the start of the password field in `/etc/shadow` means the account is locked:

```bash
sudo getent passwd linuxuser
linuxuser:x:1001:1001::/home/linuxuser:/bin/bash

sudo getent shadow linuxuser
linuxuser:$y$j9T$f4O7G12v7ecON6j8SAfiQ.$7kRt7qYpxngkDPaHATTNBlDGc6hHQc7sPuAW9iMKAJ.:19813:0:99999:7:::
```

### last, lastlog, and lastb

Three commands help you audit login history. Each reads from a different log file:

| Command | File | Description |
| :------ | :--- | :---------- |
| `last` | `/var/log/wtmp`, `/var/log/wtmp.*` | All logins and reboots |
| `lastlog` | `/var/log/lastlog` | Most recent login for each user |
| `lastb` | `/var/log/btmp` | Failed login attempts |

View all logins for a specific user:

```bash
sudo last -f /var/log/wtmp | grep linuxus
linuxuse pts/1        10.0.2.2         Mon Apr 22 08:31 - 08:34  (00:03)
linuxuse pts/0        10.0.2.2         Fri Apr 19 20:44    gone - no logout
...
```

View the most recent login for each user:

```bash
lastlog
Username         Port     From             Latest
root                                       **Never logged in**
...
linuxuser        pts/1    10.0.2.2         Mon Apr 22 08:31:02 -0400 2024
```

View failed login attempts:

```bash
sudo lastb -f /var/log/btmp
linuxuse ssh:notty    127.0.0.1        Thu Apr 18 23:04 - 23:04  (00:00)
...
```

### sudo privileges

If a user cannot run commands with `sudo`, verify their group membership and the rules in `/etc/sudoers`.

Check `/etc/sudoers` for the relevant group rules:

```bash
sudo cat /etc/sudoers
...
%admin ALL=(ALL) ALL
%sudo  ALL=(ALL:ALL) ALL
```

Confirm the user belongs to the correct group:

```bash
id linuxuser
uid=1001(linuxuser) gid=1001(linuxuser) groups=1001(linuxuser),27(sudo)
```
### GUI status

If a user can log into the terminal but cannot log into the GUI, check whether the graphical target is active:

```bash
sudo systemctl status graphical.target
● graphical.target - Graphical Interface
     Loaded: loaded (/lib/systemd/system/graphical.target; static)
     Active: active since Thu 2024-04-18 22:12:54 EDT; 5 days ago
       Docs: man:systemd.special(7)

Apr 18 22:12:54 ubuntu22 systemd[1]: Reached target Graphical Interface.
```

### Terminal status

When a user cannot log in at the terminal, check three things: whether multi-user mode is active, whether the terminal device files are intact, and whether the getty services are running. Getty services provide the login prompts for text-based terminals.

#### Multi-user mode

Verify that multi-user mode is active:

```bash
sudo systemctl status multi-user.target
● multi-user.target - Multi-User System
     Loaded: loaded (/lib/systemd/system/multi-user.target; static)
     Active: active since Thu 2024-04-18 22:12:54 EDT; 5 days ago
```

#### Terminal device files

Each terminal device file should start with `c` (character file). A `-` indicates a corrupted terminal that must be rebuilt with `mknod`:

```bash
ls -l /dev/tty?
crw--w---- 1 root tty 4, 0 Apr 21 09:44 /dev/tty0
crw--w---- 1 gdm  tty 4, 1 Apr 21 09:44 /dev/tty1
crw--w---- 1 root tty 4, 2 Apr 21 09:44 /dev/tty2
...
```

#### Getty services

Verify that the getty services are running:

```bash
sudo systemctl status getty.target
● getty.target - Login Prompts
     Loaded: loaded (/lib/systemd/system/getty.target; static)
     Active: active since Thu 2024-04-18 22:12:29 EDT; 5 days ago
```

### Locked accounts

Check the account status with `passwd -S`. An `L` in the second field means the account is locked. A `!` at the start of the password field in `/etc/shadow` confirms the lock.

1. Check the account status (`L` means locked):

   ```bash
   sudo passwd -S linuxuser
   linuxuser L 03/31/2024 0 99999 7 -1
   ```

2. Confirm the lock in `/etc/shadow`. A `!` before the password hash confirms it:

   ```bash
   sudo getent shadow linuxuser
   linuxuser:!$y$j9T$f4O7G12v7ecON6j8SAfiQ.$7kRt7qYpxngkDPaHATTNBlDGc6hHQc7sPuAW9iMKAJ.:19813:0:99999:7:::
   ```

3. Unlock the account:

   ```bash
   sudo usermod -U linuxuser
   ```

4. Verify the account is unlocked (`P` means the password is set and usable):

   ```bash
   sudo passwd -S linuxuser
   linuxuser P 03/31/2024 0 99999 7 -1
   ```

### Expired accounts

If a user cannot log in and the account appears active, check whether the account or password has expired. `chage -l` shows all expiration settings:

```bash
sudo chage -l linuxuser
Last password change          : Mar 31, 2024
Password expires              : never
Password inactive             : never
Account expires               : never
Minimum number of days between password change    : 0
Maximum number of days between password change    : 99999
Number of days of warning before password expires : 7
```

### Authentication

If account status, locks, and expiration are all clear, the issue may be in the authentication stack. Check the following:

- PAM modules and configuration in `/etc/pam.d/`
- `pam_tally2` or `faillock` for failed login lockouts
- Identity provider (IdP) configuration (LDAP, Kerberos)
- SELinux or AppArmor policies

On SELinux systems, check for policy violations with `sealert`:

```bash
sealert -a /var/log/audit/audit.log
100% done
found 0 alerts in /var/log/audit/audit.log
```

