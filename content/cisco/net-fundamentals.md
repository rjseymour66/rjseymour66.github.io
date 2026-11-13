+++
title = 'Network fundamentals'
date = '2026-11-11T09:51:12-05:00'
weight = 20
draft = false
+++

## Network devices

### Definitions

- **Wide area network (WAN):** A network that extends over a large geographical area, such as between cities.
- **Local area network (LAN):** A network that connects devices within a limited geographic area, such as an office or building.
- **Node:** Any device that connects to a network.
- **Resource:** Anything that can be accessed or used over the network.
- **Client and server:** Roles that a variety of devices can assume, not specific types of physical devices. Client and server nodes are also called endpoints or end hosts.

### Switches
Connects devices within a LAN. Doesn't provide connectivity between LANs or to external networks.

- **Port density:** Most switches have 24-48 ports.
- **Port:** A physical connector that serves as the interface between a device and the other devices on the network. The terms port and interface are often used interchangeably.

### Routers
A device that provides connectivity between LANs and the internet.

- **Home routers:** Combine the roles of router, switch, wireless access point (for Wi-Fi connectivity), and firewall in a single device. This makes them well-suited to a small office/home office (SOHO) network with only a few users.
- **Enterprise networks:** It's not feasible for a single device to fulfill all these roles at enterprise scale.

### Firewalls

- **Host-based firewall:** Examines network traffic entering and exiting the host device, then allows or denies (blocks) it based on a set of defined rules.
- **Network firewall:** A separate hardware appliance that serves a similar purpose on a larger scale. It inspects all traffic entering and exiting a network and allows or denies it based on a set of configured rules.

## Cables, Connectors, Ports

### Network standards

Ethernet isn't one single standard. It's a family of standards that define the physical aspects of wired network connections and the rules for communicating over them.

- **IEEE:** The Institute of Electrical and Electronics Engineers (IEEE, pronounced I-triple-E) is the governing body that defines these standards. In 1983, the IEEE first defined IEEE 802.3, better known as Ethernet.
- **IEEE 802.11:** Also defined by the IEEE. Better known, though not officially, as Wi-Fi.
- **Working groups:** The IEEE assigns working groups to develop specific technologies. The two groups most relevant to the CCNA are 802.3 (wired Ethernet) and 802.11 (wireless LANs, or Wi-Fi).

### Binary bits and bytes

Two computers connected by a cable communicate by sending each other long series of bits (0s and 1s), often at a rate of billions of bits per second. How the 0s and 1s are conveyed depends on the medium. For example, copper wiring conveys them by modifying the voltage of an electric signal between the two devices. Network connection speed is measured by how many bits transmit per second over the connection.

| Unit           | Bits                                    |
| :------------- | :-------------------------------------- |
| 1 kilobit (kb) | 1,000 bits                              |
| 1 megabit (Mb) | 1,000,000 bits (1,000 kilobits)         |
| 1 gigabit (Gb) | 1,000,000,000 bits (1,000 megabits)     |
| 1 terabit (Tb) | 1,000,000,000,000 bits (1,000 gigabits) |

### Copper UTP connections

The most common connection type in Ethernet LANs uses copper unshielded twisted pair (UTP) cables.

{{< admonition "Maximum length" tip >}}
The 100-meter maximum cable length is a major limitation that can make copper connections unfeasible in some cases.
{{< /admonition >}}

- **Unshielded:** The wires don't have a metallic shield around them. A shield can reduce electromagnetic interference (EMI), but UTP cables don't have one.
- **Twisted pair:** The eight wires in the cable are twisted together into four pairs of two wires each. Twisting the wires reduces EMI between the wires in each pair.
- **Connector:** The 8 position 8 contact (8P8C) connector has eight pins, one for each of the eight wires inside the cable. It's also called an RJ45 connector (RJ stands for Registered Jack).


#### IEEE 802.3 standards (copper)

| Speed    | Speed-derived name | IEEE task group | Informal name | Maximum cable length |
| :------- | :----------------- | :-------------- | :------------ | :------------------- |
| 10 Mbps  | Ethernet           | IEEE 802.3i     | 10BASE-T      | 100 m                |
| 100 Mbps | Fast Ethernet      | IEEE 802.3u     | 100BASE-T     | 100 m                |
| 1 Gbps   | Gigabit Ethernet   | IEEE 802.3ab    | 1000BASE-T    | 100 m                |
| 10 Gbps  | 10 Gig Ethernet    | IEEE 802.3an    | 10GBASE-T     | 100 m                |

- **Cabling standards:** Cable standards are defined by the Electronic Industries Alliance (EIA) and the Telecommunications Industry Association (TIA).
- **Maximum length:** Using a UTP cable longer than the listed maximum can cause signal attenuation and decreased performance.

#### Straight-through and crossover cables

- **Straight-through cable:** Connects the same pin pair on one connector to the same pin pair on the other connector. Works well when connecting a PC to a switch.
  - PCs transmit (Tx) on the 1-2 pin pair and receive (Rx) on the 3-6 pin pair.
  - Switches transmit (Tx) on the 3-6 pin pair and receive (Rx) on the 1-2 pin pair.
- **Pair count by standard:** 10BASE-T and 100BASE-T use two pairs (four wires). 1000BASE-T and 10GBASE-T use four pairs (eight wires).
- **Cable choice:** Using the correct cable matters most for 10BASE-T and 100BASE-T connections, to connect each pin to the correct pin on the other end.

#### Crossover cables

A crossover cable connects opposite pin pairs: pins 1 and 2 on one end connect to pins 3 and 6 on the other end. This lets devices that transmit on the same pin pair communicate with each other.

| Device type | Transmit (Tx) pins | Receive (Rx) pins |
| :---------- | :----------------- | :---------------- |
| Router      | 1 and 2            | 3 and 6           |
| Firewall    | 1 and 2            | 3 and 6           |
| PC/Server   | 1 and 2            | 3 and 6           |
| Switch      | 3 and 6            | 1 and 2           |

#### Auto MDI-X

- **Auto MDI-X:** Short for Auto Medium-Dependent Interface Crossover. Lets a device automatically change which pins it uses to transmit and receive data, based on the device it's connected to.

#### 1000BASE-T and 10GBASE-T

- Use all eight wires in the cable, as four wire pairs.
- Use the same 1-2 and 3-6 pin/wire pairs as 10BASE-T and 100BASE-T, plus two more: the pair at positions 4 and 5, and the pair at positions 7 and 8.
- Each wire pair can transmit and receive data simultaneously.

### Fiber-optic connections

Fiber-optic cables transmit light signals along a glass fiber instead of electrical signals along a copper wire.

- **Handling:** Handle fiber-optic cables with care. A sharp bend in the cable can damage the glass fiber and render the cable unusable.
- **Cost:** Fiber-optic connections are more expensive than copper UTP connections, largely because of the cost of SFP transceivers.

#### Anatomy of a fiber-optic cable

A typical fiber-optic connection uses two cables, one for transmitting data and one for receiving data. These cables connect to a Small Form-Factor Pluggable (SFP) transceiver, which is inserted into an SFP port on the device.

There are two main types of fiber-optic cabling: multimode fiber (MMF) and single-mode fiber (SMF).

| Attribute        | Multimode fiber (MMF)                                | Single-mode fiber (SMF)         |
| :--------------- | :--------------------------------------------------- | :------------------------------ |
| Core width       | Wider                                                | Narrower                        |
| Light source     | LED                                                  | Laser (more expensive than LED) |
| Light path       | Multiple angles (modes), reflecting off the cladding | Single angle                    |
| Maximum distance | Several hundred meters                               | Tens of kilometers              |

### UTP vs fiber

- **UTP:** Most common for connections from switches to end hosts. In an office setting, switches are typically on each floor, and the 100-meter maximum cable length is usually enough for an end host to reach a switch on its floor.
- **Fiber-optic:** More common for connections between network infrastructure, for example, connecting switches and routers located on separate floors or in separate buildings.
- **EMI vulnerability:** UTP cables are vulnerable to EMI and can emit (leak) their signal outside of the cable.
- **Client devices:** Most client devices, such as PCs, don't have SFP ports for fiber-optic connections, so a UTP connection is the only choice.


## TCP/IP networking models

A networking model is a framework that defines the functions data needs to travel from source to destination. Models divide these functions into layers, and each layer performs a specific role in network communication.

This section covers two models:

- **Open Systems Interconnection (OSI) model**
- **TCP/IP model:** Named after two of its key protocols, Transmission Control Protocol (TCP) and Internet Protocol (IP). Modern networks worldwide use the TCP/IP model.

### OSI reference model

The OSI model is a seven-layer conceptual framework that describes the functions involved in network communication.

To remember the layers from Layer 1 to Layer 7, use a mnemonic such as "Please Do Not Teach Students Pointless Acronyms." The first letter of each word matches the first letter of a layer name.

| Layer | Name         |
| :---- | :----------- |
| 7     | Application  |
| 6     | Presentation |
| 5     | Session      |
| 4     | Transport    |
| 3     | Network      |
| 2     | Data Link    |
| 1     | Physical     |


### TCP/IP model

The TCP/IP model grew out of research that the US Department of Defense (DoD) Defense Advanced Research Projects Agency (DARPA) funded. Originally called the ARPANET reference model, it evolved into the Internet Protocol Suite that Request for Comments (RFC) 1122 defines. The Internet Engineering Task Force (IETF) publishes RFCs to define standard internet protocols.

Each layer has a distinct role:

- **Layer 7 (Application):** Provides the interface between applications and the network.
- **Layer 4 (Transport):** Delivers messages from application to application.
- **Layer 3 (Network):** Delivers messages end to end.
- **Layer 2 (Data Link):** Delivers messages hop to hop.
- **Layer 1 (Physical):** Defines the physical medium that carries the communication.

The following table maps the OSI layers to the four-layer and five-layer TCP/IP models:

<table class="merged">
  <thead>
    <tr>
      <th>OSI model</th>
      <th>Four-layer TCP/IP model</th>
      <th>Five-layer TCP/IP model</th>
      <th>Example protocols</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Application</td>
      <td rowspan="3">Application</td>
      <td rowspan="3">Application</td>
      <td rowspan="3">HTTP<br>HTTPS<br>FTP<br>SSH</td>
    </tr>
    <tr>
      <td>Presentation</td>
    </tr>
    <tr>
      <td>Session</td>
    </tr>
    <tr>
      <td>Transport</td>
      <td>Transport</td>
      <td>Transport</td>
      <td>TCP<br>UDP</td>
    </tr>
    <tr>
      <td>Network</td>
      <td>Internet</td>
      <td>Network</td>
      <td>IPv4<br>IPv6</td>
    </tr>
    <tr>
      <td>Data Link</td>
      <td rowspan="2">Link</td>
      <td>Data Link</td>
      <td rowspan="2">Ethernet<br>802.11 (Wi-Fi)</td>
    </tr>
    <tr>
      <td>Physical</td>
      <td>Physical</td>
    </tr>
  </tbody>
</table>

#### Physical layer

The Physical layer defines the physical requirements for transmitting data, a series of bits, from one node to another. Those bits can travel as electrical signals on a copper cable, light signals on a fiber-optic cable, or radio waves on a wireless connection.

#### Data link layer

The Data Link layer prepares data for transmission over the physical medium so the next node in the path can receive it. It forwards the message from one hop to the next until the message reaches the destination host. To deliver messages hop to hop, the Data Link layer uses media access control (MAC) addresses.

A **hostname** is a name that identifies a device on the network.

#### Network layer

Hop-to-hop delivery alone doesn't get a message to its destination. The source host also needs a way to address the message to the final destination host. The Network layer provides this end-to-end delivery.

Layers 2 and 3 work together to deliver a message, and this relationship is a fundamental networking concept:

- Layer 2 uses MAC addresses to deliver messages hop to hop.
- Layer 3 uses IP addresses to deliver messages end to end.
- Together, Layers 2 and 3 move a message through the network to its final destination.
- The destination IP address stays the same for the entire journey. The destination MAC address changes at each hop.

#### Transport layer

The Transport layer addresses data to a specific application process on the destination host. Layer 4 uses its own addressing scheme: **port numbers**. When you address a message to a port, the message reaches the matching application process on the destination host.

The two most common Layer 4 protocols are Transmission Control Protocol (TCP), the "TCP" in TCP/IP, and User Datagram Protocol (UDP). Both protocols address messages to specific application services on the destination host, but they differ in several ways.

#### Application layer

The Application layer is most commonly called Layer 7. Applications use Layer 7 protocols to prepare messages for transmission over the network.

Layer 7 protocols such as HTTPS aren't user applications. They provide services that let user applications communicate with applications on other computers over the network.


### Data encapsulation

**Encapsulation** is the five-step process a host uses to send data. Each layer adds its own information to the data before passing it down.

1. **Layer 7:** The Application layer protocol prepares the data.
2. **Layer 4:** Adds a header addressed to a port number on the destination host.
3. **Layer 3:** Adds a header addressed to the IP address of the destination host.
4. **Layer 2:** Adds a header addressed to the MAC address of the next-hop device. It also adds a trailer, which is used to check for errors.
5. **Layer 1:** The host transmits the bits over the physical medium, such as electrical signals over a UTP cable.

| Layer | Adds             | Addressed to                        |
| :---- | :--------------- | :---------------------------------- |
| 7     | (Prepares data)  | N/A                                 |
| 4     | Header           | Port number on the destination host |
| 3     | Header           | IP address of the destination host  |
| 2     | Header + trailer | MAC address of the next hop         |
| 1     | (Transmits bits) | N/A                                 |

- **Header:** Supplemental data added to the front of a message before it's transmitted over a network. A protocol's header contains the data that protocol uses.
- **Trailer:** Supplemental data added to a message before it's transmitted over a network. The Ethernet trailer contains a small block of data used to check the message for errors.

### Data de-encapsulation

**De-encapsulation** reverses the encapsulation process on the destination host:

1. The destination host receives the message.
2. The host inspects the Layer 2 header and trailer, removes them, and passes the message to Layer 3.
3. The host inspects the Layer 3 header, removes it, and passes the message to Layer 4.
4. The host inspects the Layer 4 header, removes it, and sends the data to the appropriate application.
5. The application receives and processes the data.

### Protocol data units

Each layer has a name for its protocol data unit (PDU):

| PDU         | Layer | Also called | Contents                            |
| :---------- | :---- | :---------- | :---------------------------------- |
| **Segment** | 4     | L4PDU       | Data + Layer 4 header               |
| **Packet**  | 3     | L3PDU       | Segment + Layer 3 header            |
| **Frame**   | 2     | L2PDU       | Packet + Layer 2 header and trailer |

### Adjacent- and same-layer interactions

- **Adjacent-layer interaction:** Each layer of the TCP/IP model provides a service for the layer above it.
- **Same-layer interaction:** The same layer on different computers communicates.

The following table shows same-layer interaction at each layer:

| Layer | What the sender sends               | Where it's received and inspected                              |
| :---- | :---------------------------------- | :------------------------------------------------------------- |
| 7     | Application data                    | An application on another computer                             |
| 4     | Segment (data + Layer 4 header)     | Layer 4 of the destination host, which inspects the header     |
| 3     | Packet (segment + Layer 3 header)   | Layer 3 of the destination host, which inspects the header     |
| 2     | Frame (packet + header and trailer) | Layer 2 of the next hop, which inspects the header and trailer |
| 1     | Signals out of a physical port      | A physical port on another device                              |


## Ethernet LAN switching

Switches are **Layer 2 devices**, which means they operate at Layer 2. Switches and routers both make forwarding decisions, but they use different headers:

| Device | Forwards based on | Header   |
| :----- | :---------------- | :------- |
| Switch | Layer 2 header    | Ethernet |
| Router | Layer 3 header    | IP       |

### LANs

Another term for a LAN is a **Layer 2 domain**. A Layer 2 domain is a portion of a network where:

- Switches forward frames.
- Hosts connected to the switches can communicate with each other without a router.

### Ethernet header and trailer

Switches make forwarding decisions with information in the Ethernet header. To understand switching, you need to understand the contents of the header and trailer.

The Preamble and Start Frame Delimiter (SFD) aren't part of the Ethernet frame.

#### Preamble and SFD

- The Preamble and SFD let the receiving device synchronize its receiver clock and prepare to receive the incoming frame.
- Clock synchronization affects how the receiving device interprets the incoming electrical signals. The receiving device must determine the precise length of 1 bit.
- The device sends the Preamble and SFD with every frame, but they aren't part of the frame. They're purely a Layer 1 (Physical layer) function, and they don't contain information that influences what the receiving device does with the frame.

#### Destination and Source

- The **Destination** field is the destination MAC address of the frame.
- The **Source** field is the source MAC address of the frame.
- Layer 2 provides hop-to-hop delivery of messages, and MAC addresses enable that delivery.
- Within a LAN, a switch reads the destination MAC address of a frame and forwards the frame to the appropriate destination.

#### MAC addresses

- A MAC address is a 6-byte (48-bit) address that the manufacturer assigns to each port of a network device.
- Another name for a MAC address is a **burned-in address (BIA)**, because the address is "burned into" the physical port.
- MAC addresses are globally unique.
- The first 3 bytes (the first half) are an **organizationally unique identifier (OUI)**. The IEEE assigns the OUI to the manufacturer.
- The last 3 bytes are unique to the port itself.
- MAC addresses are written as 12 hexadecimal characters.

{{< admonition "Number prefixes" note >}}
MAC addresses use hexadecimal. The prefix `0d` indicates a decimal number, and `0x` indicates a hexadecimal number.
{{< /admonition >}}

#### Type/Length

The Type/Length field is 2 bytes. It indicates one of two things:

- **Type:** The type of the encapsulated packet, such as an IPv4 packet or an IPv6 packet. When the field indicates type, it's called the **EtherType** field.
- **Length:** The length of the encapsulated packet, in bytes.

In almost all cases, the field indicates type. A special signal after the frame marks the end of the frame, so the field doesn't need to indicate length.

#### Frame Check Sequence

The Frame Check Sequence is the only field in the Ethernet trailer. Its 4 bytes in length. It works like this:

1. The sender calculates a checksum for the frame.
2. The frame's destination host receives the frame and calculates its own checksum with the same algorithm.
3. The receiver compares its checksum to the sender's checksum.
4. If the checksums are different, the receiver discards the frame.

The checksum algorithm is a **cyclic redundancy check (CRC)**:

| Term       | Meaning                                                                                                 |
| :--------- | :------------------------------------------------------------------------------------------------------ |
| Cyclic     | The kind of algorithm used to calculate the checksum                                                    |
| Redundancy | The field is redundant. It expands the size of the message but doesn't add any information.             |
| Check      | The field checks whether the frame traveled from source to destination without the data being corrupted |



### Frame switching

#### MAC address learning

- To decide how to forward a frame, a switch looks up the frame's destination MAC address in its **MAC address table**. The MAC address table lists the MAC addresses in the LAN and the port each one is connected to.
- Another name for the MAC address table is the **CAM table**. The name comes from the kind of memory that stores the table: content addressable memory (CAM).
- When a switch receives a frame on a port, it examines the Source field. It then creates an entry in its MAC address table that associates the source MAC address with that port. This process is called **MAC address learning**.

| Type                    | How the switch gets it                          |
| :---------------------- | :---------------------------------------------- |
| **Dynamic MAC address** | Learned automatically (dynamically)             |
| **Static MAC address**  | Configured manually (statically). This is rare. |

- **MAC aging:** A switch removes a dynamic MAC address from its MAC address table after 5 minutes of inactivity. Inactivity means the switch doesn't receive a frame from that MAC address for 5 minutes.

#### Frame flooding and forwarding

- A **unicast frame** is a frame addressed to a single destination host.
- A **known unicast frame** is a unicast frame whose destination MAC address already has an entry in the switch's MAC address table.
- An **unknown unicast frame** is a unicast frame whose destination MAC address has no entry in the switch's MAC address table.
- To **flood** a frame is to send it out of all ports except the port the frame was received on.

The switch action depends on the kind of unicast frame:

| Frame                 | Action      | What the switch does                                                         |
| :-------------------- | :---------- | :--------------------------------------------------------------------------- |
| Known unicast frame   | **Forward** | Sends the frame out of the port in the destination MAC address's table entry |
| Unknown unicast frame | **Flood**   | Sends the frame out of all ports except the one it was received on           |

For example:

- **Known unicast:** PC1 sends a frame to PC2's MAC address. SW1 checks its MAC address table and sees that it should forward the frame out of its G0/2 port.
- **Unknown unicast:** The switch uses the frame's Source field to learn PC1's MAC address. It then floods the frame.

A switch is **transparent** to its connected hosts:

- PC1 and PC3 address their messages directly to each other, not to SW1 or SW2. This is why a message passing through a switch isn't considered a hop.
- Switches don't modify frames.

#### MAC address table in Cisco IOS

To view the MAC address table, run `show mac address-table`:

```bash
SW1# show mac address-table
          Mac Address Table
-------------------------------------------
  
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    5254.0017.7cd2    DYNAMIC     Gi0/0
   1    d8bb.c1cc.ff01    DYNAMIC     Gi0/1
   1    d8bb.c1cc.ff02    DYNAMIC     Gi0/2
   1    d8bb.c1cc.ff03    DYNAMIC     Gi0/0
   1    d8bb.c1cc.ff04    DYNAMIC     Gi0/0
```

To clear all dynamic MAC addresses, run `clear mac address-table dynamic`:

```bash
SW1# clear mac address-table dynamic
SW1# show mac address-table                     
          Mac Address Table
-------------------------------------------
  
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
```

You can also remove a specific address, or all addresses learned on a specific port:

| Command                                                      | Description                                              |
| :----------------------------------------------------------- | :------------------------------------------------------- |
| `clear mac address-table dynamic`                            | Clears all dynamic MAC addresses                         |
| `clear mac address-table dynamic address` *mac-address*      | Clears a specific dynamic MAC address                    |
| `clear mac address-table dynamic interface` *interface-name* | Clears all dynamic MAC addresses learned on an interface |

### Address Resolution Protocol (ARP)

ARP lets a host learn the MAC address of another host in the LAN. ARP uses two messages:

| Message     | Purpose                                         |
| :---------- | :---------------------------------------------- |
| ARP request | Asks another host what its MAC address is       |
| ARP reply   | Informs another host of this host's MAC address |

- The ARP request is sent in a **broadcast** frame, not a unicast frame.
- The ARP reply is unicast.
- The sender specifies which host's MAC address it wants by including that host's IP address in the ARP request.
- ARP maps a known Layer 3 address (IP address) to an unknown Layer 2 address (MAC address). ARP can be thought of as the bridge between Layers 2 and 3 of the TCP/IP model.
- A host stores learned MAC addresses in its **ARP table**, a list of IP addresses and their associated MAC addresses.
- A **broadcast domain** is a group of devices that receive a broadcast frame sent by any member of the group.

#### ARP request and reply

1. PC1 sends an ARP request addressed to the broadcast MAC address (`ffff.ffff.ffff`).
2. SW1 floods the frame. PC2 sees the ARP request isn't for its own IP address, so it drops the message.
3. SW2 floods the frame. PC4 sees the ARP request isn't for its own IP address, so it drops the message. PC3 sees the ARP request is for its own IP address.
4. PC3 sends an ARP reply to PC1.
5. SW2 forwards the ARP reply out of its G0/0 port.
6. SW1 forwards the ARP reply out of its G0/1 port. PC1 now knows PC3's MAC address, so it can send the original message to PC3.

After the ARP request-reply exchange, SW1 and SW2 have already learned the MAC addresses of PC1 and PC3. When PC1 sends its original message to PC3, the switches don't flood it. They forward it out of the appropriate port, because it's a known unicast message.

#### Message types

| Type      | Delivery                                 |
| :-------- | :--------------------------------------- |
| Unicast   | One-to-one                               |
| Broadcast | One-to-all                               |
| Multicast | One-to-multiple, but not necessarily all |

### Ping

**Ping** is a software utility that tests the reachability of hosts over a network.

- Ping is a component of the Internet Control Message Protocol (ICMP).
- Ping uses two messages: an **ICMP echo request** and an **ICMP echo reply**. Unlike ARP, both messages are unicast.
- Ping can also measure the round-trip time (RTT) between two hosts. RTT is the time it takes a message to travel from one host to another and back.
- **Destination IP address:** How a host knows the IP address of the host it wants to send a message to depends on the situation. Two possibilities:
  - The **Domain Name System (DNS)** converts hostnames, such as `manning.com`, into IP addresses.
  - The user of the device manually specifies the IP address, such as when using ping to test connectivity.

```bash
PC1# ping 10.0.0.12
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.12, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 3/3/4 ms
```

The five exclamation marks (`!!!!!`) in the output indicate successful pings. Each one is an ICMP echo request that received an ICMP echo reply.


## IPv4 addressing

- Routers use information in the Layer 3 header to make forwarding decisions.
- IPv4 is the foundation of modern networks such as the internet. RFC 791, titled "Internet Protocol," officially defines IPv4.

### IPv4 header

The IPv4 header length is variable because the Options field is optional and variable in size:

| Options field         | Header length |
| :-------------------- | :------------ |
| Not present           | 20 bytes      |
| Maximum size (40 bytes) | 60 bytes    |

The 20-byte length runs from the first bit of the Version field to the last bit of the Destination Address field.

The following diagram shows the layout of the IPv4 header fields:

<div class="bitfield-wrap">
<table class="merged bitfield">
  <thead>
    <tr><th></th><th>Byte</th><th colspan="8">0</th><th colspan="8">1</th><th colspan="8">2</th><th colspan="8">3</th></tr>
    <tr><th>Byte</th><th>Bit</th><th>0</th><th>1</th><th>2</th><th>3</th><th>4</th><th>5</th><th>6</th><th>7</th><th>8</th><th>9</th><th>10</th><th>11</th><th>12</th><th>13</th><th>14</th><th>15</th><th>16</th><th>17</th><th>18</th><th>19</th><th>20</th><th>21</th><th>22</th><th>23</th><th>24</th><th>25</th><th>26</th><th>27</th><th>28</th><th>29</th><th>30</th><th>31</th></tr>
  </thead>
  <tbody>
    <tr><th>0</th><th>0</th><td colspan="4">Version</td><td colspan="4">IHL</td><td colspan="6">DSCP</td><td colspan="2">ECN</td><td colspan="16">Total Length</td></tr>
    <tr><th>4</th><th>32</th><td colspan="16">Identification</td><td colspan="3">Flags</td><td colspan="13">Fragment Offset</td></tr>
    <tr><th>8</th><th>64</th><td colspan="8">Time To Live</td><td colspan="8">Protocol</td><td colspan="16">Header Checksum</td></tr>
    <tr><th>12</th><th>96</th><td colspan="32">Source Address</td></tr>
    <tr><th>16</th><th>128</th><td colspan="32">Destination Address</td></tr>
    <tr><th>20</th><th>160</th><td colspan="32" rowspan="3">Options</td></tr>
    <tr><th>&vellip;</th><th>&vellip;</th></tr>
    <tr><th>56</th><th>448</th></tr>
  </tbody>
</table>
</div>

#### Version field

The Version field indicates which version of IP the packet uses:

| Value             | Version |
| :---------------- | :------ |
| `0b0100` (`0d4`)  | IPv4    |
| `0b0110` (`0d6`)  | IPv6    |

#### IHL field

- The **Internet Header Length (IHL)** field is the second field in the header. It's 4 bits and indicates the length of the IPv4 header.
- The field is necessary because the header length varies. The Options field may or may not be present, and its own length varies.
- Any value greater than 5 indicates that the Options field is present.
- The maximum value is 15, which indicates a 60-byte header, the maximum IPv4 header length.

#### DSCP and ECN fields

| Field                                     | Length |
| :---------------------------------------- | :----- |
| Differentiated Services Code Point (DSCP) | 6 bits |
| Explicit Congestion Notification (ECN)    | 2 bits |

These fields are used for **Quality of Service (QoS)**, a network feature that prioritizes specific types of network traffic over other types.

#### Total Length field

The Total Length field indicates the total length of the packet: the IPv4 header and its payload.

| Field        | Measures                    | Unit                 |
| :----------- | :-------------------------- | :------------------- |
| IHL          | IPv4 header only            | 4-byte increments    |
| Total Length | IPv4 header and payload     | Bytes                |

#### Identification, Flags, and Fragment Offset fields

The Identification, Flags, and Fragment Offset fields total 32 bits. Together, they support **fragmentation**: breaking a packet into multiple smaller packets called **fragments**.

- IPv4 uses the **maximum transmission unit (MTU)** to indicate the maximum size a packet should be. Any packet larger than the MTU is fragmented.
- The final destination host of the packet reassembles the fragments to restore the original packet.
- The typical MTU is 1,500 bytes. Fragmentation can happen when:
  - A router in the packet's path to the destination has a lower MTU. The router fragments the packet.
  - A host sends packets larger than the standard 1,500-byte size. Packet sizes up to 9,000 bytes are sometimes used.

##### Identification field

Identifies which original packet a fragment belongs to.

##### Flags field

Controls and identifies fragments:

| Bit | Name                 | Meaning                                                                                                                                                 |
| :-- | :------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 0   | Reserved             | Its use isn't defined, so it's always set to 0.                                                                                                         |
| 1   | Don't Fragment (DF)  | If set to 1, the packet shouldn't be fragmented. If the packet is larger than the MTU, it's discarded.                                                  |
| 2   | More Fragments (MF)  | If set to 1, more fragments remain and this one isn't the last. The final fragment has a value of 0. An unfragmented packet always has a value of 0.   |

##### Fragment Offset field

- Indicates the position of the fragment within the original packet.
- If there are multiple paths to a destination, different fragments might take different paths and arrive out of order. This is rare.

#### TTL field

- The **Time To Live (TTL)** field is 8 bits.
- When a host sends a packet, it sets an initial value in this field. A common value is 64.
- Each router that forwards the packet decreases the value by 1.
- If the value reaches 0, the router drops the packet.
- TTL prevents packets from looping around the network infinitely. A **loop** is when a message travels around the network without being able to find its destination.

#### Protocol field

Indicates what kind of message is encapsulated in the packet:

| Value | Protocol                          |
| :---- | :-------------------------------- |
| 1     | ICMP                              |
| 6     | Transmission Control Protocol (TCP) |
| 17    | User Datagram Protocol (UDP)      |
| 89    | Open Shortest Path First (OSPF)   |

#### Header Checksum field

Checks for errors in the IPv4 header:

| Field                     | Checks           |
| :------------------------ | :--------------- |
| IPv4 Header Checksum      | IPv4 header only |
| Ethernet FCS              | Entire frame     |

#### Source Address and Destination Address fields

| Field               | Contains                                    |
| :------------------ | :------------------------------------------ |
| Source Address      | IP address of the host sending the packet   |
| Destination Address | IP address of the intended recipient        |

#### Options field

The Options field is rarely used.
