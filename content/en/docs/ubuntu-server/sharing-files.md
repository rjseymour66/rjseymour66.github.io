---
title: "xSharing and transferring files"
# linkTitle: "Sharing files"
weight: 110
# description:
---

## Sharing files

A file server gives users a location to store files for easy access and collaboration:
- Sets up a daemon that accepts connections and shares specified directories to authorized users
- Samba and NFS are the most common technologies. Can run both services on one machine, but they have their use cases
- Need to select a directory to store the file shares

### Samba

Samba is great for Windows and Linux environments
- Accessible because it uses Server Message Blick (SMB) protocol which has wide platform support
- Managing perms is difficult because they are not granted with native Unix/Linux perms
  - You have to force all users to access content as a user specified in the config files, or you can set up Windows Active Directory, which is complicated
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
vim /etc/samba/smbshared.conf                           # 5. Create second config file
mkdir /share                                            # 6. Create shared files
mkdir /share/documents
mkdir /share/public
chown -R <myuser>:<users> /share                        # 7. Grant privileges
testparm                                                # 8. Check Samba config files
systemctl start smbd                                    # 9. Start smbd service
systemctl status smbd                                   # 10. Check status of smbd service


# --- /etc/samba/smb.conf --- #
[global]                                # These settings will impact Samba as a whole - [this] is called a 'stanza'
server string = File Server             # server string is description field for File Server
workgroup = WORKGROUP                   # Windows naming convention - workgroup is a namespace for a collection of machines
security = user                         # users auto to server with usernames and passwds - 'user' means use local users 
map to guest = Bad User                 # Unauth'd users are guests that can access shares with guest perms
name resolve order = host bcast wins    # How to resolve hostnames - wins is deprecated but here for legacy networks
include = /etc/samba/smbshared.conf     # Insert contents of this file as if it were one config file

# --- /etc/samba/smbshared.conf --- #
[Documents]                             # Share name - on Windows path is //<servername>/Documents
path = /share/documents                 # Share location on on host machine
force user = <myuser>                     # Overrides user ownership - users are treated as `<myuser>`, not their user acct
force group = <users>                     # Overrides group ownership - users are treated as members of `<users>` group
public = yes                            # Share is publicly available
writable = no                           # No one can make changes to share contents

[Public]
path = /share/public
force user = <myuser>
force group = <users>
create mask = 0664                      # Setting file permissions
force create mode = 0664
directory mask = 0777                   # Setting directory permissions
force directory mode = 0777
public = yes                            # Share is publicly available
writable = yes                          # Users can make changes
```


### NFS

NFS is great for Linux or Unix environments
- Perms are granted with standard Unix/Linux methods
- Some versions of Windows support the Services for NFS plugin that lets your Windows installation access NFS shares
- Each share is called an _export_
- Requires that you store contents in a designated directory. For example, store NFS shares in `/exports` because that is the main file that NFS reads share info from
- Creates default config file in `/etc/exports`
- Need to mount shares on all clients manually
- Additional config file at `/etc/nfs.conf`

```bash
# --- server steps --- #
mkdir /exports{backup,documents,public} # 1. Create directory for NFS share
apt install nfs-kernel-server           # 2. Install package
mv /etc/exports /etc/exports.orig       # 3. Rename default config file to preserve, before create new one
vim /etc/exports                        # 4. Create new config file
man exports                             # 5. View config options
vim /etc/idmapd.conf                    # 6. UID permission mapping config file
systemctl restart nfs-kernel-server     # 7. Restart NFS
systemctl status nfs-kernel-server      # 8. Check status

exportfs -a                             # Read /etc/exports again and update exports, no downtime
# --- client machine steps --- #
apt install nfs-common                                      # 9. Install package
mount <servername-or-ip>:/<share-name> /mtn/<mnt-point>     # 10. Mount share on local system
mount 10.20.30.40:/documents /mnt/documents                 # no need to add /exports/ because it was declared as 
                                                            # the export root in /etc/exports



# --- /etc/exports config file --- #
# Only machines in 192.168.1.0/24 can view this share
# ro - read only
# rw - read/write in these shares
# no_subtree_check - increases reliability, NFS might complain if omitted
# no_root_squash - lets root user on other system act with root privs in the exports
/exports *(ro,fsid=0,no_subtree_check)                           
/exports/backup 192.168.1.0/255.255.255.0(rw,no_subtree_check)   
/exports/documents 192.168.1.0/255.255.255.0(ro,no_subtree_check)
/exports/public 192.168.1.0/255.255.255.0(rw,no_subtree_check)

# --- /etc/idmapd.conf settings --- #
[General]

Verbosity = 0
# set your own domain here, if it differs from FQDN minus hostname
Domain = mynetworksettings.com          # Uncomment and add domain name

[Mapping]

Nobody-User = nobody
Nobody-Group = nogroup
```

## Transferring files

### rsync

Copies files from one place to another for complex jobs, with many options:
- preserving permissions
- backing up replaced files
- incremental backups
- server to server across the network, or same fs on one server
- each time it runs, it copies what is different from the source to target
- `-a`: wrapper option equivalent to `-rlptgoD`
  - `r`: recursive
  - `l`: symbolic links
  - `p`: preserve permissions
  - `t`: preserve modification times
  - `g`: preserve group ownership
  - `o`: perserve owner
  - `D`: preserves Device files
- When copying over ssh, add trailing slash to copy only dir contents. To copy dir and contents, leave trailing slash off:
  - `/home/me/only-contents/`
  - `/home/me/dir-and-contents`



```bash
rsync -r /home/<user> /backup           # copy /home to /backup (recursive, no perms)
rsync -a /home/<user> /backup           # copy /home to /backup (recursive, maintain perms and metadata)

rsync -av /home/<user> /backup          # copy /home to /backup (verbose, recursive, maintain perms and metadata)
rsync -av /home/<user>/rsync-test/ normaluser@<ip-addr>:/backup  # copy over SSH

rsync -av --delete /src /target         # sync directories - delete files in /target if not in /src. DESTRUCTIVE

# backup files
rsync -avb --delete /src /target        # sync directories - create file backups instead of delete
rsync -avb --delete --backup-dir=/backup/incremental src/ target/ # sync dirs - move removed files to backup-dir

CURDATE=$(date +%m-%d-%Y)
rsync -avb --delete --backup-dir=/backup/incremental/$CURDATE src/ target/      # save in a dated dir
```

### scp

Secure copy - comes with the OpenSSH client:
- Better for "one-off" tasks, single files, small groups of files
- Copy files from local to remote, or remote to local
- Default dir for remote is `/home/<user>`, so can use paths relative to that dir

```bash
scp [FILE] user@10.20.30.40:/absolute/path/to/target-dir
-C # compresses the file during transfer
-p # preserves file access, modification times, and perms
-r # recursive copy
-v # verbose

scp file1 normaluser@<ip-addr>:/home/normaluser/scp-test    # copy file1 to remote dir
scp file1 normaluser@<ip-addr>:                             # copy file1 to /home on remote
scp <user>@<ip-addr>:/home/<user>/scp-test/file2 .          # copy file2 from remote to local


scp -r /home/<user>/scp-test/cp-dir <user2>@<ip-addr>:/home/<user2>/scp-test                # copy dir and contents
scp -P 2222 -r /home/username/scp-test/cp-dir <user2>@<ip-addr>:/home/<user2>/scp-test      # designate port (if not 22)
scp -rv /home/username/scp-test/cp-dir normaluser@$REMOTE:/scp-test                         # verbose dir copy
```

### sftp

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