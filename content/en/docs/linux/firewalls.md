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

