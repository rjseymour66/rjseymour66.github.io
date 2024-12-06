---
title: "Systemd"
linkTitle: "Systemd"
# weight: 1000
# description:
---

`systemd` is the init system, the first process that the kernel starts after boot that starts all other processes in user mode.
- Works with _units_ 
- There are types of units, specified by their extension (ex: `*.service`)
  - you can omit the extension when starting and stopping services
- `systemctl` executes instruction in unit files

## systemctl

Great [Digital Ocean tutorial](https://www.digitalocean.com/community/tutorials/how-to-use-systemctl-to-manage-systemd-services-and-units).

Management tool for `systemd`:

```bash
# check status of system daemon
$ dpkg -s openssh-server
Package: openssh-server
Status: install ok installed
Priority: optional
...

# ------------------------------------
# Units
systemctl list-units                            # list all active units
systemctl                                       # list all active units
systemctl list-units --all                      # list all units
systemctl list-units --all --state=inactive     # filter for inactive units
systemctl list-units --all --type=service       # filter by type
systemctl list-unit-files                       # list every available unit file
systemctl cat <service>                         # view currently loaded service file
systemctl list-dependencies <service>           # list dependencies (other units) required to start service
systemctl list-dependencies <service> --all     # include recursive dependencies (other units)
systemctl list-dependencies <service> --reverse # list dependencies that depend on the unit
systemctl list-dependencies --before <service>  # dependencies that must start before service
systemctl list-dependencies --after <service>   # dependencies that must start after service

systemctl edit <service>                        # edit a service as a snippet
systemctl edit --full <service>                 # edit a service directly
rm -r /etc/systemd/system/nginx.service.d       # remove changes made to unit as snippet
rm /etc/systemd/system/nginx.service            # remove changes made to unit

# ------------------------------------
# Unit Properties
systemctl show <service>                # list unit properties
systemctl show <service> -p <prop>      # show specific unit property

# ------------------------------------
# Mask a Unit - Make it impossible to start the unit

# mask the unit
sudo systemctl mask apache2.service
Created symlink /etc/systemd/system/apache2.service → /dev/null.

# verify its masked
systemctl list-unit-files | grep apache2
apache2.service                              masked          enabled
apache2@.service                             disabled        enabled

# try to start the service - failure
sudo systemctl start apache2.service
Failed to start apache2.service: Unit apache2.service is masked.

# unmask the service
sudo systemctl unmask apache2.service
Removed "/etc/systemd/system/apache2.service".

# verify its unmasked
systemctl list-unit-files | grep apache2
apache2.service                              enabled         enabled
apache2@.service                             disabled        enabled



# ------------------------------------
# Service management
systemctl start <service>               # start service
systemctl is-active <service>           # check service is running
systemctl stop <service>                # stop system daemon
systemctl restart <service>             # restart service
systemctl reload <service>              # reload config without restart - only some services can do this
systemctl reload-or-restart <service>   # reload config in place if possible, otherwise restart
systemctl enable <service>              # autoload daemon on system startup
systemctl is-enabled <service>          # check service is enabled
systemctl disable <service>             # disable from starting automatically
systemctl status <service>              # check status
systemctl is-failed <service>           # check if service failed
```
## Service units

Files that contain instructions for `systemctl` commands:
- System's service file is in `/lib/systemd/system` or `/etc/systemd/system`
- `systemd` looks for autostart files in `/etc/systemd/system/<target>.target.wants`. 


```bash
# system service file location
la -l /lib/systemd/system
total 1528
-rw-r--r-- 1 root root  438 Mar 18  2024 apache2.service
-rw-r--r-- 1 root root  491 Mar 18  2024 apache2@.service
-rw-r--r-- 1 root root  603 Mar 18  2024 apache-htcacheclean.service
-rw-r--r-- 1 root root  612 Mar 18  2024 apache-htcacheclean@.service
...

# systemd looks for autostart files in 

# Ex: apache2 system unit
systemctl cat apache2
# /usr/lib/systemd/system/apache2.service
[Unit]
Description=The Apache HTTP Server
After=network.target remote-fs.target nss-lookup.target
Documentation=https://httpd.apache.org/docs/2.4/

[Service]
Type=forking
Environment=APACHE_STARTED_BY_SYSTEMD=true
ExecStart=/usr/sbin/apachectl start
ExecStop=/usr/sbin/apachectl graceful-stop
ExecReload=/usr/sbin/apachectl graceful
KillMode=mixed
PrivateTmp=true
Restart=on-abort
OOMPolicy=continue

[Install]
WantedBy=multi-user.target

```

### Targets

A target is a special unit file that groups together and organizes other units into a logical state (target state):
- Similar to how other systems use runlevels
- Each system has a default target that it uses during boot
- You can specify the state, instead of each unit individually
- If a unit is part of the target, it can be `WantedBy=<target>`, `RequiredBy=<target>`
- If a unit needs a target to be available, it `Wants=<target>`, `Requires=<target>`, or `After=<target>`

```bash
# get default target
systemctl get-default
graphical.target

# set default target
sudo systemctl set-default graphical.target

# list available targets
systemctl list-unit-files --type=target

# list active targets - targets that systemd has tried to start
systemctl list-units --type=target

# ------------------------------------
# Shortcuts
sudo systemctl rescue               # put system in single-user mode
sudo systemctl halt                 # halt the system
sudo systemctl poweroff             # shutdown the system 
sudo systemctl reboot               # reboot the system (same as sudo reboot)
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