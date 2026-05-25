+++
title = 'OSI reference model'
date = '2026-05-25T09:06:36-04:00'
weight = 10
draft = false
+++

The *OSI (Open Systems Interconnection) model* is a conceptual framework for understanding and standardizing how network communication works. Rather than treating a network as a single system, it breaks communication into distinct functional areas: data routing, user interaction, or a mobile device connecting wirelessly. This separation lets engineers design, troubleshoot, and understand networks one layer at a time.

## Layers

The OSI model divides network communication into seven distinct layers. Each layer depends on the layer beneath it and provides services to the layer above. When data travels from your device to another, it moves down the stack on the sender's side and back up on the receiver's side.

Two mnemonics cover both directions:

- **Top to bottom (7→1):** *All People Seem To Need Data Processing*: Application, Presentation, Session, Transport, Network, Data Link, Physical
- **Bottom to top (1→7):** *Please Do Not Throw Sausage Pizza Away*: Physical, Data Link, Network, Transport, Session, Presentation, Application

| Layer | Name | Function | Protocols and devices |
|:------|:-----|:---------|:----------------------|
| 7 | Application | Provides the interface between user-facing applications and the network. Handles high-level protocols, data representation, and user services. | HTTP, HTTPS, FTP, SMTP, DNS, Telnet, SNMP, SSH, SSL/TLS; WAFs, IPS |
| 6 | Presentation | Translates data between the application layer and the network. Handles encryption, decryption, compression, and character encoding. | SSL/TLS, JPEG, MPEG, ASCII, Unicode |
| 5 | Session | Establishes, manages, and terminates communication sessions between applications. Controls dialog between two systems. | NetBIOS, RPC |
| 4 | Transport | Provides end-to-end delivery of data between hosts. Handles segmentation, flow control, and error recovery. | TCP, UDP |
| 3 | Network | Routes packets across multiple networks using logical addressing. Determines the best path from source to destination. | IP, ICMP, OSPF, BGP; routers, Layer 3 switches, firewalls |
| 2 | Data Link | Transfers data between devices on the same network segment using physical (MAC) addressing, framing, and error detection. | Ethernet, Wi-Fi (802.11), ARP, PPP; switches, bridges, MAC addressing |
| 1 | Physical | Transmits raw bits over a physical medium. Defines cables, connectors, voltages, and signal timing. | Ethernet cables, fiber optic, coaxial cable; hubs, repeaters, NICs |

{{< admonition "SSL/TLS in layers 6 and 7" note >}}
SSL/TLS appears in both Layer 6 and Layer 7 because it spans two concerns. Layer 6 handles the encryption and decryption of data in transit. Layer 7 handles the SSL/TLS handshake and session negotiation between the client and server. For the Network+ exam, the primary association is Layer 6 (Presentation), but understanding that the handshake occurs at Layer 7 gives you the full picture.
{{< /admonition >}}

## Layer 1: Physical

Layer 1 is the physical foundation of the OSI model. It defines how raw bits move across a physical medium: electrical signals over copper, light pulses through fiber, or radio waves through the air. Every higher layer depends on Layer 1 to carry those bits from point to point.

Devices that connect to a LAN with an Ethernet cable are *Ethernet devices*.

Common Layer 1 components:

- **Hubs:** Multi-port devices that broadcast incoming data to every connected port. Examples: generic unmanaged hubs (largely obsolete in modern networks).
- **Repeaters:** Devices that regenerate a degraded signal to extend network range. Examples: TP-Link signal boosters, inline Ethernet repeaters.
- **Network cables:** The physical medium that carries data as electrical or light signals. Examples: Cat5e, Cat6, Cat6a copper twisted-pair cable; OM3/OM4 multimode fiber; OS2 single-mode fiber.
- **Wireless:** Radio frequency hardware that carries data without physical cables. Examples: Cisco Meraki MR access points, Ubiquiti UniFi APs, Netgear Nighthawk routers, Intel Wi-Fi 6 AX200 adapters.

### Hub

A *hub* is a basic device that connects multiple computers or network devices. When it receives data on one port, it broadcasts that data to every other port. Hubs cannot identify the intended recipient of a frame and do not distinguish between devices. This broadcast behavior makes them inefficient and a security liability. Modern networks use switches instead.

### Repeaters

A *repeater* extends network range by regenerating a signal before it degrades. As data travels over cable, the signal weakens with distance. A repeater receives the degraded signal and retransmits a clean, full-strength copy, preserving signal integrity across longer cable runs.

## Layer 2: Data Link

Layer 2 transfers data between devices on the same network segment and provides the first layer of error detection. It organizes data into *frames*. Each frame contains:

- Destination MAC address
- Source MAC address
- Data (payload)
- *CRC (Cyclic Redundancy Check)* error-checking code

The receiving device uses the CRC to verify that the frame arrived without corruption.

Common Layer 2 devices:

- **Switches:** Intelligent devices that learn each device's MAC address and forward frames only to the correct port, reducing unnecessary network traffic. Examples: Cisco Catalyst 9200, Ubiquiti UniFi switches, Netgear ProSAFE series, TP-Link TL-SG series.
- **Bridges:** Devices that connect two or more network segments using MAC address filtering, making the segments function as a single network. Examples: Ubiquiti NanoStation in bridge mode, Cisco Aironet in bridge mode, older standalone network bridges (largely replaced by switches in modern networks).

### Switch

A *switch* is more intelligent and more secure than a hub. It maintains a *MAC address table* that maps each device's MAC address to a specific port. When a frame arrives, the switch looks up the destination MAC address and forwards the frame only to the correct port. This targeted forwarding reduces unnecessary traffic and limits data exposure to unintended recipients.

### Bridges

A *bridge* connects two or more network segments so they function as a single network. It uses MAC addresses to filter and forward frames between segments. When a frame arrives, the bridge checks the destination MAC address. If the destination is on the same segment, the bridge drops the frame. If it is on another segment, the bridge forwards it.

### NICs

A *NIC (Network Interface Card)* is hardware installed in a device that provides a physical connection to the network. It converts device data into a format suitable for transmission over the network medium. Every NIC has a unique MAC address assigned at the factory.

NICs operate at both Layer 1 and Layer 2:

- At **Layer 1**, the NIC handles physical signal transmission and reception: encoding bits as electrical signals on copper or light pulses on fiber, and managing timing and voltage levels.
- At **Layer 2**, the NIC handles MAC addressing, frame assembly and disassembly, and error detection using CRC.

## Layer 3: Network

Layer 3 routes data between different networks. Where Layer 2 moves data between devices on the same segment, Layer 3 moves data across multiple networks. This is the layer where routers operate.

Layer 3 organizes data into *packets*. Each packet contains:

- Source IP address
- Destination IP address
- Additional routing information (TTL, protocol, header checksum)

Layer 3 also handles *IP addressing* and divides networks into *subnets*: logical subdivisions that organize devices and control traffic flow.

Common Layer 3 devices:

- **Routers:** Devices that forward packets between networks by analyzing destination IP addresses. Examples: Cisco ISR 4000 series, Juniper MX series, Netgear Nighthawk, pfSense software routers.
- **Layer 3 switches:** Devices that combine Layer 2 switching with Layer 3 routing in a single unit. Examples: Cisco Catalyst 3850, Aruba 2930F, Ubiquiti UniFi managed switches.
- **Stateful firewalls:** Devices that filter traffic between networks based on connection state and ruleset. Examples: Palo Alto PA series, Fortinet FortiGate, Cisco ASA, pfSense.

### Router

A *router* forwards data packets between networks by analyzing the destination IP address of each packet. To determine the most efficient path, the router consults its *routing table*: a database that maps known network destinations to the next-hop address, outgoing interface, and route metric (cost). When a packet arrives, the router matches its destination IP against the table and forwards it out the appropriate interface. If no match exists, the router uses its *default route* (also called the *gateway of last resort*) to forward the packet, or drops it if no default route is configured.

Routing tables are built two ways:

- *Static routes*: manually configured by an administrator. Reliable, but do not adapt to network changes.
- *Dynamic routes*: learned automatically from neighboring routers using protocols such as OSPF or BGP. Adapt to topology changes without manual intervention.

### Layer 3 switch

A *Layer 3 switch* combines the high-speed frame forwarding of a standard switch with the ability to route between network segments. Within a LAN, it forwards traffic based on MAC addresses like a standard switch. Between segments, it routes based on IP addresses like a router.

*VLANs (Virtual Local Area Networks)* are logical segments created within a switch that group devices independently of their physical location. A Layer 3 switch can route traffic between VLANs without a separate router, a capability called *inter-VLAN routing*.

The ideal use case is a large campus or enterprise LAN where many devices need fast intra-building switching and multiple VLANs or subnets need to communicate. A Layer 3 switch handles both functions in one device at wire speed, reducing latency and eliminating the need for a dedicated router between internal segments.

### Stateful firewalls

A *stateful firewall* operates at both Layer 3 and Layer 4. At Layer 3, it filters traffic by source and destination IP address. At Layer 4, it tracks the state of active TCP and UDP connections, so it can make decisions based on the full context of a connection rather than individual packets in isolation.

The firewall maintains a *state table* that logs each active connection: source IP, destination IP, source port, destination port, and TCP flags such as SYN, ACK, and FIN. When a packet arrives, the firewall checks it against the state table. Packets belonging to an established, permitted connection pass through. Packets that match no known connection state are evaluated against the ruleset and either allowed or dropped.

A *Layer 7 firewall*, also called a *next-generation firewall (NGFW)*, inspects traffic at the application level. Where a stateful firewall sees IP addresses, ports, and connection state, an NGFW can identify specific applications, detect malicious payloads inside permitted connections, and enforce policies based on application behavior.

## Layer 4: Transport

Layer 4 manages end-to-end communication between applications on different hosts. It handles error correction, data flow control, and ensures data arrives completely and in order. When an application sends data, Layer 4 breaks it into smaller units and reassembles those units at the destination.

Layer 4 is implemented in software on the operating system of computers and networking devices such as firewalls.

*TCP (Transmission Control Protocol)* and *UDP (User Datagram Protocol)* are the two primary Layer 4 protocols. TCP provides reliable, ordered delivery at the cost of overhead. UDP sacrifices reliability for speed.

| | TCP | UDP |
|:--|:----|:----|
| Connection | Connection-oriented (requires handshake) | Connectionless (no setup required) |
| Reliability | Guaranteed delivery with retransmission | No delivery guarantee |
| Ordering | Segments arrive in order | Datagrams may arrive out of order |
| Speed | Slower due to overhead | Faster, minimal overhead |
| Use cases | Web browsing, email, file transfer | Video streaming, VoIP, DNS, gaming |

Layer 4 data units:

- TCP uses *segments*. Each segment includes source and destination port numbers, a sequence number, and additional control fields.
- UDP uses *datagrams*. Each datagram includes source and destination port numbers and a length field.

**TCP segment:**

```
+------------------+------------------+
|   Source Port    | Destination Port |
+------------------+------------------+
|            Sequence Number          |
+-------------------------------------+
|         Acknowledgment Number       |
+--------+--------+-------------------+
| Offset | Flags  |    Window Size    |
+--------+--------+-------------------+
|     Checksum    |  Urgent Pointer   |
+-----------------+-------------------+
|                Data                 |
+-------------------------------------+
```

**UDP datagram:**

```
+------------------+------------------+
|   Source Port    | Destination Port |
+------------------+------------------+
|      Length      |    Checksum      |
+------------------+------------------+
|                Data                 |
+-------------------------------------+
```

Common Layer 4 devices:

- **Firewalls:** Security devices that filter traffic based on IP addresses and port numbers. Examples: Palo Alto PA series, Fortinet FortiGate, Cisco ASA, pfSense, Windows Defender Firewall, Linux iptables.

### TCP

*TCP (Transmission Control Protocol)* provides reliable, ordered, and error-checked delivery of data between applications over IP.

#### Three-way handshake

Before transmitting data, TCP establishes a connection:

1. The client sends a *SYN* (synchronize) segment to the server with its initial sequence number.
2. The server responds with a *SYN-ACK*, acknowledging the client's sequence number and sharing its own.
3. The client sends an *ACK*, confirming the server's sequence number. The connection is now established.

#### Error detection and retransmission

Each TCP segment includes a checksum that the receiver uses to detect corruption. If a segment is lost or corrupted, the receiver withholds its acknowledgment. The sender detects the gap and retransmits the missing segment.

#### Flow control and congestion control

TCP manages transmission rate to prevent overwhelming the receiver. The *window size* field in each segment tells the sender how much data the receiver can accept before requiring an acknowledgment.

TCP also adjusts its rate based on network conditions. When it detects congestion through packet loss or delayed acknowledgments, it reduces the transmission rate and increases it gradually as conditions improve.

### UDP

*UDP (User Datagram Protocol)* is a connectionless protocol. It sends datagrams directly to the destination without establishing a connection and does not verify delivery or order.

This makes UDP faster and lower overhead than TCP. It is the right choice when speed matters more than reliability, or when the application manages its own error handling.

Common use cases:

- **Video streaming** (Netflix, YouTube): minor packet loss is preferable to the buffering caused by retransmission.
- **VoIP and video calls** (Zoom, Teams, FaceTime): real-time delivery matters more than perfect accuracy.
- **Online gaming**: low latency is critical; missed packets are discarded rather than retransmitted.
- **DNS lookups**: short single-request queries where speed matters and the application retries on failure.
- **DHCP**: broadcast-based discovery that runs before a device has an IP address.

### Firewalls

A *firewall* monitors and controls incoming and outgoing network traffic based on a configured ruleset. Firewalls are available as hardware appliances, software applications, or a combination of both.

At Layer 4, a firewall examines source and destination IP addresses alongside port numbers to allow or block traffic. For example, a firewall might permit port 443 (HTTPS) and block port 23 (Telnet).

Firewalls operate across multiple layers. At Layers 3 and 4 together, they filter traffic by IP address, port number, and connection state. Most firewall rules are written around these values, making Layers 3 and 4 the core of standard traffic filtering.

## Layer 5: Session

Layer 5 establishes, manages, and terminates sessions between two devices. A *session* is a logical, persistent connection between two applications that frames their entire exchange from start to finish. Where Layer 4 manages individual segment delivery, Layer 5 manages the conversation as a whole.

### Session establishment

A session begins when two applications negotiate the terms of their communication. This includes authentication (verifying identity), agreement on session parameters (who initiates, idle timeouts, termination rules), and synchronization of starting state so both sides begin from a known, consistent point.

### Checkpointing and synchronization

*Checkpoints* are markers inserted into the data stream at regular intervals. Each checkpoint records the transmission state at that moment. When both sides acknowledge a checkpoint, it becomes a known-good recovery point.

If a session is interrupted by a network failure or device crash, the session layer uses the most recent checkpoint to resynchronize both sides and resume from that point rather than restarting from the beginning. For example, if a large file transfer reaches 80% before the connection drops, checkpointing allows the transfer to resume from 80%.

*Session synchronization* keeps data exchange in the correct order and ensures consistency across both ends. It coordinates which side sends and which receives, and confirms both sides remain aligned throughout the exchange.

### Graceful termination

When a session ends, the session layer ensures all data has been transmitted and acknowledged before closing. Resources allocated for the session (buffers, connection state, memory) are released cleanly. This prevents data loss that would result from abruptly dropping a connection mid-transfer.

Common Layer 5 protocols:

- **NetBIOS (Network Basic Input/Output System):** An API and protocol suite that provides session establishment, name resolution, and datagram services within a LAN. NetBIOS runs over TCP/IP as *NBT (NetBIOS over TCP/IP)* on port 139. Examples: legacy Windows file and printer sharing, older Windows domain networking.
- **SMB (Server Message Block):** A protocol for sharing files, printers, and other resources across a network. SMB manages session establishment and maintenance between client and server. Modern versions SMB2 and SMB3 offer improved performance, security, and resilience. Examples: Windows network shares (\\server\share), Samba on Linux and macOS for Windows interoperability, Azure Files.

## Layer 6: Presentation

Layer 6 translates data between the application layer and the lower layers of the OSI model. It acts as a translator: converting data from applications into a format suitable for network transmission, and reversing that process on the receiving end.

Layer 6 does not deal with packets directly. Instead, it modifies the data within them through three functions: encryption and decryption, compression, and format translation.

### Encryption and decryption

Layer 6 encrypts outgoing data to protect it during transmission and decrypts incoming data so the application can read it. For example, when you connect to a website over HTTPS, SSL/TLS encrypts your request before it travels the network and decrypts the server's response when it arrives.

### Compression

Layer 6 compresses data to reduce its size before transmission and decompresses it at the destination. For example, video encoded as MPEG compresses hours of footage into a manageable file size for streaming. The receiver decompresses the data to restore it for playback.

### Format translation

Layer 6 converts data between formats so two systems using different encodings can communicate. For example, a Windows system using ASCII and a Unix system using UTF-8 can exchange text because Layer 6 handles the conversion. Similarly, an image in one format can be converted to a more portable format before transmission.

Common Layer 6 protocols and formats:

- **SSL/TLS:** Encrypts and decrypts data in transit. Examples: HTTPS connections, secure email (SMTPS), VPN tunnels.
- **JPEG, PNG, GIF:** Image compression and encoding formats for photos and graphics.
- **MPEG, MP4, H.264:** Video compression formats for streaming and storage.
- **ASCII, Unicode (UTF-8):** Character encoding standards that define how text is represented as binary data.

## Layer 7: Application

Layer 7 is the closest layer to the end user. It provides network services directly to applications such as web browsers, email clients, and FTP programs. Unlike lower layers, which handle delivery and routing, Layer 7 defines how applications communicate over the network.

Layer 7 relies entirely on the layers beneath it to deliver data. When an application sends data, each layer adds its own header as the data moves down the stack. This process is called *encapsulation*. At Layer 4, the data becomes a segment or datagram. At Layer 3, it becomes a packet. At Layer 2, it becomes a frame. At Layer 1, it becomes raw bits on the wire.

On the receiving end, the process reverses. Each layer reads and strips its header as data moves up the stack. This is called *decapsulation*. The original application data arrives at Layer 7 intact.

Common Layer 7 protocols:

- **HTTP (Hypertext Transfer Protocol):** The foundation of data exchange on the web. Browsers use HTTP to request and receive web pages. HTTPS adds SSL/TLS encryption. Examples: web browsing, REST APIs, web application traffic.
- **FTP (File Transfer Protocol):** Transfers files between a client and server. Uses port 21 for control and port 20 for data. Examples: uploading files to a web server, transferring files between systems.
- **SMTP (Simple Mail Transfer Protocol):** Sends email between mail servers and from clients to servers. Uses port 25 for server-to-server delivery and port 587 for client submission. Examples: sending email through Gmail, Outlook, or any SMTP relay.

Layer 7 security devices inspect traffic at the application level, going beyond IP addresses and ports to analyze the content and behavior of application data.

- **WAFs (Web Application Firewalls):** Analyze and filter HTTP/HTTPS traffic to protect web applications from application-layer attacks. Examples: AWS WAF, Cloudflare WAF, F5 Advanced WAF, ModSecurity.
- **IPS (Intrusion Prevention System):** Inline devices that scan traffic in real time for known attack signatures, anomalies, and policy violations. Examples: Cisco Firepower, Palo Alto Threat Prevention, Snort, Suricata.
- **NGFWs (Next-Generation Firewalls):** Stateful firewalls extended with application-layer inspection. Identify and control traffic by application rather than port and protocol alone. Examples: Palo Alto PA series, Fortinet FortiGate, Check Point NGFW.

### WAF

A *WAF (Web Application Firewall)* protects web applications by analyzing HTTP and HTTPS traffic between clients and servers. Where a standard firewall filters by IP address and port, a WAF examines the content of web requests and responses.

When a request arrives, the WAF inspects the URL, headers, cookies, and request body against a ruleset. Requests that match known attack patterns are blocked before they reach the application. Common threats a WAF blocks include:

- *SQL injection*: an attacker inserts malicious SQL code into a form field or URL parameter to manipulate the application's database. The WAF detects SQL syntax in the request and rejects it.
- *XSS (Cross-Site Scripting)*: an attacker injects malicious scripts into content served to other users. The WAF detects script tags or encoded payloads in request data and blocks them.

WAFs operate in two modes. In *detection mode*, they log suspicious requests without blocking. In *prevention mode*, they actively block requests that violate rules.

### IPS

An *IPS (Intrusion Prevention System)* is an inline security device that monitors traffic in real time and blocks threats before they reach their target. At Layer 7, an IPS inspects the full content of application-layer traffic rather than just headers and ports.

The IPS scans traffic using three detection methods:

- *Signature-based detection*: compares traffic against a database of known attack patterns. Fast and accurate for known threats, but cannot detect novel attacks.
- *Anomaly-based detection*: establishes a baseline of normal traffic behavior and flags deviations. Can detect unknown threats but may produce false positives.
- *Policy-based detection*: blocks traffic that violates defined security policies regardless of whether it matches a known signature.

When the IPS identifies a threat, it rejects the traffic, resets the connection, or alerts administrators. Because it sits inline in the traffic path, it acts in real time without manual intervention.
