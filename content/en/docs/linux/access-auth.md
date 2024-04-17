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

- certificate authority (CA) structure
  - CA verifies a person's ID, then issues a digital cert
  - cert provides identification proof and an embedded key
  - Cert holder uses cert's key to encrypt data and sign it using the certificate
- Commercial CAs charge money
- Can use self-signed cert

### Key concepts

Two types of cipher keys:
- Private keys. Symmetric keys: also called private or secret keys. Encrypts data with a crypto algorithm and a single key. Protected by a passphrase.
  - Bc there is only one key, you need to share the key if others are going to decrypt the data
- Public/private key pairs. Assymetric keys: Used by SSH. Encrypts data with a crypto algorithm and a two keys. Public key is used to encrypt, private key is used to decrypt. Public key is meant to be shared.

#### Hashing

One-way algorithm that turns plaintext into fixed-length ciphertext. The ciphertext is called a _message digest_, hash, hash value, fingerprint, or signature.
- Hashing algorithm needs to be collision free - it cannot create the same output for two different inputs.
- _non-salted_ and _non-keyed message digests_ only use plaintext file as input.
- _salt_ is random data included with input file. Creates a _salted hash_.
- _keyed message digest_ is created with the plaintext file and private key. Used in SSH.

#### Digital signatures

A _digital signature_ is a message digest of the original plaintext data, which is then encrypted with a user's private key and sent along with the ciphertext. It provides authentication and data verification. 
- Ciphertext receiver decrypts the digital signature with the sender's public key to get the original message digest
- receiver also decrypts the ciphertext and hashes its plaintext data to create a new message digest
- original and new message digests are compared.

## SSH

- Secure Shell (SSH), de facto standard software to send data securely across the network.
- Uses assymetric (public/private) key pairs.
- Keeps track of previously connected hosts in `~/.ssh/known_hosts`

| Distro | OpenSSH package names |
|--------|-----------------------|
| Rocky/RHEL | `openssh`, `openssh-clients`, `openssh-server` |
| Ubuntu | `openssh-client`, `openssh-server` |

```bash
ssh [options] username@hostname

# send commands to remote system
ssh username@hostname "ls -lsogh ~/home/dirname"
```

### rsync

`rsync` uses SSH to quickly copy files to a remote system in an encrypted tunnel:

```bash
rsync <filename> username@hostname:~/path/to/destination
```

### Configuring SSH