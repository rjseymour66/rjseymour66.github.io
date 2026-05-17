+++
title = 'GNS3 network lab'
date = '2026-05-16T09:00:00-04:00'
weight = 30
draft = false
+++

*GNS3* is a network emulator that runs real network OS images inside virtual machines. This lab
gives you a working environment with a router, a switch, and Linux hosts to practice routing,
VLAN segmentation, and SSH-based automation with Go and Bash.

The router runs *VyOS*, a free, open-source network OS with SSH and a REST API that matches what
you encounter in production environments.

## Hardware requirements

Your host machine needs the following to run this lab:

| Component | Minimum                      | Recommended |
| --------- | ---------------------------- | ----------- |
| RAM       | 8 GB                         | 16 GB       |
| CPU       | 4 cores (VT-x/AMD-V enabled) | 8 cores     |
| Disk      | 20 GB free                   | 50 GB free  |

Enable hardware virtualization (VT-x on Intel, AMD-V on AMD) in your BIOS before installing GNS3.

The lab allocates these resources across its virtual devices:

| Device             | RAM       | vCPUs |
| ------------------ | --------- | ----- |
| GNS3 VM            | 2 GB      | 2     |
| R1 (VyOS)          | 512 MB    | 1     |
| PC1 (Alpine Linux) | 128 MB    | 1     |
| PC2 (Alpine Linux) | 128 MB    | 1     |
| PC3 (Alpine Linux) | 128 MB    | 1     |
| **Total**          | **~3 GB** | **6** |

## Software prerequisites

- GNS3 2.2 or later
- GNS3 VM (VMware Workstation or VirtualBox)
- VyOS rolling release image in GNS3 appliance format
- Alpine Linux GNS3 appliance (or VPCS nodes for basic ping testing)

## Lab topology

```
          [R1: VyOS]
          eth1 (trunk)
               |
        [SW1: L2 Switch]
       /        |        \
   Port2      Port3      Port4
  VLAN 10    VLAN 10    VLAN 20
     |          |          |
   [PC1]      [PC2]      [PC3]
   eth0       eth0       eth0
```

### Devices

| Device | OS                   | Role                                 |
| ------ | -------------------- | ------------------------------------ |
| R1     | VyOS                 | Router, inter-VLAN gateway           |
| SW1    | GNS3 Ethernet Switch | Layer 2 switching, VLAN segmentation |
| PC1    | Alpine Linux         | End host, VLAN 10                    |
| PC2    | Alpine Linux         | End host, VLAN 10                    |
| PC3    | Alpine Linux         | End host, VLAN 20                    |

### VLAN assignments

| VLAN | Name    | Subnet       | Gateway   |
| ---- | ------- | ------------ | --------- |
| 10   | servers | 10.0.10.0/24 | 10.0.10.1 |
| 20   | clients | 10.0.20.0/24 | 10.0.20.1 |

## Management network options

Your Go and Bash scripts need a reliable path to SSH into devices. Two approaches exist.

### Option 1: Out-of-band management (recommended)

A dedicated management network connects your workstation to every device's management interface
through a GNS3 NAT cloud node. Your scripts always reach devices at stable `192.168.100.x`
addresses, even when the data-plane topology is misconfigured during practice.

This approach matches the standard used in enterprise network automation environments. If you
break routing on the data plane, you can still SSH in and fix it.

**Management IP addresses:**

| Device | Interface | Management IP     |
| ------ | --------- | ----------------- |
| R1     | eth0      | 192.168.100.1/24  |
| PC1    | eth1      | 192.168.100.11/24 |
| PC2    | eth1      | 192.168.100.12/24 |
| PC3    | eth1      | 192.168.100.13/24 |

**Full topology with out-of-band management:**

```
  [NAT Cloud: 192.168.100.0/24]
   |         |        |       |
  R1        PC1      PC2     PC3
eth0       eth1     eth1    eth1
.1         .11      .12     .13

          [R1: VyOS]
          eth1 (trunk)
               |
        [SW1: L2 Switch]
       /        |        \
   Port2      Port3      Port4
  VLAN 10    VLAN 10    VLAN 20
     |          |          |
   [PC1]      [PC2]      [PC3]
   eth0       eth0       eth0
```

### Option 2: In-band management

Scripts reach devices through the same interfaces that carry VLAN traffic. This is simpler to
set up since you don't need a separate NAT node or second NICs on each host. The trade-off:
misconfigure a route and your scripts lose access until you fix it through the GNS3 console.

Use this approach when you want to practice troubleshooting scenarios where restoring
connectivity is the exercise.

Scripts connect to R1 at `10.0.10.1` from VLAN 10 hosts, or `10.0.20.1` from PC3.

## Install GNS3

#### Ubuntu/Debian

```bash
sudo add-apt-repository ppa:gns3/ppa
sudo apt update
sudo apt install gns3-gui gns3-server
```

Select **Yes** when the installer asks whether to allow non-root users to run GNS3.

#### Fedora/RHEL

GNS3 is not in the default repositories. Download the AppImage from gns3.com, then make it
executable and launch it:

```bash
chmod +x GNS3-*.AppImage
./GNS3-*.AppImage
```

## Import VyOS into GNS3

1. Download the VyOS GNS3 appliance from the VyOS community site. Select the image labeled
   **GNS3** and download both the `.gns3a` appliance file and the `.qcow2` disk image.
2. In GNS3, open **File > Import appliance**.
3. Select the `.gns3a` file and follow the import wizard.
4. When prompted, select the `.qcow2` disk image you downloaded.

VyOS appears in the **Routers** category in your GNS3 device library.

## Build the topology

1. Create a new project: **File > New blank project**.
2. Drag a **VyOS** node onto the canvas and rename it `R1`.
3. Drag an **Ethernet Switch** node onto the canvas and rename it `SW1`.
4. Drag three **Alpine Linux** nodes and rename them `PC1`, `PC2`, and `PC3`.
5. Connect the devices:
   - R1 `eth1` → SW1 `Port1`
   - SW1 `Port2` → PC1 `eth0`
   - SW1 `Port3` → PC2 `eth0`
   - SW1 `Port4` → PC3 `eth0`
6. If you chose out-of-band management, add a **NAT** node and connect:
   - R1 `eth0` → NAT
   - PC1 `eth1` → NAT
   - PC2 `eth1` → NAT
   - PC3 `eth1` → NAT

## Configure SW1

Right-click SW1 and select **Configure**. Set each port:

| Port  | Mode   | VLAN   |
| ----- | ------ | ------ |
| Port1 | Trunk  | 10, 20 |
| Port2 | Access | 10     |
| Port3 | Access | 10     |
| Port4 | Access | 20     |

The built-in GNS3 switch has no SSH and is configured only through the GUI. This is intentional
for a first lab. Once you're comfortable with the basics, replace it with a second VyOS instance
configured as a switch to practice SSH-based switch automation.

## Configure R1 (VyOS)

Start R1 and open its console. Log in with username `vyos` and password `vyos`.

Enter configuration mode:

```bash
configure
```

Set the hostname:

```bash
set system host-name R1
```

#### Configure inter-VLAN routing

R1's `eth1` connects to SW1 as a trunk. Create a VLAN *subinterface* for each VLAN—R1 routes
traffic between them:

```bash
set interfaces ethernet eth1 vif 10 address '10.0.10.1/24'
set interfaces ethernet eth1 vif 10 description 'VLAN10-servers'
set interfaces ethernet eth1 vif 20 address '10.0.20.1/24'
set interfaces ethernet eth1 vif 20 description 'VLAN20-clients'
```

#### Enable SSH

```bash
set service ssh port '22'
set service ssh listen-address '0.0.0.0'
```

#### Configure the management interface (out-of-band only)

Skip this step if you chose in-band management.

```bash
set interfaces ethernet eth0 address '192.168.100.1/24'
set interfaces ethernet eth0 description 'management'
```

Commit and save the configuration, then exit configuration mode:

```bash
commit
save
exit
```

## Configure the Linux hosts

Run these commands on each host's console. Skip the management interface line if you chose
in-band management.

#### PC1

```bash
ip addr add 10.0.10.10/24 dev eth0
ip route add default via 10.0.10.1
ip addr add 192.168.100.11/24 dev eth1
```

#### PC2

```bash
ip addr add 10.0.10.20/24 dev eth0
ip route add default via 10.0.10.1
ip addr add 192.168.100.12/24 dev eth1
```

#### PC3

```bash
ip addr add 10.0.20.10/24 dev eth0
ip route add default via 10.0.20.1
ip addr add 192.168.100.13/24 dev eth1
```

## Verify connectivity

Run these commands from PC1 to confirm the data plane works end-to-end:

```bash
# Gateway reachable
ping -c 3 10.0.10.1

# PC2 reachable on the same VLAN
ping -c 3 10.0.10.20

# PC3 reachable across VLANs — confirms inter-VLAN routing
ping -c 3 10.0.20.10
```

SSH into R1 to confirm management access:

```bash
# Out-of-band — from your workstation
ssh vyos@192.168.100.1

# In-band — from PC1 or PC2
ssh vyos@10.0.10.1
```

## Automate with Bash

Collect interface status from R1:

```bash
#!/usr/bin/env bash

ROUTER="192.168.100.1"  # use 10.0.10.1 for in-band
USER="vyos"

ssh -o StrictHostKeyChecking=no "${USER}@${ROUTER}" 'show interfaces'
```

Loop across all hosts and print their IP addresses:

```bash
#!/usr/bin/env bash

HOSTS=("192.168.100.11" "192.168.100.12" "192.168.100.13")
USER="root"

for host in "${HOSTS[@]}"; do
  printf '=== %s ===\n' "${host}"
  ssh -o StrictHostKeyChecking=no "${USER}@${host}" \
    'ip addr show eth0 | grep inet'
done
```

## Automate with Go

This program SSHes into R1 and runs a command. It uses the `golang.org/x/crypto/ssh` package.

Install the package:

```bash
go get golang.org/x/crypto/ssh
```

```go
package main

import (
    "fmt"
    "log"
    "os"

    "golang.org/x/crypto/ssh"
)

func sshRun(host, user, password, cmd string) error {
    config := &ssh.ClientConfig{
        User: user,
        Auth: []ssh.AuthMethod{
            ssh.Password(password),
        },
        HostKeyCallback: ssh.InsecureIgnoreHostKey(), // replace with known_hosts verification in production
    }

    client, err := ssh.Dial("tcp", host+":22", config)
    if err != nil {
        return fmt.Errorf("connect: %w", err)
    }
    defer client.Close()

    session, err := client.NewSession()
    if err != nil {
        return fmt.Errorf("session: %w", err)
    }
    defer session.Close()

    session.Stdout = os.Stdout
    session.Stderr = os.Stderr
    return session.Run(cmd)
}

func main() {
    if err := sshRun("192.168.100.1", "vyos", "vyos", "show interfaces"); err != nil {
        log.Fatal(err)
    }
}
```

## Lessons

Complete these lessons in order. Each one builds on the previous and moves you from basic
connectivity toward real automation and network operations tasks.

---

### Lesson 1: Verify baseline connectivity

Your lab is wired and configured. Before writing any automation, you need to confirm the
network works end-to-end. Automation against a broken environment produces misleading
results, and you will waste hours debugging scripts when the real problem is a
misconfigured interface or missing route.

#### What to accomplish

- Confirm each host can reach its default gateway.
- Confirm hosts on the same VLAN can reach each other.
- Confirm a host on VLAN 10 can reach a host on VLAN 20, which requires traffic to pass
  through R1.
- Read the routing table on each host and understand which routes are present and why.
- Trace the forwarding path from PC1 to PC3 and identify each hop it crosses.

#### You're done when

All hosts can reach their gateway, each other within the same VLAN, and hosts across
VLANs. The path from PC1 to PC3 passes through R1.

---

### Lesson 2: Harden SSH across the lab

Your devices currently accept password-based SSH logins. In production environments,
password auth is disabled — it exposes credentials and breaks automation scripts when
passwords change. This lesson migrates every device to key-based authentication and
hardens the SSH daemon.

#### What to accomplish

- Generate an ED25519 SSH key pair on your workstation.
- Deploy your public key to R1 and all three hosts so key-based logins work.
- Disable password authentication on each device's SSH daemon so only key-based logins
  are accepted.
- Verify that SSH sessions from your workstation succeed without entering a password.
- Confirm that a login attempt using a password is explicitly rejected.

#### You're done when

You can SSH into R1, PC1, PC2, and PC3 without being prompted for a password. A login
attempt using a password fails with an explicit rejection.

---

### Lesson 3: Capture and analyze VLAN traffic

Most engineers understand VLANs conceptually but have never seen the 802.1Q tag in an
actual frame. This lesson makes VLAN segmentation visible. You will capture live traffic
on R1's trunk interface, read the VLAN tags in the output, and confirm that the switch is
tagging frames correctly before they reach the router.

#### What to accomplish

- Start a packet capture on R1's trunk-facing interface while hosts generate traffic
  across VLANs.
- Identify 802.1Q-tagged frames in the capture and read the VLAN ID from each.
- Confirm that traffic from VLAN 10 hosts and VLAN 20 hosts carries different tags.
- Observe where tags appear and where they disappear as traffic crosses the topology.

#### You're done when

You can point to a specific captured frame, identify its VLAN tag, and explain why that
tag is present at that location in the topology.

---

### Lesson 4: Bash — automated configuration backup

Network device configurations change, and without backups you cannot recover from a
misconfiguration or compare what changed between incidents. This lesson builds the backup
workflow that every production network should have.

#### What to accomplish

- Write a Bash script that SSHes into R1 and retrieves its running configuration.
- Save the output to a local file named with the current date and time so each backup is
  uniquely identified and sortable.
- Store backups in a dedicated directory that the script creates if it does not already
  exist.
- Schedule the script to run automatically every hour using cron.
- Verify the schedule works by confirming a new backup file appears at the next interval.

#### You're done when

A timestamped backup file for R1's configuration exists in your backup directory, and
`crontab -l` shows an entry that runs the script hourly.

---

### Lesson 5: Configure and test static routes

Your current topology uses VyOS VLAN subinterfaces, which automatically create connected
routes. This lesson removes that automatic behavior and forces you to build the routing
table by hand. Static routing is the foundation for understanding dynamic protocols: you
cannot understand what OSPF is doing for you until you have done it yourself.

#### What to accomplish

- Remove the VLAN subinterface configuration from R1.
- Add explicit static routes for each VLAN subnet, specifying the correct next hop for
  each destination.
- Verify the routing table on R1 shows the static routes and that inter-VLAN reachability
  still works.
- Deliberately remove one static route, observe exactly what breaks and why, then restore
  it.

#### You're done when

PC1 can reach PC3 using only static routes on R1 with no VLAN subinterfaces. You can
explain what happens to traffic when a static route is absent and how to diagnose it.

---

### Lesson 6: Configure a zone-based firewall on R1

Your lab currently applies no access control between VLANs — any host on VLAN 20 can
reach any service on VLAN 10. In a real network, the client segment should not have
unrestricted access to the server segment. This lesson introduces zone-based firewalling
to enforce a security boundary between segments.

#### What to accomplish

- Define two firewall zones on R1: one for VLAN 10 (servers) and one for VLAN 20
  (clients).
- Write a policy that blocks all traffic initiated from VLAN 20 to VLAN 10, except ICMP.
- Apply the policy between the zones.
- Verify that a host on VLAN 20 can ping a VLAN 10 host but cannot establish a TCP
  connection to any port on it.
- Verify that VLAN 10 hosts can still initiate connections to VLAN 20 without
  restriction.

#### You're done when

A ping from PC3 to PC1 succeeds. A TCP connection attempt from PC3 to PC1 fails. A TCP
connection from PC1 to PC3 succeeds.

---

### Lesson 7: Bash — interface monitoring

An interface going down silently is a common failure mode. Without monitoring, you learn
about it when users complain. This lesson builds a polling script that checks interface
state on a schedule and alerts when something is wrong, which is the pattern behind tools
like Nagios and simple health-check agents.

#### What to accomplish

- Write a Bash script that SSHes into R1 on a fixed interval and reads the current state
  of all interfaces.
- Parse the output to detect any interface reporting a down state.
- Print a timestamped alert when a down interface is detected.
- Print a timestamped status message when all interfaces are up.
- Run the script, then use the GNS3 interface menu to bring a link down on R1 and confirm
  the script detects it within one polling interval.

#### You're done when

The script prints a timestamped alert within one polling interval of a link going down,
and a recovery message when the link comes back up.

---

### Lesson 8: Configure OSPF between two routers

Static routes do not scale. When the topology changes, every router needs a manual update.
OSPF is a *link-state* routing protocol where routers discover neighbors, exchange
topology information, and compute routes automatically. This lesson introduces OSPF by
adding a second router and establishing a real adjacency.

#### What to accomplish

- Add a second VyOS router (R2) to your GNS3 topology and connect it to R1 with a
  point-to-point link.
- Assign addresses to the link between R1 and R2.
- Configure OSPF area 0 on both routers, advertising all subnets each router knows about.
- Verify that R1 and R2 reach OSPF neighbor adjacency in FULL state.
- Confirm that R2 learns the VLAN 10 and VLAN 20 subnets from R1 with no static routes
  configured on R2.

#### You're done when

R1 and R2 show each other as FULL OSPF neighbors. A host connected to R2 can reach PC1
and PC3 without any static routes on R2.

---

### Lesson 9: Centralize syslog from all devices

Log data scattered across individual devices is nearly useless for troubleshooting or
auditing. Centralized logging aggregates all events to one place so you can correlate
activity across devices and search efficiently. This is a prerequisite for any meaningful
monitoring setup.

#### What to accomplish

- Configure PC1 as a syslog server that listens for incoming messages on UDP port 514.
- Configure PC2 and PC3 to forward all local log messages to PC1.
- Configure R1 to forward log messages at info level and above to PC1.
- Generate a test log message on PC2 and verify it appears in the log store on PC1.
- Confirm that messages from each device are distinguishable by hostname in the collected
  logs.

#### You're done when

You can trigger a log event on any device and find that specific event in the log files on
PC1, attributed to the correct source hostname.

---

### Lesson 10: Go — concurrent multi-device automation

Your Bash scripts run sequentially — each SSH session must complete before the next
begins. In a large network, sequential polling is too slow. Go's concurrency model lets
you query all devices simultaneously and collect results as they arrive. This is how
real network automation tools work at scale.

#### What to accomplish

- Write a Go program that maintains a list of all four devices with their management IPs,
  SSH credentials, and the command to run on each.
- Use goroutines to SSH into all devices at the same time rather than one after another.
- Collect output from each device into a shared results structure without data races.
- Write a report file that shows each device's address and the command output.
- Handle errors per device without crashing the whole program — if one device is
  unreachable, the others must still complete.
- Measure and print the total collection time, then compare it to what a sequential
  version would take.

#### You're done when

The program queries all four devices simultaneously, writes a complete report, handles
individual device failures gracefully, and finishes faster than a sequential equivalent.
