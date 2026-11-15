+++
title = 'Routing Fundamentals'
date = '2026-11-13T21:52:49-05:00'
weight = 30
draft = false
+++

## Routing fundamentals

**Routing** refers to two processes:

- How routers build their **routing table**, a database of known destinations and how to forward packets toward them.
- How routers forward packets.

### How hosts send packets

A host must encapsulate a packet in a frame before it sends it. Packets never cross the cable (or radio waves) without a frame.

The frame's destination MAC address depends on the packet's destination IP address:

- **Same network:** The destination MAC address is the destination host's MAC address.
- **Different network:** The host sends the packet to its **default gateway**, the router that provides connectivity to other networks. The destination MAC address is the default gateway's MAC address.

For example, for PC1 and PC2 to send packets to destinations outside 192.168.1.0/24, they encapsulate the packet in a frame addressed to the MAC address of R1's G0/0 interface.

#### Default gateway

The default gateway's IP address is usually the first usable address of the network:

| Network        | Default gateway |
| -------------- | --------------- |
| 192.168.1.0/24 | 192.168.1.1     |
| 192.168.2.0/24 | 192.168.2.1     |

A host learns its default gateway's IP address in one of two ways:

- **Manual configuration:** An admin specifies the default gateway on each device. This is rare for user devices like PCs.
- **Dynamic Host Configuration Protocol (DHCP):** The host automatically learns information like the default gateway's IP address and its own IP address. User devices usually use this method.

To learn the default gateway's MAC address, the host sends an ARP request to the default gateway's IP address.

#### Frame and packet addressing

| Destination       | Packet destination IP address | Frame destination MAC address |
| ----------------- | ----------------------------- | ----------------------------- |
| Same network      | Destination host              | Destination host              |
| Different network | Destination host              | Default gateway               |

In either case, the host uses ARP to learn the MAC address of the destination host or the default gateway.

### Basics of routing

When a router receives a frame, it acts on the destination addresses:

| Condition                                          | Router action                                               |
| -------------------------------------------------- | ----------------------------------------------------------- |
| Frame's destination MAC address isn't the router's | Discards the frame                                          |
| Frame's destination MAC address is the router's    | De-encapsulates the frame and examines the packet           |
| Packet's destination IP address is the router's    | Continues to de-encapsulate. The message is for the router. |
| Packet's destination IP address isn't the router's | Looks up the destination IP address in its routing table    |
| Routing table has a suitable route                 | Forwards the packet according to that route                 |
| Routing table has no suitable route                | Discards the packet                                         |

#### Routing table

The routing table is a database of destinations known by the router. Think of it as a set of instructions:

- To send a packet to destination X, forward the packet to next hop Y.
- If the destination is in a directly connected network, forward the packet directly to the destination.
- If the destination is the router's own IP address, continue to de-encapsulate the message. Don't forward the packet.

View the routing table with `show ip route`:

```text
R1# show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR
 
Gateway of last resort is not set
```

Configure IP addresses on the router and enable its interfaces:

```text
R1# configure terminal
R1(config)# interface g0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# interface g0/1
R1(config-if)# ip address 192.168.2.1 255.255.255.0
R1(config-if)# no shutdown
```

The routing table now has two connected routes (code `C`) and two local routes (code `L`):

```text
R1# show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
. . .
Gateway of last resort is not set
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, GigabitEthernet0/0
L        192.168.1.1/32 is directly connected, GigabitEthernet0/0
      192.168.2.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.2.0/24 is directly connected, GigabitEthernet0/1
L        192.168.2.1/32 is directly connected, GigabitEthernet0/1
```

The line `192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks` isn't a route. It means the routing table has two routes to subnets within the 192.168.1.0/24 class C network, with two different netmasks (/24 and /32). The 192.168.2.0/24 line means the same thing.

##### Connected routes

A **connected route** is a route to the network an interface is connected to. The router automatically adds one connected route for each interface that has an IP address and is in an up/up state. Check the interface state with `show ip interface brief`.

To display only lines that include `C`, filter the output with a pipe (`|`) followed by `include C`:

```text
R1# show ip route | include C
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
C        192.168.1.0/24 is directly connected, GigabitEthernet0/0
C        192.168.2.0/24 is directly connected, GigabitEthernet0/1
```

A **network route** is a route to more than one destination IP address. It's a route to a network, rather than to a single destination IP address. A connected route is a network route.

##### Local routes

A **local route** is a route to the exact IP address configured on the router's interface. It tells the router that packets destined for that IP address are for the router itself. The router doesn't forward these packets. It receives them for itself.

The local route distinguishes the router's own IP address from other IP addresses in the connected network.

To view only R1's local routes, use `show ip route | include L`:

```text
R1# show ip route | include L
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
. . .
L        192.168.1.1/32 is directly connected, GigabitEthernet0/0
L        192.168.2.1/32 is directly connected, GigabitEthernet0/1
```

A local route uses a /32 prefix length, so all bits of the netmask are set to 1. A local route specifies only a single IP address. For example, 192.168.1.1/32 includes only 192.168.1.1. All bits are part of the network portion and can't change.

##### Host routes and network routes

| Route type        | Prefix length    | Destination              | Example         |
| ----------------- | ---------------- | ------------------------ | --------------- |
| **Host route**    | /32              | A single host IP address | Local route     |
| **Network route** | Shorter than /32 | A network                | Connected route |

#### Route selection

**Route selection** is how a router decides which route in its routing table to use to forward a packet. The router selects the most specific matching route:

- **Matching route:** The packet's destination IP address is part of the network specified in the route. If it isn't, the router can't forward the packet using this route.
- **Most specific:** The route with the longest prefix length.

For example:

1. R1 receives a frame on its G0/0 interface. The destination MAC address is its own, so R1 de-encapsulates the frame and examines the packet inside.
2. The packet's destination IP address is 192.168.1.1. R1 looks up the routing table and finds two matching routes: the connected route to 192.168.1.0/24 and the local route to 192.168.1.1/32. R1 selects the most specific route: 192.168.1.1/32.
3. R1 selected a local route, so it receives the packet for itself. It doesn't forward the packet.

{{< admonition "Layer 2 vs 3" note >}}
A router looks up a packet's destination IP address in its routing table and looks for the most specific matching route. A switch looks up a frame's destination MAC address in its MAC address table and looks for an exact match. Partial matches don't count.
{{< /admonition >}}

#### Actions a router can take on a packet

A router never floods packets. It forwards the packet, receives the packet for itself, or drops the packet:

| Matching condition                                                      | Router's action                                                  |
| ----------------------------------------------------------------------- | ---------------------------------------------------------------- |
| The packet's destination IP address matches one or more nonlocal routes | Forward the packet according to the most specific matching route |
| The packet's destination IP address matches a local route               | Receive the packet for itself                                    |
| The packet's destination IP address doesn't match any routes            | Drop the packet                                                  |
