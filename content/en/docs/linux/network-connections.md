---
title: "Network connections"
weight: 70
description: >
  How to configure network connections.
---



## Configuration files

- Debian: `/etc/network/interfaces`
- Red Hat: `/etc/sysconfig/network-scripts`

```bash
# local hostname of system
cat /etc/hostname 
precision-5540

# Defines DNS server
cat /etc/resolv.conf 
# This is /run/systemd/resolve/stub-resolv.conf managed by man:systemd-resolved(8).
# Do not edit.
...

nameserver 127.0.0.53       # DNS server assigned to your network
options edns0 trust-ad      # 
search hsd1.ma.comcast.net  # additional domains used to search for hostnames

```

## Network manager CLI

```bash
# text-based menu
nmtui

# cli-based menu
nmcli
enp0s3: connected to Wired connection 1
        ...

lo: unmanaged
        "lo"
        loopback (unknown), 00:00:00:00:00:00, sw, mtu 65536

DNS configuration:
        servers: 10.0.2.3
        domains: hsd1.ma.comcast.net
        interface: enp0s3

Use "nmcli device show" to get complete information about known devices and
"nmcli connection show" to get an overview on active connection profiles.

Consult nmcli(1) and nmcli-examples(7) manual pages for complete usage details.

# show all devices
nmcli device show
```

## iproute2 utilities

[`ip` command in linux guide](https://www.linode.com/docs/guides/how-to-use-the-linux-ip-command/)

`ip` is the most used program in this project, replaces `ifconfig`:

```bash
# get all network interfaces
# UP - kernel thinks this connection is UP
# LOWER_UP - there is a physical data link connection (electric signal)
# link/ether - ethernet MAC address
ip link
# linux host itself
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: wlp59s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DORMANT group default qlen 1000
    link/ether 78:2b:46:1e:3c:aa brd ff:ff:ff:ff:ff:ff
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default 
    link/ether 02:42:f4:af:5c:07 brd ff:ff:ff:ff:ff:ff

# get all IP addresses
# lo - always inet 127.0.1/8
ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    ...
2: wlp59s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 78:2b:46:1e:3c:aa brd ff:ff:ff:ff:ff:ff
    ...
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:f4:af:5c:07 brd ff:ff:ff:ff:ff:ff
    ...

# hardware status
ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: wlp59s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DORMANT group default qlen 1000
    link/ether 78:2b:46:1e:3c:aa brd ff:ff:ff:ff:ff:ff
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default 
    link/ether 02:42:f4:af:5c:07 brd ff:ff:ff:ff:ff:ff


# only IP4
ip -4 addr

# only IP6
ip -6 addr

# show a specific device
ip addr show dev wlp59s0

# assign IP address (sudo). Not persistant until reboot
ip addr add 10.20.30.40/24 dev <device>

# remove IP address (sudo)
ip addr del 10.20.30.40/24 dev <device>

# bring interface down
ip link set dev enp0s3 down
ip link show

# bring interface up
ip link set dev enp0s3 up
ip link show

# view routing table
ip route
ip route list

# view specific routing table entry
ip route list 169.254.0.0/16
169.254.0.0/16 dev enp0s3 scope link metric 1000 

# add IP to routing table
ip route add 169.254.0.0/16 dev enp0s3

# delete IP to routing table
ip route del 169.254.0.0/16 dev enp0s3
```

- Local loopback interface is a special virtual network interface
- enp0s3 interface is the wired network connection for Linux.

#### Assign network address

```bash
# 1. specify host addr and netmask for the interface
ip address add 10.0.2.15/24 dev enp0s3

# 2. set default router for interface
ip route add default via 192.168.1.254 dev en0ps3

# 3. activate interface with link option
ip link set enp0s3
```

### net-tools legacy tools

The net-tools package is the old way to work on linux systems:

| Command    | Description |
|------------|-------------|
| `ethtool`  | Ethernet settings for the network interface |
| `ifconfig` | Displays or sets IP addr and netmask values for a network interface |
| `iwconfig` | Sets SSID and encryption key for a wireless interface |
| `route`    | Sets default router address |


## Advanced network features

If your network uses dynamic host configuration protocol (DHCP), you need a DHCP client program running on your computer. Common programs:
- `dhcpd` (most popular)
- `dhclient`
- `pump`

### Bonding

_Bonding_ is when you aggregate multiple interfaces into one virtual network device. You must load it as a kernel module:

```bash
# load the kernel module, creates bond0 network interface
modprobe bonding

# define bond0 interface
ip link add bond0 type bond mode 4

# add network interfaces to the bond
ip link set eth0 master bond0
ip link set eth1 master bond0
```

### netcat

`netcat` can act like either a network server or client, sending TCP or UDP packets. Its either `netcat` or `nc`:

```bash
nc host port
# nc and netcat link to the same location
readlink -f /usr/bin/nc
/usr/bin/nc.openbsd
readlink -f /usr/bin/netcat
/usr/bin/nc.openbsd

# HTTP re to server and see HTTP response and HTML code
$ printf "GET / HTTP/1.0\r\n\r\n" | nc website.com 80

######################
# simple chat app
# first host with -l (listen) option
nc -l 8000

# second host
nc <hostname> 8000


######################
# file transfers
# first host (receives file):
nc -l 8000 > file.txt

# second host (sends file):
nc hostname 8000 < file.txt
```

## Basic network troubleshooting

### ping or ping6

Send Internet Control Message Protocol (ICMP) packets to remote hosts. These packets work behind the scenes to track connectivity and provide control messages btwn systems:

```bash
ping 10.20.30.40
# send 8 packets
ping -c 8 www.google.com

# ping6 requires you specify the inteface
ping6 -c 4 fe80::c418:2ed0:aead:cbce%enp0s3
```
### traceroute

`traceroute` lets you see the network routers from client to server. It uses a feature of ICMP packets that restrict the number of network hops:

```bash
traceroute www.google.com
```

### mtr

mtr (my traceroute) combines `traceroute` and `ping` to document network availability and latency in a real-time chart:

```bash
mtr linux.org
```
### host

Tests a hostname. Queries the DNS server to determine the IP addresses assigned to the hostname:

```bash
# hostname
host www.linux.org
www.linux.org has address 172.67.73.26
www.linux.org has address 104.26.14.72
www.linux.org has address 104.26.15.72
www.linux.org has IPv6 address 2606:4700:20::681a:f48
www.linux.org has IPv6 address 2606:4700:20::ac43:491a
www.linux.org has IPv6 address 2606:4700:20::681a:e48

# IP address
host 69.147.82.60
```

### dig

`dig` displays all DNS data records associated wtih a host or network:

```bash
dig www.linux.org
```

### nslookup

`nslookup` is a network administration command-line tool for querying the Domain Name System to obtain the mapping between domain name and IP address, or other DNS records:

```bash
nslookup
> www.google.com
Server:		127.0.0.53
Address:	127.0.0.53#53
...
> www.wikipedia.org
Server:		127.0.0.53
Address:	127.0.0.53#53
...
> exit
```

### whois

`whois` attempts to connect to the centralized Internet domain registry at `http://whois.networksolutions.com` to retrieve who registered the domain name:

```bash
whois linux.com
```

## Advanced network troubleshooting

### netstat

Part of the net-tools package, can provide a lot of info about your machine's network connections:

```bash
# list all open network connections
netstat

# tcp connections
netstat -t

# udp connections
netstat -u

# apps and their ports
netstat -l

# statistics for different types of packets (determine if there are issues with a protocol)
netstat -s
```

### ss 

`ss` links which system processes are using which network sockets that are active. A program connection to a port is a _socket_:

```bash
# listening and est. tcp sessions and their process
ss -anpt
```

### tcpdump

Legacy tool, captures network data on the system and can do rudimentary packet decoding, packet filtering.

### wireshark and tshark

Wireshark is the GUI version of tshark, which performs advanced network packet analysis.