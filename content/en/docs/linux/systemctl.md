---
title: "systemctl"
linkTitle: ""
# weight: 1000
# description:
---




## systemctl

```bash
# check status of system daemon
$ dpkg -s openssh-server
Package: openssh-server
Status: install ok installed
Priority: optional
...

# stop system daemon
systemctl stop <daemon>

# autoload daemon on system startup
systemctl enable <daemon>
```