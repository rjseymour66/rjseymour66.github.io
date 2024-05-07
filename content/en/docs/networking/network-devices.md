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

Carrier Sense Multiple Access with Collision Avoidance.

For wireless networks, device checks the radio frequency for _physical carrier sense_ when sending a frame. Very involved process, so actual throughput on wireless LAN is about 1/2 advertised rate.

Process desciption between wireless and wired:
1. Machine sends frame on wireless frequency
2. Frame goes to access point (AP)
   1. If frame is going to another wireless device on the LAN, the frame is forwarded and uses the same process.
   2. If frame is going to a wired station on the LAN:
      1. AP drops the 802.11 MAC header
      2. AP builds new Ethernet MAC header. AP is source addr, gateway is destination addr
      3. LAN router receives frame, performs CSMA/CD
   3. If frame is returned, AP receives them and creates 802.11 MAC header to send back to wireless device

CSMA/CA operation process when Laptop A sends frame to Laptop B:
1. A performs carrier sense to see if anything is being received on its transmitter
   - If traffic is not clear, A decrements the random back-off algorithm counter. Sends when this counter is down to zero.
   - If traffic is clear, A sends the frame
1. Frame goes to AP
2. AP sends ACK to A. All stations are silent until A receives the ACK
3. AP caches the frame with other frames, and sends when its turn
4. AP sends frame from A to B
5. B sends ACK back to AP. All stations are silent until AP receives the ACK

### CSMA/CD

Carrier Sense Multiple Access with Collision Detection.

For wired networks, similar to CSMA/CA but no backoff timer. Instead, listens and sends jam signal so all stations stop transmitting. Computers involved in the collision wait a random amount of time (timer) and resend. So, ethernet lets you only use timers when required.

Device checks wire, called carrier sense:
1. If clear, device transmits. Performs carrier sense entire time
2. If not clear, there is a collision that is detected by both machines through carrier sense
   1. Both devices issue a jam signal to all other devices on the network
   2. Both devices increment a retransmission counter, and abort if it reaches a threshold
   3. Both devices calculate a random amount of time and backoff, if necessary
   4. Random times mean that another collision will not occur.

## DHCP server

Dynamic Host Configuration Protocol servers assign IP addresses to hosts. Easier than static IP assignment.

Process:
1. DHCP server gets IP info req from DHCP client using a broadcast
2. DHCP server is configured by an admin with a pool of addresses that it can use for this purpose
   1. This pool includes addresses that are off limits, called IP exclusions or _exclusion ranges_. Ex: router interface address

- DHCP server has to be on the same segment as the DHCP client bc routers don't forward broadcasts.
- 