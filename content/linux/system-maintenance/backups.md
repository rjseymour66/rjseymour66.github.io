+++
title = 'Backups'
date = '2025-09-07T18:49:31-04:00'
weight = 90
draft = false
+++


Follow the Backup Rule of Three: always keep three copies of your data:

- Two local copies on different media.
- One remote copy in case of disaster.

A backup is sometimes called an archive. An archive is a group of files with associated metadata that can be restored if the data becomes corrupted. When planning a backup strategy, consider the backup type, compression method, and tools best suited to your needs.

## Understanding backup types

_System image_
: A clone, a copy of the OS binaries, config files, and whatever you need to boot.

_Full_
: Copy of all data, ignoring its modification date. Quickly restores system data, but takes a long time to create the backup.

_Incremental_
: Copy of data modified since the last backup, compared by timestamp. The initial backup is a full copy. Each subsequent backup captures only what changed since the previous one — for example, B captures changes since A, C captures changes since B, and so on. Quick to create, but can take longer to restore.

_Differential_
: Copy of all data that changed since the last full backup. The initial backup is a full copy. Each subsequent backup captures everything that changed since that initial backup — for example, B captures changes since A, C also captures all changes since A, and so on. A good balance between full and incremental backups.

_Snapshot_
: A hybrid approach. The first backup creates a full, usually read-only, copy on the backup media. Hard links then build a reference table connecting the backup data to the original. On each subsequent backup, only modified files are copied and the reference table is updated. This lets you restore from any point in time and is efficient in both storage and processing.

  `rsync` takes the snapshot approach.

_Snapshot clone_
: After a snapshot is created, it is cloned. Useful in high I/O environments. It is modifiable and mountable, so you can use it as disaster recovery.


## Backup files

Linux stores default backup files in `/var/backups`. Common files include:

- `passwd.bak`
- `group.bak`
- `shadow.bak`
- `fstab.bak`

## Partition/filesystem backups

Run `df -h` to identify real partitions versus pseudo partitions. Pseudo partitions consume no storage.

## Backup config files

This script backs up configuration files by comparing each file in `/etc` against its counterpart in `/var/backups/`. If the files are not identical, it copies the `/etc` version to `/var/backups/`. Add the filenames to back up as arguments to the `for` loop:

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

`rsync` syncs files over the network. It copies new files and updated files to the destination, skipping anything already up to date. Use it for large file transfers and incremental backups.

The following examples demonstrate common options and transfer patterns:

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

```bash
tar [OPTIONS...] [FILENAME]...
-d # compare tar archive file members with external files
-t # display tar archive file's contents (members)
-W # verify each file as it is processed. Can't use with compression.
```

### Full and incremental backups

`tar` organizes backups in levels. Level 0 is a full backup containing all files. Level 1 is the first incremental backup, level 2 is the second, and so on.



To create a full backup and add an incremental backup:

1. Create a full archive. `tar` generates a `.snar` snapshot file that records timestamp metadata for tracking changes.

    ```bash
    tar -g FullArchive.snar -Jcvf Project42.txz Project4?.txt
    ...
    ```

2. Verify the archive and snapshot file were created.

    ```bash
    ls FullArchive.snar Project42.txz
    FullArchive.snar  Project42.txz
    ```

3. Modify a file to simulate a change.

    ```bash
    echo 'Answer to everything' >> Project42.txt
    ```

4. Create an incremental backup. `tar` compares timestamps against the `.snar` file and archives only the files that changed since the last backup.

    ```bash
    tar -g FullArchive.snar -Jcvf Project42_Inc.txz Project4?.txt
    Project42.txt
    ...
    ```
### Restore

Restoring from a tar archive follows the same pattern as creating one. Replace the `-c` flag with `-x` to extract:

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
## Schedule backups

### cron

Cron schedules recurring jobs and tasks. It reads from several directories and files, each serving a different purpose:

- `cron.[hourly|daily|weekly|monthly|yearly]` — files in these directories run at the interval indicated by the directory name.
- `cron.d` — files here define their own schedule. Add files to this directory to run jobs at specific times. Files in `cron.d` are overwritten during upgrades.
- `crontab` — the system crontab. Do not edit this file directly, as it is overwritten during upgrades.
- `/var/spool/cron` — stores per-user crontab files.

The following examples show time formatting and common crontab commands:

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

`anacron` schedules irregular jobs on machines that do not run 24/7, such as a laptop. Unlike `cron`, it schedules jobs relative to the most recent boot time rather than absolute time. It takes priority over `cron` and may need to be installed separately. Job status information is saved to `/var/spool/anacron/`.

The following example shows the anacrontab format and a sample entry:

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


### at

`at` runs a script at a specified time. Unlike `cron`, it does not schedule recurring jobs. Each job must be submitted individually. When you submit a job, the system adds it to a queue with instructions for when to run it.

The `atd` daemon starts at boot and monitors the queue, executing jobs at the scheduled time. The queue is stored in `/var/spool/at`, and 26 priority levels are available using the letters `a` through `z`.

Access is controlled by `/etc/at.allow` and `/etc/at.deny`. If neither file exists, only root can submit jobs.

- If `/etc/at.allow` exists, only users listed in that file can submit jobs.
- If `/etc/at.deny` exists, any user not listed in that file can submit jobs.

The `<time>` argument accepts several formats:

- `10:15` or `10:15 p.m.`
- `now`, `noon`, `midnight`, or `teatime` (4 p.m.)
- `MMDDYY`, `MM/DD/YY`, or `DD.MM.YY`
- `Jul 4` or `Dec 25`
- `now + 25 minutes`
- `10:15 p.m. tomorrow` or `22:15 tomorrow`
- `10:15 + 7 days`

The following examples show how to check and manage the job queue:

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
