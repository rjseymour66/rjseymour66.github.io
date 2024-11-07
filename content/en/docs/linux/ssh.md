---
title: "SSH"
linkTitle: ""
# weight: 1000
# description:
---

## ssh

Config files are in `/etc/ssh`.

### Generate new keypair

```bash
# ed25519 algorigthm by default (no universal support)
$ ssh-keygen
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/linuxuser/.ssh/id_ed25519):                            
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/linuxuser/.ssh/id_ed25519
...

# rsa with 4KB key size
$ ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/home/linuxuser/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
...
```

### Copy keys to remote

```bash 
# copy to .ssh/autorized_keys (-i specifies the key)
$ ssh-copy-id -i .ssh/id_rsa.pub linuxuser@192.168.56.50
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: ".ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
linuxuser@192.168.56.50\'s password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh \'linuxuser@192.168.56.50\'"
and check to make sure that only the key(s) you wanted were added.

# manual copy with pipes
$ cat .ssh/id_rsa.pub \
> | ssh linuxuser@192.168.56.50 \
> "cat >> .ssh/authorized_keys"
linuxuser@192.168.56.50's password: 
```

### dpkg

```bash
# get installed version and update status
$ dpkg -s openssh-server
Package: openssh-server
Status: install ok installed
Priority: optional
...
```

## systemctl

```bash
# check status of system daemon
$ dpkg -s openssh-server
Package: openssh-server
Status: install ok installed
Priority: optional
...

# stop system daemon
systemctl stop <daemon>

# autoload daemon on system startup
systemctl enable <daemon>
```