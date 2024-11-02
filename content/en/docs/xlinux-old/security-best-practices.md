---
title: "Security best practices"
weight: 180
---

## User security

### Authenication methods

User ID and password authentication is not enough to secure a system, so Linux provides a number of additional solutions.

#### Kerberos

- Single sign-on (SSO), so you sign into the network only once and can view all the resources on it.
- Centralizes auth process, but each server must maintain a db of objects on the server that the user acct has access to.
- Three pieces to Kerberos auth:
  - Authentication server (AS): Users log into the AS to initiate the auth process
  - Key distribution center (KDC): AS sends login req to KDC, which issues the user a ticket-granting ticket (TGT) and keeps it on the server.TGT is encrypted, has timestamp and time limit for how long it is valid.
  - Ticket-granting service (TGS): After user has ticket, can log into any machine on the network.
    - During login, the server connects with TGS to validate the users ticket.
    - If valid, ticket is stored in cache with `kinit` utility
    - Use `klist` to view tickets in server cache

#### LDAP

Lightweight Directory Access Protocol
- Uses hierarchical tree db structure to store info about network users and resources
- Admins store privileges in a central LDAP auth db, and network resources check with the db when users authenticate to a resource
- LDAP is distributed, so you can store part of the db tree across servers on the network.

#### RADIUS

Remote Authentication Dial-In User Service
- Old technology, originally to provide central auth services for dial-up bulletin board servers
- Simple, so still used for network access. For example, IEEE 802.1x auth protection on switches
- Auth server authenticates user acct and other info like network addr, phone number, and access privileges

#### TACAS+

Terminal Access Controller Access-Control System
- Family of protocls that provide remot auth in a server env
- Early Unix
- Users have to log into each network server individually, even though auth info is stored in centralized server

#### Multifactor authentication

Requires user to have two pieces of info to log into a system: something they know (password) and something they possess:
- Biometrics: Phyical feature, like fingerprints
- Tokens: Hardware (USB drives) or software (files on a network device)
- Public key infrastructure (PKI): public and private key, shares public key with the server
- One-time password: Login with user ID and passwd, then sent email or text with additional password

### Unique user accounts

_Nonrepudiation_ is the goal of monitoring user accounts. It means that every action a user takes can be tracked back to that exact user.

### Strong passwords

- `/etc/login.defs` contains settings that apply to the length and age of a password
- PAM modules can control its complexity
  - `pwquality.so` library defines password rules that apply to system user accounts.

```bash
cat /etc/pam.d/common-password 
#
# /etc/pam.d/common-password - password-related modules common to all services
#
# This file is included from other service-specific PAM config files,
# and should contain a list of modules that define the services to be
# used to change user passwords.  The default is pam_unix.
...
# here are the per-package modules (the "Primary" block)
password	requisite			pam_pwquality.so retry=3    # file and quality setting line
...
```


PAM password directives:

| Directive | Description |
|-----------|-------------|
| `difok` | number of char changes from old passwd to new |
| `enforce_for_root` | if passwd rules apply to root |
| `maxrepeat` | max number of repeat chars |
| `minlen` | minimum length |
| `reject_username` | reject if passwd contains username, in forward or reverse |
| `retry` | passwd attempts allowed |

PAM password credits (requirements):

| Directive | Description |
|-----------|-------------|
| dcredit | num of numeric chars |
| lcredit | num of lowercase chars |
| ocredit | num of special chars |
| ucredit | num of uppercase chars |

Specify credits in negative numbers:

```bash
password	requisite			pam_pwquality.so retry=3 dcredit=-1 lcredit=-2 ucredit=-1
```

### Restrict root

#### Block root access

`su` and `sudo` is helpful because you can track who is using root privileges to perform root actions. To block root access, change the default shell to `/usr/sbin/nologin`:
```
cat /etc/passwd | grep root
root:x:0:0:root:/root:/usr/sbin/nologin
```

#### Block root access from specific devices

To prevent logins from a console physically attached to the system, create a `/etc/securetty` file on the system
- If this is blank, the root user account cannot log in from any physical console
- You can still login on the network

#### Block root access from SSH

Modify OpenSSH configuration file:

```bash
cat /etc/ssh/sshd_config

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.
...
# Authentication:

#LoginGraceTime 2m
#PermitRootLogin prohibit-password # remove leading '#' and change this line to yes or not
...

```


## System security


### Separation of data

- By default, Linux creates a single partition for the root of the virtual directory.
- This might be an issue if users overuse storage. The system halts if there is no available disk space.
- In multiuser enviornment, you should separate system and user storage:
  - Create two partitions on disk, and assign one to root (`/`) and the other to home (`username/home`)

### Data encryption

Encrypting files is tedious because you have to encrypt and decrypt each time you want to read the file data. You can encrypt an entire partition:
- Encrypt every file on the partition at the kernel level
- Automatically encrypts and decrypts as you read or write to the disk
- _Linux Unified Key Setup_ (LUKS) is the app that lets you interact with an encrypted partition, and uses these two components:
  - `dm-crypt`: plugs into kernel and provides interface between virtual mapped drive and actual physical disk
  - `cryptmount`: creates virtual mapped drive and interfaces it w the physical drive to ensure all data passed to virtual drive is encrypted before it is written to disk


### chroot jail to restrict apps

In a multiuser environment, there might be collisions between applications. You can prevent this with a _chroot jail_.
- `chroot` runs a command in a new root directory structure within the virtual filesystem
- All disk access is restricted to this new root dir
- Apps in chroot jail are unaware of the real directory structure, so you must copy any utilities or libraries into the new root dir

```bash
# starting-dir: location to start new dir
# command: command to run within new dir structure
chroot <starting-dir> <command>
```

### Unauthorized reboots

If your Linux server is publically accessible, you need to make sure unauthorized users can't reboot the server and take control.

#### BIOS/UEFI

Add password to either BIOS or UEFI.

#### GRUB bootloader

Add password on the GRUB bootloader. GRUB files are plaintext, so encrypt it before you store it in a config file:

```bash
# Debian systems
# 1. Generate the password
grub-mkpasswd-pbkdf2

# 2. Add the following to /etc/grub.d/40_custom
# userid: acct you use to log into GRUB
# password: value provided in step 1
set superuser "<userid>"
password_pbkdf2 <userid> <password>

# RHEL systems
# 1. Generate the password
grub-md5-crypt

# 2. Add the following to /etc/grub.d/40_custom
password -md5 <password>
```

#### Disable CTRL + ALT + DEL

Disable on `systemd` distros:

```bash
systemctl mask ctrl-alt-del.target
```

### Unapproved jobs

Users can schedule jobs with `at` and `cron`, which use the following allow and deny lists:
- `/etc/at.allow`
- `/etc/at.deny`
- `/etc/cron.allow`
- `/etc/cron.allow`

Workflow:
- If user is not found in `*.allow` file, system checks `*.deny` file.
- If user is not found in `*.deny` file, system lets the user create job.

### Banners and messages

Present canned information to users after they log in to system, usually located in these files:
- `/etc/login.warn`: Displayed before login prompt at console. Usually for legal disclaimers and warnings to potential attackers.
- `/etc/motd`: Message of the Day. Displays info like hardware failures or upcoming scheduled maintenance.


### USB devices

Use `modprobe`:
- When a device is inserted, the kernel looks for a module to support the device.
- If there isn't one, it calls `modprobe` to load the kernel module that can support the device.
  1. Edit `/etc/modprobe.d/blacklist.conf` config files to block the modules required to interface with USB devices
  2. Then, save and reboot


```bash
# view file
cat /etc/modprobe.d/blacklist.conf 
# This file lists those modules which we don't want to be loaded by
# alias expansion, usually so some other driver will be loaded for the
# device instead.

# evbug is a debug tool that should be loaded explicitly
blacklist evbug
...

# block USB modules
blacklist uas
blacklist usb:storage
```

### Auditing

The `auditd` package provides logging features not available with just `rsyslog`. Define rules with:
- `/etc/audit/audit.rules` file for persistent rules
- `auditctl` utility only valid until there is a reboot

## Network security

### Deny hosts

- `/etc/hosts.deny`: Create a list of hosts that you want to deny access to your system 
  - TCP Wrappers program reads this file and blocks the hosts
  - accepts host name or IP address
- `/etc/hosts.allow`: Create list of hosts that can access your system
  - More extreme than `*.deny` list
- If both `*.deny` and `*.allow` lists are empty, then system lets every host access

### Unused services

You might not use some of the legacy Linux services that are still included in distros. Examples include the following:
- FTP, file transfer protocol.
  Ports 21 and 22
- Telnet, uses plaintext.
  Port 23
- Finger, remote lookup services to find users.
  Port 79.
- Mail services, good practice to uninstall if the system does not send or receive email. Common apps are sendmail and Postfix.
  Port 25.

### Default ports

A port is a unique number assigned to an application so that when a remote client communicates with the server, the server knows which application to send the connection to.
- Might want to move apps that use well-known ports to private ports, but hackers use port scanners.

Most popular network application ports are listed in `/etc/services`. You can change the port here and restart the application.

_Well-known ports_ (0 - 1023)
: Formerly assigned by the Internet Assigned Numbers Authority (IANA).

_Registered ports_ (1024 - 49151)
: Registered with IANA but not formerly assigned

_Private ports_ (> 49151)
: Used by anny application