+++
title = 'Sharing files'
date = '2025-09-07T19:03:23-04:00'
weight = 30
draft = false
+++


A file server gives users a central location to store and share files. It runs a daemon
that accepts connections and shares specified directories with authorized users.

Samba and NFS are the most common file-sharing technologies on Linux. You can run both
on one machine, but each suits different environments: Samba works well in mixed
Windows/Linux networks, while NFS is designed for Linux and Unix environments. Before
configuring either service, choose a directory to store your file shares.

## Samba

Samba works well in mixed Windows and Linux environments. It uses the Server Message
Block (SMB) protocol, which has wide platform support.

Permission management in Samba differs from standard Unix permissions. Rather than using
native Unix permissions, you force all users to access content as a specified system user.
Alternatively, you can integrate with Windows Active Directory.

After installation, Samba runs the `smbd` daemon. The default configuration file is
`/etc/samba/smb.conf`. You can split the configuration across multiple files:

- `/etc/samba/smb.conf`
- `/etc/samba/smbshared.conf`

Store Samba shares in dedicated directories, for example `/share/documents` and `/share/public`.

### Install Samba

Install and configure Samba to serve file shares:

1. Install the package.
   ```bash
   apt install samba
   ```
2. Stop the service during initial configuration.
   ```bash
   systemctl stop smbd
   ```
3. Back up the default configuration file before creating a new one.
   ```bash
   mv /etc/samba/smb.conf /etc/samba/smb.conf.orig
   ```
4. Create the configuration files.
   ```bash
   vim /etc/samba/smb.conf
   vim /etc/samba/smbshared.conf
   ```
5. Create the share directories and set ownership.
   ```bash
   mkdir -p /share/documents /share/public
   chown -R <myuser>:<users> /share
   ```
6. Validate the configuration files.
   ```bash
   testparm
   ```
7. Start the service and verify it is running.
   ```bash
   systemctl start smbd
   systemctl status smbd
   ```


### Configuration files

#### smb.conf

The `[global]` stanza applies settings to Samba as a whole:

```ini
[global]
server string = File Server             # description displayed to clients
workgroup = WORKGROUP                   # Windows workgroup name — a namespace for a collection of machines
security = user                         # authenticate users with local usernames and passwords
map to guest = Bad User                 # unauthenticated users get guest permissions
name resolve order = host bcast wins    # hostname resolution order — wins is deprecated but included for legacy networks
include = /etc/samba/smbshared.conf     # include this file as if it were part of smb.conf
```

#### smbshared.conf

Each stanza defines a share. The share name in brackets sets the Windows UNC path (`//<servername>/<ShareName>`):

```ini
[Documents]
path = /share/documents                 # share location on the host
force user = <myuser>                   # all users access content as <myuser>
force group = <users>                   # all users are treated as members of <users>
public = yes                            # share is publicly accessible
writable = no                           # share is read-only

[Public]
path = /share/public
force user = <myuser>
force group = <users>
create mask = 0664                      # permissions applied to new files
force create mode = 0664
directory mask = 0777                   # permissions applied to new directories
force directory mode = 0777
public = yes
writable = yes
```


## NFS

NFS works best in Linux and Unix environments. It grants permissions using standard
Unix methods. Some versions of Windows support NFS shares through the Services for NFS
plugin.

Each share in NFS is called an export. Store exports in a designated directory —
conventionally `/exports` — and declare them in `/etc/exports`, which NFS reads on
startup. An additional configuration file is available at `/etc/nfs.conf`. Clients
must mount shares manually after the server is configured.

### Install NFS

#### Server setup

1. Create the export directories.
   ```bash
   mkdir -p /exports/{backup,documents,public}
   ```
2. Install the NFS server package.
   ```bash
   apt install nfs-kernel-server
   ```
3. Back up the default configuration file before creating a new one.
   ```bash
   mv /etc/exports /etc/exports.orig
   ```
4. Create the exports configuration file. Run `man exports` to view all available options.
   ```bash
   vim /etc/exports
   ```
5. Edit the UID permission mapping configuration file.
   ```bash
   vim /etc/idmapd.conf
   ```
6. Restart the service and verify it is running.
   ```bash
   systemctl restart nfs-kernel-server
   systemctl status nfs-kernel-server
   ```

To reload exports without restarting the service:

```bash
exportfs -a
```

#### Client setup

1. Install the NFS client package.
   ```bash
   apt install nfs-common
   ```
2. Mount the share. Because `/exports` is declared as the export root in `/etc/exports`, you do not need to include it in the path:
   ```bash
   mount <servername-or-ip>:/<share-name> /mnt/<mount-point>
   mount 10.20.30.40:/documents /mnt/documents
   ```



### Configuration files

#### /etc/exports

Each line declares a share, its allowed clients, and its options. Use a subnet in CIDR
or netmask notation to restrict access. Common options:

| Option | Description |
|:---|:---|
| `ro` | Read-only access |
| `rw` | Read/write access |
| `fsid=0` | Declares this as the NFS root export |
| `no_subtree_check` | Improves reliability; NFS may log warnings if omitted |
| `no_root_squash` | Allows the root user on a client to act as root in the export |

```bash
/exports *(ro,fsid=0,no_subtree_check)
/exports/backup 192.168.1.0/255.255.255.0(rw,no_subtree_check)
/exports/documents 192.168.1.0/255.255.255.0(ro,no_subtree_check)
/exports/public 192.168.1.0/255.255.255.0(rw,no_subtree_check)
```

#### /etc/idmapd.conf

This file maps user and group IDs between the server and clients. Set `Domain` to your
network domain if it differs from the fully qualified domain name minus the hostname:

```ini
[General]
Verbosity = 0
Domain = mynetworksettings.com

[Mapping]
Nobody-User = nobody
Nobody-Group = nogroup
```

## Transferring files

### rsync

`rsync` copies files between locations efficiently, transferring only what has changed
since the last run. Use it for recursive copies, incremental backups, and server-to-server
transfers over SSH.

The `-a` (archive) flag is a shorthand equivalent to `-rlptgoD`:

| Flag | Effect |
|:---|:---|
| `r` | Recursive |
| `l` | Preserve symbolic links |
| `p` | Preserve permissions |
| `t` | Preserve modification times |
| `g` | Preserve group ownership |
| `o` | Preserve owner |
| `D` | Preserve device files |

When copying over SSH, a trailing slash on the source path copies only the directory
contents. Without the trailing slash, rsync copies the directory itself and its contents:

- `/home/me/only-contents/` — copies contents only
- `/home/me/dir-and-contents` — copies the directory and its contents



Use these commands for common copy and sync operations:

```bash
rsync -r /home/<user> /backup           # recursive copy, no permissions preserved
rsync -a /home/<user> /backup           # recursive copy, preserve permissions and metadata
rsync -av /home/<user> /backup          # same as above, verbose output
rsync -av /home/<user>/rsync-test/ normaluser@<ip-addr>:/backup    # copy over SSH
```

The `--delete` flag removes files from the target that no longer exist in the source. Use it with care:

```bash
rsync -av --delete /src /target         # sync directories, delete files in target not in source
```

To back up removed files instead of deleting them:

```bash
rsync -avb --delete /src /target        # move removed files to a backup location
rsync -avb --delete --backup-dir=/backup/incremental src/ target/  # move removed files to a specific backup directory

CURDATE=$(date +%m-%d-%Y)
rsync -avb --delete --backup-dir=/backup/incremental/$CURDATE src/ target/  # use a dated backup directory
```

### scp

`scp` (secure copy) ships with the OpenSSH client. Use it for one-off transfers of
single files or small groups of files, both to and from remote systems. The default
remote destination is `/home/<user>`, so you can use paths relative to that directory.

Common flags:

| Flag | Effect |
|:---|:---|
| `-C` | Compress the file during transfer |
| `-p` | Preserve file access and modification times and permissions |
| `-r` | Recursive copy |
| `-v` | Verbose output |
| `-P` | Specify a non-default port |

```bash
scp file1 normaluser@<ip-addr>:/home/normaluser/scp-test    # copy a file to a remote directory
scp file1 normaluser@<ip-addr>:                             # copy a file to the remote home directory
scp <user>@<ip-addr>:/home/<user>/scp-test/file2 .          # copy a file from remote to local

scp -r /home/<user>/scp-test/cp-dir <user2>@<ip-addr>:/home/<user2>/scp-test           # recursive copy
scp -P 2222 -r /home/username/scp-test/cp-dir <user2>@<ip-addr>:/home/<user2>/scp-test # specify port
scp -rv /home/username/scp-test/cp-dir normaluser@$REMOTE:/scp-test                    # verbose recursive copy
```

### sftp

`sftp` (SSH File Transfer Protocol) provides an interactive session for transferring
files over SSH. Use it for larger files or archives where you want to browse the remote
filesystem, create directories, and verify transfers in place.

Log in the same way as SSH. Standard commands work on the remote system. Prepend `l`
to run a command on the local system instead, for example `lcd` or `lls`.

Common commands:

| Command | Description |
|:---|:---|
| `bye` / `exit` | Exit the sftp session |
| `get` | Download a file from remote to local |
| `reget` | Resume an interrupted download |
| `put` | Upload a file from local to remote |
| `reput` | Resume an interrupted upload |
| `ls` | List files in the remote working directory |
| `lls` | List files in the local working directory |
| `mkdir` | Create a directory on the remote system |
| `lmkdir` | Create a directory on the local system |
| `progress` | Toggle transfer progress display (on by default) |

#### Example session

Connect, browse, upload a file, and verify:

```bash
sftp username@localhost
username@localhosts password:
Connected to localhost.

sftp> ls
Desktop  Development  Documents  Downloads ...

sftp> lls
Extract  Project42.txt  Project43.txt  Project44.txt  TarStorage ...

sftp> lmkdir sftp_files

sftp> lls
Extract  Project42.txt  sftp_files ...

sftp> put Project4x.tar
Uploading Project4x.tar to /home/username/linux-playground/archives/sftp_files/Project4x.tar
Project4x.tar                                100%   10KB   7.5MB/s   00:00

sftp> ls
Project4x.tar

sftp> bye
```