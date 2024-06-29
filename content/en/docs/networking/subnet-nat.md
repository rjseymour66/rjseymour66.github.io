---
title: "Subnets and NAT"
weight: 70
---

## Basics

Subnetting is when you take one network address range and create multiple subnetworks within that range. Benefits include:
- **Reduced network traffic**: routers keep most traffic on the local network, only packets destined for that network travel through its router. Each router creates a broadcast domain, so the more broadcast domains you create, the smaller the less network traffic on each segment.
- **Optimized network performance**: Reward for reducing network traffic
- **Simplified management**: Easier to ID and isolate network problems in a group of smaller connected networks than within one giant network
- **Facilitated spanning of large geographical distances**: Connecting multiple smaller networks is more efficient bc data doesn't have to travel as far

## Create a subnet

To create a subnet, you take bits from the host portion of the IP address and resever them to define the subnet address. Use the following process to determine your needs:
1. Determine the number of network IDs
   - one for each subnet
2. Determine the number of hosts per subnet:
   - each TCP/IP host
   - each router interface
3. Using info from previous steps, create the following:
   - One subnet mask for the entire network
   - Unique subnet ID for each physical segment
   - Range of host IDs for each subnet

#### Remember powers of 2:

- Each successive power of 2 is double the previous one.
- For example, `2^3 = 8`, so `2^4 = 16`

## Subnet masks

A 32-bit value that allows the recipient of IP packets to distinguish the network ID portion of the IP address from the host ID portion of the IP address
- Network admin creates a 32-bit subnet mask of 1s and 0s
- 1s represent the network (or subnet) addresses

### Default subnet masks

| Class | Format | Default subnet mask | Subnet mask (binary) |
|---|---|---|---|
| A | _network_._host_._host_._host_ | `255.0.0.0` | `11111111.00000000.00000000.00000000` |
| B | _network_._network_._host_._host_ | `255.255.0.0` | `11111111.11111111.00000000.00000000` |
| C | _network_._network_._network_._host_ | `255.255.255.0` | `11111111.11111111.11111111.00000000` |


## Classless Inter-Domain Routing (CIDR)

Method that ISPs use to allocate a number of addresses to a company or home connection.
- provided addresses in a block size
- For example: `192.168.10.32/28`
  - `/XX` is the number of bits turned on (`1`). So `28` is the network
  - Most available is `/30` because you need to keep at least 2 bits for network and broadcast addresses
  - So `255.0.0.0` is a a `/8` because all bits in the first octet are on

### All subnets and CIDR vals

| Subnet mask | CIDR value | Class
|---|---|---|
| `255.0.0.0` | `/8` | A |
| `255.128.0.0` | `/9` | A |
| `255.192.0.0` | `/10` | A |
| `255.224.0.0` | `/11` | A |
| `255.240.0.0` | `/12` | A |
| `255.248.0.0` | `/13` | A |
| `255.252.0.0` | `/14` | A |
| `255.254.0.0` | `/15` | A |
| `255.255.0.0` | `/16` | A, B |
| `255.255.128.0` | `/17` | A, B |
| `255.255.192.0` | `/18` | A, B |
| `255.255.224.0` | `/19` | A, B |
| `255.255.240.0` | `/20` | A, B |
| `255.255.248.0` | `/21` | A, B |
| `255.255.252.0` | `/22` | A, B |
| `255.255.254.0` | `/23` | A, B |
| `255.255.255.0` | `/24` | A, B, C |
| `255.255.255.128` | `/25` | A, B, C |
| `255.255.255.192` | `/26` | A, B, C |
| `255.255.255.224` | `/27` | A, B, C |
| `255.255.255.240` | `/28` | A, B, C |
| `255.255.255.248` | `/29` | A, B, C |
| `255.255.255.252` | `/30` | A, B, C |

## Subnetting Class C

Class C only has 8 bits available for defining hosts:

| Binary | Decimal | CIDR |
|---|---|---|
| 00000000 | 0   | /24 |
| 10000000 | 128 | /25 |
| 11000000 | 192 | /26 |
| 11100000 | 224 | /27 |
| 11110000 | 240 | /28 |
| 11111000 | 248 | /29 |
| 11111100 | 252 | /30 |

### Quick subnetting

Answer the following questions:
- How many subnets does the chosen subnet mask produce
  - 2^x == number of subnets, where x is the number of 1s in the subnet mask. 11100000 gives 8 subnets, bc 2^3 = 8.
- How many valid hosts per subnet are available
  - 2^y - 2 == number of hosts, where y is the number of 0s in the subnet mask. 11100000 gives 30 hosts, because 2^5 = 32, and 32 - 2 = 30. You subtract 2 for the broadcast and network addresses.
- What are the valid subnets?
  - ???????????????
- What's the broadcast addr of each subnet?
  - The broadcast address is always the number right before the next subnet. The 64 subnet has a broadcast address of 127 because the next subnet starts at 128.
- What are the valid hosts in each subnet?
  - All numbers between the subnets, omitting the 0s and 1s. If 64 is the subnet number and 127 is the broadcast, so valid addresses are 65 - 126.
