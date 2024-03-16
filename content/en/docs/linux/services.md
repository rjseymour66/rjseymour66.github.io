---
title: "Services overview"
weight: 20
description: >
  Brief overview of Linux services.
---

A _service_ is a shared resource. The Linux server runs services to multiple users (clients) in a network environment.

## Launching services

Services don't magically run--you have to know how to launch and maintain services for client interactions. Services can run in one of the following ways:
- Background process (daemon). These services usually have a 'd' at the end of their name.
- Process spawned by a parent program that listens for requests.

### Super-servers

_Super-servers_ are servers that listen for network connections for several applications. There are two super-servers since Linux began:
- `inetd` (internet daemon) runs as a daemon and waits for requests.
  
  Configure it in the `/etc/inetd.conf` file, where you can define the services that it handles requests for.
- `xinetd` (extended internet daemon) is the next generation of inetd with advanced features like ACLs, advanced logging, and the ability to schedule services at specific times.

### Listening for clients

Each service uses a separate network protocol to communicate with clients. These protocols are standardized by the IETF and published as RFCs.

The protocol defines how clients connect to the service. TCP and UDP standards use ports to help separate network traffic that is headed to the same IP address. Clients use a common IP address to reach a server then different ports to reach individual services.

The `/etc/services` file contains all the ports defined on the Linux server.

[List of TCP and UDP port numbers](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers)

## Internet services

### Web servers

#### Apache server

- Developed by the National Center for Supercomputing Applications (NCSA)
- Most popular server on the internet because of modularity
- Each advanced feature is a plug-in module

#### nginX server

- Meant to improve on Apache. Has additional features:
  - web proxy
  - mail proxy
  - web page cache
  - load balancer
- Better for high-volume environments (10,000+ network connections) than Apache because it has a smaller footprint
- Great for dynamic web apps

#### lighthttpd package

- Lightweight server for client requests
- Great for embedded systems because low memory and CPU usage

### Database servers

First big milestone was the relational database that allowed multiple clients to access the same data from a centralized location with structured query language (SQL).

#### PostgreSQL

- Open source, began as a university project
- Complete object-relational database
- Known for advanced features

#### MySQL

- Started as a project to create a simple but fast database system--basic features that preformed quickly
- Speed made it the defacto db for web applications
- Part of the famous LAMP stack (Linux, Apache, MySQL, Php)

#### MongoDB

- Released in 2009 as a NoSQL-compliant db system
- Object-oriented database
- Uses documents instead of tables
- Documents can contain different, unique, and independent data elements
- Stores data records in JSON

### Mail servers

Linux uses multiple small programs to process email messages:
- **Mail transfer agent (MTA)**. Handles incoming and outgoing email on the server. Connects to other MTA agents if the email leaves the network
  - sendmail: Very complex, but many very important features like virtual domains, message forwarding, user aliases, mail lists, and host masquerading. Complex config file.
  - postfix: Simple. Multiple modular programs to implement MTA. Uses 2 small, plaintext config files.
  - exim: One large program, attempts to avoid message queueing.
- **Mail delivery agent (MDA)**. Delivers email messages from the MTA to local users or locations defined by the user.
  - binmail: Most popular and simple, located in `/bin/mail`. Reads messages stored in `/var/spool/mail` or points to alternative mailbox.
  - procmail: Popular because of versatility, and sometimes installed by default. User can create a `.procmail` file in `$HOME` to direct incoming mail.
- **Mail user agent (MUA)**. Client-side apps that interact with users so that they can view and manipulate email messages

## Serving local networks

Linux is often used to provide the following simple network services.

### File servers

Multiple people can create and edit files in a common folder. There are two basic methods for file sharing:
- **peer-to-peer**: one workstation enables another workstation to access files stored locally on its hard drive. Good when you do not need to share between more than 2 people.
- **client-server**: files are stored in a centralized file server for sharing files that multiple clients can access and modify as needed. The most common are **NFS** and **Samba**.

#### NFS

Network file system, shares a portion of its virtual directory in a network environment.

Linux package is `nfs-utils`:
- drivers to support NFS
- underlying client and server software to both share local folders on the network and connect to remote folders shared by other linux system on the local network

#### Samba

Lets you share files on a network with Windows servers. Windows' default file sharing method is System Message Block (SMB) protocol, which was released as a network standard so you can create open-source software with it.

Linux can act as either a client and connect to a Windows server to get shared folders, or as a server that allows Windows machines access to files on a Linux server.

### Print servers

Network printer standard is Common Unix Printing System (CUPS). You connect using a common API that operates over dedicated printer drivers. CUPS uses the Internet Printing Protocol (IPP).

You can also share a locally connected printer to other linux systems.

### Network resource servers

A network needs to run different resources to keep clients and servers in sync.

#### IP addresses

The Dynamic Host Configuration Protocol (DHCP) assigns and tracks IP addresses for computers on the network. Each computer on a network must have a unique IP address so it can communicate with other resources on the network.

Most common package is DHCPd. There are three popular versions of it:
- `dhclient`, often used by Red Hat
- `dhcpcd`
- `pump`

#### Logging

Log files are normally stored in `/var/log`, but it might be helpful for Linux servers to store their system logs on a remote logging server. This provides a backup of the original log files and a storage location that is safe from crashes or hackers.

Two main remote logging services that accept logging data from remote servers. Both services use config files to setup what is logged and which clients can connect:
- `rsyslogd`: SysVinit and Upstart systems use this
- `journald`: Systemd systems use this

#### Name servers

DNS maps IP addresses to a host naming scheme. Linux uses the BIND software package for DNS, which uses `named`, a server daemon that runs on Linux servers to resolve hostnames to IP addresses for clients on a local network. One BIND server can communicate with another on remote networks so you can resolve any IP address on the internet.

BIND should support the DNSSEC security protocol to prevent attacks like hostname spoofing.

#### Network management

Monitors which devices are active or which servers are running at capacity. Most popular is `net-snmp`.

Simple Network Management Protocol (SNMP) lets admins query remote network devices and servers to get the following info:
- configuration
- status
- performance

Network devices run the SNMP service and listen to requests from SNMP clients. There are 3 versions of SNMP, with each improving security. SNMPv3 uses strong password authentication and data encryption.

#### Time

Network Time Protocol (NTP) syncs internal clocks on network clients and servers. `ntpd` syncs Linux systems with NTP servers on the internet. Usually, one server syncs to remote time standard server, and then the other servers in the network sync to that server.