+++
title = 'Cables Connectors Ports'
date = '2026-11-11T08:53:17-05:00'
weight = 20
draft = false
+++

## Network standards

Ethernet isn't one single standard. It's a family of standards that define the physical aspects of wired network connections and the rules for communicating over them.

- **IEEE:** The Institute of Electrical and Electronics Engineers (IEEE, pronounced I-triple-E) is the governing body that defines these standards. In 1983, the IEEE first defined IEEE 802.3, better known as Ethernet.
- **IEEE 802.11:** Also defined by the IEEE. Better known, though not officially, as Wi-Fi.
- **Working groups:** The IEEE assigns working groups to develop specific technologies. The two groups most relevant to the CCNA are 802.3 (wired Ethernet) and 802.11 (wireless LANs, or Wi-Fi).

## Binary bits and bytes

Two computers connected by a cable communicate by sending each other long series of bits (0s and 1s), often at a rate of billions of bits per second. How the 0s and 1s are conveyed depends on the medium. For example, copper wiring conveys them by modifying the voltage of an electric signal between the two devices. Network connection speed is measured by how many bits transmit per second over the connection.

| Unit           | Bits                                     |
| :------------- | :---------------------------------------- |
| 1 kilobit (kb) | 1,000 bits                              |
| 1 megabit (Mb) | 1,000,000 bits (1,000 kilobits)         |
| 1 gigabit (Gb) | 1,000,000,000 bits (1,000 megabits)     |
| 1 terabit (Tb) | 1,000,000,000,000 bits (1,000 gigabits) |

## Copper UTP connections

The most common connection type in Ethernet LANs uses copper unshielded twisted pair (UTP) cables.

- **Unshielded:** The wires don't have a metallic shield around them. A shield can reduce electromagnetic interference (EMI), but UTP cables don't have one.
- **Twisted pair:** The eight wires in the cable are twisted together into four pairs of two wires each. Twisting the wires reduces EMI between the wires in each pair.
- **Connector:** The 8 position 8 contact (8P8C) connector has eight pins, one for each of the eight wires inside the cable. It's also called an RJ45 connector (RJ stands for Registered Jack).
- **Maximum length:** The 100-meter maximum cable length is a major limitation that can make copper connections unfeasible in some cases.

### IEEE 802.3 standards (copper)

| Speed    | Speed-derived name | IEEE task group | Informal name | Maximum cable length |
| :------- | :------------------ | :--------------- | :------------- | :-------------------- |
| 10 Mbps  | Ethernet           | IEEE 802.3i     | 10BASE-T      | 100 m                |
| 100 Mbps | Fast Ethernet      | IEEE 802.3u     | 100BASE-T     | 100 m                |
| 1 Gbps   | Gigabit Ethernet   | IEEE 802.3ab    | 1000BASE-T    | 100 m                |
| 10 Gbps  | 10 Gig Ethernet    | IEEE 802.3an    | 10GBASE-T     | 100 m                |

- **Cabling standards:** Cable standards are defined by the Electronic Industries Alliance (EIA) and the Telecommunications Industry Association (TIA).
- **Maximum length:** Using a UTP cable longer than the listed maximum can cause signal attenuation and decreased performance.

### Straight-through and crossover cables

- **Straight-through cable:** Connects the same pin pair on one connector to the same pin pair on the other connector. Works well when connecting a PC to a switch.
  - PCs transmit (Tx) on the 1-2 pin pair and receive (Rx) on the 3-6 pin pair.
  - Switches transmit (Tx) on the 3-6 pin pair and receive (Rx) on the 1-2 pin pair.
- **Pair count by standard:** 10BASE-T and 100BASE-T use two pairs (four wires). 1000BASE-T and 10GBASE-T use four pairs (eight wires).
- **Cable choice:** Using the correct cable matters most for 10BASE-T and 100BASE-T connections, to connect each pin to the correct pin on the other end.

### Crossover cables

A crossover cable connects opposite pin pairs: pins 1 and 2 on one end connect to pins 3 and 6 on the other end. This lets devices that transmit on the same pin pair communicate with each other.

| Device type | Transmit (Tx) pins | Receive (Rx) pins |
| :----------- | :------------------ | :----------------- |
| Router      | 1 and 2            | 3 and 6           |
| Firewall    | 1 and 2            | 3 and 6           |
| PC/Server   | 1 and 2            | 3 and 6           |
| Switch      | 3 and 6            | 1 and 2           |

### Auto MDI-X

- **Auto MDI-X:** Short for Auto Medium-Dependent Interface Crossover. Lets a device automatically change which pins it uses to transmit and receive data, based on the device it's connected to.

### 1000BASE-T and 10GBASE-T

- Use all eight wires in the cable, as four wire pairs.
- Use the same 1-2 and 3-6 pin/wire pairs as 10BASE-T and 100BASE-T, plus two more: the pair at positions 4 and 5, and the pair at positions 7 and 8.
- Each wire pair can transmit and receive data simultaneously.

## Fiber-optic connections

Fiber-optic cables transmit light signals along a glass fiber instead of electrical signals along a copper wire.

- **Handling:** Handle fiber-optic cables with care. A sharp bend in the cable can damage the glass fiber and render the cable unusable.
- **Cost:** Fiber-optic connections are more expensive than copper UTP connections, largely because of the cost of SFP transceivers.

### Anatomy of a fiber-optic cable

A typical fiber-optic connection uses two cables, one for transmitting data and one for receiving data. These cables connect to a Small Form-Factor Pluggable (SFP) transceiver, which is inserted into an SFP port on the device.

There are two main types of fiber-optic cabling: multimode fiber (MMF) and single-mode fiber (SMF).

| Attribute | Multimode fiber (MMF) | Single-mode fiber (SMF) |
| :--- | :--- | :--- |
| Core width | Wider | Narrower |
| Light source | LED | Laser (more expensive than LED) |
| Light path | Multiple angles (modes), reflecting off the cladding | Single angle |
| Maximum distance | Several hundred meters | Tens of kilometers |

## UTP vs fiber

- **UTP:** Most common for connections from switches to end hosts. In an office setting, switches are typically on each floor, and the 100-meter maximum cable length is usually enough for an end host to reach a switch on its floor.
- **Fiber-optic:** More common for connections between network infrastructure, for example, connecting switches and routers located on separate floors or in separate buildings.
- **EMI vulnerability:** UTP cables are vulnerable to EMI and can emit (leak) their signal outside of the cable.
- **Client devices:** Most client devices, such as PCs, don't have SFP ports for fiber-optic connections, so a UTP connection is the only choice.