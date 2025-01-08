---
title: "Connecting to networks"
linkTitle: "Networks"
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

