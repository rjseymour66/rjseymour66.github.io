---
title: "Protecting files"
weight: 110
description: >
  "Protecting files"
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
### tar restore

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

Creates low-level copies of an entire hard drive or partition (including MBR), such as:
- creating system images for forensics
- copying damaged disks
- wiping partitions

```bash
# input- and output-device is either entire drive or partition
dd if=<input-device> of=<output-device> [OPERANDS]
dd if=[SOURCE] of=[TARGET] [OPERANDS]
bs=BYTES        # sets max block size to read and write. Default is 512
count=N         # sets number of input blocks to copy
status=level    # sets amount of info to display to STDERR. Set to one of the following:
                #   none: displays only error messages
                #   noxfer: does not display final transfer stats
                #   progress: displays periodic transfer stats

# copy entire disk
# 1. list all block devices to make sure that drives are not mounted
lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
...
# 2. copy /dev/sdb to /dev/sdc
dd if=/dev/sdb of=/dev/sdc status=progress

# zero (wipe) a disk if you are throwing it away
# /dev/zero device file writes zeroes to the disk
dd if=/dev/zero of=/dev/sdc status=progress
```

## Replication

### rsync

Better than `scp` for large files.

Great at backing up larger files over the network. Also, backup files locally:

```bash
rsync [OPTION]... SOURCE DEST
-e # changes the program for network connection, OpenSSH by default
-z # compresses the data with zlib during transfer, good for bad network connections
# previously discussed options
-a # archive mode, equivalent to -rlptgoD (dir tree backups)
-D # retain Device and special files
-g # retain file group
-h # human-readable numeric output
-l # copy symbolic links
-o # retain file owner
-p # retain file perms
-P, --progress # display progress of file copy
-r # recursive
--stats # display file transfer stats
-t # retain file's modification time
-v # verbose

# backup files locally
rsync -avh *.tar TarStorage/
sending incremental file list
Project4x.tar
ProjectVerify.tar

sent 20.67K bytes  received 54 bytes  41.46K bytes/sec
total size is 20.48K  speedup is 0.99

ls TarStorage/
Project4x.tar  ProjectVerify.tar

# securely over the network (-e option uses OpenSSH)
rsync -avP -e ssh *.tar user1@10.20.30.40:~/path/to/target_dir
```

## Offsite/Off-System backups

### scp

Secure Copy Protocol

- Quickly transfers files in a noninteractive mannger between two systems on a network
- Uses SSH
- Best for small copies you need on the fly, bc if it gets interrupted, you cannot pick back up where you left off
- Will overwrite files on remote host if they have the same name:

```bash
scp [FILE] user@10.20.30.40:/absolute/path/to/target-dir
-C # compresses the file during transfer
-p # preserves file access, modification times, and perms
-r # recursive copy
-v # verbose

# copy from remote to remote
scp user@10.20.30.40:~/home user2@10.20.30.41:~/home/files
```

### sftp

SSH File Transfer Protocol.

Good for transferring larger files or archives. Provides interactive experience, for example:
- create directories as needed
- immediately check on transferred files,
- determine remote systems pwd

```bash
sftp username@localhost
username@localhosts password: 
Connected to localhost.
sftp> 

# common commands
bye # exits and quits sftp
exit # exits and quits sftp
get # gets file (downloads) from remote to local machine
reget # resumes interrupted get operation
put # sends (uploads) files from local to remote
reput # resumes interruped put operation
ls # list files in remote pwd
lls # list files in local pwd
mkdir # create dir on remote
lmkdir # create dir on local
progress # toggle progress display on/off (default is on)

# upload file to remote (localhost, here)
sftp username@localhost
username@localhosts password: 
Connected to localhost.
# check remote pwd
sftp> ls
Desktop                  Development              Documents                Downloads                
...
# check local pwd  
sftp> lls
Extract		  Project42_Inc.txz  Project43.txt  Project46.txt      TarStorage
FullArchive.snar  Project42.txt      Project44.txt  Project4x.tar
not-absolute	  Project42.txz      Project45.txt  ProjectVerify.tar
# make directory on local (would normally be remote, but we use localhost)
sftp> lmkdir sftp_files
# verify the dir was made (would be 'ls' on remote)
sftp> lls
Extract		  Project42_Inc.txz  Project43.txt  Project46.txt      sftp_files
...
# upload file
sftp> put Project4x.tar
Uploading Project4x.tar to /home/username/linux-playground/archives/sftp_files/Project4x.tar
Project4x.tar                                                     100%   10KB   7.5MB/s   00:00
# check for file on remote
sftp> ls
Project4x.tar   
# exit
sftp> bye
```

## Backup integrity

