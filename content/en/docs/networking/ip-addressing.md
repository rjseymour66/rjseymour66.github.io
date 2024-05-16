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

### Class A

Uses the following format:

_network_._host_._host_._host_

Network address is 1 byte long:
- first bit is reserved and remaining 7 bits can be used for addressing
- first bit must always be "off", or 0
  - Class A address must be between 0 and 127 in the first byte, inclusive:
    00000000 = 0
    01111111 = 127
  - All 0s is the default route
  - 127 is reserved for diagnostics
  - actual number of usable Class A network addresses is 126
- 3 bytes for the host address of the machine
- All host bits off is the network address: 10.0.0.0
- All host bits on is the broadcast address: 10.255.255.255

### Class B

Uses the following format:

_network_._network_._host_._host_