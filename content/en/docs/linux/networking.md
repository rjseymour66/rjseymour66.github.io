---
title: "Networking"
linkTitle: "Networking"
# weight: 1000
# description:
---



## Static IP

Set a permanent IP address:

```bash
# bridged network adaptor virtualbox
cat /etc/netplan/99_config.yaml 
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: no
      addresses:
        - 192.168.56.50/24


# test the config - resets after 120s
sudo netplan try

# restart netplan
sudo netplan apply
```

Set IP for session:

```bash
# ip addr add <ip-addr> dev <interface> 
ip addr add 10.20.30.40/24 dev etho
```

## Ports

### Port designations

There are 65,535 ports

| Range | Description |
|:---|:---|
| 1 - 1023 | Well-known ports. Never use one of these ports for your own application. |
| 1024 - 49151 | Registered ports. Set aside for applications, even if you don't use them. Ex, MySQL uses registed port 3306. |
| 49152 - 65535 | Unregistered/private ports. Use these ports for your applications. |


### Open ports

An _open port_ is a port that is listening for requests for a service:

#### ss

Socket statistics:

```bash
ss                                           # list all connections
ss -a                                        # listening and non-listening ports
ss -l                                        # listening ports
ss -t                                        # list all TCP connections
ss -lt                                       # listening TCP connections
ss -ua                                       # list all UDP connections
ss -lu                                       # list all listening UDP connections
ss -p                                        # display socket PIDs
ss -s                                        # display summary statistics
ss -4                                        # display IPv4 connections
ss -6                                        # display IPv6 connections
ss -at '( dport = :22 or sport = :22 )'      # filter by source or dest port
ss -at '( dport = :ssh or sport = :ssh )'    # filter by source or dest port


# listening TCP ports with PID
ss -ltp
State                    Recv-Q                   Send-Q                                     Local Address:Port                                       Peer Address:Port                  Process                   
LISTEN                   0                        4096                                             0.0.0.0:ssh                                             0.0.0.0:*                                               
LISTEN                   0                        4096                                       127.0.0.53%lo:domain                                          0.0.0.0:*                                               
LISTEN                   0                        4096                                          127.0.0.54:domain                                          0.0.0.0:*                                               
LISTEN                   0                        511                                                    *:http                                                  *:*                                               
LISTEN                   0                        4096                                                [::]:ssh                                                [::]:*            
```

#### netstat
```bash
# netstat
# n - show port number
# p - show PID
# l - listening sockets only
netstat -npl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      1000/mariadbd       
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      545/systemd-resolve 
tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN      545/systemd-resolve 
tcp6       0      0 :::22                   :::*                    LISTEN      1/init              
tcp6       0      0 :::80                   :::*                    LISTEN      1095/apache2        
udp        0      0 127.0.0.54:53           0.0.0.0:*                           545/systemd-resolve 
udp        0      0 127.0.0.53:53           0.0.0.0:*                           545/systemd-resolve 
udp        0      0 10.0.2.15:68            0.0.0.0:*                           879/systemd-network 
raw6       0      0 :::58                   :::*                    7           879/systemd-network 
raw6       0      0 :::58                   :::*                    7           879/systemd-network 
Active UNIX domain sockets (only servers)
Proto RefCnt Flags       Type       State         I-Node   PID/Program name     Path
unix  2      [ ACC ]     STREAM     LISTENING     4385     1/init               /run/lvm/lvmpolld.socket
unix  2      [ ACC ]     STREAM     LISTENING     4390     1/init               /run/systemd/fsck.progress
unix  2      [ ACC ]     STREAM     LISTENING     4396     1/init               /run/systemd/journal/stdout
unix  2      [ ACC ]     SEQPACKET  LISTENING     4400     1/init               /run/udev/control
unix  2      [ ACC ]     STREAM     LISTENING     8800     1227/systemd         /run/user/1000/systemd/private
unix  2      [ ACC ]     STREAM     LISTENING     8811     1227/systemd         /run/user/1000/bus
unix  2      [ ACC ]     STREAM     LISTENING     4470     319/systemd-journal  /run/systemd/journal/io.systemd.journal
unix  2      [ ACC ]     STREAM     LISTENING     8812     1227/systemd         /run/user/1000/gnupg/S.dirmngr
...
```

## iftop

Displays greediest network activity on a network interface:

```bash
# see all traffic wlp59s0
iftop -i wlp59s0
```

## nethogs

Like `iftop`, but includes PIDs:

```bash
nethogs wlp59s0
# ...
NetHogs version 0.8.6-3

    PID USER     PROGRAM                                                                                                                                                       DEV         SENT      RECEIVED      
 954504 ryanse.. /opt/google/chrome/chrome --type=utility --utility-sub-type=network.mojom.NetworkService --lang=en-US --service-sandbox-type=none --string-annotations --c..  wlp59s      0.722       0.127 KB/sec
   1464 root     /usr/sbin/NetworkManager                                                                                                                                      wlp59s      0.000       0.000 KB/sec
      ? root     unknown TCP       
```

## tc

Traffic control. Lets you manage the bandwidth available to specific connections use on your system:

```bash
# view current tc rules for an interface
tc -s qdisc ls dev enp0s3
qdisc fq_codel 0: root refcnt 2 limit 10240p flows 1024 quantum 1514 target 5ms interval 100ms memory_limit 32Mb ecn drop_batch 64 
 Sent 137233 bytes 2012 pkt (dropped 0, overlimits 0 requeues 0) 
 backlog 0b 0p requeues 0
  maxpacket 54 drop_overlimit 0 new_flow_count 1 ecn_mark 0
  new_flows_len 0 old_flows_len 0

# add 100ms to each network transfer
tc qdisc add dev enp0s3 root netem delay 100ms
# check the rule was added
tc -s qdisc ls dev enp0s3
qdisc netem 8001: root refcnt 2 limit 1000 delay 100ms    # <- new rule
 Sent 0 bytes 0 pkt (dropped 0, overlimits 0 requeues 0) 
 backlog 0b 0p requeues 0
# delete the rule
tc qdisc del dev enp0s3 root
```

## nmon

Network monitor:

```bash
# open menu
nmon
```

## Troubleshooting strategy

1. Check your network interfaces
2. Ping the router
3. Ping a DNS address
4. Traceroute a destination address


### ip

View interface information:

```bash
ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:e9:0d:72 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 metric 100 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 77232sec preferred_lft 77232sec
    inet6 fe80::a00:27ff:fee9:d72/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:8d:49:bb brd ff:ff:ff:ff:ff:ff
    inet 192.30.40.50/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe8d:49bb/64 scope link 
       valid_lft forever preferred_lft forever
```

### lspci

List all PCI hardware on your machine:

```bash
# "Controller" is a hardware network internet device
lspci | grep Controller
00:01.0 PCI bridge: Intel Corporation 6th-10th Gen Core Processor PCIe Controller (x16) (rev 07)
00:12.0 Signal processing controller: Intel Corporation Cannon Lake PCH Thermal Controller (rev 10)
00:14.0 USB controller: Intel Corporation Cannon Lake PCH USB 3.1 xHCI Host Controller (rev 10)
00:15.0 Serial bus controller: Intel Corporation Cannon Lake PCH Serial IO I2C Controller #0 (rev 10)
00:15.1 Serial bus controller: Intel Corporation Cannon Lake PCH Serial IO I2C Controller #1 (rev 10)
00:16.0 Communication controller: Intel Corporation Cannon Lake PCH HECI Controller (rev 10)
00:17.0 SATA controller: Intel Corporation Cannon Lake Mobile PCH SATA AHCI Controller (rev 10)
00:1f.0 ISA bridge: Intel Corporation Cannon Lake LPC Controller (rev 10)
00:1f.4 SMBus: Intel Corporation Cannon Lake PCH SMBus Controller (rev 10)
00:1f.5 Serial bus controller: Intel Corporation Cannon Lake PCH SPI Controller (rev 10)
3a:00.0 USB controller: Intel Corporation JHL6340 Thunderbolt 3 USB 3.1 Controller (C step) [Alpine Ridge 2C 2016] (rev 02)
3d:00.0 Non-Volatile memory controller: Toshiba Corporation XG6 NVMe SSD Controller

```


### ip route

List your computer's routing table:

```bash
ip route
default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100 
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100 
10.0.2.2 dev enp0s3 proto dhcp scope link src 10.0.2.15 metric 100 
10.0.2.3 dev enp0s3 proto dhcp scope link src 10.0.2.15 metric 100 
192.168.56.0/24 dev enp0s8 proto kernel scope link src 192.168.56.50 
```

### traceroute

Traces a packet's route across the network to its destination:

```bash
# "* * *" means the packet didn't make it back
traceroute google.com
traceroute to google.com (142.250.65.206), 64 hops max
  1   10.0.2.2  0.873ms  0.249ms  0.295ms 
  2   192.168.1.1  2.828ms  2.900ms  2.902ms 
  3   96.230.114.1  7.333ms  7.806ms  8.325ms 
  4   100.41.25.202  7.442ms  5.157ms  6.292ms 
  5   *  *  * 
  6   209.85.149.208  13.637ms  10.866ms  11.418ms 
  7   *  *  * 
  8   142.251.65.102  11.870ms  10.729ms  11.936ms 
  9   142.251.60.239  10.140ms  10.091ms  10.276ms 
 10   142.250.65.206  10.229ms  10.540ms  10.730ms 
```

### netstat

Displays TCP network connection system info:

```bash
# all sockets currently listening
netstat -l 
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 _localdnsproxy:domain   0.0.0.0:*               LISTEN     
tcp        0      0 localhost:smtp          0.0.0.0:*               LISTEN     
tcp        0      0 localhost:mysql         0.0.0.0:*               LISTEN     
tcp        0      0 _localdnsstub:domain    0.0.0.0:*               LISTEN     
tcp6       0      0 [::]:http               [::]:*                  LISTEN     
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN     
udp        0      0 _localdnsproxy:domain   0.0.0.0:*                          
udp        0      0 _localdnsstub:domain    0.0.0.0:*                          
udp        0      0 ubuntu-24:bootpc        0.0.0.0:*                          
raw6       0      0 [::]:ipv6-icmp          [::]:*                  7          
raw6       0      0 [::]:ipv6-icmp          [::]:*                  7          
Active UNIX domain sockets (only servers)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  2      [ ACC ]     STREAM     LISTENING     3579     /run/systemd/userdb/io.systemd.DynamicUser
unix  2      [ ACC ]     STREAM     LISTENING     3580     /run/systemd/io.systemd.ManagedOOM
...

# list network interfaces
# RX - packets recieved
# TX - packets transmitted
# ERR - errors
# DRP - dropped

netstat -i
Kernel Interface table
Iface             MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
enp0s3           1500    32902      0      0 0          2566      0      0      0 BMRU
enp0s8           1500     4327      0      0 0          3158      0      0      0 BMRU
lo              65536      293      0      0 0           293      0      0      0 LRU
```

### netcat or nc

Read and write data across the network with TCP or UDP:

```bash
# scan for listening daemons
nc -z -v bootstrap-it.com 443 80
Connection to bootstrap-it.com (52.3.203.146) 443 port [tcp/https] succeeded!
Connection to bootstrap-it.com (52.3.203.146) 80 port [tcp/http] succeeded!
```

### nmap

Scan servers you own or localhost:

```bash
# scans for TCP connection (-sT) at port 80
nmap -sT -p80 bootstrap-it.com
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-29 15:18 UTC
Nmap scan report for bootstrap-it.com (52.3.203.146)
Host is up (0.016s latency).
rDNS record for 52.3.203.146: ec2-52-3-203-146.compute-1.amazonaws.com

PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 0.22 seconds


# scan for open ports within a range
nmap -sT -p1-1023 bootstrap-it.com
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-29 15:11 UTC
Nmap scan report for bootstrap-it.com (52.3.203.146)
Host is up (0.019s latency).
rDNS record for 52.3.203.146: ec2-52-3-203-146.compute-1.amazonaws.com
Not shown: 1018 filtered tcp ports (no-response)
PORT    STATE  SERVICE
25/tcp  closed smtp
80/tcp  open   http
443/tcp open   https
465/tcp closed smtps
587/tcp closed submission

Nmap done: 1 IP address (1 host up) scanned in 4.87 seconds

# scan localhost for bound services
nmap localhost
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-08 10:31 EST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000082s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 0.12 seconds

# scan machine to see whats open to the network
nmap 192.168.20.10
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-08 10:33 EST
Nmap scan report for vpn-server (192.168.20.10)
Host is up (0.000095s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 0.14 seconds

```