---
title: "Securing your server"
linkTitle: "Security"
weight: 190
# description:
---

All applications that are accessible from outside your network is a risk:
- Attack surface is the list of things that are exploitable
- Lowering attack surface is when you lock down an application against threats
- You have to allow some access to servers (e.g. web services), but you should completely lock down internal apps 
- uninstall unused apps 
- setup firewalls to allow only specific connections
- use strong, randomly-generated passwords

## Check ports

Check which ports are listening for network connections:
- processes listening on 0.0.0.0 accept connections from any network 
- processes listening on 127.0.0.1 are not accepting connections
- stop unwanted services or delete the package. Can always install at a later time 
- 

```bash
sudo ss -tulpn              # check what ports are listening - more info with 'sudo'    
```


## Check packages

Get a list of all installed packages and remove unneeded ones:
- research before you remove a package - might be a system package!
- when you come up w a list of packages that your servers don't need, create an ansible playbook to make sure they are not installed 

```bash 
dpkg --get-selections > installed_packages.txt          # creates a file containing all installed packages
apt-cache rdepends <package>                            # check whether other packages depend on <package>
                                                        # packages in output need <package> to run
```

## Principle of least privilege (script some of these)

Do not trust users - only give them privs that they need to do their job. Document these changes so you can apply to all servers:
- Add users to smallest num of groups
- Network shares should default to read-only 
- audit servers for user accts that haven't logged in for a long time 
- set acct expirations
- limit access to system directories
- restrict sudo to specific commands 

## Common Vulnerabilities and Exposures (CVEs)

https://ubuntu.com/security/cves

When a security flaw is revealed, it is reported on security sites and given a CVE number so researchers can document their findings:
- CVEs are found in online catalogs - many distros maintain their own CVE catalogs
  - which distro version is vulnerable 
  - which CVEs have responded to 
  - updates to install to address them 
- Might have to restart server after applying updates 


## Installing security updates 

Package updates are sometimes made daily:
- include new features and security updates 
- For LTS releases, security updates are made frequently
- Some admins don't install updates regularly bc they might make major changes to the server and break something
  - create virtual clones of prod system and test updates there 
  - for clusters, maybe just update one server and then go from there 
  - for workstations, select some users to receieve updates and then push to other users
- Don't close terminal window when upgrade is in progress
- You can run `apt upgrade` followed by `apt dist-upgrade` to upgrade new packages or kernel updates
- Might have to restart services
- In case of rollback, previously downloaded package versions are available in `/var/cache/apt/archives`

```bash
# --- standard package updates --- #
sudo apt update                     # 1. update local repo index - checks all subscribed repos for changes
sudo apt upgrade                    # 2. update packages
sudo apt dist-upgrade               #    upgrade - safest to use. does not remove packages, just updates installed packages
                                    #    dist-upgrade - updates everything it can: packages + dependencies, updated kernel
sudo apt -f install                 # 3. fix installed packages, if possible
sudo systemctl restart <service>    # 4. restart services, as needed

# --- rollback a package version --- #
sudo dpkg -i /var/cache/apt/archives/<package-name>
```

## Kernel upgrades 

Distros handle kernel upgrades differently:
- Some distros can have only one kernel isntalled at a time (arch linux) and require reboots
- Ubuntu can have multiple kernels installed simultaneously, so it runs the new version alongside your older version 
  - GRUB boots with the new version
  - If there is an issue, press Esc during boot and select a known-working kernel 

### Canonical Livepatch

Lets you get kernel updates and apply them without rebooting:
- Not free or included - requires a subscription at https://auth.livepatch.canonical.com/
  - Create account to receive a token that you ahve to install
  - Free on up to 3 servers 
- Complex security updates might still require a reboot 

```bash 
sudo snap install canonical-livepatch           # install livepatch snap
sudo canonical-livepatch enable <token>         # apply your token
sudo canonical-livepatch status                 # check status
```

## OpenSSH

Configuration file is in `/etc/ssh/sshd_config`:
- always restart daemon after you change the settings
- connection attemps are logged in `/var/log/auth.log`

Security changes:
- Change port from 22 to X
  - Helps keep logs clean - you only see port scan intrusion attempts
  - Have to specify port when ssh-ing into the machine
- For older installations, make sure you are using Protocol 2.
  - If its not in the file, then you are using protocol 2
- Set `AllowUsers` to restrict who can ssh into the server
  - This setting is not found in the config file by default
- Set `AllowGroups` to restrict who can ssh into the server by group
  - Create group for ssh, like `sshusers`
- `AllowUsers` overrides `AllowGroups`, but `AllowGroups` is easier to manage because you just add the user to the group and no need to change the config file
  - Might have to recopy ssh keys to server with `ssh-copy-id -i ...`
- Set `PermitRootLogin` to `no`. This lets a root user login with ssh. The default `prohibit-password` means that root can use a key to ssh in, but not use password. This is bad too.
  - If a cloud provider requires you login as root, then give regular user sudo privs and log in w that user
- Set `PasswordAuthentication` to `no`
  - Prevents brute-force attacks
  - Must set up public key access 


```bash
systemctl restart ssh

# --- /etc/ssh/sshd_config --- #
Port 65332                          # change default port
AllowUsers user1 user2[ user3...]   # restrict ssh access by user
AllowGroups admin sshusers          # restrict ssh access by group (sudo groupadd sshusers)
PermitRootLogin no                  # disable passwd and key login for root user
PasswordAuthentication              # disable passwd auth, require public key auth
```

## Fail2ban

Watches log files for authentication failures and can block an IP address after a set number of failures:
- starts at boot
- config file is `/etc/fail2ban/jail.conf` but that is overwritten if there are updates to fail2ban. Create a `/etc/fail2ban/jail.local` file - the `.local` file superseded the default `.conf` file.
- All Jails are disabled by default, except SSH. Add `enable = true`
  - Add `enabled` line to [sshd] section too, just to be sure
  - A jail is a security guard that watches a specific log file and blocks bad actors.
  - Always restart fail2ban after updating a jail
  - You must have the service installed and running to enable a jail
- Enable these jails:
  - `[apache-modsecurity]` jail if you use SSL for apache
  - `[apache-shellshock]` to protect against ShellShock bash command
- After you make a change, check the status 

Helpful settings:
- `ignoreip`: do NOT ban IPs in this list (`::1` is localhost in IPv6). Set this so you do not get locked ou

```bash
apt install fail2ban                                    # install package
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local     # copy config file

# health
systemctl restart fail2ban                              # restart fail2ban
systemctl status -l fail2ban                            # check fail2ban status
fail2ban-client status                                  # list all enabled jails

# --- /etc/fail2ban/jail.local config file --- #
ignoreip: 127.0.0.1/8 ::1 20.30.40.0/24 20.30.40.158/24 # DO NOT block these hosts/networks
bantime                 # how long host is banned
maxretry                # number of failures before fail2ban bans an IP for bantime setting

# configuring jails
[sshd]                  # ssh is enabled
...
enabled = true
port    = 65332         # if you changed ssh default port, set it here

```

## Database servers

DBs are not hard to secure:
- Should not be reachable from the internet - only internal servers 
  - Ex: DB for web server only accepts connections from web server
  - use `/ect/hosts.allow` and `/ect/hosts.deny` to manage this. `/ect/hosts.allow` overrides `/etc/hosts.deny`
  - Add IP for SSH and webserver to `.allow`
    - Never add `ALL: ALL`
    - Add all internal network IP addr
  - Deny everything in `.deny` file - overridden by `.allow`
    - Make sure you setup `.allow` file first!

```bash 
# --- /etc/hosts.allow --- #
ALL: 192.168.1.30               # Allows this IP (e.g. ssh IP)
ALL: 192.168.1.0/255.255.255.0  # Allow any IP from 192.168.1 network
ALL 192.168.1.                  # Wildcard - any IP from this network
ssh: 192.168.1.                 # Daemons - allow ssh from any host on that network

# --- /etc/hosts.deny --- #
ALL: ALL
```
### User access

Restrict user logins by IP:
- `%` is a wildcard, so never allow connections from `...'appuser'@'%'...`
- Only allow subnet access if its not the same subnet that your users are on

```bash
# Restrict access to single IP
GRANT SELECT ON mydb.* TO 'appuser'@'192.168.1.50' IDENTIFIED BY 'password';
FLUSH PRIVILEGES;

# Restrict access to subnet - not as good as single IP
GRANT SELECT ON mydb.* TO 'appuser'@'192.168.1.%` IDENTIFIED BY 'password';
FLUSH PRIVILEGES;
```

## Firewalls

Firewalls are easy to implement but difficult to implement well:
- Allow or disallow access to a network port
  - When admins start a service, they open a port for it 
  - Firewall allows port access from specific IPs or places
- Ubuntu uses uncomplicated Firewall (UFW)
  - Easy interface to `iptables` - use `ufw`, not `iptables`
  - inactive by default
  - configure, then activate it
- Allow access to ports that your server needs for functionality. Ex: 80 and 443 for webservers

```bash
sudo apt install ufw            # install package
sudo ufw status                 # check status - confirm its inactive
sudo ufw enable                 # activate ufw

ufw allow from 192.168.1.158 to any port 22         # allow SSH from this IP
ufw allow from 192.168.1.0/24 to any port 22        # allow SSH from entire subnet
ufw allow 80                                        # allow all traffic to port 80
ufw allow 443                                       # allow all traffic to port 443
```

## LUKS

You should encrypt any data on disk with personal or sensitive information:
- Choose full-disk encryption when you setup your Linux installation
  - Otherwise, a person can just boot a Live OS disc and mount the hard drive to view your data
- `cryptsetup` package lets us encrypt and decrypt disks
- Encrypting a disk removes any data on it, so use a clean one

```bash
apt install cryptsetup                              # 1. install packages
fdisk -l                                            # 2. verify the disk that you want to format
cryptsetup luksFormat /dev/<disk>                   # 3. format the disk
cryptsetup luksOpen /dev/sda <disk-name>            # 4. opens the disk and names it <disk-name> so you can format it 
fdisk -l                                            # 5. verify it worked in /dev/mapper/<disk-name>
mkfs.ext4 -L "usb_drive" /dev/mapper/<disk-name>    # 6. create fs on drive, add label with -L
mkdir /media/<disk-name>                            # 7. create mount point
mount /dev/mapper/<disk-name> /media/<disk-name>    # 8. mount on local fs

# --- unmount encrypted drive --- #
umount /media/<disk-name>                           # unmount the volume
cryptsetup luksClose /dev/mapper/<disk-name>        # close the volume

# --- remount encrypted drive --- #
fdisk -l                                            # get disk names
crytpsetup luksOpen /dev/sda <disk-name>            # open the volume
fdisk -l                                            # get disk names
mount /dev/mapper/<disk-name> /media/<disk-name>    # mount the volume

# --- change encrypted drive passphrase --- #
cryptsetup luksChangeKey /dev/sda -s 0
```