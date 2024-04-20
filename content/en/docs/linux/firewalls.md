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
sudo firewall-cmd --set-default-zone=home
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
sudo firewall-cmd --list-services --zone=dmz
ssh

# assign service to zone
sudo firewall-cmd --add-service=<service> --zone=<zone>

# disable all network traffic
sudo firewall-cmd --panic-on

# re-enable all network traffic
sudo firewall-cmd --panic-off
```


## iptables

