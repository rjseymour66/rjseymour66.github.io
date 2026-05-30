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
2: wlp59s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000    # 3
    inet 192.168.1.162/24 brd 192.168.1.255 scope global dynamic noprefixroute wlp59s0         # 4
       valid_lft 86328sec preferred_lft 86328sec
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default    # 5
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
5: lxdbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000    # 6
    inet 10.69.129.1/24 scope global lxdbr0
       valid_lft forever preferred_lft forever
9: vxlan.calico: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default    # 7
    inet 10.1.224.0/32 scope global vxlan.calico
       valid_lft forever preferred_lft forever
291: virbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000    # 8
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
```

The interface index numbers (1, 2, 4, 5, 9, 291) are not sequential because the kernel assigns them incrementally and never reuses them. Gaps mark interfaces that were created and deleted since boot.

1. `lo` is the loopback interface, used for host-internal communication only. Traffic on `lo` never reaches a physical network.
   - `LOOPBACK` identifies it as the loopback type
   - `UP` means the interface is administratively enabled
   - `LOWER_UP` means the kernel sees a signal on the physical layer. For loopback, this is always true
   - `mtu 65536` is the Maximum Transmission Unit, the largest packet in bytes the interface sends in a single frame. Loopback uses 65536 because there is no physical medium to constrain it. Ethernet uses 1500
   - `qdisc noqueue` is the queuing discipline, the algorithm the kernel uses to schedule outbound packets. Loopback uses `noqueue` because packets are delivered immediately with no buffering
   - `state UNKNOWN` means the kernel does not track link state for loopback the way it does for physical interfaces
   - `group default` is an administrative group label used to manage multiple interfaces together
   - `qlen 1000` is the transmit queue length, the number of packets the kernel buffers before it starts dropping them
2. `inet 127.0.0.1/8` is the loopback IPv4 address.
   - The `/8` prefix means the entire `127.0.0.0/8` block is reserved for loopback. Traffic to any address in that range stays on the host and never leaves
   - `valid_lft forever` means the address has no expiration
3. `wlp59s0` is a wireless network interface. The `wl` prefix identifies it as wireless; `p59s0` places it on PCI bus 59, slot 0.
   - `BROADCAST` and `MULTICAST` mean the interface supports broadcast and multicast traffic
   - `UP` and `LOWER_UP` confirm the interface is enabled and has an active wireless association
   - `state UP` confirms the wireless link is established
4. `inet 192.168.1.162/24` is the IPv4 address assigned by DHCP.
   - `noprefixroute` tells the kernel not to automatically add a connected route for this prefix. NetworkManager adds and manages routes itself rather than letting the kernel do it automatically
   - `valid_lft 86328sec` is approximately 24 hours of remaining DHCP lease time
5. `docker0` is a virtual bridge created by the Docker daemon. It acts as the default gateway and software switch for Docker containers on this host. Each running container connects to it through a veth (virtual Ethernet) pair, with one end inside the container and one end attached to this bridge. `NO-CARRIER` and `state DOWN` mean no containers are currently attached. The host holds `172.17.0.1` as the gateway IP for container traffic.
6. `lxdbr0` is a virtual bridge created by the LXD container daemon, serving the same role as `docker0` for LXD system containers. `NO-CARRIER` and `state DOWN` mean no LXD containers are currently running and attached.
7. `vxlan.calico` is a VXLAN (Virtual Extensible LAN) overlay interface created by the Calico CNI (Container Network Interface) plugin in a Kubernetes cluster. VXLAN encapsulates Layer 2 Ethernet frames inside UDP packets, allowing pods on different nodes to communicate as if they were on the same local segment. This interface is the VTEP (VXLAN Tunnel Endpoint) for this node, the point where overlay traffic enters and exits. The `/32` address means it represents this specific node in the overlay network. The `mtu 1450` is 50 bytes less than standard Ethernet to account for VXLAN header overhead.
8. `virbr0` is a virtual bridge created by libvirt, the virtualization management library used by KVM/QEMU. It acts as a software switch and NAT gateway for virtual machines on this host. VMs connect to it through TAP interfaces that the hypervisor attaches to the bridge when a VM starts. The host holds `192.168.122.1` as the gateway for the VM network. `state UP` confirms at least one VM is currently attached.

### ifconfig

`ifconfig` is deprecated. It is part of the `net-tools` package, which `iproute2` has
replaced. Install it only if you need to support legacy scripts:

```bash
apt install net-tools       # install the package
ifconfig                    # view network interfaces
ifconfig enp0s3 down        # bring the enp0s3 interface down
ifconfig enp0s3 up          # bring the enp0s3 interface up
```

## Managing an interface

Ubuntu uses two tools to manage network interfaces. Knowing which tool owns an interface is required before making any configuration changes.

- **Netplan:** the default on Ubuntu Server. Reads YAML configuration files in `/etc/netplan/`. Changes require editing a file and running `netplan apply`.
- **NetworkManager (`nmcli`):** the default on Ubuntu Desktop and all Red Hat-based distributions (RHEL, CentOS, Fedora). Manages connections dynamically through connection profiles. Changes take effect immediately with `nmcli con up`.

### Identify the tool

Before configuring an interface, confirm which tool manages it:

```bash
systemctl status NetworkManager    # check if NetworkManager is running
nmcli device status                # show what NM is managing
```

If the interface shows `unmanaged`, NetworkManager is not controlling it. Netplan is managing it. Use Netplan to configure it. If the interface shows `connected` or `disconnected`, NetworkManager manages it. Use `nmcli`.

### Switching tools

#### Netplan to nmcli

Switch to nmcli when you need per-user connection profiles, want NetworkManager's automatic reconnect behavior, or are working on a desktop system where NetworkManager is already running. Netplan works well for servers with static configurations, but nmcli is better suited when connections change frequently or require more dynamic control.

##### Temporary

Hands the interface to NetworkManager until the next reboot. Use this to test nmcli without committing to a permanent change:

1. Run `nmcli device set enp1s0 managed yes`.

##### Permanent

Removes the Netplan config and gives NetworkManager full ownership.

Netplan processes every `.yaml` file in `/etc/netplan/` regardless of the filename prefix. To preserve a config without Netplan reading it, rename it with a `.bak` extension instead of deleting it:

```bash
sudo mv /etc/netplan/your-config.yaml /etc/netplan/your-config.yaml.bak
```

If you are connected over SSH, run `netplan apply` inside a `tmux` session. The apply temporarily drops the interface, which kills the SSH connection and leaves the command hanging in the frozen terminal. Run `tmux` before starting, so the command continues on the server if the connection drops.

1. Remove the Netplan configuration file:
   ```bash
   sudo rm /etc/netplan/your-config.yaml
   ```
2. Apply the change:
   ```bash
   sudo netplan apply
   ```
3. Restart NetworkManager:
   ```bash
   sudo systemctl restart NetworkManager
   ```
4. Configure the interface with `nmcli`.

To hand all interfaces to NetworkManager without removing Netplan files:

1. Add the following to `/etc/NetworkManager/NetworkManager.conf`:
   ```ini
   [keyfile]
   unmanaged-devices=none
   ```
2. Restart NetworkManager:
   ```bash
   sudo systemctl restart NetworkManager
   ```

If a Netplan config still exists for the interface, both tools will compete for control.

#### nmcli to Netplan

On a system where NetworkManager has been managing the interface, it auto-generates a `90-NM-<uuid>.yaml` file in `/etc/netplan/`. That file contains `renderer: NetworkManager`, which hands the interface back to NetworkManager when Netplan reads it. Delete it first, or the switch will not take effect.

1. Delete the auto-generated Netplan file:
   ```bash
   sudo rm /etc/netplan/90-NM-<uuid>.yaml
   ```
2. Delete the NetworkManager connection profile:
   ```bash
   nmcli con show                          # find the connection name
   nmcli con delete <connection-name>      # remove it
   ```
3. Create a Netplan config with `renderer: networkd`.
4. Apply the configuration:
   ```bash
   sudo netplan apply
   ```

NetworkManager releases the interface once networkd takes ownership. Confirm with `nmcli device status`. The interface should show `unmanaged`.

#### Auto-generated Netplan files

When NetworkManager manages an interface on a system that has Netplan, it writes its own YAML file to `/etc/netplan/` to keep the two tools in sync. The filename follows the pattern `90-NM-<uuid>.yaml`. You do not create or edit this file directly — NetworkManager owns it and regenerates it when the connection profile changes.

```yaml
network:
  version: 2
  ethernets:
    NM-2c1507d6-5f34-477a-92c8-9c6cf166d2aa:
      renderer: NetworkManager
      match:
        name: "enp1s0"
      addresses:
      - "192.168.122.200/24"
      nameservers:
        addresses:
        - 192.168.122.1
      dhcp6: true
      mtu: 9000
      routes:
      - to: "10.10.11.0/24"
        via: "192.168.122.11"
      networkmanager:
        uuid: "2c1507d6-5f34-477a-92c8-9c6cf166d2aa"
        name: "conn1"
        passthrough:
          ipv4.method: "manual"
```

- `renderer: NetworkManager` — tells Netplan to delegate this interface to NetworkManager rather than `networkd`. This is how the two tools coexist without conflict.
- `match: name: "enp1s0"` — binds the profile to the interface by name. NetworkManager uses a UUID as the YAML key and uses `match` to resolve it to the real interface.
- `networkmanager: passthrough:` — carries raw NetworkManager connection properties that have no direct Netplan equivalent, such as `ipv4.method: manual`. Netplan passes them through to NetworkManager untouched.
- The `90-` prefix means this file loads after the default `50-cloud-init.yaml` and takes precedence over it.

### Netplan

Netplan is Ubuntu Server's default network configuration tool. It reads YAML files from `/etc/netplan/` and generates configuration for the underlying renderer — `networkd` for servers. Netplan processes files in lexical order, so `99-config.yaml` applies after the default `50-cloud-init.yaml`. Changes require editing a file and running `sudo netplan apply`.

#### Assign static IP address

A static IP address stays fixed across reboots, which makes it useful for servers, printers, and any device you need to reach by a predictable address. When assigning a static address, choose one that falls outside the DHCP server's assignment range to avoid conflicts with dynamically assigned addresses. Alternatively, configure a static DHCP lease on the router, which assigns the same address every time the device connects.

The DNS server address is often the same as the default gateway but not always. Check your network configuration to confirm. If you apply changes over SSH, run `netplan apply` inside a `tmux` session so the command continues if the connection drops.

See the [Ubuntu docs for configuring networks](https://ubuntu.com/server/docs/configuring-networks#static-ip-address-assignment) for Ubuntu 24.04.

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      dhcp4: false
      addresses:
        - 192.168.122.100/24
      routes:
        - to: default
          via: 192.168.122.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
```

Apply the configuration:

```bash
sudo netplan apply
```

### nmcli

`nmcli` is the command-line interface for NetworkManager, the default network management tool on Ubuntu Desktop and all Red Hat-based distributions. It manages connections through persistent profiles and applies changes immediately without requiring a file edit. Install the package if it is not already present:

```bash
sudo apt install network-manager
```

#### Assign static IP address

A static IP address stays fixed across reboots. Choose one that falls outside the DHCP server's assignment range to avoid conflicts with dynamically assigned addresses.

Create a connection profile with a static address, gateway, and DNS server:

```bash
nmcli con add type <type> ifname <interface> con-name <profile-name> \
  ip4 <address>/<prefix> gw4 <gateway>
```

- `type` — the connection type. Use `ethernet` for wired interfaces
- `ifname` — the physical interface to bind the profile to (for example, `enp1s0`)
- `con-name` — the label NetworkManager uses to identify the profile. Convention is to match the interface name, but any name works
- `ip4` — the static IPv4 address and prefix length in CIDR notation
- `gw4` — the default gateway address

```bash
nmcli con add type ethernet ifname enp1s0 con-name enp1s0 \
  ip4 192.168.122.100/24 gw4 192.168.122.1
nmcli con mod enp1s0 ipv4.dns "8.8.8.8 1.1.1.1"
nmcli con up enp1s0
```

##### Disable DHCP

To disable DHCP on an existing connection, set `ipv4.method` to `manual` and assign a static address at the same time. The interface comes up with no IP if you disable DHCP without setting an address:

```bash
nmcli con mod <connection-name> ipv4.method manual \
  ipv4.addresses <address>/<prefix> \
  ipv4.gateway <gateway> \
  ipv4.dns "<dns>"
nmcli con up <connection-name>
```

For a KVM VM on the default `192.168.122.0/24` network, use the host bridge `192.168.122.1` as both the gateway and DNS server:

```bash
nmcli con mod enp1s0 ipv4.method manual \
  ipv4.addresses 192.168.122.100/24 \
  ipv4.gateway 192.168.122.1 \
  ipv4.dns "192.168.122.1"
nmcli con up enp1s0
```

##### Change IP address

{{< admonition "Changing IP over SSH" warning >}}
Changing the IP address drops your SSH connection immediately. The session is bound to the old address, and the interface comes up on the new address with no active session attached. Access the host through the console before running these commands, or accept that you will need to reconnect on the new address.
{{< /admonition >}}

You must set `ipv4.method manual` together with the address, gateway, and DNS in a single command. Setting the address alone leaves NetworkManager in DHCP mode, which ignores the static value you configured:

```bash
nmcli con mod <connection-name> ipv4.method manual \
  ipv4.addresses <address>/<prefix> \
  ipv4.gateway <gateway> \
  ipv4.dns "<dns>"
nmcli con up <connection-name>
```

For a KVM VM on the default `192.168.122.0/24` network:

```bash
nmcli con mod conn1 ipv4.method manual \
  ipv4.addresses 192.168.122.201/24 \
  ipv4.gateway 192.168.122.1 \
  ipv4.dns "192.168.122.1"
nmcli con up conn1
```

##### Rename a connection

Use this when a profile was created with a typo or a name that doesn't match the interface. The `NAME` column in `nmcli con show` is the `connection.id`:

```bash
nmcli con show                                          # find the current name
nmcli con mod <current-name> connection.id <new-name>   # rename it
```

##### Interactive editor

`nmcli connection edit` opens an interactive editor to configure a connection profile. Use it when you want to browse and set multiple properties without running separate `nmcli con mod` commands for each:

```bash
nmcli connection edit type ethernet        # create a new ethernet connection
nmcli connection edit <connection-name>    # edit an existing connection
```

Inside the editor, use these commands:

```
print all               # show all settings across every section
print <section>         # show settings for a specific section (e.g., print ipv4)
set <property> <value>  # set a property value
describe <property>     # show a description of a property
save                    # save the connection profile
quit                    # exit the editor
```

Common properties by section:

**Profile details (`connection`)**

| Property                    | Description                            |
| :-------------------------- | :------------------------------------- |
| `connection.id`             | Profile name shown in `nmcli con show` |
| `connection.interface-name` | Interface the profile binds to         |
| `connection.autoconnect`    | `yes` to connect automatically at boot |

**Ethernet (`802-3-ethernet`)**

| Property                     | Description             |
| :--------------------------- | :---------------------- |
| `802-3-ethernet.mtu`         | MTU size                |
| `802-3-ethernet.speed`       | Interface speed in Mbps |
| `802-3-ethernet.duplex`      | `full` or `half`        |
| `802-3-ethernet.mac-address` | MAC address to assign   |

**IPv4 (`ipv4`)**

| Property          | Description                                               |
| :---------------- | :-------------------------------------------------------- |
| `ipv4.method`     | `auto` for DHCP, `manual` for static                      |
| `ipv4.addresses`  | IP address and prefix (for example, `192.168.122.100/24`) |
| `ipv4.gateway`    | Default gateway                                           |
| `ipv4.dns`        | DNS servers, space-separated                              |
| `ipv4.dns-search` | DNS search domains                                        |
| `ipv4.routes`     | Static routes                                             |

**Proxy (`proxy`)**

| Property        | Description                                  |
| :-------------- | :------------------------------------------- |
| `proxy.method`  | `none`, `auto`, or `manual`                  |
| `proxy.pac-url` | URL to a PAC (Proxy Auto-Configuration) file |

##### Delete and recreate a connection

To delete a connection profile and recreate it, first find the exact profile name, then delete and recreate it:

```bash
nmcli con show                          # find the exact connection name
nmcli con delete <connection-name>      # delete the profile
nmcli con add type ethernet ifname enp1s0 con-name enp1s0
nmcli con up enp1s0
```

The name shown in `nmcli con show` is the name to pass to `con delete`. Typos in the original `con-name` argument carry into the stored profile, so always verify with `nmcli con show` before deleting.

Avoid configuring the same interface in both Netplan and NetworkManager. Mixing them causes conflicts.

### Test IP config

Verify the configuration in order: confirm the profile was saved, confirm the address is active on the interface, confirm the gateway route is present, then test connectivity:

```bash
nmcli con show <connection-name> | grep ipv4    # verify the profile settings
ip addr show <interface>                         # confirm the address is active on the interface
ip route                                         # confirm the gateway route is present
ping <gateway>                                   # test connectivity to the gateway
```

## Routes

The routing table tells the kernel where to send packets. When a packet leaves the host, the kernel looks up the destination IP in the routing table and always selects the most specific matching route. A route for `192.168.122.50` matches before `192.168.122.0/24`, which matches before `default`. If the destination isn't in the routing table at all, the kernel falls back to the default route and sends the packet to the configured gateway. Use `ip route` to view the routing table:

```bash
ip route
default via 192.168.122.1 dev enp1s0 proto dhcp src 192.168.122.200 metric 100         # 1
192.168.122.0/24 dev enp1s0 proto kernel scope link src 192.168.122.200 metric 100     # 2
192.168.122.1 dev enp1s0 proto dhcp scope link src 192.168.122.200 metric 100          # 3
```

1. The default route matches any destination that no more specific route covers. The kernel forwards those packets to the gateway.
   - `via 192.168.122.1` is the next-hop gateway IP address, the router the kernel sends packets to when no specific route matches
   - `dev enp1s0` is the outgoing interface
   - `proto dhcp` means the DHCP client installed this route automatically when it received a lease
   - `src 192.168.122.200` is the preferred source address the kernel uses when sending packets on this route
   - `metric 100` is the route cost. When multiple routes match a destination, the kernel prefers the one with the lowest metric
2. The connected network route tells the kernel that all hosts on `192.168.122.0/24` are directly reachable on the local segment. No routing to a gateway is needed. The kernel sends traffic to any address in that range directly onto the wire using ARP to resolve the destination MAC address.
   - `192.168.122.0/24` is the destination network in CIDR notation
   - `proto kernel` means the kernel added this route automatically when the interface was assigned its IP address
   - `scope link` means the destination is reachable directly on the local link. No gateway is needed
   - `src 192.168.122.200` is the preferred source address for traffic to this subnet
3. The link-local route is a host route (a `/32` entry covering exactly one IP address) for the gateway itself, installed by the DHCP client. In routing, *link-local* (`scope link`) means the kernel reaches the destination by sending a frame directly onto the wire. No L3 next-hop lookup or gateway is required, only ARP resolution on the local segment.

   A *local link* is the network segment a host's interface is directly attached to, the stretch of network between the host and the first router shared by all devices on the same subnet. Traffic on the local link travels at Layer 2 using MAC addresses resolved through ARP, with no router involved. The term *link-local* in routing (`scope link`) and the IETF link-local address range share the same idea: neither crosses a router. `169.254.0.0/16`, defined in RFC 3927, is a reserved address range the OS uses as a fallback when DHCP fails. If a host boots with no static address and the DHCP discovery process gets no response, the OS assigns itself an address from that range so it can still communicate with other devices on the same physical segment. The first two octets (`169.254`) are fixed by the RFC and are never routed. Any packet with a `169.254.x.x` source or destination address is dropped at the first router it hits. The OS picks the last two octets randomly and then ARP-probes the segment to confirm no other host is already using that address before binding it to the interface.
   - `192.168.122.1` is the gateway's IP address, covered by this host route
   - `scope link` means the destination is directly reachable on the local segment without a gateway
   - `proto dhcp` means the DHCP client installed this route so the host can reach the gateway to renew its lease, even before the default route is active

### Add a route

{{< admonition "Gateway vs next hop" note >}}
A *gateway* is the router a host uses to reach networks outside its local segment, typically installed as the default route. A next hop is not always the gateway. It can be any router on the local segment that knows the path to a specific network. In the home office example below, `192.168.1.1` is the default gateway for internet traffic, but `192.168.1.254` is a separate router used only as the next hop for the lab network. Both are routers, but only `192.168.1.1` is the gateway.
{{< /admonition >}}

Add a route when a host needs to reach a network that is not covered by the default gateway. Without an explicit route, the kernel sends traffic for that subnet to the default gateway, which may have no path to it. The destination is the network you want to reach. The next hop is the router the kernel hands the packet to first, and must be reachable on the local segment.

#### nmcli

```bash
nmcli con mod <connection-name> +ipv4.routes "<destination>/<prefix> <next-hop>"
nmcli con up <connection-name>
```

In a KVM setup, `virbr0` (`192.168.122.1`) is the host bridge that acts as the default gateway for all VMs on the `192.168.122.0/24` network. It also sits between KVM networks, so it can forward traffic from one to another. To reach a second KVM network (`192.168.100.0/24`) from a VM, add a route pointing to the host bridge as the next hop:

```bash
nmcli con mod enp1s0 +ipv4.routes "192.168.100.0/24 192.168.122.1"
nmcli con up enp1s0
```

In a small home office, you might have a main network at `192.168.1.0/24` and a separate lab or IoT network at `10.0.0.0/24` served by a second router at `192.168.1.254`. To reach the lab network from a workstation on the main network, add a route pointing to that second router as the next hop:

```bash
nmcli con mod enp1s0 +ipv4.routes "10.0.0.0/24 192.168.1.254"
nmcli con up enp1s0
```

#### Netplan

Add routes under the interface's `routes:` key. Each entry takes a `to` field for the destination and a `via` field for the next hop:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      routes:
        - to: 192.168.100.0/24
          via: 192.168.122.1
```

To add multiple routes, list each one as a separate entry:

```yaml
      routes:
        - to: default
          via: 192.168.122.1
        - to: 192.168.100.0/24
          via: 192.168.122.1
        - to: 10.0.0.0/24
          via: 192.168.1.254
```

Apply the configuration:

```bash
sudo netplan apply
```

#### Troubleshooting

If a route does not appear in `ip route` after applying:

1. Check whether the generated networkd config includes the route:
   ```bash
   cat /run/systemd/network/10-netplan-enp1s0.network
   ```
   Look for a `[Route]` section. If it is missing, the route did not make it from the Netplan YAML into the generated config. The most common cause is a YAML indentation error — the route block is silently skipped.
2. If the route is in the generated config but still absent from `ip route`, check the next hop. The next hop must be on the same subnet as the interface. A next hop on a different subnet is unreachable, so the kernel silently fails to install the route:
   ```bash
   networkctl status enp1s0    # check state and active routes
   ip route                    # verify the routing table
   ```
   If the interface is on `192.168.122.0/24`, the next hop must also be in that range — typically the gateway at `192.168.122.1`. A next hop like `192.168.1.254` from a different subnet will not install.
3. If the interface shows `State: routable (configuring)` and logs show repeated `DHCPv6 lease lost` messages, check whether `dhcp6: true` is set in the Netplan config and remove it if you are not using IPv6. Apply and verify:
   ```bash
   sudo netplan apply
   networkctl status enp1s0
   ```

### Remove a route

Remove a route when it is no longer needed or was added incorrectly. Use the same destination and next-hop values that were used to add it:

```bash
nmcli con mod <connection-name> -ipv4.routes "<destination>/<prefix> <next-hop>"
nmcli con up <connection-name>
```

```bash
nmcli con mod enp1s0 -ipv4.routes "192.168.100.0/24 192.168.122.1"
nmcli con up enp1s0
```

### Egress routes

An *egress route* controls the path outbound traffic takes when leaving the host. You add one when a host has multiple interfaces or gateways and you need to control which path specific traffic uses to exit the network.

Common reasons to add one:

- A VM has two interfaces and you want internet traffic to exit through a specific one
- You want traffic to a specific external network to leave through a different gateway than the default
- You want backup or management traffic to exit through a dedicated interface instead of competing with production traffic

There is no separate egress route command. The distinction is conceptual: any route that directs outbound traffic toward an external destination is an egress route.

#### nmcli

To send traffic to a specific external network through a non-default gateway:

```bash
nmcli con mod <connection-name> +ipv4.routes "<external-network>/<prefix> <gateway>"
nmcli con up <connection-name>
```

To change which gateway all outbound traffic uses:

```bash
nmcli con mod <connection-name> ipv4.gateway <new-gateway>
nmcli con up <connection-name>
```

#### Netplan

To send specific external traffic through a non-default gateway, add a route entry pointing to that gateway:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      routes:
        - to: 203.0.113.0/24
          via: 192.168.1.254
```

To change which gateway all outbound traffic uses, update the default route:

```yaml
      routes:
        - to: default
          via: 192.168.1.254
```

Apply the configuration:

```bash
sudo netplan apply
```

### Temporary routes

`ip route add` adds a route directly to the kernel routing table without touching NetworkManager or Netplan. The route is active immediately but does not survive a reboot or a `nmcli con up` call, which rewrites the routing table from the connection profile.

Use temporary routes to test a route before committing it to the connection profile:

```bash
ip route add <destination>/<prefix> via <next-hop>         # add a temporary route
ip route del <destination>/<prefix> via <next-hop>         # remove it
ip route show                                              # verify the routing table
```

```bash
ip route add 192.168.100.0/24 via 192.168.122.1
```

## MTU (Maximum Transmission Unit)

The *MTU* (Maximum Transmission Unit) is the largest IP packet an interface can send without fragmenting it. It covers the IP header and everything inside it. The Ethernet standard sets the default at 1500 bytes, a value chosen in the 1980s as a balance between efficiency and fairness on shared networks. Larger frames carry more data per header, but they also monopolize the wire longer on half-duplex links.

A *PDU* (Protocol Data Unit) is the name for the data unit at each layer of the network stack. At Layer 2, the PDU is a *frame*. At Layer 3, it is a *packet*. At Layer 4, it is a *segment* (TCP) or *datagram* (UDP). The MTU and MSS operate at different layers and serve different purposes.

The *MSS* (Maximum Segment Size) is a TCP-specific value that controls the largest amount of data in a single TCP segment, not counting the TCP or IP headers. The sender and receiver negotiate it during the three-way handshake. MSS does not apply to UDP.

| Component            | Size       | Notes                                              |
| :------------------- | :--------- | :------------------------------------------------- |
| Ethernet frame (max) | 1518 bytes | 14-byte header + 1500-byte payload + 4-byte FCS    |
| MTU                  | 1500 bytes | Payload portion of the Ethernet frame              |
| Max IP packet        | 1500 bytes | Fits exactly within the MTU                        |
| MSS                  | 1460 bytes | MTU minus 20-byte IP header and 20-byte TCP header |

### Jumbo frames

You raise the MTU when bulk data transfer efficiency matters more than compatibility with older hardware. Fewer, larger packets mean fewer headers, fewer interrupts, and less CPU overhead per byte transferred.

*iSCSI* (Internet Small Computer System Interface) runs SCSI storage commands over TCP/IP. A 1 MB I/O at the default 1500-byte MTU requires roughly 700 packets, each triggering a CPU interrupt and carrying its own header overhead. At 9000 bytes, the same transfer needs around 110 packets. Storage traffic is sequential and high-volume, so the reduction in interrupt load and header overhead translates directly into throughput.

The most common jumbo frame MTU is *9000 bytes*. This is not an IEEE standard — it is a vendor convention that became the industry default. All devices in the path must be configured to the same MTU. A mismatch causes packets to be fragmented silently or dropped, depending on whether the DF bit is set.

Jumbo frames are deployed on *10 Gbps and faster networks*, including 25 Gbps, 40 Gbps, and 100 Gbps data center fabrics, storage networks running iSCSI or NFS over Ethernet, and HPC (High Performance Computing) clusters.

Even larger MTUs appear in specialized environments. InfiniBand supports MTUs up to 65520 bytes. The Linux loopback interface (`lo`) uses 65536 bytes because there is no physical medium to constrain it.

### Smaller MTU

You reduce the MTU when the path between two hosts cannot carry full-size packets. The two most common causes are tunnel encapsulation and high-latency WAN links.

#### DF bit

The *DF* (Don't Fragment) bit is a flag in the IPv4 header. When set, it instructs every router in the path not to fragment the packet. If a router receives a packet with DF set and the outgoing link has a smaller MTU, it drops the packet and returns an ICMP "Fragmentation Needed" message to the sender. The sender uses that feedback to reduce its packet size. This process is called *Path MTU Discovery* (PMTUD).

Common scenarios where DF matters:

- **VPNs and tunnels:** IPsec adds 50 to 60 bytes of overhead per packet (ESP header, initialization vector, padding, authentication tag, and a new outer IP header). If the inner MTU stays at 1500 bytes, the outer packet grows to 1550 or more and exceeds the physical link MTU. With DF set, the router drops it. Without DF, it fragments the packet on every hop, which degrades performance. The correct fix is to reduce the inner MTU (typically to 1440 bytes for IPsec) or use TCP MSS clamping, which rewrites the MSS in the TCP handshake so endpoints never send segments large enough to cause fragmentation. VXLAN adds 50 bytes of overhead (8-byte VXLAN header, 8-byte UDP header, 20-byte outer IP header, 14-byte outer Ethernet header), so deployments typically set the inner MTU to 1450 bytes. GRE tunnels add at least 24 bytes.

  Encapsulation works by wrapping the original packet inside a new packet with its own headers. The outer packet is what travels across the physical network. Its total size must fit within the physical MTU. Because the outer headers consume space, the inner packet must shrink to compensate. If it does not, the outer packet is too large and gets fragmented or dropped.

- **Satellite links:** Geostationary satellites introduce round-trip latency of 600 milliseconds or more. On high-latency links, a corrupted or lost packet forces a long wait before retransmission. Smaller frames reduce the cost of that retransmission and limit the amount of data in flight that must be resent on error. A 512-byte frame size is common on satellite links. Smaller frames also reduce bufferbloat, where large queues of full-size packets inflate latency further on congested links.

### Setting the MTU

MTU is a property of the link itself, not of the IP protocol. The physical interface determines how large a frame can be, and the IP layer works within that constraint. Configuring MTU at the Ethernet layer tells NetworkManager or Netplan to set the interface MTU in the kernel, which applies to all traffic on the interface regardless of protocol. IPv4, IPv6, and any other protocol using the interface inherit the same limit.

#### nmcli

Set the MTU on a connection profile with `802-3-ethernet.mtu`, then bring the connection up to apply it. Verify the change with `ip -4 a`:

```bash
sudo nmcli con mod conn1 802-3-ethernet.mtu 9000    # set MTU to 9000 on the conn1 profile
sudo nmcli connection up conn1                       # apply the change by reactivating the connection
ip -4 a                                             # confirm the new MTU on the interface
```

- `802-3-ethernet.mtu 9000` sets the MTU to 9000 bytes on the connection profile. NetworkManager writes this to the interface when the connection is activated. The `802-3-ethernet` prefix identifies this as an Ethernet-layer setting rather than an IP-layer setting.
- `nmcli connection up conn1` reactivates the connection so NetworkManager applies the new MTU to the interface immediately. Without this, the change sits in the profile but does not take effect until the next reboot or reconnect.
- `ip -4 a` confirms the MTU is active. Look for `mtu 9000` on the interface line.

#### Netplan

Add `mtu` under the interface definition and apply the configuration:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      mtu: 9000
      dhcp4: false
      addresses:
        - 192.168.122.100/24
      routes:
        - to: default
          via: 192.168.122.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
```

Apply the configuration to activate the new MTU on the interface:

```bash
sudo netplan apply
```

##### Troubleshooting

If the MTU still shows the old value in `ip -4 a` after removing the `mtu` line and reapplying, `netplan apply` reconfigured the interface without tearing it down. Networkd applies the new config but does not reset omitted values to their kernel defaults — the MTU stays at whatever it was last explicitly set to.

Set it back explicitly in Netplan, apply, then remove the line:

```yaml
      mtu: 1500
```

```bash
sudo netplan apply
# remove the mtu line, then apply again
sudo netplan apply
```

To reset it immediately without editing the config:

```bash
sudo ip link set enp1s0 mtu 1500
ip -4 a
```

The `ip link` change takes effect immediately but does not persist. It resets on the next `netplan apply` or reboot. To make 1500 permanent, set it explicitly in Netplan, apply and confirm it shows 1500, then remove the line and apply once more:

```yaml
      mtu: 1500
```

```bash
sudo netplan apply     # apply mtu: 1500
ip -4 a                # confirm mtu 1500
# remove the mtu line from the config file
sudo netplan apply     # apply without the mtu line
ip -4 a                # confirm mtu is still 1500
```

## /etc/resolv.conf

{{< admonition "Not the preferred way to manage DNS" warning >}}
Do not edit `/etc/resolv.conf` or `/etc/systemd/resolved.conf` directly to configure DNS. Set DNS servers through Netplan or nmcli so the network manager owns the configuration and keeps it consistent across reboots and reconnects.
{{< /admonition >}}

`/etc/resolv.conf` is the system's DNS resolver configuration file. On Ubuntu, it is a symlink to `/run/systemd/resolve/stub-resolv.conf`, managed by `systemd-resolved`. Do not edit it directly.

```bash
nameserver 127.0.0.53
options edns0 trust-ad
search .
```

- `nameserver 127.0.0.53` — the `systemd-resolved` stub resolver running locally. Every DNS query goes here first, then `systemd-resolved` forwards it to the real upstream DNS server configured on the interface. It is a middleman, not the actual DNS server.
- `options edns0` — enables EDNS0 (Extension Mechanisms for DNS), which allows responses larger than the original 512-byte limit.
- `options trust-ad` — tells the stub to trust the Authenticated Data flag in responses, used by DNSSEC to signal a cryptographically validated answer.
- `search .` — the DNS search domain. A value of `.` means no suffix is appended to unqualified hostnames. If set to `search local.lan`, typing `ping fileserv` would automatically try `fileserv.local.lan`.

To see the actual upstream DNS server in use, run `resolvectl status`. The `DNS Servers` field under the active interface shows where queries are forwarded after hitting the stub.

The main `systemd-resolved` configuration file is `/etc/systemd/resolved.conf`. All settings are commented out by default, meaning the service runs on compile-time defaults and gets its upstream DNS server from the network interface configuration. The recommended way to customize it is to create a drop-in file in `/etc/systemd/resolved.conf.d/` rather than editing the main file directly.
