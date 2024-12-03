---
title: "SSH"
linkTitle: ""
# weight: 1000
# description:
---

## User config files

Create a `~/.ssh/config` file that assigns a logical name to the `ssh`, `scp`, `sftp`, and `rsync` connection details for a machine:

```bash
# config file format
Host <hostname>             # logical name you assign
    HostName <ip-addr>      # IP addr or remote hostname (from /etc/hosts)
    User <username>         # user you ssh as
    Port <port>             # SSH port

# For example:
Host u24
	HostName 10.20.30.40
	User linux
	Port 2222

# get all config options
man ssh_config
```

Then, log in like this:

```bash
ssh u24
```
If you have multiple host definitions (stanzas), then it applies all rules that it matches. See [Linuxize](https://linuxize.com/post/using-the-ssh-config-file/).

## Daemon config files

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

## Change default port number

SSH uses port 22, but you can change that in `/etc/ssh/sshd_config`:

```bash
cat /etc/ssh/sshd_config

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.
...

#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

systemclt restart ssh
ssh -p<new-port> <username>@<ip-or-hostname>
```

## Security

Make these changes to `/etc/ssh/sshd_config`:

```bash
# restart ssh after changes to sshd_config
systemctl restart ssh

# changes to sshd_config
cat /etc/ssh/sshd_config

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.
...
#LoginGraceTime 2m
PermitRootLogin no          # disable root login (ex: ssh root@ip)
...

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication yes  # change to 'no' to require auth with keypairs, not passwords
...

```