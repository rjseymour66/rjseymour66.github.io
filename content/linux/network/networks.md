+++
title = 'Networks'
date = '2025-09-07T19:03:01-04:00'
weight = 10
draft = false
+++


## Hostname

The hostname identifies a server on a network. Choose a name that reflects the server's
purpose and develop a consistent naming scheme, such as `webserver-01`, `webserver-02`.
The hostname is stored in both `/etc/hostname` and `/etc/hosts`. Update both files when
you change it: `hostnamectl` only updates `/etc/hostname`.

The default shell prompt displays the hostname up to the first period (`.`). Use these
commands to view and change the hostname:

```bash
hostname                                        # view the current hostname
hostnamectl set-hostname <hostname>             # update /etc/hostname
vim /etc/hostname                               # edit /etc/hostname manually
vim /etc/hosts                                  # edit /etc/hosts manually
```

The `/etc/hosts` file maps IP addresses to hostnames for local resolution:

```bash
127.0.0.1 localhost                         # allows the server to reach itself through the network stack
127.0.1.1 ubuntu-server24                   # additional local address and optional FQDN (<servername>.<org-domain-name>)
```

## Network interfaces

A server uses its network interface to connect to a network.

### Naming convention

Ubuntu uses a predictable naming convention for network interfaces based on the physical
location of the hardware. A name like `enp0s3` reflects the card's position on the
system bus and does not change between reboots unless you physically move the hardware.

Ethernet interfaces begin with `en`. Wireless interfaces begin with `wl`.

The name `enp0s3` identifies an Ethernet card on the system's first PCI bus in slot 3:

| Segment | Meaning |
|:---|:---|
| `en` | Ethernet |
| `p0` | First PCI bus (0-indexed) |
| `s3` | PCI slot 3 |

### ip

`ip` is part of the `iproute2` utility suite, which replaced the older `net-tools`
package and its `ifconfig` command. Use these commands to inspect and manage interfaces:

```bash
ip addr show                # view network interfaces and current status
ip a                        # shorthand for ip addr show
ip link set enp0s3 down     # bring the enp0s3 interface down
ip link set enp0s3 up       # bring the enp0s3 interface up
```

### ifconfig

`ifconfig` is deprecated. It is part of the `net-tools` package, which `iproute2` has
replaced. Install it only if you need to support legacy scripts:

```bash
apt install net-tools       # install the package
ifconfig                    # view network interfaces
ifconfig enp0s3 down        # bring the enp0s3 interface down
ifconfig enp0s3 up          # bring the enp0s3 interface up
```

## Assign static IP address

After installation, a server receives a dynamically assigned address from the DHCP server.
For servers that need a consistent address, choose one of these two approaches:

| Method | Description |
|:---|:---|
| Static IP | You assign an address manually, outside the DHCP server's range. The server never requests an address from DHCP. |
| Static lease | The DHCP server assigns the same address to the server each time it connects. The DHCP server remains the single source of truth for address assignment. This approach is recommended when possible. |

When using a static IP, choose an address outside the DHCP server's assignment range to
avoid conflicts with dynamically assigned addresses.


### Netplan

Netplan manages network configuration on Ubuntu Server using YAML files in `/etc/netplan/`.
NetworkManager is available on Ubuntu Desktop only. Netplan processes configuration files
in lexical order, so `99_config.yaml` is applied after the default `50-cloud-init.yaml`.

The DNS server address (`nameservers`) is often the same as the default gateway, but not
always. Check your network configuration to confirm.

When configuring networking over SSH, run `netplan apply` inside a `tmux` session. If the
connection drops during the change, the command continues running in the background.

See the [Configuring networks](https://ubuntu.com/server/docs/configuring-networks#static-ip-address-assignment) Ubuntu docs for Ubuntu 24.04.

Apply the configuration after editing the file:

```bash
netplan apply
```

#### /etc/netplan/99_config.yaml

```bash
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
```

## Name resolution

Linux does not always use the DNS server to resolve hostnames. The `/etc/nsswitch.conf`
file controls which sources are checked and in what order.

`/etc/hosts` maps IP addresses to hostnames locally. You can add entries to this file,
but a central DNS server is easier to maintain at scale.

Do not edit `/etc/resolv.conf` manually. Since Ubuntu 22.04, it redirects lookups to
`systemd-resolved`, which reads its DNS configuration from Netplan or the DHCP server.
The file is generated automatically.

Use `resolvectl` to view your current DNS servers. The `nsswitch.conf` entry for hosts
shows the resolution order:

```bash
resolvectl                      # view DNS servers your system points to

cat /etc/nsswitch.conf
hosts:          files dns       # checks /etc/hosts first, then the DNS server
```

## ss

`ss` displays socket statistics. Use it to list open and listening ports:

```bash
ss -tunlp           # list listening ports
```