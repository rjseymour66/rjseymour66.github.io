---
title: "Logging services"
weight: 160
---

A logging service tracks system events and stores them in log files in `/var/log`. The most popular linux logging methods are:
- `rsyslog`
- `systemd-journald`

## syslog protocol

Defines a standard message format that specifies the timestamp, type, severity, and details of an event.
- Type of event is a [_facility_ value](https://en.wikipedia.org/wiki/Syslog#Facility), which defines what is generating the message (`kern`, `user`, `daemon`, etc.)
- Each event has a [_severity_](https://en.wikipedia.org/wiki/Syslog#Severity_level) that defines how important the message is to the health of the system


## rsyslog

- "r" stands for "rocket fast"
- Has become the standard logging package for many Linux distros
- Uses all features of original syslog protocol, including config format and logging actions

### Configuration

- `rsyslogd` program to monitor events and log them
- `/etc/rsyslog.conf` config file defines what events to listen for and how to handle them
  - Might use `/etc/rsyslog.d/` directory to store individual files that are part of the `rsyslog.conf` configuration

Config file contains rules that define how program handles syslog event received from system, kernel, or apps in the following format:

```
facility.priority action
```

```bash
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
```

### Making log entries

Use `logger` command-line tool:

```bash
logger [-isd] [-f file] [-p priority] [-t tag] [-u socket] [message]
-i # PID of program that created the log entry
-p # event priority
-t # add tag to help find log message
-f # message as file
-d # send event message over the network
-u # send event message over the network
-s # send to STDERR

# example log message
logger This is a test message to test logger

# view the event message
sudo tail /var/log/syslog 
...
Apr 19 21:30:00 ubuntu22 linuxuser: This is a test message to test logger
```

## systemd-journald

This is a journal utility, _not_ a logging utility. `systemd-journald` uses its own methods to store event messages, apart from syslog.

Great for when you are looking for specific event entries in the journal bc you can filter out events easily with options and matches.

### Configuration

- Reads configuraiton from `/etc/systemd/journal.conf` file
  - No rules, only settings
- `Storage` determines how event messages are stored.
  - `auto` stores messages in `/var/log/journal` if it exists. Otherwise, in `/run/log/journal`
  - `persistent` creates `/var/log/journal` automatically
  - `volatile` stores event messages in the temporary directory
- `ForwardToSyslog` determines if `systemd-journald` forwards event messages to a differnt syslog program for multiple logging layers.

### Viewing logs

Stores files in custom binary file format that works like a db

`journalctl` is the interface to the journal files:

```bash
journalctl [options] [matches]

# available options
-a # display all data fields
-e # jump to end of journal and use pager to display entries
-l # displayall printable data fields
-n N # show the N most recent number of journal entries
-r # reverses order of journal entries in the output

# available matches
Fields
Kernel
PRIORITY=<val>
_UID=<userid>
_HOSTNAME=<host>
_TRANSPORT=<trans>
_UDEV_SYSNAME=<dev>
OBJECT_PID=<pid>

# 
journalctl -r _TRANSPORT=kernel
Apr 19 22:22:35 precision-5540 kernel: IPv6: ADDRCONF(NETDEV_CHANGE): wlp59s0: link becomes ready
Apr 19 22:22:35 precision-5540 kernel: wlp59s0: Limiting TX power to 30 (30 - 0) dBm as advertised >
Apr 19 22:22:35 precision-5540 kernel: wlp59s0: associated
Apr 19 22:22:35 precision-5540 kernel: wlp59s0: RX AssocResp from 90:58:51:d4:02:73 (capab=0x1511 s>
...
```

### tail logs

```bash
tail -f /var/log/syslog
```





















### lastb

View messages from `/var/log/wtmplog` file for user logins:

```bash
sudo lastb
username ssh:notty    127.0.0.1        Thu Apr 18 23:04 - 23:04  (00:00)
username ssh:notty    127.0.0.1        Thu Apr 18 23:04 - 23:04  (00:00)
username ssh:notty    127.0.0.1        Thu Apr 18 23:04 - 23:04  (00:00)
username ssh:notty    127.0.0.1        Thu Apr 18 23:04 - 23:04  (00:00)

btmp begins Thu Apr 18 23:04:27 2024
```