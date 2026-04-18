+++
title = 'Troubleshooting'
date = '2025-09-07T18:49:41-04:00'
weight = 100
draft = false
+++



You need to know where to start troubleshooting, and some common techniques.

## Evaluating the scope

Determine where the problem is and how many systems and services are affected. Reproducing the problem makes this easier. If you can't reproduce it, trace how each network component contributes to the symptoms.

Start by asking: What are the symptoms? When did the problem start? Has this happened before? Were there recent network changes? If so, check the DNS or DHCP server. Identify which servers and users are impacted.

If the problem affects a single machine, check the login logs and `.bash_history` for recently executed commands, or run `history` after logging in as the affected user. Run `w` to see who is currently logged in and their IP addresses.
  
## Root cause analysis

After you resolve a problem, figure out how it started and how to prevent it from happening again. Root cause analysis (RCA) is a learning experience. Document the events that led to the issue: which app or hardware was affected, the date and time the problem was first noticed, and the events, configurations, or faults that caused it. Include a list of steps taken to correct the issue.

RCA is often imperfect. You may resolve the issue without knowing with certainty why it happened.

## System logs

Logs are a great first step in figuring out what happened. There are two ways to view them:

- For services managed by systemd, use `journalctl -u <service>`.
- For services that don't use systemd, check `/var/log/<file>.log`. Application logs are created by the app; system logs are created by the distro.

Some daemons write to dedicated files in `/var/log` — for example, Apache writes to `/var/log/apache2/`. Daemons without a dedicated log file write to `/var/log/syslog` instead. Cron jobs, `dhclient`, and the `systemd` init daemon all log there. Use `grep` to filter syslog contents:

```bash
cat /var/log/syslog | grep <pattern>
```

When reproducing an issue, run `tail -f` to follow log output in real time.

### Important log files

`/var/log/auth.log`
: Records security-related events. Requires root access and includes authentication attempts from the server and SSH. A high volume of auth attempts may signal an intrusion attempt.

`/var/log/syslog`
: A catch-all for daemons that don't have a dedicated log file. Cron jobs, `dhclient`, and the `systemd init` daemon all write here.

`/var/log/dpkg.log`
: Records package installation and upgrade activity. Check it if the server behaves unexpectedly after an install or update.

### Compressed logs

`logrotate` compresses and renames old log files, giving them a `.gz` extension. Use these commands to work with compressed logs without extracting them:

- `zcat` — view file contents directly.
- `zless` — page through file contents.

### Commands

Common log viewing commands:

```bash
# --- two methods to view logs --- #
cat /var/log/<file>.log     # older method
jouralctl -u <service>      # newer method, -u option shows messages (logs) for <service>
jouralctl -uf <service>     # newer method, -f option (follow) continuously prints log entries as they're added


# --- /var/log/<file> --- #
tail /var/log/apache2/access.log        # view last 10 entries
tail -n /var/log/apache2/access.log     # view last X entries
tail -f /var/log/apache2/access.log     # follow, continuosly print logs as they're added = good when reproducing issue
less /var/log/apache2/access.log        # page through the log file
cat /var/log/syslog | grep <pattern>    # filter log file contents

zcat /var/log/syslog.2.gz               # view compressed file contents
zless /var/log/syslog.2.gz              # page through compressed file contents
dmesg                                   # view kernel's ring buffer contents - good for hardware issues
dmesg -w                                # follow dmesg messages
dmesg --follow
```

## Network issues

Linux recognizes all network cards and connects your machine to a network if it can reach a DHCP server. Network problems generally fall into four categories.

### Routing issues

Test each destination endpoint one by one. Start by checking your routing table with `ip route`. If you can't reach a device in another subnet, check your routes. If you can't reach the internet, check the default gateway:

```bash
ip route                # view routing table to see default gateway
apt install traceroute
traceroute <ip>         # trace the hops to different IPs in the routing table
```

### DNS issues

If a host can't reach an internal or external host by name, DNS is likely the problem. Ping your router, then run `nslookup` on a domain. Run `resolvectl status` to identify your DNS server. If you suspect your internal DNS, switch to Google's DNS (`8.8.8.8` or `8.8.4.4`) to test. If that works, edit the forwarders section of the `bind9` config — this controls where it sends traffic it can't resolve from its internal host list. Use `dig` to retrieve the address (A) record from a DNS zone file:

```bash
nslookup <domain-name|ip-addr>      # if you can't resolve by domain name, DNS may be the issue
resolvectl status                   # get DNS server name and IP
cat /etc/netplan/<file>.yaml        # check netplan config for static IP issues
dig <hostname | ip-addr>            # get the A record for the domain
```

### Hardware issues

Linux supports most hardware, with each point release adding new driver support. If your network card isn't working, you may be missing a kernel module or driver. Get the hardware name with `lspci | grep -i net`, then search for it online to find the correct driver:

```bash
lspci | grep -i net     # get the hardware name to search for the correct driver
```

### DHCP issues

Always check `/var/log/syslog` for `dhcpd` events. Look for logs about an exhausted address pool or failed IP lease attempts. Run `tail -f /var/log/syslog` while reproducing the issue. Common causes include:

- Running out of IP addresses due to an overly generous lease time. One day is usually enough.
- The `isc-dhcp-server` daemon is not running.
- An invalid configuration.
- Host clocks out of sync. DHCP requests are timestamped, so large clock skew can confuse the server.

Use these commands to investigate:

```bash
cat /var/log/syslog | grep dhcpd    # search syslog for DHCP daemon events
tail -f /var/log/syslog             # follow syslog while reproducing the issue
```

## Resource issues

Resource issues include storage, memory, and CPU problems — too many files, a process hogging CPU time, or a server running out of memory.

### Storage

Check disk usage and inode count. Too many log files or a mail daemon can exhaust inodes. Use `ncdu` to pinpoint storage issues interactively, or `du` if `ncdu` isn't available. To force a filesystem check at the next boot, create an empty `forcefsk` file in the root directory of the filesystem you want to check. The server detects the file and checks the filesystem on startup:

```bash
df -h                               # disk usage, human-readable
df -i                               # inode count
ncdu -x /home                       # inspect home directory, limited to one filesystem
du -cksh * | sort -hr | head -15    # top 15 largest directories in current directory
sudo touch /forcefsk                # force check on root filesystem at next boot
sudo touch /partition-a             # force check on a specific partition at next boot
```

### Memory

Check how much memory and swap are available with `free -m`. Defective RAM is hard to detect but should be one of the first things you rule out. **Memtest86+** is a memory diagnostic available at boot. It can take hours, but errors usually appear within the first 15 minutes. If you find errors, identify the faulty memory module. Memory errors can corrupt data, so it's best to wipe the disk and start over:

```bash
free -m     # view memory and swap usage
```

### CPU

Check what is consuming CPU time, then consider adding more memory or tuning the app. Use `htop` to view per-process CPU and memory usage. If the load average is high but no single process stands out, run `iotop` — a bottleneck in disk reads or writes can slow everything down. Use the left and right arrow keys to sort by column. In both tools, press **F9** to kill a process:

```bash
htop            # view processes by CPU and memory usage
sudo iotop      # view disk reads and writes by process
```