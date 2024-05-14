---
title: "Internet Protocol"
weight: 50
---

## TCP/IP

Created by the DoD to ensure data integrity and maintain communications in the event of catastropic war.
- First request for comments (RFC) to the Internet Engineering Task Force (IETF) was in 1974
- 1983, TCP/IP replaced Network Control Protocol (NCP) as the official data transport for ARPAnet (Advanced research Projects Agency), which was the DoD's precursor to the internet
- Most development work on TCP/IP happened at UC Berkeley
- BSD distro packaged TCP/IP and it took off

### DoD model

Condensed version of OSI model:
- Process/Application layer (OSI levels 7-5): Defines protocols for node-to-node application communication and controls UI specs
- Host-to-host layer (OSI level 4): Defines protocols for setting up the level of transmission service for applications:
  - e2e connections
  - error-free data delivery
  - packet sequencing
  - data integrity
- Internet layer (OSI level 3): Designates protocols for logical transmission of packets over the network:
  - Logical host addresses (IP assignment)
  - routes packets among multiple networks
- Network Access layer (OSI levels 2-1): Monitors the data exchange between host and network:
  - Hardware addressing
  - defines protocols for physical transmission of data

## Protocols per layer

| Layer | Protocols |
|---|---|
| Process/Application | Telnet, FTP LPD, SNMP, TFTP, SMTP, NFS, X Windows |
| Host-to-Host | TCP, UDP |
| Internet | ICMP, ARP, RARP, IP |
| Network Access | Ethernet, Fast Ethernet, Gigabit Ethernet, Wireless 802.11 |

## Process/Application layer

### FTP
(TCP 20,21)

File Transfer Protocol lets you transfer files across an IP network. Also a program:
- limited to listing and maniuplating directories, typing file contents, and copying files between hosts
  - all data sent as plain text, unencrypted (use SFTP)
- as a protocol, used by apps
- as a program, perform file transfer tasks manually
- requires that you authenticate

### SSH
Secure Shell (TCP 22)

SSH sets up a secure Telnet session over TCP/IP connection:
- Logging into remote systems
- running programs on remote systems
- Moving files from one system to another

### SFTP
Secure File Transfer Protocol (TCP 22)

SFTP transfers files over an encrypted connection with SSH
- Same as FTP, just secure. Transfers files between computers on an IP network

### Telnet

Uses terminal emulation to allow a user on a remote client machine (Telnet client) to access resources of another machine (Telnet server)
- Makes the server think that the client machine is using a termnal attached to the local network
- Plain text only, no security. Replaced by SSH
  
### SMTP
Simple Mail Transfer Protocol (TCP 25)

SMTP delivers email with a spooled, or queued, method of delivery:
- After a message is sent to a destination, the message is _spooled_ to a device
- server software regularly hecks the queue for messages
  - When sees a message, it delivers it to the destination
- SMTP sends mail
- POP3 receives mail

### DNS
Domain Name System (TCP and UDP 53)

DNS resolves an internet hostname to its corresponding IP address
- If you move your website to a different internet provider, DNS helps resolve the new IP addr
- Fully qualified domain name (also called DNS namespace) is a hierarchy that can logically locate a system based on its domain identifier.
  - Must type in the FQDN to resolve an address
- If you can ping a device with its IP addr but can't use the FQDN, you have a DNS issue

### DHCP/BootP
Dynamic Host Configuration Protocol/Bootstrap Protocol (UDP 67/68)

#### DHCP

DHCP assigns IP addresses to hosts with information provided by a server
- routers can be used as DHCP server
- Can provide the following info to a host when the host is requesting an IP addr
  - IP addr
  - subnet mask
  - domain name
  - default gateway (routers)
  - DNS
- DHCP client discover broadcast > DHCP server offer to client unicast > DHCP client request broadcast > DHCP server ACK to finalize exchange
  - Client sends out DHCP discover message as a broadcast at layer 2 and layer 3
  - Layer 2 broadcast is all _f_ in hex: FF:FF:FF:FF:FF:FF.
  - Layer 3 broadcast is 255.255.255.255, which means all networks and all hosts
  - Uses UDP at transport layer

#### BootP

Bootstrap protocol (BootP) assigns an IP address but the host hardware address must be entered manually in a BootP table
- Can send an OS that a host can boot from (?)

### TFTP
Trivial File Transfer Protocol (UDP 69)

TFTP is a stripped-down version of FTP. Cannot browse dirs, can only send and receive files
- Sends much smaller blocks of data than FTP
- No authentication
- Few sites support it

### HTTP
Hypertext Transfer Protocol (TCP 80)

HTTP managees communications between web browsers and web servers
- Opens te right resource when you click a link

### POP3
Post Office Protocol v3 (TCP 110)

POP3 provides a storage facility for incoming mail
- When a client connects to a POP3 server, messages addressed to that client are released for downloading
- Cannot download messages selectively
- Being replaced by the more secure IMAP

### NTP
Network Time Protocol (UDP 123)

Sychronizes the clocks on computers to one standard time source.
- Works with other utilities to make sure that all computers on a network agree on time
- Synchronized clocks prevent system failures

### IMAP
Internet Message Access Protocol (TCP 143)

IMAP4 lets you control how you download your mail
- can peek at message header or partially download 
- store messages on the email server hierarchically and link to documents and user groups too
- provides search commands to use to hunt for messages based on subject, header, content, etc
- authentication features
  - supports Kerberos

### SNMP 
Simple Network Management Protocol (UDP 161/162)

Collects and manipulates valuable network information
- SNMPv3 is the standard and uses TCP and UDP with enhanced security and authentication
- gathers data by polling the devices on the network from a management station or at fixed intervals
- SNMP receives a _baseline_, which is a report delimiting the operational traits of a healthy network
- Can serve as a watchdog over the network
  - 'watchdogs' are called _agents_
  - agents send a _trap_ to management station
  - Network Management System (NMS) polls agents through the Management Information Base (MIB)
  - MIB is a db with predefined questions for agents to investigate the health of the network or device

### LDAP
Lightweight Directory Access Protocol (TCP 389)

Admins have a directory that tracks all network resources, like devices and users. Use LDAP to query directory services like MS Active Directory.

### HTTPS
(TCP 443)

Hypertext Transfer Protocol Secure. Secure version of HTTP that provides security tools for securing transactions bwetween a web browser and a server.
- Securely fill out forms, sign in, authenticate, and send encrypted messages

### TLS/SSL
(TCP 995/465)

Transport Layer Security and Secure Sockets Layer. Cryptographic protocols that secure online data-transfer activities like browsing the web, IMing, etc.
- Often referenced interchangeably
- Both use X.509 certificates and asymmetric cryptography to authenticate to host and exchange a key
  - key encrypts data flow between hosts
- Isn't really tied down to any specific port

### SMB
(TCP 445)

Server Message Block. Shares access to files and printers and other communications between hosts on a Windows network. Can also run on these ports:
- UDP 137, 138
- TCP 137, 139 using NetBIOS

### Syslog
(UDP 514)

[syslog severity levels](https://en.wikipedia.org/wiki/Syslog#Severity_level)

Log system messages to a syslog server, which stores, time-stamps, and squence messages. (You can also read them from a switch or router buffer, but syslog is easier.)
- Can search messages
- send emails based on severity
- devices can forward a syslog message to various destinations:
  - logging buffer
  - console line
  - terminal lines
  - syslog server
  

### SMTPS 
(TCP 587)

Simple Mail Transfer Protocol Secure. Same as SMTP but encrypted:
- TLS encrypted
- Port 587 is sometimes enforced

### LDAPS
(TCP 636)

Lightweight Directory Access Protocol Secure. Requires a CA cert.

### IMAP over SSL
(TCP 993)

IMAP traffic travels over a secure socket to a security port (TCP 993, usually).

### POP3 over SSL
(TCP 995)

POP provides a storage facility for incoming mail. THis is the encrypted version.

### SQL Server
(TCP 1433)

Structured Query Language Server. Relational database engine
- clients connect to port 1433, the official IANA socket number for SQL server

### SQLnet
(TCP 1521)

Also called SQL*Net and Net8. Oracle's networking software that allows remote data access between programs that use the Oracle Database.
- Apps and dbs are shared with different machines and continue communicating as if they were local
- Based on Oracle's Transparent Network Substrate (TNS) that provides a generic interface to all network protocols. This is no longer needed bc of TCP/IP
- Used by client and server to communicate with one another

### MySQL
(TCP 3306)

MySQL is a relational db management system based on SQL:
- most commonly used for cloud-based dbs
- used for data warehousing, e-commerce, logging apps

### RDP
(TCP 3389)

Remote Desktop Protocol. Designed by Microsoft, allows you to connect to another computer to run programs.
- Operates like Telnet, but gives you a GUI instead of terminal
- Macs have preinstalled RDP client
- Official server software is called Remote Desktop Services
- Official client software is called Remote Desktop Connection

### SIP (VoIP)
(TCP or UDP 5060/TCP 5061)

Session Initiation Protocol. Singaling protocol that constructs and deconstructs multimedia communication sessions for voice and video calls, videoconferencing, streaming multimedia, IMs, presence info, and online games
- often works with RTP (VoIP) to set up connections between endpoints

### RTP (VoIP)
(UDP 5004/TCP 5005)

Real-time Transport Protocol (RTP) is a packet-formatting standard for delivering audio and video over the internet.
- Originally multicast, but now unicast too
- de facto VoIP protocol
- Used for Streaming media, video conferencing, and push-to-talk systems

### MGCP (Multimedia)
(TCP 2427/2727)

Media Gateway Control Protocol. Standard protocol for handling the signaling and session managment needed during a multimedia conference.
- defines means of communications between a media gateway and the media gateway controller
- media gateway converts data from circuit switched network to packet switched network formats

### H.323
(TCP 1720)

Provides a standard for video on an IP network that defines how real-time audio, ideo, and data information is transmitted.
- provides signaling, multimedia, and bandwidth control mechanisms

### IGMP

Internet Group Management Protocol. The TCP/IP protocol used for managing IP multicast sessions.
- works at the Network layer and does not use port numbers
- sends out unique IGMP messages over the network to reveal the multi-cast group landscape, and to figure out which hosts belong to which multicast group
- host machines in an IP netowrk se IGMP to become members of the group and to quit the group

### NetBIOS

Network Basic Input/Output System. Works only in the upper layers of OSI model
- allows interface on separate computers to communicate over a network

## Host-to-Host layer

Host-to-host protocols handle the network. Takes data stream from the upper layer and any instructions, and prepares the information to be sent over the network.

### TCP

Transmission Control Protocol:
- takes large blocks of information from the upper layers and breaks it into segments for transmission over the Internet Layer
  - numbers and sequences each segment so that the destination's TCP process can put the segments back together in the correct order
  - TCP client sends the segments, then waits for an ACK from the receiver
  - client retransmits segments that it didn't receive an ACK for
- connection-oriented, must first set up a three-way handshake
- full-duplex, reliable, and accurate but complicated and costly in terms of network overhead
  - UDP is much less overhead
- When data is all sent, there is a call to tear down the virtual circuit

#### TCP segment

```
__________________________________________________________________
|       Source port (16)           |    Destination port (16)    |
|                           Sequence number (32)                 |
|                       Acknowledgement number (32)              |
| Header length (4) | Reserved (6) | Code bits (6) | Window (16) |
|            Checksum (16)         |        Urgent (16)          |
|                       Options (0 or 32 if any)                 |
|                         Data/payload (varies)                  |
------------------------------------------------------------------
```

Source port
: Port number of the app on the host sending the data

Destination port
: Port number of the app requested on the destination host

Sequence number
: Number TCP uses to put data back in the correct order or retransmits missing or damaged data during sequencing

Acknowledgement number
: TCP octet that is expected next

Header length
: Number of 32-bit words in the TCP header, which indicates where the data begins.

Reserved
: Always set to 0

Code bits/TCP flags
: Controls functions that set up and terminate a session

Window
: Window size that the sender is willing to accept, in octets

Checksum
: Cyclic redundancy check (CRC) on the header and data fields. TCP doesn't trust the lower layers, so it checks everything

Urgent
: Valid only if the Urgent pointer in the code bits is set. If set, this value indicates the offset (in octets) from the current sequence number where the segment of non-urgent data begins

Options
: Can be 0 or a multiple of 32 bits. If an option is used but the field doesn't total 32 bits, you must pad it with 0s

Payload (Data)
: Handed down to the TCP protocol at the transport layer, which includes upper-layer headers.


### UDP

User Datagram Protocol.

Transports information that doesn't require reliable delivery. An _unreliable_ protocol:
- _connectionless protocol_ bc it doesn't create a virtual circuit or contact the destination before sending data
- Also called a _thin protocol_ because it doesn't take up much bandwidth
- SNMP uses this to send status messages and alerts
- DNS handles its own reliability issues, so TCP is redundant
- Does not sequence segments and does not send ACKs
  - discards segments after received

```
__________________________________________________________________
|       Source port (16)           |    Destination port (16)    |
|       Length (16)                |    Checksum (16)            |
|                       Data/payload (varies)                    |
------------------------------------------------------------------
```

### Port numbers

TCP and UDP use _port numbers_ to communicate with the upper layers because that is how the local host tracks different simultaneous conversations that it started or accepted.
- Originating source port numbers are dynamically assigned by the source host and have a value of 1024 and higher
- Ports 1023 and lower are _well-known port numbers_, defined in RFC 3232.
- Virtual circuits that don't use an application with a well-known port number are assigned port numbers randomly from a specific range
  - used by upper layers to set up sessions with other hosts and by TCP as source and destination identifiers in the TCP segment


## Internet layer

In the DoD model, the Internet layer provides the following:
- routing
- single network interface for the upper layers
  - without the single interface, app devs would need to write hooks into the application for each different network access protocol
    - Ex: hook for wired ethernet, hook for wireless ethernet

All paths in the internet layer go through the IP

### Internet Protocol

The IP is aware of all interconnected networks
- each machine has a software (logical) address called IP address
- IP looks at the destination address, then uses routing table to find the best path to the destination
  - Layers at the bottom only know about physical links on local networks
- To ID devices on networks:
  - Which network is it on? The software address, or logical address
  - What is its ID on the network? The hardware address, the logical ID is the IP address

#### IPv4 header

1. IP receives segments from host-to-host layer
2. Fragments segments into packets, if necessary
   1. Each packet is assigned IP addr of the sender and recipient so that each layer 3 router that receives the packet can make routing decisions based on the destination address
3. On receiving side, IP reassembles packets into segments


```
____________________________________________________________________________
|  Version (4) | Header length (4) | Priority and Type | Total length (16)  |
|              |                   |   of Service (8)  |                    |
|           Identification (16)    | Flags (3) |    Fragment offset (13)    |
| Time to live (8)  | Protocol (8) |        Header checksum (16)            |
|                        Source IP address (32)                             |
|                       Destination IP address (32)                         |
|                         Options (0 or 32 if any)                          |
|                           Data (varies if any)                            |
-----------------------------------------------------------------------------
```

Version
: IP version number

Header length
: (HLEN) in 32-bit words

Priority and Type of Service
: Tells how the datagram should be handled. First 3 bits are priority bits, also called the differentiated services bits.

Total length
: Length of packet, including header and data

Identification
: Unique IP-packet value used to differentiate fragmented packets from different datagrams

Flags
: Specifies whether packet should be fragmented

Fragment offset
: Provides fragmentation and reassembly if the packet is too large for a single frame. Allows different maximum transmission units (MTUs) on the internet

Time to live
: Set when packet is generated, if the packet doesn't arrive at its destination before the TTL expires, it disappears. Stops packets from circling the network forever.

Protocol
: Port number for upper-layer protocol

Header checksum
: CRC on header only

Source IP address
: 32-bit address of sending station

Destination IP address
: 32-bit address of the station that this packet is destined for

Options
: Used for network testing, debugging, security, and more

Data
: Upper-layer data

##### Protocol field

This table describes how the protocol field tells the IP to send data to the transport layer (ex: TCP or UDP):

| Protocol | Protocol number |
|---|---|
| ICMP | 1 |
| IP in IP (tunneling) | 4 |
| TCP | 6 |
| UDP | 17 |
| EIGRP | 88 |
| OSPF | 89 |
| IPv6 | 41 |
| GRE | 47 |
| Layer 2 tunnel (L2TP) | 115 |

### ICMP

Internet control message protocol

Works at the network layer as a management protocol and messaging service provider for IP. Messages are carried as IP packets:
- Provide hosts with info about network problems
- Encapsulated within IP datagrams

#### Use cases

Destination unreachable
: If a router can't send a datagram any further, it uses ICMP to send a Destination unreachable message back to the sender.

Buffer full
: If a router's memory buffer for receiving datagrams is full, it sends Buffer full messages until it can receive more datagrams in the buffer.

Hops
: Each IP datagram is alloted a specific number of routers (hops) to pass through. When it reaches its limit, the last router deletes it and sends an ICMP message to the sender.

Ping
: Uses ICMP echo request and reply messages to check the physical and logical connectivity of machines on the internet.

Traceroute
: Uses IP packet time to live time-outs to discover the path that a packet takes to traverse the internet.

### ARP

Address resolution protocol

Translates the software (IP) address int a hardware address of a host using a known IP address:
- When IP has datagram to send, it must inform a Network Access protocol (like Ethernet) of the destination's hardware address on the local network.
- If IP doesn't find the destination host's hardware address in the ARP cache, it sends out a broadcast asking for a reply from the machine with the hardware address

### RARP

Reverse address resolution protocol

Sends out MAC address and a request for an IP address for a diskless machine:
- When an IP machine is diskless, it cannot know its IP address, but it knows its MAC address.
  - Designated RARP server replies

### GRE

Generic Routing Encapsulation

Tunneling protocol that can encapsulate many protocols inside IP tunnels
- example protocols include EIGRP, OSPF, and IPv6
- does not provide security

### IPSec

Internet Protocol Security

More secure than GRE for tunneling information across the internet. Uses two primary protocols: AH and ESP. Has some limitations:
- Does not support IP broadcast or multicast, so can't use routing protocols that need them
- No support for multiprotocol traffic
  - Called GRE over IPSec, Used with GRE to run a routing protocol, IP multicast, and multiprotocl traffic across network

#### Authentication Header (AH)

Provides authentication for the data and the IP header of a packet using a one-way hash for packet authentication:
- Guarantees authenticity
  - Sender generates a one-way hash
  - receiver generates the same hash
  - If the packet is changed in any way, it is dropped
- Checks packet, but no encryption services

#### Encapsulating Security Payload (ESP)

Provides the following:

Confidentiality
: Sending device encrypts packets before transmitting with symmetric encryption algorithm. Both endpoints must use the same confidentiality encryption:
  - HMAC-SHA1/SHA2 for integrity protection and authenticity
  - TripleDES-CBC for confidentiality
  - AES-CBC and AES-CBC for confidentiality
  - AES-GCM and CHaCha20-Poly1305 for confidentiality and authentication 

Data integrity
: Receiver verifies that the data was not altered in any way. Uses checksums.

Authentication
: Connection is made with the correct partner.

Anti-Replay Service
: Based upon the receiver, the service is effective only if the receiver checks the sequence number.
  - Replay attack: Hacker copies an authenticated packet and transmits it to the intended destination. Sequence number field is meant to stop this type of attack.

Traffic flow
: Need at least tunnel mode selectd.
  - Most effective at a security gateway w tons of traffic because its easier for you to mask the true source-destination patterns from people that want to break into your network

#### Internet Key Exchange (IKE)

Management protocol that negotiates security associations (SA) between endpoints:
- SA defines authentication, encryption, and IPSec protocols used for the IPSec connection

Has two phases that are managed by the Internet Security Association and Key Management Protocol (ISAKMP):
- Phase 1 (Main mode): WHere parameters (policies) are agreed upon by the endpoints, known as HAGLE:
  - hash
  - authentication
  - group
  - lifetime
  - encryption
  After agreed upon, both endpoints authenticate and calculate a shared secret symmetrical encryption key and create the encryption tunnel.
- Phase 2 (Quick mode): Negotiation and connection of IPSec, called the IPSec transform set:
  - Contains details about the AH and ESP protocols used, encryption, hashing, and mode that the IPSec tunnel operates in

### Data encapsulation

When data is transmitted across a network to another device, it goes through _encapsulation_, which means that it is wrapped with protocol information at each layer of the OSI model.
- Each layer communicates with its peer layer on the receiving device
- Each layer uses _protocol data units_ (PDUs) that hold the control info attahced to the data at each layer
  - Usually attached to the header in front of the data field, but can also be at the end, or the _trailer_.
  - PDUs are attached at each layer and has a specific name depending on the header information
  - PDU info is read only by the peer layer on the receiving devices. After its read, it is stripped off, then the data is handed to the next layer up



```
Application                 --------------
Presentation                | Datagrams  |  (data stream)
Session              _______|____________|
Transport            | TCP header | Data |  (segment)
                     --------------------|
Network              |  IP header | Data |  (packet or datagram)
              ----------------------------
Data link     | Frame | LLC | Data | FCS |  (frame)
              ----------------------------
Physical      | Transmitted as 1s and 0s |  (bits)
              ----------------------------
```

#### PDU life cycle

User information > data stream > segments > packets/datagrams > frames > bits

1. Data stream is handed to the Transport layer
2. Transport layer sets up virtual circuit with sync packet
3. Data stream is broken into smaller pieces
4. Transport layer header (PDU) is created and attached to the header field
   1. Now the data is called a segment, sequenced so the data stream can be reassembled by the receiver
5. Segment is handed off to network layer for logical addressing and routing through internet
6. Network layer adds controlheader to segment.
   1. Now it is called a packet or datagram
   2. Locates destination hardware address with ARP to either local machine or router for transmission over the internet
7. Data link takes packets and puts them on network medium, such as cable or wireless
   1. Encapulates each packet in a frame. Called frame because there is a header and trailer that bookends it.
   2. Frame header carries hardware address of source and destination
   3. After it is routed to destination network, a new frame is added to get it to the host
8. Physical layer encodes digits into a digital signal
   1. Recieving physical layer decodes the digital signal into digits
9. Receiver builds frames
10. Receiver runs the cyclical redundancy check (CRC)
11. Receiver hecks the CRC answer against the frames Frame Check Squence (FCS) field
    1.  If this matches, the packet is pulled from the frame in a process called _de-encapsulation_. Rest is discarded.
12. Packet is handed to network layer where IP address is checked
13. If IP address matches, segment is pulled from packet and rest is discarded.
14. Transport layer processes the segment and rebuilds the data stream for the upper layers

#### Ports

- Transport layer users port numbers to define the virtual circuit and upper-layer process
  - Turns the data stream into segments and creates a reliable session
  - TCP: virtual circuit is defined by source port number
    - Destination port defines the upper-layer process (application) that the data stream is handed to