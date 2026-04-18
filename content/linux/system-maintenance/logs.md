+++
title = 'Logs'
date = '2025-09-07T18:49:22-04:00'
weight = 80
draft = false
+++


## syslog

Logging is now handled by `journald`, but historically it was done by `syslogd`. The `syslogd` daemon collected log messages from system data sources and sent them to the `/dev/log` pseudo device. It then read the metadata and headers to route each message to the correct plain text log file in `/var/log/`.

### Example rules

The following example shows the default rsyslog distribution rules:

```bash
# view logging distribution rules
cat /etc/rsyslog.d/50-default.conf

# routes log messages by facility
auth,authpriv.*             /var/log/auth.log       # auth events
*.*;auth,authpriv.none      -/var/log/syslog         # everything except auth
#cron.*                     /var/log/cron.log
kern.*                      -/var/log/kern.log
...
```

### Syntax reference

```bash
kern.crit                       # kernel events at critical severity or higher
kern.=crit                      # kernel events at critical severity only
*.emerg                         # all facilities at emergency severity
auth,authpriv.*                 # separate multiple facilities with a comma
authpriv.none -/var/log/syslog  # .none excludes a facility; - skips sync after write
*.emerg :omusrmsg:*             # send all emergency events to all logged-in users
```

### log rotation

Log rotation prevents logs from consuming too much disk space. Three locations control how rotation works:

| Location | Description |
| :--- | :--- |
| `/etc/logrotate.conf` | Main configuration file. Provides defaults for any service not configured in `/etc/logrotate.d/`. |
| `/etc/logrotate.d/` | Per-service configuration files. Merged with `logrotate.conf` to build the full configuration. |
| `/etc/cron.daily/logrotate` | The cron script that triggers rotation daily. |



The following example shows the logrotate configuration for `apt` and how rotated files are named:

```bash
# uncompressed files are the most recent, in case they are needed immediately
ls /var/log/ | grep auth
auth.log
auth.log.1
auth.log.2.gz
auth.log.3.gz

# per-service log rotation configs
ll /etc/logrotate.d/
total 76
drwxr-xr-x.   2 root root  4096 Nov 23 21:11 ./
drwxr-xr-x. 116 root root 12288 Nov 24 22:28 ../
-rw-r--r--.   1 root root   120 Feb  5  2024 alternatives
-rw-r--r--.   1 root root   397 Mar 18  2024 apache2
-rw-r--r--.   1 root root   126 Apr 22  2022 apport
-rw-r--r--.   1 root root   173 Mar 22  2024 apt
...

cat /etc/logrotate.d/apt
/var/log/apt/term.log {
  rotate 12     # rotate 12 times, then delete
  monthly       # rotate once per month
  compress      # compress immediately
  missingok
  notifempty
}
```

To view the main logrotate configuration file:

```bash
cat /etc/logrotate.conf
```

### Example settings

The following reference shows all available options for a logrotate configuration block:

```bash
/var/log/filename.log
{
    size 1k                     # rotate if log file is >= 1k (bytes by default; supports k, M, G)
    weekly                      # rotate weekly, monthly, or yearly (use one at a time)
    monthly
    yearly
    create 700 <user> <group>   # permissions for the new log file after rotation
    extension <ext>             # extension to use for rotated files
    rotate 4                    # rotate 4 times before deleting; keeps 4 most recent files
    mail user@example.com       # email logs before removing (requires a mail transfer agent)
    dateext                     # add date suffix in YYYYMMDD format; daily rotation only
    compress                    # compress rotated files
    compresscmd /bin/bzip2      # specify the compression program
    compressext .bz2            # extension for compressed files
    delaycompress               # delay compression until the next rotation cycle
    notifempty                  # do not rotate if the log file is empty
    missingok                   # do not generate an error if the log file is missing
    prerotate                   # run a custom script before rotation
      /path/to/script.sh
    postrotate                  # run a custom script after rotation
      /path/to/script.sh
    maxage 100                  # remove rotated files older than X days
    copytruncate                # copy the active log before truncating; prevents lost writes
}
```
### Manually create log file and rotate

Manually testing a logrotate configuration lets you verify that rotation, compression, and file naming work as expected before relying on the daily cron job. To test a logrotate configuration manually:

1. Create a test configuration file.

    ```bash
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
    ```

2. Write to the log file to give logrotate something to rotate.

    ```bash
    echo "Write anything to logtest so it will rotate" > /var/log/logtest
    ```

3. Trigger rotation manually. Logrotate is configured to run daily, so call it directly.

    ```bash
    logrotate /etc/logrotate.conf
    ```

4. Verify the rotation occurred.

    ```bash
    ls -l /var/log/ | grep logtest
    -rw-r--r--  1 root      root                 0 Dec 17 10:27 logtest
    -rw-r--r--  1 root      root               103 Dec 17 10:22 logtest-20241217.yay
    ```

### syslogd files

The following table describes common `syslogd` log files and the events each one records:

| File       | Events                      |
| :--------- | :-------------------------- |
| `auth.log` | System auth and security    |
| `boot.log` | Boot-related                |
| `dmesg`    | Kernel-ring device driver   |
| `dpkg.log` | Software package management |
| `kern.log` | Kernel events               |
| `syslog`   | All logs                    |
| `wtmp`     | User sessions               |

## journald

`journald` is `systemd`'s logging implementation. It stores log data in binary format rather than plain text, and exposes logs through the `journalctl` command line utility.

### Configuration

To view the journald configuration file:

```bash
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
```

### Persist between reboots

By default, journald writes logs to `/run/log/journal`, which is destroyed on shutdown. To persist logs across reboots, create `/var/log/journal` so journald writes there instead:

```bash
mkdir -p /var/log/journal
systemd-tmpfiles --create --prefix /var/log/journal
```

### journalctl

`journalctl` retrieves journald logs with filters. The following reference covers common options, matches, and usage examples:

```bash
journalctl [options] [matches]
-a          # display all data fields
-e          # jump to end of journal and use pager to display entries
-f          # follow logs as they occur
-l          # display all printable data fields
-n N        # show the N most recent journal entries
-p <level>  # filter by priority: debug, info, notice, warning, err, crit, alert, emerg
-r          # reverse order of journal entries
--since     # show logs starting at this time
--until     # show logs ending at this time

# available matches
PRIORITY=<val>
_UID=<userid>
_HOSTNAME=<host>
_TRANSPORT=<trans>
_UDEV_SYSNAME=<device>
```

#### View recent entries

To quickly check what the system has been doing, view the most recent log entries:

```bash
journalctl -n 5
Nov 26 20:39:31 ubuntu-24 systemd[1]: phpsessionclean.service: Deactivated successfully.
Nov 26 20:39:31 ubuntu-24 systemd[1]: Finished phpsessionclean.service - Clean php session files.
Nov 26 20:40:14 ubuntu-24 systemd[1]: Starting sysstat-collect.service - system activity accounting tool...
Nov 26 20:40:14 ubuntu-24 systemd[1]: sysstat-collect.service: Deactivated successfully.
Nov 26 20:40:14 ubuntu-24 systemd[1]: Finished sysstat-collect.service - system activity accounting tool.
```

#### Filter by priority

To investigate alerts or errors, filter logs by severity level. This is useful when checking for failed sudo attempts or other security events:

```bash
journalctl -p alert
Nov 24 22:24:15 ubuntu-24 sudo[1744]: otheruser : user NOT in sudoers ; TTY=pts/0 ; PWD=/home/linuxuser/groups-least-privs ; USER=root ; COMMAND=/usr/sbin/groupadd app-data-group
Nov 24 23:24:09 ubuntu-24 sudo[2211]: otheruser : user NOT in sudoers ; TTY=pts/0 ; PWD=/home/otheruser ; USER=root ; COMMAND=/usr/bin/netstat -npl
```

#### Follow logs in real time

To watch logs as they are written, useful when reproducing an issue or monitoring a service restart:

```bash
journalctl -f
```

#### Filter by date and time

To narrow down logs to a specific window, useful when investigating an incident at a known time:

```bash
journalctl --since 03:40:00 --until 03:50:00
Nov 26 03:40:03 ubuntu-24 systemd[1]: Starting sysstat-collect.service - system activity accounting tool...
Nov 26 03:40:03 ubuntu-24 systemd[1]: fwupd.service: Deactivated successfully.
Nov 26 03:40:03 ubuntu-24 systemd[1]: fwupd.service: Consumed 1.232s CPU time.
Nov 26 03:40:03 ubuntu-24 systemd[1]: sysstat-collect.service: Deactivated successfully.
Nov 26 03:40:03 ubuntu-24 systemd[1]: Finished sysstat-collect.service - system activity accounting tool.
```

## Searching logs

Linux provides three tools for searching and processing log files. Each serves a different purpose:

- `grep` finds lines that match a pattern. Use it to locate specific events quickly.
- `awk` processes log lines and applies conditions, counts, or extractions. Use it when you need to compute or summarize data across multiple lines.
- `sed` transforms text streams with substitutions and filters. Use it to reformat log output or strip unwanted content before saving or further processing.

### grep

`grep` searches log files for matching patterns. It is the fastest way to find specific events in a log file. The following examples search `auth.log` for authentication failures:

```bash
# search for authentication failures
cat /var/log/auth.log | grep 'Authentication failure'

# return line before and after match
cat /var/log/auth.log | grep -B 1 -A 1 failure
```

### awk

`awk` processes log files line by line and applies conditions or transformations. Use it when you need to count, extract, or compute values from log output. The following example counts warning messages in a log file:

```bash
# return number of warning messages in logs
cat error.log | awk '$3 ~/[Warning]/ | wc
```


### sed

`sed` makes substitutions and applies filters to text streams. Redirect the output to a file to save the result. The following examples demonstrate common log processing patterns:

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