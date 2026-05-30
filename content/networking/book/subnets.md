+++
title = 'Subnets'
date = '2026-05-30T08:08:31-04:00'
weight = 20
draft = false
+++

Subnetting divides a larger IP network into smaller, more manageable segments. Each subnet operates as an independent broadcast domain, which limits unnecessary traffic and gives administrators control over network boundaries. This page covers the mechanics of subnet masks, how addresses are structured and reserved within a subnet, and how IPv4 classful addressing organizes the address space.

## Subnet masks

IPv4 addresses each device on a network uniquely by combining an IP address with a *subnet mask*. Together, they identify which part of the address identifies the network and which part identifies the individual device.

A *subnet* is a logical subdivision of a larger IP network. Grouping devices into subnets limits broadcast traffic and organizes the address space. An *octet* is one of the four 8-bit groups that make up an IPv4 address, each representing a decimal value from 0 to 255 and separated by periods: `192.168.122.182`.

A subnet mask is a 32-bit number where binary 1s mark the network portion and binary 0s mark the host portion. The kernel performs a bitwise AND between the IP address and the mask to determine the network address. The count of leading 1s is the prefix length, written after a slash: `/24` means the first 24 bits are the network portion.

#### /24 example

`192.168.122.182/24` has a 24-bit network portion and an 8-bit host portion:

```
Address:  192.168.122.182  =  11000000.10101000.01111010.10110110
Mask /24: 255.255.255.0    =  11111111.11111111.11111111.00000000
                              |--------network (24 bits)--|--host--|
Network:  192.168.122.0    =  11000000.10101000.01111010.00000000
```

The first 24 bits identify the network (`192.168.122.0`). The last 8 bits identify the host (`182` = `10110110`). With 8 host bits, the subnet supports 2^8 - 2 = 254 usable addresses.

#### /20 example

A `/20` prefix extends the network portion by 4 bits into the third octet, shrinking the host portion to 12 bits:

```
Address:  192.168.122.182  =  11000000.10101000.01111010.10110110
Mask /20: 255.255.240.0    =  11111111.11111111.11110000.00000000
                              |------network (20 bits)--|---host---|
Network:  192.168.112.0    =  11000000.10101000.01110000.00000000
```

The network address is `192.168.112.0`. The 12 host bits support 2^12 - 2 = 4094 usable addresses across the range `192.168.112.1` through `192.168.127.254`. Compared to the 254 addresses a `/24` provides, the `/20` fits 16 times as many devices in the same subnet.

### Special purpose addresses

Not every address in a subnet is available for assignment to a device. Some addresses are reserved by the protocol for functions that the network stack depends on.

#### Broadcast address

The *broadcast address* is the last address in a subnet, formed by setting all host bits to 1. Any packet sent to the broadcast address is delivered to every device on the subnet.

For `192.168.122.182/24`, all 8 host bits become 1:

```
Address:   11000000.10101000.01111010.10110110
Mask /24:  11111111.11111111.11111111.00000000
Broadcast: 11000000.10101000.01111010.11111111  =  192.168.122.255
```

For the same address with a `/20` mask, all 12 host bits become 1:

```
Address:   11000000.10101000.01111010.10110110
Mask /20:  11111111.11111111.11110000.00000000
Broadcast: 11000000.10101000.01111111.11111111  =  192.168.127.255
```

### Multicast addresses

A *multicast* address identifies a group of devices that have opted into receiving traffic for that group. A single packet sent to a multicast address is delivered to all group members without sending a separate copy to each. Use multicast when the same data needs to reach multiple receivers efficiently. Streaming video to a set of network-attached displays is a common example: one stream travels the network once, and switches or routers replicate it only where group members diverge.

Multicast addresses occupy the `224.0.0.0/4` range. The first four bits are always `1110`:

```
Binary:   1110xxxx.xxxxxxxx.xxxxxxxx.xxxxxxxx
Decimal:  224.0.0.0  through  239.255.255.255
```

Well-known IPv4 multicast addresses:

| Address | Protocol | Purpose |
|:--------|:---------|:--------|
| 224.0.0.1 | General | All hosts on the subnet |
| 224.0.0.2 | General | All routers on the subnet |
| 224.0.0.12 | DHCP | DHCP servers and relay agents |
| 224.0.0.18 | VRRP | Virtual Router Redundancy Protocol |
| 224.0.0.102 | HSRP | Hot Standby Router Protocol (Cisco, v2) |
| 224.0.1.1 | NTP | Network Time Protocol servers |
| 224.0.0.113 | AllJoyn | AllJoyn device discovery |

See the full registry at the [IANA IPv4 Multicast Address Space Registry](https://www.iana.org/assignments/multicast-addresses/multicast-addresses.xhtml).

### Address classes

Before CIDR (Classless Inter-Domain Routing) replaced it in 1993, IPv4 used a *classful* addressing model. Each address class defined a fixed *classful subnet mask*, a default network boundary determined entirely by the leading bits of the address. The point where the network portion ends and the host portion begins is the *classful boundary*. Classful concepts still appear in legacy routing protocols, documentation, and certification material.

| Class | Leading bits | Subnet mask (bits) | Subnet mask (decimal) | First address | Last address |
|:------|:-------------|:-------------------|:----------------------|:--------------|:-------------|
| A | 0 | /8 | 255.0.0.0 | 0.0.0.0 | 127.255.255.255 |
| B | 10 | /16 | 255.255.0.0 | 128.0.0.0 | 191.255.255.255 |
| C | 110 | /24 | 255.255.255.0 | 192.0.0.0 | 223.255.255.255 |
| D | 1110 | N/A | N/A (multicast) | 224.0.0.0 | 239.255.255.255 |
| E | 1111 | N/A | N/A (reserved) | 240.0.0.0 | 255.255.255.255 |

## Private addresses (RFC 1918)

*Private addresses* are IPv4 ranges designated by RFC 1918 for internal use within an organization. Internet routers do not forward packets with private source or destination addresses. Traffic to or from a private address must pass through a NAT (Network Address Translation) device, which replaces the private address with a routable public IP before the packet leaves the organization.

Private addressing exists for two reasons. First, the IPv4 address space holds roughly 4.3 billion addresses total, and public addresses are scarce. Private ranges let every organization reuse the same address space internally without consuming public address allocations. Second, private hosts are not directly reachable from the internet, which limits exposure to external threats without requiring additional firewall rules to enforce it.

RFC 1918 defines one private range in each address class:

| Class | CIDR | First address | Last address | Total addresses |
|:------|:-----|:--------------|:-------------|:----------------|
| A | 10.0.0.0/8 | 10.0.0.0 | 10.255.255.255 | 16,777,216 |
| B | 172.16.0.0/12 | 172.16.0.0 | 172.31.255.255 | 1,048,576 |
| C | 192.168.0.0/16 | 192.168.0.0 | 192.168.255.255 | 65,536 |

### Identifying the address class from the first octet

You can confirm which class a private address belongs to by converting its first octet to binary and comparing the leading bits against the classful boundaries. Class A addresses begin with `0`. Class B addresses begin with `10`. Class C addresses begin with `110`.

```
Class A  10.0.0.0     first octet  10  =  00001010  leading bit:  0    Class A
Class B  172.16.0.0   first octet 172  =  10101100  leading bits: 10   Class B
Class C  192.168.0.0  first octet 192  =  11000000  leading bits: 110  Class C
```
