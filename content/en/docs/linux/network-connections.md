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

## Exploring network issues

When you run into issues, develop a troubleshooting plan:
1. Identify symptoms
2. Review recent network config changes
3. Formulate potential problem cause theories
4. Work your way up the OSI model:
   - Physical
   - Data link
   - Network
   - Transport
   - Session
   - Presentation
   - Application

### Terms

Bandwidth
: Measurement of the maximum data amount that can be transferred between two network points over a period of time, usually bytes per second (bps)

Throughput
: Measurement of the actual data amount that is transferred between two network points over a period of time. Bandwidth is the _maximum_ rate, while throughput is the _actual_ rate.
  
  As an analogy, a road might be able to handle traffic at 65 mph (bandwidth), but bad potholes slow traffic to 55 mph (throughput).

Saturation
: Also called _bandwidth saturation_ or _congestion_. When network traffic exceeds capacity.

  As an analogy, its when too many cars are on the roadway and traffic slows down.

Latency
: Time between a source sending a packet and the packet's destination receiving it. High latency is slow (an issue), and low latency is fast (desired).

  _Jitter_ describes when there is high deviation from a network's average latency. Often, high latency is caused by low bandwidth or saturation.

Routing
: Routers connect network segments and forward IP packets to the appropriate network segment, and eventually their final destination.
  
  A router has a buffer that holds network packets when the outbound queues become too long. If the network is saturated for too long, the router might drop the packets instead of delivering them. If the buffer is too large, that can cause buffer bloat, which is an increase in network latency in congested segments due to packets staying too long in the router's buffer.

### Timeouts and losses

Packet drop/packet loss is when a packet fails to reach its final destination. Common causes:
- unreliable network cables
- failing adapters
- network traffic congestion
- underperforming devices

UDP does not guarantee packet delivery. That's why there can be choppiness during a VoIP call.
TCP guarantees packet delivery, so drops/losses cause network delays. TCP retransmits packets, which can worsen congestion.
- Denial of service (DoS) attacks can target routers and cause them to drop packets.

Timeouts are preset time periods for handling issues/unplanned events. Common reasons:
- A system is down
- Incorrect IP address was used
- Service is not running or not offered on the system
- A firewall is blocking the traffic
- Network traffic is congested which causes packet loss

### Name resolution

Translating a systems fully qualified domain name (FQDN) and its IP address is called _name resolution_.
- Domain Name System (DNS) is a network protocol that uses a distributed database to provide the needed name resolutions
- Client-side DNS is when a system asks another server for name resolution information
  - configure client-side DNS with `/etc/resolv.conf` and `/etc/hosts`

Things to consider with name resolution:
- **Name server location**: If the client-side DNS name server that you set in `/etc/resolv.conf` is physically far away, your system will resolve names slowly
- **Caching**: A caching-only name server holds recent name resolutions in memory. Consider using software like `dnsmasq` to improve resolution speeds with caching.
- **Secondary server**: For enterprise-level DNS, consider a secondary server that receives its info from the first DNS server. This can increase name resolution speeds by offloading the burden from the first server.

### Configuration troubleshooting

#### Interface configurations

- NIC config and status is important
- Does the NIC have a static or DHCP-provided IP address?
- For `firewalld` systems, NetworkManager automatically adds a new device to the `default` zone

#### Ports and sockets

port
: A number used by protocols (ex: TCP or UDP) to identify which service or application is transmitting data.

socket
: A program connection to a port is a _socket_. A _network socket_ is one of the endpoints in the network connections two endpoints. The single endpoint is on the local system and bound to a port, so the network socket uses both an IP address (local system) and a port number

#### Localhost vs Unix socket

localhost
: Hostname for _local loopback interface_. Allows programs on current system to test or implement networking services with TCP.
  - IPv4 address 127.0.0.1
  - IPv6 address of ::1

unix socket
: Also called Unix domain socket. Similar to network socket, but the unix socket connects two endpoints on the same system.
  - Perform interprocess communications (IPC), which is similar to TCP/IP
  - Also called IPC sockets
  - Better performance than localhost because localhost uses standard network behavior that consume resources, like handshaking
  - Unix sockets and socket files understand linux permissions, so you can use access control

#### Adapters

Network adapters are system hardwaree that allows network communications
- Wired or wireless
- Not typically used in enterprise envs
- Commonly have faulty or failing hardware, or inefficient drivers

#### RDMA

Remote Direct Memory Access allows direct access between a client's and server's memory.
- Greatly reduces latency.
- Requires special hardware, such as soft-RoCE (RDMA over Converged Ethernet)

## Network performance

Commands to check for high latency and saturation:

| Command | Description |
|----|----|
| `iperf`, `iperf3` | Network throughput tests |
| `iftop -i <adaptor>` | Displays network bandwidth usage (throughput) for <adapter> in continuous graph |
| `mrt` | Displays approximate travel times and packet loss percentages between the first 10 routers in the path from the source to the destination in a continuous graph or report format |
| `nc` (`netcat`) | Network throughput tests |
| `netstat -s` | DEPRECATED. Displays summary stats that are broken down by protocol and contain packet rates, but not throughput. |
| `ping`, `ping6` | Simple ICMP packet throughput tests and display stats on items such as round-trip times. |
| `ss -s` | Displays summary stats that are broken down by socket type and contain packet rates but not throughput. |
| `tracepath`, `tracepath6` | Display approximate travel times between each router from the source to the destination, discovering the maximum transmission unit (MTU) along the way. |
| `traceroute`, `tracerout6` | Display approximate travel times between each router from the source to the destination. |

### iperf

```bash
iperf [OPTIONS]
-s # run as server
-c <server-address> # creates client that connects to server at <server-address>
-b size # sets bandwidth to size bps (default is 1 Mb)
-d # perform bidirectional test between client and server
-P n # creates and runs n parallel client threads
-e # provides enhanced output
-i n # pauses between periodic bandwidth reports for n seconds
-t n # stop server after n seconds
# server
iperf -s -t 120
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size:  128 KByte (default)
------------------------------------------------------------
```

### Overloaded routers

Sometimes routers get overloaded, like when the MTU is set too low. Use `tracepath` or `traceroute` to view this.

### mtr

`mtr` (my traceroute) combines `traceroute` and `ping` to document network availability and latency in a real-time chart, including packets path, travel time, and optionally jitter.

```bash
mtr <hostname-or-ip>
-o # what stats to view
-c # num of times a packet is sent through
-r # provide a static report
# L (loss) D (drop) A (time travel avg) J (jitter)

# continuous graph display
mtr -o "L D A J" google.com

# produce a static report
mtr -o "L D A J" -c 20 -r google.com
Start: 2024-04-22T08:45:33-0400
HOST: precision-5540              Loss%  Drop    Avg  Jttr
  1.|-- 2601:184:4881:68c0:9258:5  0.0%     0    5.3   0.2
  2.|-- 2001:558:4023:65::1        0.0%     0   21.7   1.7
  3.|-- po-302-1209-rur01.cambrid  0.0%     0   13.0   1.5
  4.|-- ???                       100.0    20    0.0   0.0
  5.|-- be-501-ar01.needham.ma.bo 80.0%    16   14.4   0.1
  6.|-- be-32021-cs02.newyork.ny.  0.0%     0   21.8   3.7
  7.|-- be-3211-pe11.111eighthave  0.0%     0   18.9   2.2
  8.|-- 2001:559:0:19::a          15.0%     3   19.0   2.4
  9.|-- 2607:f8b0:8009::1          0.0%     0   19.2   0.2
 10.|-- 2001:4860:0:1::5716        5.0%     1   19.9   2.1
 11.|-- 2001:4860:0:1::858e        0.0%     0   22.2   1.9
 12.|-- 2001:4860::c:4002:6522     0.0%     0   19.6   0.4
 13.|-- 2001:4860::9:4003:205c     0.0%     0   19.9   3.8
 14.|-- 2001:4860:0:1::83ed        0.0%     0   18.6   4.4
 15.|-- 2001:4860:0:1::3bbf        0.0%     0   19.5   0.6
 16.|-- lga34s40-in-x0e.1e100.net  0.0%     0   20.1   1.4
```

### Faulty adapter

The following commands show summary stats for the specified `<adapter>`:

- `ethtool -S <adapter>`
- `ip -s link show <adapter>`
- `ifconfig <adapter>` (DEPRECATED)
- `netstat -i <adapter>` (DEPRECATED)

```bash
# view network adaptor stats
ip -s link show wlp59s0
2: wlp59s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DORMANT group default qlen 1000
    link/ether 78:2b:46:1e:3c:aa brd ff:ff:ff:ff:ff:ff
    RX:  bytes packets errors dropped  missed   mcast           
    3277201789 3021878      0       0       0       0 
    TX:  bytes packets errors dropped carrier collsns           
     495625792 1134010      0       0       0       0 
```

### tshark

Wireshark is the GUI version of tshark, which performs advanced network packet analysis.

```bash
tshark
-i # specify the interface
-c # number of packets to capture

sudo tshark -i enp0s3 -c 10
Running as user "root" and group "root". This could be dangerous.
Capturing on 'enp0s3'
 ** (tshark:12322) 09:00:34.300736 [Main MESSAGE] -- Capture started.
 ** (tshark:12322) 09:00:34.300813 [Main MESSAGE] -- File: "/tmp/wireshark_enp0s3GEQHM2.pcapng"
    1 0.000000000    10.0.2.15 → 10.0.2.2     SSH 274 Server: Encrypted packet (len=220)
    2 0.000351255     10.0.2.2 → 10.0.2.15    TCP 60 58878 → 22 [ACK] Seq=1 Ack=221 Win=65535 Len=0
    3 0.512374174    10.0.2.15 → 10.0.2.2     SSH 290 Server: Encrypted packet (len=236)
    4 0.513037781     10.0.2.2 → 10.0.2.15    TCP 60 58878 → 22 [ACK] Seq=1 Ack=457 Win=65535 Len=0
    5 1.022202390    10.0.2.15 → 10.0.2.2     SSH 290 Server: Encrypted packet (len=236)
    6 1.022858936     10.0.2.2 → 10.0.2.15    TCP 60 58878 → 22 [ACK] Seq=1 Ack=693 Win=65535 Len=0
    7 1.566309211    10.0.2.15 → 10.0.2.2     SSH 290 Server: Encrypted packet (len=236)
    8 1.566886379     10.0.2.2 → 10.0.2.15    TCP 60 58878 → 22 [ACK] Seq=1 Ack=929 Win=65535 Len=0
    9 2.078744764    10.0.2.15 → 10.0.2.2     SSH 290 Server: Encrypted packet (len=236)
   10 2.079461178     10.0.2.2 → 10.0.2.15    TCP 60 58878 → 22 [ACK] Seq=1 Ack=1165 Win=65535 Len=0
10 packets captured

```

## Network configuration

You can use `nmcli` to view and troubleshoot adaptor settings.

### MAC addresses

On the local network, routers use MAC addresses to locate local systems (not IPs).
- IPv4 mapping: MACs are mapped with the Address Resolution Protocol (ARP) table
- IPv6 mapping: MACs are mapped with the Neighborhodd Discovery (NDisk) table

| Command | Description |
|----|----|
| `arp` | Displays the ARP table for network's neighborhood. |
| `ip neigh` | Displays ARP and NDisk tables for network's neighborhood, and checks for incorrect or duplicate MAC addresses. |

```bash
# arp
arp
Address                  HWtype  HWaddress           Flags Mask            Iface
_gateway                 ether   10::20:30:40:50     C                     wlp59s0

# ip neigh
ip neigh
10.0.0.1 dev wlp59s0 lladdr 10::20:30:40:50 REACHABLE
100::200:300:400:500 dev wlp59s0 lladdr 10::20:30:40:50 router REACHABLE
```

### DNS configuration

| Command | Description |
|----|----|
| `host <FQDN>` | Queries DNS server for FQDN and displays IP addr. |
| `dig <FQDN>` | Performs queries on DNS server for FQDN and displays all associated IP addresses. |
| `nslookup` | Executes DNS queries in interactive or noninteractive mode. |
| `whois` | Queries Whois servers and displays FQDN info. |

```bash
# test DNS lookup speeds
time nslookup www.linux.org 8.8.8.8
Server:		8.8.8.8
Address:	8.8.8.8#53

Non-authoritative answer:
Name:	www.linux.org
Address: 104.26.15.72
Name:	www.linux.org
Address: 104.26.14.72
Name:	www.linux.org
Address: 172.67.73.26
Name:	www.linux.org
Address: 2606:4700:20::ac43:491a
Name:	www.linux.org
Address: 2606:4700:20::681a:e48
Name:	www.linux.org
Address: 2606:4700:20::681a:f48


real	0m0.104s
user	0m0.004s
sys	0m0.008s
```

### nmap

Network Mapper (`nmap`) is usually used in pen testing but is helpful for network troubleshooting:

```bash
# view ports and their services
nmap -sT localhost
Starting Nmap 7.80 ( https://nmap.org ) at 2024-04-22 09:44 EDT
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00011s latency).
Not shown: 991 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
631/tcp  open  ipp
...

Nmap done: 1 IP address (1 host up) scanned in 0.11 seconds

# scan network segments and detect each OS
nmap -O localhost
Starting Nmap 7.80 ( https://nmap.org ) at 2024-04-22 09:47 EDT
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000086s latency).
Not shown: 991 closed ports
PORT     STATE SERVICE
...
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6.32
OS details: Linux 2.6.32
Network Distance: 0 hops

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1.62 seconds
```