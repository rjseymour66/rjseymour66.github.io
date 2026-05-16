+++
title = 'GNS3 network lab'
date = '2026-05-16T09:00:00-04:00'
weight = 90
draft = false
+++

*GNS3* is a network emulator that runs real network OS images inside virtual machines. This lab
gives you a working environment with a router, a switch, and Linux hosts to practice routing,
VLAN segmentation, and SSH-based automation with Go and Bash.

The router runs *VyOS*, a free, open-source network OS with SSH and a REST API that matches what
you encounter in production environments.

## Hardware requirements

Your host machine needs the following to run this lab:

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| RAM | 8 GB | 16 GB |
| CPU | 4 cores (VT-x/AMD-V enabled) | 8 cores |
| Disk | 20 GB free | 50 GB free |

Enable hardware virtualization (VT-x on Intel, AMD-V on AMD) in your BIOS before installing GNS3.

The lab allocates these resources across its virtual devices:

| Device | RAM | vCPUs |
|--------|-----|-------|
| GNS3 VM | 2 GB | 2 |
| R1 (VyOS) | 512 MB | 1 |
| PC1 (Alpine Linux) | 128 MB | 1 |
| PC2 (Alpine Linux) | 128 MB | 1 |
| PC3 (Alpine Linux) | 128 MB | 1 |
| **Total** | **~3 GB** | **6** |

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

| Device | OS | Role |
|--------|----|------|
| R1 | VyOS | Router, inter-VLAN gateway |
| SW1 | GNS3 Ethernet Switch | Layer 2 switching, VLAN segmentation |
| PC1 | Alpine Linux | End host, VLAN 10 |
| PC2 | Alpine Linux | End host, VLAN 10 |
| PC3 | Alpine Linux | End host, VLAN 20 |

### VLAN assignments

| VLAN | Name | Subnet | Gateway |
|------|------|--------|---------|
| 10 | servers | 10.0.10.0/24 | 10.0.10.1 |
| 20 | clients | 10.0.20.0/24 | 10.0.20.1 |

### Data-plane IP addresses

| Device | Interface | IP address |
|--------|-----------|------------|
| R1 | eth1.10 | 10.0.10.1/24 |
| R1 | eth1.20 | 10.0.20.1/24 |
| PC1 | eth0 | 10.0.10.10/24 |
| PC2 | eth0 | 10.0.10.20/24 |
| PC3 | eth0 | 10.0.20.10/24 |

## Management network options

Your Go and Bash scripts need a reliable path to SSH into devices. Two approaches exist.

### Option 1: Out-of-band management (recommended)

A dedicated management network connects your workstation to every device's management interface
through a GNS3 NAT cloud node. Your scripts always reach devices at stable `192.168.100.x`
addresses, even when the data-plane topology is misconfigured during practice.

This approach matches the standard used in enterprise network automation environments. If you
break routing on the data plane, you can still SSH in and fix it.

**Management IP addresses:**

| Device | Interface | Management IP |
|--------|-----------|---------------|
| R1 | eth0 | 192.168.100.1/24 |
| PC1 | eth1 | 192.168.100.11/24 |
| PC2 | eth1 | 192.168.100.12/24 |
| PC3 | eth1 | 192.168.100.13/24 |

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

| Port | Mode | VLAN |
|------|------|------|
| Port1 | Trunk | 10, 20 |
| Port2 | Access | 10 |
| Port3 | Access | 10 |
| Port4 | Access | 20 |

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
