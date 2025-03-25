---
title: "Backups"
# linkTitle: "Backups"
# weight: 1000
# description:
---

Backup Rule of Three: always have 3 copies:
- 1 remote system in case of disaster
- 2 local copies, on different media

A backup is sometimes called an _archive_. An archive is a group of files with associated metadata. It is a copy of data that can be restored sometime in the future if the data becomes corrupted. You need to consider the following:
- Backup type
- Compression methods
- Utilities that will help the most

## Understanding backup types

_System image_
: A clone, a copy of the OS binaries, config files, and whatever you need to boot.

_Full_
: Copy of all data, ignoring its modification date. Quickly restores system data, but takes a long time to create the backup.

_Incremental_
: Copy of data that has been modified since the last backup operation, by comparing timestamps. This method is quick, but might take a long time to actually restore.

_Differential_
: Copy of all data that changed since last full backup. Good balance between full and incremental backup.

_Snapshot_
: Hybrid approach - a full (usually read-only) copy of data is made to backup media. Then pointers (ex: hard links) are employed to create a reference table linking the backup data with the original data. During next backup, only modified files are copied to backup media, and the pointer reference table is copied and updated.
 
  You can go back to any point in time (restore point) and restore the data from there. Very efficient and takes less space and processing power.

  `rsync` uses the snapshot approach.

_Snapshot clone_
: After a snapshot is created, it is cloned. Useful in high I/O environments. It is modifiable and mountable, so you can use it as disaster recovery.


## Backup files

`/var/backups`:
- `passwd.bak`
- `group.bak`
- `shadow.bak`
- `fstab.bak`

## Types of backups

A _differential backup_ takes an initial backup, and all subsequent backups are the diff between the current and initial backups. For example, backup A is the initial backup, then backup B is the diff between A and B, backup C is the diff between A and C, and so on.

An _incremental backup_ takes an initial backup, and all subsequent backups are the diff between the previous backup. For example, backup A is the initial backup, then B is the diff between A and B, backup C is the diff between B and C, and so on. 

## Partition/filesystem backups

- Run `df -h` to figure out which partitions are real and which are pseudo. Pseudo partitions use 0 storage.

## Object backup

## Compression

## Backup config files

This script backs up any configuration files provided as arguments to the `for` loop. It compares the files in `/var/backups/` with the files in `/etc`. If the files are not identical, they are copied to `/var/backups/`:

```bash
#!/bin/sh

cd /var/backups || exit 0

for FILE in passwd group shadow gshadow; do
    test -f /etc/$FILE              || continue         # if no backup file, next loop arg
    # -s suppresses output
    cmp -s $FILE.bak /etc/$FILE     && continue         # if files match, next loop arg
    cp -p /etc/$FILE $FILE.bak && chmod 600 $FILE.bak   # overwrite old file w/new, root perms
done
```

## rsync

Remote sync. Copies large files quickly over the network. It copies file updates, and files that do not exist in the destination directory.

```bash
rsync [OPTION]... SOURCE DEST
-a # archive mode, equivalent to -rlptgoD (dir tree backups)
-D # retain Device and special files
-g # retain file group
-h # human-readable numeric output
-l # copy symbolic links
-o # retain file owner
-p # retain file perms
--progress # display progress of file copy
-r # recursive
--stats # display file transfer stats
-t # retain file's modification time
-v # verbose

# copy to /home dir on remote
rsync -av Downloads/filename.ext linuxuser@ubuntu-24:/home/linuxuser

# 1. send all files in pwd to remote dir
rsync -av * linuxuser@ubuntu-24:syncdirectory
sending incremental file list
file1
file2
file3
file4
file5

sent 317 bytes  received 111 bytes  856.00 bytes/sec
total size is 0  speedup is 0.00

# 2. Create new file and edit file3, rsync sends diff
rsync -av * linuxuser@ubuntu-24:syncdirectory
sending incremental file list
file3
newfile

sent 257 bytes  received 54 bytes  207.33 bytes/sec
total size is 23  speedup is 0.07

```

## tar full and incremental backups

`tar` views full and incremental backups in levels:
- level 0 includes all files
- level 1 is first incremental backup
- level 2 is second incremental backup, etc...

```bash
tar [OPTIONS...] [FILENAME]...
-d # compare tar archive file members with external files 
-t # display tar archive file's contents (members)
-W # verify each file as it is processed. Can't use with compression.

# 1. creates snapshot .snar file w timestamp metadata to create backups
tar -g FullArchive.snar -Jcvf Project42.txz Project4?.txt
Project42.txt
Project43.txt
Project44.txt
Project45.txt
Project46.txt

# 2. verify created
ls FullArchive.snar Project42.txz
FullArchive.snar  Project42.txz

# 3. Update file
echo 'Answer to everything' >> Project42.txt 

# 4. create incremental backup. Project42_Inc.txz contains only Project42.txt
#    because its the only file that was modified since the previous backup.
tar -g FullArchive.snar -Jcvf Project42_Inc.txz Project4?.txt
Project42.txt

# view tarball files/members
tar -tf Project4x.tar.gz 
Project42.txt
Project43.txt
Project44.txt
Project45.txt
Project46.txt

# compare archive files against current files
$ tar -df Project4x.tar.gz 
Project42.txt: Mod time differs
Project42.txt: Size differs

# verify backup after archive is created. can't compress, must
# be in next step.
tar -Wcvf ProjectVerify.tar Project4?.txt
Project42.txt
...
Verify Project42.txt
...
```
## tar restore

Basically same as compress command, but sub the `-c` for `-x`:

```bash
tar [OPTIONS...] [FILENAME]...
-x # extract files from tarball or archive and place in cwd
-z # decompresses with gunzip
-j # decompresses with bunzip2
-J # decompresses with unxz

# extract gzip tarball (tarball is not removed)
tar -zxvf Project4x.tar.gz 
Project42.txt
Project43.txt
Project44.txt
Project45.txt
Project46.txt
```
## dd

## Schedule backups

### cron

Cron schedules jobs and tasks:
- `cron.[hourly|daily|weekly|monthly|yearly]`: files in these directories run at times specified by dir name.
- `cron.d`: files in this dir have time that defines when the job runs. Add files here to run at specified times.
- `crontab` is overwritten during upgrades, so don't update.
- User crontabs are stored in `/var/spool/cron`  
- Remove old files with `find ... -delete` cron jobs
  
  > Do not add files in `cron.d`--they are overwritten during upgrades.

```bash
ls -lF | grep cron
-rw-r--r-- 1 root root        419 Nov 18 01:39 anacrontab
drwxr-xr-x 2 root root       4096 Nov 18 01:32 cron.d/
drwxr-xr-x 2 root root       4096 Nov 18 01:29 cron.daily/
drwxr-xr-x 2 root root       4096 Aug 27 14:26 cron.hourly/
drwxr-xr-x 2 root root       4096 Nov 18 01:29 cron.monthly/
-rw-r--r-- 1 root root       1136 Aug 27 14:26 crontab
drwxr-xr-x 2 root root       4096 Nov 18 01:29 cron.weekly/
drwxr-xr-x 2 root root       4096 Aug 27 14:26 cron.yearly/

####### TIME FORMATTING
min hour dayofmonth month dayofweek command

# 10:15am each day
15 10 * * * /full/path/to/program.sh

# 4:15pm every Monday (0 - Sun, 6 - Sat)
15 16 * * 1 /full/path/to/program.sh

# 12 noon first day of each month
00 12 1 * * /full/path/to/program.sh

# list existing cron table
crontab -l
no crontab for linuxuser

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

# delete user's crontab file
crontab -r
```

### anacron

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
