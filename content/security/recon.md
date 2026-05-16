+++
title = 'Reconnaissance'
date = '2026-05-09T11:27:34-04:00'
weight = 10
draft = false
+++

Reconnaissance is the first phase of any attack or security assessment. Before exploiting a system, an attacker gathers as much information as possible about the target: its network layout, exposed services, software versions, and the people who manage it. The more an attacker knows, the more precisely they can strike.

Security professionals perform the same reconnaissance. Understanding what an attacker can discover about your systems lets you reduce your exposure before someone takes advantage of it.

## Passive vs. active

Reconnaissance falls into two categories:

*Passive reconnaissance* collects information without interacting directly with the target. You gather data from public sources: DNS records, WHOIS data, job postings, social media profiles, and archived web pages. The target never sees you.

*Active reconnaissance* interacts directly with the target. Port scans, banner grabs, and vulnerability probes send packets to the target's systems. This approach yields richer data but generates logs and may trigger alerts.

## What reconnaissance reveals

A thorough reconnaissance phase surfaces:

- Open ports and running services
- Software versions and known vulnerabilities
- Network topology and IP ranges
- Employee names, roles, and email formats
- Domain structure and subdomains
- Technologies in use (web frameworks, CMS, mail servers)

Defenders use this same lens to audit their own attack surface. If you can find it, an attacker can too.

## Creating target lists

### Generating IP addresses

#### `seq` and for loop

Generate consecutive IP addresses with `seq`:

```bash
#!/usr/bin/env bash

# Generate IP addresses from a given range
for ip in $(seq 1 254); do
    echo "172.16.10.${ip}" >> 172-16-10-hosts.txt    
done
```

#### `echo`, `sed`, and brace expansion

Without `sed`, `echo` prints each number on a single line separated by a space. `sed` replaces that space with a newline character:

```bash
echo 10.1.10.{1..254} | sed 's/ /\n/g'
```

#### `printf`

`printf` doesn't require piping to `sed` to replace the space with a newline:

```bash
printf "10.1.0.%d\n" {1..254}
```

## Subdomains

You can find a list of subdomains from GitHub gists. Use the following search query:

```
subdomain wordlist site: gist.github.com
```

## Host discovery

### `ping`

Sends ICMP echo requests to a host to test reachability and measure round-trip time. There are no options to run the command against multiple hosts, so use the following script that reads a list of hosts from a file and prints which ones respond to a ping. Pass the file path as the first argument:

```bash
#!/usr/bin/env bash

FILE="${1}"

while read -r host; do
	if ping -c 1 -W 1 -w 1 "${host}" &> /dev/null; then 
		echo "${host} is up"
	fi
done < "${FILE}"
```

The while loop drives the scan:

- `while read -r host; do`: reads one line from standard input and assigns it to `host`. The `-r` flag prevents backslash interpretation. The loop runs once per line until the file is exhausted.
- `if ping -c 1 -W 1 -w 1 "${host}" &> /dev/null; then`: pings the host once (`-c 1`), waits up to one second for a reply (`-W 1`), and exits after one second regardless of result (`-w 1`). All output goes to `/dev/null`. If ping exits with code 0, the host responded and the `if` block runs.
- `echo "${host} is up"`: prints the host to stdout.
- `done < "${FILE}"`: closes the loop and redirects the file as standard input, feeding one line at a time to `read`.

### Nmap

Nmap is a network scanner used for host discovery, port scanning, and service detection. The `-sn` flag disables port scanning so Nmap only checks whether hosts are up. This is called a *ping sweep*.

```bash
nmap -sn 172.16.10.0/24
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-16 06:42 -0400
Nmap scan report for 172.16.10.10
Host is up (0.000030s latency).
MAC Address: EA:21:3B:D3:BE:CD (Unknown)
Nmap scan report for 172.16.10.11
Host is up (0.000034s latency).
MAC Address: 1A:94:51:D8:E2:EE (Unknown)
Nmap scan report for 172.16.10.12
Host is up (0.000072s latency).
MAC Address: 86:5E:F8:1D:71:1B (Unknown)
Nmap scan report for 172.16.10.13
Host is up (0.000056s latency).
MAC Address: CA:3B:EE:2B:B8:5D (Unknown)
Nmap scan report for 172.16.10.1
Host is up.
Nmap done: 256 IP addresses (5 hosts up) scanned in 3.04 seconds
```

The command scans all 256 addresses in the `172.16.10.0/24` subnet. The `-sn` flag tells Nmap to skip port scanning and send only host discovery probes: ICMP echo requests, a TCP SYN to port 443, a TCP ACK to port 80, and an ICMP timestamp request.

Each block in the output represents a live host:

- `Nmap scan report for <IP>`: the address that responded.
- `Host is up (Xs latency)`: round-trip time for the discovery probe.
- `MAC Address`: the hardware address of the host's network interface, visible only when scanning a local subnet.

The final line summarizes the scan: total addresses checked, hosts found alive, and elapsed time.

#### Cleaner output

```bash
nmap -sn 172.16.10.0/24 | grep "Nmap scan" | awk -F'report for ' '{print $2}'
172.16.10.10
172.16.10.11
172.16.10.12
172.16.10.13
172.16.10.1
```

The command pipes Nmap's output through two filters to extract only the IP addresses.

`grep "Nmap scan"` keeps only lines that contain `Nmap scan`, which are the `Nmap scan report for <IP>` lines. All latency, MAC address, and summary lines are discarded.

`awk -F'report for ' '{print $2}'` splits each remaining line on the delimiter `report for ` and prints the second field. The second field is everything after that string: the IP address alone.

### arp-scan

`arp-scan` sends *Address Resolution Protocol (ARP)* requests to hosts on a network and displays responses. ARP maps IP addresses to MAC addresses at layer 2. Because ARP operates at layer 2, `arp-scan` only works on local networks. It requires `sudo` to open a raw socket.

The `--ouifile` and `--macfile` flags specify the vendor lookup files. Without them, `arp-scan` looks for the files in the current working directory and fails. Pass absolute paths to avoid this. See [Troubleshooting](#troubleshooting) for details.

#### Single host

```bash
sudo arp-scan 172.16.10.10 -I br_public \
    --ouifile=/usr/share/arp-scan/ieee-oui.txt \
    --macfile=/etc/arp-scan/mac-vendor.txt
Interface: br_public, type: EN10MB, MAC: de:06:27:4e:8b:01, IPv4: 172.16.10.1
Starting arp-scan 1.10.0 with 1 hosts (https://github.com/royhills/arp-scan)
172.16.10.10	ea:21:3b:d3:be:cd	(Unknown: locally administered)

1 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 1 hosts scanned in 0.194 seconds (5.15 hosts/sec). 1 responded
```

Scans a single IP address. Use this to confirm a specific host is up and retrieve its MAC address.

The header line shows the scanning interface, its MAC address, and its IPv4 address. Each result line contains the target IP, its MAC address, and the hardware vendor. The summary shows total hosts scanned, elapsed time, scan rate, and how many responded.

#### Subnet

```bash
sudo arp-scan 172.16.10.0/24 -I br_public \
    --ouifile=/usr/share/arp-scan/ieee-oui.txt \
    --macfile=/etc/arp-scan/mac-vendor.txt
Interface: br_public, type: EN10MB, MAC: de:06:27:4e:8b:01, IPv4: 172.16.10.1
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
172.16.10.10	ea:21:3b:d3:be:cd	(Unknown: locally administered)
172.16.10.11	1a:94:51:d8:e2:ee	(Unknown: locally administered)
172.16.10.12	86:5e:f8:1d:71:1b	(Unknown: locally administered)
172.16.10.13	ca:3b:ee:2b:b8:5d	(Unknown: locally administered)

4 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 1.983 seconds (129.10 hosts/sec). 4 responded
```

Scans all 256 addresses in the subnet. Pass the network address with host bits zeroed (`172.16.10.0`, not `172.16.10.10`) to avoid a warning.

#### File

```bash
sudo arp-scan -f /tmp/172-16-10-hosts.txt -I br_public \
    --ouifile=/usr/share/arp-scan/ieee-oui.txt \
    --macfile=/etc/arp-scan/mac-vendor.txt
Interface: br_public, type: EN10MB, MAC: de:06:27:4e:8b:01, IPv4: 172.16.10.1
Starting arp-scan 1.10.0 with 254 hosts (https://github.com/royhills/arp-scan)
172.16.10.10	ea:21:3b:d3:be:cd	(Unknown: locally administered)
172.16.10.11	1a:94:51:d8:e2:ee	(Unknown: locally administered)
172.16.10.12	86:5e:f8:1d:71:1b	(Unknown: locally administered)
172.16.10.13	ca:3b:ee:2b:b8:5d	(Unknown: locally administered)

4 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 254 hosts scanned in 1.964 seconds (129.33 hosts/sec). 4 responded
```

Reads target IP addresses from a file, one per line. The file must be readable by `nobody`. See [Troubleshooting](#troubleshooting).

#### Troubleshooting

**`Permission denied` for vendor files**

`arp-scan` drops all privileges to `nobody` after opening the raw socket. It then looks for `ieee-oui.txt` and `mac-vendor.txt` in the current working directory, not in their installed locations. `nobody` cannot read files in most users' working directories.

Fix: pass absolute paths with `--ouifile` and `--macfile`:

```bash
--ouifile=/usr/share/arp-scan/ieee-oui.txt \
--macfile=/etc/arp-scan/mac-vendor.txt
```

**`Permission denied` for the hosts file**

`arp-scan` opens the hosts file after dropping to `nobody`. If the file is inside a home directory with `700` permissions (`drwx------`), `nobody` cannot traverse the path to reach it.

Two fixes are available. Allow traversal of your home directory without exposing its contents:

```bash
chmod o+x /home/<user>
```

Or copy the file to a world-accessible location:

```bash
cp ~/scripts/files/172-16-10-hosts.txt /tmp/
```

**`WARNING: host part of X/Y is non-zero`**

You passed a host address instead of the network address in CIDR notation. Use the network address with host bits set to zero:

```bash
# Wrong
sudo arp-scan 172.16.10.10/24 ...

# Correct
sudo arp-scan 172.16.10.0/24 ...
```

`arp-scan` still scans the full subnet but logs the warning.


### Host monitoring script

The script continuously monitors a subnet for new hosts. When `arp-scan` finds a host not already in the known hosts file, it records it and sends an email alert.

```bash
#!/bin/bash

# sends a notification upon new host discovery
KNOWN_HOSTS="172-16-10-hosts.txt"
NETWORK="172.16.10.0/24"
INTERFACE="br_public"
FROM_ADDR="kali@blackhatbash.com"
TO_ADDR="security@blackhatbash.com"

while true; do 
  echo "Performing an ARP scan against ${NETWORK}..."
  sudo arp-scan -x -I ${INTERFACE} ${NETWORK} | while read -r line; do   # 1
    host=$(echo "${line}" | awk '{print $1}')                            # 2
    if ! grep -q "${host}" "${KNOWN_HOSTS}"; then                        # 3
      echo "Found a new host: ${host}!"
      echo "${host}" >> "${KNOWN_HOSTS}"                                 # 4
      sendemail -f "${FROM_ADDR}" \                                      # 5
        -t "${TO_ADDR}" \
        -u "ARP Scan Notification" \
        -m "A new host was found: ${host}"
    fi
  done 
  sleep 10
done
```

The outer `while true` loop runs the scan on a 10-second interval. The inner loop processes each line of `arp-scan` output:

1. Runs `arp-scan` with `-x` to suppress the header and footer lines, then pipes each result line into the inner loop.
2. Extracts the IP address from the first field of the `arp-scan` output line.
3. Checks whether the host already exists in the known hosts file. The `!` negates the condition so the block runs only for new hosts. `-q` suppresses grep output.
4. Appends the new host to the known hosts file so it isn't flagged again on the next scan.
5. Sends an email alert with the new host's IP address.

`sleep 10` pauses the outer loop for 10 seconds between scans.

## Port scanning

After you discover hosts, you can run a port scanner to discover their open ports and the services they are running.

### Nmap

Nmap is the most widely used port scanner. Use it to identify open ports, running services, and software versions on each host you discover.

By default, Nmap performs a *SYN scan* against the top 1,000 TCP ports. A SYN scan sends a SYN packet to each port and reads the response. An open port replies with SYN/ACK. A closed port replies with RST. A filtered port returns no response, which usually means a firewall is dropping the packets. Nmap reports three port states: `open`, `closed`, and `filtered`.

#### Single target

Scan a hostname or IP by passing it as the only argument:

```bash
nmap scanme.nmap.org
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-16 08:14 -0400
Nmap scan report for scanme.nmap.org (45.33.32.156)
Host is up (0.074s latency).
Other addresses for scanme.nmap.org (not scanned): 2600:3c01::f03c:91ff:fe18:bb2f
Not shown: 994 closed tcp ports (reset)
PORT      STATE    SERVICE
7/tcp     filtered echo
19/tcp    filtered chargen
22/tcp    open     ssh
80/tcp    open     http
9929/tcp  open     nping-echo
31337/tcp open     Elite

Nmap done: 1 IP address (1 host up) scanned in 8.01 seconds
```

`scanme.nmap.org` is Nmap's official public test host. The output lists each port with three columns: port number and protocol, state, and service name. The `Not shown:` line reports how many ports were suppressed from output and why. `closed tcp ports (reset)` means those ports replied with RST. The two `filtered` ports (7/tcp and 19/tcp) returned no response, likely blocked upstream.

Scanning a local IP produces the same format:

```bash
nmap 172.16.10.1
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-16 08:20 -0400
Nmap scan report for 172.16.10.1
Host is up (0.00020s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 1 IP address (1 host up) scanned in 4.94 seconds
```

`Not shown: 999 filtered tcp ports (no-response)` indicates nearly all ports on this host are firewalled.

#### Multiple targets

Pass space-separated hostnames or IPs to scan more than one target at once:

```bash
nmap localhost scanme.nmap.org
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-16 08:20 -0400
Nmap scan report for localhost (127.0.0.1)
Host is up (0.0000090s latency).
Other addresses for localhost (not scanned): ::1
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh

Nmap scan report for scanme.nmap.org (45.33.32.156)
Host is up (0.070s latency).
Other addresses for scanme.nmap.org (not scanned): 2600:3c01::f03c:91ff:fe18:bb2f
Not shown: 994 closed tcp ports (reset)
PORT      STATE    SERVICE
7/tcp     filtered echo
19/tcp    filtered chargen
22/tcp    open     ssh
80/tcp    open     http
9929/tcp  open     nping-echo
31337/tcp open     Elite

Nmap done: 2 IP addresses (2 hosts up) scanned in 5.55 seconds
```

Nmap prints a separate report for each host and a summary of total IPs scanned at the end.

#### Greppable output

The `-oG` flag writes results in a condensed, single-line-per-host format designed for parsing with `grep` and `awk`. Pass `-` as the filename to send output to stdout instead of a file.

```bash
nmap -iL files/172-16-10-hosts.txt --open -oG -
# Nmap 7.99 scan initiated Sat May 16 09:43:38 2026 as: /usr/lib/nmap/nmap --privileged -iL files/172-16-10-hosts.txt --open -oG -
Host: 172.16.10.1 ()	Status: Up
Host: 172.16.10.1 ()	Ports: 22/open/tcp//ssh///	Ignored State: filtered (999)
Host: 172.16.10.10 ()	Status: Up
Host: 172.16.10.10 ()	Ports: 8081/open/tcp//blackice-icecap///	Ignored State: closed (999)
Host: 172.16.10.11 ()	Status: Up
Host: 172.16.10.11 ()	Ports: 21/open/tcp//ftp///, 80/open/tcp//http///	Ignored State: closed (998)
```

Each host produces two lines. The `Status` line confirms the host responded to discovery probes. The `Ports` line lists open ports in the format `port/state/protocol//service///`. The `Ignored State` field reports how many ports were suppressed and their state. The leading comment line records scan metadata including the timestamp and command used.

Greppable output works well for extracting all hosts with a specific port open:

```bash
nmap -iL files/172-16-10-hosts.txt --open -oG - | grep "80/open"
```

#### XML output

The `-oX` flag writes results as XML. This format integrates with tools that consume nmap XML directly, such as Metasploit, Faraday, and Dradis. Pass `-` to send output to stdout.

```bash
nmap -iL files/172-16-10-hosts.txt --open -oX -
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE nmaprun>
<?xml-stylesheet href="file:///usr/share/nmap/nmap.xsl" type="text/xsl"?>
<!-- Nmap 7.99 scan initiated Sat May 16 09:44:29 2026 as: /usr/lib/nmap/nmap -&#45;privileged -iL files/172-16-10-hosts.txt -&#45;open -oX - -->
<nmaprun scanner="nmap" args="/usr/lib/nmap/nmap -&#45;privileged -iL files/172-16-10-hosts.txt -&#45;open -oX -" start="1778939069" startstr="Sat May 16 09:44:29 2026" version="7.99" xmloutputversion="1.05">
<scaninfo type="syn" protocol="tcp" numservices="1000" services="1,3-4,6-7,9,13,17,19-26,...65129,65389"/>
<verbose level="0"/>
<debugging level="0"/>
<hosthint><status state="up" reason="arp-response" reason_ttl="0"/>
<address addr="172.16.10.10" addrtype="ipv4"/>
<address addr="36:77:21:A5:02:6B" addrtype="mac"/>
<hostnames>
</hostnames>
</hosthint>
<hosthint><status state="up" reason="arp-response" reason_ttl="0"/>
<address addr="172.16.10.11" addrtype="ipv4"/>
<address addr="82:85:42:7E:2F:07" addrtype="mac"/>
<hostnames>
</hostnames>

```

The `<nmaprun>` element wraps the entire scan and records the command, start time, and nmap version. `<scaninfo>` describes the scan type, protocol, and the port range checked. Each `<hosthint>` element identifies a host detected during the pre-scan discovery phase, before full port results are available. It includes the host's IPv4 address, MAC address, and hostnames. Full port and service data appears in `<host>` elements in the complete output.

#### Service version detection

The `-sV` flag probes open ports to identify the software and version behind each one. Use `-iL` to read targets from a file:

```bash
nmap -sV -iL scripts/files/172-16-10-hosts.txt 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-16 08:29 -0400
Nmap scan report for 172.16.10.1
Host is up (0.00023s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.2p1 Debian 6 (protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for 172.16.10.10
Host is up (0.0000070s latency).
Not shown: 999 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
8081/tcp open  http    Werkzeug httpd 3.0.1 (Python 3.12.3)
MAC Address: EA:21:3B:D3:BE:CD (Unknown)

...
```

The output adds a `VERSION` column showing the detected application and version string. Use `Service Info` lines to identify the operating system.

To see only open ports across all scanned hosts, pipe through `grep`:

```bash
nmap -sV -iL scripts/files/172-16-10-hosts.txt | grep open
22/tcp open  ssh     OpenSSH 10.2p1 Debian 6 (protocol 2.0)
8081/tcp open  http    Werkzeug httpd 3.0.1 (Python 3.12.3)
21/tcp open  ftp     vsftpd 3.0.5
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
```

This strips all header, footer, and closed/filtered lines, leaving a flat list of every open port across the entire scan.

Or use `--open` to have Nmap filter the output itself:

```bash
nmap -sV -iL scripts/files/172-16-10-hosts.txt --open
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-16 08:33 -0400
Nmap scan report for 172.16.10.1
Host is up (0.00016s latency).
Not shown: 999 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.2p1 Debian 6 (protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for 172.16.10.10
Host is up (0.0000070s latency).
Not shown: 999 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
8081/tcp open  http    Werkzeug httpd 3.0.1 (Python 3.12.3)
MAC Address: EA:21:3B:D3:BE:CD (Unknown)

...

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 254 IP addresses (5 hosts up) scanned in 15.96 seconds
```

Unlike the `grep` approach, `--open` preserves the per-host structure and includes `Service Info` lines. It reports only hosts with at least one open port and omits closed and filtered results from each report.

### RustScan

RustScan is a fast port scanner written in Rust. It uses asynchronous I/O to scan ports significantly faster than Nmap. On Kali, `rustscan` runs as a Docker container. The command wraps `docker run` behind the scenes.

The typical workflow is to use RustScan to identify open ports quickly, then hand those results to Nmap for service detection.

If a previous scan was interrupted, the container may still exist. Remove it before rerunning:

```bash
docker rm -f rustscan
```

#### Basic scan

```bash
rustscan -a 172.16.10.0/24
...
Open 172.16.10.11:21
Open 172.16.10.1:22
Open 172.16.10.13:22
```

`-a` specifies the address or CIDR range to scan. The output prints one `Open IP:port` line for each open port found across the subnet.

#### Port range with greppable output

```bash
rustscan -g -a 172.16.10.0/24 -r 1-1024
```

`-r 1-1024` limits the scan to ports 1 through 1024 instead of all 65,535. `-g` switches to greppable output, which writes one line per host in the format `IP -> [port1, port2, ...]`. This format is easier to parse with standard text tools.

#### Parsing greppable output

Strip the `->` delimiter with `awk` to separate the IP from the port list:

```bash
rustscan -g -a 172.16.10.0/24 -r 1-1024 | awk -F'->' '{print $1, $2}'
```

`-F'->'` sets `->` as the field separator. `$1` is the IP address and `$2` is the port list in brackets.

To remove the brackets and produce plain output, pipe through `tr`:

```bash
rustscan -g -a 172.16.10.0/24 -r 1-1024 | awk -F'->' '{print $1, $2}' | tr -d '[]'
```

`tr -d '[]'` deletes every `[` and `]` character from the stream, leaving just the IP addresses and port numbers.


### Netcat

Netcat (`nc`) is a general-purpose network utility for reading and writing data across network connections. Use it for quick port checks when Nmap or RustScan are unavailable, or when you need a simple one-line scan of a small port range. Netcat is available on nearly every Unix-like system without installation.

The `-z` flag puts Netcat in *zero-I/O mode*: it connects to each port without sending data, then closes the connection. The `-v` flag enables verbose output so open ports appear in the results.

```bash
nc -zv 172.16.10.11 1-1024
172.16.10.11: inverse host lookup failed: Unknown host
(UNKNOWN) [172.16.10.11] 80 (http) open
(UNKNOWN) [172.16.10.11] 21 (ftp) open
```

The command scans ports 1 through 1024 on the target. Netcat reports only the ports it successfully connected to.

The `inverse host lookup failed` warning means nc could not resolve the IP to a hostname. It does not affect the scan. `(UNKNOWN)` in each result line reflects the same failed lookup. The port number, protocol name, and state follow in brackets.

### Organizing scan results

When you scan a large subnet, nmap returns hundreds of lines of output. Organizing that output lets you quickly identify all hosts running a specific service, feed targeted lists into downstream tools like Nikto or Hydra, or map your attack surface by service type.

Common strategies:

- *By port:* One file per open port, each listing the hosts that expose it. Feed `port-22.txt` directly into a brute-force tool without extra filtering.
- *By host:* One file per IP, listing that host's open ports. Useful for building a complete profile of individual targets.
- *By service version:* Group hosts by the banner nmap returns. Useful for finding all hosts running a specific vulnerable version.

The script uses the by-port strategy: it parses nmap output and writes each host's IP to a file named after the open port.

```bash
#!/bin/bash
HOSTS_FILE="/home/ryan/scripts/files/172-16-10-hosts.txt"
RESULT=$(nmap -iL ${HOSTS_FILE} --open | grep "Nmap scan report\|tcp open")   # 1

while read -r line; do
    if echo "${line}" | awk -q "report for"; then                             # 2
        ip=$(echo "${line}" | grep open | awk -F'/' '{print $1}')             # 3
    else
        port=$(echo "${line}" | grep open | awk -F'/' '{print $1}')           # 4
        file="port-${port}.txt"                                               # 5
        echo "${ip}" >> "${file}"                                             # 6
    fi
done <<< "${RESULT}"
```

The `while` loop processes each line of the filtered nmap output:

1. Runs nmap against every host in `HOSTS_FILE` (`-iL`), reports only open ports (`--open`), then filters output to keep only scan report headers (`Nmap scan report for`) and open port lines (`tcp open`).
2. Checks whether the current line is a scan report header. Note: `awk -q` is not a valid pattern-match flag; this should use `grep -q "report for"` instead.
3. Extracts the host IP from the report header and stores it in `ip`. Note: `grep open` matches nothing on a report header line; the correct extraction is `awk '{print $NF}'` to get the last field.
4. Extracts the port number from the port line. `awk -F'/' '{print $1}'` splits on `/` and returns the first field—for example, `80` from `80/tcp open http`.
5. Constructs a filename from the port number—for example, `port-80.txt`.
6. Appends the current IP to the port file.

`done <<< "${RESULT}"` feeds the entire `RESULT` variable into the loop as standard input using a here-string.

After the script runs, each `port-N.txt` file contains one IP per line: every host in the subnet with port N open.