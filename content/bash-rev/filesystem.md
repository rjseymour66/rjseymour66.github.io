---
title: "Navigating the filesystem"
weight: 40
description: >
  Finding files, searching content, navigating directory trees, and collecting system information.
---

Knowing how to navigate the Linux filesystem quickly and find what you are looking for is a skill that pays off in nearly every situation: investigating a configuration file that changed unexpectedly, finding large files filling up a disk, searching a codebase for a string, or collecting logs before an incident review. This page covers the primary tools for filesystem navigation and search, along with practical patterns for gathering system information.

## Listing files with ls

`ls` changes its output format depending on its destination. When stdout goes to a terminal, it formats output in multiple columns for human readability. When stdout is redirected (into a pipe or a file), it switches to one filename per line:

```bash
ls /bin                    # formatted, multi-column output on a terminal
ls /bin | cat              # one filename per line in a pipeline
ls -1                      # force single-column output regardless of destination
ls -C                      # force multi-column output regardless of destination
```

This difference matters when counting or processing filenames in a pipeline:

```bash
ls                         # shows 4 files on one line
ls | wc -l                 # counts 4 lines in the pipeline
ls -C | wc -l              # counts 1 line (forced single-line output)
```

## Navigating with CDPATH

`CDPATH` holds a list of directories that bash searches when you run `cd`. It works like `PATH`, but for directory navigation rather than executable lookup. When you change to a directory that matches a `CDPATH` entry, bash prints the absolute path to confirm where it landed.

Define `CDPATH` in your `.bashrc` to make it permanent:

```bash
CDPATH=$HOME:$HOME/projects:..
```

After this definition, `cd api` resolves to `$HOME/projects/api` from anywhere on the filesystem. Adding `..` to `CDPATH` lets you change to sibling directories without typing `../sibling`. For example, if you are in `/home/user/projects/frontend`, running `cd backend` changes to `/home/user/projects/backend` directly.

`CDPATH` works best when your directory names are unique. Duplicate names across directories can cause `cd` to resolve to the wrong location.

You can generate `CDPATH` automatically from your home directory structure by adding this to `.bashrc`:

```bash
CDPATH=$HOME$(cd && ls -d */ | sed -e 's@^@:$HOME/@g' -e 's@/$@@' | tr '\n' ':')..
```

## Finding files with find

`find` searches a directory tree recursively and prints matching file paths. It is the primary tool for locating files by name, type, size, owner, or modification time.

```bash
find /path -options
```

### Searching by name

Search for a file by name pattern:

```bash
find /etc -name "*.conf"                        # all .conf files in /etc
find /home -name "*.txt"                        # all text files under /home
find /home -name ".*"                           # hidden files (dotfiles)
find /home -name "*password*" 2>/dev/null       # suppress permission errors
```

### Searching by type

Pass `-type` to restrict results to a specific file type:

```bash
find /var/log -type f -name "*.log"             # regular files only
find /etc -type d                               # directories only
find /home -type l                              # symbolic links only
```

### Searching by size

Pass `-size` with a suffix to specify units: `k` for kilobytes, `M` for megabytes, `G` for gigabytes. Prefix with `+` for "greater than" and `-` for "less than":

```bash
find /var -size +100M                           # files larger than 100 MB
find /tmp -size -1k                             # files smaller than 1 KB
find /home -size +5G                            # files larger than 5 GB
```

Find the five largest files on the filesystem, suppressing permission errors:

```bash
ls / -R -s 2>/dev/null | sort -n -r | head -5
```

### Searching by modification time

The `-mtime` and `-mmin` options filter by when the file was last modified:

```bash
find /home -mmin -5                             # modified less than 5 minutes ago
find /home -mtime -1                            # modified less than 24 hours ago
find /home -mtime +2                            # modified more than 2 days ago
find /home -atime -1                            # accessed less than 24 hours ago
```

A practical scenario: find all files modified since last night's backup to identify what changed overnight:

```bash
find /var/www/html -type f -mtime -1 -ls
```

### Executing commands on results

The `-exec` option runs a command on each file that find returns. Replace `{}` with the file path and end the clause with `\;`:

```bash
find /etc -name "*.conf" -exec ls -l {} \;
```

Copy all files accessed in the last 24 hours to a staging directory:

```bash
find /home -type f -atime -1 -exec cp '{}' /tmp/recent/ \;
```

The `{}`  expands to the path of each found file. The `\;` ends the `-exec` clause.

For safe deletes, first do a dry run with `echo rm`, then execute:

```bash
find ~/downloads -type f -name "*~" -exec echo rm {} \;     # dry run: print what would be deleted
find ~/downloads -type f -name "*~" -delete                 # execute: use the built-in -delete option
```

### find in a loop

Pipe `find` output to a `while` loop to process each file:

```bash
find /var/log -type f -name "*.log" | while read -r LOGFILE; do
    LINES=$(wc -l < "$LOGFILE")
    if (( LINES > 10000 )); then
        echo "Large file: $LOGFILE ($LINES lines)"
    fi
done
```

## Searching file contents with grep

`grep` searches files for lines that match a string or pattern. Pass `-r` or `-R` to search an entire directory tree:

```bash
grep -r "PermitRootLogin" /etc/ssh/             # search for SSH config option
grep -r -i "password" /home 2>/dev/null         # find files mentioning passwords, case-insensitive
grep -r "DatabaseHost" /etc/app/ --include="*.conf"  # restrict to .conf files only
```

Find all files in `/home` that contain the string `password`:

```bash
grep -r -i /home -e 'password' 2>/dev/null
```

Combine `find` and `grep` to copy all files in `/home` that contain the word `password` to the current directory:

```bash
find /home -type f -exec grep -l 'password' '{}' \; -exec cp '{}' . \;
```

## Archiving and collecting logs with tar

`tar` creates and extracts archive files. The most common options are `c` (create), `z` (gzip compression), `f` (specify filename), and `x` (extract):

```bash
tar -czf archive.tar.gz /path/to/directory      # create a compressed archive
tar -xzf archive.tar.gz                         # extract a compressed archive
tar -tzf archive.tar.gz                         # list contents without extracting
```

Collect logs from a machine for forensic analysis. Parameter expansion retrieves the hostname automatically:

```bash
tar -czf "${HOSTNAME}_logs_$(date +%Y%m%d).tar.gz" /var/log
```

Transfer the archive to another machine with `scp`:

```bash
scp "${HOSTNAME}_logs_$(date +%Y%m%d).tar.gz" analyst@forensics.internal:/cases/
```

## Gathering system information

When you need a snapshot of a system's state, the following table compares Linux and Windows commands for common information-gathering tasks:

| Purpose | Linux command | Windows command |
|:--------|:--------------|:----------------|
| OS version and kernel | `uname -a` | `uname -a` |
| Hardware and system info | `cat /proc/cpuinfo` | `systeminfo` |
| Network interfaces | `ifconfig` or `ip addr` | `ipconfig` |
| Routing table | `ip route` | `route print` |
| ARP table | `arp -a` | `arp -a` |
| Network connections | `netstat -a` | `netstat -a` |
| Mounted filesystems | `mount` | `net share` |
| Running processes | `ps -e` | `tasklist` |

Collect a system snapshot for documentation or troubleshooting:

```bash
{
    echo "=== System Info ==="
    uname -a
    echo "=== CPU ==="
    grep "model name" /proc/cpuinfo | head -1
    echo "=== Memory ==="
    free -h
    echo "=== Disk ==="
    df -h
    echo "=== Network ==="
    ip addr show
    echo "=== Processes ==="
    ps aux --sort=-%cpu | head -20
} > "/tmp/system_snapshot_$(date +%Y%m%d_%H%M%S).txt"
```

## Key log file locations

The following table lists the most important log files on a Linux system:

| Path | Contents |
|:-----|:---------|
| `/var/log/syslog` | General system messages (Debian/Ubuntu) |
| `/var/log/messages` | General system messages (RHEL/CentOS) |
| `/var/log/auth.log` | Authentication events including SSH logins (Debian/Ubuntu) |
| `/var/log/secure` | Authentication events (RHEL/CentOS) |
| `/var/log/kern.log` | Kernel messages |
| `/var/log/apache2/` | Apache web server access and error logs |
| `/var/log/nginx/` | Nginx web server access and error logs |

Read log configuration from `/etc/syslog.conf` or `/etc/rsyslog.conf` to understand how logs are routed on a specific system.

The `journalctl` command queries the systemd journal, which captures logs from all systemd-managed services:

```bash
journalctl -u nginx --since "1 hour ago"        # nginx logs from the last hour
journalctl -p err -b                            # errors since last boot
journalctl -f                                   # follow in real time
```

## Computing file checksums

`sha1sum` and `md5sum` compute integrity hashes for files. Checksums are useful for verifying that a file has not been modified since it was copied or downloaded:

```bash
sha1sum /etc/passwd                             # compute hash
sha1sum -c hashes.txt                           # verify against a list of expected hashes
```

Generate a hash of a sensitive configuration file before making changes, so you can verify it later:

```bash
sha1sum /etc/ssh/sshd_config > /root/sshd_config.sha1
sha1sum -c /root/sshd_config.sha1               # verify: outputs OK or FAILED
```
