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

### Application

The layer where users communicate or interact with the computer. Users interact with the network stack through application processes, interfaces, or APIs that connect the app to the OS:
- Comes into play when it is clear that network access is needed
- Acts as an interface between the application program and the next layer down by provideing ways for the application to send information down through the protocol stack
  - For example, Google Chrome doesn't reside within the application layer--it interfaces with Application layer protocols when it needs to work with remote resources
- Identifies and establishes the availability of the intended communication partner and determines whether there are sufficient resources for the communication to occur

### Presentation

Presents data to the Application layer, translates data, and formats code.
- Data-transfer should adapt the data into a standard format before transmission
  - Computers receive generically formatted data and convert it to its native format (e.g. Unicode -> ASCII)
  - Makes sure that data from one system's Application layer can be read by another systsm's Application layer
- Examples of data formatting:
  - compression
  - decompression
  - encryption
  - decryption

### Session layer

Keeps one applications data separate from another application's data:
- Sets up, manages, tears down sessions between Presentation layer entities, and provides dialogue control between devices or nodes.
- Coordinates communications btwn systems w one of three modes:
  - simplex: one direction
  - half-duplex: both directions, but only one at a time
  - full-duplex: bidirectional 

### Transport layer

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

