+++
title = 'nftables'
date = '2026-06-13T13:58:37-04:00'
weight = 20
draft = false
+++


`nftables` was merged into Linux 3.13 in 2014 as the designed replacement for iptables. It unifies all four tools into a single `nft` binary that handles IPv4, IPv6, ARP, and bridges with one consistent syntax.

The performance model is fundamentally different. nftables uses an in-kernel virtual machine to evaluate rules and supports *sets* and *maps*: data structures that let you match a packet against thousands of IP addresses or ports in a single O(log n) or O(1) lookup instead of walking a linear chain. Ruleset updates are atomic: the entire new ruleset is loaded as a single transaction, so there is never a moment with a partial or inconsistent policy in place. The VM-based evaluation model also uses less CPU than iptables, which walks every rule in sequence for each packet.

nftables exposes a Netlink-based API that orchestration tools can read and write directly. Terraform, Ansible, Puppet, Chef, and Salt all support this API, letting you define firewall rules as code and deploy them to many hosts in parallel. This makes nftables a natural fit for infrastructure-as-code workflows where consistency and speed of deployment matter.

On Debian 10, RHEL 8, and Ubuntu 20.04 and later, the `iptables` command is now a compatibility shim called `iptables-nft`. It accepts the same syntax you already know and translates it into nftables rules under the hood. The old kernel module-based backend is still available as `iptables-legacy`.

To check which backend your system is using:

```bash
iptables --version
# iptables v1.8.7 (nf_tables)   ← nftables backend
# iptables v1.8.7 (legacy)      ← old iptables backend
```

Use `nft` directly on any modern system. Use `iptables` syntax only when working with legacy scripts or systems that predate the nftables transition.

## iptables migration

`iptables-translate` converts a single iptables rule into its nftables equivalent. It ships with the `iptables` package on modern systems and requires no separate install. It prints the translated command but does not apply it or modify any tables.

```bash
iptables-translate -A INPUT -p tcp --dport 22 -j ACCEPT
# nft add rule ip filter INPUT tcp dport 22 counter accept
```

`iptables-restore-translate` converts a full `iptables-save` dump into an nftables ruleset file:

```bash
sudo iptables-save | iptables-restore-translate -f /etc/nftables.conf
```

Review the output before loading it. The translation covers the common cases but does not handle every iptables extension or custom chain pattern. Test the generated ruleset in a non-production environment first.

## Commands

nftables has no built-in tables or chains. Before adding rules, you must create the table and chain that will hold them. The order is always: table first, then chain, then rules.

### Add a table

Create the table with `nft add table`. The address family defaults to `ip` (IPv4) when omitted:

```bash
nft add table filter
```

### Add a chain

Create the chain inside the table. To hook the chain into the network stack so it actually processes packets, include a type, hook, and priority:

```bash
nft add chain ip filter INPUT '{ type filter hook input priority 0 ; policy drop ; }'
```

Without the hook definition, the chain only runs if another base chain jumps to it. It will not process packets on its own.

### Add rules

With the table and chain in place, add rules with `nft add rule`. Rules are appended in order and evaluated sequentially, so more specific rules must appear before broader ones.

#### Permit SSH from the management subnet

```bash
nft add rule ip filter INPUT iifname "enp1s0" ip saddr 1.2.3.0/24 tcp dport 22 counter accept comment \"Permit Admin\"
```

Accepts inbound SSH from `1.2.3.0/24` arriving on `enp1s0`. The `counter` keyword tracks how many packets match. Because this rule appears before the SSH drop rule, the management subnet is evaluated first.

#### Block all other SSH

```bash
nft add rule ip filter INPUT iifname "enp1s0" tcp dport 22 counter drop comment \"Block Admin\"
```

Drops any remaining SSH traffic on `enp1s0` that did not match the preceding accept rule. No source address restriction — this catches every other host.

#### Block a specific HTTPS source

```bash
nft add rule ip filter INPUT iifname "enp1s0" ip saddr 1.2.3.5 tcp dport 443 counter drop comment \"Block inbound Web\"
```

Drops HTTPS from the single host `1.2.3.5`. Placed before the broad HTTPS accept rule so this host is blocked before the permit is evaluated.

#### Permit all HTTPS

```bash
nft add rule ip filter INPUT tcp dport 443 counter accept comment \"Permit all Web Access\"
```

Accepts HTTPS from any source on any interface. No `iifname` or `ip saddr` match — this is the catch-all permit for port 443 after all specific exceptions have been evaluated.

### View ruleset

Each table entry shows its address family (`ip`), name (`mangle`, `filter`, `nat`), and the chains it contains. Each chain entry shows its type, the Netfilter hook it attaches to, the priority that determines processing order when multiple chains share the same hook, and the default policy. This output reflects the iptables-nft shim: these tables were created by `iptables-nft` translating existing iptables rules, not by `nft` directly.


```bash
nft list ruleset
```

```bash
table ip mangle {
    chain PREROUTING {
        type filter hook prerouting priority mangle; policy accept;
    }

    chain INPUT {
        type filter hook input priority mangle; policy accept;
    }

    chain FORWARD {
        type filter hook forward priority mangle; policy accept;
    }

    chain OUTPUT {
        type route hook output priority mangle; policy accept;
    }

    chain POSTROUTING {
        type filter hook postrouting priority mangle; policy accept;
    }
}
table ip filter {
    chain INPUT {
        type filter hook input priority filter; policy accept;
    }

    chain FORWARD {
        type filter hook forward priority filter; policy accept;
    }

    chain OUTPUT {
        type filter hook output priority filter; policy accept;
    }
}
table ip nat {
    chain PREROUTING {
        type nat hook prerouting priority dstnat; policy accept;
    }

    chain INPUT {
        type nat hook input priority srcnat; policy accept;
    }

    chain OUTPUT {
        type nat hook output priority dstnat; policy accept;
    }

    chain POSTROUTING {
        type nat hook postrouting priority srcnat; policy accept;
    }
}
```


### Common options

| Option      | Description                                                                                  |
| :---------- | :------------------------------------------------------------------------------------------- |
| `ip`        | Address family. `ip` for IPv4, `ip6` for IPv6, `inet` for both. Defaults to `ip` if omitted. |
| `iifname`   | Match packets arriving on the named interface.                                               |
| `ip saddr`  | Match source IPv4 address or network (for example, `1.2.3.0/24`).                            |
| `tcp dport` | Match TCP destination port.                                                                  |
| `counter`   | Track packet and byte counts for the rule. Appears in `nft list ruleset` output.             |
| `accept`    | Pass the packet.                                                                             |
| `drop`      | Discard the packet silently.                                                                 |
| `comment`   | Attach a human-readable label to the rule. Appears in `nft list ruleset` output.             |

## Basic configuration

Before writing nftables rules, flush any existing iptables rules. On systems where `iptables` is the `iptables-nft` shim, both tools write into the same kernel table. Leaving iptables rules in place means packets pass through two independent rulesets, making policy hard to reason about and troubleshoot. On systems using the legacy iptables backend, both subsystems run in parallel and both process every packet.

Flush iptables rules before switching:

```bash
sudo iptables -F
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo iptables -X
```

If `netfilter-persistent` is installed and loading iptables rules at boot, disable it:

```bash
sudo systemctl disable netfilter-persistent
```

### nft syntax

nftables organizes rules into *tables* and *chains*, similar to iptables, but you define both explicitly. There are no built-in tables or chains. A table is a container that holds chains. A chain holds rules. You specify the address family (`ip`, `ip6`, `inet`, `arp`, `bridge`) when you create the table.

`inet` covers both IPv4 and IPv6 in a single ruleset, replacing the need for separate `iptables` and `ip6tables` rules.

### Include files

Rather than maintaining one monolithic `/etc/nftables.conf`, you can split rules into separate files organized by service class and load them in order with `include` statements. A single file handles all SSH policy across every host that includes it. Another handles web traffic. A third handles database access. When policy changes for a service class, you update one file and reload.

```bash
include "/etc/nftables/ssh.conf"
include "/etc/nftables/web.conf"
include "/etc/nftables/vlans/*.conf"   # load all .conf files in a directory
```

The master file defines the tables and chains, sets the default policy, and includes the rule files in the order you want them evaluated.

#### Master file

**`/etc/nftables.conf`**:

```bash
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;
        iif lo accept
        ct state established,related accept
    }
    chain forward {
        type filter hook forward priority 0; policy drop;
    }
    chain output {
        type filter hook output priority 0; policy accept;
    }
}

include "/etc/nftables/ssh.conf"
include "/etc/nftables/web.conf"
```

#### Service files

Each service class has its own file. Deploy the same file to every host that runs that service — SSH policy stays consistent across all servers without duplicating rules.

**`/etc/nftables/ssh.conf`** — SSH policy:

```bash
# Permit SSH from management subnet; block all other SSH
add rule inet filter input ip saddr 192.168.1.0/24 tcp dport 22 accept comment "Permit admin SSH"
add rule inet filter input tcp dport 22 drop comment "Block all other SSH"
```

**`/etc/nftables/web.conf`** — web server policy:

```bash
# Permit inbound HTTP and HTTPS from any source
add rule inet filter input tcp dport { 80, 443 } accept comment "Permit HTTP and HTTPS"
```

#### Advanced rules

You can build more complex rule files using IP header fields. A QoS file can match on DSCP values to apply different forwarding policies, or mark traffic before it leaves the host:

```bash
# Mark SSH interactive sessions as high priority (AF41 = DSCP 34)
add rule inet filter output tcp dport 22 ip dscp set af41 comment "Prioritize SSH"

# Deprioritize bulk transfers (CS1 = DSCP 8)
add rule inet filter output tcp dport 443 ip dscp set cs1 comment "Deprioritize bulk HTTPS"
```

For hosts running IPsec, separate include files can apply rules at the PREROUTING and POSTROUTING hooks. Traffic must be classified and marked before encryption rewrites the headers in OUTPUT — the same constraint that applies in iptables. A `prerouting.conf` file handles policy matching on original addresses; a `postrouting.conf` file handles NAT or masquerading after the routing decision.

#### Verdict maps

A *verdict map* (`vmap`) routes packets to the correct chain based on destination IP range, replacing a long sequence of individual match rules. Each chain holds the permit and deny rules for that segment and can be defined in its own include file:

```bash
sudo nft add rule ip Firewall Forward ip daddr vmap '{ 192.168.21.1-192.168.21.254 : jump chain-pci21, 192.168.22.1-192.168.22.254 : jump chain-servervlan, 192.168.23.1-192.168.23.254 : jump chain-desktopvlan23 }'
```

The kernel evaluates the map in a single O(log n) lookup instead of walking one rule per range.

### Viewing the ruleset

```bash
sudo nft list ruleset                         # show all tables, chains, and rules
sudo nft list tables                          # show table names only
sudo nft list chain inet filter input         # show rules in a specific chain
```

### A basic host firewall

This example creates an `inet` table with a `filter` chain that drops inbound traffic by default and permits SSH and established connections:

```bash
# Create the table
sudo nft add table inet filter

# Create the input chain with a default drop policy
sudo nft add chain inet filter input '{ type filter hook input priority 0 ; policy drop ; }'

# Allow loopback
sudo nft add rule inet filter input iif lo accept

# Allow established and related traffic
sudo nft add rule inet filter input ct state established,related accept

# Allow SSH from a management subnet
sudo nft add rule inet filter input ip saddr 192.168.1.0/24 tcp dport 22 accept

# Verify
sudo nft list ruleset
```

### Persisting rules

nftables rules are lost on reboot unless saved to `/etc/nftables.conf`. The file uses native `nft` syntax and can be edited directly and reloaded rather than rebuilding the ruleset from individual `nft add` commands.

1. Save the current ruleset to the configuration file.
   ```bash
   sudo nft list ruleset | sudo tee /etc/nftables.conf
   ```
2. Enable the `nftables` service to load the file at boot.
   ```bash
   sudo systemctl enable nftables
   sudo systemctl start nftables
   ```
3. To apply changes from the file without rebooting:
   ```bash
   sudo nft -f /etc/nftables.conf
   ```

### Flush ruleset

`nft flush ruleset` removes all tables, chains, and rules in one command. Unlike iptables, there is no need to flush each table separately:

```bash
sudo nft flush ruleset
```

This clears the in-kernel ruleset only. It does not touch `/etc/nftables.conf`. If the `nftables` service is enabled, the file is reloaded at the next boot and the rules return. To permanently remove the rules, overwrite or delete the file after flushing.
