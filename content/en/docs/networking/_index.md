---
title: "Networking"
weight: 15
---

A network is two or more connected computers that can share resources such as data and applications, office machines, an internet connection, or any combination of the previous things.
- Hosts on a network communicate through binary code

## Local area network

A LAN is usually restricted to a location that is the size of an office building, single office, or home office.
- Best to split a big LAN into smaller logical zones called _workgroups_ to make admin easier.
  - Workgroups are physically in the same network segment
  - Create workgroups according to function. For example, an accounting workgroup and a sales workgroup
  - You can connect workgroups with a router. This allows you to keep the workgroups separate so they aren't on one large network, competing for access to resources.

### Common components

Workstations
: Computers that run more than one CPU and whose resources are avialable to other users on the network.

Servers
: Computers that are "at the service" of the network and run specialized software known as the network operating system to maintain and control the network.
  
  Servers are highly specialized and run one labor-intensive task:
  - File server: Stores and dispenses files
  - Mail server: Handles email functions
  - Print server: Manages printers on the network
  - Web server: Manages web-based activities by running HTTPS for storing web content and accessing web pages
  - Fax server: Sends and receives paperless faxes over the network
  - Application server: Manages network applications
  - Telephony server: Handles call center and call routing like a sophisticated answering machine
  - Proxy server: Handles tasks in the place of other machines on the network, particularly an internet connection

Hosts
: Any network device with an IP address, such as a workstation or server. Generally any machine that works with TCP/IP.
  
  The term _host_ comes from the early days of networking when machines were called _mainframes_. These mainframes were also called hosts if they had TCP/IP functionality. Machines that did not have TCP/IP were called _dumb terminals_ becuase only mainframes (hosts) were given IP addresses.

  The term _gateway_ referred to any layer 3 machines like routers.

### Network types

Different network types use varying technologies for connectivity.

Metropolitan area network
: MANs cover a metropolitan area and connects various buildings and facilities over a carrier provider network. Usually offer internet over fiber cable. Think of a MAN as a concentrated WAN.

Wide area network
: WANs span a large geographic area. They are different from LANs in the following ways:
  - WANs need a router port or ports
  - WANs span larger geographic areas
  - WANs are usually slower
  - We can choose when we connect to a WAN. Our workstation is almost always connected to a LAN, or there is no network access.
  - WANs can use private or public data transport media, like phone lines
  
  _Internet_ comes from _internetwork_. An internetwork is a type of LAN or WAN that connects networks, or _intranets_. On internetworks, hosts use hardware addressed to communicate with other hosts on the LAN. To communicate with hosts on a different LAN, they use logical addresses--IP addresses--and routers. Routers have connections to logical routers.

  The internet is a _distributed WAN_, an internetwork that is made up of a lot of interconnected computers in different places.

  A _centralized WAN_ has a central computer or location that remote computers or devices connect to. For example, when satellite offices connect to the main office.

Personal area network
: PANs include close proximity connections between smartphones or laptops to send data between devices. Another example is when you connect to a projector.

Campus area network
: A CAN covers a college or corporate campus. It connects LANs in different buildings and has WiFi roaming. Between a LAN and WAN in scope.

Storage area network
: A SAN is designed for and used exclusively by storage systems. SANs connect servers to storage arrays with banks of hard drive or other storage media. Usually only found in data centers:
  - Do not mix traffic with other LANs
  - Uses protocols specifically for storage, such as Fibre Channel and iSCSI
  - Has routers and switches designed specifically for storage protocols

Software-defined wide area network
: An SDWAN is a virtual WAN. It uses software to manage connections, services, and devices. Decouple hardware from the control mechanism.

Multiprotocol label switching
: MPLS defines the actual layout of what is one of the most popular WAN protocols used today. Has advantages over other WAN technologies:
  - Physical layout flexibility.
  - Prioritizing of data. Voice data can take priority over basic data.
  - Redundancy in case of link failure.
  - One-to-many connection.
  
  MPLS adds labels to data and then uses the labels to figure out where to send traffic.

Multipoint generic routing encapsulation
: mGRE is a carrier or service provider that dynamically creates and terminates connections to nodes on a network.
  - Used in Dynamic Multipoint VPN deployments that enables dynamic connections without having to pre-confgiure static tunnel endpoints
  - Encapsulates user data, creates a VPN connection, tears down connection when complete

## Network architecture

There are two network types with physical and logical differences that you need to know about:
- peer-to-peer
- client-server

### Peer-to-peer

All computers are peers, meaning that no one computer has authority, they are all equal.
- there is no central computer
- all can be clients that access resources
- all can be servers that provide resources
- security risks bc you are not securing a single, central server
- backups are difficult

### Client-server

A single server uses a network operating system for managing the whole network:
- client request goes to main server, which handles security and directs client to correct resource
  - client doesn't have to know where each resource is
  - Ex: Active directory. User info and passwds are stored in central location, not on each machine
- scalable because client-server networks can have unlimited workstations

## Physical network topologies

Defines characteristics of the network, like where all workstations and other devices are located and the arrangement of physical media, such as cables:
- phyiscal topology: actual physical layout
- logical topology: how a digital signal or data travels through the physical topology

When selecting a typology, consider the following:
- cost
- ease of installation
- ease of maintenance
- fault-tolerance requirements
- security requirements

### Bus

Most basic, consists of two terminated ends, and each computer connecting to a single, unbroken cable running the entire length.
- Cheap bc save $ on cable
- Before, computers used to connect with wire taps, then drop cables.
- Now, use a T connector into the main cable to connect device
- All computers see the data on the network, but only the computer that the data is addressed to receives it
- Problems:
  - Hard to troubleshoot, change, or move
  - No fault tolerance--how well a system can automatically respond to and resolve an issue without users knowing--if the cable goes down, whole network is down

### Star

Also called hub-and-spoke, all computers are connected to a central point with their own wired or wireless connection.
- Central point is usually a hub, switch, or accesspoint
- If the cable fails, only brings down that network segment
  - Helps troubleshooting
  - more fault-tolerant
- More scalable
- If the central point fails, the entire network fails
- Higher cost bc cable cost
- _Point-to-point_ link lets you extend the network by making any device attached to the center into a hub for other connections.

### Ring

Each computer is directly connected to other computers in the same network.
- Not popular
  - Costly
  - hard to reconfigure
  - not fault-tolerant
- If you want to add a computer, you have to break the cable ring (brings down the network)

### Mesh

There is a path from every machine to every other one in the network.
- Lots of connections to create redundancy
  - For each _n_ hosts, you get _n_(_n_ - 1)/2 connections
  - Ex: 10(9)/2 = 45 connections for 10 hosts
- If each device is not connected, its called a _hybrid mesh_ or _partial mesh_
- Usually only very small networks
- Not many collisions--when two hosts try to communicate simultaneously
- Not often used in LANs
  - WANs sometimes use a hybrid mesh

### Point-to-point

Direct connection between two routers or switches, providing one communication path
- 
- Network types:
  - Physical network: connected by a serial cable
  - Logical network: connected by circuit within a Frame Relay or MPLS network
- Not scalable

### Point-to-multipoint

Consists of a succession of connections between an interface on one router and multiple destination routers. Each router and connection is part of the same network.


### Hybrid

A combination of two or more types of physical or logical network typologies working together within the same network.

## Network backbone

The network backbone connects all network segments and servers. It gives the network its structure.
- Similar to the human backbone--its a nerve center
- Must use fast tech, such as Gigabit Ethernet or faster
- For speed and efficiency, you want to connect all network servers and segments directly to the network backbone

## Network segments

A small section of the network that is connected to the backbone, but is not part of the backbone. 