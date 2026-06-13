+++
title = 'nftables'
date = '2026-06-13T13:58:37-04:00'
weight = 20
draft = false
+++


`nftables` was merged into Linux 3.13 in 2014 as the designed replacement for iptables. It unifies all four tools into a single `nft` binary that handles IPv4, IPv6, ARP, and bridges with one consistent syntax.

The performance model is fundamentally different. nftables uses an in-kernel virtual machine to evaluate rules and supports *sets* and *maps*: data structures that let you match a packet against thousands of IP addresses or ports in a single O(log n) or O(1) lookup instead of walking a linear chain. Ruleset updates are atomic: the entire new ruleset is loaded as a single transaction, so there is never a moment with a partial or inconsistent policy in place.

On Debian 10, RHEL 8, and Ubuntu 20.04 and later, the `iptables` command is now a compatibility shim called `iptables-nft`. It accepts the same syntax you already know and translates it into nftables rules under the hood. The old kernel module-based backend is still available as `iptables-legacy`.

To check which backend your system is using:

```bash
iptables --version
# iptables v1.8.7 (nf_tables)   ← nftables backend
# iptables v1.8.7 (legacy)      ← old iptables backend
```

Use `nft` directly on any modern system. Use `iptables` syntax only when working with legacy scripts or systems that predate the nftables transition.