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

| Options field           | Header length |
| :---------------------- | :------------ |
| Not present             | 20 bytes      |
| Maximum size (40 bytes) | 60 bytes      |

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

| Value            | Version |
| :--------------- | :------ |
| `0b0100` (`0d4`) | IPv4    |
| `0b0110` (`0d6`) | IPv6    |

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

| Field        | Measures                | Unit              |
| :----------- | :---------------------- | :---------------- |
| IHL          | IPv4 header only        | 4-byte increments |
| Total Length | IPv4 header and payload | Bytes             |

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

| Bit  | Name                | Meaning                                                                                                                                              |
| :--- | :------------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0    | Reserved            | Its use isn't defined, so it's always set to 0.                                                                                                      |
| 1    | Don't Fragment (DF) | If set to 1, the packet shouldn't be fragmented. If the packet is larger than the MTU, it's discarded.                                               |
| 2    | More Fragments (MF) | If set to 1, more fragments remain and this one isn't the last. The final fragment has a value of 0. An unfragmented packet always has a value of 0. |

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

| Value | Protocol                            |
| :---- | :---------------------------------- |
| 1     | ICMP                                |
| 6     | Transmission Control Protocol (TCP) |
| 17    | User Datagram Protocol (UDP)        |
| 89    | Open Shortest Path First (OSPF)     |

#### Header Checksum field

Checks for errors in the IPv4 header:

| Field                | Checks           |
| :------------------- | :--------------- |
| IPv4 Header Checksum | IPv4 header only |
| Ethernet FCS         | Entire frame     |

#### Source Address and Destination Address fields

| Field               | Contains                                  |
| :------------------ | :---------------------------------------- |
| Source Address      | IP address of the host sending the packet |
| Destination Address | IP address of the intended recipient      |

#### Options field

The Options field is rarely used.


### IPv4 address format

- Routers are **Layer 3 devices**. They make forwarding decisions based on the destination IP address in the Layer 3 header.
- Switches don't have IP addresses because they aren't Layer 3 aware.
- For routers, the term **interface** is typically used instead of port.

**Dotted decimal notation** writes an IPv4 address in two steps:

1. Split the 32-bit address into four groups of 8 bits called **octets**, separated by periods.
2. Convert each octet to decimal.

An IPv4 address has a network portion and a host portion. The prefix length indicates the size of the network portion in bits. The remaining bits are the host portion.

#### Prefix length

The **prefix length** indicates the size of the network portion in the format /X, where X is the number of bits in the network portion.

#### Netmasks

A **netmask** is often called a **subnet mask**. Each netmask bit maps to the bit in the same position of the IP address:

| Netmask bit | IP address bit belongs to |
| :---------- | :------------------------ |
| 1           | Network portion           |
| 0           | Host portion              |

#### Configure IPv4 addresses on a router

End hosts like PCs usually receive their IP addresses automatically using **Dynamic Host Configuration Protocol (DHCP)**. On a router, you configure them manually:

1. From user EXEC mode, move to privileged EXEC mode and then global configuration mode.
2. Access interface configuration mode for G0/0, configure an IP address and netmask, and enable the interface.
3. Access interface configuration mode for G0/1, configure an IP address and netmask, and enable the interface.

```bash
R1> enable
R1# configure terminal
R1(config)# interface g0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0 
R1(config-if)# no shutdown
R1(config-if)# interface g0/1
R1(config-if)# ip address 192.168.2.1 255.255.255.0
R1(config-if)# no shutdown
```

##### Verify the default state

Before you configure a device, confirm its current state:

```bash
R1# show ip interface brief
Interface           IP-Address   OK? Method Status                 Protocol
GigabitEthernet0/0  unassigned   YES unset  administratively down  down
GigabitEthernet0/1  unassigned   YES unset  administratively down  down
GigabitEthernet0/2  unassigned   YES unset  administratively down  down
GigabitEthernet0/3  unassigned   YES unset  administratively down  down
```

The default state is **administratively down**. Cisco router interfaces are disabled by default, and you must enable them manually.

##### Configure the interface

| Command                           | Mode                    | Description                            |
| :-------------------------------- | :---------------------- | :------------------------------------- |
| `interface` *interface-name*      | Global configuration    | Enters interface configuration mode.   |
| `ip address` *ip-address netmask* | Interface configuration | Configures the interface's IP address. |
| `no shutdown`                     | Interface configuration | Enables the interface.                 |

1. Enter interface configuration mode:

   ```bash
   R1# configure terminal
   Enter configuration commands, one per line.  End with CNTL/Z.
   R1(config)# interface gigabitethernet0/0
   R1(config-if)#
   ```

2. Configure the IP address:

   ```bash
   R1(config-if)# ip address 192.168.1.1 255.255.255.0
   ```

   G0/0 still isn't ready to forward traffic because the interface is still disabled.

3. Enable the interface:

   ```bash
   R1(config-if)# no shutdown
   %LINK-3-UPDOWN: Interface GigabitEthernet0/0,
    changed state to up
   %LINEPROTO-5-UPDOWN: Line protocol on Interface
    GigabitEthernet0/0, changed state to up
   ```

Router interfaces are disabled by default because they have the `shutdown` command applied. Putting `no` in front of a command removes it from the configuration, so `no shutdown` removes `shutdown` and enables the interface.

##### Verify the configuration

```bash
R1# show ip interface brief
Interface           IP-Address   OK? Method Status                 Protocol
GigabitEthernet0/0  192.168.1.1  YES manual up                     up
GigabitEthernet0/1  192.168.2.1  YES manual up                     up
GigabitEthernet0/2  unassigned   YES unset  administratively down  down    
GigabitEthernet0/3  unassigned   YES unset  administratively down  down
```

To double-check the netmask, use `show ip interface` [*interface-name*]:

| Command                  | Output                          |
| :----------------------- | :------------------------------ |
| `show ip interface`      | Information for all interfaces. |
| `show ip interface g0/0` | Information for G0/0 only.      |

Save the configuration with `write`, `write memory`, or `copy running-config startup-config`.

##### End-to-end and hop-to-hop delivery

- Layer 3 provides end-to-end delivery.
- Layer 2 provides hop-to-hop delivery.

Before PC1 can encapsulate the packet in a frame, it must use ARP to learn R1 G0/0's MAC address.

### Attributes of an IPv4 network

#### Network address

The **network address** is the first address of any network. It identifies the network and can't be assigned to a host.

#### Broadcast address

The **broadcast address** is the last address of any network. Like the network address, it can't be assigned to a host.

- You can use it to address a message to all hosts in the local network.
- An IPv4 address is a broadcast address if all bits of its host portion are set to 1.

| Broadcast address                                | Who uses it                                                                                              |
| :----------------------------------------------- | :------------------------------------------------------------------------------------------------------- |
| 255.255.255.255                                  | Hosts usually use this specially reserved address to send a message to all devices on the local network. |
| Network broadcast (for example, 192.168.100.255) | Hosts in other networks can use it to send a message to all hosts in that network (192.168.100.0/24).    |

#### Maximum number of hosts

| Value                   | Formula           | /24 example             |
| :---------------------- | :---------------- | :---------------------- |
| Total IP addresses      | 2<sup>y</sup>     | 2<sup>8</sup> = 256     |
| Maximum number of hosts | 2<sup>y</sup> − 2 | 2<sup>8</sup> − 2 = 254 |

- *y* is the number of host bits. A /24 prefix length has eight host bits.
- Subtract 2 because the network and broadcast addresses can't be assigned to hosts.

The following are the maximum number of hosts in networks with /8, /16, and /24 prefix lengths:

| Prefix length | Host bits | Total IP addresses | Maximum hosts |
| :------------ | :-------- | :----------------- | :------------ |
| /8            | 24        | 16,777,216         | 16,777,214    |
| /16           | 16        | 65,536             | 65,534        |
| /24           | 8         | 256                | 254           |

#### First and last usable address

| Address              | Definition                                                                                        | How to find it                                                                                      |
| :------------------- | :------------------------------------------------------------------------------------------------ | :-------------------------------------------------------------------------------------------------- |
| First usable address | The first IP address you can assign to a host. It's the first address after the network address.  | Change the host portion octets to 0 (the network address), then add 1 to the last octet.            |
| Last usable address  | The last IP address you can assign to a host. It's the last address before the broadcast address. | Change the host portion octets to 255 (the broadcast address), then subtract 1 from the last octet. |

The first usable address is often assigned to the network's router.

### IPv4 address classes

| Class | First octet bit pattern | First octet decimal range | Prefix length       |
| :---- | :---------------------- | :------------------------ | :------------------ |
| A     | 0xxxxxxx                | 0–127                     | /8 (255.0.0.0)      |
| B     | 10xxxxxx                | 128–191                   | /16 (255.255.0.0)   |
| C     | 110xxxxx                | 192–223                   | /24 (255.255.255.0) |
| D     | 1110xxxx                | 224–239                   | None                |
| E     | 1111xxxx                | 240–255                   | None                |

- Networks that follow the class A, B, and C rules are called **classful networks**.
- A smaller network portion means fewer networks with more hosts in each network.
- A larger network portion means more networks with fewer hosts in each network.
- The class system is now obsolete. **Classless networking** replaced it. In classless networking, prefix lengths aren't restricted by class.

#### Reserved class A ranges

| Range       | Purpose                                                      |
| :---------- | :----------------------------------------------------------- |
| 0.0.0.0/8   | Reserved. Any IP address with the first octet 0 is reserved. |
| 127.0.0.0/8 | Reserved for **loopback addresses**.                         |

A message sent to any loopback address (for example, `ping 127.0.0.1`) loops back to the local host, the device you're working on, without crossing the network. Use it to test the networking software on the local device.


## Router and switch interfaces

### Configuring interfaces

This section covers how to configure three aspects of an interface: description, speed, and duplex.

In the example topology, R1 G0/1 connects to SW1 G0/1. PC1 connects to SW1 F0/1, and PC2 connects to SW1 F0/2.

1. On R1, configure G0/1, the interface connected to SW1. Set the IP address and netmask, the description `## to SW1 ##`, a speed of 1000, and full duplex. Then enable the interface with `no shutdown`:

   ```bash
   R1(config)# interface g0/1
   R1(config-if)# ip address 192.168.1.1 255.255.255.0
   R1(config-if)# description ## to SW1 ##
   R1(config-if)# speed 1000
   R1(config-if)# duplex full
   R1(config-if)# no shutdown
   ```

2. On SW1, configure G0/1, the interface connected to R1. Set the description `## to R1 ##`, a speed of 1000, and full duplex:

   ```bash
   SW1(config)# interface g0/1
   SW1(config-if)# description ## to R1 ##
   SW1(config-if)# speed 1000
   SW1(config-if)# duplex full
   ```

3. On SW1, use `interface range` to configure F0/1 and F0/2, the interfaces connected to PC1 and PC2. Set the description `## end users ##`, and set speed and duplex to `auto`:

   ```bash
   SW1(config-if)# interface range f0/1-2
   SW1(config-if)# description ## end users ##
   SW1(config-if)# speed auto
   SW1(config-if)# duplex auto
   ```

4. On SW1, use `interface range` to configure the unused interfaces F0/3 through F0/8 and G0/2. Set the description `## not in use ##`, and disable the interfaces with `shutdown`:

   ```bash
   SW1(config-if)# interface range f0/3-8, g0/2
   SW1(config-if)# description ## not in use ##
   SW1(config-if)# shutdown
   ```


R1's G0/1 interface has an IP address, but SW1's interfaces don't:

- Switch interfaces don't need IP addresses to forward frames within a LAN.
- Switches aren't Layer 3 aware. They use only Layer 2 information (MAC addresses) to decide how to forward frames.

Switch interfaces are enabled by default so that switches can operate in a plug-and-play manner. A switch designed for plug-and-play use is called an **unmanaged switch**.

### Interface descriptions

An **interface description** commonly indicates what device is connected to the interface. Descriptions that you configure consistently and update as needed make it much easier to identify each interface's purpose when you view the configuration later.

| Command                                     | Description                                                                                                       |
| :------------------------------------------ | :---------------------------------------------------------------------------------------------------------------- |
| `description` *description*                 | Configures the interface description. *description* is a text string, such as "connected to R1's G0/1 interface." |
| `interface range` *type slot*/*port*-*port* | Configures a range of interfaces.                                                                                 |

| `interface range` parameter | Meaning                                             |
| :-------------------------- | :-------------------------------------------------- |
| *type*                      | Ethernet, FastEthernet, GigabitEthernet, and so on. |
| *slot*                      | The first number in the interface name.             |
| *port*                      | The second number in the interface name.            |

To include interfaces of different types in the same `interface range` command, separate the interface names with a comma.

```bash
SW1# configure terminal
SW1(config)# interface range f0/1-2
SW1(config-if)# description ## end users ##
```

The two hash symbols (`##`) at the beginning and end of the description aren't required. They make descriptions stand out in the CLI.

#### View interface descriptions

Use `show interfaces description` on a router or switch:

```bash
SW1# show interfaces description
Interface   Status         Protocol   Description
Fa0/1       up             up         ## end users ##
Fa0/2       up             up         ## end users ##
Fa0/3       down           down       ## not in use ##
. . .    
Gi0/1       up             up         ## to R1 ##
Gi0/2       down           down       ## not in use ##
```

You can also use `show interfaces status`:

```bash
SW1# show interfaces status
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/1     ## end users ##    connected    1          a-full  a-100 10/100BaseTX
Fa0/2     ## end users ##    connected    1          a-full  a-100 10/100BaseTX
Fa0/3     ## not in use ##   notconnect   1            auto   auto 10/100BaseTX
. . .
Gi0/1     ## to R1 ##        connected    1          a-full a-1000 10/100/1000BaseTX
Gi0/2     ## not in use ##   notconnect   1            auto   auto 10/100/1000BaseTX
```

### Interface speed

| Interface type  | Supported speeds                                    | Also called           |
| :-------------- | :-------------------------------------------------- | :-------------------- |
| FastEthernet    | 10 and 100 Mbps                                     | 10/100 interface      |
| GigabitEthernet | 10, 100, and 1,000 Mbps (1 Gbps) on most interfaces | 10/100/1000 interface |

With **autonegotiation**, the connected devices communicate with each other to determine the speed to operate at.

To configure an interface's speed manually, use `speed` {*speed* | `auto`}. Specify *speed* in megabits per second. You must either specify a speed value or use `auto` to enable autonegotiation.

```bash
SW1(config)# interface g0/1
SW1(config-if)# speed ?
  10    Force 10 Mbps operation
  100   Force 100 Mbps operation
  1000  Force 1000 Mbps operation
  auto  Enable AUTO speed configuration    
SW1(config-if)# speed 1000
SW1(config-if)# do show running-config interface g0/1
. . .
interface GigabitEthernet0/1    
 description ## to R1 ##    
 speed 1000    
End
```

Use `show running-config interface` *interface-name* to view the active configuration for a specific interface.

In the following output, `speed auto` doesn't appear because it's the default setting. Many default settings are hidden to avoid clutter in a device's configuration files.

```bash
SW1(config)# interface range f0/1-2
SW1(config-if)# speed auto
SW1(config-if)# do show running-config interface f0/1
. . .
interface FastEthernet0/1
 description ## end users ##
End
```

### Interface duplex

| Mode            | Behavior                                                           | Example                                         |
| :-------------- | :----------------------------------------------------------------- | :---------------------------------------------- |
| **Half duplex** | The interface can send and receive data, but not at the same time. | IEEE 802.11 wireless LAN (Wi-Fi)                |
| **Full duplex** | The interface can send and receive data at the same time.          | Wired Ethernet LAN using a switch (or switches) |

The opposite of duplex is **simplex**, which is one-way communication.

#### Ethernet hubs

- Hubs aren't Layer 2 aware. When a hub receives bits on one port, it repeats those bits out of all other ports.
- Hubs are considered **Layer 1 devices**. They receive and repeat electrical signals but don't examine those signals to make forwarding decisions.

#### Collision domains

A **collision** occurs when two messages are sent simultaneously over a shared medium and collide, resulting in an incoherent signal. A **collision domain** is a network segment in which simultaneous transmissions result in collisions.

Collisions shouldn't occur in a switched LAN:

- Switches store frames in memory and forward (or flood) them one after the other, avoiding collisions.
- Connections to a switch are **Ethernet point-to-point connections**: connections between only two devices, the switch and its connected device.
- Devices connected to a switch don't share the medium, so they don't have to wait their turn to transmit.

#### Carrier-sense multiple access with collision detection

Devices on a shared medium (a LAN using a hub) communicate using **carrier-sense multiple access with collision detection (CSMA/CD)**.

| Term                | Meaning                                                                                                                        |
| :------------------ | :----------------------------------------------------------------------------------------------------------------------------- |
| Carrier-sense       | Devices try to sense whether the medium is in use (by "listening" for electrical signals) before transmitting.                 |
| Multiple access     | Multiple devices access a shared medium.                                                                                       |
| Collision detection | If a collision occurs, devices connected to the medium detect it (and send a signal to notify other devices of the collision). |

Interfaces operating in half-duplex mode must use CSMA/CD:

1. Before sending a frame, devices wait until they detect that other devices aren't sending.
2. When a collision occurs, devices that detect it send a **jamming signal** to inform the other devices.
3. Each device waits a random period of time before sending frames again.
4. The process repeats.

#### Configure interface duplex

To configure an interface's duplex, use `duplex` {`auto` | `full` | `half`}. In modern wired networks, you can expect all devices to operate in full duplex.

1. Confirm the current status:

   ```bash
   SW1# show interfaces status
   Port      Name             Status      Vlan  Duplex  Speed Type
   Fa0/1     ## end users ##  connected   1     a-full  a-100 10/100BaseTX
   Fa0/2     ## end users ##  connected   1     a-full  a-100 10/100BaseTX
   Fa0/3     ## not in use ## notconnect  1     auto    auto 10/100BaseTX
   . . .
   Gi0/1     ## to R1 ##      connected   1     a-full   1000 10/100/1000BaseTX
   Gi0/2     ## not in use ## notconnect  1     auto    auto 10/100/1000BaseTX
   ```

2. Configure the duplex and verify:

   ```bash
   SW1# configure terminal
   SW1(config)# interface g0/1
   SW1(config-if)# duplex full
   SW1(config-if)# interface range f0/1-2
   SW1(config-if)# duplex auto
   SW1(config-if)# do show interfaces status
   Port      Name             Status     Vlan  Duplex  Speed Type
   Fa0/1     ## end users ##  connected  1     a-full  a-100 10/100BaseTX
   Fa0/2     ## end users ##  connected  1     a-full  a-100 10/100BaseTX
   Fa0/3     ## not in use ## notconnect 1     auto   auto 10/100BaseTX
   . . .
   Gi0/1     ## to R1 ##      connected  1     full   1000 10/100/1000BaseTX
   Gi0/2     ## not in use ## notconnect 1     auto   auto 10/100/1000BaseTX
   ```

### Autonegotiation

In the **autonegotiation** process, each device advertises its capabilities to its neighbor. The two devices agree on the best operational mode that both support.

| Connection                                                            | Recommendation                                                |
| :-------------------------------------------------------------------- | :------------------------------------------------------------ |
| Between network infrastructure devices (such as routers and switches) | Some engineers prefer to configure speed and duplex manually. |
| To end devices such as PCs                                            | Leave autonegotiation enabled.                                |

Greater speeds are prioritized over lesser speeds, and full duplex is prioritized over half duplex. For a 10/100/1000 interface:

| Priority | Operational mode        |
| :------- | :---------------------- |
| 1        | 1,000 Mbps, full duplex |
| 2        | 1,000 Mbps, half duplex |
| 3        | 100 Mbps, full duplex   |
| 4        | 100 Mbps, half duplex   |
| 5        | 10 Mbps, full duplex    |
| 6        | 10 Mbps, half duplex    |

When autonegotiation runs between a router and a switch, each device advertises its capabilities and chooses the best mode both devices share (for example, 100 Mbps, full duplex).

#### Autonegotiation on one end only

If speed and duplex are configured manually on one end of the connection and autonegotiation is enabled on the other end, the device with autonegotiation enabled behaves as follows:

| Setting | Behavior                                                                                                                                            |
| :------ | :-------------------------------------------------------------------------------------------------------------------------------------------------- |
| Speed   | Tries to sense the speed of the other device. If that fails, it uses the slowest supported speed (for example, 10 Mbps on a 10/100/1000 interface). |
| Duplex  | Uses half duplex if the speed is 10 or 100 Mbps. Uses full duplex if the speed is 1,000 Mbps or greater.                                            |

In the following example, R1 G0/1 connects to SW1 F0/1. R1 uses autonegotiation, and SW1 is configured manually:

```bash
R1(config)# interface g0/1
R1(config-if)# speed auto
R1(config-if)# duplex auto
```

```bash
SW1(config)# interface f0/1
SW1(config-if)# speed 100
SW1(config-if)# duplex full
```

| Device   | Speed        | Duplex   |
| :------- | :----------- | :------- |
| R1 G0/1  | 100 (sensed) | **Half** |
| SW1 F0/1 | 100          | **Full** |

R1 senses SW1's speed of 100 Mbps. Because the speed is 100 Mbps, R1 uses half duplex, but SW1 uses full duplex. The result is a duplex mismatch.

For how speed and duplex mismatches affect the interfaces, see [Speed mismatches](#speed-mismatches) and [Duplex mismatches](#duplex-mismatches).

### Interface errors

When a device encounters errors while sending or receiving messages on an interface, it increments the relevant counters. The following `show interfaces f0/1` output shows these counters:

```bash
SW1# show interfaces f0/1
. . .
     164850273 packets input, 138587749740 bytes, 0 no buffer
     Received 606 broadcasts (0 multicasts)
     0 runts, 0 giants, 0 throttles
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     0 watchdog, 0 multicast, 0 pause input
     0 input packets with dribble condition detected
     165209751 packets output, 180164587250 bytes, 0 underruns
     0 output errors, 0 collisions, 0 interface resets
     0 unknown protocol drops
     0 babbles, 0 late collision, 0 deferred
     0 lost carrier, 0 no carrier, 0 pause output
     0 output buffer failures, 0 output buffers swapped out
```

#### Received frame counters

| Counter      | Counts                                                                                     | Cause                                                                                   |
| :----------- | :----------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------- |
| Runts        | Frames received that are smaller than the minimum frame size of 64 bytes.                   | Can be caused by collisions.                                                            |
| Giants       | Frames received with a payload greater than the interface's maximum transmission unit (MTU), typically 1,500 bytes. | Usually a sign of misconfiguration: devices in the network aren't using consistent MTU values. |
| Input errors | All errors for received frames.                                                            |                                                                                         |
| CRC          | Frames that failed the Frame Check Sequence (FCS) check in the Ethernet trailer. CRC stands for Cyclic Redundancy Check. | Could be the result of electromagnetic interference (EMI) causing data corruption.      |

- A **runt** is a frame received that's smaller than the **minimum frame size**, which is 64 bytes.
- When a device wants to send a frame smaller than 64 bytes, it's supposed to add **padding** (bytes of all 0s) to the end of the message to make it 64 bytes.

#### Transmitted frame counters

| Counter        | Counts                                                                   | What it means                                                                                                                     |
| :------------- | :----------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------- |
| Output errors  | All errors for transmitted frames.                                       |                                                                                                                                   |
| Collisions     | All collisions that happen while the device transmits a frame.           | Expected if the device connects to a hub. In a switched LAN, this counter should remain at 0.                                     |
| Late collision | Collisions that happen after the 64th byte of the frame is transmitted.  | Often indicates that one device isn't using CSMA/CD to check the medium before transmitting (probably due to a duplex mismatch). |

The late collision counter is significant because of CSMA/CD timing: collisions should occur only within the first 64 bytes of a frame's transmission.

#### Speed mismatches

A speed mismatch puts both interfaces in a **down/down** state. Down/down refers to the Status and Protocol columns in the `show ip interface brief` output:

```bash
R1# show ip interface brief
Interface           IP-Address   OK? Method Status   Protocol
GigabitEthernet0/1  192.168.1.1  YES NVRAM  down     down
. . .     
```

```bash
SW1# show ip interface brief
Interface           IP-Address   OK? Method Status    Protocol
FastEthernet0/1     Unassigned   YES NVRAM  down      down
. . .
```

{{< admonition "Remember" tip >}}
The result of a speed mismatch is that both interfaces are in a down/down state.
{{< /admonition >}}

#### Duplex mismatches

Even with a duplex mismatch, both interfaces are operational. They're in an **up/up** state and can forward network traffic.

A duplex mismatch can occur when:

- Each end of the connection is configured with a different duplex setting (`duplex full` and `duplex half`).
- One end uses autonegotiation and the other doesn't.

With a duplex mismatch:

- The half-duplex device must wait for the other device to stop transmitting before it can transmit any data.
- If the full-duplex device transmits a frame while the half-duplex device is also transmitting, the half-duplex device interprets it as a collision.
- When the half-duplex device thinks it detected a collision, it sends a jamming signal (as part of the CSMA/CD process). The jamming signal destroys any frames that either device is currently sending.
