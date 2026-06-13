+++
title = 'Firewalls'
date = '2026-06-06T18:00:57-04:00'
weight = 40
draft = false
+++

Linux implements firewalling inside the kernel through *Netfilter*, a framework of hooks embedded in the network stack. Every packet that enters, exits, or passes through the machine triggers these hooks, giving the kernel the opportunity to inspect, accept, drop, or modify it.

Two user-space tools configure Netfilter rules: *iptables* and its successor *nftables*. Higher-level frontends such as `ufw` on Ubuntu and `firewalld` on RHEL and Fedora sit on top of these and provide simpler interfaces for common configurations. Underneath, they all write rules into the same Netfilter framework.

## Chains and tables

### Chains

Netfilter organizes rules into *tables* and *chains*. A chain is a list of rules evaluated in order for packets at a specific point in the processing path:

| Chain         | When it applies                                             |
| :------------ | :---------------------------------------------------------- |
| `INPUT`       | Packets destined for the local machine                      |
| `OUTPUT`      | Packets originating from the local machine                  |
| `FORWARD`     | Packets passing through the machine between interfaces      |
| `PREROUTING`  | Packets immediately on arrival, before routing decisions    |
| `POSTROUTING` | Packets immediately before leaving, after routing decisions |

### Tables

Netfilter organizes chains into *tables* based on what operation the rule performs:

| Table    | Purpose                                                                                                                                            | Chains                          |
| :------- | :------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------ |
| `filter` | Default table. All accept/drop decisions go here. Running `iptables -L` with no `-t` flag shows this table.                                        | INPUT, FORWARD, OUTPUT          |
| `nat`    | Address translation. Use PREROUTING for DNAT (port forwarding) and POSTROUTING for SNAT/MASQUERADE (sharing a single public IP across many hosts). | PREROUTING, OUTPUT, POSTROUTING |
| `mangle` | Modify packet headers: set DSCP for QoS, adjust TTL, clamp TCP MSS on VPN tunnels. Most host firewalls never use this table.                       | All five chains                 |
| `raw`    | Exempt specific packets from connection tracking before conntrack runs.                                                                            | PREROUTING, OUTPUT              |

### Connection tracking

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

## iptables and nftables

### iptables

`iptables` was introduced with Linux 2.4 in 2001 as the first major Netfilter user-space tool. It became the standard Linux firewall interface for the next two decades.

Its design reflects its age. Each protocol required a separate binary: `iptables` for IPv4, `ip6tables` for IPv6, `arptables` for ARP, and `ebtables` for Ethernet bridge filtering. Rules are evaluated linearly. Every packet walks the chain from top to bottom until a matching rule is found. Each rule ends with a *target* that tells Netfilter what to do with the matching packet:

- `ACCEPT`: pass the packet through.
- `DROP`: discard the packet silently. The sender receives no response.
- `RETURN`: stop evaluating the current chain and return to the calling chain. In the built-in chains, `RETURN` triggers the default policy.

When no rule matches, the chain's *default policy* applies. On a host with hundreds of rules, linear evaluation becomes expensive. Most hardened host firewalls set the INPUT chain's default policy to `DROP` and add explicit rules only for permitted services.

Rule updates are not atomic: adding or removing rules happens one at a time, which creates brief windows where the ruleset is in an inconsistent state.


### nftables

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

## iptables commands

All `iptables` commands operate on the `filter` table by default. If you omit `-t`, you are reading or writing the filter table. Pass `-t nat` or `-t mangle` to target those tables explicitly. Every example in this section uses the filter table unless `-t` is shown.

### Viewing rules

iptables rules live in kernel memory at runtime. To view all tables at once or dump the full ruleset:

```bash
iptables -L -n -v           # list all filter table rules
iptables -t nat -L -n -v    # list nat table rules
iptables -t mangle -L -n -v # list mangle table rules
```

The kernel exposes limited metadata through `/proc`:

```bash
cat /proc/net/ip_tables_names    # active tables (filter, nat, mangle, raw)
cat /proc/net/ip_tables_matches  # loaded match modules
cat /proc/net/ip_tables_targets  # loaded target modules
```


Run `iptables -L -v` to list all rules in the `filter` table with packet and byte counters:

```bash
$ iptables -L -v
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
```

This output is from a host with no rules configured. Each chain header reports its default policy and running counters:

- `policy ACCEPT`: all packets that reach the end of the chain without matching a rule are accepted. A hardened host sets this to `DROP` on INPUT.
- `0 packets, 0 bytes`: traffic counters since the last reset. Reset them with `iptables -Z`.

The column headers describe each rule field:

| Column        | Description                                                             |
| :------------ | :---------------------------------------------------------------------- |
| `pkts`        | Number of packets matched by this rule.                                 |
| `bytes`       | Total bytes matched by this rule.                                       |
| `target`      | Action to take: `ACCEPT`, `DROP`, `RETURN`, or a jump to another chain. |
| `prot`        | Protocol: `tcp`, `udp`, `icmp`, or `all`.                               |
| `opt`         | IP option flags. Usually `--` (none).                                   |
| `in`          | Inbound interface the rule applies to. `*` means any.                   |
| `out`         | Outbound interface the rule applies to. `*` means any.                  |
| `source`      | Source IP or network. `0.0.0.0/0` means any.                            |
| `destination` | Destination IP or network. `0.0.0.0/0` means any.                       |

Empty chains with `policy ACCEPT` on all three chains mean the host is passing all traffic without restriction. This is the default state on a freshly installed system before any firewall rules are applied.



### Adding a rule

Admin services like SSH should only accept connections from trusted sources: a management subnet, a jump host, or a specific workstation. Leaving SSH open to the internet exposes the host to brute-force and credential-stuffing attacks. Application ports like HTTPS serve all clients, so they accept connections from any source.

Use `iptables -A` to append a rule to a chain or `-I` to insert at a specific position. Common options:

| Option             | Description                                                                                                              |
| :----------------- | :----------------------------------------------------------------------------------------------------------------------- |
| `-I <chain> [pos]` | Insert a rule at position `pos` (default: 1, the top). Rules inserted at position 1 are evaluated first.                 |
| `-p <protocol>`    | Match a protocol: `tcp`, `udp`, `icmp`, or `all`. Required before using `--dport` or `--sport`.                          |
| `-s <address>`     | Match a source IP address or network (for example, `192.168.1.0/24`).                                                    |
| `--dport <port>`   | Match a destination port number or range (for example, `22` or `8080:8090`). Requires `-p tcp` or `-p udp`.              |
| `-j <target>`      | Jump to a target: `ACCEPT`, `DROP`, `RETURN`, or another chain.                                                          |
| `-v`               | Verbose output. Shows packet and byte counters, interface names, and full rule details when listing rules.               |
| `--line-numbers`   | Display rule position numbers when listing a chain. Use with `-L` to identify rules for insertion or deletion by number. |

```bash
# Allow inbound SSH from a specific subnet
iptables -A INPUT -p tcp -s 192.168.1.0/24 --dport 22 -j ACCEPT

# Insert a rule at the top of INPUT to drop a specific host
iptables -I INPUT 1 -s 10.0.0.5 -j DROP
```

These commands configure SSH and HTTPS connections. The last command lets you view the ruleset:
```bash
# Allow inbound SSH on a specific interface from a subnet
iptables -A INPUT -i enp1s0 -p tcp -s 192.168.122.0/24 --dport 22 -j ACCEPT

# Allow inbound HTTPS from any source
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Verify rules with line numbers
iptables -L -n -v --line-numbers

Chain INPUT (policy ACCEPT 12 packets, 912 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1      257 18380 ACCEPT     6    --  enp1s0 *       192.168.122.0/24     0.0.0.0/0            tcp dpt:22
2        0     0 ACCEPT     6    --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:443

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination
```

- `policy ACCEPT 12 packets` on INPUT: 12 packets reached the end of the chain without matching either rule and were accepted by the default policy.
- `num`: the rule's position in the chain. Use this number with `iptables -D INPUT <num>` to delete a specific rule.
- `257 18380`: rule 1 has matched 257 packets totalling 18,380 bytes. Rule 2 has never matched.
- `prot 6`: `-n` suppresses name resolution, so protocols are shown as numbers. `6` is TCP.
- `in enp1s0` on rule 1: the rule only matches packets arriving on that interface. Rule 2 shows `*`, meaning any interface.
- `tcp dpt:22` / `tcp dpt:443`: the port match conditions appended after the address columns.

### Labeling a rule

The `-m comment --comment` option attaches a human-readable label to a rule. The label does not affect matching or packet handling. It appears in `iptables -L` output as `/* text */` and is useful for documenting why a rule exists when auditing a ruleset.

```bash
iptables -A INPUT -i enp1s0 -p tcp -s 192.168.122.0/24 --dport 22 -j ACCEPT -m comment --comment "Permitted admin"
iptables -I INPUT 2 -i enp1s0 -p tcp --dport 22 -j DROP -m comment --comment "Block admin"
iptables -I INPUT 3 -i enp1s0 -p tcp -s 1.2.3.4/24 --dport 443 -j DROP -m comment --comment "Block inbound web"
iptables -A INPUT -p tcp --dport 443 -j ACCEPT -m comment --comment "Permit all web access"

iptables -L -n --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination
1    ACCEPT     6    --  192.168.122.0/24     0.0.0.0/0            tcp dpt:22
2    DROP       6    --  0.0.0.0/0            0.0.0.0/0            tcp dpt:22 /* Block admin */
3    DROP       6    --  1.2.3.0/24           0.0.0.0/0            tcp dpt:443 /* Block inbound web */
4    DROP       6    --  0.0.0.0/0            0.0.0.0/0            tcp dpt:22
5    DROP       6    --  1.2.3.0/24           0.0.0.0/0            tcp dpt:443
6    ACCEPT     6    --  0.0.0.0/0            0.0.0.0/0            tcp dpt:443
7    LOG        0    --  0.0.0.0/0            0.0.0.0/0            LOG flags 0 level 4
8    LOG        0    --  192.168.122.0/24     0.0.0.0/0            LOG flags 0 level 4
9    LOG        0    --  192.168.122.0/24     0.0.0.0/0            LOG flags 0 level 3 prefix "*SSH FROM VM NETWORK"
10   LOG        0    --  192.168.122.0/24     0.0.0.0/0            LOG flags 0 level 3 prefix "*SSH FROM VM NETWORK*"
11   ACCEPT     6    --  192.168.122.0/24     0.0.0.0/0            tcp dpt:22 /* Permitted admin */
12   ACCEPT     6    --  0.0.0.0/0            0.0.0.0/0            tcp dpt:443 /* Permit all web access */
...
```

Rules 2, 3, 11, and 12 show their comments inline. Rules 1 and 4–10 were added in earlier examples without `-m comment` and show no label. The two new rules were inserted at positions 2 and 3 and appended at the end, accumulating on top of the existing ruleset. In a clean ruleset, add comments to every rule from the start.

### Ordering rules

Keep related rules adjacent to each other and put the most specific rule first within each group. The broad default for a port goes last in the group, after all exceptions have been evaluated. Because rules are evaluated sequentially, place the groups that handle the most traffic first. A web application server likely receives far more HTTPS traffic than SSH, so HTTPS rules belong at the top.

When you insert a rule at a specific position, every existing rule at that position and below shifts down by one.

The starting state for this table: rule 1 accepts SSH from the management subnet, rule 2 accepts HTTPS from anywhere:

```bash
# Insert a catch-all DROP for SSH at position 2 — pushes the HTTPS rule to position 3
iptables -I INPUT 2 -i enp1s0 -p tcp --dport 22 -j DROP

# Insert a DROP for a blocked HTTPS source at position 3 — pushes the HTTPS ACCEPT to position 4
iptables -I INPUT 3 -i enp1s0 -p tcp -s 1.2.3.4/24 --dport 443 -j DROP

iptables -L -n --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination
1    ACCEPT     6    --  192.168.122.0/24     0.0.0.0/0            tcp dpt:22
2    DROP       6    --  0.0.0.0/0            0.0.0.0/0            tcp dpt:22
3    DROP       6    --  1.2.3.0/24           0.0.0.0/0            tcp dpt:443
4    ACCEPT     6    --  0.0.0.0/0            0.0.0.0/0            tcp dpt:443

Chain FORWARD (policy ACCEPT)
num  target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination
```

Rules 1–2 form the SSH group: allow the management subnet, drop everyone else. Rules 3–4 form the HTTPS group: drop a blocked source range, allow everyone else. A packet from `192.168.122.5` to port 22 matches rule 1 and is accepted without ever reaching rule 2. A packet from `1.2.3.100` to port 443 matches rule 3 and is dropped before reaching rule 4.

### Default policy

The default policy determines what happens to packets that reach the end of a chain without matching any rule. Set it with `-P`:

```bash
iptables -P INPUT ACCEPT    # pass unmatched packets (permissive baseline)
iptables -P INPUT DROP      # discard unmatched packets (restrictive baseline)
```

`ACCEPT` is the default on a freshly installed system. `DROP` is the correct posture for any host exposed to untrusted traffic.

**Set all permit rules before changing the policy to DROP.** If you run `iptables -P INPUT DROP` over an SSH session without first adding a rule to allow SSH, the kernel enforces the new policy immediately. Your current session ends and you cannot reconnect. Recovery requires physical console access or, on a cloud instance, the provider's out-of-band console.

The safe sequence for locking down a host:

1. Allow the loopback interface (required by many local services).
   ```bash
   iptables -A INPUT -i lo -j ACCEPT
   ```
2. Allow established and related traffic so return packets for outbound connections are not blocked.
   ```bash
   iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
   ```
3. Allow SSH from the management subnet only.
   ```bash
   iptables -A INPUT -i enp1s0 -p tcp -s 192.168.122.0/24 --dport 22 -j ACCEPT
   ```
4. Allow application traffic.
   ```bash
   iptables -A INPUT -p tcp --dport 443 -j ACCEPT
   ```
5. Verify the rules look correct before changing the policy.
   ```bash
   iptables -L -n --line-numbers
   ```
6. Set the default policy to DROP. Do this last.
   ```bash
   iptables -P INPUT DROP
   iptables -P FORWARD DROP
   ```

Leave `OUTPUT` as `ACCEPT` unless you have a specific reason to restrict outbound traffic. Restricting OUTPUT requires rules for every outbound service the host initiates (DNS, NTP, package updates) and is error-prone to maintain.

**Flushing rules safely.** `iptables -F` removes all rules but leaves the default policy in place. If you flush rules while the policy is `DROP`, you will lose SSH access immediately. Always reset the policy to `ACCEPT` before flushing:

```bash
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -F
```

### Persisting rules

Rules are lost on reboot unless persisted to disk. Where they are saved depends on the distribution:

| Distro                                | Path                                                  |
| :------------------------------------ | :---------------------------------------------------- |
| Debian/Ubuntu (`iptables-persistent`) | `/etc/iptables/rules.v4`, `/etc/iptables/rules.v6`    |
| RHEL/CentOS/Fedora                    | `/etc/sysconfig/iptables`, `/etc/sysconfig/ip6tables` |

On Debian/Ubuntu, save the current ruleset with:

```bash
iptables-save                                   # dump all rules in saveable format
sudo iptables-save > /etc/iptables/rules.v4     # redirect saveable output to a file
```

The `netfilter-persistent` service loads these files at boot. On RHEL-family systems, the `iptables` service handles this automatically when you run `service iptables save`.

### Deleting a rule

Delete a rule by its position number using `-D`. Rule numbers shift up after a deletion, so verify the current numbering with `--line-numbers` before and after:

```bash
iptables -L -n --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination
1    ACCEPT     6    --  192.168.122.0/24     0.0.0.0/0            tcp dpt:22
2    DROP       6    --  0.0.0.0/0            0.0.0.0/0            tcp dpt:22
3    DROP       6    --  1.2.3.0/24           0.0.0.0/0            tcp dpt:443
4    ACCEPT     6    --  0.0.0.0/0            0.0.0.0/0            tcp dpt:443
...

# Delete rule 3 (the DROP for 1.2.3.0/24 on port 443)
iptables -D INPUT 3

iptables -L -n --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination
1    ACCEPT     6    --  192.168.122.0/24     0.0.0.0/0            tcp dpt:22
2    DROP       6    --  0.0.0.0/0            0.0.0.0/0            tcp dpt:22
3    ACCEPT     6    --  0.0.0.0/0            0.0.0.0/0            tcp dpt:443

Chain FORWARD (policy ACCEPT)
num  target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination
```

The DROP rule for `1.2.3.0/24` is gone. The HTTPS ACCEPT rule that was at position 4 has moved up to position 3. Any rule number you stored before the deletion is now wrong by one.

### Logging

The `LOG` target records matching packets to the kernel log. Unlike `ACCEPT` or `DROP`, `LOG` is non-terminating: the packet continues to the next rule after being logged. Place a `LOG` rule immediately before a `DROP` rule for the same traffic if you want a record of what you are blocking.

```bash
# Log all INPUT packets (very noisy — use only for short-term debugging)
iptables -A INPUT -j LOG

# Log packets from a specific subnet
iptables -A INPUT -s 192.168.122.0/24 -j LOG

# Log with a custom prefix and severity level
iptables -A INPUT -s 192.168.122.0/24 -j LOG --log-level 3 --log-prefix '*SSH FROM VM NETWORK*'
```

- `--log-prefix`: prepends a string to every log entry. Use it to identify the rule that generated the entry when searching logs.
- `--log-level`: syslog severity level. `3` is `err`. Common values: `4` (warning), `6` (info), `7` (debug). Lower numbers are more severe.

To log dropped packets, add a `LOG` rule before the `DROP` rule for the same match:

```bash
iptables -A INPUT -s 1.2.3.0/24 -j LOG --log-prefix 'BLOCKED: '
iptables -A INPUT -s 1.2.3.0/24 -j DROP
```

Viewing logs on Ubuntu:

```bash
journalctl -k | grep 'SSH FROM VM NETWORK'
grep 'iptables' /var/log/kern.log
```

Viewing logs on RHEL/Fedora:

```bash
journalctl -k | grep 'SSH FROM VM NETWORK'
grep 'iptables' /var/log/messages
```

## NAT table

*Network Address Translation (NAT)* rewrites IP addresses and ports in packet headers as packets pass through the firewall. From the perspective of the host receiving the packet, the traffic appears to originate from a different address than it actually does.

### Address translation

Every device on a private network uses an RFC 1918 address (10.0.0.0/8, 172.16.0.0/12, or 192.168.0.0/16). These addresses are not routable on the public internet. When a packet from `192.168.1.5` needs to reach a server at `1.2.3.4`, the firewall must translate the source address to a public IP before forwarding the packet. The server at `1.2.3.4` sees the connection arrive from the firewall's public address, not from the private host.

This is *Source NAT (SNAT)*. The reverse is *Destination NAT (DNAT)*: rewriting the destination address. Port forwarding is DNAT. Traffic arriving at the firewall's public IP on port 80 is rewritten to target an internal server at 192.168.1.10:80.

### Many hosts, one public address

Most home and small-office networks share a single public IP address across all internal hosts. The technique is called *Port Address Translation (PAT)* or *masquerading*. The firewall tracks each internal connection using its full 5-tuple and assigns each a unique source port on the public address.

An internal host at `192.168.1.5:54321` connecting to `1.2.3.4:443` is translated to `203.0.113.1:32768` connecting to `1.2.3.4:443`. A second host at `192.168.1.9:51000` connecting to the same server becomes `203.0.113.1:32769`. The external server sees two connections from the same IP but different ports. The firewall knows which internal host each belongs to because every source port on the public address maps to exactly one inside tuple.

The terminology for this technique varies by vendor and context. Cisco IOS calls it *NAT overload*. Linux calls it *masquerade*. The protocol-level term is *PAT* or *NAPT (Network Address Port Translation)*. All three refer to the same mechanism: many inside addresses sharing one outside address, distinguished by port number.

### Tuple mapping

The firewall records each NAT mapping as a pair of tuples: the original (inside) tuple and the translated (outside) tuple.

| Direction            | src IP      | src port | dst IP  | dst port |
| :------------------- | :---------- | :------- | :------ | :------- |
| Inside (original)    | 192.168.1.5 | 54321    | 1.2.3.4 | 443      |
| Outside (translated) | 203.0.113.1 | 32768    | 1.2.3.4 | 443      |

When the server sends a reply to `203.0.113.1:32768`, the firewall looks up the outside tuple, finds the matching inside tuple, rewrites the destination back to `192.168.1.5:54321`, and forwards the packet to the internal host. Without this table, return traffic would arrive at the firewall with no way to determine which internal host to deliver it to.

### The NAT table in memory

Linux stores NAT mappings in the *connection tracking* subsystem (`conntrack`), the same subsystem that powers stateful firewall rules. Each entry records both tuples, the protocol, the current connection state, and a remaining timeout.

View the live table using the `conntrack` command from the `conntrack-tools` package:

```bash
apt install conntrack       # Debian/Ubuntu
dnf install conntrack-tools # RHEL/Fedora

sudo conntrack -L           # list all entries
sudo conntrack -L -p tcp    # list TCP entries only
```

`/proc/net/nf_conntrack` also exposes the table directly, but only when the `nf_conntrack` kernel module is loaded and connection tracking is active:

```bash
cat /proc/net/nf_conntrack
```

A TCP entry looks like this:

```bash
ipv4  2  tcp  6  86394  ESTABLISHED \
  src=192.168.1.5  dst=1.2.3.4  sport=54321  dport=443 \
  src=1.2.3.4      dst=203.0.113.1  sport=443  dport=32768  [ASSURED]
```

The first address pair is the original direction. The second is the reply direction with translated addresses. `[ASSURED]` means traffic has been seen in both directions. The number `86394` is the remaining timeout in seconds before the entry is removed if no further packets arrive.

### Removing entries

TCP signals connection lifecycle explicitly through its handshake and teardown. Conntrack watches for the FIN exchange and advances the entry through states toward removal. After both sides have sent FIN and received ACK, the entry enters TIME_WAIT. It is removed after the TIME_WAIT timer expires.

| conntrack state   | Default timeout  | sysctl                                 |
| :---------------- | :--------------- | :------------------------------------- |
| ESTABLISHED       | 5 days (432000s) | `nf_conntrack_tcp_timeout_established` |
| TIME_WAIT         | 120s             | `nf_conntrack_tcp_timeout_time_wait`   |
| CLOSE (after RST) | 10s              | `nf_conntrack_tcp_timeout_close`       |

A RST packet moves the entry directly to CLOSE and it is purged in seconds. This is why an abruptly closed TCP connection frees the NAT table slot faster than a graceful FIN teardown.

UDP has no connection lifecycle signals. Conntrack removes UDP entries purely by timeout after the last packet is seen.

| UDP state                           | Default timeout | sysctl                            |
| :---------------------------------- | :-------------- | :-------------------------------- |
| Unidirectional (no reply yet)       | 30s             | `nf_conntrack_udp_timeout`        |
| Stream (bidirectional traffic seen) | 180s            | `nf_conntrack_udp_timeout_stream` |

A UDP mapping disappears 30 seconds after the last packet if the remote host never replied, or 180 seconds after the last packet in an established exchange. This is why UDP-based applications such as VPNs, online games, and VoIP can silently break through NAT: the mapping expires while the application believes the session is still open, and the next packet from the server arrives at the firewall with no matching entry to reverse.

### iptables NAT commands

Enable IP forwarding before adding any NAT rules:

```bash
# Enable IP forwarding (required for routing between interfaces)
echo 1 > /proc/sys/net/ipv4/ip_forward

# Make it persistent
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
```

Common options:

| Option                               | Description                                                                                                                            |
| :----------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------- |
| `-t nat`                             | Target the `nat` table. Required for all NAT rules.                                                                                    |
| `-A POSTROUTING`                     | Append a rule to POSTROUTING. Used for SNAT and MASQUERADE, which rewrite the source address after the routing decision.               |
| `-A PREROUTING`                      | Append a rule to PREROUTING. Used for DNAT, which rewrites the destination address before the routing decision.                        |
| `-o <interface>`                     | Match packets leaving on this interface. Used to restrict SNAT and MASQUERADE to the WAN interface.                                    |
| `-i <interface>`                     | Match packets arriving on this interface. Used to restrict DNAT to inbound traffic on the WAN interface.                               |
| `-s <address>`                       | Match a source IP or subnet. Scopes SNAT to specific internal hosts.                                                                   |
| `-j MASQUERADE`                      | Rewrite the source address to the outbound interface's current IP. Re-reads the IP on every packet. Use when the public IP is dynamic. |
| `-j SNAT --to-source <ip>`           | Rewrite the source address to a fixed IP. More efficient than MASQUERADE. Use when the public IP is static.                            |
| `-j DNAT --to-destination <ip:port>` | Rewrite the destination address and optionally the port. Used for port forwarding.                                                     |

**Masquerade (dynamic public IP)**

```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

Rewrites the source address of every packet leaving `eth0` to match that interface's current IP address. The kernel assigns a unique source port to track each session in the NAT table. Use this when the WAN IP is assigned by DHCP and may change. Because `MASQUERADE` looks up the interface address on every packet, it is slightly slower than `SNAT`.

**SNAT (static public IP)**

```bash
iptables -t nat -A POSTROUTING -o eth0 -s 192.168.1.0/24 -j SNAT --to-source 203.0.113.1
```

Rewrites the source address of packets from `192.168.1.0/24` leaving `eth0` to `203.0.113.1`. The target address is fixed at rule creation, so the kernel does not need to look it up per packet. Use this when the WAN IP is static.

**DNAT / port forwarding**

```bash
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 192.168.1.10:80
```

Rewrites the destination address of inbound TCP packets arriving on `eth0` with destination port 80. The kernel changes the destination to `192.168.1.10:80` before making the routing decision, so the packet is forwarded to the internal server rather than delivered to the firewall itself.

#### Deleting nat table rules

```bash
iptables -t nat -D POSTROUTING 1    # delete rule 1 from POSTROUTING
iptables -t nat -D PREROUTING 1     # delete rule 1 from PREROUTING
iptables -t nat -F                  # flush all chains in the nat table
iptables -t nat -F POSTROUTING      # flush only the POSTROUTING chain
```

### Encryption and NAT

NAT modifies IP and TCP/UDP headers. Encryption protects the payload. They coexist as long as encryption does not cover the fields NAT needs to rewrite. Where they conflict, the order in which you apply them determines whether the combination works.

TLS and application-layer encryption work transparently through NAT. NAT rewrites the source IP and port in the IP and TCP headers. TLS encrypts only the application payload above those headers. There is no conflict.

IPsec is where order matters:

| IPsec mode      | NAT compatibility                                                                                                                                                                  |
| :-------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ESP tunnel mode | Compatible with NAT using NAT-T. The original packet is encrypted and wrapped in a new outer IP header. NAT rewrites the outer header without touching the encrypted inner packet. |
| AH mode         | Incompatible with NAT. AH authenticates the entire IP header including the source address. NAT changes the source address, which breaks the authentication check on the far end.   |

*NAT Traversal (NAT-T)* resolves the ESP tunnel mode problem by wrapping IPsec packets in UDP port 4500. NAT can translate UDP headers without touching the encrypted payload. IKE detects NAT automatically during negotiation and switches to UDP 4500.

Applying NAT before encryption is the standard pattern for site-to-site VPNs. Outbound traffic from a private host is NATted to the tunnel endpoint's public IP first, then encrypted and sent through the VPN. The far end sees connections from the public IP. The private addressing is never exposed outside the tunnel.

Applying encryption before NAT occurs when a VPN client is behind CGNAT, as on most mobile networks. The client encrypts traffic and sends it to the VPN gateway using its private address. The carrier's NAT translates the outer UDP 4500 packet. NAT-T handles this because the encrypted payload is untouched. The VPN gateway sees the connection from the NAT's public IP and uses the IKE identity to match the client.

The practical rule: encrypt the payload, NAT the headers. On a Linux router acting as both a NAT gateway and a VPN endpoint, apply `MASQUERADE` or `SNAT` to traffic on the WAN interface and let the VPN handle encryption on the tunnel interface independently. Do not apply SNAT to traffic that is already inside an encrypted tunnel.

## Mangle table

The mangle table modifies IP packet header fields as packets transit the Linux host. Unlike the filter table (which accepts or drops) and the nat table (which rewrites addresses), mangle makes surgical changes to values already inside the header: TCP segment size, IP flags, TTL, and QoS markings. The packet's source and destination addresses stay untouched.

Most host firewalls never use the mangle table. It becomes necessary in two specific situations: MSS clamping on PPPoE or VPN links where the MTU is smaller than standard Ethernet, and DSCP marking for QoS on managed networks. A standard server or workstation only needs the filter and nat tables.

### MSS clamping

#### The MTU problem on DSL and satellite links

Standard Ethernet carries IP packets up to 1500 bytes. DSL connections using PPPoE (Point-to-Point Protocol over Ethernet) add 8 bytes of PPPoE and PPP overhead to every frame, leaving only 1492 bytes for the IP packet. Satellite links vary but commonly impose MTUs of 1400–1460 bytes. VPN tunnels add their own overhead on top of the underlying link.

When a host behind a DSL router sends a standard 1500-byte TCP packet, the packet exceeds the PPPoE link MTU. The router must either fragment it or drop it. Fragmentation adds overhead and is increasingly disabled on modern networks. If fragmentation is disabled and the packet is too large, it is silently dropped and the connection stalls.

#### Path MTU Discovery

*Path MTU Discovery (PMTUD)* is the mechanism hosts use to find the largest packet size that fits end-to-end across a network path. The sending host sets the *Don't Fragment (DF)* bit in the IP header. If a router along the path cannot forward the packet without fragmenting it, it drops the packet and sends back an ICMP Type 3 Code 4 message (*Fragmentation Needed*) containing the MTU of the bottleneck link. The sender receives this signal, reduces its packet size, and retries. This repeats until the path MTU is found and cached in the host's routing table.

#### When PMTUD breaks

PMTUD depends entirely on ICMP Type 3 Code 4 messages reaching the sender. Many firewalls block all ICMP traffic as a blanket policy. When these messages are dropped, the sender never learns the path is constrained. It keeps sending large packets with the DF bit set. The router keeps dropping them. The connection enters a state called a *black hole*: the TCP handshake completes (SYN and SYN-ACK are small packets that fit), but large data transfers stall or hang indefinitely.

This is a well-documented problem with older Windows services such as SMB (file sharing) and older web proxies. These applications assume a standard 1500-byte MTU and send large frames immediately. On a PPPoE or VPN link with ICMP blocked, they appear to connect but then freeze when the first full-size data exchange begins.

The fix is either to unblock ICMP Type 3 Code 4 at the firewall (the correct solution), or to use MSS clamping (the practical workaround when you cannot control intermediate firewalls).

#### MSS clamping with iptables

*Maximum Segment Size (MSS)* is a TCP option exchanged during the three-way handshake that tells the remote host the largest payload it should send per segment. MSS is negotiated in the SYN and SYN-ACK packets and governs the entire connection. By rewriting the MSS in those packets, the mangle table can cap segment size before any data flows, without touching the data itself.

```bash
iptables -t mangle -A FORWARD -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --set-mss 1412
```

- `-t mangle`: apply the rule in the mangle table
- `-A FORWARD`: match packets being routed between interfaces (not destined for the local host)
- `-p tcp --tcp-flags SYN,RST SYN`: match TCP packets with the SYN flag set and RST not set — these are initial connection packets
- `-j TCPMSS --set-mss 1412`: rewrite the MSS TCP option to 1412 bytes

MSS of 1412 accounts for a 1452-byte IP payload on a PPPoE link (1492 byte MTU minus 20 bytes IP header minus 20 bytes TCP header = 1452), with an additional 40-byte margin for VPN or other tunnel overhead. Remote hosts that would have sent 1460-byte segments (standard Ethernet MSS) instead send 1412-byte segments, keeping every packet within the link MTU without relying on ICMP feedback.

#### Testing path MTU

Before adding MSS clamping rules, verify what the actual path MTU is. Two tools probe this directly.

`ping` with the DF bit and a specific payload size:

```bash
ping -M do -s 1400 8.8.8.8
```

`-M do` sets the DF bit. `-s 1400` sends 1400 bytes of ICMP payload. The total IP packet is 1428 bytes (1400 + 8 ICMP header + 20 IP header). If the path supports this size, you receive replies. If not, you receive an ICMP Fragmentation Needed message or silence if ICMP is blocked. Reduce the size until you find the largest value that receives a reply.

`nping` probes over TCP rather than ICMP, which is useful when ICMP is filtered but TCP is permitted:

```bash
nping --tcp -p 53 -df --mtu 1400 -c 1 8.8.8.8
```

`-df` sets the DF bit. `--mtu 1400` sets the packet size. `-p 53` targets DNS port 53, which is typically open through firewalls. `-c 1` sends one packet. Because this uses TCP rather than ICMP, it reaches further through firewalls that block ping. If the packet is too large, you observe no response or a TCP RST rather than an ICMP reply.

The relationship between the two tools and the iptables rule: use `ping -M do` and `nping` to measure the actual path MTU, then subtract 40 bytes (20 IP + 20 TCP headers) to derive the correct MSS value for `--set-mss`.

#### Deleting mangle table rules

```bash
iptables -t mangle -D FORWARD 1     # delete rule 1 from FORWARD
iptables -t mangle -F               # flush all chains in the mangle table
iptables -t mangle -F FORWARD       # flush only the FORWARD chain
```

### Type of Service and DSCP

The IPv4 header contains an 8-bit *Type of Service (TOS)* field. RFC 791 (1981) defined it loosely. RFC 2474 (1998) redefined the top 6 bits as the *Differentiated Services Code Point (DSCP)*, which encodes a per-hop behavior that routers use to prioritize traffic. The bottom 2 bits became *Explicit Congestion Notification (ECN)*.

A DSCP value is a 6-bit number (0–63) that signals the traffic class. Routers configured for Differentiated Services read this field and apply the appropriate queuing and scheduling policy. A packet marked with the *Expedited Forwarding (EF)* class (DSCP 46) gets priority queuing and minimal delay. A packet marked as scavenger traffic (CS1, DSCP 8) is served only when the link is otherwise idle.

| DSCP name     | Value | Typical use                                   |
| :------------ | :---- | :-------------------------------------------- |
| CS0 / Default | 0     | Best effort. Default for unmarked traffic.    |
| CS1           | 8     | Scavenger. Bulk, background transfers.        |
| AF11          | 10    | High-throughput data.                         |
| AF41          | 34    | Video conferencing.                           |
| EF            | 46    | Voice, real-time interactive. Lowest latency. |
| CS6           | 48    | Network control traffic (routing protocols).  |

The mangle table sets DSCP values on packets before they leave the host, allowing downstream routers to prioritize them correctly. Mark SSH interactive sessions as AF41 and bulk SCP transfers as CS1, and a QoS-aware router will keep the interactive session responsive even when a large file transfer saturates the link.

```bash
# Mark SSH interactive traffic for priority handling
iptables -t mangle -A POSTROUTING -p tcp --dport 22 -j DSCP --set-dscp 34

# Mark bulk transfers as low priority
iptables -t mangle -A POSTROUTING -p tcp --dport 443 -j DSCP --set-dscp 8
```

DSCP marking is only effective when every router along the path is configured to honor the markings. On the public internet, most ISPs reset or ignore DSCP values at their edge. Marking is most useful inside a managed network or between two sites connected by an MPLS or SD-WAN service that preserves and acts on DSCP.
