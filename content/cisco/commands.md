+++
title = 'Commands'
date = '2026-11-23T23:29:05-05:00'
weight = 100
draft = false
+++

## Cisco IOS CLI

| Command | Description |
| --- | --- |
| `> enable` | Enter privileged EXEC mode. |
| `# disable` | Return to user EXEC mode. |
| `# reload` | Reboot the device. |
| `# show running-config` | View the running-config. |
| `# show startup-config` | View the startup-config. |
| `# write` | Save the device configuration by copying the running-config to the startup-config. |
| `# write memory` | Save the device configuration by copying the running-config to the startup-config. |
| `# copy running-config startup-config` | Save the device configuration by copying the running-config to the startup-config. |
| `# write erase` | Erase the startup-config. The device loads a factory-default configuration after the next reboot. |
| `# erase startup-config` | Erase the startup-config. The device loads a factory-default configuration after the next reboot. |
| `# erase nvram:` | Erase the startup-config. The device loads a factory-default configuration after the next reboot. |
| `# show command \| include string` | Limit `show` command output to lines that include the specified string. |
| `(config)# hostname name` | Set the device's hostname. |
| `(config)# enable password password` | Configure an unhashed enable password. |
| `(config)# enable secret password` | Configure a hashed enable secret. |
| `(config)# service password-encryption` | Encrypt passwords in the configuration with type-7 encryption (weak). |
| `(config)# exit` | Return to the previous level in the IOS CLI hierarchy. |
| `(config)# end` | Return to privileged EXEC mode from any configuration mode. |

## Ethernet LAN switching

| Command | Description |
| --- | --- |
| `# show mac address-table` | View the switch's MAC address table. |
| `# clear mac address-table dynamic` | Remove all dynamic MAC addresses. |
| `# clear mac address-table dynamic address address` | Remove the specified dynamic MAC address. |
| <code># clear mac address-table dynamic interface<br>&nbsp;&nbsp;interface</code> | Remove all dynamic MAC addresses learned on the specified interface. |
| `# ping ip-address` | Ping the specified host (ICMP echo/reply). |

## IPv4 addressing

| Command | Description |
| --- | --- |
| `# show ip interface [interface]` | View detailed Layer 3 information about interfaces. |
| `# show ip interface brief` | View a concise summary of interface IP addresses and status. |
| `(config)# interface interface` | Enter interface config mode for the specified interface. |
| `(config-if)# ip address ip-address netmask` | Set the interface's IP address and netmask. |
| `(config-if)# [no] shutdown` | Enable or disable the interface. |

## Router and switch interfaces

| Command | Description |
| --- | --- |
| `# show interfaces [interface]` | View interface Layer 1 and Layer 2 information. |
| `# show interfaces description` | View interface descriptions and status. |
| `# show interfaces status` | View interface status, speed, and duplex. |
| `(config)# interface range interface-list` | Configure multiple interfaces at once. |
| `(config-if)# description description` | Set the interface's description. |
| `(config-if)# speed {auto \| speed}` | Set the interface's speed. |
| `(config-if)# duplex {auto \| full \| half}` | Set the interface's duplex. |

## Routing fundamentals

| Command | Description |
| --- | --- |
| `# show ip route` | View the routing table. |
| <code>(config)# ip route destination-network netmask<br>&nbsp;&nbsp;next-hop</code> | Configure a recursive static route. |
| <code>(config)# ip route destination-network netmask<br>&nbsp;&nbsp;exit-interface</code> | Configure a directly connected static route. |
| <code>(config)# ip route destination-network netmask<br>&nbsp;&nbsp;exit-interface next-hop</code> | Configure a fully specified static route. |

## VLANs

| Command | Description |
| --- | --- |
| `# show vlan brief` | View a list of VLANs and associated ports. |
| `# show interfaces trunk` | View a list of trunk ports. |
| `(config)# vlan vlan-id` | Configure a VLAN. |
| `(config-vlan)# name name` | Name the VLAN. |
| `(config-vlan)# [no] shutdown` | Enable or disable the VLAN. |
| `(config-if)# switchport mode {access \| trunk}` | Manually configure access or trunk mode. |
| `(config-if)# switchport access vlan vlan-id` | Set the port's access VLAN. |
| `(config-if)# switchport trunk encapsulation dot1q` | Set the port's trunk encapsulation protocol. |
| <code>(config-if)# switchport trunk allowed vlan {<br>&nbsp;&nbsp;vlans \| add vlans \| remove vlans \|<br>&nbsp;&nbsp;except vlans \| all \| none<br>}</code> | Specify or modify the port's list of allowed VLANs (when in trunk mode). |
| `(config-if)# switchport trunk native vlan vlan-id` | Specify the port's native VLAN. |
| `(config)# interface interface.subif-number` | Configure a subinterface (ROAS). |
| `(config-subif)# encapsulation dot1q vlan [native]` | Specify the subinterface's VLAN ID. |
| `(config)# ip routing` | Enable IP routing on a multilayer switch. |
| `(config)# interface vlan vlan-id` | Configure a switch virtual interface (SVI). |
| `(config-if)# no switchport` | Make the port a Layer 3 routed port. |

## Dynamic Trunking Protocol and VLAN Trunking Protocol

| Command | Description |
| --- | --- |
| `# show interfaces switchport` | Show port operational and administrative modes. |
| <code>(config-if)# switchport mode dynamic<br>&nbsp;&nbsp;{auto \| desirable}</code> | Use DTP to determine the port's operational mode. |
| `(config-if)# switchport nonegotiate` | Disable DTP on the port. |
| `# show vtp status` | View VTP information. |
| `(config)# vtp domain name` | Set the VTP domain name. |
| <code>(config)# vtp mode<br>&nbsp;&nbsp;{server \| client \| transparent \| off}</code> | Set the VTP mode. |
| `(config)# vtp version {1 \| 2 \| 3}` | Set the VTP version. |
| `# vtp primary` | Make this switch the VTP primary server (VTP version 3 only). |

## Spanning Tree Protocol and Rapid Spanning Tree Protocol

| Command | Description |
| --- | --- |
| `# show spanning-tree` | View STP status and information. |
| <code>(config)# spanning-tree vlan vlan priority<br>&nbsp;&nbsp;priority</code> | Set the STP bridge priority. |
| `(config)# spanning-tree vlan vlan root primary` | Set the bridge priority to 24576, or the lowest increment of 4096 needed to become the root bridge. |
| `(config)# spanning-tree vlan vlan root secondary` | Set the bridge priority to 28672. |
| <code>(config-if)# spanning-tree vlan vlan port-priority<br>&nbsp;&nbsp;priority</code> | Set the STP port priority. |
| `(config-if)# spanning-tree portfast enable` | Enable PortFast on the port. |
| `(config)# spanning-tree portfast default` | Enable PortFast on all access ports. |
| `(config-if)# spanning-tree bpduguard enable` | Enable BPDU Guard on the port. |
| `(config)# spanning-tree portfast bpduguard default` | Enable BPDU Guard on all PortFast-enabled ports. |
| `(config)# spanning-tree mode {pvst \| rapid-pvst}` | Set the STP mode (standard or rapid). |
| `# show spanning-tree pathcost method` | View the port cost calculation method. |
| <code>(config)# spanning-tree pathcost method<br>&nbsp;&nbsp;{short \| long}</code> | Set the port cost calculation method. |
| <code>(config-if)# spanning-tree link-type<br>&nbsp;&nbsp;{point-to-point \| shared}</code> | Set the port's RSTP link type. |

## EtherChannel

| Command | Description |
| --- | --- |
| <code>(config-if)# channel-group group mode<br>&nbsp;&nbsp;{desirable \| auto}</code> | Configure a PAgP EtherChannel. |
| <code>(config-if)# channel-group group mode<br>&nbsp;&nbsp;{active \| passive}</code> | Configure an LACP EtherChannel. |
| `(config-if)# channel-group group mode on` | Configure a static EtherChannel. |
| `(config)# interface port-channel number` | Configure the port-channel interface. |
| `# show etherchannel summary` | View EtherChannel status. |
| `# show pagp neighbor` | View PAgP neighbor status. |
| `# show lacp neighbor` | View LACP neighbor status. |
| `(config)# port-channel load-balance parameters` | Modify the EtherChannel load-balancing parameters. |
| `# show etherchannel load-balance` | View the load-balancing parameters. |

## Dynamic routing and Open Shortest Path First

| Command | Description |
| --- | --- |
| <code>(config)# ip route destination-network netmask<br>&nbsp;&nbsp;next-hop administrative-distance</code> | Configure a floating static route. |
| `(config)# router ospf process-id` | Create or configure an OSPF process. |
| <code>(config-router)# network ip-address wildcard-mask<br>&nbsp;&nbsp;area area</code> | Activate OSPF on interfaces with an IP address in the specified range. |
| <code>(config-router)# auto-cost reference-bandwidth<br>&nbsp;&nbsp;mbps</code> | Modify the OSPF cost reference bandwidth. |
| `(config-router)# passive-interface interface` | Configure a passive interface. |
| `(config-router)# passive-interface default` | Make all interfaces passive by default. |
| <code>(config-router)# default-information originate<br>&nbsp;&nbsp;[always]</code> | Advertise a default route to other routers. |
| `(config-router)# maximum-paths number` | Set the maximum number of routes inserted into the routing table for ECMP. |
| `(config-router)# router-id rid` | Manually set the OSPF router ID (RID). |
| `(config-if)# ip ospf process-id area area` | Activate OSPF on the interface. |
| <code>(config-if)# ip ospf network<br>&nbsp;&nbsp;{point-to-point \| broadcast}</code> | Set the interface's OSPF network type. |
| `(config-if)# ip ospf priority priority` | Set the interface's OSPF priority (for the DR/BDR election). |
| `(config-if)# ip ospf cost cost` | Set the interface's OSPF cost. |
| `(config-if)# bandwidth kbps` | Set the interface's bandwidth. |
| `(config-if)# ip ospf hello-interval seconds` | Set the interface's hello timer. |
| `(config-if)# ip ospf dead-interval seconds` | Set the interface's dead timer. |
| `(config-if)# ip ospf authentication` | Enable OSPF authentication. |
| `(config-if)# ip ospf authentication-key password` | Set the OSPF authentication password. |
| `(config-if)# ip mtu bytes` | Set the interface's IP MTU. |
| `(config)# interface loopback number` | Create or configure a loopback interface. |
| `# show ip protocols` | View information about the router's active routing protocols. |
| `# show ip ospf interface brief` | View OSPF-activated interfaces. |
| `# show ip ospf neighbor` | View OSPF neighbors and their status. |
| `# show ip ospf interface interface` | View information about an OSPF interface. |
| `# show ip ospf database` | View the OSPF LSDB. |
| `# clear ip ospf process` | Reset the OSPF process. |

## First Hop Redundancy Protocols

| Command | Description |
| --- | --- |
| `(config-if)# standby group ip virtual-ip` | Set the HSRP virtual IP address. |
| `(config-if)# standby group priority priority` | Set the HSRP priority. |
| `(config-if)# standby group preempt` | Enable HSRP preemption. |
| `(config-if)# standby version 2` | Enable HSRP version 2. |
| `# show standby [brief]` | View HSRP status and information. |

## IPv6 addressing and IPv6 routing

| Command | Description |
| --- | --- |
| `(config)# ipv6 unicast-routing` | Enable IPv6 routing. |
| `(config-if)# ipv6 address address/prefix-length` | Configure an IPv6 address. |
| <code>(config-if)# ipv6 address prefix/prefix-length<br>&nbsp;&nbsp;eui-64</code> | Configure an IPv6 address, using EUI-64 to generate the host portion. |
| <code>(config-if)# ipv6 address address/prefix-length<br>&nbsp;&nbsp;link-local</code> | Configure a link-local IPv6 address. |
| <code>(config-if)# ipv6 address address/prefix-length<br>&nbsp;&nbsp;anycast</code> | Configure an anycast IPv6 address. |
| `(config-if)# ipv6 address autoconfig [default]` | Use SLAAC to generate an IPv6 address. |
| `(config-if)# ipv6 enable` | Enable IPv6 on the interface without configuring an IPv6 address. |
| `# show ipv6 interface brief` | View a concise summary of interface IPv6 addresses and status. |
| `# show ipv6 interface [interface]` | View detailed Layer 3 information about IPv6 interfaces. |
| `# show ipv6 neighbors` | View the IPv6 neighbor table (like ARP). |
| `# show ipv6 route` | View the IPv6 routing table. |
| <code>(config)# ipv6 route destination-prefix next-hop<br>&nbsp;&nbsp;[ad]</code> | Configure a recursive static route. |
| <code>(config)# ipv6 route destination-prefix<br>&nbsp;&nbsp;exit-interface [ad]</code> | Configure a directly connected static route. |
| <code>(config)# ipv6 route destination-prefix<br>&nbsp;&nbsp;exit-interface next-hop [ad]</code> | Configure a fully specified static route. |

## Standard and extended access control lists

| Command | Description |
| --- | --- |
| <code>(config)# access-list number {permit \| deny}<br>&nbsp;&nbsp;src-ip wildcard</code> | Configure an ACE in a standard numbered ACL. |
| `(config)# access-list number remark remark` | Configure an ACL remark. |
| `(config)# ip access-list standard name` | Configure a standard named ACL. |
| <code>(config-std-nacl)# [seq] {permit \| deny} src-ip<br>&nbsp;&nbsp;wildcard</code> | Configure an ACE in the standard named ACL. |
| <code>(config)# access-list number {permit \| deny}<br>&nbsp;&nbsp;protocol source destination</code> | Configure an ACE in an extended numbered ACL, matching based on protocol and source and destination IP. |
| <code>(config)# access-list number {permit \| deny}<br>&nbsp;&nbsp;{tcp \| udp} source [operator src-port]<br>&nbsp;&nbsp;destination [operator dst-port]</code> | Configure an ACE in an extended numbered ACL, matching based on protocol, source and destination IP, and source and destination port. |
| `(config)# ip access-list extended name` | Configure an extended named ACL. |
| <code>(config-ext-nacl)# [seq] {permit \| deny} protocol<br>&nbsp;&nbsp;source destination</code> | Configure an ACE in the extended named ACL, matching based on protocol and source and destination IP. |
| <code>(config-ext-nacl)# [seq] {permit \| deny}<br>&nbsp;&nbsp;{tcp \| udp} source [operator src-port]<br>&nbsp;&nbsp;destination [operator dst-port]</code> | Configure an ACE in the extended named ACL, matching based on protocol, source and destination IP, and source and destination port. |
| `(config-if)# ip access-group acl {in \| out}` | Apply an ACL to an interface. |
| <code>(config)# ip access-list resequence acl<br>&nbsp;&nbsp;starting-seq-num increment</code> | Resequence an ACL. |
| `# show access-lists` | View all ACLs. |
| `# show ip access-lists` | View all IP ACLs. |

## Cisco Discovery Protocol and Link Layer Discovery Protocol

| Command | Description |
| --- | --- |
| `(config)# [no] cdp run` | Globally enable or disable CDP. |
| `(config)# cdp timer seconds` | Set the CDP advertisement timer. |
| `(config)# cdp holdtime seconds` | Set the CDP holdtime. |
| `(config)# [no] cdp advertise-v2` | Enable or disable CDPv2 advertisements. |
| `(config-if)# [no] cdp enable` | Enable or disable CDP on an interface. |
| `# show cdp` | View basic CDP status and information. |
| `# show cdp neighbors [detail]` | View information about CDP neighbors. |
| `# show cdp entry name` | View detailed information about the specified CDP neighbor. |
| `# show cdp interface [interface]` | View information about CDP interfaces. |
| `# show cdp traffic` | View CDP traffic statistics. |
| `(config)# [no] lldp run` | Globally enable or disable LLDP. |
| `(config)# lldp timer seconds` | Set the LLDP advertisement timer. |
| `(config)# lldp holdtime seconds` | Set the LLDP holdtime. |
| `(config)# lldp reinit seconds` | Set the LLDP reinitialization timer. |
| `(config-if)# [no] lldp transmit` | Enable or disable LLDP Tx on the interface. |
| `(config-if)# [no] lldp receive` | Enable or disable LLDP Rx on the interface. |
| `# show lldp` | View basic LLDP status and information. |
| `# show lldp neighbors [detail]` | View information about LLDP neighbors. |
| `# show lldp entry name` | View detailed information about the specified LLDP neighbor. |
| `# show lldp interface [interface]` | View information about LLDP interfaces. |
| `# show lldp traffic` | View LLDP traffic statistics. |

## Network Time Protocol

| Command | Description |
| --- | --- |
| `# show clock [detail]` | View the time of the software clock. |
| `# clock set hh:mm:ss month day year` | Set the time of the software clock. |
| `# show calendar` | View the time of the hardware calendar. |
| `# calendar set hh:mm:ss month day year` | Set the time of the hardware calendar. |
| `# clock update-calendar` | Sync the calendar to the clock's time. |
| `# clock read-calendar` | Sync the clock to the calendar's time. |
| <code>(config)# clock timezone name hours-offset<br>&nbsp;&nbsp;minutes-offset</code> | Set the device's time zone. |
| <code>(config)# clock summer-time name recurring<br>&nbsp;&nbsp;start-date-time end-date-time</code> | Enable summer time (daylight saving time). |
| `(config)# ntp server ip-address [prefer]` | Configure this device as an NTP client of the specified server. |
| `(config)# ntp update-calendar` | Use NTP to regularly update the calendar. |
| `(config)# ntp source interface` | Set the interface used to send NTP messages. |
| `(config)# ntp master [stratum]` | Make this device a primary NTP server with its own clock as the reference clock. |
| `(config)# ntp peer ip-address` | Configure a symmetric active NTP relationship on two devices. |
| <code>(config)# ntp authentication-key key-number md5<br>&nbsp;&nbsp;key</code> | Configure an NTP authentication key. |
| `(config)# ntp trusted-key key-number` | Specify an NTP trusted authentication key. |
| `(config)# ntp server ip-address key key-number` | Configure this device as an NTP client of the specified server and authenticate it with the specified key. |
| `# show ntp associations` | View a list of NTP associations (time sources). |
| `# show ntp status` | View the device's NTP status. |

## Domain Name System

| Command | Description |
| --- | --- |
| `(config)# ip domain lookup` | Enable the device to send DNS queries. |
| `(config)# ip name-server ip-address` | Configure the DNS server(s) to send DNS queries to. |
| `(config)# ip dns server` | Configure the device as a DNS server, allowing it to respond to DNS queries. |
| `(config)# ip domain name domain-name` | Set the device's default domain name. |
| `(config)# ip host hostname ip-address` | Create a manual name-to-IP mapping. |
| `# show hosts` | View all name-to-IP mappings. |

## Dynamic Host Configuration Protocol

| Command | Description |
| --- | --- |
| `(config)# ip dhcp excluded-address low-ip high-ip` | Specify a range of addresses that the device won't lease to DHCP clients. |
| `(config)# ip dhcp pool name` | Create or configure a DHCP pool. |
| <code>(dhcp-config)# network address<br>&nbsp;&nbsp;{/prefix-length \| netmask}</code> | Set the range of IP addresses to lease to clients. |
| `(dhcp-config)# default-router ip-address` | Set the pool's default gateway. |
| `(dhcp-config)# dns-server ip-address` | Set the pool's DNS server. |
| `(dhcp-config)# domain-name domain-name` | Set the pool's domain name. |
| `(dhcp-config)# lease days hours minutes` | Set the pool's lease time. |
| `# show ip dhcp binding` | View addresses leased to DHCP clients. |
| `# clear ip dhcp binding {* \| ip-address}` | Clear DHCP leases. |
| `# show ip dhcp conflict` | View DHCP conflicts. |
| `# clear ip dhcp conflict {* \| ip-address}` | Clear DHCP conflicts. |
| `(config-if)# ip address dhcp` | Configure the interface as a DHCP client. |
| `(config-if)# ip helper-address ip-address` | Configure the interface as a DHCP relay agent. |
| `(Windows) ipconfig` | View basic IP information. |
| `(Windows) ipconfig /all` | View more detailed IP information. |
| `(Windows) netstat -rn` | View the routing table. |
| `(macOS) ifconfig` | View interface information. |
| `(macOS) netstat -rn` | View the routing table. |
| `(Linux) ifconfig` | View interface information (net-tools). |
| `(Linux) route` | View the routing table (net-tools). |
| `(Linux) ip addr` | View interface information (iproute2). |
| `(Linux) ip route` | View the routing table (iproute2). |

## Secure Shell

| Command | Description |
| --- | --- |
| `(config)# username username secret password` | Create a local user account. |
| `(config)# line console 0` | Configure the console line. |
| `(config)# line vty 0 15` | Configure all 16 VTY lines. |
| `(config-line)# password password` | Configure a line password. |
| `(config-line)# login` | Require a user to provide the line password to access the line. |
| `(config-line)# login local` | Require a user to log in with a local user account to access the line. |
| `(config-line)# exec-timeout minutes` | Set the inactivity logout timer. |
| <code>(config-line)# transport input<br>&nbsp;&nbsp;{telnet \| ssh \| all \| none}</code> | Set the protocols that can be used to connect to the line. |
| `(config-line)# access-class acl in` | Apply an ACL to limit access to the line. |
| `(config)# ip default-gateway ip-address` | Set the Layer 2 switch's default gateway. |
| `# show ip default-gateway` | View the Layer 2 switch's default gateway. |
| `# show ip ssh` | View SSH settings and information. |
| `(config)# crypto key generate rsa` | Generate an RSA key pair (for SSH). |
| `(config)# ip ssh version 2` | Allow only SSH version 2 connections. |

## Simple Network Management Protocol

| Command | Description |
| --- | --- |
| `(config)# snmp-server community password ro` | Configure a read-only community string. |
| `(config)# snmp-server community password rw` | Configure a read-write community string. |
| `(config)# snmp-server enable traps` | Enable SNMP traps. |
| <code>(config)# snmp-server host ip-address [version 2c]<br>&nbsp;&nbsp;community-string</code> | Specify the NMS to send traps to. |

## Syslog

| Command | Description |
| --- | --- |
| `# terminal monitor` | Enable real-time logging to the VTY lines for the current session. |
| `(config)# logging console [severity]` | Configure logging to the console line. |
| `(config)# logging monitor [severity]` | Configure logging to the VTY lines. |
| `(config)# logging buffered [bytes] [severity]` | Configure logging to the logging buffer. |
| `(config)# logging trap [severity]` | Configure logging to a Syslog server. |
| `(config)# logging [host] ip-address` | Specify the Syslog server to send logs to. |
| <code>(config)# service timestamps log datetime<br>&nbsp;&nbsp;[localtime] [msec]</code> | Timestamp log messages with the current date and time. |
| `(config)# service timestamps log uptime` | Timestamp log messages with the device's current uptime. |
| `(config-line)# logging synchronous` | Automatically reprint the current command on a new line if cut off by a log message. |
| `# show logging` | Show Syslog settings and the logging buffer. |
| `# show memory` | Show device memory usage. |
| `# debug command` | Display detailed real-time information about various operations or events. |

## Trivial File Transfer Protocol and File Transfer Protocol

| Command | Description |
| --- | --- |
| `# show file systems` | View the device's file systems. |
| `# dir file-system` | View the contents of a file system. |
| `# more file-name` | View the contents of a file. |
| `# show flash:` | View the contents of flash memory. |
| `# copy src-file-path dst-file-path` | Copy a file. |
| `# copy tftp: flash:` | Copy a file from a TFTP server to flash. |
| `# copy ftp: flash:` | Copy a file from an FTP server to flash. |
| `(config)# ip ftp username username` | Set the default FTP username. |
| `(config)# ip ftp password password` | Set the default FTP password. |
| `# copy ftp://username:password@server-ip flash:` | Copy a file from an FTP server to flash, specifying the authentication credentials. |
| `# show version` | View information about the IOS version. |
| `# show boot` | View information about the boot image. |
| `(config)# boot system image-path` | Set the IOS image that will be loaded on the next boot. |

## Network address translation

| Command | Description |
| --- | --- |
| `(config-if)# ip nat inside` | Configure an inside interface. |
| `(config-if)# ip nat outside` | Configure an outside interface. |
| <code>(config)# ip nat inside source static inside-local<br>&nbsp;&nbsp;inside-global</code> | Configure a static NAT mapping. |
| <code>(config)# ip nat pool pool-name start-ip end-ip<br>&nbsp;&nbsp;prefix-length length</code> | Create a NAT pool. |
| <code>(config)# ip nat inside source list acl pool<br>&nbsp;&nbsp;pool-name</code> | Configure a dynamic NAT mapping of an ACL to a NAT pool. |
| <code>(config)# ip nat inside source list acl pool<br>&nbsp;&nbsp;pool-name overload</code> | Configure a dynamic PAT (NAT overload) mapping of an ACL to a NAT pool. |
| <code>(config)# ip nat inside source list acl interface<br>&nbsp;&nbsp;interface [overload]</code> | Configure a dynamic PAT mapping of an ACL to an interface IP address. |
| `# show ip nat translations` | View the NAT translation table. |
| `# show ip nat statistics` | View NAT translation statistics. |

## Quality of service

| Command | Description |
| --- | --- |
| `(config-if)# switchport voice vlan vlan-id` | Configure the port's voice VLAN. |
| `(config-if)# power inline auto` | Enable PoE on the port, and automatically detect and power devices. |
| `(config-if)# power inline static [max milliwatts]` | Enable PoE on the port with a static power reservation. |
| `(config-if)# power inline police` | Enable power policing on the port. |
| `(config-if)# power inline never` | Disable PoE on the port. |
| `# show power inline` | View PoE status and ports. |
| `# show power inline police` | View power policing status and ports. |

## Security concepts

| Command | Description |
| --- | --- |
| <code>(config)# enable algorithm-type<br>&nbsp;&nbsp;{md5 \| sha256 \| scrypt} secret password</code> | Configure an enable secret using the specified hashing algorithm. |
| `(config)# enable secret type hash` | Configure an already-hashed enable secret. |
| <code>(config)# username username algorithm-type<br>&nbsp;&nbsp;{md5 \| sha256 \| scrypt} secret password</code> | Configure a user account, and hash the password using the specified algorithm. |
| `(config)# username username secret type hash` | Configure a user account with an already-hashed password. |

## Port Security

| Command | Description |
| --- | --- |
| `(config-if)# switchport port-security` | Enable Port Security on the port. |
| <code>(config-if)# switchport port-security maximum<br>&nbsp;&nbsp;number</code> | Limit the maximum number of MAC addresses that can be learned on the port. |
| <code>(config-if)# switchport port-security mac-address<br>&nbsp;&nbsp;mac-address [voice vlan]</code> | Configure a static secure MAC address. |
| <code>(config-if)# switchport port-security violation<br>&nbsp;&nbsp;{shutdown \| restrict \| protect}</code> | Set the Port Security violation mode of the port. |
| <code>(config-if)# switchport port-security aging time<br>&nbsp;&nbsp;minutes</code> | Enable aging, and set the aging time for secure MAC addresses learned on the port. |
| <code>(config-if)# switchport port-security aging type<br>&nbsp;&nbsp;{absolute \| inactivity}</code> | Set the Port Security aging type. |
| `(config-if)# switchport port-security aging static` | Enable static secure MAC address aging. |
| <code>(config-if)# switchport port-security mac-address<br>&nbsp;&nbsp;sticky</code> | Enable sticky secure MAC address learning. |
| `# show port-security` | View a list of Port Security-enabled ports. |
| `# show port-security interface interface` | View Port Security settings for a port. |
| `# show mac address-table secure` | View secure MAC addresses. |
| <code>(config)# errdisable recovery cause<br>&nbsp;&nbsp;psecure-violation</code> | Enable ErrDisable Recovery for ports disabled by Port Security. |
| `(config)# errdisable recovery interval seconds` | Set the ErrDisable Recovery interval. |
| `# show errdisable recovery` | View ErrDisable Recovery settings. |

## DHCP Snooping

| Command | Description |
| --- | --- |
| `(config)# ip dhcp snooping` | Enable DHCP Snooping. |
| `(config)# ip dhcp snooping vlan vlans` | Activate DHCP Snooping on VLANs. |
| `(config)# no ip dhcp snooping information option` | Disable option 82 insertion. |
| `(config-if)# ip dhcp snooping trust` | Configure the port as a DHCP Snooping trusted port. |
| `(config-if)# ip dhcp snooping limit rate pps` | Rate-limit DHCP messages on the port. |
| <code>(config)# errdisable recovery cause<br>&nbsp;&nbsp;dhcp-rate-limit</code> | Enable ErrDisable Recovery for ports disabled by DHCP Snooping rate limiting. |
| `# show ip dhcp snooping` | View DHCP Snooping status and settings. |
| `# show ip dhcp snooping binding` | View the DHCP Snooping binding table. |

## Dynamic ARP inspection

| Command | Description |
| --- | --- |
| `(config)# ip arp inspection vlan vlans` | Enable DAI on the specified VLANs. |
| <code>(config)# ip arp inspection validate<br>&nbsp;&nbsp;{[src-mac] [dst-mac] [ip]}</code> | Enable optional DAI checks. |
| `(config-if)# ip arp inspection trust` | Configure the port as a DAI trusted port. |
| <code>(config-if)# ip arp inspection limit rate pps<br>&nbsp;&nbsp;[burst interval seconds]</code> | Rate-limit ARP messages on the port. |
| `(config)# errdisable recovery cause arp-inspection` | Enable ErrDisable Recovery for ports disabled by DAI rate limiting. |
| `# show ip arp inspection` | View DAI status and information. |
| `# show ip arp inspection interfaces` | View DAI interface status and settings. |

## Virtualization and cloud

| Command | Description |
| --- | --- |
| `(config)# ip vrf name` | Create a VRF instance. |
| `(config-if)# ip vrf forwarding vrf` | Assign the interface to a VRF. |
| `# show ip vrf` | View a list of VRFs and their interfaces. |
| `# show ip route vrf vrf` | View the specified VRF's routing table. |
| `# ping vrf vrf ip-address` | Ping a host from the specified VRF. |

## Wireless LAN configuration

| Command | Description |
| --- | --- |
| `(dhcp-config)# option 42 ip ip-address` | Set the DHCP pool's NTP server address. |
| `(dhcp-config)# option 43 ip ip-address` | Set the DHCP pool's WLC address (to help LWAPs identify their WLC). |

## Data formats

| Command | Description |
| --- | --- |
| `# show command \| format` | Display the output in XML format (only works for some commands). |
