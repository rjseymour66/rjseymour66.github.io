---
title: "Todo list"
linkTitle: "Todo"
# weight: 1000
# description:
---

1. Automated System Update and Cleanup Script  
   Updates the system, removes unused packages, and cleans up temporary files.
   Features: Logging, email notifications, and conditional execution.
   Commands: apt, dpkg, journalctl, mail.

2. Log File Monitoring and Alerting  
   Monitors log files (e.g., /var/log/syslog) for specific patterns (e.g., "error").
   Sends alerts (email or desktop notifications) if patterns are detected.
   Features: Uses tail -f, grep, and mail for real-time monitoring.

3. Automated User Management Script  
   Creates, deletes, or modifies users and groups.
   Supports bulk operations via a CSV file.
   Features: Integration with useradd, groupadd, and passwd.

4. Network Diagnostics and Troubleshooting Tool  
   Checks internet connectivity, DNS resolution, and measures latency to common servers.
   Generates a detailed report in a log file.
   Commands: ping, traceroute, dig, netstat.

5. Automated Backup and Restore Script  
   Backs up directories to a remote server or external drive.
   Features: Compression with tar or rsync, incremental backups, and scheduling with cron.
   Commands: tar, rsync, scp, and cron.

6. Firewall Configuration Script  
   Configures ufw (Uncomplicated Firewall) or iptables rules.
   Provides options to block IP ranges, allow services, and view current rules.
   Features: Validation of rules before application.

7. System Performance Analyzer  
   Gathers data on CPU, memory, disk I/O, and network throughput over a defined period.
   Generates a performance report with visual charts using gnuplot.
   Commands: top, iostat, sar, vnstat.

8. Service Uptime Monitor  
   Monitors critical services (e.g., nginx, mysql) and restarts them if they fail.
   Sends notifications about service status.
   Commands: systemctl, ps, mail.

9.  Disk Usage Analyzer  
    Analyzes disk usage and identifies large files or directories consuming space.
    Features: Recursive search with sorting, email notifications, and threshold alerts.
    Commands: du, sort, awk, find.

10. Custom Package Installation Script  
    Installs and configures a set of packages and services for a specific purpose (e.g., web server, database server).
    Includes custom configurations for packages like nginx, mysql, or docker.
    Features: Idempotency (runs without re-applying unchanged configurations).

11. Set up log aggregator service

## Server setup

1. Create swap file
2. Change hostname
3. Create user
   1. Add to sudoers
   2. Set password
   3. Set umask
4. Create bashrc
   1. Prompt
   2. alias
5. Set VIM as default editor
6. Setup SSH
   1. Keys
   2. Add to whichever servers
   3. No root login
   4. [Harden SSH](https://medium.com/@jasonrigden/hardening-ssh-1bcb99cd4cef)
7. Configure backup script