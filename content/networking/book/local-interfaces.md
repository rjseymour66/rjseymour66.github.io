+++
title = 'Local Interfaces'
date = '2026-05-28T23:35:57-04:00'
weight = 10
draft = false
+++

## Network interfaces

A server uses its network interface to connect to a network. Interfaces come in three types: physical interfaces bound to hardware like an Ethernet card or wireless adapter, virtual interfaces created by the OS for tunnels, bridges, or VLANs, and the loopback interface (`lo`) used for internal communication within the host. At boot, the kernel detects hardware and registers physical interfaces, then the init system brings them up according to the network configuration files.

Each interface must have an IP address assigned before it can communicate on a network. You can assign addresses statically in the network configuration or dynamically through DHCP.

### Naming convention

Ubuntu uses a predictable naming convention for network interfaces based on the physical
location of the hardware. A name like `enp0s3` reflects the card's position on the
system bus and does not change between reboots unless you physically move the hardware.

Ethernet interfaces begin with `en`. Wireless interfaces begin with `wl`.

The name `enp0s3` identifies an Ethernet card on the system's first PCI bus in slot 3:

| Segment | Meaning                   |
| :------ | :------------------------ |
| `en`    | Ethernet                  |
| `p0`    | First PCI bus (0-indexed) |
| `s3`    | PCI slot 3                |

### ip

`ip` is part of the `iproute2` utility suite, which replaced the older `net-tools`
package and its `ifconfig` command. Use these commands to inspect and manage interfaces:

```bash
ip addr show                # view network interfaces and current status
ip a                        # shorthand for ip addr show
ip -4 a                     # show only IPv4
ip -6 a                     # show only IPv6
ip link set enp0s3 down     # bring the enp0s3 interface down
ip link set enp0s3 up       # bring the enp0s3 interface up
```

`ip -4 a` displays each interface with its IPv4 addresses and state:

```bash
ip -4 a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000    # 1
    inet 127.0.0.1/8 scope host lo                                                              # 2
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000    # 3
    inet 192.168.122.200/24 metric 100 brd 192.168.122.255 scope global dynamic enp1s0         # 4
       valid_lft 3413sec preferred_lft 3413sec
```

1. `lo` is the loopback interface, used for host-internal communication only. Traffic on `lo` never reaches a physical network.
   - `LOOPBACK` identifies it as the loopback type
   - `UP` means the interface is administratively enabled
   - `LOWER_UP` means the kernel sees a signal on the physical layer — always true for loopback
   - `mtu 65536` is the Maximum Transmission Unit — the largest packet in bytes the interface sends in a single frame. Loopback uses 65536 because there is no physical medium to constrain it; Ethernet uses 1500
   - `qdisc noqueue` is the queuing discipline — the algorithm the kernel uses to schedule outbound packets. Loopback uses `noqueue` because packets are delivered immediately with no buffering
   - `state UNKNOWN` means the kernel does not track link state for loopback the way it does for physical interfaces
   - `group default` is an administrative group label used to manage multiple interfaces together
   - `qlen 1000` is the transmit queue length — the number of packets the kernel buffers before it starts dropping them
2. `inet 127.0.0.1/8` is the loopback IPv4 address.
   - The `/8` prefix means the entire `127.0.0.0/8` block is reserved for loopback — traffic to any address in that range stays on the host and never leaves
   - `valid_lft forever` means the address has no expiration
3. `enp1s0` is a physical Ethernet interface.
   - `BROADCAST` means the interface can send frames to all hosts on the local network
   - `MULTICAST` means it supports group addressing for protocols like mDNS and routing updates
   - `UP` means the interface is administratively enabled
   - `LOWER_UP` means the interface is physically connected — a cable is plugged in and the link is active
   - `mtu 1500` is the standard Ethernet MTU
   - `qdisc pfifo_fast` is the default queuing discipline for Ethernet — it sorts packets into three priority bands so interactive traffic is not delayed behind bulk transfers
   - `state UP` confirms the physical link is detected
   - `qlen 1000` sets the transmit queue depth
4. `inet 192.168.122.200/24` is the IPv4 address in CIDR (Classless Inter-Domain Routing) notation.
   - The `/24` prefix means the first 24 bits identify the network (`192.168.122.0`), leaving 8 bits for host addresses — 254 usable hosts
   - `brd 192.168.122.255` is the subnet broadcast address
   - `dynamic` indicates DHCP assigned the address
   - `valid_lft 3413sec` shows the remaining DHCP lease time in seconds before the address must be renewed

### ifconfig

`ifconfig` is deprecated. It is part of the `net-tools` package, which `iproute2` has
replaced. Install it only if you need to support legacy scripts:

```bash
apt install net-tools       # install the package
ifconfig                    # view network interfaces
ifconfig enp0s3 down        # bring the enp0s3 interface down
ifconfig enp0s3 up          # bring the enp0s3 interface up
```

## Routers

The routing table tells the kernel where to send packets. When a packet leaves the host, the kernel looks up the destination IP in the routing table and always selects the most specific matching route. A route for `192.168.122.50` matches before `192.168.122.0/24`, which matches before `default`. If the destination isn't in the routing table at all, the kernel falls back to the default route and sends the packet to the configured gateway. Use `ip route` to view the routing table:

```bash
ip route
default via 192.168.122.1 dev enp1s0 proto dhcp src 192.168.122.200 metric 100         # 1
192.168.122.0/24 dev enp1s0 proto kernel scope link src 192.168.122.200 metric 100     # 2
192.168.122.1 dev enp1s0 proto dhcp scope link src 192.168.122.200 metric 100          # 3
```

1. The default route matches any destination that no more specific route covers. The kernel forwards those packets to the gateway.
   - `via 192.168.122.1` is the next-hop gateway IP address — the router the kernel sends packets to when no specific route matches
   - `dev enp1s0` is the outgoing interface
   - `proto dhcp` means the DHCP client installed this route automatically when it received a lease
   - `src 192.168.122.200` is the preferred source address the kernel uses when sending packets on this route
   - `metric 100` is the route cost — when multiple routes match a destination, the kernel prefers the one with the lowest metric
2. The connected network route tells the kernel that all hosts on `192.168.122.0/24` are directly reachable on the local segment — no routing to a gateway is needed. The kernel sends traffic to any address in that range directly onto the wire using ARP to resolve the destination MAC address.
   - `192.168.122.0/24` is the destination network in CIDR notation
   - `proto kernel` means the kernel added this route automatically when the interface was assigned its IP address
   - `scope link` means the destination is reachable directly on the local link — no gateway is needed
   - `src 192.168.122.200` is the preferred source address for traffic to this subnet
3. The link-local route is a host route (a `/32` entry covering exactly one IP address) for the gateway itself, installed by the DHCP client. In routing, *link-local* (`scope link`) means the kernel reaches the destination by sending a frame directly onto the wire — no L3 next-hop lookup or gateway is required, only ARP resolution on the local segment.

   A *local link* is the network segment a host's interface is directly attached to — the stretch of network between the host and the first router, shared by all devices on the same subnet. Traffic on the local link travels at Layer 2 using MAC addresses resolved through ARP, with no router involved. The term *link-local* in routing (`scope link`) and the IETF link-local address range share the same idea: neither crosses a router. `169.254.0.0/16`, defined in RFC 3927, is a reserved address range the OS uses as a fallback when DHCP fails. If a host boots with no static address and the DHCP discovery process gets no response, the OS assigns itself an address from that range so it can still communicate with other devices on the same physical segment. The first two octets (`169.254`) are fixed by the RFC and are never routed — any packet with a `169.254.x.x` source or destination address is dropped at the first router it hits. The OS picks the last two octets randomly and then ARP-probes the segment to confirm no other host is already using that address before binding it to the interface.
   - `192.168.122.1` is the gateway's IP address, covered by this host route
   - `scope link` means the destination is directly reachable on the local segment without a gateway
   - `proto dhcp` means the DHCP client installed this route so the host can reach the gateway to renew its lease, even before the default route is active
