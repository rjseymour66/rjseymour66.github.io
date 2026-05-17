+++
title = 'Seed Data'
date = '2026-05-17T09:03:40-04:00'
weight = 60
draft = false
+++

Seeding a lab means putting realistic data, traffic, and state onto devices before you
start a lesson or test a script. An empty lab with no traffic, no logs, and no files does
not exercise monitoring or automation the way a real environment does. This page covers
the most common seeding methods for Alpine Linux hosts and VyOS routers in GNS3.

All host examples assume Alpine Linux. Install packages with `apk add <package>`.

---

## Test files

Use test files when you need something to transfer, a disk to fill, or a payload for
bandwidth testing.

### dd

`dd` reads from a source and writes to a destination at a specified block size. Reading
from `/dev/zero` produces null bytes; reading from `/dev/urandom` produces random data.

```bash
# 10 MB file of zeros
dd if=/dev/zero of=/tmp/test-10m.bin bs=1M count=10

# 100 MB file of random data (slower, but more realistic for compression tests)
dd if=/dev/urandom of=/tmp/test-100m.bin bs=1M count=100
```

Use `/dev/zero` when transfer speed is what you are measuring. Use `/dev/urandom` when
you need data that does not compress, such as when testing encrypted transfers.

### truncate

`truncate` creates a *sparse file* — the filesystem records the size without actually
writing bytes to disk. Sparse files appear as the target size but consume almost no
storage. Use them when you need a large apparent file size without filling the disk.

```bash
# Create a 1 GB sparse file instantly
truncate -s 1G /tmp/sparse-1g.bin
```

Sparse files are useful for filling a directory listing or testing tools that read file
metadata, but they do not generate real I/O during transfers.

### Multiple files

```bash
# Create 50 files of varying sizes in a directory
mkdir -p /tmp/testdata
for i in $(seq 1 50); do
  dd if=/dev/zero of=/tmp/testdata/file-${i}.dat bs=1K count=$((RANDOM % 512 + 1)) 2>/dev/null
done
```

---

## Network traffic

### iperf3

`iperf3` measures achievable throughput between two hosts. One host runs as the server,
the other as the client.

```bash
# Install on Alpine
apk add iperf3

# On the receiving host (server mode)
iperf3 -s

# On the sending host (client mode) — runs for 30 seconds
iperf3 -c 10.0.10.10 -t 30

# UDP test at a target rate of 5 Mbit/s (useful for QoS lessons)
iperf3 -c 10.0.10.10 -u -b 5M -t 30
```

Use `iperf3` when testing traffic shaping, QoS policies, or WAN bandwidth limits. The
`-u` flag switches to UDP, which is necessary for DSCP marking tests because UDP does not
retransmit.

### Ping

```bash
# Ping every 200ms for 60 seconds, record results to a file
ping -i 0.2 -c 300 10.0.20.10 | tee /tmp/ping-results.txt

# Flood ping — sends as fast as possible (requires root)
ping -f -c 10000 10.0.20.10
```

Run a sustained ping in the background while generating bulk traffic to observe the
latency impact — this is the core measurement in the QoS lesson.

### netcat

`netcat` (`nc`) opens raw TCP or UDP connections. Use it to test whether a port is
reachable, simulate a simple service, or transfer data between hosts.

```bash
# On the receiving host — listen on port 9000
nc -lk 9000 > /dev/null

# On the sending host — send 100 MB of zeros to the listener
dd if=/dev/zero bs=1M count=100 | nc 10.0.10.10 9000

# Test whether a port is open (exits 0 if reachable)
nc -zv 10.0.10.10 22
nc -zv 10.0.10.10 80
```

Use `netcat` to verify that firewall rules are working: if a zone-policy blocks TCP,
`nc -zv` will fail even when `ping` succeeds.

### curl

```bash
# Install on Alpine
apk add curl

# Single request
curl -s http://10.0.2.10/

# Loop — send one request per second for 5 minutes
for i in $(seq 1 300); do
  curl -s -o /dev/null -w "%{http_code} %{time_total}s\n" http://10.0.2.10/
  sleep 1
done

# Download a file and measure transfer speed
curl -o /tmp/download.bin http://10.0.2.10/testfile.bin
```

HTTP loops are useful for testing NAT translations, DMZ access policies, and basic
monitoring scripts that check service availability.

### wget

```bash
apk add wget

# Repeatedly download a file in the background
while true; do
  wget -q -O /dev/null http://10.0.2.10/test-10m.bin
  sleep 5
done &
```

Running `wget` in a loop in the background simulates persistent client traffic while you
work on other parts of the lab.

---

## Services

### Python HTTP server

Python's built-in HTTP server serves the current directory. Use it to create an instant
web target for `curl`, `wget`, or browser tests.

```bash
apk add python3

# Serve the current directory on port 80 (requires root for port < 1024)
cd /tmp/testdata
python3 -m http.server 80

# Serve on an unprivileged port
python3 -m http.server 8080
```

### netcat listener

```bash
# Listen on port 9000, restart automatically after each connection
while true; do nc -l 9000; done
```

### SSH

SSH is already running on Alpine hosts. Use it to test automation scripts, firewall
rules, and key-based authentication without installing anything extra.

```bash
# Verify SSH is running
ps aux | grep sshd

# Start SSH if it is not running (Alpine uses OpenRC)
rc-service sshd start
```

---

## Log entries

The `logger` command sends a message to the local syslog daemon, which then forwards it
wherever syslog is configured to send it — including a remote syslog server.

```bash
# Send a single info-level message
logger -p user.info "Test message from PC2"

# Send a message with a custom tag (appears as the program name in logs)
logger -t myapp -p user.warning "Disk usage above 80 percent"

# Send a message that looks like an authentication event
logger -t sshd -p auth.info "Accepted publickey for admin from 10.0.10.1 port 54321"

# Generate 20 log messages with a counter
for i in $(seq 1 20); do
  logger -t labtest -p user.info "Event number ${i} from $(hostname)"
  sleep 1
done
```

Use `logger` to trigger your syslog collection scripts and verify that remote log
forwarding works end-to-end before running a full lesson.

### Verify delivery

```bash
# Watch incoming logs in real time
tail -f /var/log/messages

# Filter for messages from a specific host
grep "PC2" /var/log/messages

# Count messages received from each host
grep -oP 'from \S+' /var/log/messages | sort | uniq -c
```

---

## Users and host state

### Add users

```bash
# Add a regular user
adduser -D alice
echo "alice:password123" | chpasswd

# Add a user with a home directory and specific shell
adduser -D -h /home/bob -s /bin/sh bob
echo "bob:password123" | chpasswd
```

### /etc/hosts

Populating `/etc/hosts` simulates DNS resolution without running a DNS server. Automation
scripts that connect by hostname rather than IP will use these entries.

```bash
echo "10.0.10.10 pc1 pc1.lab.local" >> /etc/hosts
echo "10.0.10.20 pc2 pc2.lab.local" >> /etc/hosts
echo "10.0.20.10 pc3 pc3.lab.local" >> /etc/hosts
echo "192.168.100.1 r1 r1.lab.local"  >> /etc/hosts
```

### Populate shell history

Some automation tests parse shell history. Seed it so the file is not empty.

```bash
cat >> ~/.ash_history << 'EOF'
ip addr show
ping -c 3 10.0.10.1
ssh vyos@192.168.100.1
cat /var/log/messages
df -h
EOF
```

---

## VyOS configs

### Save and restore

After configuring R1 correctly, save a copy of the configuration to your workstation.
This is your restore point for any lesson that requires a clean starting state.

```bash
# From your workstation — copy R1's config to a local file
ssh vyos@192.168.100.1 'show configuration commands' > r1-baseline.conf

# Restore the config by piping commands back to VyOS
ssh vyos@192.168.100.1 'configure' < r1-baseline.conf
```

The `show configuration commands` output is a flat list of `set` commands that can be
piped directly into a VyOS configuration session.

### Load from file

VyOS can load a configuration file from disk using the built-in `load` command. Copy the
file to R1 first, then load it.

```bash
# Copy a config file to R1
scp r1-baseline.conf vyos@192.168.100.1:/tmp/

# On R1 — load and commit
configure
load /tmp/r1-baseline.conf
commit
save
exit
```

### Factory reset

Use this when a lesson left the router in an unknown state and you want a clean start.

```bash
# On R1's console
configure
load /opt/vyatta/etc/config.boot.default
commit
save
exit
```

---

## Interface events

GNS3 lets you bring links up and down without touching device configs, which is useful
for monitoring lessons. You can also do this from a VyOS console to test scripts.

```bash
# On R1 — bring an interface down
configure
set interfaces ethernet eth2 disable
commit

# Bring it back up
delete interfaces ethernet eth2 disable
commit
exit
```

On Linux hosts, use `ip link` to simulate interface failures:

```bash
# Bring eth0 down
ip link set eth0 down

# Bring it back up
ip link set eth0 up
```

---

## Seeding by lesson

| Lesson type | What to seed before starting |
|-------------|------------------------------|
| Connectivity verification | Files in `/tmp`, entries in `/etc/hosts` |
| Firewall / zone-policy | A `netcat` listener on the target host, `curl` on the source |
| Traffic shaping / QoS | `iperf3` server on Server, sustained `ping` from PC1 |
| Syslog collection | `logger` loop running on PC2 and PC3 |
| Monitoring scripts | Interface brought down on R1, `logger` events firing |
| Backup scripts | A known good config committed and saved on R1 |
| Troubleshooting | All faults injected, consoles closed, hosts unable to ping |
| Go / Bash automation | SSH keys deployed, services running, hosts reachable |
