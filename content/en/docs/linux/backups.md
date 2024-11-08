---
title: "Backups and archives"
linkTitle: "Backups and archives"
# weight: 1000
# description:
---

## Partition/filesystem backups

- Run `df -h` to figure out which partitions are real and which are pseudo. Pseudo partitions use 0 storage.

## Object backup

## Compression

## tar
Bundles project files into a single output file for easy transfer across the network.
- Preserves folder structure and ownership.
- Does not delete original files or directories.

```bash
# - : output tar contents to STDOUT (dash in place of archive filename)
# -c: create new archive
# -C: specify directory for extracted tar contents (with -x)
# -f: archive's filename (put last)
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

## rsync

## dd