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
- Mail transfer agent (MTA). Handles incoming and outgoing email on the server. Connects to other MTA agents if the email leaves the network
  - `sendmail`: Very complex, but many very important features like virtual domains, message forwarding, user aliases, mail lists, and host masquerading. Complex config file.
  - `postfix`: Simple. Multiple modular programs to implement MTA. Uses 2 small, plaintext config files.
  - `exim`: One large program, attempts to avoid message queueing.
- Mail delivery agent (MDA). Delivers email messages from the MTA to local users or locations defined by the user.
  - `binmail`: Most popular and simple, located in `/bin/mail`. Reads messages stored in `/var/spool/mail` or points to alternative mailbox.
  - `procmail`: Popular because of versatility, and sometimes installed by default. User can create a `.procmail` file in `$HOME` to direct incoming mail.
- Mail user agent (MUA). Client-side apps that interact with users so that they can view and manipulate email messages