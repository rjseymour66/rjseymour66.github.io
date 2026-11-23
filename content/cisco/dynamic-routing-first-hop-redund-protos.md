+++
title = 'Dynamic routing first hop redundancy protocols'
date = '2026-11-20T16:15:14-05:00'
weight = 50
draft = false
+++

## Dynamic routing

With dynamic routing, routers communicate with each other and build their routing tables automatically.

### Dynamic routing vs static routing

**Dynamic routing** is a process where routers share network information with each other. Routers use this information to build their routing tables, so an administrator doesn't have to configure each route manually.

Routers share this information using a **routing protocol**. A routing protocol defines two things:

- How routers communicate with each other to share routing information
- How routers use that information to build their routing tables

Sharing routing information is also called **advertisement**.

Dynamic routing has two main advantages over static routing: adaptability and scalability.

#### Adaptability

Static routes can't react to changes in the network. Suppose a hardware failure on a router in the path makes a route invalid. The static route doesn't adjust automatically to find an alternate path.

If a router can't reach a route's next-hop IP address, it removes the route from the routing table.

In the routing table, route type `D` indicates a route learned through **Enhanced Interior Gateway Routing Protocol (EIGRP)**. `D` stands for **Diffusing Update Algorithm (DUAL)**, the algorithm EIGRP uses to calculate routes.

#### Scalability

Static routes have their own advantages: predictability and control. You configure static routes manually by specifying the next hop for each destination. This makes them useful when you want to control the exact path that packets take.

### Types of routing protocols

Routing protocols fall into two main categories:

| Category | Full name                 | Exchanges routing information                                                                      |
| :------- | :------------------------ | :------------------------------------------------------------------------------------------------- |
| IGP      | Interior Gateway Protocol | Within a single **autonomous system (AS)**, the network of a single organization                   |
| EGP      | Exterior Gateway Protocol | Between different autonomous systems, such as between an enterprise and an ISP or between two ISPs |

**Gateway** is an old term for a router. We call them routers today, but some contexts still use the term gateway.

You can further categorize routing protocols by **algorithm type**. The algorithm type describes how routers share information and calculate routes.

The following table shows the five routing protocols in common use today:

<table class="merged">
  <thead>
    <tr><th>Category</th><th>Algorithm type</th><th>Routing protocol</th></tr>
  </thead>
  <tbody>
    <tr><td rowspan="4">IGP</td><td rowspan="2">Distance-vector</td><td>Routing Information Protocol (RIP)</td></tr>
    <tr><td>Enhanced Interior Gateway Routing Protocol (EIGRP)</td></tr>
    <tr><td rowspan="2">Link-state</td><td>Open Shortest Path First (OSPF)</td></tr>
    <tr><td>Intermediate System to Intermediate System (IS-IS)</td></tr>
    <tr><td>EGP</td><td>Path-vector</td><td>Border Gateway Protocol (BGP)</td></tr>
  </tbody>
</table>

BGP is the only EGP in use today.

#### Interior gateway protocols

##### Distance-vector protocols

Two main distance-vector routing protocols are in use today:

| Protocol                                           | Developer         | Where it's used                                                                                                                        |
| :------------------------------------------------- | :---------------- | :------------------------------------------------------------------------------------------------------------------------------------- |
| Routing Information Protocol (RIP)                 | Industry standard | A very simple protocol, usually used only in very small networks and labs                                                              |
| Enhanced Interior Gateway Routing Protocol (EIGRP) | Cisco             | More advanced than RIP, and in use in many large-scale networks today. It's sometimes called an **advanced distance-vector** protocol. |

Routers using a distance-vector protocol share their known networks and their metric to reach each one. A **metric** measures the efficiency of a route to the destination network. A lower metric is preferred. Metrics are similar to STP's root cost, which measures the efficiency of a path to the root bridge.

Each router doesn't have a complete map of the network. This is the key characteristic of distance-vector routing protocols. For each destination network a router learns about, it knows only the metric and the next-hop router.

Distance-vector routing is sometimes called **routing by rumor**. Routers don't have a complete map of the network. They know only what their neighbors tell them.

##### Link-state protocols

Two main link-state routing protocols are in use today, and both are industry standards:

- Intermediate System to Intermediate System (IS-IS), most commonly used in service-provider networks such as ISP networks
- Open Shortest Path First (OSPF)

With a link-state routing protocol, each router creates a connectivity map of the network:

1. Each router shares information about its connected links and the state of those links, hence the name **link state**.
2. Routers share this information with all routers in the network, not only with directly connected neighbors. This way, every router builds the same connectivity map.
3. Each router uses the map to calculate its best route to each destination in the network.

Link-state protocols require more router CPU and memory than distance-vector protocols, which can be a concern in very large networks:

- **Memory:** the router stores the map in a structure called the **link-state database (LSDB)**.
- **CPU:** calculating routes from the LSDB can be CPU-intensive.

You can overcome this limitation in other ways, such as dividing the network into areas.

#### Exterior gateway protocols

Modern networks widely use only one EGP: Border Gateway Protocol (BGP). BGP uses a **path-vector** algorithm to calculate routes. The path is the series of autonomous systems a packet travels through on the route to the destination. For example, a packet might travel through two different ISPs before it reaches the destination AS.

BGP makes routing decisions based on the series of autonomous systems that packets travel through. Each AS likely consists of multiple routers.

### Route selection

Route selection can also refer to selecting which routes populate the routing table. Routes learned through a dynamic routing protocol aren't automatically inserted into the routing table. Manually configured static routes don't necessarily enter the routing table either. A router can't use a route to forward packets unless the route is in the routing table.

The term **route selection** has two meanings:

| Meaning                  | Process                                                                      |
| :----------------------- | :--------------------------------------------------------------------------- |
| Routing table population | Selecting which routes the router enters into its routing table              |
| Packet forwarding        | Selecting the best route in the routing table to forward a particular packet |

#### The metric parameter

Each routing protocol calculates its metric differently. EIGRP uses bandwidth and **delay**, which is how long it takes bits to travel across a link.

Table 17.1 IGP metrics:

| IGP   | Metric                              | Description                                                                                                          |
| :---- | :---------------------------------- | :------------------------------------------------------------------------------------------------------------------- |
| RIP   | Hop count                           | Each router in the path counts as one hop. The total metric is the total number of hops to the destination.          |
| EIGRP | Metric based on bandwidth and delay | A complex formula that can take into account many values. By default, bandwidth and delay are used.                  |
| OSPF  | Cost                                | The cost of each link is calculated based on bandwidth. A route's metric is the total cost of each link in the path. |

In the routing table, two values in square brackets follow each route's destination network:

```text
R1# show ip route
. . .
O     192.168.3.0/24 [110/2] via 192.168.1.6, 00:08:35, GigabitEthernet0/0
```

| Value in `[110/2]` | Meaning                                                          |
| :----------------- | :--------------------------------------------------------------- |
| `110`              | The route's administrative distance, covered in the next section |
| `2`                | The route's metric                                               |

If the best route becomes invalid, the router inserts the alternate route into the routing table. In the following example, disabling R1's G0/0 interface makes the route through R3 invalid. When you check the routing table again, the alternate route through R2 replaces it, with a metric of `4`:

```text
R1(config)# interface g0/0
R1(config-if)# shutdown
R1(config-if)# do show ip route
. . .
O     192.168.3.0/24 [110/4] via 192.168.1.2, 00:00:04, GigabitEthernet0/1
```

#### The administrative distance parameter

Sometimes a router runs multiple routing protocols. For example, two enterprises running different routing protocols might connect their networks so they can communicate.

If a router learns multiple routes to the same destination network from different routing protocols, it needs something other than the metric to select which route enters the routing table. **Administrative distance (AD)** fills that role. AD indicates how preferred a routing protocol is. A lower AD means IOS considers the protocol more "trustworthy," or more likely to select good routes.

| Value  | Compares routes learned from |
| :----- | :--------------------------- |
| Metric | The same routing protocol    |
| AD     | Different routing protocols  |

Table 17.2 Default AD values:

| Route type/protocol | Default AD |
| :------------------ | :--------- |
| Connected           | 0          |
| Static              | 1          |
| External BGP (EBGP) | 20         |
| EIGRP               | 90         |
| OSPF                | 110        |
| IS-IS               | 115        |
| RIP                 | 120        |
| Internal BGP (IBGP) | 200        |
| Unusable route      | 255        |

The least-preferred AD value is 255. Cisco routers can use this value to mark a route unusable, and the router removes the route from the routing table.

EIGRP metrics tend to be much larger than RIP and OSPF metrics because of the formula EIGRP uses. EIGRP metrics in the tens or hundreds of thousands aren't rare.

The following example shows the EIGRP route in R1's routing table. In the square brackets, EIGRP's AD is `90` and the route's metric is `3584`:

```text
R1# show ip route
. . .
D        10.0.0.0 [90/3584] via 192.168.12.2, 00:33:30, GigabitEthernet0/0
. . .
```

Metrics and AD only compare routes to the same destination, meaning the same destination network address with the same prefix length.

#### ECMP

Metrics and AD select among routes to the same destination:

| Routes to the same destination learned from | Router compares |
| :------------------------------------------ | :-------------- |
| The same routing protocol                   | Metric          |
| Different routing protocols                 | AD              |

Suppose a router learns multiple routes to the same destination from the same routing protocol, and the routes have the same metric. The router adds all the routes to the routing table and load-balances traffic over them. With two routes, the router sends half the traffic over one route and the other half over the other route. This is called **equal-cost multi-path (ECMP)** routing.

By default, ECMP routing can use a maximum of four routes to the same destination.

#### Floating static routes

Sometimes you want a static route as a backup that enters the routing table only if the main route, learned through a routing protocol, is lost. Floating static routes fill this role.

A **floating static route** is a static route configured with an AD greater than the default of 1, which makes it less preferred. For example, to make a static route less preferred than an OSPF route to the same destination, configure it with an AD greater than 110, OSPF's AD.

To configure a floating static route, add the AD value to the end of the command. For example, configure a floating recursive static route with the `ip route destination-network netmask next-hop ad` command:

```text
R1(config)# ip route 10.0.0.0 255.255.255.0 192.168.1.2 111
```

A floating static route can provide a backup path through a router that the local router doesn't exchange routing information with.

In the following example, the OSPF route is in R1's routing table at first. After R1's G0/1 interface is disabled to simulate a hardware failure, the floating static route with an AD of `111` replaces the OSPF route:

```text
R1(config)# do show ip route
. . .
O        10.0.0.0 [110/4] via 192.168.1.6, 00:25:43, GigabitEthernet0/1
. . .
R1(config)# interface g0/1
R1(config-if)# shutdown
R1(config-if)# do show ip route
. . .
S        10.0.0.0 [111/0] via 192.168.1.2
. . .
```

#### Route selection examples

#### Routing table population

In this example, R1 learns the following routes through manual configuration and dynamic routing protocols:

| Route | Destination    | Learned from   | Metric |
| :---- | :------------- | :------------- | :----- |
| A     | 203.0.113.0/24 | Static routing |        |
| B     | 203.0.113.0/25 | RIP            | 4      |
| C     | 203.0.113.0/26 | EIGRP          | 5678   |
| D     | 203.0.113.0/27 | OSPF           | 10     |

R1 inserts all four routes into the routing table because all four routes are to different destinations. They share the destination network address 203.0.113.0, but their prefix lengths differ, so they're different destinations. R1 doesn't need to compare them.

The "most specific matching route" rule is irrelevant here. The rule selects which route in the routing table forwards a packet. It doesn't select which routes enter the routing table.

Which of the two routes to 10.0.0.0/16 does R1 prefer? Both routes are learned from the same routing protocol, so R1 compares their metrics. Route D has the lower metric of the two, so R1 selects it.

##### Packet forwarding

The second aspect of route selection is packet forwarding: selecting which route in the routing table forwards a particular packet. This process is simpler because it has only one consideration, the most specific matching route. The router doesn't consider AD or metric values when forwarding a packet.

```text
R1# show ip route
. . .
S        203.0.113.0/24 [1/0] via 192.168.1.2
R        203.0.113.0/25 [120/4] via 192.168.1.6
D        203.0.113.0/26 [90/5678] via 192.168.1.10
O        203.0.113.0/27 [110/10] via 192.168.1.14
```

If the packet's destination IP address is part of the network in the route, the route is a match. The following example writes out each route and the packet's destination IP address, 203.0.113.65, in binary:

```text
203.0.113.0/24 = 11001011.00000000.01110001.00000000
203.0.113.0/25 = 11001011.00000000.01110001.00000000
203.0.113.0/26 = 11001011.00000000.01110001.00000000
203.0.113.0/27 = 11001011.00000000.01110001.00000000
203.0.113.65   = 11001011.00000000.01110001.01000001
```

{{< admonition "Exam tip" tip >}}
You don't have to write out all 32 bits of every address. In this example, the shortest prefix length is /24, and the four routes and the destination IP address all begin with 203.0.113. Write out only the relevant octets, which is the final octet in this case.
{{< /admonition >}}

The route to 203.0.113.0/26 doesn't match the packet's destination because the 26th bit doesn't match.

The two aspects of route selection:

| Aspect                   | How the router selects a route                                                                                                                                                                     |
| :----------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Routing table population | **Metric:** selects among routes to the same destination learned from the same routing protocol.<br>**AD:** selects among routes to the same destination learned from different routing protocols. |
| Packet forwarding        | Selects the most specific matching route.                                                                                                                                                          |

### The network command

You configure RIP, EIGRP, and OSPF by activating the protocol on one or more of the router's interfaces. The router then advertises the interface's network prefix, the network address and netmask.

All three protocols share one command: the `network` command. This command tells the router to do the following:

1. Look for interfaces with an IP address in the specified range.
2. Activate the routing protocol on those interfaces.
3. Advertise the network prefix of those interfaces to its neighbors.

To configure OSPF on a Cisco router, use the `router ospf process-id` command in global config mode. This command enters **router config mode**, where you can use the `network` command. A router can run separate OSPF processes, or instances, so you must specify a `process-id`.

The `network` command activates OSPF on a router's interfaces. Its syntax is `network ip-address wildcard-mask area area-id`. The key to this command is the **wildcard mask**, which looks like an inverted netmask but serves a different purpose.

OSPF uses areas to logically divide the network.

```text
R1(config)# router ospf 1
R1(config-router)# network 192.168.1.0 0.0.0.3 area 0
R1(config-router)# network 192.168.1.4 0.0.0.3 area 0
R1(config-router)# network 192.168.2.0 0.0.0.255 area 0
```

| Command                                | Activates OSPF on                 |
| :------------------------------------- | :-------------------------------- |
| `network 192.168.1.0 0.0.0.3 area 0`   | G0/0 (192.168.1.0/30, link to R2) |
| `network 192.168.1.4 0.0.0.3 area 0`   | G0/1 (192.168.1.4/30, link to R3) |
| `network 192.168.2.0 0.0.0.255 area 0` | G0/2 (192.168.2.0/24)             |

A wildcard mask is a series of 32 bits, like a netmask. It indicates which bits must match between two IP addresses and which bits don't. In the `network` command, the wildcard mask compares the IP address in the command with the IP address on each router interface:

| Wildcard mask bit | Bits in the same position of the two IP addresses |
| :---------------- | :------------------------------------------------ |
| `0`               | Must match                                        |
| `1`               | Don't have to match                               |

Table 17.3 /24+ netmasks and wildcard masks:

| Prefix length | Netmask         | Wildcard mask (last octet binary) |
| :------------ | :-------------- | :-------------------------------- |
| /24           | 255.255.255.0   | 0.0.0.255 (11111111)              |
| /25           | 255.255.255.128 | 0.0.0.127 (01111111)              |
| /26           | 255.255.255.192 | 0.0.0.63 (00111111)               |
| /27           | 255.255.255.224 | 0.0.0.31 (00011111)               |
| /28           | 255.255.255.240 | 0.0.0.15 (00001111)               |
| /29           | 255.255.255.248 | 0.0.0.7 (00000111)                |
| /30           | 255.255.255.252 | 0.0.0.3 (00000011)                |
| /31           | 255.255.255.254 | 0.0.0.1 (00000001)                |
| /32           | 255.255.255.255 | 0.0.0.0 (00000000)                |

{{< admonition "Why wildcard masks?" note >}}
Why use wildcard masks instead of netmasks? Recall what netmasks do: they identify the size of a network prefix by separating the network portion of an IP address from the host portion. For example, <code>ip address 192.168.1.1 255.255.255.0</code> on a router interface tells the router that its IP address is 192.168.1.1. The first three octets are the network portion, the last octet is the host portion, and the prefix is 192.168.1.0/24.<br><br>
The OSPF <code>network</code> command uses the IP address and wildcard mask differently. They don't define a network prefix. Instead, they specify a range of IP addresses that aren't necessarily part of the same subnet. The router uses this range to determine which interfaces take part in OSPF, meaning which interfaces send and receive OSPF routing information.<br><br>
<strong>Recap:</strong><br>
<strong>Netmask (subnet mask):</strong> distinguishes the network and host portions of an IP address. It determines the length of a subnet's network prefix.<br>
<strong>IP address and wildcard mask in the OSPF <code>network</code> command:</strong> define a range of IP addresses, not a network prefix. The range determines which router interfaces participate in the OSPF process.
{{< /admonition >}}

The `network` command is flexible. OSPF activates on an interface as long as the bits indicated by a `0` in the wildcard mask match between the IP address in the `network` command and the interface's IP address.

```text
R1(config)# router ospf 1
R1(config-router)# network 192.168.1.0 0.0.0.7 area 0
R1(config-router)# network 192.168.2.1 0.0.0.0 area 0
```

| Command                              | Activates OSPF on |
| :----------------------------------- | :---------------- |
| `network 192.168.1.0 0.0.0.7 area 0` | G0/0 and G0/1     |
| `network 192.168.2.1 0.0.0.0 area 0` | G0/2              |

The first command activates OSPF on both G0/0 and G0/1 because the appropriate bits of each interface's IP address match the IP address in the `network` command.

The second command activates OSPF on G0/2 differently than the previous example did. It specifies G0/2's exact IP address with a wildcard mask of `0.0.0.0`, which is equivalent to a /32 netmask (255.255.255.255). This tells R1 to activate OSPF only on the interface with IP address 192.168.2.1, which is R1's G0/2 interface.

| Method | Command | Use |
| :----- | :------ | :-- |
| Exact interface IP address | `network ip-address 0.0.0.0 area area-id` | The recommended method |
| All interfaces | `network 0.0.0.0 255.255.255.255 area 0` | A shortcut that's handy for quickly activating OSPF in a lab, but not recommended in a real network. A wildcard mask of 255.255.255.255 is equivalent to a netmask of 0.0.0.0 and matches all possible IP addresses. |

{{< admonition "Network command misconceptions" warning >}}
Many students have two major misconceptions about the OSPF <code>network</code> command.<br><br>
<strong>Misconception 1: the wildcard mask must match the interface's netmask.</strong> It doesn't, as the previous examples show. OSPF activates on an interface as long as the bits specified by the wildcard mask match between the IP address in the <code>network</code> command and the interface's IP address.<br><br>
<strong>Misconception 2: the <code>network</code> command specifies which networks OSPF advertises.</strong> It doesn't. It specifies which interfaces OSPF activates on. The router then advertises each interface's network prefix.
{{< /admonition >}}

To see how the second misconception plays out, look at what the two commands in the previous example do. Here is the first command:

```text
R1(config-router)# network 192.168.1.0 0.0.0.7 area 0
```

This command tells R1 to do the following:

1. Look for interfaces with an IP address in the 192.168.1.0 to 192.168.1.7 range. R1's G0/0 and G0/1 IP addresses are in this range.
2. Activate OSPF on those interfaces.
3. Advertise the network prefix of those interfaces to neighbors. R1 advertises G0/0's prefix of 192.168.1.0/30 and G0/1's prefix of 192.168.1.4/30.

Here is the second command, which activates OSPF on R1's G0/2 interface:

```text
R1(config-router)# network 192.168.2.1 0.0.0.0 area 0
```

This command doesn't tell R1 to advertise 192.168.2.1/32. It tells R1 to activate OSPF on the interface with IP address 192.168.2.1 (G0/2) and advertise that interface's network prefix, 192.168.2.0/24.
