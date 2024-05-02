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