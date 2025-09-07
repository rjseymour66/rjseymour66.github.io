---
title: "Network services"
linkTitle: "xNetwork services"
weight: 100
# description:
---

## Network addressing

A few rules about network addressing on example network `192.168.1.0/24`:
- `192.168.1.0`: Refers to the network itself and cannot be assigned
- `192.168.1.255`: Broadcast address
- _DHCP reservations_ (or static leases) are assigned by DHCP server, but the same address is assigned each time. For devices that need a predictable address, such as an admin's desktop.

## DHCP server

- DHCP server requires a static IP address
- Default config at `/etc/dhcp/dhcpd.conf`
- In sample file, you can either have `authoritative` or `not authoritative`
- Lease info is recorded in `/var/lib/dhcp/dhcpd.leases`
- Records info in `/var/log/syslog`

```bash
apt install isc-dhcp-server                         # 1. Install server package
systemctl stop isc-dhcp-server                      # 2. Stop service - requires config first
systemctl stop isc-dhcp-server6                     # 3. Stop IPv6 equivalent
systemctl disable isc-dhcp-server6
mv /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.orig   # 4. Create backup so you can edit file
                                                    # 5. Create /etc/dhcp/dhcpd.conf file:
# --- Sample /etc/dhcp/dhcpd.conf file --- #

default-lease-time 43200;                   # Device must renew lease every 43200 secs (12 hrs)
max-lease-time 86400;                       # Max num of secs device can have a lease (1 day)
option subnet-mask 255.255.255.0;           # Telling clients their subnet mask should be set to this val
option broadcast-address 192.168.1.255;     # Telling the client this is the broadcast address
option domain-name "local.lan";             # Set domain name for all connected clients (<hostname>.local.lan)
authoritative;                              # This DHCP server is authoritative to this network (required)
subnet 192.168.1.0 netmask 255.255.255.0 {  # Declaring addresses for DHCP network
	range 192.168.1.1 192.168.1.240;        # Available to clients
	option routers 192.168.1.1;             # Default gateway
	option domain-name-servers 192.168.1.1; # DNS router
}

INTERFACESv4="enp0s3"                               # 6. Assign interface in /etc/default/isc-dhcp-server
systemctl start isc-dhcp-server                     # 7. Start service
systemctl status isc-dhcp-server                    # 8. Check status
tail -f /var/log/syslog                             # 9. View logs in real time
cat /var/lib/dhcp/dhcpd.leases                      # 10. Check lease assignments
```

## DNS server

Domain Name Server - matches an IP address to a domain or hostname:
- Some orgs have local DNS servers so you can create a local domain
  - Create a _zone file_ that contains hosts and IPs in local network so local hosts can resolve them
  - If local DNS can't resolve, send to external DNS
- Requires Berkeley INternet Name Daemon (BIND) package
  - Config file is `/etc/bind/named.conf`, contains a links to other config files 
  - Most basic function is **Caching Name Server** - doesn't resolve names, it caches responses from external server
  - Ex: If you look up mlb.com, local DNS will forward the req to external DNS the first time, but second time the local DNS server will have it cached.
  - `/etc/bind/named.conf.options` is config file for caching name server

```bash
apt install bind9
resolvectl              # IP address of DNS server
```

### DNS Caching Name server

```bash
vim /etc/bind/named.conf.options        # 1. Edit config file
options {
        ...
        forwarders {
          8.8.8.8;                      # 2. Add DNS servers (these are Google DNS servers)
          8.8.4.4;
        };
        ...
};
systemctl restart bind9                 # 3. Restart service
systemctl status bind9                  # 4. Verify status

vim /etc/dhcp/dhcpd.conf file           # 5. Add DNS server to DHCP config file - new clients get 
...
subnet 192.168.1.0 netmask 255.255.255.0 {  
	...
	option domain-name-servers <dns-addr>; 
}

apt install dnsutils                    # 6. Install test tools (dig)
dig www.mlb.com                         # 7. Run same cmd twice and check query time
...
;; Query time: 15 msec
...

dig www.mlb.com
...
;; Query time: 0 msec                   # Much faster! Cached response
...
```

### Internal DNS 

To resolve local hostnames, you need a zone file - a text file that includes some config, a list of hosts and IP addresses:
- Edit `/etc/bind/named.conf.local`

```bash
vim /etc/bind/named.conf.local
...
zone "local.lan" IN {                       # 1. Define zone and path to its config
        type master;
        file "/etc/bind/net.local.lan";
};


vim /etc/bind/net.local.lan                 # 2. Create zone config
$TTL 1D                                         # Time to Live (TTL) - how long a record is cached in the server
@ IN SOA local.lan. hostmaster.local.lan. (     # Start of Authority (SOA) - this DNS server ia authoritative over local.lan
                                                # hostmaster.local.lan is email for admin (or server owner)
202401131; serial                               # Serial number - bind uses it to track changes to file. Must increase by 1
                                                # after each change or bind won't notice - today's date + 1 digit
8H ; refresh                                    # Secondary DNS check every 8 hours for zone updates 
4H ; retry                                      # If there was an error, secondary waits 4 hours to check back in
4W ; expire                                     # Maximum age of zone file
1D ) ; minimum                                  # Minimum age of zone file
IN A 192.168.1.1                                # Name server IP
;
@ IN NS hermes.local.lan.                       # Name server hostname
fileserv        IN  A   192.168.1.3             # Hosts on local network that we want to resolve by name
hermes          IN  A   192.168.1.1             # Nameserver itself (might cause issues if omitted)
mailserv        IN  A   192.168.1.5
mail            IN  CNAME       mailserv.       # CNAME is a pointer to another resource - mailserv in this case
web01           IN  A   192.168.1.7

systemctl restart bind9                     # 3. Restart service
systemctl status bind9                      # 4. Check for errors
```

### Internet gateway setup

A gateway is the device that you go through to route from one network to another - the wider internet:
- Might want to take advantage of regular security releases
  - You have to add the security after installation!
- Typically a commercial router or firewall
- Admins often build DNS, DCHP, and routers into the same server
- Router needs two network interfaces:
  - One connected to ISP device - needs to use DHCP
  - One connected to a network switch that other servers connect to

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward         # turn server into router until reboot

cat /etc/sysctl.conf | grep net.ipv4.ip 
#net.ipv4.ip_forward=1                                  # uncomment this line to make it persist
```