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
- Application layer protocol
  - uses UDP ports 67 and 68
- DHCP server has to be on the same segment as the DHCP client bc routers don't forward broadcasts.
- Can configure the router to forward DHCP requests directly to the DHCP server
- DHCP server can be configured with a reservation list so a host always gets the same IP address
- reservation is made using the router interface MAC address. Sometimes called a MAC reservation
- 


Process:
1. DHCP server gets IP info req from DHCP client using a broadcast
2. DHCP server is configured by an admin with a pool of addresses that it can use for this purpose
   1. This pool includes addresses that are off limits, called IP exclusions or _exclusion ranges_. Ex: router interface address


_Scope options_ define IP configuration for hosts on a specific subnet.
  - _Server options_ provide IP information for all scopes configured on the server
  - So if you have a single DNS server for the network, configure server options with DNS server info. The DNS server info shows up automatically in all scopes configured on the server

Scope options also define what info a DHCP server can provide to DHCP clients:
- TTL
- DNS server
- TFTP
- Lease time. Cna indicate a DHCP problem or if that server is no longer giving IP addresses to hosts

### DHCP Relay

If you need to provide IP addresses from a DHCP server to hosts that are not on the same LAN as the DHCP server, you can configure your router interfaece to relay or forward DHCP client requests 
- If hosts off a router can't access DHCP servers bc the router doesn't forward broadcast addresses, you can configure a router interface to forward DHCP requests to a DCHP server

### IPAM

IP address management software that plans, tracks, and manages IP addresses.

## Specialized devices

There are several devices that are not directly connected to a network that actively participate in moving network data.

### Multilayer switch

MLS switches on OSI layer 2 like a normal network switch but also provides routing
- 24-port MLS operates at layer 3 (routing) and provides 24 collision domains, which a router can't do
- MLS uses application-specific integrated circuit (ASIC) hardware, instead of a microprocessor used in standard routers.

### Domain name system server

Phone book of the internet, performs _name resolution_ to to find the IP address for any given hostname.
- hostname is the name of a device that has a specific IP address
  - Part of fully qualified domain name (FQDN), which consists of hostname and domain name
- Application layer protocol
- DNS queries are made on UDP on port 53
- name resolution is performed in several ways:
  - hosts file on the host
  - request broadcast on local network
  - DNS
- Domains are hierarchical tree structure. Examples:
  - .com - commercial org
  - .edu - educational establishment
  - .gov - branch of the US gov't
  - .int - international org, such as NATO
  - .mil - branch of US military
  - .net - network organization
  - .org - nonprofit
- DNS zones:
  - forward: resolve names to IP addresses
  - reverse: resolve IP addresses to names
- CLients cache DNS server replies to reduce the number of DNS lookups
  - Each reply has a field called TTL, or time to live, which tells the client how long to cache the DNS info before requesting again

```bash
# get DNS info for www.wiley.com
nslookup www.wiley.com
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
www.wiley.com	canonical name = www.wiley.com.cdn.cloudflare.net.
Name:	www.wiley.com.cdn.cloudflare.net
Address: 104.18.42.79
Name:	www.wiley.com.cdn.cloudflare.net
Address: 172.64.145.177
Name:	www.wiley.com.cdn.cloudflare.net
Address: 2606:4700:4400::ac40:91b1
Name:	www.wiley.com.cdn.cloudflare.net
Address: 2606:4700:4400::6812:2a4f

# get CNAME info for wiley.com
$ nslookup -q=cname www.wiley.com
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
www.wiley.com	canonical name = www.wiley.com.cdn.cloudflare.net.

Authoritative answers can be found from:
www.wiley.com.cdn.cloudflare.net	internet address = 104.18.42.79
www.wiley.com.cdn.cloudflare.net	internet address = 172.64.145.177

# get mail exchange info from wiley.com
$ nslookup -q=mx www.wiley.com
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
www.wiley.com	canonical name = www.wiley.com.cdn.cloudflare.net.

Authoritative answers can be found from:
cloudflare.net
	origin = ns1.cloudflare.net
	mail addr = dns.cloudflare.com
	serial = 2338653898
	refresh = 10000
	retry = 2400
	expire = 604800
	minimum = 1800
www.wiley.com.cdn.cloudflare.net	internet address = 104.18.42.79
www.wiley.com.cdn.cloudflare.net	internet address = 172.64.145.177
```

#### DNS record types

| Record Type | Description |
|---|---|
| A | Address record. Returns the IP address of the domain. |
| AAAA | Maps hostnames to an IPv6 host address |
| TXT (DKIM) | Domain Keys Identified Mail. Provides authentication of mail sent and received by teh same email system. Also prevents spam. |
| SRV | DNS service or generalized service location record. Specifies port and IP address. |
| CAA | Certificate Authority Authorization, which allows domain name owners to specify authorized CAs. |
| CNAME | Canonical name records. Alias one domain name to another, such as firstlastname.com to lastname.com. Also, if a web server has hostname `www` and you wnat to allow FTP access to parts of the machine, you can add the name `ftp` to the server.|
| SOA | Start of authority. Provides administrative informaton about the domain or zone, such as admin email, last date domain was updated, and refresh intervals. |
| PTR | Pointer record. Used for reverse DNS loookup, which returns the domain name when given the IP address |
| MX | Mail exchanger record specifes how email messages should be routed. |
| NS | Name Server. Authoritive DNS server for the domain. These servers store and manage the offical DNS records. Other servers are intermediaries, which forward and cache DNS queries to improve performance. |

### Dynamic DNS

DNS used to be manually added into a DNS server, and edited when changes occurred. Now DNS uses dynamic assignment and works with the DHCP function. Hosts register their names with the DNS server when they receive IP address configuration from the DHCP server.
- MX and CNAME records must be created manually.

### Internal and external DNS

DNS servers can be located in the screened subnet/DMZ or inside the intranet:
- In the DMX, DNS server only contains records of devices in the DMZ

### Third-party or cloud-hosted DNS

Cloud providers provide DNS as a service.

### DNSSEC

Domain Name System Security Extensions, a suite of new extensions created by Extension Mechanisms for DNS (EDNS). EDNS was created to expand the size of DNS protocol params
- THis led to the IETF to create a new secure protocol called DNSSEC
- Provides cytopgraphic authentication, authenticated denial of existence, and data integrity

### DNS over HTTPS, DNS over TLS

DNS over HTTPS (DoH)
: Sends encrypted DNS info over HTTPS connections. DNS queries and responses are encrypted, which prevents attackers from altering the traffic.
  Uses TCP on port 443, which is used by other HTTPS traffic, so it is harder to distinguish (and hack)
  Firefox uses this by default.

DNS over TLS (DoT)
: Standard that encrypts DNS queries to provide security and privacy.
  Uses TCP on port 853 and UDP on port 8853.


### Network time protocols

NTP synchoronizes clocks on networking devices and computers on a network.
- For distributed computing that require tasks are processed in the correct sequence and recorded properly
- Needed for security and log tracking to correlate based on time
- UDP on port 123

Important terms:

Stratum
: Stratum levels are how accurate the time source is. Nuclear clock is stratum 0, and a stratum 1 takes its time from stratum 0, stratum 2 from stratum 3, and so on.

Clients
: Clients query NTP servers to set their clocks. After synchronized, it checks every 10 minutes to make sure it is still in sync.

Servers
: Can be specialized hardware that syncs to a stratum 0 device.

### Network Time Security (NTS)

Cryptographic security for client-server mode of NTP to make sure you don't receive your time from an unauthorized server. Does not increase latency in NTP time sychronization.

### Proxy server

A type of server that handles its client-machine reqs by forwarding them to other servers. This allows granualr control over the traffic between the local LAN and the internet.
- Application layer of OSI
- Receives a req, connects to the server that can fulfill the request
- Might modify the cleint request or server response
- Caches (remembers) which server provides which service
- Limits the types of sites that users on a LAN can access


Forward proxy
: takes client reqs and sends them to the internet (discussed in this section)

Reverse proxy
: taskes requests from the internet and forwards them to servers in the internal network

Two main types of forward proxy servers:

Web proxy server
: Used to create a web cache that "remembers you" and loads the site faster, recalls your personal info such as username, billing, shipping, etc.

Caching proxy server
: Speeds up the network service request by recovering info from a client's earlier request. Enchance network performance by keeping local copy often-requested resources to minimize bandwidth use.


### Encryption

Many network devices perform encryption, but it is an intensive task so it makes sense to offload it onto dedicated machines.
- Sometimes called 'encryption gateways'
- Can be in line with server or local network, encrypting/decrypt all traffic
- Can function as an application server that encrypts any file sent to it within a network

### Content filter

Scans content and filters out specific content or content types
- Dedicating a service to this offloads the work from servers or routers
- Usually more control over dedicated filters
- Email is a good example. Might want to filter out spam. Also blocked websites.

### Analog modem

A device that modulates analog carrier signals to encode digital info, and demodulates the signal to decode the transmitted information.
- Info is sent over telephone lines. One end connected to a phone line and one connected to a computer or modem bank.
- Operates at layer 1, like a repeater

### Packet sharper

Traffic management technique that delays some or all packets to bring them into compliance with your company's traffic profile.
- Optimizes performance
- Improves latency
- increases usable bandwidth for some packets while delaying others

### VPN concentrator/headend

A device that accepts multiple VPN connections from remote locations.
- Can be performed by a router or server, but a dedicated device increases performance

### Media converter

Used when you need to convert from one type of cabling to another type.

### VoIP PBX

Private branch exchange is a telephone switch that is on the customer premises with a direct connection to the telecommunication provider's switch.
- Performs call routing within the internal phone system, which allows multiple internal lines with minimal outside lines.
  - For example, can have 2 'outside' lines that route to 50 'inside' lines.

### VoIP endpoint

Desktop or wireless phone systems that are part of networks where data and voice traffic are combined.
- Allows more freedome in location and installation of these systems

### NGFW/Layer 7 firewall

Next-generation firewalls address traffic inspectino and application awareness shortcomings of the traditional stateful firewall without losing performance.
- Unified thread management (UTM) devices address some issues but use separate components to examine traffice
  - This means that the same packet might be inspected multiple times
  - NGFWs examine packets one time during the deep packet inspection phase
- NGFWs are application aware, so they can distinguish between app traffic and traffic coming in through ports

### VoIP gateway

Network device that converts voice and fax calls between an IP netwrok and a public switched telephone network (PSTN) in real time.
- Usually have 1 ethernet and 1 phone port
- Uses various protocols:
  - Media Gateway Control Protocol (MGCP)
  - Session Initiation Protocol (SIP)
  - Lightweight Telephony Protocol (LTP)

### Cable modem

Allows voice, video, and data to connect from home or small business to providers network.
- Data Over Cable Service Interface Specifications (DOCSIS) standards allow voice and data share cable with standard cable TV

### DSL modem

Digital subscriber Line used by traditional phone companies that have twisted-pair copper as local connection to homes and businesses
- Allows for voice, video, and data to travel on copper line as a high-frequency carrier above standard voice frequencies.

## Networked devices

VoIP phones
: Low bandwidth, sensitive to delay and jitter. Requires some sort of QoS.

Printers
: Printers can connect through Ethernet, and they have NIC cards that can connect to the network so machines can access it.

Physical access control devices
: Access control systems open doors and gates, in an office, for example. They are connected to an authorization server on the network that can connect back to a service like MS Active Directory.

Cameras
: Use TCP/IP and send video feeds to a central server for processing and recording.

Heating Ventilation, and AC (HVAC) sensors
: Buildings have intelligent HVAC systems that use sensors to monitor and control AC and heat. 

IoT
: Wearable devices, digital home assistants, doorbells, refrigerators, lights, etc.

Industrial control systems
: ICS tech uses sensors for monitoring and controlling anything from powergrids to machines on a factory floor. Helps flag problems and prevent downtime.

  The supervisory control and data acquisition (SCADA) architecture is an industry standard for monitoring and collecting industrial data that includes computers, sensors, and networks that display graphical data about monitor systems.

## SOHO network

Small office home office.

### Routers
Layer 3 machines (routers) need to locate networks

Routers can perform the following:
- Routers (also called layer 3 switches) break up broadcast and collision domains, hubs do not. Routers do not forward broadcast messages.
- Routers filter the network based on layer 3 info, like an IP addr
- Packet switching
- Packet filtering
- Internetwork communication
- Path selection
- Each interface on a router represents a separate network
  - each interface has a unique network number
  - each host on a network connected to the router uses the same network number

Things to remember:
- Do not forward broadcast or multicast packets, by default
- Use logical address in Network layer header to determine the next hop router to forward packets to
- Use access lists created by an admin that control security on types of packets allwed to enter or exit the interface
- Can provide layer 2 bridging functions, as needed
- Layer 3 devices connect VLANs
- Can provide QoS for specific types of network traffic

### Switches

- Layer 2 machines (switches) need to locate devices
- Essentially a multiport bridge, but bridges only have 2-4 ports. Switches can have hundreds.
  - Commonly called _bridges_. Bridges are outdated technology that act the same as switches
- Cannot create networks, they optimize a network LAN performance and create more bandwidth for users
- They switch frames from one port to another
- Cannot breakup broadcast domains
- Can break up collision domains
  - Switches segment networks, but cannot isolate broadcast or multicast packets
  - Each used switch port equals a collision domain

#### Data link layer switches

- Layer 2 switching is called hardware-based bridging bc it uses an application-specific integrated circuit (ASIC) that provide low latency
- latency is measure from when a frame enters a port to when it exits
- Process:
  - Read each frame as it passes through
  - Puts the source hardware addr in a filter table to track which port the frame was received on. The filter table helps the machine determine the location of the specific sending device. similar to routing table.
  - Forwards frames to the segment where the destination hardware address is located. If located on the same segment, the layer 2 device blocks the frame from leaving. This forwarding and blocking is called _transparent bridging_.
  - If the frame has a destingation address that isn't in the filter table, it forwards the frame to all connected segments
    - If the device replies, it is added to the filtering table
    - If the address is a broadcast device, the switch forwards all broadcasts to every connected segment
- Better than hubs bc each switch port is its own collision domain and each connected device can transmit simultaneously
  - A hub creates one large collision domain
  - A hub allows only one device per network to communicate at a time

### Hubs

Multiple-port repeater, which creates a single collision domain:
- it receives a digital signal
- reamplifies or regenerates it
- forwards the digital signal out all active ports without looking at the data

## Environmental considerations

Temperature
: Routers, switches, and hubs need a cool area to operate correctly. In high temps, servers reboot and CPUs start overworking.

Humidity
: Air can't be too damp or too dry. Has to be just right. Dry creates static electricity, which can fry some electrical compoents.

  Too damp corrodes electrical components and can lead to shorts.