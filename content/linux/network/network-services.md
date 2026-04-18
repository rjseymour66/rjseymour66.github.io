+++
title = 'Network Services'
date = '2025-09-07T19:03:14-04:00'
weight = 20
draft = false
+++


## Network addressing

In the network `192.168.1.0/24`, two addresses have reserved purposes and cannot be assigned to hosts:

| Address         | Purpose                                                       |
| :-------------- | :------------------------------------------------------------ |
| `192.168.1.0`   | Network address. Identifies the network itself.               |
| `192.168.1.255` | Broadcast address. Sends packets to all hosts on the network. |

A **DHCP reservation** (also called a static lease) assigns the same address to a specific device each time it connects. Use reservations for devices that need a predictable address, such as servers or admin workstations.

## DHCP server

The DHCP server assigns IP addresses to clients on your network. The server itself must
have a static IP address. The default configuration file is `/etc/dhcp/dhcpd.conf`.
Lease assignments are recorded in `/var/lib/dhcp/dhcpd.leases` and logged to `/var/log/syslog`.

### Install DHCP

Install and configure the DHCP server when you want to manage address assignment on your
local network directly. This is useful in a home lab or small office where you need control
over lease times, DNS settings, or address reservations.

1. Install the package.
   ```bash
   apt install isc-dhcp-server
   ```
2. Stop the service during initial configuration.
   ```bash
   systemctl stop isc-dhcp-server
   ```
3. Disable the IPv6 service if you are not using it.
   ```bash
   systemctl stop isc-dhcp-server6
   systemctl disable isc-dhcp-server6
   ```
4. Back up the default configuration file.
   ```bash
   mv /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.orig
   ```
5. Create the configuration file.
   ```bash
   vim /etc/dhcp/dhcpd.conf
   ```
6. Assign the network interface in `/etc/default/isc-dhcp-server`. Set `INTERFACESv4`
   to the name of your network interface:
   ```bash
   INTERFACESv4="enp0s3"
   ```
7. Start the service and verify it is running.
   ```bash
   systemctl start isc-dhcp-server
   systemctl status isc-dhcp-server
   ```

To view logs and check lease assignments:

```bash
tail -f /var/log/syslog             # view logs in real time
cat /var/lib/dhcp/dhcpd.leases      # check lease assignments
```

### Configuration file

#### /etc/dhcp/dhcpd.conf

Configure lease times, network options, and the client address range. Set `authoritative`
if this is the only DHCP server on the network:

```bash
default-lease-time 43200;                   # clients must renew their lease every 43200 seconds (12 hours)
max-lease-time 86400;                       # maximum lease duration in seconds (1 day)
option subnet-mask 255.255.255.0;           # subnet mask sent to clients
option broadcast-address 192.168.1.255;     # broadcast address sent to clients
option domain-name "local.lan";             # domain name assigned to clients (<hostname>.local.lan)
authoritative;                              # declares this server as authoritative for this network
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.1 192.168.1.240;        # address range available to clients
    option routers 192.168.1.1;             # default gateway sent to clients
    option domain-name-servers 192.168.1.1; # DNS server address sent to clients
}
```

## DNS server

A DNS server resolves domain names to IP addresses. Organizations often run a local DNS
server to resolve hostnames within their network. If the local server cannot resolve a
name, it forwards the request to an external DNS server.

Linux DNS servers use the Berkeley Internet Name Domain (BIND) package. The main
configuration file is `/etc/bind/named.conf`, which links to additional configuration
files. The most basic use of BIND is as a **caching name server**: it forwards queries
to an external DNS server and caches the responses. Subsequent lookups for the same
name are served from the cache, reducing latency.

Install the package and verify the current DNS server address:

```bash
apt install bind9
resolvectl              # shows the IP address of the active DNS server
```

### Caching name server

A caching name server forwards DNS queries to an external server and caches the responses.
Repeated lookups are served from the cache without contacting the external server.

1. Edit the BIND options file.
   ```bash
   vim /etc/bind/named.conf.options
   ```
   Add your forwarding DNS servers inside the `forwarders` block:
   ```bash
   options {
       ...
       forwarders {
           8.8.8.8;
           8.8.4.4;
       };
       ...
   };
   ```
2. Restart the service and verify it is running.
   ```bash
   systemctl restart bind9
   systemctl status bind9
   ```
3. Update the DHCP configuration so new clients receive the local DNS server address.
   Edit `/etc/dhcp/dhcpd.conf` and set `domain-name-servers` in the subnet block:
   ```bash
   subnet 192.168.1.0 netmask 255.255.255.0 {
       ...
       option domain-name-servers <dns-addr>;
   }
   ```
4. Install the DNS test utilities.
   ```bash
   apt install dnsutils
   ```
5. Verify caching is working by running the same query twice. The second query time
   should drop to 0 milliseconds:
   ```bash
   dig www.mlb.com
   ```
   First query:
   ```bash
   ;; Query time: 15 msec
   ```
   Second query:
   ```bash
   ;; Query time: 0 msec
   ```

### Internal DNS

An internal DNS server resolves hostnames within your local network. Configure it by
defining a zone in `/etc/bind/named.conf.local` and creating a zone file that lists your
hosts and their IP addresses:

1. Edit the local zone configuration file.
   ```bash
   vim /etc/bind/named.conf.local
   ```
   Define the zone name and the path to its zone file:
   ```bash
   zone "local.lan" IN {
       type master;
       file "/etc/bind/net.local.lan";
   };
   ```
2. Create the zone file.
   ```bash
   vim /etc/bind/net.local.lan
   ```
3. Restart the service and check for errors.
   ```bash
   systemctl restart bind9
   systemctl status bind9
   ```

#### /etc/bind/net.local.lan

The zone file defines DNS records for your local network. `@` represents the zone name
(`local.lan`). Increment the serial number after each change so BIND detects updates.
The recommended serial format is `YYYYMMDDNN`:

```bash
$TTL 1D                                             # record TTL: how long clients cache responses
@ IN SOA local.lan. hostmaster.local.lan. (         # Start of Authority: declares this server authoritative for local.lan
                                                    # hostmaster.local.lan is the administrator contact address
    202401131;  serial                              # increment after each change (format: YYYYMMDDNN)
    8H ;        refresh                             # secondary servers check for updates every 8 hours
    4H ;        retry                               # secondary servers retry after 4 hours on error
    4W ;        expire                              # zone expires after 4 weeks if the primary is unreachable
    1D )        minimum                             # minimum TTL for negative responses
IN A 192.168.1.1                                    # name server IP address
;
@ IN NS hermes.local.lan.                           # name server hostname
fileserv    IN  A       192.168.1.3                 # local host records
hermes      IN  A       192.168.1.1                 # name server host record (required or resolution may fail)
mailserv    IN  A       192.168.1.5
mail        IN  CNAME   mailserv.                   # CNAME: alias pointing to mailserv
web01       IN  A       192.168.1.7
```

### Internet gateway setup

A gateway routes traffic between your local network and an external network such as the
internet. Administrators often run DNS, DHCP, and routing on the same server. A gateway
server requires two network interfaces: one connected to the ISP device using DHCP, and
one connected to the internal network switch.

Enable IP forwarding temporarily until the next reboot:

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```

To make IP forwarding persist across reboots, uncomment the following line in
`/etc/sysctl.conf`:

```bash
net.ipv4.ip_forward=1
```

## OpenSSH

OpenSSH lets you open a command shell on a remote server. It has two components: the
server daemon (`sshd`) and the client (`ssh`). Most distributions install OpenSSH by
default and configure it to start automatically. It listens on port 22.

You have the same permissions as the user you connect as. When you end a session, all
background processes started during that session are terminated.

Manage the service with these commands:

```bash
apt install openssh-client          # install the client
apt install openssh-server          # install the server
systemctl status ssh                # verify sshd is running
systemctl start ssh                 # start the service
systemctl enable ssh                # enable the service to start at boot
ss -tunlp | grep ssh                # verify sshd is listening for connections
```

Connect to a remote server:

```bash
ssh 10.20.30.40                     # connect as the current user on port 22
ssh <username>@<addr>               # connect as <username> on port 22
ssh -p 2242 <username>@<addr>       # connect on a non-default port
exit                                # end the SSH session
```

### SSH keys

[Ubuntu docs](https://help.ubuntu.com/community/SSH/OpenSSH/Keys)

By default, SSH authenticates with a password. Public key authentication is more secure
because your password is never transmitted. You generate a mathematically linked key
pair: a private key you keep on your local machine and a public key you copy to remote
servers. When you connect, SSH verifies your identity by matching your private key
against the public key on the server.

A passphrase on your private key is optional but recommended. Use `ssh-agent` to cache
the passphrase so you only enter it once per session.

Key file permissions are set automatically: the public key is readable by all, and the
private key is readable only by its owner.

#### Generate a key pair and copy to a remote server

Generate a key pair with `ssh-keygen`, then copy the public key to each remote server
you want to access:

```bash
ssh-keygen                                          # generate an ed25519 key pair (default)
ssh-keygen -t rsa -b 4096                           # generate an RSA key pair
ssh-copy-id -i ~/.ssh/id_rsa.pub <user>@<ip-addr>   # copy the public key to a remote server
```

#### Add a key to ssh-agent

`ssh-agent` caches your passphrase so you only enter it once per session:

```bash
eval $(ssh-agent)                                   # start ssh-agent if not already running
ssh-add ~/.ssh/id_rsa                               # add the key (prompts for passphrase)
```

#### Change the passphrase on an existing key

Run `ssh-keygen -p` to update the passphrase on a key you have already generated:

```bash
ssh-keygen -p
```

### SSH config file

The SSH client reads `~/.ssh/config` on each connection. The file is not created by
default. Each `Host` entry defines a shorthand alias with a hostname, user, and port.
Use an IP address for `HostName` if the server is not listed in `/etc/hosts`.

See all available options:

```bash
man ssh_config
```

Each entry follows this format:

```bash
Host <alias>                                # shorthand name you use with the ssh command
    HostName <ip-address-or-hostname>       # IP address or hostname of the remote server
    User <username>                         # user to connect as
    Port <port>                             # SSH port (default is 22)
```

Example:

```bash
Host u24
    HostName 10.20.30.40
    User linuxuser1
    Port 22
```

### ssh-copy-id

`ssh-copy-id` copies your public key to a remote server's `authorized_keys` file. It
creates `~/.ssh/` and `authorized_keys` on the remote server if they do not exist.
Additional keys are appended to the file, one per line:

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub <user>@<ip-addr>
```
