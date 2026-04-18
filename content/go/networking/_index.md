+++
title = 'Networking'
date = '2025-08-20T16:15:30-04:00'
weight = 90
draft = false
+++

Computer networks communicate with each other with protocols. The OSI model has 7 layers, the TCP/IP model has 4. This table summarizes the layers and the protocols of the TCP/IP model:

| Layer       | Function                                                                    | Example Protocols    |
| ----------- | --------------------------------------------------------------------------- | -------------------- |
| Application | User services & data formats, how apps communicate with each other          | HTTP, FTP, DNS, SMTP |
| Transport   | Host-to-host communication, how datagrams are received (socket programming) | TCP, UDP             |
| Internet    | Routing and addressing, how bits and bytes are organized into datagrams     | IP, ICMP, IGMP       |
| Link        | Physical network access                                                     | Ethernet, Wi-Fi, ARP |


## Terminology

backpressure
: When an upstream component has to reduce its data transmission rate when a downstream component is overwhelmed. For example, a TCP network logger has to reduce its writes over the network because the log server's buffer is full and cannot process all logging requests in time.

throughput
: The actual rate that data successfully transmits over a network from sender to receiver over a period of time.