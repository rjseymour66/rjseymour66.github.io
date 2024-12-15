---
title: "Compression and archives"
linkTitle: "Compression"
# weight: 1000
# description:
---

Linux gathers files and directories in one step, and then optionally compresses them in the next step.

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

## tar

Bundles project files into a single output file for easy transfer across the network.
- Preserves folder structure and ownership.
- Does not delete original files or directories.
- `.tar.gz` is the same as `.tgz`

```bash
tar [OPTIONS] <tarfile-name> [FILES...]
# - : output tar contents to STDOUT (dash in place of archive filename)
# -c: create new archive
# -C: specify directory for extracted tar contents (with -x)
# --delete -f: delete file from archive
# --exclude="pattern": exclude objects that match pattern
# -f: archive's filename (put last)
# -j: use bzip2 compression instead of 
# -p: maintain permissions (during create)
# -r: appends file to existing tar archive (can't compress)
# -t: list contents of archive
# -v: verbose
# -x: extract files
# -z: gzip or unzip (depends whether -c or -x flag is present)

# all files in pwd
tar -cvf all-files.tar *

tar -cvzf myinits-$(date -I).tar /etc/init.d/       # with gzip
tar -cvjf myinits-$(date -I).tar.bz2 /etc/init.d/   # with bzip2

la -lhog myinits-2024-12-15.tar.bz2 myinits-2024-12-15.tar.gz   # gzip vs bzip2
-rw-r--r-- 1 16K Dec 15 11:14 myinits-2024-12-15.tar.bz2
-rw-r--r-- 1 18K Dec 15 11:09 myinits-2024-12-15.tar.gz

# extract single and multiple files from zipped archive
tar -xvf myinits-2024-12-15.tar.gz etc/init.d/anacron
tar -xvf myinits-2024-12-15.tar.gz "etc/init.d/anacron" "etc/init.d/apache2"

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

# delete file from archive

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

# check size of tar
tar -czf - myinits-2024-12-15.tar.gz | wc -c
17868

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