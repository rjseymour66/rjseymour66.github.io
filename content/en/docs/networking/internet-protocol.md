---
title: "Internet Protocol"
weight: 50
---

## TCP/IP

Created by the DoD to ensure data integrity and maintain communications in the event of catastropic war.
- First request for comments (RFC) to the Internet Engineering Task Force (IETF) was in 1974
- 1983, TCP/IP replaced Network Control Protocol (NCP) as the official data transport for ARPAnet (Advanced research Projects Agency), which was the DoD's precursor to the internet
- Most development work on TCP/IP happened at UC Berkeley
- BSD distro packaged TCP/IP and it took off

### DoD model

Condensed version of OSI model:
- Process/Application layer (OSI levels 7-5): Defines protocols for node-to-node application communication and controls UI specs
- Host-to-host layer (OSI level 4): Defines protocols for setting up the level of transmission service for applications:
  - e2e connections
  - error-free data delivery
  - packet sequencing
  - data integrity
- Internet layer (OSI level 3): Designates protocols for logical transmission of packets over the network:
  - Logical host addresses (IP assignment)
  - routes packets among multiple networks
- Network Access layer (OSI levels 2-1): Monitors the data exchange between host and network:
  - Hardware addressing
  - defines protocols for physical transmission of data

## Protocols per layer

| Layer | Protocols |
|---|---|
| Process/Application | Telnet, FTP LPD, SNMP, TFTP, SMTP, NFS, X Windows |
| Host-to-Host | TCP, UDP |
| Internet | ICMP, ARP, RARP, IP |
| Network Access | Ethernet, Fast Ethernet, Gigabit Ethernet, Wireless 802.11 |

## Process/Application

### FTP (TCP 20,21)

File Transfer Protocol lets you transfer files across an IP network. Also a program:
- limited to listing and maniuplating directories, typing file contents, and copying files between hosts
  - all data sent as plain text, unencrypted (use SFTP)
- as a protocol, used by apps
- as a program, perform file transfer tasks manually
- requires that you authenticate

### Secure Shell (TCP 22)

SSH sets up a secure Telnet session over TCP/IP connection:
- Logging into remote systems
- running programs on remote systems
- Moving files from one system to another

### Secure File Transfer Protocol (TCP 22)

SFTP transfers files over an encrypted connection with SSH
- Same as FTP, just secure. Transfers files between computers on an IP network

### Telnet

Uses terminal emulation to allow a user on a remote client machine (Telnet client) to access resources of another machine (Telnet server)
- Makes the server think that the client machine is using a termnal attached to the local network
- Plain text only, no security. Replaced by SSH
  
### Simple Mail Transfer Protocol (TCP 25)

SMTP delivers email with a spooled, or queued, method of delivery:
- After a message is sent to a destination, the message is _spooled_ to a device
- server software regularly hecks the queue for messages
  - When sees a message, it delivers it to the destination
- SMTP sends mail
- POP3 receives mail

### Domain Name System (TCP and UDP 53)

