---
title: "Localization"
weight: 80
description: >
  Linux and localization.
---

## Overview

### Character sets

Defines a standard code used to interpret and display characters in a language:
- UTF: Unicode Tranformation Format, transforms long Unicode vals into 1- or 2-byte codes.
- Unicode: International standard that uses 3-byte code
- ASCII: American Standard Code for Information Interchange, uses 7 bits

### Environment Variables

Programs get locale information from environment variables:

```bash
# lang-country.character-set format
locale

# detailed settings for each env var
locale -ck <env-var>
locale -ck LC_TIME
LC_TIME
abday="Sun;Mon;Tue;Wed;Thu;Fri;Sat"
...
```

## Setting your locale

```bash
# set all LC_* env vars at once
export LANG=en_GB.UTF-8
# display current settings
locale
# list locales
localectl list-locales
# set locale
localectl set-locale LANG=en_GB.UTF-8
```

## Time

Linux uses the date and time to keep track of running processes, to know when to start or stop jobs, and logging. There are two parts:
- Time zone
- Date and time within the time zone

### Time zone

Time zone information is set in the following locations, but you cant edit it directly:
- RHEL: `/etc/localtime`
- Debian: `/etc/timezone`

```bash
# view current TZ template file
ls -al /etc/localtime 
lrwxrwxrwx 1 root root 36 Mar 20 20:53 /etc/localtime -> /usr/share/zoneinfo/America/New_York
# view TZ template dirs and files
ls /usr/share/zoneinfo

# change time zone:
# 1. view the current tz
ls -al /etc/localtime
lrwxrwxrwx 1 root root 35 Mar 29 23:16 /etc/localtime -> /usr/share/zoneinfo/America/Chicago
# check date
date
Fri Mar 29 11:16:51 PM CDT 2024
# 2. rm softlink 
sudo unlink /etc/localtime 
# 3. create new softlink to tz file
sudo ln -s /usr/share/zoneinfo/America/New_York /etc/localtime
# 4. verify
date
Sat Mar 30 12:17:37 AM EDT 2024
```

### Date and time

```bash
# + option lets you specify format
date +"%A, %B, %d, %Y"
Saturday, March, 30, 2024

# view all clock info
timedatectl
               Local time: Sat 2024-03-30 00:25:08 EDT
           Universal time: Sat 2024-03-30 04:25:08 UTC
                 RTC time: Sat 2024-03-30 04:25:08          # hardware clock
                Time zone: America/New_York (EDT, -0400)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no

```

### Network Time Protocol

You can't change the time if you use NTP, you have to point your server to an appropriate network time server.

There are three common implementations of NTP:
- `ntpd`: Legacy software that uses Simple Network Time Protocol (SNTP) to connect to network time server
- `chrony`: Improved version of `ntpd` that has better security
  - Config settings are in `/etc/systemd/chrony.conf`
- `timesyncd`: Part of systemd startup utility that provides NTP services.
  - Config settings are in `/etc/systemd/timesyncd.conf`
  - Config settings are in `/etc/systemd/timesyncd.conf.d`

## Watching system time

### time

Displays the amount of time it takes for a program to run:

```bash
time timedatectl
               Local time: Sat 2024-03-30 00:33:46 EDT
           ...

real	0m0.099s    # elapsed time between start and end of program
user	0m0.000s    # user CPU time the program took
sys	0m0.010s        # system CPU time the program took
```