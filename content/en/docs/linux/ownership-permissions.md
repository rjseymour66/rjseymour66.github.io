---
title: "Ownership and permissions"
weight: 140
---

## File and directory permissions

Linux assigns each file and directory an owner and allows the owner to set basic security settings on the file or directory.

### Ownership

Linux has a three-tiered approach to protecting files and directories:

Owner
: Linux assigns each file and directory is assigned to an owner. The admin can assign the owner privileges to the file or directory.

Group
: Linux can assign each file and directory a single group of users. The admin can assign the group privileges that are different than the owner

Others
: Assigned to any user account that is not the owner or in the assigned group.

```bash
# ubuntu and rocky assign each user account to a separate group with the same name as the user acct
ll
-rw-rw-r--  1 <ownername> <groupname> 31 Mar 18 19:19 alphabet.txt
```

### chown

Change the owner assigned to a file or directory.

```bash
chown [options] newowner [ filenames | directorynames ]
-R # recursively changes the owner for all files in a directory
# change owner
sudo chown linuxuser alphabet.txt 
ll
total 72
-rw-rw-r--  1 linuxuser   groupname   31 Mar 18 19:19 alphabet.txt

# change owner and group at once
chown newowner:newgroup filenames

# example
sudo chown linuxuser:testgroup cat.txt counts.txt tar_tests/
ll
total 72
...
-rw-rw-r--  1 linuxuser   testgroup     31 Mar 18 19:19 alphabet.txt
-rw-rw-r--  1 linuxuser   testgroup     25 Mar 19 23:49 cat.txt
...
-rw-rw-r--  1 linuxuser   testgroup     15 Mar 18 19:19 counts.txt
...
drwxrwxr-x  2 linuxuser   testgroup   4096 Apr  7 09:10 tar_tests/

```

### chgrp

The file or directory owner, or the root user can change the group assigned to the file using `chgrp`:

```bash
chgrp [options] newgroup filenames

# change group on file
sudo chgrp testgroup alphabet.txt 
ll
total 72
...
-rw-rw-r--  1 linuxuser   testgroup     31 Mar 18 19:19 alphabet.txt
```

## Access permissions

Root user and file or directory owner can change access permissions.

After you assign the owner and group, you can assign specific permissions. You can assign each tier of protection (owner, group, others) different permissions:

Read
: Access data in file or directory

Write
: Modify data stored in the file or directory

Execute
: Run the file on the system, or to list the files in the directory

```bash
ll
total 72
...

# file
-<owner><group><others>
-rw-rw-r--  1 linuxuser   testgroup     31 Mar 18 19:19 alphabet.txt

# directory
d<owner><group><others>
drwxrwxr-x  2 linuxuser   testgroup   4096 Apr  7 09:10 tar_tests/
```

### Symbolic mode

Denote permissions with a letter code:

```bash
# 
u # owner
g # group
o # other
a # all

r # read
w # write
x # execute

# separate codes with:
# + if you want to add permission
# - if you want to remove permission
# = to set as only permission

# before
ll
-rw-rw-r--  1 linuxuser   testgroup     31 Mar 18 19:19 alphabet.txt

# remove write permission for group
sudo chmod g-w alphabet.txt 

# after
ll
-rw-r--r--  1 linuxuser   testgroup     31 Mar 18 19:19 alphabet.txt

# read, write, execute to owner and group
ll alphabet.txt 
---------- 1 linuxuser testgroup 31 Mar 18 19:19 alphabet.txt

# command
sudo chmod ug=rwx alphabet.txt 
ll alphabet.txt 
-rwxrwx--- 1 linuxuser testgroup 31 Mar 18 19:19 alphabet.txt*
```

### octal mode

The nine possible permission bits are represented as three octal numbers, one for owner, group, and others:

| Octal value | Permission |
|-------------|------------|
| 0 | `---` |
| 1 | `--x` |
| 2 | `-w-` |
| 3 | `-wx` |
| 4 | `r--` |
| 5 | `r-x` |
| 6 | `rw-` |
| 7 | `rwx` |

```bash
# octal for ugo
sudo chmod 664 alphabet.txt 
ll alphabet.txt 
-rw-rw-r-- 1 linuxuser testgroup 31 Mar 18 19:19 alphabet.txt
```

## Special permissions


### SUID

You can set the _Set User ID_ (SUID) on executable files. 
- Linux runs these files as the file owner, not the account actually running the executable.
- Commonly used in server apps that must run as root, but usually launched by a standard user.

Indicated by an `s` in place of an `x` for the owner permission:

```bash
# SUID set
rwsr-x-r-x

# SUID not set
rwSr-x-r-x

# set the SUID
chmod u+s <executable>
# 4 at beginning
chmod 4750 <executable>
```

### SGID/GUID

_Set Group ID_ works differently for files and dirs:
- Files: tells linux to run the program with the file's group permissions.
- Directories: any files users create i the directory are assigned the group of the directory and not that of the user. This helps create an environment where multiple users can share files. All users in the group have the same perms as all files in the shared dir.

Indicated by an `s` in place of an `x` for the group permission:

```bash
# SGID set
rwxrwsr--

# set the SGID
chmod g+s /dir
# 2 at beginning
chmod 2660 /dir
```

### Sticky bit

Prevents file from being deleted by someone that doesn't own it, even if they belong to a group with write perms on the file. Often used on dirs that are shared by groups, where the group members has read and write access, but only the owner can remove it.

Indicated by a `t` in the execute bit position for others:

```bash
# sticky bit set
rwxrw-r-t

# set sticky bit
chmod o+t /dir
# 1 at the start
chmod 1777 /dir
```

## Default permissions

### umask

The _user mask_ is a mask that is applied to the permission bits on a file or directory to define the default permissions.
- octal value that represents the bits to be removed from the octal mode:
  - octal mode `666` permissions for files
  - octal mode `777` perms for directories
- Consists of four values: `<special-perm><owner><group><other>`. Special permissions:
  - 4: SUID
  - 2: GUID
  - 1: stick bit

| `umask` | Created files | Created directories |
|---------|---------------|---------------------|
| `000` | `666` (`rw-rw-rw-`) | `777` (`rwxrwxrwx`)  |
| `002` | `664` (`rw-rw-r--`) | `775` (`rwxrwxr-x`)  |
| `022` | `644` (`rw-r--r--`) | `755` (`rwxr-xr-x`)  |
| `277` | `400` (`r--------`) | `500` (`r-x------`)  |

[Every possible combo](https://www.linuxtrainingacademy.com/all-umasks/)

When you change umask on the command line, its only for the session.
- set in script in `/etc/profile`
- To persist changes, change it in `$HOME/.bashrc`

```bash
# view umask values
umask
0002

# view permissions
ls -l
drwxrwxr-x 2 linuxuser linuxuser 4096 Apr 10 23:50 test1
-rw-rw-r-- 1 linuxuser linuxuser    0 Apr 10 23:50 test2

# change umask
umask 027
touch test3
ls -l
drwxrwxr-x 2 linuxuser linuxuser 4096 Apr 10 23:50 test1
-rw-rw-r-- 1 linuxuser linuxuser    0 Apr 10 23:50 test2
-rw-r----- 1 linuxuser linuxuser    0 Apr 10 23:51 test3
```

## Access Control Lists

You can only assign file or directory permissions to a single user or group. 
- ACL lets you specifiy a list of multiple users or groups and the permissions that are assigned to them.
- ACL permissionns use the same r,w, and x permission bits but you can assign to multiple users.

### getfacl

View ACLs assigned to a file or directory:

```bash
getfacl test 
# file: test
# owner: linuxuser
# group: linuxuser
user::rw-
group::rw-
other::r--
```

### setfacl

Modify permissions assigned to a file or directory:

```bash
# Define rule:
# u[ser]:uid:perms
# g[roup]:gid:perms
# o[ther]::perms
setfacl [options] rule filenames
-m # modify permissions to file or dir
-x # rm permissions

# add rw perms for sales group to test file
setfacl -m g:sales:rw test

# (+) sign next to perms indicates ACL 
-rw-rw-r--+ 1 linuxuser linuxuser 0 Apr 11 21:54 test

# check ACL perms
getfacl test 
# file: test
# owner: linuxuser
# group: linuxuser
user::rw-
group::rw-
group:sales:rw-
mask::rw-
other::r--

# remove sales group perms
setfacl -x g:sales test

# verify
getfacl test
# file: test
# owner: linuxuser
# group: linuxuser
user::rw-
group::rw-
mask::rw-
other::r--

# use d: option to set default ACL on dir that is inherited
# by any file in the directory.
sudo setfacl -m d:g:sales:rw $(pwd)
touch sales-test

# verify
getfacl sales-test 
# file: sales-test
# owner: linuxuser
# group: linuxuser
user::rw-
group::rwx			#effective:rw-
group:sales:rw-
mask::rw-
other::r--
```

## Context-based permissions

Linux permissions and ACLs are _discretionary access control_ (DAC). They are set by file or directory owner.

_Mandatory access control_ (MAC) lets the system administrator define securty based on the context of an object in the system, which overrides the DAC permissions.
- _Role-based access control_ (RBAC) is a subcategory of MAC, which bases perms on the roles users and processes play in the Linux system.

### SELinux for RHEL

### AppArmor for Ubuntu
