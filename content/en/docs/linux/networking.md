---
title: "Networking"
linkTitle: "Networking"
# weight: 1000
# description:
---



## Static IP

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
        - 192.168.10.11/24
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

# ss
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
```