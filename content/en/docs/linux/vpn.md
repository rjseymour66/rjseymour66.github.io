---
title: "Virtual Private Networks (VPNs)"
linkTitle: "VPNs"
# weight: 1000
# description:
---

VPNs make remote networks and their clients together as if they are a local network.

## Configure VPN server

```bash
# install packages
apt install openvpn -y
apt install easy-rsa -y

# configure firewall
ufw enable
ufw allow 22    # allow ssh
ufw allow 1194  # allow OpenVPN port

# enable packet forwading
vim /etc/sysctl.conf 
#
# /etc/sysctl.conf - Configuration file for setting system variables
# See /etc/sysctl.d/ for additional system variables.
# See sysctl.conf (5) for information.
...
# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1       # uncomment this line

# load the new setting
sysctl -p
net.ipv4.ip_forward = 1

# copy easy-rsa template dir to openvpn
cp -r /usr/share/easy-rsa /etc/openvpn
```