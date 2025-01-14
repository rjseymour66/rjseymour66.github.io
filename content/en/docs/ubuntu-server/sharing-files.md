---
title: "Sharing files"
# linkTitle: "Sharing files"
weight: 110
# description:
---

A file server gives users a location to store files for easy access and collaboration:
- Sets up a daemon that accepts connections and shares specified directories to authorized users
- Samba and NFS are the most common technologies. Can run both services on one machine, but they have their use cases
- Need to select a directory to store the file shares

## Samba

Samba is great for Windows and Linux environments
- Accessible because it uses Server Message Blick (SMB) protocol which has wide platform support
- Managing perms is difficult because they are not granted with native Unix/Linux perms
- Can store Samba shares in `/share/documents` and `/share/public` dirs
- After install, runs `smbd` daemon
- Default configuration file at `/etc/samba/smb.conf`
- Can split config between multiple files, for example:
  - `/etc/samba/smb.conf`
  - `/etc/samba/smbshared.conf`

```bash
apt install samba                                       # 1. install package
systemctl stop smbd                                     # 2. stop during initial config
mv /etc/samba/smb.conf /etc/samba/smb.conf.orig         # 3. rename config file before you create new one
vim /etc/samba/smb.conf                                 # 4. Config file #1

[global]                                # These settings will impact Samba as a whole - [this] is called a 'stanza'
server string = File Server             # server string is description field for File Server
workgroup = WORKGROUP                   # Windows naming convention - workgroup is a namespace for a collection of machines
security = user                         # users auto to server with usernames and passwds - 'user' means use local users 
map to guest = Bad User                 # Unauth'd users are guests that can access shares with guest perms
name resolve order = host bcast wins    # How to resolve hostnames - wins is deprecated but here for legacy networks
include = /etc/samba/smbshared.conf     # Insert contents of this file as if it were one config file

vim /etc/samba/smbshared.conf                           # 5. Create second config file

[Documents]                             # Share name - on Windows path is //<servername>/Documents
path = /share/documents                 # Share location on on host machine
force user = myuser                     # Overrides user ownership - users are treated as `myuser`, not their user acct
force group = users                     # Overrides group ownership - users are treated as members of `users` group
public = yes                            # Share is publicly available
writable = no                           # No one can make changes to share contents

[Public]
path = /share/public
force user = myuser
force group = users
create mask = 0664
force create mode = 0664
directory mask = 0777
force directory mode = 0777
public = yes
writable = yes


```


## NFS

NFS is great for Linux or Unix environments
- Perms are granted with standard Unix/Linux methods
- Some versions of Windows support the Services for NFS plugin that lets your Windows installation access NFS shares
- Can store NFS shares in `/exports`

```bash

```