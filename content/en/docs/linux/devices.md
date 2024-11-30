---
title: "Devices"
linkTitle: "Devices"
# weight: 1000
# description:
---


## lsusb

Lists attached USB devices:

```bash
lsusb
Bus 004 Device 033: ID 0bda:8153 Realtek Semiconductor Corp. RTL8153 Gigabit Ethernet Adapter
Bus 004 Device 032: ID 2109:0817 VIA Labs, Inc. USB3.0 Hub             
Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
...
```

### lshw

List all hardware devices on your system:

```bash
# list only network devices
lshw -class network
  *-network                 
       description: Ethernet interface
       product: Wi-Fi 6 AX200
       ...
  *-network
       description: Ethernet interface
       physical id: e
       ...

# output as html
lshw -html > lshw-output.html

# memory info
lshw -c memory

# storage
lshw -c storage

# multimedia
lshw -c multimedia

# cpu
lshw -c cpu
```