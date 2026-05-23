+++
title = 'Containerlab'
date = '2026-05-22T22:12:40-04:00'
weight = 60
draft = false
+++

*Containerlab* is a tool that builds network labs from Docker containers. You define your topology in a YAML file, run one command, and containerlab creates the containers, wires the virtual links, and assigns interface names. Tearing everything down takes the same single command. Because containers share the host kernel, labs start in seconds and use a fraction of the memory that GNS3 VMs require.

## When to use containerlab vs GNS3

Use containerlab when:

- You need a lab running in seconds from a file you can commit to git.
- Your practice focuses on routing protocols, automation, or writing code against network devices.
- RAM is limited — containers share the host kernel and need no hypervisor.
- You want to spin up and destroy identical environments repeatedly, as in CI/CD testing.

Use GNS3 when:

- You need full vendor images such as Cisco IOS-XE or Juniper Junos that only run under KVM or QEMU.
- You need hardware-accurate Layer 2 behavior from specific vendor switch ASICs.
- You prefer a visual canvas for building and understanding topologies.
- You are learning network fundamentals and benefit from seeing a graphical topology.

The two tools are complementary. Containerlab excels at fast, scripted, reproducible lab work. GNS3 excels at vendor-accurate emulation and visual learning.

## Hardware requirements

| Component | Minimum    | Recommended |
| --------- | ---------- | ----------- |
| RAM       | 4 GB       | 8 GB        |
| CPU       | 2 cores    | 4 cores     |
| Disk      | 10 GB free | 20 GB free  |

Containerlab runs on Linux only. It requires kernel 4.x or later and Docker. No hardware virtualization is needed.

## Software prerequisites

- Docker Engine 20.10 or later
- Linux host (Ubuntu 20.04+, Debian 11+, or similar)
- `bash` and `curl` for the installer

## Install containerlab

Install Docker if you have not already:

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
```

Log out and back in for the group change to take effect. Then install containerlab:

```bash
bash -c "$(curl -sL https://get.containerlab.dev)"
```

Verify the installation:

```bash
clab version
```

## Topology file structure

Every containerlab lab is defined by a YAML file. The minimum structure is a name, a list of nodes, and a list of links:

```yaml
name: my-lab

topology:
  nodes:
    r1:
      kind: linux
      image: frrouting/frr:latest
    pc1:
      kind: linux
      image: alpine:3.18

  links:
    - endpoints: ["r1:eth1", "pc1:eth0"]
```

`name` sets the lab name. Containerlab prefixes every container name with `clab-<name>-`, so the node `r1` in a lab named `my-lab` becomes the container `clab-my-lab-r1`.

`kind: linux` means a generic Linux container. `frrouting/frr:latest` is the *FRRouting* container image, which includes `vtysh` — the FRR configuration shell. `alpine:3.18` is a minimal Linux image used as a host.

`links` defines the virtual Ethernet pairs connecting the nodes. Each endpoint names a node and an interface on that node.

## Key commands

| Command                                     | What it does                                    |
| ------------------------------------------- | ----------------------------------------------- |
| `clab deploy -t <file>.yaml`                | Deploy a lab from a topology file               |
| `clab destroy -t <file>.yaml`               | Tear down the lab and remove all containers     |
| `clab inspect -t <file>.yaml`               | Show running nodes and their management IPs     |
| `clab inspect --all`                        | Show all running labs on this host              |
| `clab graph -t <file>.yaml`                 | Generate an HTML topology graph                 |
| `docker exec -it clab-<lab>-<node> sh`      | Open a shell on a node                          |

## Access nodes

Containerlab assigns each node a management IP in the `172.20.20.0/24` range. You can SSH into nodes at those addresses, or exec into them directly with Docker:

```bash
docker exec -it clab-my-lab-r1 sh
```

For FRR nodes, use `vtysh` to enter the routing shell:

```bash
docker exec -it clab-my-lab-r1 vtysh
```

## Configure FRR nodes

Inside `vtysh`, FRR uses a Cisco-like CLI. Enter configuration mode with `configure terminal`, make changes, then commit with `end` and save with `write memory`:

```
vtysh# configure terminal
vtysh(config)# interface eth1
vtysh(config-if)# ip address 10.0.1.1/24
vtysh(config-if)# no shutdown
vtysh(config-if)# end
vtysh# write memory
```

Show the routing table:

```
vtysh# show ip route
```

---

## Basic tasks

Complete these tasks before the projects. They teach the containerlab workflow so you are not learning tool mechanics and networking concepts at the same time.

---

### Task 1: Deploy a two-node lab

Create a file named `first-lab.yaml`:

```yaml
name: first

topology:
  nodes:
    node1:
      kind: linux
      image: alpine:3.18
    node2:
      kind: linux
      image: alpine:3.18

  links:
    - endpoints: ["node1:eth1", "node2:eth1"]
```

Deploy it:

```bash
clab deploy -t first-lab.yaml
```

Verify both containers are running:

```bash
docker ps | grep clab-first
```

You should see `clab-first-node1` and `clab-first-node2` in the output.

---

### Task 2: Assign addresses and ping between nodes

Open a shell on `node1` and assign an address to `eth1`:

```bash
docker exec -it clab-first-node1 sh
ip addr add 10.0.0.1/24 dev eth1
```

Open a separate terminal, exec into `node2`, and assign a different address on the same subnet:

```bash
docker exec -it clab-first-node2 sh
ip addr add 10.0.0.2/24 dev eth1
```

From `node2`, ping `node1`:

```bash
ping -c 3 10.0.0.1
```

Three replies confirm the virtual link is working.

---

### Task 3: Inspect the running lab

From your host, inspect the running lab to see node names, container IDs, and management IPs:

```bash
clab inspect -t first-lab.yaml
```

Note the management IP assigned to each node. Containerlab creates a management network (`172.20.20.0/24`) automatically, in addition to any data-plane links you define.

---

### Task 4: Generate and view the topology graph

Generate an HTML graph of the running lab:

```bash
clab graph -t first-lab.yaml
```

Open the URL printed in the output. The graph shows your two nodes connected by the link you defined in the YAML file.

---

### Task 5: Tear down and redeploy

Destroy the lab:

```bash
clab destroy -t first-lab.yaml
```

Verify the containers are gone:

```bash
docker ps | grep clab-first
```

No output confirms the lab is fully destroyed. Redeploy with the same command you used in Task 1. The lab starts from a clean state every time — no leftover state, no stale configuration.

---

### Task 6: Bind-mount a startup configuration

You can pre-load configuration into a node by bind-mounting a file at startup, which avoids manual configuration after every deploy.

Create a directory and a startup script:

```bash
mkdir -p lab-configs/node1
cat > lab-configs/node1/startup.sh << 'EOF'
#!/bin/sh
ip addr add 10.0.0.1/24 dev eth1
EOF
```

Update `first-lab.yaml` to mount and run the script on `node1`:

```yaml
name: first

topology:
  nodes:
    node1:
      kind: linux
      image: alpine:3.18
      binds:
        - lab-configs/node1/startup.sh:/startup.sh
      exec:
        - sh /startup.sh
    node2:
      kind: linux
      image: alpine:3.18

  links:
    - endpoints: ["node1:eth1", "node2:eth1"]
```

Deploy the lab. After startup, exec into `node1` and confirm `eth1` already has the address without any manual steps.

---

## Lesson 1: Two-router static routing

### Problem

Two routers each serve a LAN segment. Neither has any route to the other's LAN. A router only knows about networks directly connected to its own interfaces, so packets to the other LAN are dropped at the first hop. Before studying dynamic routing protocols, you need to understand what a routing table entry is, what a next hop means, and what happens when an entry is absent.

### Goal

Configure static routes on R1 and R2 so that hosts on each LAN can reach hosts on the other. Read the routing tables before and after adding routes, then remove one route, observe the failure, and restore connectivity.

### Topology

Create `static-routing.yaml`:

```yaml
name: static

topology:
  nodes:
    r1:
      kind: linux
      image: frrouting/frr:latest
    r2:
      kind: linux
      image: frrouting/frr:latest
    pc1:
      kind: linux
      image: alpine:3.18
    pc2:
      kind: linux
      image: alpine:3.18

  links:
    - endpoints: ["r1:eth1", "r2:eth1"]
    - endpoints: ["r1:eth2", "pc1:eth0"]
    - endpoints: ["r2:eth2", "pc2:eth0"]
```

IP addressing:

- R1 eth1: 10.0.0.1/30 (WAN link toward R2)
- R2 eth1: 10.0.0.2/30
- R1 eth2: 10.1.0.1/24 (LAN)
- PC1: 10.1.0.10/24, gateway 10.1.0.1
- R2 eth2: 10.2.0.1/24 (LAN)
- PC2: 10.2.0.10/24, gateway 10.2.0.1

Enable IP forwarding on both routers (`sysctl -w net.ipv4.ip_forward=1`). Configure all interface addresses. Do not add any routes yet.

### Success criteria

- R1's routing table contains a static route for 10.2.0.0/24 with a next hop of 10.0.0.2. R2's table contains a static route for 10.1.0.0/24 with a next hop of 10.0.0.1.
- PC1 can ping PC2.
- After removing one static route, you can identify exactly which traffic breaks, explain why from the routing table, and restore connectivity.

---

## Lesson 2: OSPF single area

### Problem

Three routers each serve a LAN segment. With static routes, every topology change requires manual updates on every router. Adding a fourth router means touching every existing router. A link-state protocol like OSPF eliminates that by having routers discover neighbors and compute routes automatically. You need to understand how OSPF forms adjacencies, floods link-state advertisements, and builds a routing table before you can troubleshoot it.

### Goal

Configure OSPF area 0 on all three routers so they share LAN routes automatically. No static routes should exist between the routers.

### Topology

Create `ospf-area0.yaml`:

```yaml
name: ospf

topology:
  nodes:
    r1:
      kind: linux
      image: frrouting/frr:latest
    r2:
      kind: linux
      image: frrouting/frr:latest
    r3:
      kind: linux
      image: frrouting/frr:latest
    pc1:
      kind: linux
      image: alpine:3.18
    pc2:
      kind: linux
      image: alpine:3.18
    pc3:
      kind: linux
      image: alpine:3.18

  links:
    - endpoints: ["r1:eth1", "r2:eth1"]
    - endpoints: ["r2:eth2", "r3:eth1"]
    - endpoints: ["r1:eth2", "r3:eth2"]
    - endpoints: ["r1:eth3", "pc1:eth0"]
    - endpoints: ["r2:eth3", "pc2:eth0"]
    - endpoints: ["r3:eth3", "pc3:eth0"]
```

IP addressing:

- R1–R2 link: 10.0.12.0/30 (R1 at .1, R2 at .2)
- R2–R3 link: 10.0.23.0/30 (R2 at .1, R3 at .2)
- R1–R3 link: 10.0.13.0/30 (R1 at .1, R3 at .2)
- R1 LAN: 10.1.0.0/24 (R1 at .1, PC1 at .10)
- R2 LAN: 10.2.0.0/24 (R2 at .1, PC2 at .10)
- R3 LAN: 10.3.0.0/24 (R3 at .1, PC3 at .10)

Enable IP forwarding and the `ospfd` daemon on each router. In each FRR container, edit `/etc/frr/daemons` and set `ospfd=yes`, then start FRR with `service frr start`. Configure all interface addresses. Do not add any static routes.

### Success criteria

- Every router shows the other two as OSPF neighbors in FULL state (`show ip ospf neighbor`).
- Each router's routing table contains all three LAN subnets learned through OSPF, not added manually.
- PC1 can ping PC2 and PC3 with no static routes on any router.

---

## Lesson 3: eBGP peering between two autonomous systems

### Problem

R1 and R2 represent routers in separate organizations. Each controls its own routing policy and decides which prefixes to share. OSPF assumes a single administrative domain and shares full topology information — that model is wrong here. BGP is the protocol of inter-organization routing. It lets each router advertise only its own prefix and apply policy at the boundary. This is the model underlying every internet routing handoff.

### Goal

Establish an eBGP session between R1 (AS 65001) and R2 (AS 65002). Each router advertises its LAN prefix and learns the other's prefix through BGP. Hosts on each LAN reach the other LAN with no static routes on either router.

### Topology

Create `ebgp.yaml`:

```yaml
name: ebgp

topology:
  nodes:
    r1:
      kind: linux
      image: frrouting/frr:latest
    r2:
      kind: linux
      image: frrouting/frr:latest
    pc1:
      kind: linux
      image: alpine:3.18
    pc2:
      kind: linux
      image: alpine:3.18

  links:
    - endpoints: ["r1:eth1", "r2:eth1"]
    - endpoints: ["r1:eth2", "pc1:eth0"]
    - endpoints: ["r2:eth2", "pc2:eth0"]
```

IP addressing:

- R1 eth1: 10.0.0.1/30 (eBGP peering link)
- R2 eth1: 10.0.0.2/30
- R1 LAN: 10.1.0.0/24 (R1 at .1, PC1 at .10)
- R2 LAN: 10.2.0.0/24 (R2 at .1, PC2 at .10)

Enable `bgpd` in `/etc/frr/daemons` on both routers. Enable IP forwarding. Configure all interface addresses. Do not add any static routes.

### Success criteria

- `show bgp summary` on both routers shows the peer in Established state with a non-zero prefix count.
- R1's routing table shows 10.2.0.0/24 learned through BGP. R2's table shows 10.1.0.0/24 learned through BGP.
- PC1 can ping PC2 with no static routes on either router.

---

## Lesson 4: Route redistribution from OSPF into BGP

### Problem

R1 runs OSPF with two internal routers (R2 and R3) that each serve a LAN. An upstream peer (R-ISP) runs BGP and knows nothing about the internal OSPF topology. R-ISP needs to reach R2's and R3's LAN subnets through R1, but you cannot manually add a static route on R1 for every internal subnet because the internal topology changes over time. Route redistribution solves this by taking routes learned from one protocol and injecting them into another automatically.

### Goal

Configure OSPF area 0 between R1, R2, and R3 so they share internal routes. Then configure eBGP between R1 and R-ISP. Redistribute OSPF routes into BGP on R1 so R-ISP learns the internal subnets through BGP without any static routes.

### Topology

Create `redistribution.yaml`:

```yaml
name: redis

topology:
  nodes:
    r1:
      kind: linux
      image: frrouting/frr:latest
    r2:
      kind: linux
      image: frrouting/frr:latest
    r3:
      kind: linux
      image: frrouting/frr:latest
    risp:
      kind: linux
      image: frrouting/frr:latest
    pc2:
      kind: linux
      image: alpine:3.18
    pc3:
      kind: linux
      image: alpine:3.18

  links:
    - endpoints: ["r1:eth1", "r2:eth1"]
    - endpoints: ["r1:eth2", "r3:eth1"]
    - endpoints: ["r1:eth3", "risp:eth1"]
    - endpoints: ["r2:eth2", "pc2:eth0"]
    - endpoints: ["r3:eth2", "pc3:eth0"]
```

IP addressing:

- R1–R2 OSPF link: 10.0.12.0/30 (R1 at .1, R2 at .2)
- R1–R3 OSPF link: 10.0.13.0/30 (R1 at .1, R3 at .2)
- R1–R-ISP BGP link: 10.0.0.0/30 (R1 at .1, R-ISP at .2)
- R2 LAN: 10.2.0.0/24 (R2 at .1, PC2 at .10)
- R3 LAN: 10.3.0.0/24 (R3 at .1, PC3 at .10)
- R1 AS: 65000, R-ISP AS: 65100

Enable `ospfd` and `bgpd` on R1. Enable only `ospfd` on R2 and R3. Enable only `bgpd` on R-ISP. Enable IP forwarding on all routers.

### Success criteria

- R1 shows R2 and R3 as OSPF neighbors in FULL state.
- R1's BGP session with R-ISP shows Established state.
- R-ISP's routing table shows 10.2.0.0/24 and 10.3.0.0/24 as BGP-learned routes with no static routes on R-ISP or R1.
- A host attached to R-ISP can ping PC2 and PC3.

---

## Lesson 5: ECMP load balancing with OSPF

### Problem

R1 and R2 are connected by two parallel links. OSPF assigns equal cost to both links. When a packet arrives at R1 destined for R2's LAN, the router has two equal-cost next hops. Without ECMP (equal-cost multipath), the router picks one path and ignores the other — you get redundancy but not load distribution. ECMP activates both paths simultaneously and distributes traffic across them, which effectively doubles throughput between R1 and R2.

### Goal

Configure OSPF across a topology where R1 reaches R2 through two equal-cost parallel links. Verify that OSPF installs both paths in the routing table simultaneously. Then confirm that OSPF reconverges to the remaining path when you take one link down.

### Topology

Create `ecmp.yaml`:

```yaml
name: ecmp

topology:
  nodes:
    r1:
      kind: linux
      image: frrouting/frr:latest
    r2:
      kind: linux
      image: frrouting/frr:latest
    pc1:
      kind: linux
      image: alpine:3.18
    pc2:
      kind: linux
      image: alpine:3.18

  links:
    - endpoints: ["r1:eth1", "r2:eth1"]
    - endpoints: ["r1:eth2", "r2:eth2"]
    - endpoints: ["r1:eth3", "pc1:eth0"]
    - endpoints: ["r2:eth3", "pc2:eth0"]
```

IP addressing:

- R1–R2 link A: 10.0.1.0/30 (R1 at .1, R2 at .2)
- R1–R2 link B: 10.0.2.0/30 (R1 at .1, R2 at .2)
- R1 LAN: 10.1.0.0/24 (R1 at .1, PC1 at .10)
- R2 LAN: 10.2.0.0/24 (R2 at .1, PC2 at .10)

Enable `ospfd` and IP forwarding on both routers. Set identical OSPF costs on both inter-router links. Do not configure any static routes.

### Success criteria

- `show ip route` on R1 shows two equal-cost next hops for 10.2.0.0/24, one per link.
- PC1 can ping PC2.
- After running `ip link set eth1 down` inside R1's container, OSPF reconverges and PC1 can still reach PC2 through the remaining link within 30 seconds.
- After restoring the link, both next hops reappear in the routing table.

---

## Lesson 6: NAT masquerade on a border router

### Problem

PC1 and PC2 have RFC 1918 addresses that are not routable on the external network. External has no route back to 10.0.0.x. Beyond reachability, PC1 hosts a service that must be accessible from External, even though PC1 has no public address. You need outbound address translation for both internal hosts and inbound port forwarding to PC1's service.

### Goal

Configure MASQUERADE (PAT) on R1's external interface so both internal hosts appear to come from R1's public IP when reaching External. Then configure a DNAT rule forwarding inbound TCP on port 8080 to PC1's internal address.

### Topology

Create `nat.yaml`:

```yaml
name: nat

topology:
  nodes:
    r1:
      kind: linux
      image: frrouting/frr:latest
    pc1:
      kind: linux
      image: alpine:3.18
    pc2:
      kind: linux
      image: alpine:3.18
    external:
      kind: linux
      image: alpine:3.18

  links:
    - endpoints: ["r1:eth1", "external:eth0"]
    - endpoints: ["r1:eth2", "pc1:eth0"]
    - endpoints: ["r1:eth3", "pc2:eth0"]
```

IP addressing:

- R1 eth1: 203.0.113.1/30 (external-facing)
- External eth0: 203.0.113.2/30
- R1 eth2: 10.0.0.1/24
- PC1: 10.0.0.10/24, gateway 10.0.0.1
- R1 eth3: 10.0.1.1/24
- PC2: 10.0.1.10/24, gateway 10.0.1.1

Enable IP forwarding on R1. Configure all interface addresses. Start a simple HTTP service on PC1 on port 8080 (`busybox httpd -p 8080 -h /tmp`). Do not configure NAT yet — confirm External can reach R1's eth1 address first. Use `iptables` on R1 for NAT.

### Success criteria

- PC1 and PC2 can reach External. A `tcpdump` on External's `eth0` shows traffic arriving from 203.0.113.1, not from 10.0.0.x.
- `iptables -t nat -L -n` on R1 shows an active MASQUERADE rule and a DNAT rule for port 8080.
- External can connect to the service on PC1 by sending a request to 203.0.113.1 on port 8080.

---

## Lesson 7: WireGuard site-to-site VPN

### Problem

Site A (10.1.0.0/24) and Site B (10.2.0.0/24) must communicate over a WAN link you do not control. Anyone monitoring the WAN can read traffic in transit. You need to encrypt all traffic between the two sites so it is unreadable to any observer on the WAN segment.

### Goal

Configure a *WireGuard* tunnel between GW-A and GW-B. All traffic between the two LAN subnets travels through the encrypted tunnel automatically with no configuration needed on the hosts.

### Topology

Create `wireguard.yaml`:

```yaml
name: wg

topology:
  nodes:
    gwa:
      kind: linux
      image: alpine:3.18
      exec:
        - apk add -q wireguard-tools iproute2
        - sysctl -w net.ipv4.ip_forward=1
    gwb:
      kind: linux
      image: alpine:3.18
      exec:
        - apk add -q wireguard-tools iproute2
        - sysctl -w net.ipv4.ip_forward=1
    pca:
      kind: linux
      image: alpine:3.18
    pcb:
      kind: linux
      image: alpine:3.18

  links:
    - endpoints: ["gwa:eth1", "gwb:eth1"]
    - endpoints: ["gwa:eth2", "pca:eth0"]
    - endpoints: ["gwb:eth2", "pcb:eth0"]
```

IP addressing:

- GW-A eth1: 203.0.113.1/30 (WAN)
- GW-B eth1: 203.0.113.2/30
- GW-A eth2: 10.1.0.1/24 (Site A LAN)
- PC-A: 10.1.0.10/24, gateway 10.1.0.1
- GW-B eth2: 10.2.0.1/24 (Site B LAN)
- PC-B: 10.2.0.10/24, gateway 10.2.0.1

On each gateway, generate a key pair with `wg genkey | tee privatekey | wg pubkey > publickey`. Create `/etc/wireguard/wg0.conf` on GW-A:

```ini
[Interface]
Address = 192.168.100.1/30
ListenPort = 51820
PrivateKey = <gwa-private-key>

[Peer]
PublicKey = <gwb-public-key>
AllowedIPs = 10.2.0.0/24
Endpoint = 203.0.113.2:51820
```

Create the mirror configuration on GW-B. Bring the tunnel up with `wg-quick up wg0` on both gateways. Add routes on each gateway directing LAN traffic into the tunnel.

Verify the gateways can ping each other on the WAN interface before starting WireGuard.

### Success criteria

- `wg show` on both gateways shows the peer with a non-zero `transfer` counter.
- PC-A can ping PC-B.
- A `tcpdump -i eth1` capture on GW-A during a ping shows UDP packets on port 51820, not plaintext ICMP. No readable host traffic is visible in the WAN capture.

---

## Lesson 8: BGP prefix filtering with route maps

### Problem

R1 advertises four prefixes to R2: two that R2 should accept and two that represent internal subnets R1 should not share with an external peer. Without filtering, all four reach R2's routing table. BGP policy enforcement uses prefix lists (which match specific prefixes) combined with route maps (which apply the permit or deny action). Filtering what you advertise outbound is a basic requirement of any real BGP configuration.

### Goal

Configure a BGP session between R1 and R2. Apply an outbound route map on R1 that permits only the two public-facing prefixes and denies the internal ones. Verify that R2 sees only the permitted prefixes.

### Topology

Create `bgp-filter.yaml`:

```yaml
name: bgpfilter

topology:
  nodes:
    r1:
      kind: linux
      image: frrouting/frr:latest
    r2:
      kind: linux
      image: frrouting/frr:latest

  links:
    - endpoints: ["r1:eth1", "r2:eth1"]
```

IP addressing:

- R1 eth1: 10.0.0.1/30
- R2 eth1: 10.0.0.2/30
- R1 AS: 65001, R2 AS: 65002

Configure R1 to advertise the following prefixes into BGP by adding them as static null routes (`ip route 10.1.0.0/24 blackhole`):

- 10.1.0.0/24 — public prefix, should reach R2
- 10.2.0.0/24 — public prefix, should reach R2
- 172.16.0.0/24 — internal, must not leave AS 65001
- 192.168.1.0/24 — internal, must not leave AS 65001

Enable `bgpd` on both routers.

### Success criteria

- `show bgp summary` on both routers shows the session in Established state.
- `show ip bgp` on R2 shows 10.1.0.0/24 and 10.2.0.0/24 with R1 as the next hop.
- `show ip bgp` on R2 does not contain 172.16.0.0/24 or 192.168.1.0/24.
- After removing the outbound route map from R1, all four prefixes appear on R2, confirming the filter was the cause and not a BGP advertisement issue.

---

## Lesson 9: iBGP with a route reflector

### Problem

Three routers in AS 65000 need to exchange routes using iBGP. The rule of iBGP is that a router will not re-advertise a route it learned from one iBGP peer to another iBGP peer. Without a route reflector, every router must peer directly with every other router. With four routers that means six sessions. With ten routers it means 45 sessions. A *route reflector* breaks the full-mesh requirement by acting as a central hub that re-advertises iBGP routes to its clients.

### Goal

First configure a full-mesh iBGP among R1, R2, and R3 and verify routes propagate. Then remove the full mesh, designate R1 as a route reflector with R2 and R3 as clients, and verify the same routes propagate with fewer sessions.

### Topology

Create `ibgp-rr.yaml`:

```yaml
name: ibgprr

topology:
  nodes:
    r1:
      kind: linux
      image: frrouting/frr:latest
    r2:
      kind: linux
      image: frrouting/frr:latest
    r3:
      kind: linux
      image: frrouting/frr:latest

  links:
    - endpoints: ["r1:eth1", "r2:eth1"]
    - endpoints: ["r1:eth2", "r3:eth1"]
    - endpoints: ["r2:eth2", "r3:eth2"]
```

IP addressing:

- R1–R2 link: 10.0.12.0/30 (R1 at .1, R2 at .2)
- R1–R3 link: 10.0.13.0/30 (R1 at .1, R3 at .2)
- R2–R3 link: 10.0.23.0/30 (R2 at .1, R3 at .2)
- R1 loopback: 1.1.1.1/32
- R2 loopback: 2.2.2.2/32
- R3 loopback: 3.3.3.3/32

All three routers are in AS 65000. Use loopback addresses as BGP router IDs and peering source. Configure OSPF to distribute loopback reachability before setting up iBGP. Enable `ospfd` and `bgpd` on all three routers.

### Success criteria

**Full-mesh phase:**

- All three routers show each other as Established iBGP peers.
- Each router's BGP table contains the loopback prefixes of the other two.

**Route reflector phase:**

- Remove the direct R2–R3 iBGP session.
- Configure R1 as a route reflector with R2 and R3 as route reflector clients.
- R2 learns R3's loopback prefix through R1. R3 learns R2's loopback prefix through R1.
- `show bgp neighbors <ip> advertised-routes` on R1 confirms it re-advertises client routes to other clients.

---

## Lesson 10: Multi-layer troubleshooting scenario

### Problem

PC1 cannot reach PC2. You do not know which layer is broken or how many layers are broken simultaneously. Guessing and trying random fixes is slow and unreliable. A systematic approach works from the bottom of the OSI model upward: form a hypothesis at each layer based on what you observe, confirm it with a test, fix it, and move to the next layer. This is the diagnostic discipline that separates engineers who resolve incidents quickly from those who thrash.

### Goal

Restore full end-to-end connectivity from PC1 to PC2 by diagnosing and fixing all four injected faults in order from L1 to L4. Record each fault as a written paragraph before moving to the next layer.

### Topology

Create `troubleshoot.yaml`:

```yaml
name: trouble

topology:
  nodes:
    r1:
      kind: linux
      image: frrouting/frr:latest
    r2:
      kind: linux
      image: frrouting/frr:latest
    pc1:
      kind: linux
      image: alpine:3.18
    pc2:
      kind: linux
      image: alpine:3.18

  links:
    - endpoints: ["r1:eth1", "r2:eth1"]
    - endpoints: ["r1:eth2", "pc1:eth0"]
    - endpoints: ["r2:eth2", "pc2:eth0"]
```

IP addressing:

- R1 eth1: 10.0.0.1/30 (WAN link to R2)
- R2 eth1: 10.0.0.2/30
- R1 LAN: 10.1.0.0/24 (R1 at .1, PC1 at .10)
- R2 LAN: 10.2.0.0/24 (R2 at .1, PC2 at .10)

Build the topology, configure all addresses and static routes, and verify PC1 can ping PC2 before injecting any faults. Then inject the four faults below and close all container shells before starting the lesson.

#### Faults to inject

| Layer | What to do                                                                                   |
| ----- | -------------------------------------------------------------------------------------------- |
| L1    | Inside R1's container, run `ip link set eth1 down` to disable the WAN interface             |
| L2    | Inside PC1's container, run `ip link set eth0 address aa:bb:cc:dd:ee:ff` to corrupt ARP    |
| L3    | Inside R2's container, change the route for 10.1.0.0/24 to point to 10.0.0.9              |
| L4    | Inside R2's container, run `iptables -A FORWARD -p icmp -j DROP` to block forwarded ICMP   |

### Success criteria

- PC1 can ping PC2.
- You have a written record with one paragraph per fault: the symptom you observed, the diagnostic command that confirmed your hypothesis, and the change you made to fix it.
- Your notes describe all four faults in L1-to-L4 order.
