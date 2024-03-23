---
title: "System startup and services"
weight: 60
description: >
  How a manage system startup and services.
---

The `init` daemon determines which services are started and in what order, and it lets you start and stop services. There are two initialization methods:
- SysV: this is older, and based on Unix System V init daemon
- systemd: new since 2010, reduces initialization time by starting services in parallel

There was a method called `Upstart` that 

## init

- parent process for every service on the systemlocated in `/etc/ 


```bash
# see all services from init
pstree -p 1

# find full path of shell command
which init
/usr/sbin/init

# see if init is linked
sudo readlink -f /usr/sbin/init 
[sudo] password for ryanseymour: 
# linked to systemd
/usr/lib/systemd/systemd

# check PID 1 
ps -p 1
    PID TTY          TIME CMD
      1 ?        00:01:29 systemd
```

### ps

View processes. A process is a running program. What program is running for a particular process in the CMD column.

```bash
# check PID 1 
ps -p 1
    PID TTY          TIME CMD
      1 ?        00:01:29 systemd
```

## systemd

Big change in how systems manage services. Services can start:
- during boot
- when you attach hardware to the system
- when other services are started
- etc...

### Units and unit files

A systemd unit defines a service, a group of services, or an action. Each unit has the following:
- name
- type
- config file

The twelve systemd unit types:
- automount
- device
- mount
- path
- scope
- service
- slice
- snapshot
- socket
- swap
- target
- timer

### systemctl

How you manage systemd and services.:

```bash
# uses less pager by default (or --no-pager option)
systemctl [OPTIONS...] COMMAND [NAME...]

# list all units loaded in your system in <name>.<type> format
systemctl list-units
  UNIT                                      LOAD   ACTIVE     SUB       DESCRIPTION         >

  dev-loop1.device                          loaded activating tentative /dev/loop1
  dev-loop30.device                         loaded activating tentative /dev/loop30
  sys-devices-virtual-block-loop0.device    loaded active     plugged   /sys/devices/virtual/block/loop0
  sys-devices-virtual-block-loop10.device   loaded active     plugged   /sys/devices/virtual/block/loop10
  ...

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.
307 loaded units listed. Pass --all to see loaded but inactive units, too.
To show all installed unit files use 'systemctl list-unit-files'.

```

### Target unit files

Groups of services are started with target unit files. At startup, `default.target` unit ensures that all required and desired services are launched at system initialization:

```bash
# get default.target without systemctl
# 1. find the file in the fs
find / -name default.target 2>/dev/null
...
/usr/lib/systemd/system/default.target
...
# 2. follow the link
readlink -f /usr/lib/systemd/system/default.target 
/usr/lib/systemd/system/graphical.target

# get easily with systemctl (USE THIS)
systemctl get-default
graphical.target
```

Common system boot target unit files

| Name | Description |
|------|-------------|
| `graphical.target`      | Gives multiple users access to the system with local terminals or network. GUI access. |
| `multi-user.target`     | Gives multiple users access to the system with local terminals or network. No GUI. |
| `network-online.target` | Runs after the system connects to the network for apps that require a network to be present. |
| `runlevel.target`       | Provides backward compatibility to SysV systems. N is set to 1-5 for the desired SysV equivalence. |

#### Master confi file

- In `/etc/systemd/system.conf`
- `man systemd-system.conf` for details