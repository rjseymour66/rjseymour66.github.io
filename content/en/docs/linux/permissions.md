---
title: "Permissions"
linkTitle: "Permissions"
# weight: 1000
# description:
---

Discretionary access control (DAC)
: Flexible access control model where the owner of a resource (file, directory, etc.) determines who can access it and what actions they can perform.

Discretionary access control (MAC)
: Strict access control model where administrators define and enforce security policies, and users cannot alter these policies.

## Viewing permissions

```bash
ls -l file.1 
-rw-rw-r-- 1 linuxuser linuxuser 23 Nov  9 14:50 file.1
 |   |  |       |         |     \_____________/   |
owner| other  owner     group         |        filename
   group                         Most recent edit

```

## Assigning permissions

Set 
## chmod

Change file or directory permissions based on the specified mode. Set permissions with one of the following systems:
- string 
- numeric (octal)
- umask

#### Shortcuts

| Permission | Character | Number |
|:-:|:-:|:-:|
| read    | `r` | 4 |
| write   | `w` | 2 |
| execute | `x` | 1 |

#### String actions

| Action | Operator |
|:-:|:-:|
| add | `+` |
| remove | `-` |
| set as only permission | `=` |

```bash
# numeric perms 
chmod 771 perms.sh
ll perms.sh 
-rwxrwx--x 1 linuxuser linuxuser 0 Nov 10 22:41 perms.sh*

# string
chmod u-x perms.sh 
ll perms.sh 
-rw-rwx--x 1 linuxuser linuxuser 0 Nov 10 22:41 perms.sh*

chmod g-x perms.sh 
ll perms.sh 
-rw-rw---x 1 linuxuser linuxuser 0 Nov 10 22:41 perms.sh*

# string set as only
chmod o=r perms.sh 
ll perms.sh 
-rw-rw-r-- 1 linuxuser linuxuser 0 Nov 10 22:41 perms.sh
```



## chown

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

## chgrp

Changes the group because originally, `chown` could not set the group, only the user.
- `chown :group` is not portable or standard
- `chown user:` is not standard

```bash
chgrp [options] newgroup filenames

chgrp testgroup alphabet.txt        # change group on file
chgrp -h accounting mysymlink       # change group on symlink
```

## Associate file to group, then granting privileges

```bash
# create group
groupadd app-data-group

# change owner to <user>:<group>
chown linuxuser:app-data-group datafile.txt 

ll datafile.txt 
-rwxrwx---. 1 linuxuser app-data-group 44 Nov 24 22:19 datafile.txt*

# add user to group
usermod -aG app-data-group <other-user>
su <other-user>
cat datafile.txt 
# // datafile contents
```

## Access Control Lists

You can only assign file or directory permissions to a single user or group.
- Tacks on specific permissions without getting rid of the base perms
- ACL lets you specifiy a list of multiple users or groups and the permissions that are assigned to them.
- ACL permissionns use the same r,w, and x permission bits but you can assign to multiple users.

### getfacl

View ACLs assigned to a file or directory:

```bash
getfacl text.cfg 
# file: text.cfg
# owner: linuxuser
# group: linuxuser
user::rw-
group::rw-
other::r--

# modify (-m) rw perms for dummy user
setfacl -m u:dummy:rw text.cfg
getfacl text.cfg 
# file: text.cfg
# owner: linuxuser
# group: linuxuser
user::rw-
user:dummy:rw-
group::rw-
mask::rw-
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
-b # erase default ACL on dir
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

# revoke all perms
setfacl -m u:<user>:- <file>
```

### mask

Limits the permisssions that you can assign on the file for groups, users, and group owner.
- Does not effect the file owner or `other` group
- `effective`ly limits permissions
- mask permissions are recalculated when you set new perms with `setfacl`

```bash
# set only reading perms
setfacl -m mask:r text.cfg
# 'effectively' limits perms for user and group, even though they have rw perms
getfacl text.cfg 
# file: text.cfg
# owner: linuxuser
# group: linuxuser
user::rw-
user:dummy:rw-			#effective:r--
group::rw-			   #effective:r--
mask::r--
other::r--
```

### Default ACLs

Assigned to a directory:
- Use the `-d` option
- Doesn't set ACL on directory, sets ACL on files in dir

```bash
setfacl -d -m u:dummy:rw test
getfacl test/
# file: test/
# owner: linuxuser
# group: linuxuser
user::rwx
group::rwx
other::r-x
default:user::rwx
default:user:dummy:rw-     # default ACL 
default:group::rwx
default:mask::rwx
default:other::r-x


touch test/file.cfg
getfacl test/file.cfg 
# file: test/file.cfg
# owner: linuxuser
# group: linuxuser
user::rw-
user:dummy:rw-             # default ACL
group::rwx			#effective:rw-
mask::rw-
other::r--

# erase default ACL
setfacl -b test/

touch test/erased.txt
getfacl test/erased.txt 
# file: test/erased.txt
# owner: linuxuser
# group: linuxuser
user::rw-
group::rw-
other::r--
```