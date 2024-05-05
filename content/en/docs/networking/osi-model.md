---
title: "OSI model"
weight: 20
---

## Overview

The _Open Systems Interconnection_ reference model--a hierarchical conceptual model of hos communications should happen--describes how data and network information are communicated from an application on one computer through the network media to an application on another computer.

When networking began, computers could only communicate with other computers from the same manufacturer. OSI model helped vendors create interaoperable network devices and software in the form of protocols so that different vendors' networks could work together.

## Layers

Layers are how the model addresses and groups all processes required for effective communication. Like different departments at a company (accounting, support, etc.).
- If you need to develop a protocol for a layer, you just focus only on that layer's functions.
- _Binding_ is what you call this layer-specific development. The communication processes that are related to each other are bound at that layer.

> Please Do Not Throw Sausage Pizza Away (layer 1 -> 7)

|Layer | Name |  Group | Function | Example |
|---|---|---|---|---|
| 7 | Application |  Upper | File, print, db, app services | Provide a UI |
| 6 | Presentation |  Upper | Data encryption, compression, translation services | Presents data, handles encyrption processing |
| 5 | Session |  Upper | Dialog control | Keeps different application data separate |
| 4 | Transport |  Lower | e2e connections | Delivery, error correction before retransmission |
| 3 | Network |  Lower | Routing | Logical addressing, which router to use for path determination |
| 2 | Data link  | Lower | Framing | Combine packets into bytes, bytes into frames, MAC address access, error detection |
| 1 | Physical |  Lower | Physical topology | Move bits between devices, specify voltage, wire speed, pin-out of cables |

- Users interact with application layer
- Upper layers are responsible for applications communicating between hosts
  - Don't know anything about networking or network addresses
- Lower layers define how data is transferred through physical media, switches, and routers
  - How to rebuild a data stream from a transmitting host to a destination host

### Application (7)

The layer where users communicate or interact with the computer. Users interact with the network stack through application processes, interfaces, or APIs that connect the app to the OS:
- Comes into play when it is clear that network access is needed
- Acts as an interface between the application program and the next layer down by provideing ways for the application to send information down through the protocol stack
  - For example, Google Chrome doesn't reside within the application layer--it interfaces with Application layer protocols when it needs to work with remote resources
- Identifies and establishes the availability of the intended communication partner and determines whether there are sufficient resources for the communication to occur

### Presentation (6)

Presents data to the Application layer, translates data, and formats code.
- Data-transfer should adapt the data into a standard format before transmission
  - Computers receive generically formatted data and convert it to its native format (e.g. Unicode -> ASCII)
  - Makes sure that data from one system's Application layer can be read by another systsm's Application layer
- Examples of data formatting:
  - compression
  - decompression
  - encryption
  - decryption

### Session layer (5)

Keeps one applications data separate from another application's data:
- Sets up, manages, tears down sessions between Presentation layer entities, and provides dialogue control between devices or nodes.
- Coordinates communications btwn systems w one of three modes:
  - simplex: one direction
  - half-duplex: both directions, but only one at a time
  - full-duplex: bidirectional 

### Transport layer (4)

Segments and reassembles data into a data stream for upper-level applications
- provide e2e data transport services
- can establish logical connection between the sending host and destination host 
- provides mechanisms for multiplexing upper-layer applications, establishing virtual connections, and tearing down virtual circuits
- TCP and UDP work at the Rransport layer
- _Reliable networking_ means that the Transport layer employs acknowledgements, sequencing, and flow control

#### Connection-oriented communication

A service is considered _connection-oriented_ if it uses the following:
- a _virtual circuit_ (three-way handshake)
- sequencing (tracks packet ordering)
- acknowledgements
- flow control

When the sender TCP process contacts the destination's TCP process to establish connection:
- Called a _virtual circuit_. Setting up the connection is called _overhead_.
- During this _handshake_ the TCP processes also agree on the amount of information sent in either direction
  - Reciever sends back an acknowledgement

Three-way handshake process:
1. Connection agreement segment (req for synchronization)
2. Acknowledge request and establish connection parameters (rules) between hosts. Also request synchronization from the original sender to create bidirectional connection.
3. Acknowledgement that tells destination host the agreement is accepted and connection has been established. 
4. Begin data transfer
5. Final segment is sent

```bash
Sender              Receiver
            SYN
    SYN ------------> # (1)
SYN/ACK <------------ # (2)
    ACK ------------> # (3)
    
    <Connection established>

   Data ------------> # (4)
    FIN ------------> # (5)
```

#### Flow control

To ensure data integrity, the receiver uses _flow control_ to govern the amount of data sent by the sender.
- Prevent buffer overflow, which can cause data loss
- Protocols ensure the following:
  - Delivered segments are acknowledged back to sender
  - Segments that are not acknowledged are retransmitted
  - Segments are correctly sequenced when they reach the destination
  - Manageable data flow is maintained to avoid congestion, overloading, or data loss

Buffer
: When too many datagrams are received for the process, the machine stores them in a buffer.
  - Helps when the datagrams are part of a small burst
  - If not small burst, the buffer overflows and datagrams are lost
  - Receiver can send a 'Not ready' or 'Stop' signal to sender
  - When receiver processes the segments in the buffer, it sends the 'ready' transport indicator. Sender resumes transmission
  - If data segments are lost, duplicated, or damaged, a failure notice is sent to the sender.

#### Windowing

The number of data segments (TCP/IP measures in bytes) that the sender can transmit without receiving an acknowledgement.
- There is time between when the sender sends a data segment and receives a response
- You can send additional data segments before you receieve the ACK from the receiver
- Window size delimits the number of bytes that can be sent at a time
  - If the window size is 1, the sender waits for an ACK before sending another
  - If the window size is 3, the sender sends 3 before it waits for an ACK from the receiver

#### Acknowledgements

To guarantee that data isn't duplicated or lost, there is a _positive acknowledgement with retransmission_. This requires that a receiving sends an acknowledgment message back to the sender when it receives data.
- The sender starts a timer and retransmits if it expires before it receives an acknowledgement from the receiver
- Sender sends segment 1, 2, 3, then receiver sends ACK requesting segment 4
- Transport layer doesn't require connection-oriented service

```
Sender                  Receiver

Send 1 --------------->
Send 2 --------------->
Send 3 --------------->
       < -------------- ACK 4
Send 4 --------------->
Send 5 ----> Lost
Send 6 --------------->
       < -------------- ACK 5
Send 5 --------------->
       < -------------- ACK 7
Send 7 --------------->
```

### Network (3)


- Manages logical device addressing
- tracks location of devices on the network
- determines best way to move data

- Routers are layer 3 and move data across networks
  1. Router receives packet
  2. Router checks the destination IP address
     1. If the packet isn't for that router, the router checks the routing table and sends it along
     2. If the router can't find an entry for the packet's destination network, it drops the packet
  3. When an _exit interface_ is selected, the router sends the packet to the interface
  4. The interface frames the packet and sends it out on the local network


There are two types of packets at Network layer:
- **Data packets**: Transport user data through the network. Protocols that support data traffic are called _routed protocols_, such as IPv4 and IPv6
- **Route-Update Packets**: Help build and maintatin routing tables on routers. Update neighboring routers about the networks connected to all routers within the internetwork. Protocols that support route-update packets are _routing protocols_:
  - Routing Information Protocol (RIP)
  - RIPv2
  - Enhanced Interior Gateway Routing Protocol (EIGRP)
  - Open Shortest Path First (OSPF)

Network addresses
: Protocol-specific network addresses. A router maintains a routing table for individual routing protocols bc each routing protocol keeps track of a network that includes different addressing schemes such as IP and IPv6.

Exit interface
: The interface that a packet takes when going to a specific network. Each interface on a router represents a separate network.

Metric
: The distance to a remote network. Different protocols use different units to compute distance. For example, RIP uses hop count--the number of routers a packet passes through. Other units include bandwidth, delay of the line, and tick count (1/18 sec)

#### Routers

- Routers locate networks, not machines
- Also called a layer 3 switch
- Routers break up broadcast domains, and switches break up collision domains
- Do not forward broadcast or multicast packets
- Forward packets using logical addresses
- Can have access lists that control security on the tupes of packets that are allowed to enter or exit an interface
- Can provide layer 2 bridging functions
- Can simultaneously route through same interface
- Layer 3 routers provide connections between vLANs
- Provide quality of service (QoS) for specific types of network traffic

### Data link (2)

Physically transmits the data and handles error notification, network topology, and flow control.
- When a packet is sent between routers, it is framed with control info at the data link layer
  - Formats messages into pieces called data frames
  - Data frame adds a custom header with destination and source address
  - When the correct router is found, the control info is stripped and only the original packet remains
- Responsible for unique identification of each device that resides on a local network
- Uses MAC address to find correct physical device
- Translates messages from network layer into bits to go across the wire

IEEE ethernet data link layer has two sublayers:
- **Media access control (MAC)**: Defines how packets are placed on the media:
  - First come, first served
  - Physical addressing is defined here
  - Defines local topologies (how traffic actually travels through the physical topology)
  - Error notification
  - ordered delivery of frames
  - optional flow control
- **Logical Link Control (LLC)**: IDs network layer protocols and then encapsulates them. Tells Data Link layer what to do witha a packet when the frame is received:
  1. Host receives a frame and for destination in LLC header
  2. Provides flow control and sequencing of control bits

#### IEEE 802

Called 802 because they met in 1980 during February, the second month (19**80**, February (**2**)). Designation for a standard is always 802.<standard>:

[List of 802 standards](https://en.wikipedia.org/wiki/IEEE_802#Working_groups)

### Physical layer (1)

Does two important things: sends bits and receives bits, either 1 or 0:
- Specifies the electrical, mechanical, procedural, and functional requirements for activating, maintaining, and deactivating a physical link between end systems.
- This layer identifies the interface between the data terminal equipment (DTE) and the data communcation equipment (DCE)
- Specifies the layout of transmission media, or the topology (star, bus, ring, etc)

## Encapsulation

When you send data across a network, each layer of the OSI model encapsulates wraps the data with protocol information. This protocol information is used by the peer layer on the receiving device.
- Each layer uses protocol data units (PDUs), attached to header or trailer (end of packet)

Encapsulation steps:

1. User info is converted to data to transmit across the network
2. Data is converted to segments
3. Reliable connection is set up between sender and receiver
4. Segments are converted to packets (TCP) or datagrams (UDP). Logical address is placed in header along with the segment of data
5. Packets or datagrams are converted to frames for transmission on the local network. Frames carry packets
   - hardware addresses (ethernet) are used to ID hosts on local network
6. Frames are converted to bits, and digital encoding and clocking scheme is used

```
_____________________________
|            | Application  |                   ____________
| Datagrams  | Presentation |                   | Datagram |
|            | Session      |                   ------------
|------------|--------------|     -------------------------
| Segments   | Transport    |     | TCP header |   Data    |
|------------|--------------|     -------------------------- 
| Packets    | Network      |     | IP header  |   Data    |
|------------|--------------|   ----------------------------
| Frames     | Data link    |   | Frame | LLC | Data | FCS |
|------------|--------------|   ----------------------------
| Bits       | Physical     |   |     1101001011011010101  |
-----------------------------   ----------------------------
```

### Modulation techniques

Process of varying the _carrier signal_ that contains the information that you are transmitting.
- Modulator modulates the signal
- Demodulator demodulates the signal.
- Modems (modulator-demodulator) can perform both