---
title: "Ethernet"
weight: 30
---

If you are on the same LAN, you find other workstations with their MAC address.
- Can use the broadcast address to find hosts on same LAN, no DNS required
- `192.168.0.255` is a broadcast address
  - Send message to broadcast address looking for workstation by name. Message includes sender MAC addr
  - After name is resolved to an IP addr, broadcast on LAN to get MAC address

## Ethernet basics

Ethernet is a contention media-access method that lets all hosts on a network share the same bandwidth of a link.
- Uses both Data Link layer and Physical link layer information

### Collision domain

Ethernet scenario where one device sends a packet on the network segment, which forces every other device on that network to pay attention to it.
- When two devices transmit at the same time, a _collision event_ occurs and forces both devices to retransmit later.
- Collision event is when each device's digital signal interferes with another on the wire
- Typical in hub environment where the single hub results in a collision domain and a broadcast domain

### Broadcast domain

The set of all devices on a network segment that hear all the broadcasts sent on that segment.
- All hosts can reach each other on the Data Link layer

### CSMA/CD

Carrier Sense Multiple Access with Collision Detection
- Media access control contention method that helps devices share the bandwidth evenly withouth having two devices transmist at the same time on the network
- Can cause delay, low throughput, and congestion

Process:
1. Host checks for other digital signals on the wire
2. If clear, host sends the transmission
3. Host monitors wire for other messages 
4. If there is another signal on the wire, the host sends out a jam signal to tell other hosts to stop sending data
5. Other hosts use backoff algorithms/timer to determine when to try to send the data again
   - All hosts have equal priority to transmit after timer expires
6. If there are 15 failed attempts to send data, the transmission times out


### Broadband/Baseband

These are the two ways that you can send a signal down a wire:
- **Broadband**: This is what most people use in their homes. Allows both analog voice and digital data on the same network cable or physical media.
  - uses frequency-division multiplexing, which is when you send multiple frequencies of different signals down the same wire at the same time
- **Baseband**: What LANs use. All bandwidth of physical media is used by one signal.
  - When multiple signals are sent, we get collisions

### Bit rate vs baud rate

Bit rate
: Measure of the number of data bits transmitted in one second in either a digital or analog signal. 10,000 bits per second (bps) means that 10,000 1s or 0s are transmitted in a second.

Baud rate
: What we previously used to measure transmitted data bits, named after Jean-Maurice-Emile Baudot. Measures one electronic state change (i.e. volts or binary). A state change might result in more than a single bit, so bps is used.

### Wavelength

The distance between high points (peaks) in a wave pattern. When two wave patterns are different, they have different wavelenghts.

### Half- and full-duplex ethernet

Defined in original 802.3 spec:
- **half-duplex**: uses only one wire pair with a digital signal either transmitting or receiving
  - 10BaseT wire which provides 3 or 4 Mbps
- **full-duplex**: can transmit and receive at the same time
  - No collisions. Provides point-to-point connection between the transmitter of sending device and receiver of receiving device (receiver is usually a switch).
  - Requires a dedicated switch port
  - Host network card and switch port must be able to operate in full-duplex mode
  - Like a freeway with multiple lanes 
  - 100% efficiency in either direction. For example, 200 Mbps for fast ethernet

200 Mbps is called aggregate rate, which means you're supposed to get this, but you probably do not
- _auto-detect-mechanism_: Ethernet ports first check with the remote end and figures out how much data it can accept, and whether it is full- or -half duplex enabled
- Pretty rare to do this, but you can manually set both the speed and duplex type on the NIC with the Network Connection Properties for a network adapter

## Data link layer

At the data link layer, ethernet is responsible for Ethernet addressing, or _hardware addressing_ or _MAC addressing_. Also frames packets from the Network layer and prepares them for transmission across the network.

### Binary, decimal, hexadecimal

MAC addresses are hexadecimal.

#### Binary

Each binary number is placed in a value spot. Each spot has double the value of the previous spot, starting from right to left.

nibble
: 4 bits together: `0101`
  Values: `8 4 2 1`
  `1111` = 15

byte
: 8 bits together: `10100011`. Also called an _octet_ in networking.
  Values: `128 64 32 16 8 4 2 1`
  `11111111` = 255

##### Subnet mask chart

| Binary | Decimal |
|---|---|
| `10000000` | 128 |
| `11000000` | 192 |
| `11100000` | 225 |
| `11110000` | 240 |
| `11111000` | 248 |
| `11111100` | 252 |
| `11111110` | 254 |
| `11111111` | 255 |

#### Hexadecimal

Convert hexadecimal to decimal by reading nibbles, not bytes. Two hex characters make a byte:

| Hexadecmial | Binary | Decimal |
|---|---|---|
| `0` | `0000` | 0 |
| `1` | `0001` | 1 |
| `2` | `0010` | 2 |
| `3` | `0011` | 3 |
| `4` | `0100` | 4 |
| `5` | `0101` | 5 |
| `6` | `0110` | 6 |
| `7` | `0111` | 7 |
| `8` | `1000` | 8 |
| `9` | `1001` | 9 |
| `A` | `1010` | 10 |
| `B` | `1011` | 11 |
| `C` | `1100` | 12 |
| `D` | `1101` | 13 |
| `E` | `1110` | 14 |
| `F` | `1111` | 15 |


### Ethernet addressing

Every Ethernet NIC has a Media Access Control (MAC) address burned into it. ITs a 48-bit hex number, broken down here by byte:
- Organizationally Unique Identifier (OUI): 3 bytes (24 bits) assigned by the IEEE. The final two bits have special use cases:
  - Individual/Group (I/G) address bit (46th bit) signifies if destintation MAC address is unicast or multicast/broadcast layer 2 address. Set to 0 for unicast, 1 for multicast/broadcast.
  - Local/Global (L/G) address bit (47th bit) tell if the MAC address is burned-in-address (BIA) or a MAC address that has been changed locally.
- Vendor assigned: 3 bytes (24 bits), locally administered or manufacturer-assigned code. Manufacturers start with 24 0s and end with 24 1s. Many use the same 6 hex digits as the last six characters of their serial number on the card.

```bash
# sample MAC
   OUI      Vendor assigned   
/------\ /------\
63:5b:44:6e:3e:bb
```

### Ethernet frames

Frames encapsulate packets handed down from the network layer for transmission on a type of physical media access. Ethernet stations pass data frames using a group of bits called a MAC frame format, which contains the following fields:

Preamble (7 bytes)
: Alternating 1,0 pattern that provides a clock at the start of each packet which allows the receiving devices to lock the incoming bit stream

Start of Frame Delimiter (SOF)/Synch (1 byte)
: One octet `10101011` where the last pair of 1s allows the reeiver to come into the alternating 1,0 pattern and still sync up and detect the beginning of the data

Destination Address (DA) (6 bytes)
: Transmitted using least significant bit (LSB), used by receiving stations to determine if an incoming packed is addressed a host or a broadcast or multicast MAC address.
  - Broadcast is all 1s (`f` in hex) and sent to all devices.
  - Multicast is sent to only a similar subset of hosts on the network.

Source address (SA) (6 bytes)
: 48-bit MAC address that IDs the sender, uses LSB. Broadcast and multicast addresses are illegal.

Type (Ethernet frame) or Length (802.3) (2 bytes)
: Identifies the network protocol.

Data and pad (46 to 1,500 bytes)
: Sent down rom the network layer

Frame Check Sequence (FCS) (4 bytes)
: Stores the cyclic redundancy check (CRC) that provides error detection (not correction!)

## Physical layer

Ethernet cable that is specified by the Electroinc Industries Association and Telecommuncations INdustry Alliance (EIA/TIA) has _inherent attenuation_, which is the loss of signal strenght as it travels the length of a cable. This is measure in decibels (dB).
- Cables used in corporate and home markets is measured in categories, where a higher quality cable has higher rated category and lower attenuation. For example, category 5 is better than category 3.

[IEEE 802.3 cable standards and variants](https://en.wikipedia.org/wiki/Ethernet_over_twisted_pair#Variants)

## Ethernet and IEEE 1905.1-2013

This standard defines a convergent digital home network for both wireless and wireline technologies:
- 802.11 (WiFi)
- 1901 (HomePlug, HD-PLC)
- Powerline networking
- 802.3 Ethernet
- Multimedia over Coax (MoCA)

Basic idea is to define a simple setup, configuration, and operation of home networking devices using both wired and wireless tech.

### Ethernet over powerline

- IEEE 1901 is the standard for Broadband over Power Line (BPL), also referred to as Power Line Communication (PLC) or Power Line Digital Subscriber Line (PDSL)
- In the future, you should be able to plug a computer into a wall socket and have 500 Mbps for up to 1500 meters

### Ethernet over HDMI

HDMI Ethernet Channel tech consolidates video, audio, and data streams into a single HDMI cable.