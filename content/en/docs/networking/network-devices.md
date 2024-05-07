---
title: "Networking devices"
weight: 40
---

## Connectivity devices 

Devices in this section connect network devices.

### Network Interface Card (NIC)

Installed on the computer to connect (interface) the computer to the network.
- Provides physical, electrical, and electronic connections to network media
- Layer 2 bc it uses the MAC address, which is layer 2 in OSI model
- Almost all are built into the motherboard, but you can still buy them as an expansion card that you plug into the machine
- Usually has lights to indicate ethernet connection and connection speed
  - Activity LED flickers to indicate intermittent transmission and reception of frames
  - There is no universal standard for NIC LEDs

### Hub

In a star topology, the device that connects all the segments of the network together without segmenting the network
- Layer 1 bc it has no intelligence. They are glorified repeaters that cannot recognize frames and data structures
- Broadcasts sent out by any device is sent to all devices connected to the hub.
  - Send anything that they receive on one port to all other ports
- Not recommended for corporate networks bc it is prone to collisions
- Transmissions received on one port are sent to all other ports in the hub
  - This means that if one station sends a broadcast, all others receive it, but only the intended recipient listens and processes
  - Simulates the physical bus that the CSMA/CD standard was based on

### Bridge

Also called a transparent bridge, it is a network device that connects two similar network segments together. 
- Keeps traffic separated on either side of the bridge to break up collision domains
- Layer 2 bc they use MAC addresses to make forwarding decisions
- Software-based version of a switch

### Switch

Connects multiple segments of a network together like a hub, but intelligent:
- Faster, smarter bridge that has more ports
- Default configuration is OK bc it can auto detect speed, duplex, and newer switches
- Can recognize frames
- pays attention to destination MAC address and port that it was received on
- Switch makes each of its port a unique, singular collision domain
- Layer 2 because uses MAC address
- when data is received on a port and knows the destination port, sends data on the destination port
  - If can't figure out the port, sends frame out on every port execpt the port it was received on
- Unmanaged switch: basic device that cannot configure advanced features, like adding an IP address or adding VLANs
- Managed switch: can add IP address and can configure for simple network management protocol SNMP or use special ports for things like VoIP
  
### Router

Device that connects many and sometimes disparate network segments together to create an internetwork.
- Also called:
  - Layer 3 switch
  - multilayer switch
- Makes intelligent decisions about the best way to get network data to its destination
- Gathers info about network performance to make these decisions
- Layer 3 bc they use IP addresses
- You can think of them as CPUs that are dedicated to routing packets
- Can configure their software to perform functions of other type of network devices, like firewalls

#### Interface configurations

Purpose of a router interface is to create and maintain broadcast domains and WAN services connectivity
- VoIP is complicated, but otherwise straightforward
- Requires an IP address on each interface
  - You can configure a specific IP addr or let the router get one from the DHCP server
  - Only do this if you have an IP addr reservation for the router, bc having a random IP address would be hard to manage

### Firewall

A firewall is the network security guard and is the most important thing to implement on the network
- Protects LAN resources from invaders
- prevents all or some of your LAN computers from accessing certain services on the internet
- can filter packets based on rules
- operate on multiple OSI layers, including up to Level 7
- can be standalone black box or software implementation
- two network connections: 
  - public side connected to internet
  - private side connected to network
- second firewall can be used to connect servers and equipment that is both public and private (e.g. web email server)
  - Operates in the demilitarized zone (DMZ)
- _screened subnet_ is the use ofone or more logical screening routers
- can define separate zones:
  - external untrusted zone
  - trusted zone (internal and DMZ, called permiter zone)

### IDS/IPS

Intrusion detection systems (IDSs) and intrusion prevention systems (IPS):
- network security appliances that monitor networks and packets for malicious activity
- IDS monitors and records problems
  - send alarm
  - create rules and remediation
  - drop malicious packets
  - provide malware protection
  - reset connection of offending source hosts
- IPS works in real time and stops threats
  - prevent and block intrusions based on rules you set up

### HIDS, PIDS, and APIDS

Host-based intrusion detection systems
- runs software on a server that detects abnormalities
- monitors system and event logs, not traffic

Protocol-based intrusion detection systems
- Monitor traffic for one protocol on one server

Application protocol-based intrusion detection systems
- Monitors traffic for a group of servers runnign the same app

### Access point

Also known as wireless access points (WAPs)
- Layer 2 hub that accepts wireless clients 
- Wireless client (router) modulates a digital signal to an analog signal, which the AP (DSL/cable modem) can read and demodulate back to a digital signal
- Creates one collision domain and runs half-duplex (which is why its kind of a hub)
- wireless will never have same throughput, security, and consistency as wired

### Wireless range extender

Radios and antenneas that operate in the same frequency or channel as WLAN technology, and receive the signal and transmit it again
- extends the reach of your WLAN
- should be placed so there is 15% coverage overlap

### Wireless LAN controller

Also called WLC
- Centralized WiFi configuration controller 
- lets you configure complete netowrk on a single device and push the configurations out to the wifi access points
  - APs are called lightweight APs bc controller does all the work
- the APs tunnel user data back to the controller

### Load balancer

A load balancer takes from a router packets that are addressed to a single IP address and sends them to multiple machines bc sometimes a single server often can't handle all requests
- publishes a virtual IP address to a domain to receive incoming traffic
- LB has pool of servers that it distributes connections to based on metrics such as:
  - round robin
  - least number of connections
  - response time
  - weighted percentage
- health checks make sure servers are operational
- add capacity by manually or automatically scaling based on workloads
- you can set rules to determine how traffic is networked:
  - least load
  - fault tolerance/redundancy
  - fastest response times
  - dividing up requests

## Contention methods

Wired and wireless environments that share a collision domain (ex: a hub or access point) have potential for frames from multiple devices colliding. This destroys packets.

A _contention method_ is a way to manage access to the medium to prevent or recover from collisions.

### CSMA/CA

### CSMA/CD