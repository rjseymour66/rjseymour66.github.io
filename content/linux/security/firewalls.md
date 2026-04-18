+++
title = 'Firewalls'
date = '2025-09-07T19:05:38-04:00'
weight = 10
draft = false
+++



A firewall controls which network ports are accessible and from where. When you start a
service, open the corresponding port in the firewall. Open only the ports your server
requires: for example, ports 80 and 443 for a web server.

Ubuntu includes Uncomplicated Firewall (UFW), a simplified interface to `iptables`. Use
UFW for most tasks. UFW is inactive by default — configure your rules before enabling it.

`iptables` is the lower-level packet-filtering engine that UFW wraps. It gives you full
control over packet flow but requires more detailed configuration.

## ufw

Install UFW and verify it is inactive before configuring rules:

```bash
sudo apt install ufw            # install the package
sudo ufw status                 # confirm UFW is inactive
```

Allow or deny traffic by port, protocol, or source address:

```bash
ufw allow from 192.168.1.158 to any port 22     # allow SSH from a specific IP
ufw allow from 192.168.1.0/24 to any port 22    # allow SSH from an entire subnet
ufw allow 80                                    # allow all traffic to port 80
ufw allow 443                                   # allow all traffic to port 443
ufw allow ssh                                   # allow SSH by service name
ufw deny http                                   # deny HTTP traffic
```

After configuring your rules, enable UFW:

```bash
sudo ufw enable
```

## iptables

`iptables` manages the Linux Netfilter packet-filtering engine. It organizes packet
handling around three core concepts:

- **Rules** define matching criteria and the action to take when a packet matches.
- **Chains** group rules and evaluate them in sequence.
- **Tables** group chains by purpose.

When a packet matches a rule, iptables applies the rule's action and stops evaluating
further rules in that chain.

### Rules

A rule pairs matching criteria with a target action. When a packet meets all criteria,
iptables applies the target and stops evaluating the chain.

A rule has three parts:

- **Matching criteria:** conditions a packet must meet. Common criteria include:
  - **Source/destination address**: matches packets from or to a specific IP address or range
  - **Protocol**: matches packets using a specific protocol, such as TCP, UDP, or ICMP
  - **Port**: matches packets directed to a specific port or port range
  - **Interface**: matches packets arriving on or leaving a specific network interface
  - **State**: matches packets based on connection tracking state, such as `NEW` or `ESTABLISHED`
- **Target:** the action to apply: `ACCEPT`, `DROP`, `REJECT`, or `LOG`.
- **Chain placement:** the chain the rule belongs to, which determines when it is evaluated.

Rules in a chain are evaluated in order. The first matching rule wins.

#### Example

Allow incoming TCP traffic on port 80 from interface `eth0`:

```bash
iptables -A INPUT -p tcp --dport 80 -i eth0 -j ACCEPT
```

| Flag         | Meaning                            |
| :----------- | :--------------------------------- |
| `-A INPUT`   | Append the rule to the INPUT chain |
| `-p tcp`     | Match TCP packets                  |
| `--dport 80` | Match packets destined for port 80 |
| `-i eth0`    | Match packets arriving on `eth0`   |
| `-j ACCEPT`  | Accept matching packets            |

### Tables

A table groups chains by the type of packet processing they perform. If a rule does not
specify a table, it applies to the `filter` table by default:

| Table      | Description                                                                      |
| :--------- | :------------------------------------------------------------------------------- |
| `filter`   | Basic packet filtering. Decides whether to accept or drop packets. (Default)     |
| `mangle`   | Packet modification. Changes headers or flags before the packet exits the chain. |
| `nat`      | Network address translation. Alters source or destination addresses.             |
| `raw`      | Pre-connection tracking. Marks packets that should bypass connection tracking.   |
| `security` | Mandatory access control. Used for SELinux-related decisions.                    |


### Chains

A chain is an ordered list of rules within a table. Packets are evaluated against each
rule in sequence. If no rule matches, the chain's default policy applies (`ACCEPT` or `DROP`).

There are two types of chains:

- **Built-in chains**: predefined by iptables, such as `INPUT`, `OUTPUT`, and `FORWARD`.
- **User-defined chains**: custom chains you create for modularity and reuse.

For most configurations, `INPUT`, `OUTPUT`, and `FORWARD` are sufficient:

| Chain         | Tables             | Description                                                      |
| :------------ | :----------------- | :--------------------------------------------------------------- |
| `PREROUTING`  | `nat`, `mangle`    | Processes packets before routing                                 |
| `INPUT`       | `filter`, `mangle` | Processes packets destined for the local host                    |
| `FORWARD`     | `filter`, `mangle` | Processes packets forwarded between interfaces                   |
| `OUTPUT`      | `filter`, `mangle` | Processes packets originating from the local host                |
| `POSTROUTING` | `nat`, `mangle`    | Processes packets after forwarding, before they leave the system |

Each table evaluates its chains in a fixed order:

| Table    | Chain order                                                   |
| :------- | :------------------------------------------------------------ |
| `filter` | `INPUT` > `OUTPUT` > `FORWARD`                                |
| `nat`    | `OUTPUT` > `PREROUTING` > `POSTROUTING`                       |
| `mangle` | `INPUT` > `OUTPUT` > `FORWARD` > `PREROUTING` > `POSTROUTING` |

### Targets

A target is the action iptables applies when a packet matches a rule. Common targets:

| Target       | Action                                                              |
| :----------- | :------------------------------------------------------------------ |
| `ACCEPT`     | Allow the packet to continue to its destination.                    |
| `DROP`       | Silently discard the packet without notifying the sender.           |
| `REJECT`     | Discard the packet and send an ICMP error to the sender.            |
| `LOG`        | Log the packet details to syslog without affecting its flow.        |
| `ULOG`       | Extended logging via netlink to userspace programs.                 |
| `REDIRECT`   | Redirect the packet to a local port, typically for proxying.        |
| `RETURN`     | Stop processing the current chain and return to the calling chain.  |
| `MIRROR`     | Swap the source and destination addresses and send the packet back. |
| `QUEUE`      | Pass the packet to a userspace program for processing.              |
| `MASQUERADE` | Rewrite the source address for NAT (used with dynamic IPs).         |
| `DNAT`       | Rewrite the destination address for NAT.                            |

### Commands

Canonical reference: [IptablesHowTo](https://help.ubuntu.com/community/IptablesHowTo)

Manage rules and chains:

```bash
iptables -L [chain]                                     # list rules for a chain (all chains if omitted)
iptables -S [chain]                                     # list rule details for a chain
iptables -F <chain>                                     # flush all rules from a chain
iptables -P <chain> <target>                            # set the default policy for a chain
iptables -A <chain> -i <interface> -j <target>          # append a rule to a chain
iptables -I <chain> <index> <rule>                      # insert a rule at a specific position
iptables -D <chain> <rule>                              # delete a rule from a chain
iptables -R <chain> <index> <rule>                      # replace a rule at a specific position
iptables [-t <table>]                                   # target a specific table (default: filter)
```

Common options:

```bash
-s <source-ip>      # match packets by source address
-d <dest-ip>        # match packets by destination address
--sport <port>      # match packets by source port
--dport <port>      # match packets by destination port
-i <interface>      # match packets arriving on an interface
-o <interface>      # match packets leaving on an interface
-p <protocol>       # match packets by protocol
-j <target>         # apply this target to matched packets
```

View current rules:

```bash
sudo iptables -L
```

Log and drop packets:

```bash
iptables -N LOGGING                                             # create a LOGGING chain
iptables -A INPUT -j LOGGING                                    # send unmatched INPUT packets to LOGGING
iptables -A OUTPUT -j LOGGING                                   # send unmatched OUTPUT packets to LOGGING
iptables -A LOGGING -m limit --limit 2/min \                    # log to syslog
    -j LOG --log-prefix "IPTables-Dropped: " --log-level 4      
iptables -A LOGGING -j DROP                                     # drop all packets in LOGGING
```