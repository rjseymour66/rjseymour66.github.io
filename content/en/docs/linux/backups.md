---
title: "Backups and archives"
linkTitle: "Backups and archives"
# weight: 1000
# description:
---

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

## tar
Bundles project files into a single output file for easy transfer across the network.
- Preserves folder structure and ownership.
- Does not delete original files or directories.

```bash
tar [OPTIONS] <tarfile-name> [FILES...]
# - : output tar contents to STDOUT (dash in place of archive filename)
# -c: create new archive
# -C: specify directory for extracted tar contents (with -x)
# --exclude="pattern": exclude objects that match pattern
# -f: archive's filename (put last)
# -p: maintain permissions (during create)
# -r: appends file to existing tar archive (can't compress)
# -t: list contents of archive
# -v: verbose
# -x: extract files
# -z: zip or unzip (depends whether -c or -x flag is present)

# all files in pwd
tar -cvf all-files.tar *

# zip files
tar -czvf number-files.tar.gz *.[0-9]*
ls -logh number-files*
-rw-rw-r-- 1 30K Nov  8 14:15 number-files.tar
-rw-rw-r-- 1 477 Nov  8 14:17 number-files.tar.gz

# view files in tar
tar -tvf number-files.tar.gz 
-rw-rw-r-- linuxuser/linuxuser 23 2024-11-08 13:48 file.1
...

# unzip and extract to a dir
tar -zxvf number-files.tar.gz -C extractions/

# create archive on remote
# pipe tar command to remote computer and create archive
tar -czvf - *.[0-9]* | ssh username@10.20.30.40 \
"cat > /home/path/to/dest/remote-tars.tar.gz"

# create archive on remote for only the current filesystem, plus /var and /usr
# might require sudo
tar -czvf - --one-file-system / /var /usr \
| ssh username@10.20.30.40 \
"cat > /home/path/to/dest/workstation-backup-Nov-8.tar.gz"

# extract and maintain permissions requires sudo
sudo tar -xzvf perms.tar.gz 

ls -l
total 4
-rw-rw-r-- 1 linuxuser linuxuser   0 Nov 10 22:54 file1
-rw-rw-r-- 1 linuxuser linuxuser   0 Nov 10 22:54 file2
-rw-rw-r-- 1 newuser   newuser     0 Nov 10 22:54 file3
-rw-rw-r-- 1 linuxuser linuxuser 157 Nov 10 22:58 perms.tar.gz

# unzip tar file with gunzip
gunzip file.tar.gz
```

## split

Breaks archives into multiple smaller files for transfer, and recreates on the remote.

```bash
# split into 100 byte archive files
split -b 100 number-files.tar.gz "number-files.tar.gz."
ls -l *.gz.*
-rw-rw-r-- 1 linuxuser linuxuser 100 Nov  8 14:41 number-files.tar.gz.aa
-rw-rw-r-- 1 linuxuser linuxuser 100 Nov  8 14:41 number-files.tar.gz.ab
-rw-rw-r-- 1 linuxuser linuxuser 100 Nov  8 14:41 number-files.tar.gz.ac
...

# rebuild tar with cat
cat number-files.tar.gz.* > splits/number-recreated.tar.gz
```
## dd

## Schedule backups

### cron

Cron schedules jobs and tasks:
- `cron.[hourly|daily|weekly|monthly|yearly]`: files in these directories run at times specified by dir name.
- `cron.d`: files in this dir have time that defines when the job runs. Add files here to run at specified times.
- `crontab` is overwritten during upgrades, so don't update.
  
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

