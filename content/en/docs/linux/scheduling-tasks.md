---
title: "Scheduling tasks"
linkTitle: "Scheduling tasks"
# weight: 1000
# description:
---

## at

Lets you specify a time when the linux system runs a script. You have to submit each job that you want to run, you don't schedule recurring jobs:
- The system adds the job to a queue with directions about when the shell should run the job.
- The `atd` daemon runs in the background (starting at boot) and checks the queue for jobs to run
  - `/var/spool/at` contains the job queue
  - There are 26 different job queues available for different priority levels, using lowercase `a` - `z`

Uses `/etc/at.allow` and `/etc/at.deny` files to manage access:
- If `at.allow` exists, only users in that file can use `at`
- If `at.deny` exists, users that are not in this file can use `at`
- If neither exist, then only root can use `at`


<time> accepts following formats:
  - 10:15
  - 10:15 p.m.
  - now, noon, midnight, or teatime (4 p.m.)
  - MMDDYY, MM/DD/YY, DD.MM.YY
  - Jul 4 or Dec 25
  - Now + 25 minutes
  - 10:15 p.m. tomorrow
  - 22:15 tomorrow
  - 10:15 + 7 days
- 

```bash
at [-f <filename>] <time>

# check pending jobs
atq
1	Tue Apr 30 22:20:00 2024 a rseymour

# delete pending job
atrm 1

# verify deleted
atq
```

## cron

Cron schedules jobs and tasks:
- `cron.[hourly|daily|weekly|monthly|yearly]`: files in these directories run at times specified by dir name.
- `cron.d`: files in this dir have time that defines when the job runs. Add files here to run at specified times.
  > Do not add files in `cron.d`--they are overwritten during upgrades.
- `crontab` is overwritten during upgrades, so don't update. Now, admins prefer to make crontab for each user
- User crontabs are stored in `/var/spool/cron`  
- Remove old files with `find ... -delete` cron jobs

```bash
ls -lF /etc/ | grep cron
-rw-r--r-- 1 root root        335 Apr  8  2024 anacrontab
drwxr-xr-x 2 root root       4096 Dec  9 22:28 cron.d/
drwxr-xr-x 2 root root       4096 Dec  9 22:28 cron.daily/
drwxr-xr-x 2 root root       4096 Aug 27 10:26 cron.hourly/
drwxr-xr-x 2 root root       4096 Dec  9 22:28 cron.monthly/
-rw-r--r-- 1 root root       1136 Aug 27 10:26 crontab
drwxr-xr-x 2 root root       4096 Dec  9 22:28 cron.weekly/
drwxr-xr-x 2 root root       4096 Aug 27 10:26 cron.yearly/

# view user crontabs
sudo ls -l /var/spool/cron/crontabs/
total 4
-rw------- 1 root crontab 1282 Dec  9 22:24 root

# list existing cron table
crontab -l
no crontab for linuxuser

# delete existing crontab
crontab -r

# list cron jobs by user
crontab -u <username> -l

# run cron jobs as other users 
sudo crontab -u <username> -e

####### TIME FORMATTING
* * * * * <command-to-be-executed>
- - - - -
| | | | |
| | | | ----- Day of week (0 - 7) (Sunday=0 or 7)
| | | ------- Month (1 - 12)
| | --------- Day of month (1 - 31)
| ----------- Hour (0 - 23)
------------- Minute (0 - 59)

# 10:15am each day
15 10 * * * /full/path/to/program.sh

# 4:15pm every Monday (0 - Sun, 6 - Sat)
15 16 * * 1 /full/path/to/program.sh

# 12 noon first day of each month
00 12 1 * * /full/path/to/program.sh

####### COMMANDS
crontab -l
# Edit this file to introduce tasks to be run by cron.
... 
# m h  dom mon dow   command
54 22 * * 1 /home/linuxuser/cron_echo.sh > cron.out

# add entry
crontab -e
(opens cron table in vi)

47 5 * * * linuxuser /home/linuxuser/scripts/upgrade.sh     # run upgrade.sh as linuxuser at 5:47 AM daily

# backup home directory to /var/backups
crontab -l
...
0 5 * * * tar -zcf /var/backups/home.$(date -I).tar.gz /home/

# cleanup old backups 7 days after modified time
crontab -l
30 5 * * * find /var/backups -name "home.*.tar.gz" -mtime +7 -delete

```

## anacron

Schedule irregular jobs for a machine--such as your laptop--that doesn't run 24/7.
- runs relative to most recent boot time, not absoulte time
- might have to install
- has priority over `cron`
- saves job status info to `/var/spool/anacron/`

```bash
# install, creates /etc/anacrontab
sudo apt install anacron


cat anacrontab
# /etc/anacrontab: configuration file for anacron

# See anacron(8) and anacrontab(5) for details.

SHELL=/bin/sh
HOME=/root
LOGNAME=root

# These replace cron's entries
...
# user-added entries
# interval      mins-after-boot     job-IDer     command
1	15	daily_apt	/home/linuxuser/scripts/upgrade.sh  # run upgrade.sh every day (1) 15 mins after boot
```

## systemd timers

`systemd` timers are completly integrated into `systemd`, so it makes sense to schedule jobs with it:
- There are also more scheduling options

```bash
# list active timers
systemctl list-timers

# list all timers
systemctl list-timers --all
NEXT                            LEFT LAST                              PASSED UNIT                           ACTIVATES                       
Mon 2024-12-09 22:40:00 EST  3min 4s Mon 2024-12-09 22:30:01 EST     6min ago sysstat-collect.timer          sysstat-collect.service
Mon 2024-12-09 22:58:05 EST    21min Sun 2024-12-08 12:43:18 EST            - apt-daily.timer                apt-daily.service
...
```

### Create a systemd timer

Each timer must be attached to a service:

```bash
# 1. Create the service and timer file

```