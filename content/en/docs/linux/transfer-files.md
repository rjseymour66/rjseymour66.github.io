---
title: "Transferring files"
# linkTitle: ""
# weight: 1000
# description:
---

## scp

Secure Copy Protocol
- Quickly transfers files in a noninteractive mannger between two systems on a network
- Uses SSH
- Best for small copies you need on the fly, bc if it gets interrupted, you cannot pick back up where you left off
- Will overwrite files on remote host if they have the same name:

```bash
scp [FILE] user@10.20.30.40:/absolute/path/to/target-dir
-C # compresses the file during transfer
-p # preserves file access, modification times, and perms
-r # recursive copy
-v # verbose

# copy from remote to remote
scp user@10.20.30.40:~/home user2@10.20.30.41:~/home/files

# copy greptest.txt from local to remote
scp greptest.txt linuxuser@192.168.10.20:/home/linuxuser

```

## sftp

SSH File Transfer Protocol:
1. Log in like ssh
2. Work in remote with standard commands (ex: `cd`), prepend movements around localhost with `l` (ex: `lcd`)

Good for transferring larger files or archives. Provides interactive experience, for example:
- create directories as needed
- immediately check on transferred files,
- determine remote systems pwd


```bash
sftp username@localhost
username@localhosts password: 
Connected to localhost.
sftp> 

# common commands
bye # exits and quits sftp
exit # exits and quits sftp
get # gets file (downloads) from remote to local machine
reget # resumes interrupted get operation
put # sends (uploads) files from local to remote
reput # resumes interruped put operation
ls # list files in remote pwd
lls # list files in local pwd
mkdir # create dir on remote
lmkdir # create dir on local
progress # toggle progress display on/off (default is on)

# upload file to remote (localhost, here)
sftp username@localhost
username@localhosts password: 
Connected to localhost.

# check remote pwd
sftp> ls
Desktop                  Development              Documents                Downloads                
...

# check local pwd  
sftp> lls
Extract		  Project42_Inc.txz  Project43.txt  Project46.txt      TarStorage
FullArchive.snar  Project42.txt      Project44.txt  Project4x.tar
not-absolute	  Project42.txz      Project45.txt  ProjectVerify.tar

# make directory on local (would normally be remote, but we use localhost)
sftp> lmkdir sftp_files

# verify the dir was made (would be 'ls' on remote)
sftp> lls
Extract		  Project42_Inc.txz  Project43.txt  Project46.txt      sftp_files
...

# upload file
sftp> put Project4x.tar
Uploading Project4x.tar to /home/username/linux-playground/archives/sftp_files/Project4x.tar
Project4x.tar                                                     100%   10KB   7.5MB/s   00:00

# check for file on remote
sftp> ls
Project4x.tar   

# exit
sftp> bye
```