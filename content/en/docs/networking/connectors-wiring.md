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
