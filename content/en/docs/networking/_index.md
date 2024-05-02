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
- Network types:
  - Physical network: connected by a serial cable
  - Logical network: connected by circuit within a Frame Relay or MPLS network
- Not scalable

### Point-to-multipoint

Consists of a succession of connections between an interface on one router and multiple destination routers. Each router and connection is part of the same network.


### Hybrid

A combination of two or more types of physical or logical network typologies working together within the same network.

## Network concepts

### Network backbone

The network backbone connects all network segments and servers. It gives the network its structure.
- Similar to the human backbone--its a nerve center
- Must use fast tech, such as Gigabit Ethernet or faster
- For speed and efficiency, you want to connect all network servers and segments directly to the network backbone

### Network segments

A small section of the network that is connected to the backbone, but is not part of the backbone.

### Service-related entry points

Boundary where one entity hands off a connection to another. For example, when a home network connects to a service provider's or carrier's WAN circuit.
- Called the _demarcation point_ or _demarc_ and defines a point of responsibility. 
- Carriers terminate with a piece of equiptment called a _smart jack_ that can run diagnostics up to the physical point where the customer network connects.

### Service provider links

A service provider is an ISP or cable and telephone company that provides networking services with a variety of technologies:
- satellite links for earth to satellite connections
- telephone companies have copper connections through digital subscriber lines (DSL). Good alternative to fiber or cable
- now telephone companies use hybrid fiber/coax networks that use a cable modem that provides service
- a _leased line_ is when the provider installs either a copper or fiber termination that interconnects two endpoints exclusive to the customer
  - no shared bandwidth
  - very secure

### Virtual networking

Networking services with software installed on the hypervisor:
- VMware's virtual switch (vSwitch) that provides the Ethernet switch and routing functions on the hypervisor, so you don't need physical hardware
  - vSwitch is configured the same as physical hardware, its just software virtualization
- virtualized servers use a _virtual network interface card_ (vNIC) to connect a virtual device > hypervisor > LAN
- _network function virtualization_ (NFV) virtualizes networking functions such as routers, switches, firewalls, load balancers, and controllers so you can run all of them on a single device

### Three-tiered model

Introduced by Cisco about 20 yrs ago and is the gold standard for network design. Consists of core, distribution, and access layers. Now there is the _collapsed core_ model that saves money on network switching, and combines the core and distribution layers:
- Core: backbone of the network, where you find connectivity between geographic areas with WAN lines.
  - Only provides routing and switching for the network
  - designed for high availability
- Distribution: referred to as the workgroup layer or aggregation layer because it connects to multiple access layer switches. Switches between core and access layer
  - Where the control plane is located
  - Packet filtering, security policies, routing between VLANs, and defining of broadcast domains are performed
- Access layer: referred to as edge switching layer, connects end-user hosts.
  - Local switching and creates collision domains
  - Designed for network access
  - Where support begins for QoS, power over Ethernet (PoE), and security

### Spine and leaf

Creates a two-tier circuit-switched network for extremely fast network switching. Referred to as a CLOS network, named after Charles Clos who invented multistage circuit-switching in 1938. Traditional three-tier and collapse-core modesl work well in enterprise networks, but not in data centers.
- Create a fast and redundant backbone that connects leaf switches. Leaf switches never talk to another leaf, only the backbone
- leaf switches connect the hosts (servers) to the backbone
- This means that two leaf switches are always two hops away from every server in the network

### Traffic flow

You need to understand where the traffic is flowing for a number of reasons, but especially because of security. Most important locations:
- north-south: to and from network to external network.
  - southbound traffic comes through a firewall and router
  - northbound traffic is routed from your internal network to the internet
- east-west: lateral traffic between server farms and data centers, including:
  - database replication
  - file transfers
  - interprocess communication (IPC)