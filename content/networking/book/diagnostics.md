+++
title = 'Diagnostics'
date = '2026-05-30T11:10:23-04:00'
weight = 30
draft = false
+++

## Commands overview

| Name              | Description                                                                                                     |
| ----------------- | --------------------------------------------------------------------------------------------------------------- |
| `arp`             | Displays and manages the ARP cache, which maps IP addresses to MAC addresses on the local network               |
| `netplan`         | Ubuntu's YAML-based network configuration tool; applies settings at boot or on demand                           |
| `ip` / `ifconfig` | Configure and inspect network interfaces, addresses, and routes; `ip` is the modern replacement for `ifconfig`  |
| `netstat` / `ss`  | Display active connections, routing tables, and socket statistics; `ss` is the modern replacement for `netstat` |
| `telnet`          | Connect to a remote host over TCP; commonly used to test whether a specific port is reachable                   |
| `nc`              | General-purpose tool for reading and writing data across TCP or UDP connections                                 |

## Tools overview

| Name    | Description                                                                                                 |
| ------- | ----------------------------------------------------------------------------------------------------------- |
| Nmap    | Network scanner for host discovery, open port enumeration, and service version detection                    |
| Kismet  | Passive wireless network detector, packet sniffer, and intrusion detection system                           |
| Wavemon | Terminal-based monitor that displays real-time signal strength and connection stats for wireless interfaces |
| Linssid | Graphical Wi-Fi scanner that lists nearby networks with channel, signal strength, and security details      |

## Layer 2: IP and MAC with ARP

### MAC addresses

A MAC (Media Access Control) address is a 48-bit identifier burned into every NIC at manufacture. It has 12 hexadecimal digits written as six colon-separated pairs: `52:54:00:2d:d3:ce`. The first three pairs form the *Organizationally Unique Identifier (OUI)*, assigned by the IEEE to the manufacturer. The last three pairs are the device-specific extension identifier.

MAC addresses operate at Layer 2 and identify devices within a broadcast domain. Every device with a network interface has one: hosts, routers (one per interface), switches (for management), wireless access points, and IoT devices.

#### OUI values

The first three bytes of a MAC address are the *OUI (Organizationally Unique Identifier)*. The IEEE assigns each OUI to a specific manufacturer. Look up any OUI in the Wireshark project's OUI database to identify the vendor behind an unknown MAC address.

Organizations can purchase longer OUI prefixes. A 28-bit or 36-bit prefix trades a larger identifier block for fewer available device addresses. A 36-bit OUI leaves only 12 bits for device IDs, enough for roughly 4,000 addresses. Smaller organizations or those with limited device counts buy these to avoid purchasing a full 24-bit block they would never fill.

OUIs are useful for network diagnostics. When an unknown device appears in an ARP table or a network scan, the OUI identifies the manufacturer, which narrows down what the device is before you investigate further.

#### Find MAC address

```bash
ip link show                      # List all interfaces
ip link show enp1s0 | grep link   # Show the MAC address for one interface
```

`ip link show` lists every network interface with its Layer 2 details. Each entry includes a `link/ether` line showing the MAC address and the broadcast address for that interface.

`ip link show enp1s0 | grep link` filters output to just the `link/ether` line for a named interface, returning a single line with the MAC.

#### Change MAC address temporarily

Most drivers require the interface to be down before you change its MAC. The driver writes the new address into its working state but never touches the hardware. The burned-in OUI is read-only on the NIC. It encodes the manufacturer's identifier in the first three octets and is assigned by the IEEE to each vendor.

```bash
sudo ip link set dev enp1s0 down
sudo ip link set dev enp1s0 address 00:11:22:33:44:55   # spoofed MAC
sudo ip link set dev enp1s0 up
```

The change reverts on reboot or when the interface reloads.

Use this when:

- Your ISP hardcoded your old firewall's MAC and you replaced the firewall. Spoofing the old MAC brings the new device online without waiting for ISP-side changes.
- You replaced a host or NIC and the upstream router has the old MAC in its ARP cache. If you can't access the router to flush it and can't wait for the entry to expire, spoofing the old MAC makes the router forward traffic to the new hardware immediately.
- A DHCP server has a reservation tied to the old MAC and you can't update the entry. Spoofing the old MAC lets the replacement host claim the same reserved address.
- Avoiding MAC-based DHCP lease tracking when moving a host between environments. Some DHCP servers assign leases by MAC, so a new MAC gets a new lease and a new IP.
- Testing network policies or firewall rules that filter by MAC address without needing a second physical device.
- Apple devices randomize their wireless MAC per network as a privacy measure, but the protection is limited. Passive observers can still correlate sessions through traffic patterns, timing, and probe behavior regardless of MAC changes.

#### Change MAC address permanently

To set a MAC address that persists across reboots, add the `macaddress` key to the interface's netplan YAML in `/etc/netplan/`.

```yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp1s0:
      dhcp4: false
      match:
          macaddress: 52:54:00:2d:d3:ce
      macaddress: 00:11:22:33:44:55
      ...
```

The `match:` key selects the physical interface by its burned-in MAC rather than its kernel-assigned name. Interface names like `enp1s0` can change after a hardware swap. `match.macaddress` uses the OUI — the hardware address permanently encoded in the NIC — as a stable identifier that survives renames.

The `macaddress` key at the same level as `match:` sets the address the driver presents to the network. Netplan applies it each time the interface comes up.

Apply the change:

```bash
sudo netplan apply
```

#### Add a static entry with ip neigh

The `iproute2` equivalent of `arp -s`. Prefer this over `arp -s` on modern Linux. The `dev` argument is required, which eliminates the interface ambiguity that causes `arp -s` to fail.

```bash
sudo ip neigh add 192.168.122.200 lladdr 00:11:22:33:44:55 dev enp1s0
```

Adds a permanent neighbor entry on `enp1s0`.

#### Add a static ARP entry

Use this when a critical device must always resolve to a known MAC, or when you want to prevent ARP spoofing from overwriting a gateway entry.

```bash
sudo arp -s 192.168.122.200 00:11:22:22:33:33 -i enp1s0
```

Maps `192.168.122.200` to `00:11:22:22:33:33` on `enp1s0`.

- `-s`: creates a static entry that never ages out
- `-i`: binds the entry to a specific interface, required when multiple interfaces can reach the same subnet. Without it, the kernel cannot determine which interface to use and returns `SIOCSARP: Invalid argument`.

Verify the entry:

```bash
arp -a | grep 192.168.122.200
networking (192.168.122.200) at 00:11:22:22:33:33 [ether] PERM on enp1s0
```

`PERM` confirms the entry is permanent. The hostname `networking` resolved from reverse DNS for that IP.

#### Configure proxy ARP

*Proxy ARP* lets a host answer ARP requests on behalf of a device it can reach but the requesting host cannot. Instead of the real device replying, the proxy replies with its own MAC. The sender delivers frames to the proxy, which forwards them to the actual destination. The sender never knows a proxy is involved.

Use this when:

- Bridging two segments where a device on one side must be reachable from the other without a routing change
- Inserting a transparent firewall into a path. Traffic destined for the real host gets redirected through this system first.
- A host is moving subnets but you need to preserve reachability from the original segment temporarily

```bash
sudo arp -i enp1s0 -Ds 10.0.0.2 eth1 pub
```

Publishes a proxy ARP entry on `enp1s0`. When any host sends an ARP request for `10.0.0.2` on `enp1s0`, this system replies with the MAC address of `eth1`.

- `-i enp1s0`: the interface on which to listen for and answer ARP requests
- `-D`: uses the MAC address of the named interface (`eth1`) rather than a manually specified address
- `-s 10.0.0.2`: the IP address to proxy for
- `eth1`: the interface whose MAC to use in the reply
- `pub`: marks the entry as published. This system answers ARP requests on behalf of `10.0.0.2` rather than caching the mapping for local use only.

### MAC address table

Switches add entries to the CAM table dynamically as frames arrive, recording the source MAC against the ingress port. If a MAC appears on a different port later, the switch updates the entry. This handles devices that move or topology changes from STP reconvergence.

Entries age out after no frames arrive from that MAC within the aging timer. This prevents the table from filling with stale entries for disconnected devices. The default aging time on most switches is 300 seconds (5 minutes).

On Cisco IOS:

```bash
! View the table
show mac address-table

! Change the aging timer (range: 10–1000000 seconds)
mac address-table aging-time 300
```

On a Linux bridge:

```bash
# View the forwarding database
bridge fdb show

# Set aging time (value in centiseconds; 30000 = 300 seconds)
ip link set dev br0 type bridge ageing_time 30000
```

### ARP requests and replies

*ARP (Address Resolution Protocol)* resolves a known IP address to its MAC address so a host can build a Layer 2 frame. When a host needs a MAC it doesn't have cached, it sends an ARP request as a Layer 2 broadcast (`ff:ff:ff:ff:ff:ff`) containing its own IP and MAC and the target IP. Every host on the segment receives it. The host with the matching IP sends a unicast ARP reply containing its MAC address. The requesting host caches the mapping and builds the frame.

When the destination IP is outside the local subnet, the host sends an ARP request for the default gateway's IP, not the final destination. Routing happens at Layer 3. The Layer 2 frame only travels one hop at a time, so the destination MAC is always the next-hop device, not the ultimate target.

### ARP and switch infrastructure

Switches build their MAC address table (*CAM table*) by inspecting the source MAC of every incoming frame and associating it with the ingress port. ARP traffic accelerates this learning: the request is a broadcast received on all ports, and the reply is a unicast. Both carry source MACs that the switch records. Once the destination MAC appears in the CAM table, the switch forwards frames only to the correct port instead of flooding all ports.

### ARP cache

The ARP cache is a kernel-maintained table mapping IP addresses to MAC addresses on a host. It eliminates repeated broadcasts for recently contacted devices. On Linux, entries use two timers: a *reachable* timer (default 30 seconds) and a *stale* timer. After an entry goes stale, the kernel sends a unicast probe to confirm reachability before expiring it. Full expiry under default settings is typically 60–90 seconds.

To inspect the stale timer per interface:

```bash
# List available neighbor parameter directories
ls /proc/sys/net/ipv4/neigh/
default  enp1s0  lo

# Check the stale timer for the default profile (seconds)
cat /proc/sys/net/ipv4/neigh/default/gc_stale_time
60

# Check the stale timer for a specific interface
cat /proc/sys/net/ipv4/neigh/enp1s0/gc_stale_time
60
```

The following `arp -a` output shows a mixed environment with a wireless uplink, KVM virtual machines, and Kubernetes Calico interfaces:

```bash
? (192.168.122.201) at 52:54:00:2d:d3:ce [ether] on virbr0
? (10.1.224.15) at c6:68:38:46:6e:19 [ether] on calibc998608fcd
? (192.168.122.200) at 52:54:00:2d:d3:ce [ether] on virbr0
CR1000A.mynetworksettings.com (192.168.1.1) at 58:96:71:28:e6:26 [ether] on wlp59s0
? (10.1.224.21) at 8a:7a:d0:44:6a:a4 [ether] on cali181697274dd
? (192.168.1.171) at 42:ae:30:0b:ba:fa [ether] on wlp59s0
? (192.168.122.209) at 52:54:00:f4:e3:a5 [ether] on virbr0
? (169.254.169.254) at <incomplete> on virbr0
RE315 (192.168.1.154) at 42:ae:30:0b:bb:e0 [ether] on wlp59s0
RE550 (192.168.1.188) at 9e:03:8e:bb:ab:44 [ether] on wlp59s0
```

Each line follows the format `hostname (IP) at MAC [type] on interface`. A `?` in the hostname field means no reverse DNS record exists for that IP. The fields break down as follows:

| Field     | Example             | Meaning                                 |
| --------- | ------------------- | --------------------------------------- |
| Hostname  | `CR1000A...` / `?`  | Resolved hostname, or `?` if none       |
| IP        | `192.168.1.1`       | Layer 3 address                         |
| MAC       | `58:96:71:28:e6:26` | Layer 2 address                         |
| Type      | `[ether]`           | Interface type (Ethernet)               |
| Interface | `wlp59s0`           | Local interface that holds this mapping |

Notable entries:

- `52:54:00:*` MACs belong to QEMU/KVM VMs. QEMU's OUI is `52:54:00`, so all three `virbr0` entries are virtual machines on the local KVM bridge.
- `cali*` interfaces are Calico CNI veth pairs used by Kubernetes pods on the `10.1.224.0/24` subnet.
- `CR1000A.mynetworksettings.com` at `192.168.1.1` is the default gateway and the only entry with a resolved hostname.
- `<incomplete>` on `169.254.169.254` means the kernel sent an ARP request but received no reply. That link-local address is the cloud instance metadata endpoint, which is unreachable in this environment.
- `RE315` and `RE550` are Wi-Fi extenders with resolved hostnames, reachable over `wlp59s0`.

### Timeout comparison

Switch CAM tables and router ARP tables use very different aging timers because they serve different purposes.

A switch's CAM table tracks which port a MAC address lives on. Layer 2 topology changes frequently: devices move ports, STP reconverges, and VLANs shift. A stale CAM entry sends frames to the wrong port, so switches use a short default aging time of 300 seconds (5 minutes).

A router's ARP table tracks which MAC address owns a given IP. IP-to-MAC mappings are stable by comparison: a host rarely changes its NIC. Cisco routers default to 14400 seconds (4 hours) per interface. If a mapping changes before expiry, a gratuitous ARP from the device updates the cache immediately.

Linux uses a neighbor state machine rather than a hard timer. An entry moves through REACHABLE → STALE → DELAY → PROBE → FAILED before the kernel removes it. The `gc_stale_time` (default 60 seconds) controls how long a STALE entry persists before the garbage collector removes it, not the total entry lifetime.

| Table          | Device       | Default timeout  | Reason for value                    |
| -------------- | ------------ | ---------------- | ----------------------------------- |
| CAM table      | Switch       | 300s (5 min)     | Layer 2 topology changes frequently |
| ARP table      | Cisco router | 14400s (4 hours) | IP-to-MAC mappings are stable       |
| Neighbor cache | Linux host   | 60s stale timer  | State machine probes before expiry  |

The long ARP timeout on routers directly conserves CPU. Each ARP miss is expensive: the router must queue or drop the packet, broadcast an ARP request out the egress interface, wait for a reply, process the reply in software, write the entry, and resume forwarding. That entire sequence is process-switched and hits the CPU rather than going through CEF or the forwarding ASIC. A four-hour timeout means the router resolves each IP-to-MAC mapping once and reuses it thousands of times before expiry, keeping the CPU free for actual routing work.

The contrast with switches is instructive. A switch's CAM table sits in hardware TCAM, which has a hard entry limit (often 8K–32K entries depending on the platform). Clearing stale entries quickly keeps the table small and prevents overflow. A full CAM table causes the switch to flood unknown unicast frames to all ports, which is both a performance problem and a security exposure. The short 300-second timer protects a scarce hardware resource. The long ARP timer on routers protects CPU.

### The /proc filesystem

`/proc` is a *virtual filesystem* (`procfs`) the kernel mounts at boot. Its files have no on-disk representation. When you read one, the kernel generates the contents on the fly from its current internal state. When you write to certain files (particularly under `/proc/sys/`), the kernel applies the change immediately.

This design lets you interact with live kernel state using standard file tools: `cat`, `grep`, and redirection all work. The `sysctl` command is a higher-level interface to `/proc/sys/` that validates inputs and supports persistent configuration through `/etc/sysctl.conf`.

The directory organizes into two main areas: numbered subdirectories (one per running process, named by PID) and named subsystem directories like `net/` for network state and `sys/` for tunable kernel parameters.

#### /proc/net/dev

`/proc/net/dev` shows per-interface byte and packet counters since the interface was last brought up.

```bash
cat /proc/net/dev
Inter-|   Receive                                                |  Transmit
 face |bytes    packets errs drop fifo frame compressed multicast|bytes    packets errs drop fifo colls carrier compressed
    lo:   37561     405    0    0    0     0          0         0    37561     405    0    0    0     0       0          0
enp1s0: 25052219   23363    0 11298    0     0          0         0  1109215    7750    0    0    0     0       0          0
```

| Field     | Side  | Meaning                                         |
| --------- | ----- | ----------------------------------------------- |
| `bytes`   | RX/TX | Total bytes transferred                         |
| `packets` | RX/TX | Total packets transferred                       |
| `errs`    | RX/TX | Hardware or driver errors                       |
| `drop`    | RX/TX | Packets discarded before delivery               |
| `fifo`    | RX/TX | Ring buffer overflows                           |
| `frame`   | RX    | Framing errors (misaligned or malformed frames) |
| `colls`   | TX    | Collisions (expected only on half-duplex links) |
| `carrier` | TX    | Carrier sense failures                          |

The `lo` loopback interface shows identical RX and TX counters because every packet sent to loopback is also received by loopback.

The `enp1s0` entry shows 11,298 drops on receive against 23,363 total packets — nearly half the inbound traffic was dropped. Drops at this layer indicate the kernel discarded packets before they reached any application. Common causes are a full ring buffer (the NIC is delivering frames faster than the kernel can drain them), firewall rules, or `RP_FILTER` rejecting packets with unexpected source addresses. The asymmetry between received bytes (25 MB) and transmitted bytes (1.1 MB) is consistent with a host that downloads more than it uploads.

#### /proc/meminfo

`/proc/meminfo` exposes the kernel's memory accounting. The three `Mem` fields give a quick picture of physical RAM state.

```bash
cat /proc/meminfo | grep Mem
MemTotal:        4009868 kB
MemFree:         3026660 kB
MemAvailable:    3575748 kB
```

| Field          | Value                | Meaning                                         |
| -------------- | -------------------- | ----------------------------------------------- |
| `MemTotal`     | 4009868 kB (~3.8 GB) | Total physical RAM                              |
| `MemFree`      | 3026660 kB (~2.9 GB) | RAM with no current use                         |
| `MemAvailable` | 3575748 kB (~3.4 GB) | RAM available to new processes without swapping |

`MemFree` and `MemAvailable` differ because the kernel uses otherwise-idle RAM for page cache and buffers. That memory is reclaimable: if a process needs it, the kernel evicts the cache and hands it over. The ~537 MB gap between `MemFree` and `MemAvailable` in this output is reclaimable cache. `MemAvailable` is the more useful figure when assessing whether the host has headroom — `MemFree` alone understates what's usable.

### ip neigh

`ip neigh` is the `iproute2` command for managing the kernel neighbor table — the same table `arp -a` displays, but with full state visibility.

Read operations:

```bash
# Show all entries
ip neigh show

# Show entries for one interface
ip neigh show dev virbr0

# Show only reachable entries
ip neigh show nud reachable
```

Write operations:

```bash
# Add a static entry
sudo ip neigh add 192.168.122.200 lladdr 00:11:22:33:44:55 dev virbr0

# Replace an existing entry (add or update)
sudo ip neigh replace 192.168.122.200 lladdr 00:11:22:33:44:55 dev virbr0

# Delete one entry
sudo ip neigh del 192.168.122.200 dev virbr0

# Flush all dynamic entries on an interface
sudo ip neigh flush dev virbr0

# Flush everything (all interfaces, all states)
sudo ip neigh flush all
```

The `State` column in `ip neigh show` output maps to the kernel neighbor state machine:

| State       | Meaning                                                       |
| ----------- | ------------------------------------------------------------- |
| `REACHABLE` | Recently confirmed reachable; used directly                   |
| `STALE`     | Timer expired; still usable but the kernel probes on next use |
| `DELAY`     | Waiting before sending a unicast probe                        |
| `PROBE`     | Actively sending unicast probes to confirm reachability       |
| `FAILED`    | Probes got no reply; entry is dead                            |
| `PERMANENT` | Statically configured; never ages out                         |
| `NOARP`     | No ARP on this interface type (for example, point-to-point)   |

The key difference from `arp`: `arp` does not display the neighbor state label. `ip neigh show` exposes the current state for every entry, which is useful when debugging unreachable hosts or incorrect MAC mappings.
