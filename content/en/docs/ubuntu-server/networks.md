---
title: "Connecting to networks"
linkTitle: "xNetworks"
weight: 90
# description:
---

## Hostname

The hostname gives a server its identity - it should reflect the server's purpose:
- Also helps to distinguish servers from one another
- Develop a naming scheme, such as `webserver-01`, `webserver-02`, etc.
- Default `PS1` prompt displays the hostname up to the first period (`.`)
- Hostname stored in `/etc/hostname` and `/etc/hosts` - CHANGE NAME IN BOTH!
  - `hostnamectl` only changes `/etc/hostname`, need to manually change `/etc/hosts`

```bash
hostname                                    # view hostname
hostnamectl set-hostname u24.<hostname>     # change hostname to <hostname> in /etc/hostname
vim /etc/hostname :vs /etc/hosts            # change hostname manually in both files

# --- /etc/hosts file description --- #

127.0.0.1 localhost                     # lets local server reach itself through the network stack
127.0.1.1 ubuntu-server24               # add'l local server addr and FQDN (<servername>.<org-domain-name>)
                                        # FQDN is not required
...
```

## Network interfaces

A server uses its network interface to connect to a network:


### Naming convention

Ubuntu recently changed the interface naming convention to make it more predictable
- a name like `enp0s3` can stay persistent between boots
- New naming convention references the physical location of the network card on the system's bus - name can't change unless you physically move it on the board or in the virtual network device.
- Ethernet (wired) network interfaces begin with `en`
- Wireless network interfaces begin with `wl`

`enp0s3`: An ethernet card on the system's first PCI bus in PCI slot 3
- `en`: Ethernet
- `p0`: The bus being used - `p0` references the system's first PCI bus (0-indexed)
- `s3`: PCI slot 3

### ip

Part of `iproute2` utility suite
- `iproute2` replaced `net-tools`
- `net-tools` included `ifconfig`, and `ip` replaces `net-tools`

```bash
ip addr show                # view network interfaces and current status
ip a                        # same as ip addr show
ip link set enp0s3 down     # bring enp0s3 interface down
ip link set enp0s3 up       # bring enp0s3 interface up
```

### ifconfig

Deprecated. Part of deprecated `net-tools` utility suite

```bash
apt install net-tools       # install package
ifconfig                    # view network interfaces
ifconfig enp0s3 down        # bring enp0s3 interface down
ifconfig enp0s3 up          # bring enp0s3 interface up
```

## Assign static IP address

Your IP should remain fixed and not changed:
- After installation, the server gets a dynamically assigned lease from the DHCP server
  - DHCP servers have a range of IP addresses that are assigned to hosts that request an assignment
  - Select an address OUTSIDE this range to avoid dupes


Two options when assigning fixed address to network host:
- Static IP assignment: Arbitrarily select an address outside the DHCP server's address assignment range
  - Configuration means that your server never requests an address from the DHCP server
- Static lease: Also called a DHCP reservation. You configure your DCHP to assign a specific address to the server each time it asks for one. Means that the DHCP server is the single source of truth for IP address assignment. This is recommended, when possible.


### Netplan

> Review the [Configuring networks](https://ubuntu.com/server/docs/configuring-networks#static-ip-address-assignment) docs for Ubuntu 24.04.

Assign server a static IP with Netplan:
- NetworkManager is installed by default only on Ubuntu Desktop
- YAML config files are in `/etc/netplan/`
- Netplan processes configuration files in lexical order based on filenames, so `99_config.yaml` is applied after the default `50-cloud-init.yaml`.
- Often, the DNS server (`nameservers`) address is the same as the default gateway, but that is not always the case
- When configuring networking over SSH, use tmux so your session won't end if there is an issue:
  - `tmux` keeps commands running in the background, even if the connection to the server is dropped
  - So start `tmux`, execute `netplan apply`, and the command will complete in the background

```bash
# --- Sample 24.04 config file --- #
cat /etc/netplan/99_config.yaml 
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: no
      addresses:
        - 192.168.56.52/24

netplan apply                   # apply config file
```

## Name resolution

When resolving names, Linux doesn't always resolve host names with the DNS server:
- Linux uses the `/etc/nsswitch.conf` file to determine the resources used to resolve names, and the order in which they are checked
- `/etc/hosts` contains IPs and hostnames - you can add to this file, but it is difficult to maintain
  - Easier to rely on central DNS server
- `/etc/resolv.conf` historically listed IP addresses for DNS servers that the system used to resolve host names
  - Since Ubuntu 22.04, it only redirects lookups to use `systemd-resolved`, which uses the Netplan config or DHCP server
  - You shouldn't manually edit `/etc/resolv.conf` - it is automatically generated by Network Manager, a legacy service that managed network interfaces
- `resolvectl` shows your current DNS servers

```bash
cat /etc/nsswitch.conf 
# /etc/nsswitch.conf
#
...
hosts:          files dns       # checks files (/etc/hosts) and then the DNS server
                                # specified by the DHCP server or the one in /etc/netplan/<file>

resolvectl                      # view DNS servers your system points to
...
```

## OpenSSH

Lets you open a command shell on a remote server:
- Consists of two components: server daemon (`sshd`) and the client (`ssh`) that lets you connect your workstation to a remote system running `sshd`
- Most distros install OpenSSH by default - configured to start and become enabled automatically
- Listens for connections on port 22 by default
- You have the same permissions as the user you are logged in as
- When you end session, all background processes you were running are terminated


```bash
# --- Management --- #
apt install openssh-client          # install client
apt install openssh-server          # install server
systemctl status ssh                # verify sshd is running and ready for connections
systemctl start ssh                 # start ssh
systemctl enable ssh                # if currently disabled, make ssh auto start at boot 
ss -tunlp | grep ssh                # verify sshd is listening for connections

# --- Connecting --- #
ssh 10.20.30.40                     # connect to server at 10.20.30.40 on port 22
ssh <username>@<addr>               # connect to server at <addr> as <username> on port 22
ssh -p 2242 <username>@<addr>       # connect to server at <addr> as <username> on port 2242
exit                                # end ssh session
```

### SSH keys

[Ubuntu docs](https://help.ubuntu.com/community/SSH/OpenSSH/Keys)

By default, you authenticate to a remote server with your password. You can use a public key to authenticate instead:
- more secure bc password is never transmitted
- you create a public and private SSH key pair - these files are linked by a complex algorithm
  - passphrase is optional but recommended for security
- when you connect to a server that has your public key, ssh checks your private key to verify your identity
  - public key has `rw` for owner, `r` for group and others
  - private key is owned by user that created it with `rw` permissions
- you copy public key to other servers
  - when you log in, it uses an algorithm to authenticate your private key with the public key stored on the server
- you can cache your SSH key passphrase with the ssh-agent
  - enter passprhase once per session

```bash
# --- Generating keys --- #
ssh-keygen                                          # Generate ed25519 keypair
ssh-keygen -t rsa -b 4096                           # Generate RSA keypair

ssh-copy-id -i ~/.ssh/id_rsa.pub <user>@<ip-addr>   # copy keys to remote - use ip addr or hostname

# --- Add key to ssh-agent --- #
eval $(ssh-agent)                                   # start ssh-agent, if not already running
ssh-add ~/.ssh/id_rsa                               # add public key to ssh-agent, requires passphrase

# --- Change passphrase of existing key --- #
ssh-keygen -p
```
### config file

Must be in `~/.ssh/` directory and be named `config`:
- Not there by default, but client checks that it exists and parses it when present
- Each `Host` entry has a `Hostname`, `User`, `Port` value
- Use IP address if you do not have a DNS entry for your server listed in `/etc/hosts`

```bash
man ssh_config              # get all config options

# --- config file format --- #
Host <hostname>             # logical name you assign
    HostName <ip-addr>      # IP addr or remote hostname (from /etc/hosts)
    User <username>         # user you ssh as
    Port <port>             # SSH port

# --- Sample config file --- #
Host u24
	HostName 10.20.30.40
	User linuxuser1
	Port 22
```

## ssh-copy-id

Copies your local SSH public key to remote server
- creates `~/.ssh/` directory if doesn't already exist
- creates `authorized_keys` file if doesn't already exist
- copies contents of `~/.ssh/id_rsa.pub` into `authorized_keys` file on target server
  - additional keys are appended to this file, one per line


## ss

```bash
ss -tunlp           # list listening ports
```