---
title: "Firewalls"
weight: 170
---

Firewalls prevent the spread of unwanted, unauthorized, or malicious network traffic. A firewall can be any of the following:
- hardware device
- software application
- network-based
- host-based
- network-layer filter
- application-layer filter

## Access control

Firewalls implement access control
- ACL in a firewall identifies which netowrk packets are allowed in or out. This is _packet filtering_.
- ACL reviews the packet's control data and other network data, including:
  - source address
  - destination address
  - network protocol
  - inbound port
  - output port
  - network state
- After ACL reviews, the firewall rules do one of the following with the packet:
  - Accept
  - Reject (sends msg back to sending app)
  - Drop (sends no msg back to sending app)
  - Log

### Firewall logs

Firewall logs are critical and can be monitored, provide alerts, take needed actions to protect a system:

Resources:
- [Graylog](https://graylog.org/) can process logs in real time
- [NIST Special Publicatin 800-92, Guide to Computer Security Log Management](https://csrc.nist.gov/pubs/sp/800/92/final)


### Services and ports

All services and their corresponding ports are listed in `/etc/services`
- Used by services and utilities
- Listed in the following format:
  ```
  ServiceName PortNumber/ProtocolName [Aliases]
  ```
- Ports 1-1023 are _privileged ports_, which means only a super user can run a service on them.

### Stateful and stateless
Firewalls can have state or be stateless.

Stateless
: Fast because they only focus on the packet and decide what to do based on the firewall's ACL rules. Is vulnerable to attacks that spread themselves across multiple packets. Does not consider any of the following:
  - network connections
  - network status
  - data flows
  - traffic patterns
  Also, ACL rules are static, so you must restart the firewall if you change them.

Stateful
: Treats packets as a team, can determine if the packets are fragmented. Tracks network connections and status. Stateful firewalls keep network info in memory in the connection table, so it can make faster decisions for established connections' individual packets. Table is slow to create but fast for existing connections. This slowness makes it vulnerable to DDoS attacks.


## Firewall technologies

`netfilter` is embedded in the Kernel to provide packet filtering services. It is used by the following firewall software and utilities:
- `firewalld`
- Uncomplicated Firewall (UFW)
- `nftables`
- `iptables`

[`netfilter`](https://netfilter.org/) is maintained by the same group as `iptables` firewall software.

### firewalld

- Supports IPv4 and IPv6
- Called the "dynamic firewall daemon" bc you can change an ACL rule without restarting
- Rules are loaded through the **D-Bus interface**
  - D-Bus is message bus daemon
  - provides communication services to any two apps on a systemwide or per-session basis
  - `firewalld` uses a `dbus` Python lib module to integrate D-Bus services

`iptables` should not run alongside `firewalld`. Always prefer `firewalld` and never use `iptables`. To check if there is a conflict between the two, enter the following command:

```bash
systemctl show --property=Conflicts firewalld
Conflicts=ipset.service iptables.service shutdown.target ebtables.service ip6tables.service nftables.service
```

```bash
# start the service
systemctl enable --now firewalld

# check status
systemctl status firewalld

# restart
systemctl restart firewalld
```

### Environments

There is a `runtime` and `permanent` environment:
- `runtime`: what is actively employed during your session. Good for testing.
- `permanent`: stored in the configuration files, loaded when system boots. Good for production.

```bash
# make runtime commands permanent
firewall-cmd --runtime-to-permanent

# make individual rules permanent
firewall-cmd <command> --permanent
```

### Zones

`firewalld` groups network traffic into _zones_. A zone is a predefined rule set that has its own configuration file, called a _trust level_.
- Group traffic by network interface or source address range, etc
- Each network connection can be a member of ONE zone at a time

Configuration file locations:
- Default files: `/usr/lib/firewalld/zones/`
- Custom files: `/etc/firewalld/zones/`

```bash
ls -A1 /usr/lib/firewalld/zones/
block.xml       # accepts only traffic that originated on the system
dmz.xml         # similar to public but used in location's demilitarized zone, with limited access to internal network
drop.xml        # drops all incoming network packets and only allows outbound network connections
external.xml    # similar to public but used on external networks
home.xml        # similar to work but used in home setting where other network systems are trusted
internal.xml    # similar to work but used on internal networks where other network systems are trusted
nm-shared.xml   # ???
public.xml      # accepts only selected network connections
trusted.xml     # accepts all network connections
work.xml        # accepts only selected network connections
```

### Services

A service is a predefined configuration set for a particular offered system service, such as DNS. The configuration set might include info such as:
- list of ports
- protocols

```bash
ls -1 /usr/lib/firewalld/services/
amanda-client.xml
amanda-k5-client.xml
amqps.xml
amqp.xml
apcupsd.xml
...
```


### firewall-cmd

View and interact with `firewalld` configuration settings:

```bash
# get zones
firewall-cmd --get-zones
block dmz drop external home internal nm-shared public trusted work

# get default zone
firewall-cmd --get-default-zone
public

# set default zone
firewall-cmd --set-default-zone=home
success
firewall-cmd --get-default-zone
home


# get active zones and traffic grouping
firewall-cmd --get-active-zones
public
  interfaces: enp0s3

# get services
firewall-cmd --get-services
RH-Satellite-6 RH-Satellite-6-capsule amanda-client amanda-k5-client amqp amqps apcupsd audit bacula bacula-client bb bgp
...

# get services by zone
firewall-cmd --list-services --zone=dmz
ssh

# assign service to zone
firewall-cmd --add-service=<service> --zone=<zone>

# disable all network traffic
firewall-cmd --panic-on

# re-enable all network traffic
firewall-cmd --panic-off
```


## iptables

The `iptables` firewall uses a series process called _chains_ that determines the path each packet takes as it enters the system and gets to the correct application. It also handles the path as the application sends data outside the system.

RHEL `iptables` are stored at these locations:
- `/etc/sysconfig/iptables`
- `/etc/sysconfig/ip6tables`

Debian `iptables` are stored at these locations:
- `/etc/iptables/rules.v4`
- `/etc/iptables/rules.v6`

`iptables` rules are not persistent, you must run the `iptables-save` command. To revert, use `iptables-restore`.

| Chain  | Description |
|--------|--|
| `PREROUTING`  | Handles packets before routing process |
| `INPUT`       | Handles packets going to local system |
| `FORWARD`     | Handles packets forwarded to remote system |
| `POSTROUTING` | Handles packets being sent to remote system, after the forward filter |
| `OUTPUT`      | Handles packets from local system |

Each chain contains tables that defines rules for packet handling:

| Table    | Description |
|----------|--|
| `filter`   | applies rules to allow or block packets from exiting the chain |
| `mangle`   | applies rules to change packet features before exiting the chain |
| `nat`      | applies rules to change packet address exiting the chain |
| `raw`      | applies a `NOTRACK` setting on packets that should not be tracked |
| `security` | applies mandatory access rules |

Each chain has a policy value that defines how a packet is handled by default, when no rules apply to the packet:
- `ACCEPT`: Pass packet to next chain
- `DROP`: Don't pass the packet to the next chain

### iptables

View and alter chains in `iptables`:

```bash
iptables [OPTION]
-L [chain] # list rules for the chain, all rules if no chain
-S [chain] # list rules details for the chain, all rules details if no chain
[-t table] # apply command to table. If no table, applied to filter table
-A chain rule #  add this new rule to this chain
-I chain index rule # insert this new rule to this chain at this index location
-D chain rule # delete this rule from this chain
-R chain index rule # remove this rule from this chain at this index location
-F [chain] # remove (flush) all rules from this chain
-P chain policy # sets as default this policy for this chain

# common options
-d address # apply rule to packets w dest address  
-s address # apply rule to packets w source address
-i name # apply rule to packets coming through name network interface
-o name # apply rule to packets going out through name network interface
-p protocol # apply rule to packets using this protocol
-j target   # apply the target action to the selected packets
            # target values:
            # ACCEPT
            # DROP
            # REJECT
# list all rules
iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination 
```


## nftables

`nftables` syntax is less complicated than `iptables` but uses the same concepts, such as tables, chains, policies, and rules.

> This is all unnecessarily complicated, so I will learn it if I ever have the need. Prefer `firewalld` for Red Hat-based distros.

```bash
# list nft tables
nft list tables
table ip filter
table ip6 filter
table bridge filter
table ip security
table ip raw
table ip mangle
table ip nat
table ip6 security
table ip6 raw
table ip6 mangle
table ip6 nat
table bridge nat
table inet firewalld
table ip firewalld
table ip6 firewalld

# add new table
nft add table ip rocky

# view table
nft list table ip rocky
table ip rocky {
}

# create a chain that contains the rules for firewall definition
nft 'add chain ip rocky test { type filter hook input priority 0; policy drop; }'
```

## UFW

- Default firewall service for Ubuntu.
- Cannot run with `firewalld` or `iptables`
- If you specify a service name, `ufw` checks the `/etc/services` file for the port and other protocol information
- Default rules are stored in `/etc/ufw/`
- User-added rules are stored in `/etc/ufw/user.rules`

You do not control `ufw` with `systemd`, you use the `ufw` command:

```bash
# start and set to start at boot
ufw enable

# stop and disable at boot
ufw disable

# disable and reset to system default
ufw reset

# reload service (not necessary, changes automatically take effect)
ufw reload

# get current state
ufw status [verbose]

# verbose status to figure out configuration
ufw status verbose
Status: active      # active or disabled, both on boot
Logging: on (low)   # off, low, medium, high, full. low is default
Default: deny (incoming), allow (outgoing), disabled (routed)   # default policies for packets
New profiles: skip  # accept, drop, reject, or skip. default policy for automatically loading new profiles.

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere                  
22/tcp (OpenSSH)           ALLOW IN    Anywhere                  
22/tcp (v6)                ALLOW IN    Anywhere (v6)             
22/tcp (OpenSSH (v6))      ALLOW IN    Anywhere (v6)

# default policy location
cat /etc/default/ufw 
# /etc/default/ufw
#

# Set to yes to apply rules to support IPv6 (no means only IPv6 on loopback
# accepted). You will need to 'disable' and then 'enable' the firewall for
# the changes to take affect.
IPV6=yes

# Set the default input policy to ACCEPT, DROP, or REJECT. Please note that if
# you change this you will most likely want to adjust your rules.
DEFAULT_INPUT_POLICY="DROP"
...
```

### Common arguments

- allow _Identifiers_
- deny _Identifiers_
- reject _Identifiers_
- delete _RULE_ | _NUM_
- insert _NUM RULE_
- default _POLICY DIRECTION_
- logging _LEVEL_

### Simple syntax

Designate the rule using only the port number or service name:

```bash
# add ACL rule
ufw allow 22/tcp

# check status
ufw status
```

### Full syntax

Full syntax gives you more options:

| Setting | Description |
|---------|-------------|
| comment "<string>" | Displays <string> for rejected traffic |
| in | Applies to incoming only |
| out | Applies to outgoing only |
| proto _protocol_ | Applies to _protocol_ |
| port _port#_ | Applies to _port#_ |
| from _source_ | Applies rule to traffic from _source_ |
| on _interface_ | Applies rule to traffic on this network _interface_ |
| to _destination_ | Applies rule to traffic going to _destination_ |

```bash

ufw deny from 192.168.0.0/24 to any port 80


ufw show added

# deny packets access to port 80 from this class C subnet
sudo ufw deny from 192.168.0.0/24 to any port 80
Rule added

# confirm  
sudo ufw show added
Added user rules (see 'ufw status' for running firewall):
ufw allow 22/tcp
ufw allow OpenSSH
ufw deny from 192.168.0.0/24 to any port 80

# view rules
ls /etc/ufw/
after6.rules  after.rules     before6.rules  before.rules  ufw.conf     user.rules
after.init    applications.d  before.init    sysctl.conf   user6.rules

# get rule number (to delete rule)
ufw status numbered
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere                  
[ 2] OpenSSH                    ALLOW IN    Anywhere                  
[ 3] 80                         DENY IN     192.168.0.0/24            
[ 4] 22/tcp (v6)                ALLOW IN    Anywhere (v6)             
[ 5] OpenSSH (v6)               ALLOW IN    Anywhere (v6)             

# delete the rule by number
ufw delete 3
Deleting:
 deny from 192.168.0.0/24 to any port 80
Proceed with operation (y|n)? y
Rule deleted
```

### Application profiles

`ufw` stores application information in `/etc/ufw/applications.d`
- Do not modify profiles in `/etc/ufw/applications.d`, create a new `/etc/ufw/custom.d` to prevent ufw from missing package updates

```bash
# view profile for apps or daemons
ls -1 /etc/ufw/applications.d/
cups
openssh-server

# view profile for apps or daemons with ufw
ufw app list
Available applications:
  CUPS
  OpenSSH

# get application details
ufw app info OpenSSH
Profile: OpenSSH
Title: Secure shell server, an rshd replacement
Description: OpenSSH is a free implementation of the Secure Shell protocol.

Port:
  22/tcp

# add new or update existing profile
ufw app update all
Rules updated for profile 'OpenSSH'
Skipped reloading firewall
```

## Forwarding IP packets

You can forward IP packets from one network interface to the other by setting a few values in the kernel, and then using routing tables to determine which network interface to forward to:

```bash
# set kernel properties
sysctl -w net.ipv4.ip_forward=1
sysctl -w net.ipv6.conf.all.forwarding=1

# verify changes
cat /proc/sys/net/ipv4/ip_forward
0
cat /proc/sys/net/ipv6/conf/all/forwarding 
0

# check routing table
ip route list
default via 10.0.2.2 dev enp0s3 proto dhcp metric 100 
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100 
169.254.0.0/16 dev enp0s3 scope link metric 1000 
```

## Dynamically setting rules

Intrusion detection systems (IDSs) are software monitors the network and applications running on the system. Some let you dynamically change rules so that these attacks are blocked.

### DenyHosts

> Some distro repos claim this tool is no longer being developed, so install with caution.

A Python script that helps protect against brute-force attacks coming through OpenSSH. Configure as a service or cron job.

### Fail2Ban

Monitors system logs, looking for repeated failures from the same host.

### IPset

Lets you create a named set of IP addresses, network interfaces, ports, MAC addresses, or subnets. You can manage the groupings through your firewall or any app that supports IPset.

