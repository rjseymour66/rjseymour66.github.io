---
title: "IP addressing"
weight: 60
---

An IP address is a numeric identifier assigned to each machine on an IP network
- logical address, not hardware.
  - Hardware is hard-coded on the NIC and used to find machines on a network
- IP addr designed to allow a host on one network to communication with a host on another network, regardless of LAN design

## Terminology

Bit
: One binary digit, either 0 or 1

Byte 
: 7 or 8 bits, depending on whether parity is used.

Octet
: Ordinary 8-bit binary number, typically displayed in decimal up to 255. Same as _octet_.

Network address
: Designation used in routing to send packets to a remote network. For example, 10.0.0.0 or 172.16.0.0.

IP address
: Logical address that defines a single host, but can also reference many or all hosts.

Broadcast address
: Used by applications and hosts to send information to all hosts on a network. For example:
  - 255.255.255.255: all netowrks and all hosts
  - 172.16.255.255: all subnets and hosts on 172.16.0.0
  - 10.255.255.255: all subnets and hosts on network 10.0.0.0

## IP addressing scheme

IP address is 32 bits long, divided into 4 sections of 1 octet each in one of the following formats:
- dotted-decimal: 172.16.30.56
- binary: 10101100.00010000.00011110.00111000
- hexadecimal: AC.10.IE.38

32-bit address is called structured, which means _hierarchical address_.
- hierarchical address can created 4.3 billion addresses
- hierarchical as opposed to flat, where each machine has its own address. This would mean that a router needs to store the IP address for every machine on the planet.
- hierarchical is structured by network and host, or network, subnet, and host

## Network addressing

Also called network number, and uniquely identifies each network
- _host address_ uniquely identifies each machine on a network
- there are classes of networks based on size:
  - Class A: large number of hosts
  - Class B: between large and small number of hsots
  - Class C: small number of hosts

```
            8 bits      8 bits      8 bits      8 bits
Class A:  ----------- ----------- ----------- -----------
          | network | |   host  | |   host  | |   host  |
          ----------- ----------- ----------- -----------

Class B:  ----------- ----------- ----------- -----------
          | network | | network | |   host  | |   host  |
          ----------- ----------- ----------- -----------

Class C:  ----------- ----------- ----------- -----------
          | network | | network | | network | |   host  |
          ----------- ----------- ----------- -----------

Class D: Multicast

Class E: Research
```

### Reserved addresses

| Address | Description |
|---|---|
| Network address all 0s | This network or segment |
| Network address all 1s | All networks |
| 127.0.0.1 | Reserved for loopback tests. The local host--you can send packets to yourself without generating network traffic |
| Host address all 0s | Network address, or any host on the network |
| Host address all 1s | All hosts, the broadcast address | 
| Entire IP set to 0s | Cisco routers designate the default route, or can mean any network |
| Entire IP set to 1s | Broadcast to all hosts on the current network | 


### Address space

| Class | Start | End |
|---|---|---|
| A | `10.0.0.0` | `10.255.255.255` |
| B | `176.16.0.0` | `176.16.255.255` |
| C | `192.168.100.0` | `192.168.100.255` |

### Class A

Uses the following format:

_network_._host_._host_._host_

Network address is 1 byte long:
- first bit is reserved and remaining 7 bits can be used for addressing
- first bit must always be "off", or `0`
  - Class A address must be between `0` and `127` in the first byte, inclusive:
    - `00000000` = 0
    - `01111111` = 127
  - All 0s is the default route
  - `127` is reserved for diagnostics
  - actual number of usable Class A network addresses is 126
- 3 bytes for the host address of the machine
- Network address - all host bits off: `10.0.0.0`
- Broadcast address - all host bits on: `10.255.255.255`

### Class B

Uses the following format:

_network_._network_._host_._host_

All CLass B network addresses start with `10`. First bit on, second bit off. This gives the following range:
- `10000000` = 128
- `10111111` = 191
- Uses 2 bytes for host addresses
- Network address - all host bits off: `176.16.0.0`
- Broadcast address - all host bits on: `176.16.255.255`

### Class C

Uses the following format:

_network_._network_._network_._host_

All Class C network addresses start with `110`. First and second bits on, third bit off. This gives the following range:
- `11000000` = 192
- `11011111` = 223
- Network address - all host bits off: `192.168.100.0`
- Broadcast address - all host bits on: `192.168.100.255`

### Class D and E

First octet `224` - `255` are reserved for D and E:
- Class D multicast: `224`-`239`
  - Multicast range is `244.0.0.0` through `239.255.255.255`
- Class E scientific: `240`-`255`

### Private IPs

Private IP addresses can be used on a private network, but they're not routable through the internet
- provides security but also saves address space
- This requires Network Address Translation (NAT)
  - takes a private IP address and converts it for use on the internet
  - provides security in that these IP addresses cannot be seen by external users - external users only see the public IP addr that the private IP is mapped to
  - multiple devices in a private network can be mapped to a single external, public IP address

### Reserved private address space

| Class | Start | End |
|---|---|---|
| A | `10.0.0.0` | `10.255.255.255` |
| B | `172.16.0.0` | `172.31.255.255` |
| C | `192.168.0.0` | `192.168.255.255` |


### Virtual IP (VIP)

Virtual IPs do not correspond to an actual physical network interface. For example:
- when a public IP address is substituted for the actual private IP address that was assigned to the network interface of the device, the public IP address is the _virtual address_.
- a subinterface configured on a physical router interface that allows you to create multiple IPs or subnets on one interface
- Whena  web proxy server substitutes its IP address for the sender's IP address before sending a packet to the internet

### APIPA

Automatic Private IP Addressing. 

When a DHCP server isn't avaialable, clients can automatically self-configure an IP address and subnet mask
- APIPA IP address range is `169.254.0.1` - `169.254.255.254`
- Client uses Class B subnet mask of `255.255.0.0`
- Hosts that use APIPA can communicate with each other, but not addresses that were statically configured
- Used as a fallback to DHCP
  - If your computer has an address in the APIPA range, then there is a DHCP issue

## IPv4 address types

Four IPv4 address types.

### Layer 2 broadcasts

Sent to all nodes on a LAN.
- Also called hardware broadcasts
- Only go out on a LAN and don't go past the LAN boundary (router)

Typical hardware address is 6 bytes and looks like this:
`0c.43.a4.f3.12.c2`

Broadcast is all 1s in binary, which is all Fs in hex:
`FF.FF.FF.FF.FF.FF`

### Broadcasts (Layer 3)

Sent to all nodes on the network. Its the address with all host bits on. For example:
- ARP request
  - When a host has a packet it has the logical IP address of the dest
  - Sends to default gateway if IP address is not on local network
  - If on local network, needs MAC address, so sends out a broadcast message that says 'if you are owner of IP address X, please forward your MAC address'

### Unicast

Address for a single interface, used to send packets to a single destination.

### Multicast (Class D)

Packets sent from a single source to many devices on different networks, referred to as _one-to-many_.
- Multicast allows point-to-multipoint communication, which is similar to broadcasts but lets multiple recipients receive messages without flooding messages to all hosts on a broadcast domain
- packets are sent to only hosts that subscribe to a group address
- Multicast addresses are `244.0.0.0` - `239.255.255.255` (Class D)

1. Sends messages or data to IP multicast group addresses
2. Routers forward copies of the packet out every interface that has hosts subscribed to a particular group address

## IPv6

We need IPv6 because we are running out of IP addresses. By default, IPv6 has the following features:
- IPSec for e2e security
- mobility, which means that a device can roam from one network to another without dropping connection
- packet header has 1/2 the fields at 64 bits
- 128 bits in length
- routing is more efficient
- doesn't use broadcast, uses multicast
  - also has unicast and anycast, which allows the same address to be placed on more than one device so that when the traffic is sent to one device addressed in this way, it is routed to the nearest host that shares the same address

### IPv6 addressing and expressions

- Eight groups of hex numbers and the groups are separated by colons
- 16-bit colon-delimited blocks

```
              subnet
               /  \
2001:0db8:2a3b:0016:0000:0000:1234:5678
|_____________|____|__________________|
 \ global prefix  /\  interface ID    /
  \-- 64-bits ---/  \---- 64-bits ---/
```
When you want to use IPv6 in a browser, you have to wrap it in brackets:

```
https://[2001:0db8:2a3b:0016:0000:0000:1234:5678]/page-name.html
```

### Shortened expression

You can leave out parts of the address to abbreviate it:
- drop any leading zeroes in each individual block: `2001:db8:2a3b:16:0:0:1234:5678`
- remove two blocks of zeroes by replacing them with double colons: `2001:db8:2a3b:16::1234:5678`
  - You CANNOT use double colons twice in the same address. When the router comes to the double colon, it replaces it with enough zeroes to reach 128 bits. It would not know how many zeros go in the first set or second.

### Address types

A single interface can have multiple types of IPv6 addresses assigned:

Unicast
: Packets addressed to unicast are delivered to a single interface. For load balancing, multiple addresses can use the same address.

Global unicast address
: These are your typical publicly routable addresses, same as in IPv4.

Link-local address
: Like APIPA addresses in IPv4, not meant to be routed and are unique for each LAN. For example, you can create a small LAN that does not need to be routed but still needs to share and access files and services locally.

  Link-local address is an `FE80::/10 address`.

Unique local address
: Nonrouting purposes, nearly globally unique. Designed to replace site-local addresses, so very similar to private IPv4 addresses: allow communication throughout a site while being routable to multiple local networks. Unique local can be routed within your organization or company.

Multicast
: Same as IPv4, packets addressed to multicast address are delivered to all interfaces identified by the multicast address.

Anycast
: identifies multiple intefaces, but the anycast packet is delivered to only one address--the first IPv6 address it finds in terms of routing distance. Could be called one-to-one-of-many addresses, or one-to-nearest.

### Special addresses

| Address | Description |
|---|---|
| `0:0:0:0:0:0:0:0` | Equal to `::`. Source address of host before assigned an IP with DHCP. |
| `0:0:0:0:0:0:0:1` | Equals `::1`. Loopback address, like `127.0.0.1` in IPv4. |
| `0::FFFF:192:168:100.1` | Format for a mixed IPv4/IPv6 mixed network environment. |
| `2000::/3` | Global unicast address range allocated for internet access. |
| `FC00::/7` | Unique local unicast range. |
| `FE80::/10` | Link-local unicast range. |
| `FF00::/8` | Multicast range. |
| `3FFF:FFF::/32` | Reserved for examples and documentation. |
| `2001:0DB8::/32` | Reserved for examples and documentation. |
| `2002::/16` | Used with 6to4 tunneling, which is an IPv4-to-IPv6 transistion system. You can send IPv6 packets over an IPv4 network without configuring explicit tunnels. |


### Stateless Address Autoconfiguration (SLAAC)

