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

