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

### Static routing

A router automatically adds connected and local routes to its routing table. It learns other routes in one of two main ways:

- **Dynamic routing:** Routers use dynamic routing protocols (for example, OSPF) to share information with each other and build their routing tables.
- **Static routing:** An engineer or admin manually configures routes on the router.

To forward packets toward destinations that aren't in directly connected networks, the router must learn those destinations using one of these methods.

To forward a packet toward a destination that isn't directly connected, the router encapsulates the packet in a frame addressed to the MAC address of the **next hop**: the next router in the path to the destination.

#### Configuring static routes

Configure a static route with `ip route` in global configuration mode. The command has three argument options:

```text
ip route destination-network netmask next-hop
ip route destination-network netmask exit-interface
ip route destination-network netmask exit-interface next-hop
```

| Static route type                                                                 | Specifies                              | Behavior                                                      |
| --------------------------------------------------------------------------------- | -------------------------------------- | ------------------------------------------------------------- |
| [Recursive](#static-routes-specifying-the-next-hop)                               | Next-hop IP address                    | Requires multiple routing table lookups                       |
| [Directly connected](#static-routes-specifying-the-exit-interface)                | Exit interface                         | Appears as a directly connected network. Relies on proxy ARP. |
| [Fully specified](#static-routes-specifying-both-the-exit-interface-and-next-hop) | Exit interface and next-hop IP address | No recursive lookup or proxy ARP                              |

##### Static routes specifying the next hop

A static route that specifies only the next-hop IP address is a **recursive static route**. It's called recursive because the router needs multiple routing table lookups to forward a packet:

1. A lookup to find the IP address of the next hop.
2. A lookup to find which interface the next hop is connected to.

```text
R1# show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
. . .
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, GigabitEthernet0/1
L        192.168.1.1/32 is directly connected, GigabitEthernet0/1
S     192.168.3.0/24 [1/0] via 192.168.12.2
      192.168.12.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.12.0/24 is directly connected, GigabitEthernet0/0
L        192.168.12.1/32 is directly connected, GigabitEthernet0/0
```

In the static route `S     192.168.3.0/24 [1/0] via 192.168.12.2`:

- `S` is the code for a static route.
- `[1/0]` is the administrative distance (AD) and metric of the route, in that order.

The static route alone doesn't tell R1 which interface to forward the packet out of. R1 performs a second lookup for the next-hop IP address, 192.168.12.2. The most specific (and only) matching route for 192.168.12.2 is the connected route to 192.168.12.0/24, which specifies the G0/0 interface. After two lookups, R1 knows the next-hop IP address and the exit interface.

##### Static routes specifying the exit interface

You can specify the **exit interface**: the interface the router forwards packets out of.

```text
R1(config)# ip route 192.168.3.0 255.255.255.0 g0/0
R2(config)# ip route 192.168.1.0 255.255.255.0 g0/0
R2(config)# ip route 192.168.3.0 255.255.255.0 g0/1
R3(config)# ip route 192.168.1.0 255.255.255.0 g0/0
```

A static route that specifies only the exit interface is a **directly connected static route**. It's called directly connected because the route appears as a directly connected network in the routing table.

R1 thinks the 192.168.3.0/24 network is directly connected to its G0/0 interface. As a result:

1. R1 tries to send packets in frames addressed directly to PC3, rather than to the next hop.
2. R1 doesn't send an ARP request to learn the next-hop MAC address. It sends an ARP request to learn PC3's MAC address.
3. R1's ARP request is a broadcast, so it doesn't reach PC3. Broadcast messages don't go beyond their local network. However, R2 can use **proxy ARP** to reply on behalf of PC3. R2 tells R1 to send the packet to R2 G0/0's MAC address.

In the `show arp` output, the same MAC address (Hardware Addr) appears for both 192.168.12.2 and 192.168.3.11. It's the MAC address of R2's G0/0 interface:

```text
R1# show arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
. . .
Internet  192.168.3.11            0   5254.0003.e684  ARPA   GigabitEthernet0/0
. . .
Internet  192.168.12.2            0   5254.0003.e684  ARPA   GigabitEthernet0/0
```

Relying on proxy ARP has two downsides:

- **Proxy ARP disabled:** If proxy ARP is disabled on R2, R2 won't reply to R1's ARP request. R1 can't forward the packet to PC3.
- **Wasted memory:** R1 thinks each host in 192.168.3.0/24 is directly connected, so it tries to learn each host's MAC address. R1 needs a separate ARP entry for every host in 192.168.3.0/24. That could waste memory on R1 if the network has a lot of hosts.

##### Static routes specifying both the exit interface and next hop

A static route that specifies both the exit interface and the next hop is a **fully specified static route**. These are the same four static routes, configured as fully specified static routes:

```text
R1(config)# ip route 192.168.3.0 255.255.255.0 g0/0 192.168.12.2
R2(config)# ip route 192.168.1.0 255.255.255.0 g0/0 192.168.12.1
R2(config)# ip route 192.168.3.0 255.255.255.0 g0/1 192.168.23.2
R3(config)# ip route 192.168.1.0 255.255.255.0 g0/0 192.168.23.1
```

The router knows both the next-hop IP address and the exit interface. It doesn't need a recursive lookup or proxy ARP.

```text
R1# show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
. . .
S     192.168.3.0/24 [1/0] via 192.168.12.2, GigabitEthernet0/0
. . .
```

{{< admonition "Which type to use" tip >}}
Recursive and fully specified static routes perform about the same. You can use either without a noticeable difference. Avoid directly connected static routes in general.
{{< /admonition >}}

#### Configuring a default route

A **default route** is a route to the least-specific destination possible: 0.0.0.0/0. It matches all IP addresses from 0.0.0.0 through 255.255.255.255. Because it's the least-specific route possible, the router selects the default route only if the routing table has no more specific route.

A default route often provides a route to the internet. The router forwards packets toward the internet using the default route. The Internet Service Provider (ISP) infrastructure then forwards the packets to the proper destination.

If a router has no default route, the routing table shows `Gateway of last resort is not set` above the routes:

```text
R1# show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
. . .
Gateway of last resort is not set
. . .
```

Configure a default route with `ip route 0.0.0.0 0.0.0.0` followed by the next hop. This example points R1's default route to the ISP at 203.0.113.2:

```text
R1(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

To view only R1's static routes, use `show ip route static`. The ISP's IP address (203.0.113.2) is now the gateway of last resort:

```text
R1# show ip route static
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
. . .
       ia - IS-IS inter area, * - candidate default, U - per-user static route
. . .
Gateway of last resort is 203.0.113.2 to network 0.0.0.0
S*    0.0.0.0/0 [1/0] via 203.0.113.2
S     192.168.2.0/24 [1/0] via 192.168.12.2
S     192.168.3.0/24 [1/0] via 192.168.13.2
```

#### Route classifications

| Classification                                            | Based on                              | Values                                           |
| --------------------------------------------------------- | ------------------------------------- | ------------------------------------------------ |
| Network and host routes                                   | The route's destination               | A network of IP addresses or a single IP address |
| Recursive, directly connected, and fully specified routes | How the route's next hop is specified | Next-hop IP address, exit interface, or both     |


## Life of a packet

Follow one packet from PC1 to PC3 across three routers. At Layer 3, the packet is addressed to PC3 (IP address 192.168.3.11) for the whole journey. At Layer 2, each hop encapsulates the packet in a new frame addressed to the next hop. R3 finally addresses its frame to PC3.

### Topology

```text
PC1 (.11) --+                                                                                   +-- PC3 (.11)
            |                                                                                   |
            SW1 --- .1 G0/1 [R1] G0/0 .1 --- .2 G0/0 [R2] G0/1 .1 --- .2 G0/0 [R3] G0/1 .1 --- SW2
            |                                                                                   |
PC2 (.12) --+                                                                                   +-- PC4 (.12)
```

| Network         | Devices and addresses                       |
| --------------- | ------------------------------------------- |
| 192.168.1.0/24  | PC1 (.11), PC2 (.12), SW1, and R1 G0/1 (.1) |
| 192.168.12.0/24 | R1 G0/0 (.1) and R2 G0/0 (.2)               |
| 192.168.23.0/24 | R2 G0/1 (.1) and R3 G0/0 (.2)               |
| 192.168.3.0/24  | R3 G0/1 (.1), SW2, PC3 (.11), and PC4 (.12) |

### What each device does

#### Source host (PC1)

1. Addresses the packet to PC3.
2. Uses ARP to learn the MAC address of R1 G0/1, its default gateway.
3. Sends the packet in a frame addressed to R1 G0/1.

#### Switches (SW1 and SW2)

1. Learn the MAC addresses of connected devices.
2. Forward or flood frames in the LAN.

#### Routers (R1, R2, and R3)

1. Select the best route to forward the packet.
2. Use ARP to learn the next hop's MAC address.
3. Forward the packet in a frame addressed to the next hop.

### MAC address tables and ARP tables

Switches and routers each use a table to forward frames:

| Table             | Maps                          | Used by               | Purpose                                                               |
| ----------------- | ----------------------------- | --------------------- | --------------------------------------------------------------------- |
| MAC address table | MAC addresses to switch ports | Switches              | Forward frames out of the correct port                                |
| ARP table         | IP addresses to MAC addresses | Routers and end hosts | Encapsulate packets in frames with the proper destination MAC address |

### Addressing at each hop

The packet's source and destination IP addresses never change. Each router builds a new frame, so the frame's source and destination MAC addresses change at every hop:

| Hop       | Frame source (Layer 2) | Frame destination (Layer 2) | Packet source (Layer 3) | Packet destination (Layer 3) |
| --------- | ---------------------- | --------------------------- | ----------------------- | ---------------------------- |
| PC1 to R1 | PC1                    | R1 G0/1                     | PC1 (192.168.1.11)      | PC3 (192.168.3.11)           |
| R1 to R2  | R1 G0/0                | R2 G0/0                     | PC1 (192.168.1.11)      | PC3 (192.168.3.11)           |
| R2 to R3  | R2 G0/1                | R3 G0/0                     | PC1 (192.168.1.11)      | PC3 (192.168.3.11)           |
| R3 to PC3 | R3 G0/1                | PC3                         | PC1 (192.168.1.11)      | PC3 (192.168.3.11)           |

Each router also decrements the packet's time to live (TTL) by 1 before it forwards the packet.

### Hop 1: PC1 to R1

PC3 is in a different network, so PC1 must send the packet to its default gateway in a frame addressed to the default gateway's MAC address, not PC3's MAC address. PC1 and PC3 are in separate LANs, so they don't need to know each other's MAC addresses.

The information PC1 needs is R1 G0/1's MAC address. PC1 uses ARP to learn it.

#### PC1 learns R1 G0/1's MAC address with ARP

1. **ARP request:**
   - PC1 sends an ARP request to 192.168.1.1.
   - SW1 learns PC1's MAC address on G0/1.
   - SW1 floods the frame because the destination MAC address is broadcast (`ffff.ffff.ffff`).
   - PC2 ignores the broadcast.
2. **ARP reply:**
   - R1 adds an ARP table entry for PC1 (192.168.1.11).
   - R1 sends an ARP reply to PC1.
   - SW1 learns R1 G0/1's MAC address on G0/0.
   - SW1 forwards the frame to PC1.
3. **PC1 updates its ARP table:** PC1 adds an ARP table entry for R1 G0/1 (192.168.1.1).

#### Table entries after the ARP exchange

| Device | Table             | Entry                      | Step |
| ------ | ----------------- | -------------------------- | ---- |
| SW1    | MAC address table | PC1 MAC on port G0/1       | 1    |
| R1     | ARP table         | 192.168.1.11 to PC1 MAC    | 2    |
| SW1    | MAC address table | R1 G0/1 MAC on port G0/0   | 2    |
| PC1    | ARP table         | 192.168.1.1 to R1 G0/1 MAC | 3    |

#### PC1 sends the packet

1. PC1 encapsulates the packet (destined for PC3) in a frame addressed to R1 G0/1's MAC address.
2. SW1 already learned R1 G0/1's MAC address on G0/0, so it forwards the frame out of G0/0 without flooding.

### Hop 2: R1 to R2

1. R1 receives the frame from PC1. The destination MAC address is its own, so R1 de-encapsulates it and examines the packet inside.
2. R1 performs a routing table lookup for the most specific matching route:

    ```text
    R1# show ip route
    . . .
          192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
    C        192.168.1.0/24 is directly connected, GigabitEthernet0/1
    L        192.168.1.1/32 is directly connected, GigabitEthernet0/1
    S     192.168.3.0/24 [1/0] via 192.168.12.2, GigabitEthernet0/0
          192.168.12.0/24 is variably subnetted, 2 subnets, 2 masks
    C        192.168.12.0/24 is directly connected, GigabitEthernet0/0
    L        192.168.12.1/32 is directly connected, GigabitEthernet0/0
    ```

    The most specific matching route for PC3 (192.168.3.11) is `S 192.168.3.0/24 [1/0] via 192.168.12.2, GigabitEthernet0/0`. The next hop is R2 G0/0 (192.168.12.2), and the exit interface is G0/0.

3. R1 knows the next hop's IP address but not its MAC address, so it uses ARP:
   - R1 sends an ARP request for 192.168.12.2 out of G0/0.
   - R2 adds an ARP table entry for R1 G0/0 (192.168.12.1) and sends an ARP reply.
   - R1 adds an ARP table entry for R2 G0/0 (192.168.12.2).
4. R1 encapsulates the packet in a new frame addressed to R2 G0/0's MAC address and forwards it out of G0/0.

### Hop 3: R2 to R3

1. R2 receives the frame from R1. The destination MAC address is its own, so R2 de-encapsulates it and examines the packet inside.
2. R2 performs a routing table lookup for the most specific matching route:

    ```text
    R2# show ip route
    . . .
    S     192.168.1.0/24 [1/0] via 192.168.12.1, GigabitEthernet0/0
    S     192.168.3.0/24 [1/0] via 192.168.23.2, GigabitEthernet0/1
          192.168.12.0/24 is variably subnetted, 2 subnets, 2 masks
    C        192.168.12.0/24 is directly connected, GigabitEthernet0/0
    L        192.168.12.2/32 is directly connected, GigabitEthernet0/0
          192.168.23.0/24 is variably subnetted, 2 subnets, 2 masks
    C        192.168.23.0/24 is directly connected, GigabitEthernet0/1
    L        192.168.23.1/32 is directly connected, GigabitEthernet0/1
    ```

    The most specific matching route for PC3 (192.168.3.11) is `S 192.168.3.0/24 [1/0] via 192.168.23.2, GigabitEthernet0/1`. The next hop is R3 G0/0 (192.168.23.2), and the exit interface is G0/1.

3. R2 uses ARP to learn R3 G0/0's MAC address:
   - R2 sends an ARP request for 192.168.23.2 out of G0/1.
   - R3 adds an ARP table entry for R2 G0/1 (192.168.23.1) and sends an ARP reply.
   - R2 adds an ARP table entry for R3 G0/0 (192.168.23.2).
4. R2 encapsulates the packet in a new frame addressed to R3 G0/0's MAC address and forwards it out of G0/1.

### Hop 4: R3 to PC3

The last hop is different. PC3 is in a network directly connected to R3, so R3 forwards the packet directly to PC3 instead of to another router.

1. R3 receives the frame from R2. The destination MAC address is its own, so R3 de-encapsulates it and examines the packet inside.
2. R3 performs a routing table lookup. The most specific matching route for PC3 (192.168.3.11) is the connected route to 192.168.3.0/24 on G0/1. R3 doesn't need a next hop. It forwards the packet to PC3 directly.
3. R3 uses ARP to learn PC3's MAC address:
   - R3 sends an ARP request for 192.168.3.11 out of G0/1.
   - SW2 learns R3 G0/1's MAC address and floods the broadcast.
   - PC4 ignores the broadcast.
   - PC3 adds an ARP table entry for R3 G0/1 (192.168.3.1) and sends an ARP reply.
   - SW2 learns PC3's MAC address and forwards the reply to R3.
   - R3 adds an ARP table entry for PC3 (192.168.3.11).
4. R3 encapsulates the packet in a new frame addressed to PC3's MAC address and forwards it out of G0/1.
5. SW2 forwards the frame to PC3.
6. PC3 receives the frame. The destination MAC address is its own, so PC3 de-encapsulates it. The destination IP address is its own, so PC3 continues to de-encapsulate the message. The packet has reached its destination.

{{< admonition "Next hop vs. destination" tip >}}
R1 and R2 match static routes, so they forward the frame to a next-hop router. R3 matches a connected route, so it forwards the frame directly to the destination host. In both cases, the router uses ARP to learn the MAC address it needs.
{{< /admonition >}}


## Subnetting IPv4 networks

The original Internet Protocol standard (RFC 791) defined **classful addressing** in 1981. The internet grew quickly, and classful addressing proved too rigid. It used addresses inefficiently, and the pool of available IPv4 addresses was drying up.

### What is subnetting

Classful addressing doesn't let you create networks of appropriate sizes. The smallest network size, a class C network, contains 254 usable addresses (2<sup>8</sup> - 2 for the network and broadcast addresses).

**Classless Inter-Domain Routing (CIDR)**, pronounced "cider," was introduced in 1993. It supports the growing internet and uses the IPv4 address space more efficiently. CIDR replaces the rules of classful addressing with a more flexible system:

- Prefix lengths don't have to be /8, /16, or /24.
- The boundary between the network portion and host portion can fall in the middle of an octet. This allows prefix lengths like /23, /26, and /28.

**CIDR notation** is the /X format for an address's prefix length. It's named for CIDR because CIDR introduced it. Before CIDR notation, a netmask such as 255.255.255.0 always indicated the prefix length.

#### Address blocks

An **address block** is a range of IP addresses. The term can refer to a network before it's subnetted.

The 192.168.1.0/24 address block is a single /24 subnet that includes all addresses from 192.168.1.0 through 192.168.1.255. You can divide it into smaller subnets:

| Prefix length | Subnets | Addresses per subnet |
| ------------- | ------- | -------------------- |
| /24           | 1       | 256                  |
| /25           | 2       | 128                  |
| /26           | 4       | 64                   |
| /27           | 8       | 32                   |
| /28           | 16      | 16                   |

{{< admonition "Remember" tip >}}
Each bit you add to the prefix length doubles the number of subnets and halves the number of addresses in each subnet.
{{< /admonition >}}

### FLSM subnetting

There are two ways to divide an address block:

| Method                                    | Subnet sizes                                            |
| ----------------------------------------- | ------------------------------------------------------- |
| **Fixed-Length Subnet Masking (FLSM)**    | Equal                                                   |
| **Variable-Length Subnet Masking (VLSM)** | Vary, depending on how many addresses each subnet needs |

In the real world, you'll use VLSM. It uses a block of addresses more efficiently because you make each subnet only as large as it needs to be. This wastes fewer addresses.

#### Subnetting /24 address blocks

You can't change the network portion of an address block. If you're given the 192.168.1.0/24 address block, you can't assign 192.168.2.1 (or any other IP address outside 192.168.1.0/24) to a host.

This is the key to subnetting: to make subnets, you **borrow** bits from the host portion and add them to the network portion. You make subnets by changing the binary value of the borrowed bits.

For example, borrowing 1 bit from 192.168.1.0/24 creates two subnets: 192.168.1.0/25 and 192.168.1.128/25. The borrowed bit was part of the host portion of the original address block. It's part of the network portion of each subnet, as the /25 prefix length shows.

The number of subnets is 2<sup>x</sup>, where x is the number of borrowed bits. Borrowing 2 bits makes 4 (2<sup>2</sup>) subnets.

To find each subnet, count up in binary. In the first subnet, the borrowed bits are 00. The next values are 01, 10, and 11. Counting in binary works the same as counting in decimal, except you have only two digits: 0 and 1.

| Borrowed bits | Subnet           |
| ------------- | ---------------- |
| 00            | 192.168.1.0/26   |
| 01            | 192.168.1.64/26  |
| 10            | 192.168.1.128/26 |
| 11            | 192.168.1.192/26 |

##### Calculating the 5 attributes of a /24 subnet

| Attribute                   | Definition                                           | How to calculate                                                 |
| --------------------------- | ---------------------------------------------------- | ---------------------------------------------------------------- |
| **Network address**         | First address of the subnet                          | Set the host portion to all 0s                                   |
| **Broadcast address**       | Last address of the subnet                           | Set the host portion to all 1s                                   |
| **First usable address**    | First address in the subnet you can assign to a host | Add 1 to the network address (change the last bit to 1)          |
| **Last usable address**     | Last address in the subnet you can assign to a host  | Subtract 1 from the broadcast address (change the last bit to 0) |
| **Maximum number of hosts** | Number of IP addresses available to assign to hosts  | 2<sup>y</sup> - 2, where y is the number of host bits            |

The maximum number of hosts subtracts 2 because you can't assign the network and broadcast addresses to hosts.

| Prefix length | Subnet mask (binary)                  | Subnet mask (decimal) | Maximum number of hosts (2<sup>y</sup> - 2) |
| ------------- | ------------------------------------- | --------------------- | ------------------------------------------- |
| /24           | `11111111.11111111.11111111.00000000` | 255.255.255.0         | 254                                         |
| /25           | `11111111.11111111.11111111.10000000` | 255.255.255.128       | 126                                         |
| /26           | `11111111.11111111.11111111.11000000` | 255.255.255.192       | 62                                          |
| /27           | `11111111.11111111.11111111.11100000` | 255.255.255.224       | 30                                          |
| /28           | `11111111.11111111.11111111.11110000` | 255.255.255.240       | 14                                          |
| /29           | `11111111.11111111.11111111.11111000` | 255.255.255.248       | 6                                           |
| /30           | `11111111.11111111.11111111.11111100` | 255.255.255.252       | 2                                           |
| /31           | `11111111.11111111.11111111.11111110` | 255.255.255.254       | 2 (special case)                            |
| /32           | `11111111.11111111.11111111.11111111` | 255.255.255.255       | 1 (special case)                            |

#### /31 and /32 special cases

Prefix lengths /31 and /32 are special cases when you calculate the maximum number of hosts.

| Prefix length | Host bits | Use                                                                                     |
| ------------- | --------- | --------------------------------------------------------------------------------------- |
| /31           | 1         | **Point-to-point links**: connections between two routers, which need only two IP addresses |
| /32           | 0         | A single IP address in a route                                                          |

/31 subnets are more efficient, but /30 subnets are still more common. /31 technically breaks the network and broadcast address rule.

RFC 3021 defines /31 prefixes on point-to-point links. Both addresses are usable because a link between two routers doesn't need a network or broadcast address.

If you configure R1 G0/0 with a /31 prefix length (subnet mask 255.255.255.254), the router warns you to use /31 cautiously:

```text
R1(config-if)# ip address 203.0.113.0 255.255.255.254
% Warning: use /31 mask on non point-to-point interface cautiously
```

#### Finding subnets with the magic number

The **magic number** is the block size of each subnet in the octet where the subnet boundary falls. Use it to find subnet boundaries without converting to binary:

1. Find the **interesting octet**: the mask octet that isn't 255 or 0.
2. Subtract the interesting octet's value from 256. The result is the magic number.
3. Subnets start at multiples of the magic number in the interesting octet.

| Mask octet   | 128 | 192 | 224 | 240 | 248 | 252 | 254 | 255 |
| ------------ | --- | --- | --- | --- | --- | --- | --- | --- |
| Magic number | 128 | 64  | 32  | 16  | 8   | 4   | 2   | 1   |

The magic number equals the place value of the lowest network bit in the interesting octet. Each time that bit increments, a new subnet begins:

```text
/27 mask, 4th octet:   1   1   1   0   0   0   0   0   = 224
Bit value:           128  64  32  16   8   4   2   1
                               ^
                     lowest network bit = 32
```

##### Example: 192.168.1.77/27

1. The mask is 255.255.255.224, so the magic number is 256 - 224 = 32.
2. The multiples of 32 are 0, 32, 64, 96, and so on. 77 falls between 64 and 96.

| Attribute         | Value                        |
| ----------------- | ---------------------------- |
| Network address   | 192.168.1.64                 |
| Broadcast address | 192.168.1.95 (96 - 1)        |
| Usable range      | 192.168.1.65 to 192.168.1.94 |

##### Example: 172.16.45.10/20

1. The mask is 255.255.240.0. The interesting octet is the third octet, so the magic number is 256 - 240 = 16.
2. The multiples of 16 are 0, 16, 32, 48, and so on. 45 falls between 32 and 48.

| Attribute         | Value                                                              |
| ----------------- | ------------------------------------------------------------------ |
| Network address   | 172.16.32.0                                                        |
| Broadcast address | 172.16.47.255 (48 - 1 in the third octet, all 1s in the fourth)    |
| Usable range      | 172.16.32.1 to 172.16.47.254                                       |

#### Subnetting /16 and /8 address blocks

Larger blocks use the same method. You borrow bits from the host portion, but the interesting octet is the second or third octet instead of the fourth.

##### Example: 172.16.0.0/16 divided into /22 subnets

1. Borrow 6 bits (22 - 16). This creates 2<sup>6</sup> = 64 subnets.
2. The mask is 255.255.252.0. The interesting octet is the third octet, so the magic number is 256 - 252 = 4.
3. The subnets are 172.16.0.0/22, 172.16.4.0/22, 172.16.8.0/22, and so on, through 172.16.252.0/22.
4. Each subnet has 10 host bits, so it supports 2<sup>10</sup> - 2 = 1,022 hosts.

| Attribute         | First subnet (172.16.0.0/22)  |
| ----------------- | ----------------------------- |
| Network address   | 172.16.0.0                    |
| Broadcast address | 172.16.3.255                  |
| Usable range      | 172.16.0.1 to 172.16.3.254    |

##### Example: 10.0.0.0/8 divided into /12 subnets

1. Borrow 4 bits (12 - 8). This creates 2<sup>4</sup> = 16 subnets.
2. The mask is 255.240.0.0. The interesting octet is the second octet, so the magic number is 256 - 240 = 16.
3. The subnets are 10.0.0.0/12, 10.16.0.0/12, 10.32.0.0/12, and so on, through 10.240.0.0/12.
4. Each subnet has 20 host bits, so it supports 2<sup>20</sup> - 2 = 1,048,574 hosts.

| Attribute         | First subnet (10.0.0.0/12)    |
| ----------------- | ----------------------------- |
| Network address   | 10.0.0.0                      |
| Broadcast address | 10.15.255.255                 |
| Usable range      | 10.0.0.1 to 10.15.255.254     |

#### Powers of 2

Use this table to count subnets (2<sup>x</sup>) and hosts (2<sup>y</sup> - 2):

| Bits | 1 | 2 | 3 | 4  | 5  | 6  | 7   | 8   | 9   | 10    |
| ---- | - | - | - | -- | -- | -- | --- | --- | --- | ----- |
| 2<sup>n</sup> | 2 | 4 | 8 | 16 | 32 | 64 | 128 | 256 | 512 | 1,024 |

#### Choosing a prefix length from a requirement

Work backward from what the network needs:

| Requirement        | Find                                                        | Prefix length                  |
| ------------------ | ----------------------------------------------------------- | ------------------------------ |
| Number of hosts    | Smallest y where 2<sup>y</sup> - 2 is at least the host count  | 32 - y                         |
| Number of subnets  | Smallest x where 2<sup>x</sup> is at least the subnet count    | Original prefix length + x     |

##### Example: 50 hosts

2<sup>5</sup> - 2 = 30 is too small. 2<sup>6</sup> - 2 = 62 is enough, so y = 6. The prefix length is 32 - 6 = /26.

##### Example: 6 subnets from a /24

2<sup>2</sup> = 4 is too few. 2<sup>3</sup> = 8 is enough, so x = 3. The prefix length is 24 + 3 = /27.

### VLSM subnetting

**Variable-Length Subnet Masking (VLSM)** subnets an address block more efficiently than FLSM because it creates subnets of varying sizes.

To assign VLSM subnets:

1. Assign the largest subnet at the start of the address block.
2. Assign the second-largest subnet after it.
3. Repeat until you assign all subnets.

{{< admonition "Hosts include routers" note >}}
Router IP addresses count in the host totals. Any device with an IP address is a host. The term **end host** usually refers to PCs, servers, and similar devices, to distinguish them from network infrastructure devices like routers.
{{< /admonition >}}

#### Example: VLSM subnetting 192.168.1.0/24

Divide 192.168.1.0/24 into subnets for 100, 50, 20, and 2 hosts:

1. Choose a prefix length for each subnet from its host count.
2. Assign the largest subnet at the start of the block.
3. Assign each next-largest subnet starting at the address after the previous subnet's broadcast address.

| Hosts needed | Prefix | Subnet           | Usable range                   | Broadcast address |
| ------------ | ------ | ---------------- | ------------------------------ | ----------------- |
| 100          | /25    | 192.168.1.0/25   | 192.168.1.1 to 192.168.1.126   | 192.168.1.127     |
| 50           | /26    | 192.168.1.128/26 | 192.168.1.129 to 192.168.1.190 | 192.168.1.191     |
| 20           | /27    | 192.168.1.192/27 | 192.168.1.193 to 192.168.1.222 | 192.168.1.223     |
| 2            | /30    | 192.168.1.224/30 | 192.168.1.225 to 192.168.1.226 | 192.168.1.227     |

Addresses 192.168.1.228 through 192.168.1.255 remain free for future subnets.

### Practice problems

<details>
<summary>1. Find the network address, broadcast address, and usable range for 10.1.1.200/26.</summary>

The magic number is 256 - 192 = 64. 200 falls between 192 and 256.

- Network address: 10.1.1.192
- Broadcast address: 10.1.1.255
- Usable range: 10.1.1.193 to 10.1.1.254

</details>

<details>
<summary>2. Find the network address, broadcast address, and usable range for 192.168.5.130/29.</summary>

The magic number is 256 - 248 = 8. 130 falls between 128 and 136.

- Network address: 192.168.5.128
- Broadcast address: 192.168.5.135
- Usable range: 192.168.5.129 to 192.168.5.134

</details>

<details>
<summary>3. Find the network address, broadcast address, and usable range for 172.16.100.50/21.</summary>

The mask is 255.255.248.0. The magic number is 256 - 248 = 8 in the third octet. 100 falls between 96 and 104.

- Network address: 172.16.96.0
- Broadcast address: 172.16.103.255
- Usable range: 172.16.96.1 to 172.16.103.254

</details>

<details>
<summary>4. What's the smallest prefix length that supports 500 hosts?</summary>

2<sup>8</sup> - 2 = 254 is too small. 2<sup>9</sup> - 2 = 510 is enough, so y = 9. The prefix length is 32 - 9 = /23.

</details>

<details>
<summary>5. How many /28 subnets fit in a /24, and how many hosts does each support?</summary>

Borrowing 4 bits (28 - 24) creates 2<sup>4</sup> = 16 subnets. Each has 4 host bits, so it supports 2<sup>4</sup> - 2 = 14 hosts.

</details>

<details>
<summary>6. Which prefix length gives a point-to-point link the fewest addresses?</summary>

/31, defined in RFC 3021. If you follow the network and broadcast address rule, use /30, which gives 2 usable addresses.

</details>
