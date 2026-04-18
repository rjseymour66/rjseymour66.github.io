+++
title = 'Securing the server'
date = '2025-09-07T19:05:34-04:00'
weight = 10
draft = false
+++


Every application accessible from outside your network increases your **attack surface**:
the set of entry points an attacker can exploit. Reduce the attack surface by limiting
what is exposed:

- Uninstall unused applications.
- Configure firewalls to allow only required connections.
- Use strong, randomly generated passwords.
- Allow external access only to services that require it. Lock down all internal
  applications completely.

## Check ports

Regularly audit which ports are listening for connections. Processes listening on
`0.0.0.0` accept connections from any network. Processes listening on `127.0.0.1`
accept local connections only. Stop and remove any service you do not need:

```bash
sudo ss -tulpn              # list listening ports (use sudo for full process details)
```


## Check packages

Audit installed packages and remove anything your server does not need. Research each
package before removing it, as some are system dependencies. Once you have identified
packages to remove across your servers, create an Ansible playbook to enforce their
absence consistently:

```bash
dpkg --get-selections > installed_packages.txt      # export a list of all installed packages
apt-cache rdepends <package>                        # list packages that depend on <package>
```

## Principle of least privilege

Grant users only the permissions they need to do their job. Document all permission
changes so you can apply them consistently across servers:

- Add users to the minimum number of groups required.
- Set network shares to read-only by default.
- Audit servers for user accounts that have not logged in recently.
- Set account expiration dates.
- Limit access to system directories.
- Restrict sudo to specific commands.

## Common Vulnerabilities and Exposures (CVEs)

When researchers discover a security flaw, they report it and assign it a CVE number.
Many distributions maintain their own CVE catalogs at [ubuntu.com/security/cves](https://ubuntu.com/security/cves),
listing which versions are vulnerable, which CVEs have been addressed, and which updates
to install. You may need to restart the server after applying security updates.


## Installing security updates

Package updates are released frequently and include both new features and security fixes.
On LTS releases, security updates are especially common. You may need to restart services
after applying updates.

Some administrators delay updates because they risk breaking production systems. To
mitigate this:

- Test updates on virtual clones of your production system before applying them.
- For clusters, update one server first and verify stability before proceeding.
- For workstations, roll out updates to a subset of users first.

Previously downloaded package versions are cached in `/var/cache/apt/archives` and can
be used to roll back an update.

### Apply updates

Run these steps on a regular schedule to keep packages and security fixes current:

1. Update the local package index.
   ```bash
   sudo apt update
   ```
2. Upgrade installed packages.
   ```bash
   sudo apt upgrade
   ```
3. Upgrade packages and dependencies, including kernel updates.
   ```bash
   sudo apt dist-upgrade
   ```
4. Fix any broken package dependencies.
   ```bash
   sudo apt -f install
   ```
5. Restart affected services as needed.
   ```bash
   sudo systemctl restart <service>
   ```

### Roll back updates

If an update breaks a service or causes unexpected behavior, use the cached package
version to restore the previous state. Previously downloaded packages are stored in
`/var/cache/apt/archives`:

```bash
sudo dpkg -i /var/cache/apt/archives/<package-name>
```

## Kernel upgrades

Distributions handle kernel upgrades differently. Some, like Arch Linux, support only
one installed kernel at a time and require a reboot to switch versions. Ubuntu supports
multiple installed kernels simultaneously. GRUB boots the newest version by default. If
a new kernel causes issues, press Esc during boot and select a known-working version.

### Canonical Livepatch

[Canonical Livepatch](https://auth.livepatch.canonical.com/) applies kernel updates
without requiring a reboot. It requires a subscription, though it is free for up to
three servers. Create an account to receive a token, then activate it on each server.
Complex security updates may still require a reboot.

```bash
sudo snap install canonical-livepatch           # install the Livepatch snap
sudo canonical-livepatch enable <token>         # activate your token
sudo canonical-livepatch status                 # check Livepatch status
```

## OpenSSH

The SSH daemon reads its configuration from `/etc/ssh/sshd_config`. Restart the service
after every change. Connection attempts are logged to `/var/log/auth.log`.

```bash
systemctl restart ssh
```

Apply these settings to harden SSH access:

| Setting | Value | Effect |
|:---|:---|:---|
| `Port` | `65332` (or any non-default) | Reduces noise in logs from automated port scanners |
| `AllowUsers` | `user1 user2` | Restricts SSH access to named users |
| `AllowGroups` | `sshusers` | Restricts SSH access to members of a group |
| `PermitRootLogin` | `no` | Disables root login entirely |
| `PasswordAuthentication` | `no` | Requires public key authentication |

`AllowGroups` is easier to manage than `AllowUsers`: add a user to the group rather
than editing the config file. If you change the port, specify it when connecting with
`ssh -p <port>`. If you restrict access by group, you may need to re-copy SSH keys
with `ssh-copy-id`.

#### /etc/ssh/sshd_config

```bash
Port 65332
AllowUsers user1 user2
AllowGroups admin sshusers
PermitRootLogin no
PasswordAuthentication no
```

## Fail2ban

Fail2ban monitors log files for repeated authentication failures and blocks offending
IP addresses after a configurable number of attempts. It starts at boot by default.

Do not edit `/etc/fail2ban/jail.conf` directly — it is overwritten on updates. Instead,
copy it to `/etc/fail2ban/jail.local`. The `.local` file takes precedence over the
default `.conf` file. Restart Fail2ban after every configuration change.

A **jail** monitors a specific log file and bans IPs that exceed the failure threshold.
All jails are disabled by default except SSH. Enable additional jails based on the
services you run:

- `[apache-modsecurity]`: enable if you use SSL with Apache
- `[apache-shellshock]`: enable to protect against the Shellshock vulnerability

### Install Fail2ban

Install and copy the default configuration to a local override file:

```bash
apt install fail2ban
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Check service health and jail status:

```bash
systemctl restart fail2ban
systemctl status -l fail2ban
fail2ban-client status              # list all enabled jails
```

#### /etc/fail2ban/jail.local

```bash
ignoreip = 127.0.0.1/8 ::1 20.30.40.0/24   # never ban these hosts (::1 is IPv6 localhost)
bantime  = 600                               # seconds to ban an IP
maxretry = 5                                 # failures before banning

[sshd]
enabled = true
port    = 65332                              # update if you changed the default SSH port
```

## Database servers

Database servers should not be reachable from the internet. Accept connections only from
internal servers that require access — for example, a web server connecting to its
database. Use `/etc/hosts.allow` and `/etc/hosts.deny` to control access. Configure
`/etc/hosts.allow` before `/etc/hosts.deny`, as the allow file takes precedence.

#### /etc/hosts.allow

```bash
ALL: 192.168.1.30               # allow this specific IP
ALL: 192.168.1.0/255.255.255.0  # allow any IP on the 192.168.1 network
ALL: 192.168.1.                 # wildcard: any IP on this network
ssh: 192.168.1.                 # allow SSH from any host on this network
```

#### /etc/hosts.deny

```bash
ALL: ALL                        # deny all connections not explicitly allowed
```

### User access

Restrict database user accounts to specific IP addresses. The `%` wildcard allows
connections from any host and should not be used for application accounts:

```bash
# restrict access to a single IP
GRANT SELECT ON mydb.* TO 'appuser'@'192.168.1.50' IDENTIFIED BY 'password';
FLUSH PRIVILEGES;

# restrict access to a subnet
GRANT SELECT ON mydb.* TO 'appuser'@'192.168.1.%' IDENTIFIED BY 'password';
FLUSH PRIVILEGES;
```

## LUKS

Encrypt disks that store personal or sensitive data. For the strongest protection,
enable full-disk encryption during your Linux installation. Without encryption, anyone
who boots a live OS can mount the drive and read its contents. Use the `cryptsetup`
package to encrypt and decrypt disks. Encrypting a disk erases all existing data, so
use a clean disk or back up the data first.

### Encrypt and mount a disk

Encrypt a new disk and mount it for use. This process erases all existing data on the
disk, so back up any data before proceeding:

1. Install the package.
   ```bash
   apt install cryptsetup
   ```
2. Identify the disk to format.
   ```bash
   fdisk -l
   ```
3. Format the disk with LUKS encryption.
   ```bash
   cryptsetup luksFormat /dev/<disk>
   ```
4. Open the encrypted disk and assign it a name.
   ```bash
   cryptsetup luksOpen /dev/sda <disk-name>
   ```
5. Verify the mapped device appeared in `/dev/mapper/`.
   ```bash
   fdisk -l
   ```
6. Create a filesystem on the mapped device.
   ```bash
   mkfs.ext4 -L "usb_drive" /dev/mapper/<disk-name>
   ```
7. Create a mount point and mount the device.
   ```bash
   mkdir /media/<disk-name>
   mount /dev/mapper/<disk-name> /media/<disk-name>
   ```

### Unmount an encrypted disk

Unmount the disk and close the LUKS container before physically removing the device:

```bash
umount /media/<disk-name>
cryptsetup luksClose /dev/mapper/<disk-name>
```

### Remount an encrypted disk

Reopen and remount a previously encrypted disk after it has been removed or the system
has been rebooted:

```bash
fdisk -l
cryptsetup luksOpen /dev/sda <disk-name>
fdisk -l
mount /dev/mapper/<disk-name> /media/<disk-name>
```

### Change the passphrase

Update the LUKS passphrase on an existing encrypted disk:

```bash
cryptsetup luksChangeKey /dev/sda -s 0
```