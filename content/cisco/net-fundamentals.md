+++
title = 'Network fundamentals'
date = '2026-11-11T09:51:12-05:00'
weight = 10
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
