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


## Network security