+++
title = 'Compression'
date = '2025-09-07T18:49:25-04:00'
weight = 50
draft = false
+++


Linux bundles files and directories in one step, then optionally compresses them in the next. The available tools range from single-file compression with `gzip`, `bzip2`, and `xz`, to multi-file archives with `tar`, to splitting large archives for transfer with `split`.

## Compression methods

The following compression methods all provide lossless compression: 
_gzip_
: Replaced the old `compress` program. Replaces the original file with a compressed version using the `.gz` extension. Run `gunzip` to reverse the operation.

_bzip2_
: Compressed the Linux kernel until 2013. Replaces the original file.

_xz_
: Popular with Linux admins. Has compressed the Linux kernel since 2013. Replaces the original file.

_zip_
: Operates on multiple files and packs them into an archive. Does not replace the original file. Instead, it places a copy into the archive.



### Comparing compression methods

To compare compression methods on the same set of files:

1. List the files to confirm their starting size.

    ```bash
    ls -lh wtmp?
    -rw-rw-r-- 1 137K Apr  4 22:08 wtmp1
    -rw-rw-r-- 1 137K Apr  4 22:08 wtmp2
    -rw-rw-r-- 1 137K Apr  4 22:08 wtmp3
    -rw-rw-r-- 1 137K Apr  4 22:08 wtmp4
    ```

2. Compress each file with a different method. Use `-<n>` to control compression level: `-1` is fastest with the lowest compression, `-9` is slowest with the highest, and `-6` is the default.

    ```bash
    gzip wtmp1      # gunzip <compressed-file> to reverse
    bzip2 wtmp2     # bunzip2 <compressed-file> to reverse
    xz wtmp3        # unxz <compressed-file> to reverse
    zip wtmp4.zip wtmp4     # unzip <compressed-file> to reverse
    ```

3. Compare the resulting file sizes. `xz` produces the smallest output.

    ```bash
    ls -lh wtmp?.*
    -rw-rw-r-- 1 5.6K Apr  4 22:08 wtmp1.gz
    -rw-rw-r-- 1 4.4K Apr  4 22:08 wtmp2.bz2
    -rw-rw-r-- 1 3.9K Apr  4 22:08 wtmp3.xz
    -rw-rw-r-- 1 5.7K Apr  4 22:18 wtmp4.zip
    ```

4. Verify that `zip` did not replace the original file.

    ```bash
    ls wtmp?
    wtmp4
    ```

## tar

`tar` bundles files into a single archive for easy network transfer. It preserves folder structure and ownership and does not delete original files or directories. The `.tar.gz` and `.tgz` extensions are equivalent.

```bash
tar [OPTIONS] <tarfile-name> [FILES...]
# - : output tar contents to STDOUT (dash in place of archive filename)
# -c: create new archive
# -C: specify directory for extracted tar contents (with -x)
# --delete -f: delete file from archive
# --exclude="pattern": exclude objects that match pattern
# -f: archive's filename (put last)
# -j: use bzip2 compression
# -p: maintain permissions (during create)
# -r: appends file to existing tar archive (can't compress)
# -t: list contents of archive
# -v: verbose
# -x: extract files
# -z: gzip or gunzip (depends on whether -c or -x is present)
```

### Create an archive

```bash
tar -cvf all-files.tar *                                        # archive all files in pwd
tar -cvzf myinits-$(date -I).tar /etc/init.d/                  # with gzip
tar -cvjf myinits-$(date -I).tar.bz2 /etc/init.d/              # with bzip2

la -lhog myinits-2024-12-15.tar.bz2 myinits-2024-12-15.tar.gz  # compare gzip vs bzip2
-rw-r--r-- 1 16K Dec 15 11:14 myinits-2024-12-15.tar.bz2
-rw-r--r-- 1 18K Dec 15 11:09 myinits-2024-12-15.tar.gz
```

### Extract files

```bash
tar -xvf myinits-2024-12-15.tar.gz etc/init.d/anacron                          # extract single file
tar -xvf myinits-2024-12-15.tar.gz "etc/init.d/anacron" "etc/init.d/apache2"   # extract multiple files
tar -zxvf number-files.tar.gz -C extractions/                                   # extract to a directory
```

### View archive contents

```bash
tar -tvf number-files.tar.gz
-rw-rw-r-- linuxuser/linuxuser 23 2024-11-08 13:48 file.1
...
```

### Create an archive on a remote machine

```bash
# pipe tar output to a remote machine
tar -czvf - *.[0-9]* | ssh username@10.20.30.40 \
"cat > /home/path/to/dest/remote-tars.tar.gz"

# archive the current filesystem plus /var and /usr (may require sudo)
tar -czvf - --one-file-system / /var /usr \
| ssh username@10.20.30.40 \
"cat > /home/path/to/dest/workstation-backup-Nov-8.tar.gz"
```

### Extract and maintain permissions

```bash
sudo tar -xzvf perms.tar.gz

ls -l
total 4
-rw-rw-r-- 1 linuxuser linuxuser   0 Nov 10 22:54 file1
-rw-rw-r-- 1 linuxuser linuxuser   0 Nov 10 22:54 file2
-rw-rw-r-- 1 newuser   newuser     0 Nov 10 22:54 file3
-rw-rw-r-- 1 linuxuser linuxuser 157 Nov 10 22:58 perms.tar.gz
```

### Check archive size

```bash
tar -czf - myinits-2024-12-15.tar.gz | wc -c
17868
```

## gzip

`gzip` is the most common Linux compression tool. By default, it compresses a file and deletes the original.

### Compress and delete original

```bash
gzip find.tar
```

### Compress and preserve original

```bash
gzip -c find.tar > find.tar.gz
```

### Decompress

```bash
gunzip find.tar.gz
```

## split

`split` breaks an archive into multiple smaller files for transfer. On the remote machine, reassemble the pieces with `cat`.

The following example splits an archive into 100-byte pieces and reassembles them:

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