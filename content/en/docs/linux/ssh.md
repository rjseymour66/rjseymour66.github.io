---
title: "SSH"
linkTitle: ""
# weight: 1000
# description:
---

## Config files

Config files are in `/etc/ssh`:

```bash
$ ls -l /etc/ssh/
total 652
-rw-r--r-- 1 root root 620042 Aug  9 02:33 moduli
-rw-r--r-- 1 root root   1649 Aug  9 02:33 ssh_config
drwxr-xr-x 2 root root   4096 Aug  9 02:33 ssh_config.d
-rw-r--r-- 1 root root   3253 Nov  7 04:09 sshd_config
drwxr-xr-x 2 root root   4096 Nov  2 17:18 sshd_config.d
-rw------- 1 root root    505 Nov  2 17:18 ssh_host_ecdsa_key
-rw-r--r-- 1 root root    176 Nov  2 17:18 ssh_host_ecdsa_key.pub
-rw------- 1 root root    411 Nov  2 17:18 ssh_host_ed25519_key
-rw-r--r-- 1 root root     96 Nov  2 17:18 ssh_host_ed25519_key.pub
-rw------- 1 root root   2602 Nov  2 17:18 ssh_host_rsa_key
-rw-r--r-- 1 root root    568 Nov  2 17:18 ssh_host_rsa_key.pub
-rw-r--r-- 1 root root    342 Dec  7  2020 ssh_import_id
```

## Generate new keypair

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

## Copy keys to remote

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

## Login with hostname

Instead of using the IP address, add the hostname and IP address mapping to `/etc/hosts`:

```bash
# open file (sudo)
vim /etc/hosts

# add new-host mapping
127.0.0.1   localhost
127.0.1.1   ubuntu-24
10.20.30.40 new-host
```