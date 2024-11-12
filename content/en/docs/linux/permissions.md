---
title: "Permissions"
linkTitle: "Permissions"
# weight: 1000
# description:
---

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

Change owner of file or directory:

```bash
chown [options] newowner[:newgroup] [ filenames | directories ]
-R # recursively changes the owner for all files in a directory

# change owner and group at once
chown newowner:newgroup filenames

# change user or group
sudo chown otheruser:otheruser owner.file 
ll owner.file 
-rw-rw-r-- 1 otheruser otheruser 0 Nov 10 22:47 owner.file
```