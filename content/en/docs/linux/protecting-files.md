---
title: "Protecting files"
weight: 110
description: >
  "Protecting files"
---

A backup is sometimes called an _archive_, and it is a copy of data that can be restored sometime in the future if the data becomes corrupted. You need to consider the following:
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

## Compression methods

The following compression methods all provide lossless compression: 
_gzip_
: Replaced old `compress` program. Replaces original file with a compressed version and .gz file extension. `gunzip` to reverse the operation.

_bzip2_
: Compressed Linux kernel until 2013. Replaces original file.

_xz_
: Popular with Linux admins. Compresses Linux Kernel since 2013. Replaces original file

_zip_
: Can operate on multiple files, packs them together in 'folder' or 'archive'. Does not replace original file--places copy of the file into an archive.



```bash
# use the -<n> option with all but zip to control compression.
# -1 is fast but lowest compression
# -9 is slow but highest compression
# -6 is default 
$ ls -lh wtmp?
-rw-rw-r-- 1 137K Apr  4 22:08 wtmp1
-rw-rw-r-- 1 137K Apr  4 22:08 wtmp2
-rw-rw-r-- 1 137K Apr  4 22:08 wtmp3
-rw-rw-r-- 1 137K Apr  4 22:08 wtmp4

# compress each file
gzip wtmp1      # gunzip <compressed-file> to reverse
bzip2 wtmp2     # bunzip2 <compressed-file> to reverse
xz wtmp3        # unxz <compressed-file> to reverse 
zip wtmp4.zip wtmp4     # unzip <compressed-file> to reverse
  adding: wtmp4 (deflated 96%)

# xz compresses the best
ls -lh wtmp?.*
-rw-rw-r-- 1 5.6K Apr  4 22:08 wtmp1.gz
-rw-rw-r-- 1 4.4K Apr  4 22:08 wtmp2.bz2
-rw-rw-r-- 1 3.9K Apr  4 22:08 wtmp3.xz
-rw-rw-r-- 1 5.7K Apr  4 22:18 wtmp4.zip

# zip did not replace the file, still there
$ ls wtmp?
wtmp4
```

## Archive and restore utilities

### cpio

_copy in and copy out_, gathers together file copies and stores them in an archive file. Good for system image and full backup bc it maintains each file's absolute directory path/reference.

```bash
# Pipe list of files into cpio
ls file-list | cpio -ov > output.cpio 
-I # specifies the archive file to use
-i # extract, copies files from archive or displays files within the archive
--no-absolute-filenames # only relative path names allowed
-o # copy-out mode, creates archive by copying files into it
-t # displays list of files within the archive
-v # verbose

# view files
ls
Project42.txt  Project43.txt  Project44.txt  Project45.txt  Project46.txt
# pipe files into cpio
ls Project4?.txt | cpio -ov > Project4x.cpio
Project42.txt
Project43.txt
Project44.txt
Project45.txt
Project46.txt
1 block
# view files
ls Project4?.*
Project42.txt  Project43.txt  Project44.txt  Project45.txt  Project46.txt  Project4x.cpio

# create archive of all files owned by <username>
find / -user <username> | cpio -ov > archive.cpio

# list archive contents
cpio -itvI Project4x.cpio 
-rw-rw-r--   1 ryanseym ryanseym        0 Apr  4 22:48 Project42.txt
-rw-rw-r--   1 ryanseym ryanseym        0 Apr  4 22:48 Project43.txt
-rw-rw-r--   1 ryanseym ryanseym        0 Apr  4 22:48 Project44.txt
-rw-rw-r--   1 ryanseym ryanseym        0 Apr  4 22:48 Project45.txt
-rw-rw-r--   1 ryanseym ryanseym        0 Apr  4 22:48 Project46.txt
1 block

# restore files
ls
Project4x.cpio
# to restore files to original location, use -ivI options
# use -no-absolute-filename option to restore to different directory
cpio -iv --no-absolute-filenames -I Project4x.cpio 
Project42.txt
Project43.txt
Project44.txt
Project45.txt
Project46.txt
1 block
# all files including .cpio
ls
Project42.txt  Project43.txt  Project44.txt  Project45.txt  Project46.txt  Project4x.cpio
```

### tar

_tape archiver_, popular for creating data backups. Collects files and stores them in a an arvhice called a _tar file_. If the tar file is compressed, it is called a _tarball_.

Because it is a tape archiver, you can place tarballs or archive files on tape, such as an SCSI tape device. Replace the `-f <filename>` with the device filename, such as `/dev/st0`.

- Good practice to use `.tar` extension.
- When you use compression, add compression method to extension: `.tar.gz` or `.tgz` for gzip.
- `.snar` for tarball snapshot file

```bash
tar [OPTIONS...] [FILENAME]...
-c # create a tar file
-f # archive file name, should use .tar extension
-u # update, appends files to an existing tar file, but only ones that  
   # were modified since original archive file was created.
-g # creates incremental or full archive based on metadata stored in provided file
-z # compresses using gzip
-j # compresses using bzip2
-J # compresses using xz
-v # verbose

# create tar file
$ tar -cvf Project4x.tar Project4?.txt
Project42.txttar -g FullArchive.snar -Jcvf Project42.txz Project4?.txt
Project42.txt
Project43.txt
Project44.txt
Project45.txt
Project46.txt
ryanseymour:~/linux-playground/archives 
$ ls FullArchive.snar Project42.txz
FullArchive.snar  Project42.txz

Project43.txt
Project44.txt
Project45.txt
Project46.txt

# tar file with gzip compression (tar)
$ tar -zcvf Project4x.tar.gz Project4?.txt
Project42.txt
Project43.txt
Project44.txt
Project45.txt
Project46.txt
```

### tar full and incremental backups

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
### tar restore options

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

### dd