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

#### Master config file

- In `/etc/systemd/system.conf`
- `man systemd-system.conf` for details

### Service unit files

Complete this:
https://linuxhandbook.com/create-systemd-services/

A service unit file is a config file for a service. Includes service info, such as:
- which env file to use
- when the service starts
- what targets want to start this service

Service unit files are located in different directories. If you have multiple copies of this file, the system uses them in the following priority order:
1. `/etc/systemd/system/`
2. `/run/systemd/system/`
3. `/usr/lib/systemd/system/`

Common states:
- `enabled`: starts at boot
- `disabled`: does not start at boot
- `static`: starts if another service depends on it, or manual start

```bash
# state is when the service starts
$ systemctl list-unit-files
UNIT FILE                                      STATE           VENDOR PRESET
proc-sys-fs-binfmt_misc.automount              static          -            
-.mount                                        generated       -            
boot-efi.mount                                 generated       -            
dev-hugepages.mount                            static          -            
dev-mqueue.mount                               static          -            
proc-sys-fs-binfmt_misc.mount                  disabled        disabled     
snap-bare-5.mount                              enabled         enabled      
snap-chromium-2768.mount                       enabled         enabled      
...  
systemd-ask-password-console.path              static          -            
systemd-ask-password-plymouth.path             static          -            
systemd-ask-password-wall.path                 static          -            
whoopsie.path                                  enabled         enabled      
session-3.scope                                transient       -            
accounts-daemon.service                        enabled         enabled      
acpid.service                                  disabled        enabled      
     

486 unit files listed.

# find systemd unit file
systemctl cat cron.service 
# /lib/systemd/system/cron.service
[Unit]
Description=Regular background program processing daemon
Documentation=man:cron(8)
After=remote-fs.target nss-user-lookup.target

[Service]
EnvironmentFile=-/etc/default/cron
ExecStart=/usr/sbin/cron -f -P $EXTRA_OPTS
IgnoreSIGPIPE=false
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

#### [Unit]
Contains directives, a setting that modifies a configuration:
- `After`: Start this unit after the units listed here
- `Before`: Start this unit before the units listed here
- `Description`: Describes the unit
- `Documentation`: List of URIs that point to doc sources.
- `Conflicts`: Do not start this unit if the units listed here are started. Opposite of `Requries`.
- `Requires`: Start together with units listed here. If any in the list don't start, do not start this unit.
- `Wants`: Start together with units listed here. If any in the list don't start, still start this unit.


#### [Service]

Sets configuraiton items for service unit files:
- `ExecReload`: scripts, commands, or options to run when reloaded
- `ExecStart`: scripts, commands, or options to run when started
- `ExecStop`: scripts, commands, or options to run when stopped
- `Environment`: Environment variable substitutions
- `Environment File`: File that contains environment variable substitutes
- `RemainAfterExit`: `no` (default) or `yes`:
  - `yes`: service is active even when the process started with `ExecStart` terminates
  - `no`: `ExecStop` is called after the process started with `ExecStart` terminates
- `Type`: startup type
  - `forking`: `ExecStart` starts a parent process that creates the service's main process as a child process and exists
  - `simple` (default): `ExecStart` starts the service's process
  - `oneshot`: `ExecStart` starts the service's main process which is typically a configuration or quick command and exits
  - `idle`: `ExecStart` starts the service's main process and waits until all other start jobs are finished.

#### [Install]

Describes what happens to a service if it is enabled or disabled:

- `Alias`: Sets additional names that systemctl can use to call the service
- `Also`: Sets additional units that must be enabled or disabled for this service. Other units are usually sockets.
- `RequiredBy`: Other units that require this service.
- `WantedBy`: Which target unit manages this service.

#### Get info about systemd and unit config files
```bash
man -k systemd
30-systemd-environment-d-generator (8) - Load variables specified by environment.d
deb-systemd-helper (1p) - subset of systemctl for machines not running systemd
deb-systemd-invoke (1p) - wrapper around systemctl, respecting policy-rc.d
gnome-logs (1)       - log viewer for the systemd journal
init (1)             - systemd system and service manager
journalctl (1)       - Query the systemd journal

# info about unit files, etc
man systemd.service

# info about directives
man systemd.directives
```

### Target unit files

Target unit files group together various services to start at system boot time. The default target file is `default.target` is symbolically linked to the target file used during boot:

Mostly the same directives, except for:
`AllowIsolate`: target file can be used with `systemctl isolate`.
```bash
systemctl get-default
graphical.target
ryanseymour:~/Development/rjseymour66.github.io (master)
$ systemctl cat graphical.target 
# /lib/systemd/system/graphical.target
#  SPDX-License-Identifier: LGPL-2.1-or-later
#
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=Graphical Interface
Documentation=man:systemd.special(7)
Requires=multi-user.target
Wants=display-manager.service
Conflicts=rescue.service rescue.target
After=multi-user.target rescue.service rescue.target display-manager.service
AllowIsolate=yes
```

### Modifying system configuration files

> NEVER modify unit files in these directories:
> - `/lib/systemd/system/`
> - `/usr/lib/systemd/system/`

If you need to modify a file, copy it and place it in `/etc/systemd/system`. This directory takes precedence over the original location, and you can preserve the original. Also, it won't be impacted by software updates.

### Extending configuration

You can extend a service configuration with a _drop in_ file. Complete the following as a super user:
1. In `/etc/systemd/system/`, create a new subdirectory named `service.service-name.d`. For example, `/etc/systemd/system/service.sshd.d/`.
2. Create any configuration files, such as `description.conf`.
3. Run `systemd-delta` to list extended or duplicated unit files. This is so you do not overwrite unit files that are already extended:
   ```bash
   systemd-delta
   [EQUIVALENT] /etc/systemd/system/default.target → /usr/lib/systemd/system/default.target
   [EXTENDED]   /usr/lib/systemd/system/rc-local.service → /usr/lib/systemd/system/rc-local.service.d/debian.conf
   [EXTENDED]   /usr/lib/systemd/system/systemd-localed.service → /usr/lib/systemd/system/systemd-localed.service.d/locale-gen.conf
   [EXTENDED]   /usr/lib/systemd/system/user@.service → /usr/lib/systemd/system/user@.service.d/10-oomd-user-service-defaults.conf
   [EXTENDED]   /usr/lib/systemd/system/user@.service → /usr/lib/systemd/system/user@.service.d/timeout.conf

   5 overridden configuration files found.
   ```
1. Enter `systemctl daemon-reload <service>`.
2. Enter `systemctl restart <service>`.

## systemctl

How you manage systemd and services:

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
### Common commands

```bash
systemctl COMMAND UNIT_NAME...

# view status
systemctl status cron
systemctl stop sshd
systemctl is-active sshd
systemctl start sshd
```
service vs unit file?

| Command | Description |
|---------|-------------|
| daemon-reload | Load the unit config file withouth stopping the service. |
| disable | Mark service to NOT start at boot time. |
| enable | Mark service to start at boot time. |
| mask | Prevent this service from starting. Links the service to `/dev/null`.

 `--now` option stops this service immediately. `--running` makes the service until the next reboot or unmask operation. |
| restart | Stop and restart the service. |
| start | Start the service. |
| status | Display the service's status. |
| stop | Stop the service. |
| reload | Load the modified service config file to make changes without stopping the service. |
| unmask | Undo previous `mask` operation. |