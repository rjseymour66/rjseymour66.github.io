---
title: "Firewalls"
linkTitle: "Firewalls"
# weight: 1000
# description:
---

Firewalls are easy to implement but difficult to implement well:
- Allow or disallow access to a network port
  - When admins start a service, they open a port for it 
  - Firewall allows port access from specific IPs or places
- Ubuntu uses uncomplicated Firewall (UFW)
  - Easy interface to `iptables` - use `ufw`, not `iptables`
  - inactive by default
  - configure, then activate it
- Allow access to ports that your server needs for functionality. Ex: 80 and 443 for webservers

## ufw

Uncomplicated firewall:

```bash
sudo apt install ufw            # install package
sudo ufw status                 # check status - confirm its inactive
sudo ufw enable                 # activate ufw

ufw allow from 192.168.1.158 to any port 22         # allow SSH from this IP
ufw allow from 192.168.1.0/24 to any port 22        # allow SSH from entire subnet
ufw allow 80                                        # allow all traffic to port 80
ufw allow 443                                       # allow all traffic to port 443

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

`iptables` and the newer `nftables` are low-level, and complex. Use `ufw` (uncomplicated firewall) for an easier solution.

## iptables

`iptables` manages the Netfilter packet-handling engine:
- Debian provides `ufw` (uncomplicated firewall) to simplify operations

There are ordered _chains_ of rules, and these chains make up _tables_. You create rules that define targets, chains, and sometimes tables. A rule is:

After the packet matches a rule, it is not tested against another rule.

### Rules

A rule is a set of conditions (matching criteria) paired with an action (target) that determines how iptables processes packets passing through a specific chain. It is the fundamental building block of the iptables framework for managing packet flow.

#### Components of a Rule

A rule consists of the following parts:

Matching Criteria: Conditions that a packet must meet to match the rule. Examples include:
- **Source/Destination Address**: Matches packets from or to a specific IP address or range.
- **Protocol**: Matches packets using a specific protocol (e.g., TCP, UDP, ICMP).
- **Port**: Matches packets directed to a specific port or port range (e.g., port 80 for HTTP).
- **Interface**: Matches packets arriving on or leaving a specific network interface (e.g., eth0).
- **State**: Matches packets based on their connection tracking state (e.g., NEW, ESTABLISHED).
- Other criteria like packet size, time of day, or flags.

Target: Specifies what action to take if the packet matches the rule. Common targets include:
- `ACCEPT`: Allow the packet.
- `DROP`: Silently discard the packet.
- `REJECT`: Discard the packet and notify the sender.
- `LOG`: Log the packet details for monitoring.

Chain Placement: A rule is added to a chain. When a packet reaches the chain, the rules are evaluated in order until a match is found or the chain is exhausted.

#### How a Rule Works

1. **Evaluation**: As packets traverse the networking stack, they are evaluated against the rules in a chain.
2. **Matching**: If the packet matches a rule's conditions, the specified target action is applied.
3. **Order Matters**: Rules in a chain are processed sequentially, so the order of rules affects behavior.

#### Example of a Rule

To allow incoming HTTP traffic (port 80) on interface eth0:

```bash
iptables -A INPUT -p tcp --dport 80 -i eth0 -j ACCEPT
```

Explanation:

- `-A INPUT`: Append the rule to the INPUT chain.
- `-p tcp`: Match packets using the TCP protocol.
- `--dport 80`: Match packets destined for port 80.
- `-i eth0`: Match packets arriving on the eth0 interface.
- `-j ACCEPT`: If the packet matches, accept it.

### Tables

A table is a collection of chains, each designed for specific types of packet processing. Each table is specialized for different functionalities. The most common tables are `filter`, `mangle`, and `nat`. If a rule does not specify a table, it applies to the `filter` table by default.

| Table      | Description                                                                                                                                                         |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `filter`   | (Default table): Used for basic packet filtering (e.g., deciding to ACCEPT or DROP packets). Allow or block packets from exiting the chain                          |
| `mangle`   | Used for modifying packets (e.g., changing headers or setting specific flags). Change packet features before exiting the chain (resetting TTL values)               |
| `nat`      | Used for network address translation (e.g., altering packet source or destination addresses). Change packet address exiting the chain (Network Address Translation) |
| `raw`      | Used for packet processing before connection tracking (e.g., marking packets to bypass tracking). `NOTRACK` setting on packets that should not be tracked           |
| `security` | Mandatory access rules. Used for SELinux-related decisions.                                                                                                         |


### Chains

A chain is a collection of rules applied to packets in sequence. Chains exist within a table and determine how packets are processed at different stages of their journey through the networking stack. Each chain processes packets based on its rules. If no rule matches, the default policy of the chain applies (e.g., ACCEPT or DROP).

There are two types of chains:
- Built-in chains: Predefined by iptables (e.g., `INPUT`, `OUTPUT`, `FORWARD` in the filter table).
- User-defined chains: Custom chains created by the user for modularity and reuse.

Chains are a list of rules that are processed in order. `INPUT`, `OUTPUT`, and `FORWARD` are usually all you need to set up rules for packets between two network interfaces (ex: your host and a remote). :

| Chain         | Tables             | Description                                                           |
| ------------- | ------------------ | --------------------------------------------------------------------- |
| `PREROUTING`  | `nat`, `mangle`    | Handles packets before routing process                                |
| `INPUT`       | `filter`, `mangle` | Handles packets going to local host                                   |
| `FORWARD`     | `filter`, `mangle` | Handles packets forwarded from one network interface to another       |
| `POSTROUTING` | `nat`, `mangle`    | Handles packets being sent to remote system, after the forward filter |
| `OUTPUT`      | `filter`, `mangle` | Handles packets from local host                                       |


| Table    | Chain order                                                   |
| -------- | ------------------------------------------------------------- |
| `filter` | `INPUT` > `OUTPUT` > `FORWARD`                                |
| `nat`    | `OUTPUT` > `PREROUTING` > `POSTROUTING`                       |
| `mangle` | `INPUT` > `OUTPUT` > `FORWARD` > `PREROUTING` > `POSTROUTING` |

### Targets

A target is the action that is applied to a packet when it matches a rule. Targets define what happens to a packet. Think of a target as the outcome or decision after a rule's match criteria are met.

Each rule has a _target clause_, which is essentially a policy to apply to a packet when it matches a rule set in a chain. Example targets include:

User-defined chains or special targets like MASQUERADE and DNAT for NAT purposes.

`ACCEPT`
: Matching packets continue to their destination. Allow the packet to continue through.

`DROP`
: Silently drop the packets. Discard the packet without notifying the sender.

`REJECT`
: Drop packets and send an ICMP error message. Discard the packet and send a notification to the sender.

`LOG`
: Tracks packets as they match rules. Log the packet details, often for troubleshooting or monitoring.

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


Each chain contains tables that defines rules for packet handling.

### How They Work Together

1. Packets enter a chain within a table based on the type of processing required.
2. Rules in the chain are evaluated in order, checking whether the packet matches.
3. If a rule matches, the specified target is applied to the packet.
4. If no rules match, the chain's default policy determines the packet's fate.

For example:

- A packet destined for the local system is processed through the INPUT chain in the filter table.
- If a rule in the INPUT chain matches the packet, its target (e.g., ACCEPT or DROP) is executed.
Would you like a specific example of how these components interact in a real-world scenario?

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