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
        - 192.168.56.50/24
```