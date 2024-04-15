---
title: "Access and authorization methods"
weight: 150
---

## PAM

Pluggable authentication modules.

PAM provides libraries that compile into the application and acts as the interface for apps that require an authentication method. This includes the following:
- Passwords: `/etc/passwd` and `/etc/shadow`
- Certificate: PKI cert stores
- Lightweight Directory Access Protocol (LDAP)
- Kerberos: Kerberos issues a token to clients when they login to the system. These tokens can authenticate to multiple network resources. Also called single sign-on (SSO) bc you just login once to access multiple networked resources.
- Multifactor auth (MFA): Includes biometrics, tokens, PKI, and password emails

Programs that use PAM services are compiled with `libpam.so` and have a PAM config file. You can determine if a program is PAM-aware with `ldd`:

```bash
# ldd prints shared object files
ldd /bin/login | grep libpam.so
	libpam.so.0 => /lib/x86_64-linux-gnu/libpam.so.0 (0x000071b01ce9a000)


# view all PAM config files
ls -l /etc/pam.d/
-rw-r--r-- 1 root root  384 Nov 11  2021 chfn
-rw-r--r-- 1 root root   92 Nov 11  2021 chpasswd
...
```
[Configuration file format](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system-level_authentication_guide/pam_configuration_files#PAM_Configuration_File_Format)

- Each PAM-MODULE is called in the order it is listed in the config file. This is called the _module stack_.

```bash
# config file syntax
type control-flag pam-module [module-options]

cat /etc/pam.d/login 
#
# The PAM configuration file for the Shadow `login' service
#
...
auth       optional   pam_faildelay.so  delay=3000000
...
auth       requisite  pam_nologin.so
...
session [success=ok ignore=ignore module_unknown=ignore default=bad] pam_selinux.so close
...
session    required     pam_loginuid.so
...
session    optional   pam_motd.so motd=/run/motd.dynamic
session    optional   pam_motd.so noupdate
...
```

### Enforcing strong passwords

## PKI

Public key infrastructure

encryption
: When _plaintext_--human-readable text--is converted into _ciphertext_ with cryptographic algorithms.

decryption
: When ciphertext is converted into plaintext.

keys/cipher keys
: Special data that encrypts and decrypts data. If you share encrypted data, you must also share a key to decipher it.

### Certificates