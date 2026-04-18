---
title: "Commands reference"
weight: 60
description: >
  All commands covered in this section, organized by category with options and usage examples.
---

This page lists all commands covered in the bash documentation section, organized by category. Each entry includes a brief description, common options, and a real-world usage example. Refer to this page when you need a quick reminder of syntax or flags.

## Text processing

### awk

A programming language for transforming structured text. Splits each line into fields and runs your program against each line. Fields are numbered `$1`, `$2`, ..., `$NF` (last field). `$0` is the whole line.

Common options:

| Option | Description |
|:-------|:------------|
| `-F:`  | Set field separator to `:` |
| `-f`   | Read awk program from a file |
| `-v`   | Set a variable before execution |

Common usage:

```bash
awk '{print $1}' file.txt                           # print first column
awk -F: '{print $1, $7}' /etc/passwd                # usernames and shells from passwd
awk '{sum+=$1} END {print sum}' numbers.txt         # sum the first column
awk '$9 >= 500' /var/log/apache2/access.log         # HTTP 5xx errors
awk '{counts[$1]++} END {for (k in counts) print counts[k], k}' ips.txt  # count by key
```

### cat

Concatenates files and prints them to stdout. Short for "concatenate."

Common usage:

```bash
cat file1.txt file2.txt                 # combine two files
cat -n file.txt                         # print with line numbers
cat /dev/null > file.txt                # empty a file without deleting it
```

### cut

Extracts specific columns from a file.

Common options:

| Option  | Description |
|:--------|:------------|
| `-f N`  | Print field N (tab-delimited by default) |
| `-f N-M`| Print fields N through M |
| `-c N`  | Print character N |
| `-c N-M`| Print characters N through M |
| `-d ,`  | Set field delimiter to `,` |

Common usage:

```bash
cut -d: -f1 /etc/passwd                 # extract usernames
cut -d: -f1,7 /etc/passwd               # extract usernames and shells
cut -f1-3 report.tsv                    # first three tab-delimited fields
cut -c1-32 checksums.txt                # first 32 characters (md5sum width)
```

### diff

Compares two files line by line and prints their differences.

Common options:

| Option | Description |
|:-------|:------------|
| `-u`   | Unified diff format (shows context) |
| `-r`   | Recursively compare directories |
| `-i`   | Ignore case differences |
| `-q`   | Report only whether files differ |

Common usage:

```bash
diff file1.txt file2.txt                            # basic comparison
diff -u config.old config.new                       # unified diff for reviewing changes
diff -r dir1/ dir2/                                 # compare two directories
diff <(sort file1.txt) <(sort file2.txt)            # compare sorted versions
```

### grep

Searches files for lines matching a pattern and prints each matching line.

Common options:

| Option | Description |
|:-------|:------------|
| `-c`   | Count matching lines |
| `-E`   | Enable extended regular expressions |
| `-i`   | Ignore case |
| `-l`   | Print only filenames with at least one match |
| `-n`   | Print line number before each match |
| `-o`   | Print only the matching text |
| `-q`   | Silent mode (exit status only) |
| `-r`   | Recursive search |
| `-R`   | Recursive, following symbolic links |
| `-v`   | Invert match: print non-matching lines |
| `-w`   | Match whole words only |
| `-P`   | Perl-compatible regular expressions |

Common usage:

```bash
grep "Failed password" /var/log/auth.log            # find failed SSH logins
grep -c "ERROR" /var/log/app.log                    # count error lines
grep -r "PermitRootLogin" /etc/ssh/                 # search SSH config recursively
grep -v '^#' /etc/nginx/nginx.conf | grep -v '^$'  # non-comment, non-blank lines
grep -E '^(web|db|cache)-[0-9]+' /etc/hosts        # match server naming patterns
```

### head

Prints the first lines of a file. Defaults to 10 lines.

Common options:

| Option | Description |
|:-------|:------------|
| `-n N` | Print the first N lines |
| `-c N` | Print the first N bytes |

Common usage:

```bash
head -20 /var/log/syslog                # first 20 lines
ls /bin | head -5                       # first 5 filenames
head -3 access.log | wc -w             # count words in first 3 lines
```

### md5sum

Computes or verifies MD5 checksums for files.

Common options:

| Option | Description |
|:-------|:------------|
| `-c`   | Verify checksums from a file |

Common usage:

```bash
md5sum /etc/passwd                                  # compute hash
md5sum *.txt | cut -d' ' -f1 | sort | uniq -c       # find duplicate files
md5sum -c checksums.md5                             # verify a list of files
```

### paste

Combines files side by side, separating columns with a tab.

Common options:

| Option  | Description |
|:--------|:------------|
| `-d ,`  | Set column delimiter to `,` |
| `-d \n` | Interleave lines from two files |

Common usage:

```bash
paste names.txt scores.txt                          # tab-delimited columns
paste -d, first.txt last.txt                        # comma-delimited
paste -d '\n' evens.txt odds.txt                    # interleave lines
paste <(seq 1 5) <(seq 6 10)                        # combine generated sequences
```

### rev

Reverses the characters on each line.

Common usage:

```bash
rev /etc/shells | cut -d/ -f1 | rev     # extract last path component from each line
echo "hello" | rev                       # olleh
```

### sed

Transforms text by applying scripts to each line. Most commonly used for substitution.

Common options:

| Option | Description |
|:-------|:------------|
| `-i`   | Modify the file in place |
| `-e`   | Add a script to run |
| `-f`   | Read scripts from a file |
| `-n`   | Suppress default output (print only with `p`) |

Common usage:

```bash
sed 's/old/new/g' file.txt                          # replace all occurrences
sed -i 's/old-host/new-host/g' config.conf          # in-place replacement
sed '/^#/d' config.conf                             # delete comment lines
sed -n '5,10p' /var/log/syslog                      # print lines 5 through 10
sed -n '/ERROR/p' app.log                           # print error lines only
echo "2025-03-29" | sed 's/\([0-9]*\)-\([0-9]*\)-\([0-9]*\)/\3\/\2\/\1/'  # reformat date
```

### sha1sum

Computes or verifies SHA-1 checksums for files.

Common options:

| Option | Description |
|:-------|:------------|
| `-c`   | Verify checksums from a file |

Common usage:

```bash
sha1sum /etc/ssh/sshd_config                        # compute a hash before editing
sha1sum deployment.tar.gz                           # verify a downloaded archive
sha1sum -c published_checksums.txt                  # batch verification
```

### sort

Sorts lines of a file.

Common options:

| Option  | Description |
|:--------|:------------|
| `-r`    | Reverse order |
| `-n`    | Numeric sort |
| `-nr`   | Numeric descending |
| `-u`    | Remove duplicates |
| `-f`    | Ignore case |
| `-k N`  | Sort by field N |
| `-t ,`  | Set field delimiter to `,` |
| `-o`    | Write output to a file |

Common usage:

```bash
sort /etc/hosts                                     # alphabetical sort
sort -rn numbers.txt                                # numeric descending
sort -k3 access.log                                 # sort by third field
sort -u /var/log/ips.txt                            # sorted, unique IPs
cut -f1 grades | sort | uniq -c | sort -rn          # grade frequency analysis
```

### tac

Prints files in reverse line order (bottom to top). The name is `cat` spelled backwards.

Common usage:

```bash
tac /var/log/apache2/access.log | head -20          # most recent 20 log entries
tac changelog.txt                                   # read a changelog newest-first
```

### tail

Prints the last lines of a file. Defaults to 10 lines.

Common options:

| Option  | Description |
|:--------|:------------|
| `-n N`  | Print the last N lines |
| `-n +N` | Print starting at line N |
| `-f`    | Follow the file as new lines are written |
| `-F`    | Follow by filename (handles log rotation) |

Common usage:

```bash
tail -f /var/log/nginx/error.log                    # live log monitoring
tail -n 50 /var/log/syslog                          # last 50 lines
tail -n +3 /etc/hosts                               # from line 3 to end
tail -F /var/log/app.log                            # follow through log rotation
```

### tr

Translates or deletes characters.

Common options:

| Option | Description |
|:-------|:------------|
| `-d`   | Delete characters in the first set |
| `-s`   | Squeeze repeated characters to one |

Common usage:

```bash
echo $PATH | tr ':' '\n'                            # print PATH entries one per line
echo "HELLO" | tr '[A-Z]' '[a-z]'                   # convert to lowercase
echo "hello world" | tr -d ' '                      # remove spaces
echo "a   b" | tr -s ' '                            # squeeze multiple spaces to one
```

### uniq

Filters adjacent duplicate lines. Always sort input first.

Common options:

| Option | Description |
|:-------|:------------|
| `-c`   | Prefix each line with its occurrence count |
| `-d`   | Print only lines that appear more than once |
| `-u`   | Print only lines that appear exactly once |
| `-i`   | Ignore case |

Common usage:

```bash
sort ips.txt | uniq                                 # unique IP addresses
sort access.log | uniq -c | sort -rn                # most frequent log entries
awk '{print $9}' access.log | sort | uniq -c | sort -rn | head  # HTTP status codes
```

### wc

Counts lines, words, and characters.

Common options:

| Option | Description |
|:-------|:------------|
| `-l`   | Count lines only |
| `-w`   | Count words only |
| `-c`   | Count characters only |

Common usage:

```bash
wc -l /var/log/auth.log                             # line count
wc -w /etc/nginx/nginx.conf                         # word count
ls -1 /etc | wc -l                                  # number of entries in a directory
grep "ERROR" app.log | wc -l                        # count error lines
```

## File system

### cd

Changes the current working directory. A built-in command.

Common usage:

```bash
cd /var/log                     # absolute path
cd ..                           # parent directory
cd -                            # previous directory
cd                              # home directory
cd ~user                        # home directory of another user
```

### chmod

Changes file or directory permissions.

Common options:

| Option  | Description |
|:--------|:------------|
| `-R`    | Recursive |
| `+x`    | Add execute permission |
| `755`   | Set owner rwx, group rx, other rx |
| `644`   | Set owner rw, group r, other r |

Common usage:

```bash
chmod +x deploy.sh                                  # make a script executable
chmod 755 /usr/local/bin/myscript                   # standard executable permissions
chmod 644 /etc/app/config.conf                      # standard config file permissions
chmod -R 750 /var/app/                              # recursive, owner and group only
```

### chown

Changes file owner and group.

Common options:

| Option | Description |
|:-------|:------------|
| `-R`   | Recursive |

Common usage:

```bash
chown alice file.txt                                # change owner to alice
chown alice:developers project/                     # change owner and group
chown -R www-data:www-data /var/www/html/           # fix web server file ownership
```

### chgrp

Changes the group ownership of a file.

Common usage:

```bash
chgrp developers /var/app/config.conf
chgrp -R staff /shared/docs/
```

### cp

Copies files and directories.

Common options:

| Option | Description |
|:-------|:------------|
| `-r`   | Recursive (for directories) |
| `-a`   | Archive mode: preserve permissions, timestamps, and links |
| `-i`   | Prompt before overwriting |
| `-p`   | Preserve file attributes |

Common usage:

```bash
cp config.conf config.conf.bak                      # back up a file before editing
cp -a /var/www/html/ /var/www/html.backup/          # archive copy of a directory
cp -r /etc/app/ /tmp/app_backup/                    # recursive copy
```

### df

Reports filesystem disk space usage.

Common options:

| Option | Description |
|:-------|:------------|
| `-h`   | Human-readable sizes |
| `-T`   | Show filesystem type |

Common usage:

```bash
df -h                                               # all mounted filesystems
df -h /var                                          # usage for a specific mount
df -h / | awk 'NR==2 {print $5}'                    # root disk usage percentage
```

### find

Searches directory trees for files matching criteria.

Common options:

| Option         | Description |
|:---------------|:------------|
| `-name`        | Match by filename (supports globs) |
| `-type f`      | Regular files only |
| `-type d`      | Directories only |
| `-size +100M`  | Files larger than 100 MB |
| `-mtime -1`    | Modified within 24 hours |
| `-mtime +7`    | Modified more than 7 days ago |
| `-exec cmd {} \;` | Run a command on each result |
| `-delete`      | Delete matching files |

Common usage:

```bash
find /etc -name "*.conf" -type f                    # config files in /etc
find /var -size +50M                                # large files
find /home -mtime -1 -type f                        # files changed today
find /tmp -type f -mtime +7 -delete                 # clean up old temp files
find / -name "authorized_keys" 2>/dev/null          # locate SSH authorized key files
```

### ls

Lists directory contents.

Common options:

| Option | Description |
|:-------|:------------|
| `-l`   | Long format (permissions, owner, size, date) |
| `-a`   | Include hidden files |
| `-h`   | Human-readable sizes with `-l` |
| `-t`   | Sort by modification time, newest first |
| `-r`   | Reverse sort order |
| `-1`   | Force one entry per line |
| `-S`   | Sort by file size |

Common usage:

```bash
ls -lh /var/log                                     # long listing with sizes
ls -lt /etc | head -10                              # 10 most recently modified files
ls -la ~                                            # all files including hidden
ls -1 /etc | wc -l                                  # count files in a directory
```

### mkdir

Creates directories.

Common options:

| Option | Description |
|:-------|:------------|
| `-p`   | Create parent directories as needed |

Common usage:

```bash
mkdir /var/app/logs                                 # create a directory
mkdir -p /var/app/{logs,config,data}                # create multiple directories with parents
mkdir -p deploy/{staging,production}/configs        # nested structure in one command
```

### mv

Moves or renames files and directories.

Common options:

| Option | Description |
|:-------|:------------|
| `-i`   | Prompt before overwriting |
| `-n`   | Do not overwrite existing files |

Common usage:

```bash
mv config.conf config.conf.bak                      # rename a file
mv *.log /var/archive/                              # move files to another directory
mv /tmp/new_config.conf /etc/app/config.conf        # deploy a new config
```

### rm

Removes files and directories.

Common options:

| Option | Description |
|:-------|:------------|
| `-r`   | Recursive (required for directories) |
| `-f`   | Force: no prompts, ignore non-existent files |
| `-i`   | Prompt before each deletion |

Common usage:

```bash
rm old_report.txt                                   # delete a file
rm -i *.tmp                                         # delete with confirmation
rm -rf /tmp/build_cache/                            # force-delete a directory tree
```

### tar

Creates and extracts archive files.

Common options:

| Option | Description |
|:-------|:------------|
| `-c`   | Create archive |
| `-x`   | Extract archive |
| `-z`   | Compress or decompress with gzip |
| `-f`   | Specify the archive filename |
| `-t`   | List archive contents without extracting |
| `-v`   | Verbose output |

Common usage:

```bash
tar -czf backup.tar.gz /var/www/html/               # create a compressed archive
tar -xzf backup.tar.gz                              # extract
tar -tzf backup.tar.gz                              # list contents
tar -czf "${HOSTNAME}_logs_$(date +%Y%m%d).tar.gz" /var/log   # timestamped log archive
```

### touch

Creates empty files or updates the modification timestamp of existing files.

Common usage:

```bash
touch new_file.txt                                  # create an empty file
touch -t 202501010000 file.txt                      # set a specific timestamp
touch file{0..999}.txt                              # create 1000 empty files at once
```

## Processes

### bg

Resumes a suspended job in the background.

Common usage:

```bash
bg                  # resume the most recently suspended job
bg %2               # resume job number 2
```

### fg

Brings a background or suspended job to the foreground.

Common usage:

```bash
fg                  # foreground the most recent job
fg %1               # foreground job number 1
```

### jobs

Lists all jobs running in the current shell.

Common usage:

```bash
jobs                # list jobs with status
jobs -l             # include process IDs
```

### kill

Sends a signal to a process.

Common options:

| Option   | Description |
|:---------|:------------|
| `-9`     | SIGKILL: force-terminate immediately |
| `-15`    | SIGTERM: polite termination (default) |
| `-l`     | List all signal names |

Common usage:

```bash
kill 12345                      # send SIGTERM to process 12345
kill -9 12345                   # force-kill
kill %1                         # kill background job 1
kill -l                         # list all signals
```

### ps

Reports information about running processes.

Common options:

| Option  | Description |
|:--------|:------------|
| `aux`   | All processes with user, CPU, and memory usage |
| `-ef`   | All processes in full format |
| `-f`    | Full listing for current user |

Common usage:

```bash
ps aux                                              # all processes
ps aux | grep nginx                                 # find nginx processes
ps aux --sort=-%mem | head -10                      # top 10 by memory usage
ps -ef | grep "[p]ython"                            # find Python processes (brackets avoid matching grep itself)
```

### sleep

Pauses script execution for a specified duration.

Common usage:

```bash
sleep 5                         # pause for 5 seconds
sleep 0.5                       # pause for half a second
sleep 2h                        # pause for 2 hours (suffixes: s, m, h, d)
```

## Networking

### arp

Displays and manipulates the ARP cache (Address Resolution Protocol).

Common usage:

```bash
arp -a                          # display the ARP table
```

### curl

Transfers data from or to a server. Prints output to stdout by default.

Common options:

| Option  | Description |
|:--------|:------------|
| `-O`    | Save output to a file using the remote filename |
| `-o`    | Save output to a specified file |
| `-s`    | Silent: suppress progress output |
| `-L`    | Follow redirects |
| `-I`    | Fetch headers only |

Common usage:

```bash
curl https://example.com/api/health                 # check an endpoint
curl -O https://example.com/file.tar.gz             # download a file
curl -s https://api.example.com/status | jq .       # parse JSON response
curl -I https://example.com                         # inspect response headers
```

### ip

Queries and configures network interfaces and routes. Replaces the older `ifconfig` and `route` commands.

Common usage:

```bash
ip addr show                            # list all interfaces and IP addresses
ip addr show eth0                       # show a specific interface
ip route                                # display the routing table
ip link show                            # show link status for all interfaces
```

### netstat

Displays network connections, routing tables, and interface statistics. On modern systems, `ss` is a faster alternative.

Common options:

| Option | Description |
|:-------|:------------|
| `-a`   | All connections and listening sockets |
| `-n`   | Numeric addresses (skip DNS resolution) |
| `-t`   | TCP connections only |
| `-u`   | UDP connections only |
| `-p`   | Show process name and PID |

Common usage:

```bash
netstat -an | grep :80                              # connections on port 80
netstat -tlnp                                       # listening TCP ports with process info
ss -tlnp                                            # modern equivalent with ss
```

### scp

Copies files between hosts over SSH.

Common options:

| Option | Description |
|:-------|:------------|
| `-r`   | Recursive |
| `-p`   | Preserve timestamps and permissions |

Common usage:

```bash
scp file.txt user@host:/tmp/                        # copy to remote machine
scp user@host:/var/log/app.log /tmp/                # copy from remote machine
scp -r ./configs/ user@host:/etc/app/               # recursive copy
```

### ssh

Opens a secure shell connection to a remote host.

Common options:

| Option | Description |
|:-------|:------------|
| `-T`   | Suppress pseudo-terminal output |
| `-i`   | Specify identity (key) file |
| `-p`   | Specify port |

Common usage:

```bash
ssh user@host                                       # interactive session
ssh user@host ps aux                                # run a single command
ssh user@host bash < ./deploy.sh                    # run a local script remotely
echo "df -h" | ssh -T user@host                     # pipe a command to SSH
```

### wget

Downloads files from the web and saves them to disk.

Common options:

| Option   | Description |
|:---------|:------------|
| `-q`     | Quiet mode |
| `-O`     | Save to a specified filename |
| `-r`     | Recursive download |
| `--limit-rate` | Limit download speed |

Common usage:

```bash
wget https://example.com/file.tar.gz                # download a file
wget -q -O /tmp/status.json https://api.example.com/status  # quiet, custom filename
```

## System information

### cal

Displays a calendar.

Common usage:

```bash
cal                     # current month
cal 2025                # full year calendar
cal 3 2025              # March 2025
```

### date

Prints or sets the system date and time.

Common usage:

```bash
date                                    # current date and time
date +%Y-%m-%d                          # ISO 8601 date
date +%s                                # Unix epoch timestamp
date --date "2 weeks ago" +%Y-%m-%d     # date relative to today
```

### iostat

Reports CPU and I/O statistics.

Common usage:

```bash
iostat                          # summary since boot
iostat 2 5                      # report every 2 seconds, 5 times
```

### last

Shows login history by reading the `/var/log/wtmp` file.

Common usage:

```bash
last                            # all logins
last -n 20                      # last 20 logins
last alice                      # login history for user alice
last reboot                     # system reboots
```

### mount

Lists mounted filesystems or mounts a filesystem.

Common usage:

```bash
mount                           # list all mounted filesystems
mount | grep /var               # check if /var is mounted
```

### sar

Collects and reports system activity statistics. Requires the `sysstat` package.

Common usage:

```bash
sar 1 5                         # CPU usage every second, 5 times
sar -r 1 5                      # memory usage
sar -d 1 5                      # disk activity
```

### uname

Prints system information.

Common options:

| Option | Description |
|:-------|:------------|
| `-a`   | All available information |
| `-r`   | Kernel release only |
| `-m`   | Machine hardware name |

Common usage:

```bash
uname -a                        # full system information
uname -r                        # kernel version
```

### vmstat

Reports virtual memory, processes, and CPU activity.

Common usage:

```bash
vmstat                          # snapshot
vmstat 2 5                      # report every 2 seconds, 5 times
```

### w

Displays who is logged in and what they are doing. An extended version of `who` that also shows uptime.

Common usage:

```bash
w                               # logged-in users with activity
w alice                         # activity for a specific user
```

### who

Shows who is currently logged in to the system.

Common usage:

```bash
who                             # current users
who am i                        # your own login info: name, terminal, login time, and origin
```

### whoami

Prints your current username.

Common usage:

```bash
whoami                          # useful in scripts to check if running as root
[ "$(whoami)" = "root" ] || { echo "Must run as root" >&2; exit 1; }
```

## Shell utilities

### alias

Creates or lists command shortcuts.

Common usage:

```bash
alias                           # list all aliases
alias ll="ls -lh"               # define an alias
alias rm="rm -i"                # safer rm
unalias ll                      # remove an alias
```

### export

Marks a variable as an environment variable, making it available to child processes.

Common usage:

```bash
export PATH=$HOME/bin:$PATH     # extend PATH
export DB_HOST=db.internal      # pass configuration to scripts
export EDITOR=vim               # set default editor
```

### history

Lists commands previously run in the current shell.

Common usage:

```bash
history                         # all history
history 20                      # last 20 commands
history | grep ssh              # filter history
history -c                      # clear history
```

### printenv

Prints environment variables.

Common usage:

```bash
printenv                        # all environment variables
printenv PATH                   # a specific variable
printenv | sort                 # sorted listing
```

### read

Reads a line from stdin and stores it in a variable.

Common options:

| Option | Description |
|:-------|:------------|
| `-p`   | Display a prompt string |
| `-s`   | Silent (do not echo input) |
| `-a`   | Read into an array |
| `-r`   | Raw mode (do not interpret backslashes) |
| `-t N` | Time out after N seconds |

Common usage:

```bash
read -p "Enter hostname: " HOST
read -s -p "Password: " PASS
readarray -t LINES < /etc/hosts
```

### seq

Generates a sequence of numbers.

Common usage:

```bash
seq 1 10                        # 1 to 10
seq 0 2 20                      # 0, 2, 4, ... 20
seq -w 1 10                     # zero-padded: 01, 02, ... 10
seq -s, 1 5                     # 1,2,3,4,5 (comma-separated)
```

### set

Displays all shell variables and functions, or controls shell options.

Common usage:

```bash
set                             # list all variables
set -x                          # enable debug output (trace commands)
set +x                          # disable debug output
set -e                          # exit on error
set -u                          # treat unset variables as errors
```

### source

Runs a script in the current shell rather than a child process. Changes to variables and the environment take effect in your current shell.

Common usage:

```bash
source ~/.bashrc                # reload shell configuration
source ./env.sh                 # load environment variables into current shell
. ~/.bashrc                     # shorthand for source
```

### type

Identifies how the shell will interpret a command: as a file, built-in, alias, or keyword.

Common usage:

```bash
type ls                         # ls is /usr/bin/ls
type cd                         # cd is a shell builtin
type ll                         # ll is aliased to `ls -lh`
type if                         # if is a shell keyword
type -t cat                     # print type only: "file"
```

### which

Locates the executable file that the shell will run for a given command.

Common usage:

```bash
which python3                   # /usr/bin/python3
which bash                      # /usr/bin/bash
```

### xargs

Builds and executes commands from stdin.

Common options:

| Option | Description |
|:-------|:------------|
| `-n N` | Pass at most N arguments per command |
| `-I {}` | Use `{}` as a placeholder for each input string |
| `-0`   | Read null-delimited input (use with `find -print0`) |
| `-P N` | Run N processes in parallel |

Common usage:

```bash
find . -name "*.py" -print0 | xargs -0 wc -l           # line count across all Python files
cat servers.txt | xargs -I {} ssh {} uptime             # run uptime on each server
find . -name "*.log" -print0 | xargs -0 grep "ERROR"   # search logs with null separators
ls *.jpg | xargs -n1 -I {} convert {} -resize 800x {}_thumb.jpg  # batch image resize
```

## User and permissions

### chgrp

Changes the group ownership of a file.

Common usage:

```bash
chgrp developers /var/app/                          # change group
chgrp -R staff /shared/docs/                        # recursive
```

### passwd

Changes a user's password.

Common usage:

```bash
passwd                          # change your own password
sudo passwd alice               # change another user's password (as root)
```
