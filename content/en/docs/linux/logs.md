---
title: "Logging"
linkTitle: "Logs"
# weight: 1000
# description:
---

## syslog

Now, logging is done with `journald`, but historically it was done with `syslogd`:
- `syslogd` collected log messages from system data sources and sent it to the `/dev/log` pseudo device
- then, it read metadata and headers to direct the log messages to the correct plain text log in `/var/log/`

```bash
# logging distribution in rsyslog.d
cat /etc/rsyslog.d/50-default.conf 
#  Default rules for rsyslog.
#
#			For more information see rsyslog.conf(5) and /etc/rsyslog.conf

#
# First some standard log files.  Log by facility.
#
auth,authpriv.*			/var/log/auth.log
*.*;auth,authpriv.none		-/var/log/syslog
#cron.*				/var/log/cron.log
#daemon.*			-/var/log/daemon.log
kern.*				-/var/log/kern.log
...

############### Explained:
# log kernel events critical or higher
kern.crit

# log only critical kernel events
kern.=crit

# log all events with emergency serverity
*.emerg

# separate multiple facility types with comma
auth,authpriv.*

# (.none) all events except security
# (-) do not sync file after each write to increase perfomance
authpriv.none   -/var/log/syslog

# send all emergency events to all users on system
*.emerg         :omusrmsg:*
# ---
```

### log rotation

Rotate your logs so they don't consume too much disk space:

- `/etc/logrotate.conf`: logrotation config file, provides defaults if not defined for each service in `/etc/logroate.d/`
- `/etc/logrotate.d/`: contains all logrotate files--these are merged to create the full configuration
- `/etc/cron.daily/logrotate`: actual logrotate script



```bash
# config file for logrotation
cat /etc/logrotate.conf 

# ---
# uncompressed files are most recent in case they are needed
ls /var/log/ | grep auth
auth.log
auth.log.1
auth.log.2.gz
auth.log.3.gz

# service or application log rotation configs
ll /etc/logrotate.d/
total 76
drwxr-xr-x.   2 root root  4096 Nov 23 21:11 ./
drwxr-xr-x. 116 root root 12288 Nov 24 22:28 ../
-rw-r--r--.   1 root root   120 Feb  5  2024 alternatives
-rw-r--r--.   1 root root   397 Mar 18  2024 apache2
-rw-r--r--.   1 root root   126 Apr 22  2022 apport
-rw-r--r--.   1 root root   173 Mar 22  2024 apt
...

# Example
cat /etc/logrotate.d/apt 
/var/log/apt/term.log {
  rotate 12     # rotate 12 times, then delete
  monthly       # rotate once per month
  compress      # compress immediately
  missingok
  notifempty
}
...
```

### Example settings

```bash
# size
/var/log/filename.log
{
    size 1k                     # Run logrotate if log file >= 1k (bytes default, k, M, G)
    weekly                      # rotate logs either weekly, monthly, or yearly (not all at once)
    monthly
    yearly
    create 700 <user> <group>   # Permissions for log file
    extension <ext>             # 
    rotate 4                    # How many times a log is rotated before old logs are rm'd
                                # keep 4 most recent logfiles
    mail user@example.com       # Logs emailed and removed (req's mail transfer agent)
    dateext                     # Add date to rotated log files in YYYYMMDD
                                # ONLY daily rotation bc of date format
    compress                    # Compress rotated files
    compresscmd /bin/bzip2      # Specify compression algo
    compressext .bz2            # Ext for compressed files
    delaycompress               # Delay compression if you need to do something else
    notifempty                  # Do not rotate if empty
    missingok                   # Do not generate error if log file is not present
    prerotate                  # Run custom scripts before rotation
      /path/to/script.sh
    postrotate                  # Run custom scripts after rotation
      /path/to/script.sh
    maxage 100                  # Remove files after X days
    copytruncate                # Copy active log file for rotation, truncate active log file
                                # makes sure logs aren't lost if rotated during writing
}
```
### Manually create log file and rotate

```bash
# create test config
cat /etc/logrotate.d/logtest 
/var/log/logtest {
        rotate 1
        compress
        compresscmd /usr/bin/bzip2
        compressext .yay
        dateext
        maxage 30
        size 2         # rotate after 2 bytes
        notifempty
        missingok
        copytruncate
}

# write to log file
echo "Write anything to logtest so it will rotate" > /var/log/logtest

# manually call logrotate bc its configured to run daily
logrotate /etc/logrotate.conf

# verify rotation
ls -l /var/log/ | grep logtest
-rw-r--r--  1 root      root                 0 Dec 17 10:27 logtest
-rw-r--r--  1 root      root               103 Dec 17 10:22 logtest-20241217.yay

```

### syslogd files

Describes the common `syslogd` log files and the types of events they log:

| File | Events |
|:---|:---|
| `auth.log` | System auth and security |
| `boot.log` | Boot-related |
| `dmesg` | Kernel-ring device driver |
| `dpkg.log` | Software package management |
| `kern.log` | Kernel events |
| `syslog` | All logs |
| `wtmp` | User sessions |

## journald

`journald` is `systemd`'s logging implementation
- `journalctl` command line utility
- stores log data in binary format, not plain text

### Configuration

```bash
# configure journald
cat /etc/systemd/journald.conf 
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it under the
...

[Journal]
#Storage=auto
...
#SystemMaxUse=
...
#RuntimeMaxUse=
...

# create persistent journald log file (/run/log/journal is destroyed during shutdown)
mkdir -p /var/log/journal   # if doesn't already exist
systemd-tmpfiles --create --prefix /var/log/journal
```

### journalctl

Retrieve journald logs with filters:

```bash
journalctl [options] [matches]
-a # display all data fields
-e # jump to end of journal and use pager to display entries
-f # follow - watch logs as they occur
-l # displayall printable data fields
-n N # show the N most recent number of journal entries
-p <priority # debug, info, notice, warning, err, crit, alear, emerg (syslog levels)
-r # reverses order of journal entries in the output
--since # logs beginning at this time
--until # logs ending at this time

# available matches
Fields
Kernel
PRIORITY=<val>
_UID=<userid>
_HOSTNAME=<host>
_TRANSPORT=<trans>
_UDEV_SYSNAME=<de

# 20 most recent logs
journalctl -n 5
Nov 26 20:39:31 ubuntu-24 systemd[1]: phpsessionclean.service: Deactivated successfully.
Nov 26 20:39:31 ubuntu-24 systemd[1]: Finished phpsessionclean.service - Clean php session files.
Nov 26 20:40:14 ubuntu-24 systemd[1]: Starting sysstat-collect.service - system activity accounting tool...
Nov 26 20:40:14 ubuntu-24 systemd[1]: sysstat-collect.service: Deactivated successfully.
Nov 26 20:40:14 ubuntu-24 systemd[1]: Finished sysstat-collect.service - system activity accounting tool.

# by priority
journalctl -p alert
Nov 24 22:24:15 ubuntu-24 sudo[1744]: otheruser : user NOT in sudoers ; TTY=pts/0 ; PWD=/home/linuxuser/groups-least-privs ; USER=root ; COMMAND=/usr/sbin/groupadd app-data-group
Nov 24 23:24:09 ubuntu-24 sudo[2211]: otheruser : user NOT in sudoers ; TTY=pts/0 ; PWD=/home/otheruser ; USER=root ; COMMAND=/usr/bin/netstat -npl

# view logs as they happen
journalctl -f

# filter by date and time
journalctl --since 03:40:00 --until 03:50:00
Nov 26 03:40:03 ubuntu-24 systemd[1]: Starting sysstat-collect.service - system activity accounting tool...
Nov 26 03:40:03 ubuntu-24 systemd[1]: fwupd.service: Deactivated successfully.
Nov 26 03:40:03 ubuntu-24 systemd[1]: fwupd.service: Consumed 1.232s CPU time.
Nov 26 03:40:03 ubuntu-24 systemd[1]: sysstat-collect.service: Deactivated successfully.
Nov 26 03:40:03 ubuntu-24 systemd[1]: Finished sysstat-collect.service - system activity accounting tool.

```

## Searching logs

### grep

```bash
# search for authentication failures
cat /var/log/auth.log | grep 'Authentication failure'

# return line before and after match
cat /var/log/auth.log | grep -B 1 -A 1 failure
```

### awk

```bash
# return number of warning messages in logs
cat error.log | awk '$3 ~/[Warning]/ | wc
```


### sed

Can make complex substitutions on text streams
- redirect to a file to save output

```bash
# p - print

# return number of warning messages in logs
cat error.log | awk '$3 ~/[Warning]/ | sed -n '$=''

# basic substituion
echo 'hello world' | sed 's/world/planet earth/'
hello planet earth

# remove numbers at beginning of lines and save in new file
# replace a number at the beginning of a line with nothing
sed "s/^ *[0-9]* //g" numbers.txt > new-numbers.txt


# list only directories (output that begins with 'd')
ll /var/log/ | sed -n '/^d/ p'
drwxrwxr-x. 12 root      syslog            4096 Nov 26 03:07 ./
drwxr-xr-x. 14 root      root              4096 Nov 20 01:02 ../
drwxr-x---.  2 root      adm               4096 Nov 27 12:57 apache2/
drwxr-xr-x.  2 root      root              4096 Nov 27 13:12 apt/
drwxr-xr-x.  2 root      root              4096 Aug 21 16:55 dist-upgrade/
drwxrwx---.  4 root      adm               4096 Nov  2 17:15 installer/
drwxr-sr-x+  3 root      systemd-journal   4096 Nov  2 17:17 journal/
drwxr-xr-x.  2 landscape landscape         4096 Aug 27 14:26 landscape/
drwx------.  2 root      root              4096 Nov 24 15:54 letsencrypt/
drwx------.  2 root      root              4096 Aug 27 14:21 private/
drwxr-xr-x.  2 root      root              4096 Nov 27 12:57 sysstat/
drwxr-x---.  2 root      adm               4096 Nov  2 17:18 unattended-upgrades/
```