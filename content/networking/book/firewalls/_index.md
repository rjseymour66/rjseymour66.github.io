+++
title = 'Firewalls'
date = '2026-06-13T13:56:15-04:00'
weight = 40
draft = false
+++

Linux implements firewalling inside the kernel through *Netfilter*, a framework of hooks embedded in the network stack. Every packet that enters, exits, or passes through the machine triggers these hooks, giving the kernel the opportunity to inspect, accept, drop, or modify it. Netfilter is the engine: it does the actual work of evaluating rules against packets.

Two user-space tools write rules into Netfilter: *iptables* and its successor *nftables*. Both configure the same underlying framework, but they use different kernel interfaces. iptables uses the `x_tables` kernel module and requires a separate binary for each protocol family: `iptables` for IPv4, `ip6tables` for IPv6, `arptables` for ARP, and `ebtables` for Ethernet bridging. nftables uses the `nf_tables` module and a single `nft` binary that handles all protocol families with one consistent syntax.

On Debian 10, RHEL 8, and Ubuntu 20.04 and later, the `iptables` command is a compatibility shim called `iptables-nft`. It accepts familiar iptables syntax and translates it into nftables rules. The distinction matters for troubleshooting: rules written with `iptables` and rules written with `nft` appear in the same kernel table but may look different depending on which tool you use to view them.

Higher-level frontends such as `ufw` on Ubuntu and `firewalld` on RHEL and Fedora sit on top of these tools and provide simpler interfaces for common configurations. Underneath, they all write rules into Netfilter.

## Connection tracking

Netfilter becomes a *stateful* firewall through the *connection tracking* module (`conntrack`). Rather than evaluating each packet in isolation, conntrack assigns every packet a state label:

| State         | Meaning                                                                                                                     |
| :------------ | :-------------------------------------------------------------------------------------------------------------------------- |
| `NEW`         | First packet of a new connection.                                                                                           |
| `ESTABLISHED` | Packet belongs to a connection that has seen traffic in both directions.                                                    |
| `RELATED`     | Packet is associated with an existing connection (for example, an FTP data channel opened by a tracked control connection). |
| `INVALID`     | Packet does not match any known connection and is not a valid new one.                                                      |

A rule that allows `ESTABLISHED` and `RELATED` traffic inbound lets return packets through without opening every high-numbered ephemeral port. This is the pattern most host firewalls use: default DROP on INPUT, allow outbound, and allow established and related traffic back in.

## Host firewall vs. inter-VLAN firewall

A *host firewall* protects a single machine. Its rules govern what traffic the machine itself accepts or sends. The `INPUT` and `OUTPUT` chains handle this. A typical host firewall allows a handful of inbound services (SSH on port 22, HTTPS on port 443) and drops everything else. Each host manages its own rules, so policy must be applied individually: by hand or through configuration management tools like Ansible or Puppet.

An *inter-VLAN firewall* sits at the Layer 3 boundary between network segments. Traffic flowing from one VLAN to another must pass through a router or Layer 3 switch. When that device runs Linux, the `FORWARD` chain processes all routed traffic. A single set of rules controls which hosts in VLAN 10 can reach hosts in VLAN 20, which protocols are permitted, and which are blocked. This applies regardless of what firewall rules run on the individual hosts behind it.

The key distinction is scope:

|                 | Host firewall          | Inter-VLAN firewall                   |
| :-------------- | :--------------------- | :------------------------------------ |
| Protects        | One machine            | All hosts behind a network boundary   |
| Relevant chain  | `INPUT` / `OUTPUT`     | `FORWARD`                             |
| Policy location | Each host individually | Centralized on the router or firewall |
| Failure impact  | One host exposed       | Entire segment exposed                |

A defense-in-depth architecture uses both: an internal segmentation firewall enforces inter-VLAN policy, and host firewalls provide a second layer if a device on the same segment is compromised.

## Firewall types

Firewalls vary in how deeply they inspect traffic. Linux Netfilter is a stateful packet filter: it inspects packet headers and connection state but does not analyze application-layer content. The four common types are:

| Type                            | Inspection depth                                                                                                                                  |
| :------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------ |
| Stateless packet filter         | Header fields only (source IP, destination IP, port, protocol). No connection state.                                                              |
| Stateful firewall               | Header fields plus connection state. Understands whether a packet belongs to an established session.                                              |
| Next-Generation Firewall (NGFW) | Stateful inspection plus application-layer identification, inline IPS, and threat intelligence. Can identify protocols regardless of port number. |
| Web Application Firewall (WAF)  | HTTP/HTTPS only. Inspects request content for injection attacks, XSS, and OWASP Top 10 threats.                                                   |

An NGFW can identify that traffic on port 443 is not HTTPS but a tunneled protocol and block it by application regardless of port. A stateful firewall sees only that port 443 is permitted.



## Order of operations

Netfilter processes each packet through tables and chains in a fixed sequence. Understanding this sequence determines where a rule must be placed to have the intended effect. Rules in the wrong table or chain run too early, too late, or never at all.

The packet flow through the tables, from arrival to delivery:

| Stage            | Tables applied (in order)           |
| :--------------- | :---------------------------------- |
| PREROUTING       | raw → mangle → nat                  |
| Routing decision | (kernel selects outbound interface) |
| INPUT            | mangle → filter                     |
| FORWARD          | mangle → filter                     |
| OUTPUT           | raw → mangle → nat → filter         |
| POSTROUTING      | mangle → nat                        |

A full diagram of the flow is available at [Phill writes stuff](https://stuffphilwrites.com/fw-ids-iptables-flowchart-v2024-05-22/).

### Encrypted traffic

A VPN or IPsec policy needs a *match list* that defines which traffic to encrypt: for example, all packets from `192.168.1.0/24` destined for `10.0.0.0/8`. This classification must happen before the nat table processes the packet. If DNAT runs first in PREROUTING, the destination address is rewritten before the VPN policy sees it, and the match fails. By marking packets in the mangle table (which runs before nat in PREROUTING), you preserve the original addresses at the point where the encryption decision is made.

### Policy-based routing

Policy-based routing selects the outbound link based on traffic characteristics rather than destination alone. You might route backup traffic over a link with lower per-packet cost, and route latency-sensitive traffic over a faster link with better characteristics. The routing decision happens after PREROUTING but before FORWARD or OUTPUT. This means traffic must be classified and marked in mangle PREROUTING, before the nat table runs, so the routing decision can act on the original source and destination addresses. If NAT rewrites the addresses first, the policy match sees the translated address instead of the original, and the wrong link is selected.