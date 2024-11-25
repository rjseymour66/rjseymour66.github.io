---
title: "Systemd"
linkTitle: "Systemd"
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

## systemd timers

Create jobs run by systemd:

- create in `/etc/systemd/system`


```bash
# list all timers
systemctl list-timers --all
NEXT                            LEFT LAST                           PASSED UNIT                           ACTIVATES                       
Mon 2024-11-18 01:50:00 UTC 1min 12s Mon 2024-11-18 01:40:11 UTC  8min ago sysstat-collect.timer          sysstat-collect.service
Mon 2024-11-18 02:27:52 UTC    39min Mon 2024-11-18 01:01:51 UTC 46min ago fwupd-refresh.timer            fwupd-refresh.service
Mon 2024-11-18 02:37:40 UTC    48min Tue 2024-11-12 13:04:02 UTC         - motd-news.timer                motd-news.service
Mon 2024-11-18 06:21:09 UTC 4h 32min Sun 2024-11-17 22:54:28 UTC         - apt-daily-upgrade.timer        apt-daily-upgrade.service
Mon 2024-11-18 07:32:37 UTC 5h 43min -                                   - anacron.timer                  anacron.service
Mon 2024-11-18 10:14:24 UTC       8h Sun 2024-11-17 22:54:28 UTC         - man-db.timer                   man-db.service
Mon 2024-11-18 16:50:23 UTC      15h Mon 2024-11-18 01:29:55 UTC 18min ago apt-daily.timer                apt-daily.service
Tue 2024-11-19 00:00:00 UTC      22h Mon 2024-11-18 00:51:42 UTC 57min ago dpkg-db-backup.timer           dpkg-db-backup.service
...

19 timers listed.
```

## Check active services

```bash
systemctl list-unit-files --type=service --state=enabled
UNIT FILE                              STATE   PRESET 
anacron.service                        enabled enabled
apache2.service                        enabled enabled
apparmor.service                       enabled enabled
apport.service                         enabled enabled
blk-availability.service               enabled enabled
...
```