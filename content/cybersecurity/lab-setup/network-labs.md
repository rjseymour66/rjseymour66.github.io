+++
title = 'Network Labs'
date = '2026-05-17T08:26:00-04:00'
weight = 50
draft = false
+++

Each lesson uses a unique topology that you build from scratch in GNS3. All routers run
VyOS. Switches are Open vSwitch appliances (where STP or SSH access is required) or the
built-in GNS3 Ethernet Switch (for simple forwarding). Hosts run Alpine Linux.

---

## Lesson 1: Multi-router OSPF with route redistribution

### Problem

You manage three routers in a triangle, each serving its own LAN segment. The routers
need to know about each other's networks. You could add static routes, but every time the
topology changes you would have to update every router manually. You also need all routers
to reach the internet through R1, without manually configuring a default route on R2 and
R3.

### Goal

Run OSPF area 0 across all three routers so they share LAN routes automatically. Then
redistribute R1's static default route into OSPF so R2 and R3 learn it without any manual
configuration on their part.

### Topology

Drag the following into a new GNS3 project:

- R1, R2, R3 — three VyOS routers
- PC1, PC2, PC3 — three Alpine Linux hosts, one attached to each router's LAN
- External — one Alpine Linux host representing an internet server

Wire them as follows:

- R1 eth1 to R2 eth1 — point-to-point link, subnet 10.0.12.0/30 (R1 at .1, R2 at .2)
- R1 eth2 to R3 eth1 — point-to-point link, subnet 10.0.13.0/30 (R1 at .1, R3 at .2)
- R2 eth2 to R3 eth2 — point-to-point link, subnet 10.0.23.0/30 (R2 at .1, R3 at .2)
- R1 eth0 to PC1 — LAN subnet 10.0.10.0/24 (R1 at .1, PC1 at .10)
- R2 eth0 to PC2 — LAN subnet 10.0.20.0/24 (R2 at .1, PC2 at .10)
- R3 eth0 to PC3 — LAN subnet 10.0.30.0/24 (R3 at .1, PC3 at .10)
- R1 eth3 to External — subnet 203.0.113.0/30 (R1 at .1, External at .2)

Configure all interface addresses and host default gateways before starting the lesson.
Do not configure any routing yet.

### Success criteria

- Every router shows the other two as OSPF neighbors in FULL state.
- R2 and R3's routing tables contain all three LAN subnets, learned through OSPF — not
  added manually.
- The default route appears in R2 and R3's tables as an OSPF external route.
- PC2 and PC3 can reach the External host with no static routes on R2 or R3.

---

## Lesson 2: eBGP peering between autonomous systems

### Problem

R1 and R2 are administered by different organizations. Each controls its own routing
policy and decides independently which prefixes to share with the other. OSPF is not
appropriate here — it shares full topology information and assumes a single administrative
domain. You need a protocol that lets each router advertise only its own prefix and apply
policy at the boundary. This is how internet routing works at every ISP handoff point.

### Goal

Establish an eBGP session between R1 and R2. Each router advertises its LAN prefix and
learns the other's prefix through BGP, with no static routes configured on either router.

### Topology

Drag the following into a new GNS3 project:

- R1 — VyOS router, AS 65001
- R2 — VyOS router, AS 65002
- PC1 — Alpine Linux host on R1's LAN
- PC2 — Alpine Linux host on R2's LAN

Wire them as follows:

- R1 eth0 to R2 eth0 — WAN link, subnet 10.0.0.0/30 (R1 at .1, R2 at .2)
- R1 eth1 to PC1 — LAN subnet 10.1.0.0/24 (R1 at .1, PC1 at .10)
- R2 eth1 to PC2 — LAN subnet 10.2.0.0/24 (R2 at .1, PC2 at .10)

Configure all interface addresses and host gateways. Do not configure any routing yet.

### Success criteria

- The BGP session between R1 and R2 shows Established state on both sides.
- R1's routing table shows 10.2.0.0/24 learned through BGP. R2's routing table shows
  10.1.0.0/24 learned through BGP.
- PC1 can ping PC2. No static routes exist on either router.

---

## Lesson 3: VLAN trunking and Spanning Tree Protocol

### Problem

Two trunk links connect SW1 and SW2, creating a Layer 2 loop. Without intervention, a
broadcast frame would circulate endlessly between the two switches — every device on the
network would receive millions of copies per second, saturating all links instantly. STP
prevents this by putting one of the redundant ports into a blocking state. The challenge
is understanding which port STP chose to block and why, then taking control of that
decision by manipulating bridge priority.

### Goal

Observe STP's initial port state decisions after enabling it on both switches. Then lower
SW1's bridge priority so it wins the root bridge election and controls which port enters
the blocking state. Verify that the standby link takes over when you shut down the active
trunk.

### Topology

Drag the following into a new GNS3 project:

- SW1 — Open vSwitch appliance (not the built-in GNS3 switch)
- SW2 — Open vSwitch appliance
- PC1 — Alpine Linux host
- PC2 — Alpine Linux host

Wire them as follows:

- SW1 eth0 to SW2 eth0 — first trunk link, carrying VLANs 10 and 20
- SW1 eth1 to SW2 eth1 — second trunk link, carrying VLANs 10 and 20
- SW1 eth2 to PC1 — access port, VLAN 10 (PC1 at 10.0.10.10/24)
- SW2 eth2 to PC2 — access port, VLAN 20 (PC2 at 10.0.20.10/24)

There is no router in this topology. Configure trunk and access port roles on both
switches, then enable STP on both before starting the lesson.

### Success criteria

- One trunk port on either switch shows in blocking state in STP output.
- PC1 can reach PC2 through the forwarding trunk.
- After you lower SW1's priority, STP reconverges with SW1 as root and the expected port
  blocking.
- After shutting down the active trunk, STP converges and traffic resumes through the
  previously blocked link within 30 seconds.

---

## Lesson 4: VRRP gateway redundancy

### Problem

Both hosts point to a single gateway address. If the router holding that address fails,
all hosts lose connectivity until someone manually redirects them or the router recovers.
You need two routers to share a single virtual IP so that failover is automatic and
transparent to hosts.

### Goal

Configure VRRP group 1 on both routers sharing the virtual IP 10.0.10.1. R1 should hold
MASTER state based on a higher priority. Verify automatic failover by stopping R1 and
confirming R2 takes over and hosts continue working.

### Topology

Drag the following into a new GNS3 project:

- R1 — VyOS router (will be VRRP master)
- R2 — VyOS router (will be VRRP backup)
- SW1 — GNS3 built-in Ethernet Switch
- PC1, PC2 — Alpine Linux hosts
- NAT cloud — for external connectivity

Wire them as follows:

- R1 eth0 to SW1
- R2 eth0 to SW1
- SW1 to PC1
- SW1 to PC2
- R1 eth1 to NAT cloud
- R2 eth1 to NAT cloud

IP addressing:

- R1 real LAN IP: 10.0.10.2/24
- R2 real LAN IP: 10.0.10.3/24
- VRRP virtual IP: 10.0.10.1 — the address both hosts use as their gateway
- PC1: 10.0.10.10/24, gateway 10.0.10.1
- PC2: 10.0.10.20/24, gateway 10.0.10.1

Configure interface addresses on R1 and R2. Set PC1 and PC2 to use 10.0.10.1 as their
gateway before VRRP is configured — pings from the hosts will fail until VRRP is up, and
that failure is the starting condition for this lesson.

### Success criteria

- R1 shows MASTER state. R2 shows BACKUP state. The virtual IP responds to ARP from PC1.
- PC1 can reach an external address through 10.0.10.1.
- After stopping R1, PC1's connectivity recovers within 3 seconds as R2 takes over.
- After restarting R1, it reclaims MASTER state because of its higher priority.

---

## Lesson 5: NAT and PAT on a border router

### Problem

PC1 and PC2 have RFC 1918 addresses that are not routable on the external network.
External cannot route packets back to 10.0.0.x. Additionally, a service on PC1 must be
reachable from External, even though PC1 has no public address of its own.

### Goal

Configure PAT (masquerade) on R1 so internal hosts appear to come from R1's external IP
when reaching External. Then configure a DNAT rule forwarding inbound TCP on port 8080
to PC1.

### Topology

Drag the following into a new GNS3 project:

- R1 — VyOS router (the edge/border router)
- SW1 — GNS3 built-in Ethernet Switch
- PC1, PC2 — Alpine Linux hosts (internal)
- External — Alpine Linux host (simulates an internet server)

Wire them as follows:

- R1 eth0 to External — outside subnet 203.0.113.0/30 (R1 at .1, External at .2)
- R1 eth1 to SW1
- SW1 to PC1
- SW1 to PC2

IP addressing:

- External: 203.0.113.2/30
- R1 outside: 203.0.113.1/30
- R1 inside: 10.0.0.1/24
- PC1: 10.0.0.10/24, gateway 10.0.0.1
- PC2: 10.0.0.20/24, gateway 10.0.0.1

Before starting, configure all addresses. Start a simple HTTP service on PC1 on port
8080. Add a static route on External pointing 10.0.0.0/24 toward 203.0.113.1 — this
simulates R1's external IP being reachable from the internet.

### Success criteria

- PC1 and PC2 can both reach External. External sees traffic arriving from 203.0.113.1,
  not from 10.0.0.x.
- The NAT translation table on R1 shows active entries for both hosts during traffic.
- External can connect to the service on PC1 by reaching 203.0.113.1 on port 8080.

---

## Lesson 6: Policy-based routing on a dual-homed host

### Problem

PC1 connects to two different ISPs through two different interfaces. Linux has one routing
table and one default route, so all traffic exits through a single interface regardless of
which source IP it uses. When a session is initiated from `eth0`'s address but returns
through `eth1`, the routing is asymmetric and the connection breaks. You need each source
address to consistently use its own interface for all traffic.

### Goal

Configure Linux policy routing on PC1 using two additional routing tables — one per
interface — and source-based routing rules that direct traffic by source IP to the correct
table.

### Topology

Drag the following into a new GNS3 project:

- R1 — VyOS router (ISP A)
- R2 — VyOS router (ISP B)
- PC1 — Alpine Linux host with two network interfaces

Wire them as follows:

- R1 eth0 to PC1 eth0 — subnet 10.0.10.0/24 (R1 at .1, PC1 at .10)
- R2 eth0 to PC1 eth1 — subnet 10.0.20.0/24 (R2 at .1, PC1 at .10)
- R1 eth1 to a NAT cloud (R1's upstream)
- R2 eth1 to the same NAT cloud (R2's upstream)

Configure addresses on all interfaces. Both routers need a default route toward the NAT
cloud. PC1 gets an IP on each interface but no default route — that is part of the lesson.

### Success criteria

- Traffic sourced from PC1's `eth0` address exits through `eth0` and reaches R1.
- Traffic sourced from PC1's `eth1` address exits through `eth1` and reaches R2.
- Packet captures on R1 and R2 confirm each router sees only traffic with the expected
  source address — no cross-interface leakage.

---

## Lesson 7: IPsec site-to-site VPN

### Problem

Site A (10.1.0.0/24) and Site B (10.2.0.0/24) need to communicate over a WAN link you
do not control. Anyone monitoring the WAN could read or modify the traffic in transit. You
need to encrypt all traffic between the two sites end-to-end so it is unreadable to any
observer on the WAN.

### Goal

Configure an IKEv2 IPsec tunnel between R1 and R2 using a pre-shared key. All traffic
between the two LAN subnets should travel through the encrypted tunnel automatically
without any configuration on the hosts.

### Topology

Drag the following into a new GNS3 project:

- R1 — VyOS router (Site A gateway)
- R2 — VyOS router (Site B gateway)
- PC-A — Alpine Linux host at Site A
- PC-B — Alpine Linux host at Site B

Wire them as follows:

- R1 eth0 to R2 eth0 — WAN link, subnet 203.0.113.0/30 (R1 at .1, R2 at .2)
- R1 eth1 to PC-A — LAN subnet 10.1.0.0/24 (R1 at .1, PC-A at .10)
- R2 eth1 to PC-B — LAN subnet 10.2.0.0/24 (R2 at .1, PC-B at .10)

Configure all interface addresses and host gateways. Verify R1 can ping R2's WAN IP
before starting. Do not add any routes between the LAN subnets — those should only work
once the IPsec tunnel is up.

### Success criteria

- The IPsec security association shows as active on both routers with non-zero packet and
  byte counts.
- PC-A can ping PC-B.
- A packet capture on the WAN interface between R1 and R2 shows ESP packets — not
  plaintext ICMP. You should see no readable content from the host traffic in the
  captured frames.

---

## Lesson 8: Traffic shaping and QoS

### Problem

PC1 runs a latency-sensitive application — late packets are unusable, even if they
eventually arrive. PC2 runs bulk transfers that consume all available bandwidth when
running. On a first-come-first-served link, PC2's bulk traffic queues in front of PC1's
packets and causes unacceptable delay. Without intervention, the link has no concept of
traffic priority.

### Goal

Apply a VyOS traffic-policy shaper on R1's WAN-facing interface that caps total
throughput at 2 Mbit/s, guarantees PC1's traffic at least 512 Kbit/s by matching its
DSCP marking (EF), and gives all remaining bandwidth to bulk traffic.

### Topology

Drag the following into a new GNS3 project:

- R1 — VyOS router (the router with the bottleneck WAN link)
- R2 — VyOS router (upstream)
- PC1 — Alpine Linux host (simulates a latency-sensitive application such as VoIP)
- PC2 — Alpine Linux host (simulates bulk file transfer)
- Server — Alpine Linux host (traffic destination)

Wire them as follows:

- PC1 eth0 to R1 eth0 — subnet 10.0.1.0/24 (R1 at .1, PC1 at .10)
- PC2 eth0 to R1 eth1 — subnet 10.0.2.0/24 (R1 at .1, PC2 at .10)
- R1 eth2 to R2 eth0 — WAN link, subnet 10.0.0.0/30 (R1 at .1, R2 at .2)
- R2 eth1 to Server — subnet 10.0.3.0/24 (R2 at .1, Server at .10)

In GNS3, right-click the link between R1 and R2 and set the bandwidth limit to 2 Mbit/s.
Configure static routes on R1 and R2 so all subnets are reachable. Install `iperf3` on
PC2 and Server for bulk traffic generation.

### Success criteria

- Measure PC1's round-trip latency while PC2 runs a bulk transfer without shaping
  applied. Latency should exceed 200ms.
- Apply the shaper and repeat the measurement. PC1's latency should stay below 50ms
  during the same bulk transfer.
- Total throughput on the WAN link does not exceed 2 Mbit/s.

---

## Lesson 9: Network segmentation with a DMZ

### Problem

Web must be reachable from External on port 80, but it must not be able to reach inside
hosts. If Web is compromised, the attacker should have no path into the internal network.
Inside hosts need to reach both Web and External freely. Currently, all three segments
are fully open to each other.

### Goal

Implement a three-zone security policy on R1 using VyOS zone-policy: outside can only
reach Web on port 80; Web cannot initiate any connection to the inside; inside has
unrestricted outbound access to both the DMZ and the outside.

### Topology

Drag the following into a new GNS3 project:

- R1 — VyOS router with three interfaces (acts as a three-legged firewall)
- Web — Alpine Linux host (DMZ web server)
- SW1 — GNS3 built-in Ethernet Switch (inside LAN)
- PC1, PC2 — Alpine Linux hosts (inside)
- External — Alpine Linux host (simulates an internet client)

Wire them as follows:

- R1 eth0 to External — outside subnet 203.0.113.0/30 (R1 at .1, External at .2)
- R1 eth1 to Web — DMZ subnet 10.0.2.0/24 (R1 at .1, Web at .10)
- R1 eth2 to SW1 — inside subnet 10.0.1.0/24 (R1 at .1)
- SW1 to PC1 (10.0.1.10/24) and PC2 (10.0.1.20/24)

Configure all interface addresses and host gateways. Start a simple web service on Web
on port 80. Temporarily allow all traffic and verify that every host can reach every
other host before adding any zone policy — that verified state is your baseline.

### Success criteria

- External can reach the web service on Web at port 80. It cannot connect to any service
  on PC1 or PC2.
- Web cannot ping or connect to any inside host.
- PC1 and PC2 can reach Web on port 80 and External without restriction.

---

## Lesson 10: Multi-layer troubleshooting scenario

### Problem

PC1 cannot reach PC2. You do not know which layer is broken, or how many layers are
broken simultaneously. You must work from the bottom of the OSI model upward — forming a
hypothesis at each layer based on what you observe, confirming it with a test, fixing it,
then moving to the next layer. The temptation is to guess and try random fixes. Resist it.

### Goal

Restore full end-to-end connectivity from PC1 to PC2 by diagnosing and fixing all four
faults in order from L1 to L4. Document each fault as you find it.

### Topology

Drag the following into a new GNS3 project:

- R1, R2 — two VyOS routers
- SW1, SW2 — two Open vSwitch appliances
- PC1 — Alpine Linux host (left end)
- PC2 — Alpine Linux host (right end)

Wire them left to right:

- PC1 eth0 to SW1 — access port, VLAN 10
- SW1 eth0 to R1 eth0 — trunk carrying VLAN 10
- R1 eth1 to R2 eth0 — point-to-point WAN link
- R2 eth1 to SW2 eth0 — trunk carrying VLAN 20
- SW2 eth1 to PC2 — access port, VLAN 20

IP addressing:

- PC1: 10.0.10.10/24, gateway 10.0.10.1
- R1 VLAN 10 subinterface: 10.0.10.1/24
- R1 WAN interface: 10.0.0.1/29
- R2 WAN interface: 10.0.0.2/30 (intentional mask mismatch with R1)
- R2 VLAN 20 subinterface: 10.0.20.1/24
- PC2: 10.0.20.10/24, gateway 10.0.20.1

Build and fully verify the topology first — confirm PC1 can ping PC2. Then inject the
faults below and close all consoles before starting the lesson.

#### Faults to inject

| Layer | What to change |
|-------|----------------|
| L1 | Move the cable from R1 to R2's eth0 to R2's eth2 instead |
| L2 | Change SW1's port facing PC1 from VLAN 10 to VLAN 20 |
| L3 | Change R2's WAN interface mask from /29 to /30 |
| L4 | Add a firewall rule on R2 dropping all inbound ICMP |

### Success criteria

- PC1 can ping PC2.
- You have a written record — one paragraph per fault — describing the symptom you
  observed, the diagnostic step that confirmed the hypothesis, and the change you made.
- A Bash script captures interface state, routing tables, and firewall rules from all
  devices into a timestamped log file that you ran before and after your fixes.
