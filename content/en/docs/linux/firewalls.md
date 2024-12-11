---
title: "Firewalls"
linkTitle: "Firewalls"
# weight: 1000
# description:
---

`iptables` and the newer `nftables` are low-level, and complex. Use `ufw` (uncomplicated firewall) for an easier solution.

## iptables

`iptables` manages the Netfilter packet-handling engine:
- Debian provides `ufw` (uncomplicated firewall) to simplify operations

### Concepts

There are ordered _chains_ of rules, and these chains make up _tables_. You create rules that define targets, chains, and sometimes tables. A rule is:

```bash

```

After the packet matches a rule, it is not tested against another rule.

#### Tables

The most common tables are `filter`, `mangle`, and `nat`. If a rule does not specify a table, it applies to the `filter` table by default.

| Table      | Description |
|------------|-------------|
| `filter`   | Allow or block packets from exiting the chain |
| `mangle`   | Change packet features before exiting the chain (resetting TTL values) |
| `nat`      | Change packet address exiting the chain (Network Address Translation) |
| `raw`      | `NOTRACK` setting on packets that should not be tracked |
| `security` | Mandatory access rules |

#### Chains

Chains are a list of rules that are processed in order. `INPUT`, `OUTPUT`, and `FORWARD` are usually all you need to set up rules for packets between two network interfaces (ex: your host and a remote). :

| Chain  |  Tables | Description |
|--------|---------|-------------|
| `PREROUTING`  | `nat`, `mangle`    | Handles packets before routing process |
| `INPUT`       | `filter`, `mangle` | Handles packets going to local host |
| `FORWARD`     | `filter`, `mangle` | Handles packets forwarded from one network interface to another |
| `POSTROUTING` | `nat`, `mangle`    | Handles packets being sent to remote system, after the forward filter |
| `OUTPUT`      | `filter`, `mangle` | Handles packets from local host |


| Table | Chain order |
|-------|-------------|
| `filter` | `INPUT` > `OUTPUT` > `FORWARD` |
| `nat`    | `OUTPUT` > `PREROUTING` > `POSTROUTING` |
| `mangle` | `INPUT` > `OUTPUT` > `FORWARD` > `PREROUTING` > `POSTROUTING` |

#### Targets

Each rule has a _target clause_, which is essentially a policy to apply to a packet when it matches a rule set in a chain. Example targets include:

`ACCEPT`
: Matching packets continue to their destination.

`DROP`
: Silently drop the packets.

`REJECT`
: Drop packets and send an ICMP error message.

`LOG`
: Tracks packets as they match rules.

`ULOG`
: Expands LOG.

`REDIRECT`
: Sends packets to a proxy. You can send all website traffic to a service.

`RETURN`
: Terminates user-defined chains.

`MIRROR`
: Swaps IP source and destination addresses.

`QUEUE`
: Hands packets to local user programs.



Each chain contains tables that defines rules for packet handling:

### Commands

Canonical [IptablesHowTo](https://help.ubuntu.com/community/IptablesHowTo)

```bash
# common formats
iptables -F <chain-name>                                # flush all rules from chain
iptables -P <chain-name> <target>                       # set policy (target) on chain
iptables -A <chain-name> -i <interface> -j <target>     # append this policy to the end of the chain

# additional formats
iptables -L [chain]                                     # list all rules for the chain, all rules if no chain
iptables -S [chain]                                     # list rules details for the chain, all rules details if no chain
iptables [-t table]                                     # apply command to table. If no table, applied to filter table
iptables -I <chain-name> <index> rule                   # insert this rule to this chain at this index location
iptables -D <chain-name> rule                           # delete this rule from this chain
iptables -R <chain-name> <index> rule                   # remove this rule from this chain at this index location
iptables -P <chain-name> policy                         # sets this policy as default on chain

# common options
-s source-ip        # apply rule to packets w source address
-d dest-ip          # apply rule to packets w dest address  
-sport source-port  # apply to packets from source-port
-dport dest-port    # apply to packets headed to dest-port
-i name             # apply rule to packets coming through name network interface
-o name             # apply rule to packets going out through name network interface
-p protocol         # apply rule to packets using this protocol
-j target           # apply the target action to the selected packets
                    # target values:
                    # ACCEPT
                    # DROP
                    # REJECT

# view current rules
sudo iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination 

# log dropped incoming packets
sudo iptables -N LOGGING            # Create new chain called LOGGING
sudo iptables -A INPUT -j LOGGING   # All remaining packets sent to LOGGING chain
sudo iptables -A LOGGING -m limit --limit 2/min -j LOG --log-prefix "IPTables-Dropped: " --log-level 4  # log to syslog
sudo iptables -A LOGGING -j DROP    # Drop all packets in logging chain

# log dropped incoming packets
iptables -N LOGGING
iptables -A OUTPUT -j LOGGING
iptables -A LOGGING -m limit --limit 2/min -j LOG --log-prefix "IPTables-Dropped: " --log-level 4
iptables -A LOGGING -j DROP

# log incoming and outgoing dropped packets
iptables -N LOGGING
sudo iptables -A INPUT -j LOGGING
iptables -A OUTPUT -j LOGGING
iptables -A LOGGING -m limit --limit 2/min -j LOG --log-prefix "IPTables-Dropped: " --log-level 4
iptables -A LOGGING -j DROP
```

## ufw

```bash
# allow ssh
ufw allow ssh
Rules updated
Rules updated (v6)

# deny http
ufw deny http
Rules updated
Rules updated (v6)

# enable new rules
sudo ufw enable


```