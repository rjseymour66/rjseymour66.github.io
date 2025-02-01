---
title: "Troubleshooting your server"
linkTitle: "Troubleshooting"
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

Log files are rotated by `logrotate`:
- Files with `.gz` extension are log files that were compressed and renamed
- `zcat` to view compressed log files. Doesn't make you uncompress and open them
- `zless` to page through compressed file contents

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
```