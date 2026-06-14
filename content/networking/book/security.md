+++
title = 'Security'
date = '2026-06-14T09:01:22-04:00'
weight = 50
draft = false
+++

A fresh installation prioritizes functionality: every service the distribution ships starts automatically, default firewall policies pass all traffic, and SSH accepts password authentication from any source. This is reasonable during setup. It becomes a liability the moment the system connects to an untrusted network.

*Hardening* reduces a system's attack surface by removing or disabling what you don't need, restricting what you do need, and verifying that nothing remains open by accident. The underlying principle is *least privilege*: every service, user, and process should have exactly the access it needs and no more.

Linux defaults leave several categories of exposure:

- *Network exposure*: The default iptables and nftables policy is ACCEPT. All inbound and outbound traffic passes without restriction until you write rules.
- *Service exposure*: Package managers install software with its service enabled. A web server install starts Apache or Nginx immediately. Each running service is a potential entry point.
- *Authentication exposure*: OpenSSH allows password authentication and root login by default on many distributions. Password authentication is vulnerable to brute force. A successful attack against root grants immediate full control.
- *Kernel exposure*: Several kernel parameters ship permissive. ICMP redirects, source routing acceptance, and SYN flood protections are often not configured to their most restrictive values.

This section covers the common hardening steps for Linux hosts: disabling unnecessary services, configuring the firewall to default-deny, restricting SSH, and tuning kernel parameters that affect network security.

## CIS Critical Security Controls

The *CIS Critical Security Controls* are a prioritized set of 18 defensive actions published by the Center for Internet Security. They address the most common attack techniques and are organized by *implementation groups* (IGs) that reflect organizational maturity and resources:

- *IG1*: Essential cyber hygiene for any organization. Covers the safeguards every organization should implement regardless of size or resources.
- *IG2*: For organizations with dedicated IT staff and moderate risk exposure. Builds on IG1 with additional controls for managing sensitive data and more sophisticated attacks.
- *IG3*: For mature organizations with security expertise and high-risk profiles. Includes all 18 controls and their full safeguard sets.

The groups are cumulative: IG2 includes everything in IG1, and IG3 includes everything in IG2.

| #   | Control                                                | What it covers                                                                             |
| --- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------ |
| 1   | Inventory and control of enterprise assets             | Track all hardware connected to the network. You can't protect what you don't know exists. |
| 2   | Inventory and control of software assets               | Track all authorized software. Unauthorized programs are a primary malware vector.         |
| 3   | Data protection                                        | Classify, handle, retain, and dispose of data assets securely.                             |
| 4   | Secure configuration of enterprise assets and software | Harden default configurations on all devices, OSes, and applications.                      |
| 5   | Account management                                     | Manage credentials and authorization for user and service accounts.                        |
| 6   | Access control management                              | Create, assign, manage, and revoke access privileges across the environment.               |
| 7   | Continuous vulnerability management                    | Assess and remediate vulnerabilities continuously to reduce attacker opportunity.          |
| 8   | Audit log management                                   | Collect, alert on, review, and retain logs that support detection and recovery.            |
| 9   | Email and web browser protections                      | Defend against phishing and drive-by attacks targeting users.                              |
| 10  | Malware defenses                                       | Prevent installation, spread, and execution of malicious code.                             |
| 11  | Data recovery                                          | Maintain tested backups that restore systems to a known-good state.                        |
| 12  | Network infrastructure management                      | Track and harden routers, switches, and other network devices.                             |
| 13  | Network monitoring and defense                         | Monitor traffic for threats and respond to anomalies.                                      |
| 14  | Security awareness and skills training                 | Train users to recognize and respond to security threats.                                  |
| 15  | Service provider management                            | Assess third-party vendors that handle sensitive data or systems.                          |
| 16  | Application software security                          | Manage security across the lifecycle of developed and acquired software.                   |
| 17  | Incident response management                           | Maintain documented plans, roles, and procedures for responding to incidents.              |
| 18  | Penetration testing                                    | Test defenses by exploiting weaknesses before attackers do.                                |

### Hardware inventory

Before hardening a system, document what you're working with. These commands read from the kernel's virtual filesystem and require no additional tools.

```bash
cat /proc/cpuinfo        # CPU model, core count, clock speed, and supported feature flags
cat /proc/meminfo        # total and available RAM, swap usage, and kernel memory statistics
cat /etc/issue           # distribution name and version (the pre-login banner)
cat /proc/version        # kernel version and the compiler used to build it
uname -v                 # kernel version string with build date and time
uname -n                 # hostname of the system
sudo fdisk -l            # disk devices, partition tables, and sizes
sudo dmesg               # kernel ring buffer — hardware detection and driver messages at boot
sudo dmidecode           # BIOS/UEFI firmware table: motherboard, memory slots, chassis, and serial numbers
```

#### Inventory script

This script collects hardware, OS, network, CPU, memory, and disk information in a single pass and prints each category between separator lines. Run it as root so `dmidecode` and `fdisk` can access firmware and partition data.

```bash
#!/bin/bash

echo -n "Basic Inventory for Hostname: "
uname -n
#
echo ============================================
dmidecode | sed -n '/System Information/,+2p' | sed 's/\x09//'
dmesg | grep Hypervisor
dmidecode | grep "Serial Number" | grep -v "Not Specified" |
grep -v None
#
echo ============================================
echo "OS Information: "
uname -o -r
if [ -f /etc/redhat-release ]; then
    echo -n " "
    cat /etc/redhat-release
fi
if [ -f /etc/issue ]; then
    cat /etc/issue
fi
#
echo ============================================
echo "IP information: "
ip ad | grep inet | grep -v "127.0.0.1" | grep -v "::1/128" | tr -s " " | cut -d " " -f 3
echo ============================================
echo "CPU Information: "
cat /proc/cpuinfo | grep "model name\|MH\|vendor_id" | sort -r | uniq
echo -n "Socket Count: "
cat /proc/cpuinfo | grep processor | wc -l
echo -n "Core Count (Total): "
cat /proc/cpuinfo | grep cores | cut -d ":" -f 2 | awk '{sum+=$1} END {print sum}'
echo ============================================
echo "Memory Information: "
grep MemTotal /proc/meminfo | awk '{print $2,$3}'
echo ============================================
echo "Disk Information: "
fdisk -l | grep Disk | grep dev
```

Several parts use techniques worth understanding:

- `echo -n` combined with `uname -n`: The `-n` flag suppresses `echo`'s trailing newline so `uname -n` prints on the same line as the label.
- `sed -n '/System Information/,+2p'`: The `,+2` range prints the matching line plus the next two lines. The second `sed` strips `\x09` (the hex escape for tab) to remove the indentation `dmidecode` adds to its output.
- `dmesg | grep Hypervisor`: The kernel logs a line such as `Hypervisor detected: KVM` at boot when running in a VM. This grep returns the hypervisor type or nothing on bare metal.
- Chained `grep -v` for serial numbers: `dmidecode` uses `Not Specified` and `None` as placeholders when no serial number is set, which is common on VMs. The two `grep -v` calls strip these non-values so only real serial numbers appear.
- `tr -s " " | cut -d " " -f 3`: `ip addr` uses variable whitespace. `tr -s " "` squeezes runs of spaces into one so `cut -f 3` reliably extracts the IP/prefix field regardless of indentation.
- `grep processor | wc -l` vs. the `cores` awk block: The first counts logical processors including hyperthreads. The `awk '{sum+=$1}'` block sums physical cores per socket across all CPU packages, giving the true core count.
- `fdisk -l | grep Disk | grep dev`: `fdisk -l` mixes device lines, partition entries, and disk label info. The second `grep dev` isolates physical device lines (`/dev/sda`, `/dev/nvme0n1`) and drops everything else.

### Software inventory

Knowing what software is installed is the first step toward removing what you don't need. This maps to CIS Control 2. On Debian and Ubuntu systems, `dpkg` is the authoritative source for installed package data.

```bash
sudo apt list --installed | wc -l    # count of installed packages (includes one header line)
dpkg -l                              # all installed packages with version, architecture, and status
dpkg -L <package-name>               # every file installed by a specific package
```

On Red Hat family systems, use `rpm` instead:

```bash
rpm -qa                              # all installed packages
rpm -qi <package-name>               # detailed info for a specific package: version, size, description, and install date
```

### osquery

`osquery` exposes the operating system as a relational database you can query with standard SQL. It ships three components:

- *osqueryi*: An interactive SQL shell for ad-hoc queries. Results print to the terminal. No daemon required.
- *osqueryd*: A daemon that runs scheduled queries defined in a configuration file and writes results to a log for collection by a SIEM or monitoring tool.
- *osqueryctl*: A helper script that wraps `systemctl` to start, stop, and check the status of `osqueryd`.

Launch the interactive shell:

```bash
osqueryi
```

#### Shell commands

These commands control the osqueryi session rather than querying data:

```bash
.help       # list all available dot commands and options
.tables     # list every table available in the osquery schema
```

#### OS version

```sql
SELECT * FROM os_version;
```

Returns the OS name, version string, major, minor, and patch numbers, build identifier, platform family, and codename.

#### Network interfaces

```sql
SELECT interface, address, mask
FROM interface_addresses
WHERE interface NOT LIKE '%lo%';
```

Returns all non-loopback network interfaces with their IP address and subnet mask. The `NOT LIKE '%lo%'` filter excludes loopback addresses.

#### ARP cache

```sql
SELECT * FROM arp_cache;
```

Returns the local ARP table: each IP address mapped to its MAC address and the interface that learned it. Useful for spotting unexpected hosts or duplicate IPs on a segment.

#### Installed packages

```sql
SELECT * FROM deb_packages LIMIT 2;
```

Returns installed Debian package records including name, version, architecture, and install size. Use `rpm_packages` on Red Hat systems.

#### Recently started processes

```sql
SELECT pid, name
FROM processes
ORDER BY start_time DESC
LIMIT 10;
```

Returns the 10 most recently started processes by PID and name. Useful for spotting processes launched after a known-good baseline.

#### Process hashes and owners

```sql
SELECT DISTINCT h.sha256, p.name, u.username
FROM processes AS p
INNER JOIN hash AS h ON h.path = p.path
INNER JOIN users AS u ON u.uid = p.uid
ORDER BY start_time DESC
LIMIT 5;
```

Joins three tables to return the SHA256 hash of each running executable, the process name, and the username it runs as. The hash lets you verify binaries against known-good values. Processes running as unexpected users or with unrecognized hashes are worth investigating.
