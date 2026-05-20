+++
title = 'Admin Labs'
date = '2026-05-17T08:29:39-04:00'
weight = 40
draft = false
+++

These lessons use the base GNS3 lab defined in the GNS3 network lab guide. Complete that
setup before starting here. Each lesson describes any topology additions needed beyond the
base lab.

## Base topology

All lessons start from the following unless noted otherwise:

- R1 — VyOS router
  - eth0: management interface, connected to a GNS3 NAT cloud (192.168.100.1/24)
  - eth1: trunk to SW1, with VLAN subinterfaces eth1.10 (10.0.10.1/24) and eth1.20
    (10.0.20.1/24)
- SW1 — GNS3 built-in Ethernet Switch
  - Port1: trunk to R1 (VLANs 10 and 20)
  - Port2: access VLAN 10, connected to PC1
  - Port3: access VLAN 10, connected to PC2
  - Port4: access VLAN 20, connected to PC3
- PC1 — Alpine Linux, 10.0.10.10/24, management eth1 at 192.168.100.11/24
- PC2 — Alpine Linux, 10.0.10.20/24, management eth1 at 192.168.100.12/24
- PC3 — Alpine Linux, 10.0.20.10/24, management eth1 at 192.168.100.13/24
- NAT cloud — connected to R1 eth0, PC1 eth1, PC2 eth1, and PC3 eth1

---

## Lesson 1: Verify baseline connectivity

### Problem

The lab is wired and configured, but you have not confirmed it actually works. Automation
tools, monitoring scripts, and every other lesson in this series depend on a functioning
network. Running automation against a broken environment produces misleading results —
you will spend hours debugging a script when the real problem is a misconfigured interface
or a missing route.

### Goal

Confirm that every host can reach its gateway, its VLAN peers, and hosts on the other
VLAN. Understand the forwarding path between VLANs and be able to explain each hop.

### Topology

This lesson uses the base lab. No additions needed.

### Success criteria

- PC1, PC2, and PC3 each reach their default gateway.
- PC1 and PC2 reach each other — same VLAN, no routing required.
- PC1 reaches PC3 — different VLANs, traffic must route through R1.
- You can identify every hop in the path from PC1 to PC3 and explain why each one
  appears.

---

## Lesson 2: Harden SSH across the lab

### Problem

Every device in the lab accepts password-based SSH logins. In production environments,
password authentication is disabled — passwords are easy to brute-force, hard to rotate
across many devices, and break automation scripts whenever they change. Key-based
authentication is the standard, and it is a prerequisite for the automation lessons that
follow.

### Goal

Migrate all four devices to SSH key-based authentication and disable password
authentication on each one.

### Topology

This lesson uses the base lab. No additions needed.

### Success criteria

- You can SSH from your workstation into R1, PC1, PC2, and PC3 without entering a
  password.
- Attempting to log in with a password is explicitly rejected on all four devices — not
  just ignored or timed out.

---

## Lesson 3: Capture and analyze VLAN traffic

### Problem

You understand that VLAN 10 and VLAN 20 traffic is separated at Layer 2, but you have
never seen the 802.1Q tags that enforce that separation. The trunk link between SW1 and
R1 carries tagged frames for both VLANs simultaneously — without the tag, R1 would not
know which subinterface to deliver each frame to. This lesson makes that tagging visible
in a live capture.

### Goal

Capture traffic on R1's trunk-facing interface while hosts generate cross-VLAN traffic.
Read the VLAN tags in the captured frames and confirm that the switch tags frames
correctly.

### Topology

This lesson uses the base lab. No additions needed.

### Success criteria

- You capture live frames on R1's trunk interface while PC1 pings PC3.
- You can identify the 802.1Q tag in the captured output and read the VLAN ID from it.
- You can point to a specific frame and explain why it carries that VLAN tag at that
  point in the topology — and where the tag is added and where it is removed.

---

## Lesson 4: Bash — automated configuration backup

### Problem

R1's configuration changes as you work through the lab. If you misconfigure something,
you need a way to recover the previous state. Without backups, your only option is to
rebuild from memory. In production, network configurations are the most critical artifact
to protect — a backup that runs once, manually, is not a backup strategy.

### Goal

Write a Bash script that retrieves R1's running configuration over SSH, saves it to a
timestamped file in a dedicated backup directory, and runs automatically on a schedule.

### Topology

This lesson uses the base lab. No additions needed.

### Success criteria

- A timestamped backup file for R1's running configuration exists in your backup
  directory after running the script.
- The filename includes the date and time so each backup is uniquely identified and
  alphabetically sortable.
- Running `crontab -l` shows an entry that executes the script every hour.
- A new file appears in the backup directory at the next scheduled interval without you
  running anything manually.

---

## Lesson 5: Configure and test static routes

### Problem

VyOS VLAN subinterfaces automatically install connected routes into the routing table —
you never had to think about routing at all. That automatic behavior hides how routing
actually works. Before studying OSPF or BGP, you need to understand what a router does
when it consults the routing table, what a next hop means, and what happens when a route
is absent.

### Goal

Rebuild inter-VLAN connectivity on R1 using explicit static routes. Observe the routing
table before and after, then break it deliberately and diagnose the failure.

### Topology

This lesson uses the base lab. Before starting, remove the VLAN subinterface
configuration from R1 so it no longer has automatic connected routes for the VLAN
subnets. The starting state is a router with configured interfaces but no routes to either
VLAN segment.

### Success criteria

- PC1 reaches PC3 with no VLAN subinterfaces on R1 — only static routes.
- You can read R1's routing table and explain which entry forwards each packet.
- After removing one static route, you can identify exactly which traffic breaks, explain
  why from the routing table, and restore connectivity.

---

## Lesson 6: Configure a zone-based firewall on R1

### Problem

VLAN 20 (clients) can currently reach any service on VLAN 10 (servers) without
restriction. In a production network, this is a security risk — a compromised client
machine can attack any server directly. The network needs access control between segments,
not just routing. Firewall rules on a router interface only filter traffic to and from the
router itself. Zone-policy enforces rules on traffic passing through the router between
segments.

### Goal

Configure VyOS zone-policy to restrict traffic from VLAN 20 to VLAN 10 so that clients
can ping servers but cannot establish TCP connections. Leave traffic in the opposite
direction unrestricted.

### Topology

This lesson uses the base lab. No additions needed.

### Success criteria

- PC3 (VLAN 20) can ping PC1 (VLAN 10).
- PC3 cannot establish a TCP connection to any port on PC1.
- PC1 can establish a TCP connection to PC3 without restriction.

---

## Lesson 7: Bash — interface monitoring

### Problem

If a link on R1 goes down silently, you will not know until a user reports something is
unreachable. At that point, you have lost time and have no record of when the failure
occurred. Proactive monitoring — even a simple polling script — gives you visibility and
a timestamp for the event. This is the pattern behind monitoring tools like Nagios and
Zabbix: a process checks device state on a fixed interval and fires an alert when
something is wrong.

### Goal

Write a Bash script that polls R1's interface state over SSH at a fixed interval, prints
a timestamped alert when any interface reports as down, and prints a normal status message
when all interfaces are up.

### Topology

This lesson uses the base lab. No additions needed.

### Success criteria

- The script runs continuously, printing a timestamped status line on each poll cycle.
- After you bring a link down on R1 using the GNS3 interface menu, the script prints a
  timestamped alert within one polling interval.
- After you restore the link, the script prints a recovery message on the next cycle.

---

## Lesson 8: Configure OSPF between two routers

### Problem

R1 has routes to VLAN 10 and VLAN 20 because its subinterfaces create them
automatically. But R2 has no way to reach those subnets, and R1 has no way to reach R2's
LAN — without static routes on both routers. Every time you add a subnet, every router
needs a manual update. OSPF eliminates that by letting routers discover and share routes
automatically.

### Goal

Configure OSPF area 0 on R1 and R2 so they exchange routes with no static routes between
them. R2 learns about VLAN 10 and VLAN 20 through OSPF, and R1 learns about R2's LAN
the same way.

### Topology

This lesson extends the base lab. Add the following to your existing GNS3 project:

- R2 — a second VyOS router
- PC4 — an Alpine Linux host connected to R2's LAN

Wire the additions as follows:

- R1 eth2 to R2 eth0 — point-to-point link, subnet 10.0.0.0/30 (R1 at .1, R2 at .2)
- R2 eth1 to PC4 — LAN subnet 10.0.30.0/24 (R2 at .1, PC4 at .10)

Configure interface addresses on R2 and a default gateway on PC4. Remove any static
routes on R1 pointing toward R2's subnet — those should be learned through OSPF.

### Success criteria

- R1 and R2 show each other as OSPF neighbors in FULL state.
- R2's routing table shows 10.0.10.0/24 and 10.0.20.0/24 as OSPF-learned routes, not
  static routes.
- PC4 can ping PC1 and PC3 with no static routes on R2.

---

## Lesson 9: Centralize syslog from all devices

### Problem

R1, PC1, PC2, and PC3 each write logs locally. Investigating an incident that spans
multiple devices means logging into each one separately, checking separate log files, and
manually correlating timestamps. That process is slow and error-prone. Centralized logging
sends all events to a single host in real time so you can search and correlate from one
place.

### Goal

Configure PC1 as a syslog server that collects log messages from all other devices in the
lab. Verify that a message generated on any device appears in PC1's log files, tagged
with the originating hostname.

### Topology

This lesson uses the base lab. No additions needed.

### Success criteria

- PC1 accepts incoming syslog messages on UDP port 514.
- A log event generated on PC2 or PC3 appears in PC1's log file, attributed to the
  correct source hostname.
- R1 forwards log messages at info level and above to PC1, and those messages appear in
  PC1's log file with R1's hostname.

---

## Lesson 10: Go — concurrent multi-device automation

### Problem

A Bash monitoring script SSHes into devices sequentially — connect, collect, disconnect,
then repeat for the next device. With four devices, total collection time is the sum of
all four SSH round trips. At scale, sequential polling is too slow. Go's goroutines let
you SSH into all devices simultaneously and collect results as they arrive, reducing
total time to roughly the slowest single session rather than the sum of all sessions.

### Goal

Write a Go program that SSHes into all four devices simultaneously using goroutines,
collects a status command from each, and writes the results to a report file. The program
must handle individual device failures without stopping — if one device is unreachable,
the remaining three must still complete.

### Topology

This lesson uses the base lab. No additions needed.

### Success criteria

- The program queries R1, PC1, PC2, and PC3 at the same time, not one after another.
- A report file is written containing the output from all reachable devices, labeled by
  device address.
- If you stop one device before running the program, the report still contains output
  from the other three, and the unreachable device appears with an error note rather than
  crashing the program.
- The program prints total elapsed time. It should be close to the time of a single SSH
  session — not four times that.
