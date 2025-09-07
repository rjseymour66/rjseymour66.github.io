---
title: "Troubleshooting your server"
linkTitle: "xTroubleshooting"
weight: 200
# description:
---

You need to know where to start troubleshooting, and some common techniques.

## Evaluating the scope

Determine where the problem is and how many systems and services are affected:
- Easy if you can reproduce the problem, but not easy if you can't
- How does each component in a network contribute to the problem?

Questions to ask:
- What are the symptoms?
- When did the problem start?
- Where there network changes?
  - Check DNS or DHCP server
- Has this happened before?
- What servers are impacted?
  - Single machine? Check login logs, then `.bash_history` for all commands executed. Or use `history` command after logging in as user
  - Use `w` to see who is logged into the system to get IP addr
- What users are impacted?
  
## Root cause analysis

After you resolve a problem, figure out how the problem started and how to prevent it from happening again:
- RCA is a learning experience
- details events that led to issue
  - which app or hardware it happened to
  - data and time first noticed
  - events, configurations, or faults that caused the issue
- List of steps to correct the issue
- Root cause analysis is often imperfect - you might be able to resolve the issue but never be 100% positive about why it happened

## System logs

Logs are a great first step in figuring out what happened:
- Two methods of viewing logs
  - systemd logs: `journalctl -u <service>` start with this
  - `/var/log/<file>.log` for services that don't use systemd
    - application logs are created by an app and not the distro
    - system logs are created by distro
- Some daemons have their own log files in `/var/log`, others use `/var/syslog`
  - DHCP server logs to `/var/log/syslog`
  - `grep` the contents of `/var/syslog`
- Use follow mode with `tail` (`tail -f`) while you reproduce any issues

### Important log files

Authorization Log - `/var/log/auth.log` for security issues:
- root access only
- includes authentication attempts from server and SSH
- lot of auth attempts might signal intrusion attempt

System Log - `/var/log/syslog` contains lots of different logging info
- If daemon doesn't have dedicated log file, uses syslog
- cron jobs are written here
- `dhclient` logs here (gets IP from DHCP server)
- `systemd init` daemon logs here

Packages - `/var/log/dpkg.log` contains info about installing and upgrading packages
- view if server acts weird after installs or updates

### Compressed logs

Log files are rotated by `logrotate`:
- Files with `.gz` extension are log files that were compressed and renamed
- `zcat` to view compressed log files. Doesn't make you uncompress and open them
- `zless` to page through compressed file contents

### Commands

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

### Connectivity

Linux recognizes all network cards and will connect your machine to a network if it can access a DHCP server:
- Routing issues: test each destination endpoint one-by-one
  - First, check your routing table with `ip route`
  - Maybe can't access a device in another subnet
  - Cannot connect to the internet - check default gateway
- DNS issues: you can tell this is the issue if a host cannot access an internal or external host by name
  - If its internal, its your DNS. External, ISP DNS
  - ping your router, then `nslookup` a domain
  - `resolvectl status` to find DNS server
  - to see if it is your internal DNS, switch your DNS server to use Google's DNS (`8.8.8.8` or `8.8.4.4`)
    - edit the forwarders section of the `bind9` config daemon - this is where it sends traffic if it can't resolve your request based on an internal list of hosts
  - `dig` command resturns info about the address (A) record of the DNS zone file
- Hardware issues: Linux should support your hardware, and each point release adds support for new hardware features
  - Might be missing critical kernel module/driver. To fix, google "<hardware-name> Ubuntu". Get <hardware-name> with `lspci | grep -i net`
- DHCP might be the issue:
  - Always check `/var/log/syslog` for `dhcpd` events
    - Look for logs about an exhausted pool
    - Look for attempts for machines to get an IP address
    - `tail -f /var/log/syslog` can be helpful
  - Common DHCP issues
    - Ran out of IP addresses - maybe because lease time is too generous. Lease time of 1 day is enough
    - `isc-dhcp-server` daemon is not running
    - invalid config
    - host clocks out of sync - DHCP requests are timestamped. If clocks are off by a lot, the DHCP server might be confused

```bash
# --- Routing issues --- #
ip route                                # 1. view routing table to see default gateway
apt install traceroute      
traceroute <ip>                         # 2. trace the hops to different ips in routing table

# --- DNS issues --- #
nslookup <domain-name|ip-addr>          # if you can't resolve by domain name, it might be DNS
resolvectl status                       # get DNS server name and IP
cat /etc/netplan/<file>.yaml            # static IP - check netplan config
dig <hostname | ip-addr>                # get the A record for the domain

# --- Hardware issues --- #
lspci | grep -i net                     # get name of hardware that might require kernel driver

# --- DHCP issues --- #
cat /var/log/syslog | grep dhcpd        # search syslog for daemon issues
tail -f /var/log/syslog                 # follow syslog while you try to get an IP
```

## Resource issues

Resources include CPU, memory, disk, IO, etc:
- too many files, process hogging CPU, server running out of memory
- Storage: check the disk usage and inode count
  - Too many log files or mail daemon might use too many inodes
  - `ncdu` command to pinpoint storage issues with interactive screen
  - use `du` if you can't use `ncdu`
  - Manually force a filesystem check at the next boot. Can detect fs corruptions. Just create empty file `forcefsk` in root dir of the fs you want to check and server will detect the file and check the fs at the next boot
- Memory: check how much memory and swap are available
  - `free -m` is best bet
  - Very hard to troubleshoot defective RAM, usually first thing you troubleshoot because its hard to detect
  - **Memtest86+** is a memory test available during boot. Can take hours, depending on how much memory you have
    - Errors usually show up within the first 15 minutes
    - If you get errors, you need to pinpoint which memory module is at fault
  - If you get memory errors, its best to erase the hard disk and start over bc it can lead to corrupted data
- CPU: Check to see what is consuming CPU time, and then maybe add more memory to the server or tune the app to use less memory
  - `htop` helps view CPU usage, such as PERCENT_CPU and PERCENT_MEM
  - `iotop` shows how much data is written to or read from your disks. Good to troubleshoot if your CPU load average is high but there is no specific process consuming too much CPU
    - bottleneck in data reads or writes can slow everything down
    - use left and right buttons on keyboard to sort by column
  - for both tools, press F9 to kill the processes

```bash
# --- Storage --- #
df -h                               # disk usage - human readable
df -i                               # inode count
ncdu -x /home                       # inspect home dir. -x limits command to one fs
du -cksh * | sort -hr | head -15    # top 15 larges directories in cwd
sudo touch /forcefsk                # check root fs
sudo touch /partition-a             # check fs on partitiona

# --- Memory/RAM --- #
free -m                             # view memory and swap usage

# --- CPU --- #
htop                                # view processes hogging CPU
sudo iotop                          # view reads and writes
```