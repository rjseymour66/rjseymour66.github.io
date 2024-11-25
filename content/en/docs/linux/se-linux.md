---
title: "SELinux"
linkTitle: "SELinux"
# weight: 1000
# description:
---

SELinux is native to RHEL, you must install on Debian:

```bash
# install on ubuntu 24
apt install selinux-utils policycoreutils setools selinux-basics

# manually activate selinux then reboot
selinux-activate
reboot

# get current SELinux status
sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             default
Current mode:                   permissive
Mode from config file:          permissive
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      33

# settings in config file
cat /etc/selinux/config 
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
# enforcing - SELinux security policy is enforced.
# permissive - SELinux prints warnings instead of enforcing.
# disabled - No SELinux policy is loaded.
SELINUX=permissive
# SELINUXTYPE= can take one of these two values:
# default - equivalent to the old strict and targeted policies
# mls     - Multi-Level Security (for military and educational use)
# src     - Custom policy built from source
SELINUXTYPE=default

# SETLOCALDEFS= Check local definition changes
SETLOCALDEFS=0

# get SELinux file status
ll -Z index.html 
-rw-r--r--. 1 root root system_u:object_r:httpd_sys_content_t:s0 382 Nov 20 01:22 index.html

# change context type
chcon -t samba_share_t /var/www/html/index.html 
# check SELinux file status
/var/www/html$ ll -Z index.html 
-rw-r--r--. 1 root root system_u:object_r:samba_share_t:s0 382 Nov 20 01:22 index.html


```