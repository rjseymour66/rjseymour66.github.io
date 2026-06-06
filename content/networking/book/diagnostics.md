+++
title = 'Diagnostics'
date = '2026-05-30T11:10:23-04:00'
weight = 30
draft = false
+++

## Commands overview

| Name              | Description                                                                                                     |
| :---------------- | :-------------------------------------------------------------------------------------------------------------- |
| `arp`             | Displays and manages the ARP cache, which maps IP addresses to MAC addresses on the local network               |
| `netplan`         | Ubuntu's YAML-based network configuration tool; applies settings at boot or on demand                           |
| `ip` / `ifconfig` | Configure and inspect network interfaces, addresses, and routes; `ip` is the modern replacement for `ifconfig`  |
| `netstat` / `ss`  | Display active connections, routing tables, and socket statistics; `ss` is the modern replacement for `netstat` |
| `telnet`          | Connect to a remote host over TCP; commonly used to test whether a specific port is reachable                   |
| `nc`              | General-purpose tool for reading and writing data across TCP or UDP connections                                 |

## Tools overview

| Name    | Description                                                                                                 |
| :------ | :---------------------------------------------------------------------------------------------------------- |
| Nmap    | Network scanner for host discovery, open port enumeration, and service version detection                    |
| Kismet  | Passive wireless network detector, packet sniffer, and intrusion detection system                           |
| Wavemon | Terminal-based monitor that displays real-time signal strength and connection stats for wireless interfaces |
| Linssid | Graphical Wi-Fi scanner that lists nearby networks with channel, signal strength, and security details      |

## IP and MAC with ARP (L2)

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
| :-------- | :------------------ | :-------------------------------------- |
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
| :------------- | :----------- | :--------------- | :---------------------------------- |
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
| :-------- | :---- | :---------------------------------------------- |
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
| :------------- | :------------------- | :---------------------------------------------- |
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
| :---------- | :------------------------------------------------------------ |
| `REACHABLE` | Recently confirmed reachable; used directly                   |
| `STALE`     | Timer expired; still usable but the kernel probes on next use |
| `DELAY`     | Waiting before sending a unicast probe                        |
| `PROBE`     | Actively sending unicast probes to confirm reachability       |
| `FAILED`    | Probes got no reply; entry is dead                            |
| `PERMANENT` | Statically configured; never ages out                         |
| `NOARP`     | No ARP on this interface type (for example, point-to-point)   |

The key difference from `arp`: `arp` does not display the neighbor state label. `ip neigh show` exposes the current state for every entry, which is useful when debugging unreachable hosts or incorrect MAC mappings.


## ARP cache

The *ARP cache* is the kernel's in-memory table of resolved IP-to-MAC mappings. When a host sends a packet to a destination on the same subnet, it checks the ARP cache first. A cache hit skips the ARP broadcast and sends the frame directly. A cache miss triggers an ARP request.

On Linux, the kernel exposes the IPv4 ARP cache at `/proc/net/arp`:

```bash
cat /proc/net/arp
```

That file shows the IP address, hardware type, flags, MAC address, mask, and interface for each cached entry. It covers IPv4 only and omits neighbor state labels.

`ip neigh show` reads the same kernel neighbor table but includes both IPv4 and IPv6 entries and displays the neighbor state (`REACHABLE`, `STALE`, `PERMANENT`, and so on).

Entries age out automatically. Linux marks a reachable entry `STALE` after `gc_stale_time` seconds (default 60 seconds) and removes it if unused.

## Ports (L4)

TCP and UDP identify services by port number. A port is a 16-bit integer (0–65535) that the OS uses to direct incoming traffic to the correct process. TCP uses ports to establish reliable, connection-oriented sessions. UDP uses ports for connectionless delivery. In both cases, traffic flows from a source port on the client to a destination port on the server.

### Port assignments

IANA divides the port space into three ranges:

| Range       | Name             | Description                                                                |
| :---------- | :--------------- | :------------------------------------------------------------------------- |
| 0–1023      | Well-known ports | Reserved for standard services: HTTP (80), HTTPS (443), SSH (22), DNS (53) |
| 1024–49151  | Registered ports | Assigned to applications by IANA; not reserved by the OS                   |
| 49152–65535 | Dynamic ports    | Ephemeral ports assigned by the OS for outgoing connections                |

The full IANA registry is at [service-names-port-numbers](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml).

### Common ports

| Port | Transport | Service           | Notes                                                       |
| :--- | :-------- | :---------------- | :---------------------------------------------------------- |
| 20   | TCP       | FTP data          | Active mode data transfer                                   |
| 21   | TCP       | FTP control       | Command channel                                             |
| 22   | TCP       | SSH               | Secure shell; also used for SCP and SFTP                    |
| 23   | TCP       | Telnet            | Unencrypted; avoid in production                            |
| 25   | TCP       | SMTP              | Mail transfer between servers                               |
| 49   | TCP       | TACACS+           | Cisco AAA; encrypts the entire payload                      |
| 53   | TCP/UDP   | DNS               | UDP for queries; TCP for zone transfers and large responses |
| 67   | UDP       | DHCP server       | Listens for client broadcasts                               |
| 68   | UDP       | DHCP client       | Receives server offers                                      |
| 69   | UDP       | TFTP              | Used for device config and image transfers                  |
| 80   | TCP       | HTTP              | Unencrypted web traffic                                     |
| 88   | TCP/UDP   | Kerberos          | Authentication in Active Directory environments             |
| 123  | UDP       | NTP               | Time synchronization                                        |
| 161  | UDP       | SNMP              | Manager polls agent                                         |
| 162  | UDP       | SNMP trap         | Agent sends unsolicited notifications to manager            |
| 179  | TCP       | BGP               | eBGP and iBGP session establishment                         |
| 389  | TCP       | LDAP              | Directory queries                                           |
| 443  | TCP       | HTTPS             | TLS-encrypted web traffic                                   |
| 445  | TCP       | SMB               | Windows file sharing and Active Directory                   |
| 514  | UDP       | Syslog            | Log forwarding to a syslog server                           |
| 636  | TCP       | LDAPS             | TLS-encrypted LDAP                                          |
| 830  | TCP       | NETCONF           | XML-based network configuration (RFC 6241)                  |
| 1812 | UDP       | RADIUS auth       | Authentication and authorization                            |
| 1813 | UDP       | RADIUS accounting | Session accounting                                          |
| 3389 | TCP       | RDP               | Windows Remote Desktop                                      |
| 5060 | UDP/TCP   | SIP               | VoIP signaling                                              |
| 5061 | TCP       | SIP (TLS)         | Encrypted VoIP signaling                                    |

### Ephemeral ports

When a client opens a connection, the OS picks a temporary source port from the ephemeral range — any available port above 1024 and below 65535. The client holds that port for the life of the connection, then releases it. On Linux, the OS selects from a configurable range stored in `/proc/sys/net/ipv4/ip_local_port_range` (default: 32768–60999).

### Tuples

A *tuple* is an ordered set of values that together identify a network flow. A *5-tuple* uniquely identifies a connection:

| Field            | Example       |
| :--------------- | :------------ |
| Source IP        | 192.168.1.10  |
| Source port      | 54321         |
| Destination IP   | 93.184.216.34 |
| Destination port | 443           |
| Protocol         | TCP           |

No two active connections share the same 5-tuple. The OS uses it to demultiplex incoming packets to the correct socket. Each IP address in the 5-tuple belongs to an autonomous system. BGP uses the destination IP's ASN to route the packet across AS boundaries to reach the far endpoint.

To the network stack, the 5-tuple is a hash key. When a packet arrives, the kernel extracts the five fields from the IP and TCP/UDP headers and hashes them to an integer index. That index locates a bucket in the connection table. Each bucket holds a linked list of `struct nf_conntrack_tuple` entries to handle collisions. The hash function uses a per-boot random seed (SipHash) to prevent hash-flooding attacks.

The tuple serves three roles in the stack:

- *Demultiplex*: delivers the incoming packet to the correct socket and process
- *State tracking*: conntrack stores the tuple so stateful firewalls and NAT know which connection a packet belongs to
- *NAT key*: when a packet is masqueraded, conntrack rewrites the source fields and stores the original tuple to reverse the translation on the reply

`ss` shows active tuples. The local address:port is the source IP and port. The peer address:port is the destination IP and port. The `Netid` column is the protocol:

```bash
ss -tunapo
Netid  State   Recv-Q  Send-Q  Local Address:Port   Peer Address:Port    Process
tcp    ESTAB   0       0       192.168.1.10:54321   93.184.216.34:443    users:(("curl",pid=1234,fd=3))
```

`conntrack -L` shows the same connection from the netfilter tracking table:

```bash
tcp   6  299  ESTABLISHED src=192.168.1.10 dst=93.184.216.34 sport=54321 dport=443 ...
```

Check the connection table limits:

```bash
cat /proc/sys/net/netfilter/nf_conntrack_max       # max conntrack entries
cat /proc/sys/net/netfilter/nf_conntrack_buckets    # hash table bucket count
```

### QoS, TOS, and DSCP

*Quality of Service (QoS)* prioritizes traffic so latency-sensitive flows (voice, video) are forwarded ahead of bulk transfers (backups, file copies).

QoS markings travel in the IP header alongside the 5-tuple fields:

- *TOS (Type of Service)*: the original 8-bit IPv4 header field for traffic priority
- *DSCP (Differentiated Services Code Point)*: the modern replacement defined in RFC 2474; reuses the top 6 bits of the TOS field to encode 64 possible forwarding behaviors called *per-hop behaviors (PHBs)*

Routers and switches read the DSCP value and apply a queuing policy. The marking travels with the packet end to end. It is set by the sender or remarked at a network trust boundary.

### Autonomous System Numbers

An *Autonomous System (AS)* is a collection of IP prefixes under common administrative control — an ISP, a cloud provider, or a large enterprise. The *Autonomous System Number (ASN)* identifies each AS. IANA assigns ASNs through five Regional Internet Registries (RIRs): ARIN (North America), RIPE NCC (Europe and Middle East), APNIC (Asia-Pacific), LACNIC (Latin America), and AFRINIC (Africa).

ASNs are 16-bit (1–65535) or 32-bit (1–4294967295, introduced in RFC 4893). IANA reserves private ranges for internal use: 64512–65535 for 16-bit and 4200000000–4294967294 for 32-bit.

BGP uses ASNs to exchange routing information between autonomous systems. Each BGP router announces the IP prefixes it can reach, tagged with its ASN and the AS path to the destination.

### ss

`ss` is the modern replacement for `netstat`. It reads directly from kernel socket structures rather than `/proc`, making it faster and more detailed. Use it to inspect active connections, listening ports, socket queues, and TCP timer state.

| Option | Description                                                 |
| :----- | :---------------------------------------------------------- |
| `-t`   | TCP sockets                                                 |
| `-u`   | UDP sockets                                                 |
| `-a`   | All sockets, including listening                            |
| `-n`   | Numeric addresses and ports; suppresses name resolution     |
| `-p`   | Show process name, PID, and file descriptor for each socket |
| `-o`   | Show TCP timer information                                  |

Common usages:

```bash
sudo ss -tuap                                                                                               # all TCP/UDP sockets with process info
sudo ss -tuapn                                                                                              # numeric output, no name resolution
sudo ss -to                                                                                                 # TCP sockets with timer information
sudo ss -tuap | tr -s ' ' | cut -d ' ' -f 1,2,4,5,6 --output-delimiter=$'\t'                              # extract key columns
sudo ss -tuap | tr -s ' ' | cut -d ' ' -f 1,2,4,5,6 --output-delimiter=$'\t' > ports.tsv                  # save to file
sudo ss -tuap | tr -s ' ' | cut -d ' ' -f 1,2,4,5,6 --output-delimiter=$'\t' | grep "EST" | tee ports.out # filter established, save and print
```

Each row represents one socket:

| Column               | Description                                                                    |
| :------------------- | :----------------------------------------------------------------------------- |
| `Netid`              | Socket type: `tcp`, `udp`, `tcp6`, `udp6`                                      |
| `State`              | Connection state; `UNCONN` means UDP with no connected peer                    |
| `Recv-Q`             | Bytes received but not yet read by the process                                 |
| `Send-Q`             | Bytes sent but not yet acknowledged                                            |
| `Local Address:Port` | Local endpoint; service names appear unless `-n` is added                      |
| `Peer Address:Port`  | Remote endpoint; `0.0.0.0:*` on a listener or unconnected UDP socket           |
| `Process`            | Program name, PID, and file descriptor in `users:(("name",pid=N,fd=N))` format |

```bash
Netid    State     Send-Q    Local Address:Port          Peer Address:Port
udp      UNCONN    0         127.0.0.54:domain           0.0.0.0:*
udp      UNCONN    0         127.0.0.53%lo:domain        0.0.0.0:*
tcp      LISTEN    4096      0.0.0.0:ssh                 0.0.0.0:*
tcp      LISTEN    4096      127.0.0.54:domain           0.0.0.0:*
tcp      LISTEN    4096      127.0.0.53%lo:domain        0.0.0.0:*
tcp      ESTAB     0         192.168.122.200:ssh         192.168.122.1:59516
tcp      LISTEN    4096      [::]:ssh                    [::]:*
```

- UDP sockets show `UNCONN` instead of `LISTEN` because UDP is connectionless. A UDP socket binds to a port and receives datagrams without establishing a session.
- `Send-Q` on a `LISTEN` socket shows the accept backlog size (4096), not bytes. It is the maximum number of pending connections the kernel will queue before refusing new ones.
- `127.0.0.53%lo:domain` — the `%lo` suffix is a zone ID scoping the address to the `lo` interface. It distinguishes sockets bound to the same address on different interfaces.
- `[::]:ssh` — IPv6 wildcard listener; `[::]` is the IPv6 equivalent of `0.0.0.0`.
- Service names (`ssh`, `domain`) appear because `-n` is not set. Add `-n` to see numeric ports.

#### TCP timers

`ss -to` adds a `timer:` field to each TCP socket showing the active kernel timer:

```bash
State  Recv-Q  Send-Q  Local Address:Port       Peer Address:Port    Timer
ESTAB  0       0       192.168.122.200:ssh       192.168.122.1:59516  timer:(keepalive,40min,0)
```

The three values in `timer:(type,remaining,retransmits)`:

- `keepalive`: the OS sends a small probe on idle connections to detect whether the remote end is still alive
- `40min`: time remaining before the next probe fires
- `0`: retransmission count; a rising value means probes are going unanswered

The keepalive timer is a critical diagnostic for long-lived TCP sessions routed through a stateful firewall. Stateful firewalls track every TCP session in a connection table held in memory. To reclaim memory, they apply an idle timeout: if no traffic passes on a session for a set period (typically 30–60 minutes), the firewall silently removes the entry. It sends no FIN and no RST. Both endpoints still believe the connection is `ESTAB`, but the next packet arrives at the firewall with no matching state entry and is dropped. The session hangs without either side knowing why.

TCP keepalive prevents this by generating traffic on idle connections to reset the firewall's idle timer. Linux defaults to sending the first probe after two hours (`tcp_keepalive_time = 7200`). Most firewalls time out sessions well before that. If `ss -to` shows a keepalive timer of 40 minutes but the firewall's idle timeout is 30 minutes, the session will be killed before the probe fires.

This is where large file backups fail. A backup job (`rsync`, `scp`, tar over SSH) holds a session open for hours. While data flows the connection is never idle. But if the transfer stalls — a disk I/O bottleneck, a slow tape device, or a network hiccup — traffic stops. The firewall times out the entry, and when the backup resumes the next packet finds no state. The session is killed mid-transfer with a broken pipe or connection reset.

To reduce `tcp_keepalive_time` below the firewall's idle timeout:

```bash
sudo sysctl -w net.ipv4.tcp_keepalive_time=1800    # first probe after 30 minutes
sudo sysctl -w net.ipv4.tcp_keepalive_intvl=30     # 30 seconds between subsequent probes
sudo sysctl -w net.ipv4.tcp_keepalive_probes=5     # drop after 5 unanswered probes
```

To persist across reboots, add to `/etc/sysctl.conf`:

```bash
net.ipv4.tcp_keepalive_time = 1800
net.ipv4.tcp_keepalive_intvl = 30
net.ipv4.tcp_keepalive_probes = 5
```

Then apply:

```bash
sudo sysctl -p
```

### netstat

`netstat` displays active connections, listening ports, and socket statistics. It is deprecated on modern Linux in favor of `ss`, but remains widely available in scripts and documentation. Use it when `ss` is unavailable or when working with older systems.

| Option | Description |
| :--- | :--- |
| `-t` | TCP sockets |
| `-u` | UDP sockets |
| `-a` | All sockets, including listening ports |
| `-n` | Numeric addresses and ports; suppresses name resolution |
| `-l` | Listening sockets only |
| `-p` | Show PID and program name; run with `sudo` to see processes owned by other users |

Common usages:

```bash
netstat -tuan           # all TCP/UDP sockets with state, numeric output
sudo netstat -tulpn     # listening sockets only with process name, numeric output
```

Each row represents one socket:

| Column | Description |
| :--- | :--- |
| `Proto` | Protocol: `tcp`, `tcp6`, `udp`, or `udp6` |
| `Recv-Q` | Bytes received but not yet read by the application |
| `Send-Q` | Bytes sent but not yet acknowledged by the remote end |
| `Local Address` | Local IP and port |
| `Foreign Address` | Remote IP and port; `0.0.0.0:*` on a listener means no peer yet |
| `State` | TCP connection state; blank for UDP |

```bash
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:1313          0.0.0.0:*               LISTEN
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN
tcp        0      0 192.168.1.162:33370     199.38.167.130:443      ESTABLISHED
tcp        0      0 127.0.0.1:59310         127.0.0.1:1313          ESTABLISHED
tcp        0      0 192.168.122.1:59516     192.168.122.200:22      ESTABLISHED
tcp        0      0 127.0.0.1:48656         127.0.0.1:9099          TIME_WAIT
tcp       25      0 192.168.1.162:50338     216.219.92.22:443       CLOSE_WAIT
tcp6       0      0 :::22                   :::*                    LISTEN
tcp6       0      0 :::16443                :::*                    LISTEN
udp        0      0 192.168.1.162:68        192.168.1.1:67          ESTABLISHED
udp        0      0 0.0.0.0:67              0.0.0.0:*
udp        0      0 0.0.0.0:4789            0.0.0.0:*
udp        0      0 224.0.0.251:5353        0.0.0.0:*
```

- `0.0.0.0:22` LISTEN — SSH daemon accepting connections on all IPv4 interfaces
- `127.0.0.1:3306` LISTEN — MySQL bound to loopback only; not reachable from outside the host
- `127.0.0.1:1313` LISTEN — Hugo dev server, loopback only
- `192.168.1.162:33370 → 199.38.167.130:443` ESTABLISHED — outbound HTTPS to an external host
- `127.0.0.1:59310 → 127.0.0.1:1313` ESTABLISHED — browser connected to the Hugo dev server on loopback
- `192.168.122.1:59516 → 192.168.122.200:22` ESTABLISHED — SSH session from the hypervisor into a VM on the 192.168.122.0/24 bridge
- `Recv-Q 25, CLOSE_WAIT` — the remote end closed the connection but 25 bytes remain unread; the local application has not called close yet
- `udp 192.168.1.162:68 → 192.168.1.1:67` ESTABLISHED — active DHCP lease; client port 68 bound to the server on port 67
- `udp 0.0.0.0:4789` — VXLAN overlay port listening; indicates a container overlay network is active
- `udp 224.0.0.251:5353` — mDNS multicast listener

Adding `-p` shows the owning process. A `-` in `PID/Program name` means the process is owned by another user and `sudo` was not used:

```bash
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1234/sshd
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      5678/mysqld
tcp        0      0 127.0.0.1:1313          0.0.0.0:*               LISTEN      9012/hugo
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      910/systemd-resolve
udp        0      0 0.0.0.0:67              0.0.0.0:*                           1456/dnsmasq
udp        0      0 0.0.0.0:4789            0.0.0.0:*                           -
```

#### TCP states

| State | Description |
| :--- | :--- |
| `LISTEN` | Waiting for incoming connections on a local port |
| `ESTABLISHED` | Connection is open and data flows in both directions |
| `TIME_WAIT` | Connection closed locally; waiting to absorb delayed packets before the port is reused |

Less common states appear during connection setup and teardown:

| State | Description |
| :--- | :--- |
| `SYN_SENT` | Client sent a SYN and is waiting for a SYN-ACK from the server |
| `SYN_RECV` | Server received a SYN and sent a SYN-ACK; waiting for the final ACK |
| `FIN_WAIT_1` | Sent a FIN; waiting for an ACK or a simultaneous FIN from the remote end |
| `FIN_WAIT_2` | Received the ACK of the FIN; waiting for the remote end's FIN |
| `CLOSE_WAIT` | Received a FIN from the remote end; waiting for the local application to close the socket |
| `LAST_ACK` | Sent a FIN after entering CLOSE_WAIT; waiting for the final ACK |
| `CLOSING` | Both sides sent a FIN simultaneously; waiting for the remote ACK |
| `CLOSE` | Connection fully closed; no longer tracked |

### lsof

`lsof` (list open files) lists every file a process has open, including network sockets. Use it when you need to answer a specific question: which process is holding a port open, which user owns a connection, or whether a port is shared across multiple processes. Where `ss` and `netstat` show the socket table, `lsof` shows the relationship between sockets and processes with more detail — including the user, file descriptor, and resolved hostnames.

```bash
sudo lsof -i :22    # show all processes with a socket on port 22
```

```bash
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd    1 root  126u  IPv4   7298      0t0  TCP *:ssh (LISTEN)
systemd    1 root  127u  IPv6   7302      0t0  TCP *:ssh (LISTEN)
sshd    1035 root    3u  IPv4   7298      0t0  TCP *:ssh (LISTEN)
sshd    1035 root    4u  IPv6   7302      0t0  TCP *:ssh (LISTEN)
sshd    7754 root    4u  IPv4  42499      0t0  TCP networking:ssh->_gateway:59516 (ESTABLISHED)
sshd    7852 ryan    4u  IPv4  42499      0t0  TCP networking:ssh->_gateway:59516 (ESTABLISHED)
```

A few things to note in this output:

- Both `systemd` (PID 1) and `sshd` (PID 1035) share the same `DEVICE` number (7298/7302), meaning they hold the same socket file descriptor. `systemd` socket activation opened the port; `sshd` inherited it at startup.
- `sshd` appears twice for the established session (PIDs 7754 and 7852): the root-owned parent process and the user-owned child process that handles the authenticated session after privilege drop.
- `networking:ssh->_gateway:59516` shows resolved hostnames rather than IPs. Add `-n` to suppress resolution and show numeric addresses.

## TCP handshake (L4)

TCP establishes a connection with a three-way handshake before any data flows. The handshake synchronizes sequence numbers between the two endpoints and confirms both sides are reachable and ready.

TCP tracks every byte in a stream with a *sequence number (SEQ)*. Each side picks a random *Initial Sequence Number (ISN)* at connection start to prevent sequence prediction attacks. The *acknowledgment number (ACK)* tells the other side which byte to send next: it is always the last received SEQ plus one.

### Three-way handshake

The client picks an ephemeral source port and sends the first packet to the server's fixed destination port with the SYN bit set and a random SEQ. The server replies with both SYN and ACK bits set, its own random SEQ, and an ACK equal to the client's ISN plus one. The client completes the handshake by sending an ACK.

```bash
Client :54321                         Server :443
     |                                     |
     |----  SYN  SEQ=1000  --------------> |  client initiates
     |                                     |
     |<---  SYN-ACK  SEQ=5000  ACK=1001   -|  server synchronizes
     |                                     |
     |----  ACK  SEQ=1001  ACK=5001  ----> |  client confirms
     |                                     |
     |<========= data exchange ===========>|
```

After the third packet, the connection is *ESTABLISHED* and data flows in both directions.

### Connection teardown

TCP closes each direction independently. Either side sends a *FIN* to signal it has no more data to send. The other side acknowledges the FIN, then sends its own FIN when it is also done. The initiating side acknowledges and enters *TIME_WAIT* for twice the maximum segment lifetime (typically 60 seconds) to absorb any delayed packets still in flight.

```bash
Client                                Server
     |                                     |
     |----  FIN  SEQ=1500  --------------> |  client done sending
     |                                     |
     |<---  ACK  ACK=1501  --------------- |  server acknowledges
     |                                     |
     |<---  FIN  SEQ=6000  --------------- |  server done sending
     |                                     |
     |----  ACK  ACK=6001  --------------> |  client acknowledges
```

The server may combine steps 2 and 3 into a single *FIN-ACK* packet when it has no remaining data to send.

### RST

A *RST (reset)* packet abruptly terminates a connection without the FIN sequence. The receiving side discards the connection immediately. No acknowledgment is required.

Common causes:

- A packet arrives for a port with no listening process
- A firewall drops mid-connection packets and injects a RST
- An application crashes and the OS clears its sockets
- A half-open connection is detected: one side believes the connection is open and the other does not

RST differs from FIN in that FIN is a graceful close that allows in-flight data to drain. RST discards everything immediately.

## Wireless

### LinSSID

LinSSID is a graphical WiFi scanner that displays nearby networks in a table and plots them on a channel utilization graph. Use it to identify which channels are congested before deploying an access point, or to verify that your AP is on the least-used channel in a dense environment.

Install it with:

```bash
apt install linssid
```

LinSSID requires root to open raw sockets for scanning:

```bash
sudo linssid
```

The main window has two panes. The top pane lists every visible network with columns for SSID, BSSID, channel, signal (dBm), security protocol, and detected vendor. The bottom pane shows a live channel graph where each network appears as a curve centered on its channel with a width matching its channel bandwidth. Overlapping curves indicate co-channel or adjacent-channel interference.

For 2.4 GHz networks, only channels 1, 6, and 11 are non-overlapping. If you see several networks on channel 6, move your AP to channel 1 or 11. For 5 GHz, the non-overlapping channel set is much larger, so congestion is less common but still visible in the graph.

### wavemon

{{< admonition "Driver compatibility" warning >}}
`wavemon` requires the legacy Wireless Extensions API. On Ubuntu 20.04 and later, many drivers have dropped this API in favor of nl80211. If `wavemon` returns `no supported wireless interfaces found`, your driver does not support Wireless Extensions and `wavemon` will not work with that adapter.
{{< /admonition >}}

`wavemon` is an ncurses terminal monitor for wireless interfaces. It shows real-time signal strength, noise level, link quality, and connection details without requiring monitor mode. Use it when you are connected to a network and want to watch signal conditions live. It is useful for walking a space to map coverage or diagnosing an unstable connection.

Install it with:

```bash
apt install wavemon
```

Launch it against your wireless interface:

```bash
wavemon -i wlan0
```

Running `wavemon` with no arguments uses the first wireless interface it finds.

`wavemon` has several screens you navigate with function keys:

| Key | Screen |
| :--- | :--- |
| `F1` | Info. Real-time signal, noise, SNR, bitrate, TX/RX statistics, and connection details. |
| `F2` | Level histogram. Scrolling graph of signal and noise over time. |
| `F3` | Scan list. All visible networks with SSID, BSSID, channel, and signal strength. |
| `F9` | Preferences. Configure interface, thresholds, and display options. |
| `F10` | Quit. |

The info screen shows the most useful values for diagnosing a live connection:

| Field | Description |
| :--- | :--- |
| Link quality | A ratio (for example, 63/70) derived from the driver's quality metric. Higher is better. |
| Signal level (dBm) | Received signal strength. -50 dBm is excellent; below -80 dBm expect packet loss. |
| Noise level (dBm) | Background RF noise. A large gap between signal and noise means a clean channel. |
| Bit rate | Current negotiated PHY rate between client and AP. |
| TX/RX | Cumulative transmitted and received bytes since association. |

### Kismet

{{< admonition "Older Ubuntu versions" note >}}
The package in the Ubuntu repository on versions 18.04 and earlier is the outdated pre-2019 Kismet. It has a different interface and lacks most features of the current release. Use the Kismet project repository instead.
{{< /admonition >}}

Kismet is a passive wireless network detector, packet sniffer, and intrusion detection system. It supports 802.11 WiFi, Bluetooth, Zigbee, and other RF protocols depending on your hardware. Unlike active scanners, Kismet never transmits. It only listens, which makes it invisible to the networks it observes.

Use Kismet when you need to discover all wireless networks in range (including those with hidden SSIDs), identify rogue access points, or capture raw frames for offline analysis in Wireshark.

Install it from the distribution package or from the Kismet project repository for the latest version:

```bash
# Debian/Ubuntu
apt install kismet

# From the Kismet repository (newer releases)
wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
echo "deb https://www.kismetwireless.net/repos/apt/release/$(lsb_release -sc) $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/kismet.list
sudo apt update && sudo apt install kismet
```

Add your user to the `kismet` group so you can run it without `sudo`:

```bash
sudo usermod -aG kismet $USER
```

Log out and back in for the group change to take effect.

#### Monitor mode

Kismet requires a wireless interface in *monitor mode* to capture raw 802.11 frames. A normal interface in *managed mode* only receives frames addressed to it. Monitor mode captures everything the radio hears.

```bash
# Put the interface into monitor mode
sudo ip link set wlan0 down
sudo iw dev wlan0 set type monitor
sudo ip link set wlan0 up

# Verify
iw dev wlan0 info
```

Kismet can also manage the interface mode itself when you pass `-c` at startup.

#### Usage

Start Kismet and point it at your wireless interface:

```bash
kismet -c wlan0
```

Kismet starts a web server at `http://localhost:2501`. Open it in a browser to access the full UI. On first run, it prompts you to set an admin password.

To run headless without the interactive terminal output:

```bash
kismet --no-ncurses -c wlan0
```

To capture from multiple interfaces simultaneously:

```bash
kismet -c wlan0 -c wlan1
```

Kismet writes logs automatically to the current directory. The `.kismet` file is its native database format. To capture in pcap format for Wireshark:

```bash
kismet -c wlan0 --log-types pcapng
```

Common things Kismet surfaces in the UI:

| Field | Description |
| :--- | :--- |
| SSID | Network name. Blank entries are networks broadcasting a null or hidden SSID. |
| BSSID | MAC address of the access point. |
| Channel | The 802.11 channel the AP is operating on. |
| Signal (dBm) | Received signal strength. Closer to 0 is stronger; below -80 dBm is weak. |
| Encryption | Security protocol in use: WPA2, WPA3, WEP, or open. |
| Clients | Number of devices currently associated with the AP. |

## Commands

### telnet

`telnet` opens a raw TCP connection to a host and port. Use it to test whether a port is open and accepting connections before you commit to debugging the application layer. Because `telnet` completes the three-way handshake, a successful connection confirms the port is reachable and that no firewall is blocking the path.

`telnet` is not installed by default on most modern systems. Install it with `apt install telnet` or `dnf install telnet`.

Common options:

| Option | Description |
| :--- | :--- |
| `-n <file>` | Log the session to a file |
| `-l <user>` | Specify a login name (used with telnet servers, not port testing) |

Common usages:

```bash
telnet <host> <port>          # test TCP connectivity to a port
telnet 192.168.1.1 80         # check if port 80 is open
telnet mail.example.com 25    # test SMTP reachability
telnet 10.0.0.1 443           # check if HTTPS port accepts connections
```

When `telnet` connects, it prints `Connected to <host>` and opens an interactive session. You can send raw protocol commands. For example, type `GET / HTTP/1.0` followed by two Enter presses to send a minimal HTTP request. Press `Ctrl+]` then type `quit` to close the session.

Connection outcomes:

| Output | Description |
| :--- | :--- |
| `Connected to <host>` | Three-way handshake completed. Port is open. |
| `Connection refused` | Nothing is listening on that port. |
| Hangs with no output | Port is filtered. A firewall is silently dropping packets. |

`telnet` sends data in plaintext and is not appropriate for production use or accessing services with credentials. For encrypted connections, use `openssl s_client -connect <host>:<port>` instead.

### netcat (nc)

`nc` (netcat) is a general-purpose TCP and UDP tool. Use it instead of `telnet` when you need to test UDP ports, scan a range of ports, or listen on a port to verify that a remote host can reach you. `nc` writes connection results to stderr, so most scan workflows redirect stderr to stdout with `2>&1` before piping to `grep`.

Common options:

| Option | Description |
| :--- | :--- |
| `-z` | Zero-I/O mode. Connect and immediately close. Use for port scanning. |
| `-v` | Verbose. Print connection success or refusal for each port. |
| `-u` | Use UDP instead of TCP. |
| `-l` | Listen mode. Wait for an incoming connection. |
| `-w <sec>` | Timeout in seconds per connection attempt. |

Common usages:

```bash
# Test a single TCP port
nc -zv 192.168.122.200 22

# Scan a port range (TCP)
nc -zv 192.168.122.200 1-1024

# Scan a port range and suppress refused ports — show only open ports
nc -zv 192.168.122.200 1-1024 2>&1 | grep -v 'refused'

# Full TCP port scan
nc -zv 192.168.122.200 1-65535 2>&1 | grep -v 'refused'

# Test a single UDP port
nc -u -zv 192.168.122.200 53

# Time a UDP range scan and show only successful ports
date ; nc -u -zv 192.168.122.200 1-1024 2>&1 | grep succeed ; date

# Listen on a port (server side)
nc -l 8080
```

For the single-port test, `nc` prints a success or refusal line and exits:

```bash
$ nc -zv 192.168.122.200 22
Connection to 192.168.122.200 22 port [tcp/ssh] succeeded!
```

Port range output includes a line per port. Without filtering, refused ports flood the output. Piping through `grep -v 'refused'` leaves only the open ports:

```bash
$ nc -zv 192.168.122.200 1-1024 2>&1 | grep -v 'refused'
Connection to 192.168.122.200 22 port [tcp/ssh] succeeded!
Connection to 192.168.122.200 80 port [tcp/http] succeeded!
```

UDP scanning is slower than TCP. With TCP, a closed port sends back an immediate RST. With UDP, a closed port may send an ICMP port-unreachable message, or return nothing at all if the firewall drops it silently. `nc` has to wait for a timeout on each non-responsive port. Wrapping the scan with `date` shows the total elapsed time so you can estimate how long a full UDP sweep will take.

#### Listen mode

Listen mode turns `nc` into a temporary server. Run it on the host you want to verify is reachable, then connect from the remote host:

```bash
# On the server
nc -l 8080

# On the client
nc 192.168.122.200 8080
```

If the connection succeeds, anything you type on one side appears on the other. This confirms the network path and any firewall rules allow traffic on that port, independent of whether an application is running. Press `Ctrl+C` to close both ends.

You can also combine listen mode with a loop to serve a file repeatedly. This is useful for testing that HTTP clients or load balancers can reach a host before a real web server is running:

```bash
bash -c 'while true; do cat index.html | nc -l -p 80 -q 1; done'
```

Each time a client connects, `nc` sends `index.html` and exits after one second of silence (`-q 1`). The `while` loop immediately restarts `nc` so the next client can connect. This is not a real web server: it sends raw file contents with no HTTP headers. It confirms that TCP connections to port 80 succeed and that the file transfers over the wire.

#### Send a file

Redirect a file into `nc` to send its contents to a remote host over TCP. This is useful for transferring files quickly without SSH or copying data to a host that is running a listener:

```bash
# Receiver (run first)
nc -l 1234 > received-file.txt

# Sender
nc -w 3 192.168.122.200 1234 < sent-file.txt
```

`-w 3` closes the connection after 3 seconds of inactivity so the sender exits cleanly once the file is fully transmitted. Without `-w`, `nc` waits indefinitely for more input.

### Nmap

`nmap` is a network scanner that goes well beyond what `nc` or `telnet` offer. Use it when you need to discover live hosts on a subnet, scan multiple ports at once, detect service versions, or identify which vendor made a device. Install it before use:

```bash
apt install nmap    # Debian/Ubuntu
dnf install nmap    # RHEL/Fedora
```

When `nmap` scans a local subnet, it resolves each MAC address to a vendor name using the same OUI database described in the MAC addresses section. This lets you quickly identify device types before probing ports.

UDP scanning requires `sudo` because `nmap` must craft raw IP packets to send probes and receive ICMP port-unreachable replies. The kernel restricts raw socket access to root. TCP scans can work unprivileged because they use the standard `connect()` system call, but UDP has no equivalent.

UDP also has no Layer 5 session concept. TCP's three-way handshake gives `nmap` a definitive answer: a SYN-ACK means open, a RST means closed. UDP is connectionless. A closed UDP port may return an ICMP unreachable message, or return nothing if a firewall is silently dropping it. When `nmap` receives no reply, it marks the port `open|filtered` because it cannot distinguish between a service quietly accepting packets and a firewall blocking them.

Common options:

| Option | Description |
| :--- | :--- |
| `-sn` | Host discovery only. Skip port scanning. |
| `-sU` | UDP scan. |
| `-sV` | Probe open ports to detect service and version. |
| `-p <port>` | Scan a specific port or range. |
| `--open` | Show only open ports in output. |
| `--reason` | Show why each port is in its reported state. |

Common usages:

```bash
# List host discovery options
nmap | grep -i ping

# Discover live hosts on a subnet without scanning ports
nmap -sn 192.168.122.0/24

# Find hosts with port 443 open
nmap -p 443 --open 192.168.122.0/24

# Show the reason for each port state
nmap -p 443 --reason 192.168.122.0/24

# UDP scan for DNS (port 53) across a subnet
nmap -sU -p 53 --open 192.168.122.0/24

# UDP scan with version detection and state reasons
sudo nmap -sU -p 53 --open -sV --reason 192.168.122.0/24
```

The version-detection scan produces output like this:

```bash
Starting Nmap 7.80 ( https://nmap.org ) at 2026-06-06 10:08 EDT
Nmap scan report for precision-5540 (192.168.122.1)
Host is up, received localhost-response.

PORT   STATE SERVICE REASON       VERSION
53/udp open  domain  udp-response dnsmasq 2.90

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 256 IP addresses (3 hosts up) scanned in 8.25 seconds
```

- `localhost-response`: the scanner is scanning its own host. The reply came from the loopback interface rather than the wire.
- `udp-response`: the port is open because the host sent back a UDP reply to the probe.
- `dnsmasq 2.90`: `-sV` identified the service by probing the open port and matching the response against Nmap's version database.
- 8.25 seconds for 256 addresses: UDP scans are slower than TCP because `nmap` must wait for ICMP unreachable replies or timeouts on non-responsive ports.

#### Tuning

By default, `nmap` adapts its timing to network conditions. You can override this for faster scans on trusted local networks, or slower scans when you need to avoid triggering rate limits or intrusion detection systems.

**Timing templates** are the quickest way to tune the scan. Each template sets parallelism, timeouts, and delays as a preset:

| Template | Description |
| :--- | :--- |
| `-T0` | Paranoid. One probe at a time, 5-minute delay between probes. For IDS evasion. |
| `-T1` | Sneaky. Slow sequential scanning. |
| `-T2` | Polite. Reduces bandwidth and load on the target. |
| `-T3` | Normal. Default behavior. |
| `-T4` | Aggressive. Assumes a fast, reliable network. Recommended for local scans. |
| `-T5` | Insane. Maximum speed. May drop results on lossy networks. |

For finer control, set individual timing parameters:

| Option | Description |
| :--- | :--- |
| `--min-parallelism <n>` | Keep at least *n* probes in flight simultaneously. |
| `--max-parallelism <n>` | Cap probes in flight at *n*. Useful for reducing load on fragile devices. |
| `--host-timeout <time>` | Abandon a host after this much time. Prevents slow hosts from stalling a wide scan. |
| `--initial-rtt-timeout <time>` | Starting estimate for how long to wait for a probe reply before calibration. |
| `--min-rtt-timeout <time>` | Floor for the adaptive round-trip timeout. Prevents `nmap` from dropping the timeout too low. |
| `--max-rtt-timeout <time>` | Cap for the adaptive round-trip timeout. |
| `--scan-delay <time>` | Minimum wait between probes sent to a single host. |
| `--max-scan-delay <time>` | Cap on the adaptive scan delay. |

Time values accept `ms` (milliseconds), `s` (seconds), and `m` (minutes). Examples:

```bash
# Fast scan on a local trusted network
nmap -T4 --min-parallelism 50 192.168.122.0/24

# Slow scan to avoid IDS detection
nmap -T1 --scan-delay 1s 192.168.122.0/24

# Set hard timeouts for a wide internet scan
nmap --host-timeout 30s --max-rtt-timeout 500ms --max-parallelism 20 192.168.122.0/24

# Reduce RTT floor on a low-latency LAN to speed up probing
nmap -T4 --min-rtt-timeout 50ms --max-rtt-timeout 200ms 192.168.122.0/24
```

Increasing parallelism speeds up wide subnet scans but can overwhelm slow switches or embedded devices. `--host-timeout` is most useful when scanning large ranges where a single unresponsive host would otherwise block the scan for minutes. `--scan-delay` is the primary knob for IDS evasion: most signature-based systems trigger on rapid port probes from a single source, and adding even a 100ms delay defeats many rate-based detectors.

#### Nmap scripts

Nmap includes a scripting engine (*NSE*) that runs Lua scripts from `/usr/share/nmap/scripts` against scan targets. Scripts extend `nmap` beyond port scanning into protocol-level queries, vulnerability checks, and service enumeration. Run a script with `--script=<name>`:

```bash
nmap --script=ssl-cert -p 443 192.168.122.0/24
```

##### Network discovery

| Script | Description |
| :--- | :--- |
| `path-mtu` | Discovers the path MTU between the scanner and target by probing with decreasing packet sizes. Useful for diagnosing fragmentation problems. |
| `broadcast-eigrp-discovery` | Sends EIGRP hello packets on the local segment and lists any routers that respond. Requires the scanner to be on the same broadcast domain as EIGRP speakers. |
| `broadcast-igmp-discovery` | Discovers multicast group memberships by sending IGMP queries and collecting membership reports from hosts on the segment. |
| `broadcast-ospf2-discover` | Sends OSPFv2 hello packets and lists routers that respond. Useful for mapping the OSPF topology without joining the process. |
| `broadcast-rip-discover` | Broadcasts a RIPv2 request and collects route advertisements from any routers that respond. |
| `broadcast-ripng-discover` | Same as `broadcast-rip-discover` but for RIPng (IPv6). |
| `broadcast-wpad-discover` | Queries the network for a Web Proxy Auto-Discovery (WPAD) server. A response here can indicate a misconfigured or rogue proxy. |
| `llmnr-resolve` | Resolves a hostname using LLMNR (Link-Local Multicast Name Resolution). Useful for identifying hosts that rely on LLMNR when DNS is unavailable. |

##### DNS

| Script | Description |
| :--- | :--- |
| `broadcast-dns-service-discovery` | Sends mDNS queries to discover services advertised on the local network. |
| `dns-srv-enum` | Enumerates DNS SRV records for common services such as LDAP, Kerberos, and SIP. Useful for mapping Active Directory and VoIP infrastructure. |
| `dns-recursion` | Checks whether a DNS server allows recursive queries from external hosts. An open recursive resolver can be abused for amplification attacks. |

##### DHCP

| Script | Description |
| :--- | :--- |
| `dhcp-discover` | Sends a DHCP discover packet to the target and displays the options the server returns, including offered IP, lease time, gateway, and DNS servers. |
| `broadcast-dhcp-discover` | Same as `dhcp-discover` but broadcasts on the local segment to find any DHCP server present. |
| `broadcast-dhcp6-discover` | Sends a DHCPv6 solicit message to discover IPv6 DHCP servers on the segment. |

##### Databases and application services

| Script | Description |
| :--- | :--- |
| `broadcast-ms-sql-discover` | Broadcasts a SQL Server Browser query and lists any Microsoft SQL Server instances that respond with their instance name and version. |
| `broadcast-sybase-discover` | Discovers Sybase database servers on the local segment. |
| `oracle-tns-version` | Connects to the Oracle TNS listener port (1521) and retrieves its version string. |
| `broadcast-db2-discover` | Broadcasts a discovery request for IBM DB2 instances. |
| `couchdb-databases` | Lists databases on a CouchDB instance. A server with no authentication exposes this without credentials. |
| `mongodb-info` | Connects to MongoDB and retrieves server info including version, hostname, and replication status. |
| `broadcast-jenkins-discover` | Broadcasts a Jenkins UDP discovery request and lists any Jenkins instances that respond with their URL and version. |
| `snmp-info` | Queries a host via SNMP and returns the system description, uptime, contact, and location from the MIB-II system group. |

##### VPN and remote access

| Script | Description |
| :--- | :--- |
| `ike-version` | Sends an IKEv1 or IKEv2 probe to UDP port 500 and retrieves the supported transforms and vendor ID. Useful for fingerprinting VPN endpoints. |
| `http-cisco-anyconnect` | Connects to a Cisco AnyConnect endpoint and retrieves the version string and platform information from the XML response. |

##### SSL/TLS

| Script | Description |
| :--- | :--- |
| `ssl-cert` | Retrieves and displays the SSL/TLS certificate for an open port. Shows subject, issuer, validity dates, and SANs. |
| `ssl-date` | Reads the server's clock from the TLS handshake and compares it to the scanner's clock. A large skew can indicate Kerberos issues or clock drift. |
| `ssl-dh-params` | Checks the Diffie-Hellman parameters used in the TLS handshake. Flags weak key sizes and known vulnerable parameter sets such as Logjam. |
| `ssl-enum-ciphers` | Enumerates all cipher suites the server accepts and rates them. Flags weak ciphers, export-grade ciphers, and protocol versions such as SSLv3 and TLS 1.0. |

##### Remote access protocols

| Script | Description |
| :--- | :--- |
| `rdp-enum-encryption` | Connects to an RDP port and identifies which encryption protocols and security layers the server supports, including Classic RDP, TLS, and CredSSP. |
| `ssh2-enum-algos` | Lists the key exchange, encryption, MAC, and compression algorithms advertised by an SSH server. Useful for identifying deprecated or weak algorithms. |
| `sshv1` | Checks whether a host accepts SSHv1 connections. SSHv1 is cryptographically broken and should not be present on any modern host. |
| `bitcoin-info` | Connects to a Bitcoin node and retrieves protocol version, block height, and peer connections. |

