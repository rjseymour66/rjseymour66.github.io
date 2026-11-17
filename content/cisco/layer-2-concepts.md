+++
title = 'Layer 2 concepts'
date = '2026-11-15T21:58:54-05:00'
weight = 40
draft = false
+++


## VLANs

**Virtual LANs (VLANs)**, pronounced "V-LANs," are like subnets: they divide a network into smaller parts. VLANs divide a LAN (a broadcast domain) into smaller LANs.

### Why you need VLANs

#### Layer 3 segmentation

In an ideal network, hosts in each subnet connect to their own switch. In practice, subnets usually share switches to reduce cost.

When traffic between departments must pass through the router, you can implement security policies on the router to control which traffic is permitted.

#### Layer 2 segmentation with VLANs

Switches aren't Layer 3 aware. From SW1's perspective, all hosts are still in the same LAN (broadcast domain). When any host connected to SW1 sends a broadcast frame or an unknown unicast frame, all other connected hosts receive it.

In this context, a **LAN** is a broadcast domain: the group of devices that receive a broadcast frame sent by any other member of the group.

VLANs divide a single physical switch into multiple virtual switches. This divides one broadcast domain into multiple broadcast domains.

As a general rule, map subnets to VLANs one-to-one: one subnet per VLAN.

### Configuring VLANs and access ports

#### Default VLANs

`show vlan brief` lists the VLANs that exist on the switch (the VLAN database) and the ports in each VLAN. Before you configure any VLANs, the output looks like this:

```text
SW1# show vlan brief
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/0, Gi0/1, Gi0/2, Gi0/3
                                                Gi1/0, Gi1/1, Gi1/2, Gi1/3
                                                Gi2/0, Gi2/1, Gi2/2, Gi2/3
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
```

**VLAN 1** is the **default VLAN**: the VLAN that all ports are in by default. The MAC address table confirms this. The leftmost column shows that the switch learns all MAC addresses in VLAN 1:

```text
SW1# show mac address-table 
          Mac Address Table
-------------------------------------------
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    5254.0000.04c5    DYNAMIC     Gi2/0
   1    5254.000e.a694    DYNAMIC     Gi2/1
   1    5254.000f.d41a    DYNAMIC     Gi1/0
   1    5254.0011.fbcf    DYNAMIC     Gi0/1
. . .
```

VLANs 1002–1005 also exist by default. They're reserved for FDDI and Token Ring, two legacy Data Link Layer technologies. Modern networks don't use FDDI or Token Ring, but even modern versions of Cisco IOS reserve these four VLANs for backward compatibility. You can't delete them or use them for Ethernet VLANs.

#### VLAN ranges

| VLAN      | Use                                                  |
| :-------- | :--------------------------------------------------- |
| 0         | Reserved for special purposes (beyond the CCNA exam) |
| 1–1001    | Usable (VLAN 1 is the default VLAN)                  |
| 1002–1005 | Reserved for FDDI and Token Ring                     |
| 1006–4094 | Usable                                               |
| 4095      | Reserved for special purposes (beyond the CCNA exam) |

The usable ranges total 4,090 VLANs. A single LAN (broadcast domain) can be divided into a maximum of 4,090 VLANs, far more than most LANs ever need.

#### Creating and naming VLANs

| Command          | Mode                 | Action           |
| :--------------- | :------------------- | :--------------- |
| `vlan vlan-id`   | Global configuration | Creates the VLAN |
| `name vlan-name` | VLAN configuration   | Names the VLAN   |

```text
SW1(config)# vlan 10
SW1(config-vlan)# name Engineering
SW1(config-vlan)# vlan 20
SW1(config-vlan)# name HR
SW1(config-vlan)# vlan 30
SW1(config-vlan)# name Sales
SW1(config-vlan)# end
SW1# show vlan brief
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/0, Gi0/1, Gi0/2, Gi0/3
                                                Gi1/0, Gi1/1, Gi1/2, Gi1/3
                                                Gi2/0, Gi2/1, Gi2/2, Gi2/3
10   Engineering                      active
20   HR                               active
30   Sales                            active
. . .
```

#### Disabling and deleting VLANs

To temporarily disable a VLAN, use `shutdown` in VLAN configuration mode. This example disables VLAN 10 on SW1 and confirms the `act/lshut` status with `show vlan brief`:

```text
SW1(config)# vlan 10
SW1(config-vlan)# shutdown
SW1(config-vlan)# end
SW1# show vlan brief
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
. . .
10   Engineering                      act/lshut
. . .
```

To delete a VLAN entirely, negate the command that created it with `no`: `no vlan 10`.

#### Assigning ports to VLANs

An **access port** is a switch port that belongs to a single VLAN. A **trunk port** carries traffic in multiple VLANs.

Cisco switch ports use **Dynamic Trunking Protocol (DTP)** to automatically determine whether each port operates in access mode or trunk mode. Best practice is to manually configure access or trunk mode instead of letting DTP decide.

To assign ports to a VLAN:

1. Configure the ports in access mode with `switchport mode access` in interface configuration mode.
2. Configure the port's VLAN with `switchport access vlan vlan-id`.

```text
SW1(config)# interface range g0/0-3
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 10
SW1(config-if-range)# interface range g1/0-3
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 20
SW1(config-if-range)# interface range g2/0-3
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 30
```

{{< admonition "VLANs are created automatically" tip >}}
If you assign a port to a VLAN that doesn't exist yet, `switchport access vlan` creates the VLAN automatically. You don't need to create VLANs with the `vlan` command before you assign ports to them.
{{< /admonition >}}

### Connecting switches with trunk ports

Trunk ports aren't assigned to a single VLAN. They forward traffic in multiple VLANs. For hosts in each VLAN to communicate with each other, the link between SW1 and SW2 must carry traffic in multiple VLANs. To enable that, the switches tag the frames they send to each other to indicate which VLAN each frame belongs to.

Another option is a separate access link between the switches for each VLAN (one for VLAN 10, one for VLAN 20, and one for VLAN 30). This works in small networks with few VLANs, but it doesn't scale to networks with many VLANs. A trunk link is the better option.

#### Tagged and untagged ports

The switch forwarding a frame adds a tag before it sends the frame out of a trunk port. Access ports are associated with only one VLAN, so they don't need a tag to identify the VLAN of the frames they send and receive.

|                       | Trunk port                         | Access port                          |
| :-------------------- | :--------------------------------- | :----------------------------------- |
| Other name            | **Tagged port**                    | **Untagged port**                    |
| VLANs                 | Multiple                           | One                                  |
| Frames forwarded      | Tagged                             | Not tagged                           |
| Frame VLAN set by     | The tag                            | The `switchport access vlan` command |
| Typically connects to | Other switches (sometimes routers) | End hosts, such as PCs               |

#### IEEE 802.1Q tag

**IEEE 802.1Q** is typically pronounced "dot one Q." The 802.1Q tag is 4 bytes and sits between the Source and EtherType fields of the Ethernet header.

<div class="bitfield-wrap">
<table class="merged bitfield">
  <thead>
    <tr><th>7 bytes</th><th>1 byte</th><th>6 bytes</th><th>6 bytes</th><th>4 bytes</th><th>2 bytes</th><th></th><th>4 bytes</th></tr>
  </thead>
  <tbody>
    <tr><td>Preamble</td><td>SFD</td><td>Destination</td><td>Source</td><td><strong>802.1Q Tag</strong></td><td>EtherType</td><td>Packet</td><td>FCS</td></tr>
  </tbody>
</table>
</div>

The 802.1Q tag contains these fields:

<div class="bitfield-wrap">
<table class="merged bitfield">
  <thead>
    <tr><th rowspan="2">TPID</th><th colspan="3">TCI</th></tr>
    <tr><th>PCP</th><th>DEI</th><th>VID</th></tr>
  </thead>
  <tbody>
    <tr><td>16 bits</td><td>3 bits</td><td>1 bit</td><td>12 bits</td></tr>
  </tbody>
</table>
</div>

| Field                                | Size    | Purpose                                                                                                                                         |
| :----------------------------------- | :------ | :---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Tag Protocol Identifier (TPID)**   | 16 bits | Always `0x8100`. It sits where the EtherType field normally is. When the switch sees `0x8100` here, it knows the frame is tagged with 802.1Q. |
| **Tag Control Information (TCI)**    | 16 bits | Second half of the tag. Contains the PCP, DEI, and VID subfields.                                                                               |
| **Priority Code Point (PCP)**        | 3 bits  | Marks frames as higher or lower priority. Used for Quality of Service (QoS).                                                                   |
| **Drop Eligible Indicator (DEI)**    | 1 bit   | Indicates frames that can be dropped if the network is congested. Also used for QoS.                                                           |
| **VLAN Identifier (VID)**            | 12 bits | Indicates which VLAN the frame is in. This is the most important field.                                                                         |

The VID is 12 bits, so there are 4,096 VLANs in total (2<sup>12</sup> = 4,096).

##### ISL

Before IEEE 802.1Q, Cisco developed **Inter-Switch Link (ISL)** to tag frames over trunk links.

|                  | 802.1Q          | ISL                                                 |
| :--------------- | :-------------- | :-------------------------------------------------- |
| Developed by     | IEEE            | Cisco (proprietary)                                 |
| Switch support   | Not limited     | Cisco switches only                                 |
| Status           | Current         | Deprecated, not supported on new Cisco switches     |

#### Configuring trunk ports

##### Setting the encapsulation

On a switch that supports both 802.1Q and ISL, `switchport mode trunk` alone is rejected:

```text
SW1(config)# interface g0/0
SW1(config-if)# switchport mode trunk
Command rejected: An interface whose trunk encapsulation 
is "Auto" can not be configured to "trunk" mode.
```

By default, ports on a switch that supports both 802.1Q and ISL use DTP. You can't manually configure trunk mode and still let DTP decide whether to use 802.1Q or ISL. To manually configure trunk mode, you must also manually configure the encapsulation protocol with `switchport trunk encapsulation {dot1q | isl}`:

```text
SW1(config-if)# switchport trunk encapsulation dot1q
SW1(config-if)# switchport mode trunk
SW1(config-if)#
```

If a switch supports only 802.1Q (not ISL), you don't need `switchport trunk encapsulation` before `switchport mode trunk`. The switch doesn't even support the command.

##### Verifying trunk ports

After you configure a port as a trunk, it no longer appears in `show vlan brief`:

```text
SW1# show vlan brief 
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi2/0, Gi2/1, Gi2/2, Gi2/3
10   Engineering                      active    Gi0/1, Gi1/0, Gi1/1
20   HR                               active    Gi0/2, Gi1/2
30   Sales                            active    Gi0/3, Gi1/3
```

To verify trunk ports, use `show interfaces trunk`:

```text
SW1# show interfaces trunk
Port        Mode             Encapsulation  Status        Native vlan
Gi0/0       on               802.1q         trunking      1
  
Port        Vlans allowed on trunk
Gi0/0       1-4094
  
Port        Vlans allowed and active in management domain
Gi0/0       1,10,20,30
. . .
```

The first section (the top two lines) lists each trunk port and basic information:

| Column        | Value      | Meaning                                                                       |
| :------------ | :--------- | :---------------------------------------------------------------------------- |
| Mode          | `on`       | G0/0 is manually configured as a trunk with `switchport mode trunk`           |
| Encapsulation | `802.1q`   | Set by `switchport trunk encapsulation dot1q`                                 |
| Status        | `trunking` | Expected, because G0/0 is manually configured in trunk mode                   |
| Native vlan   | `1`        | See [The native VLAN](#the-native-vlan)                                       |

The second section lists the VLANs allowed on each trunk port (`Vlans allowed on trunk`). The `1-4094` value shows that all VLANs are allowed on a trunk port by default. The port can forward and receive traffic in all VLANs.

The **management domain** in `Vlans allowed and active in management domain` refers to the VLAN Trunking Protocol (VTP) domain.

##### Modifying the list of allowed VLANs

All VLANs are allowed on a trunk port by default, but best practice is to allow only the necessary VLANs. Configure the list with `switchport trunk allowed vlan`:

```text
SW1(config-if)# switchport trunk allowed vlan
  WORD    VLAN IDs of the allowed VLANs when this port is 
    in trunking mode
  add     add VLANs to the current list
  all     all VLANs
  except  all VLANs except the following
  none    no VLANs
  remove  remove VLANs from the current list
```

| Keyword          | Effect                                                                             |
| :--------------- | :--------------------------------------------------------------------------------- |
| `WORD`           | Replaces the list with the VLANs you specify, such as `10,20,30`                   |
| `add`            | Adds VLANs to the current list                                                     |
| `remove`         | Removes VLANs from the current list                                                |
| `all`            | Allows all VLANs (the default)                                                     |
| `none`           | Allows no VLANs. The port can't forward or receive any traffic.                    |
| `except`         | Allows all VLANs except the VLANs you specify                                      |

###### WORD

`switchport trunk allowed vlan 10,20,30` allows only VLANs 10, 20, and 30 on the trunk:

```text
SW1(config-if)# switchport trunk allowed vlan 10,20,30
SW1(config-if)# do show interfaces trunk
. . .
Port        Vlans allowed on trunk
Gi0/0       10,20,30
. . .
```

###### add and remove

```text
SW1(config-if)# switchport trunk allowed vlan add 1
SW1(config-if)# switchport trunk allowed vlan remove 30
SW1(config-if)# do show interfaces trunk
. . .
Port        Vlans allowed on trunk
Gi0/0       1,10,20
. . .
```

{{< admonition "Don't forget the add keyword" warning >}}
Forgetting `add` is a common rookie mistake. If you want to add VLAN 40 but enter `switchport trunk allowed vlan 40`, you don't add VLAN 40 to the list. You replace the whole list with only VLAN 40.
{{< /admonition >}}

###### all and none

```text
SW1(config-if)# switchport trunk allowed vlan all
SW1(config-if)# do show interfaces trunk
. . .
Port        Vlans allowed on trunk
Gi0/0       1-4094
. . .
SW1(config-if)# switchport trunk allowed vlan none
SW1(config-if)# do show interfaces trunk
. . .
Port        Vlans allowed on trunk
Gi0/0       none
. . .
```

###### except

```text
SW1(config-if)# switchport trunk allowed vlan except
 1-9,11-19,21-29,31-4094
SW1(config-if)# do show interfaces trunk
. . .
Port        Vlans allowed on trunk
Gi0/0       10,20,30
. . .
```

##### The native VLAN

The **native VLAN** is the VLAN that a trunk port assigns untagged received traffic to. A trunk port also forwards traffic in the native VLAN without a tag. By default, the native VLAN is VLAN 1:

```text
SW1# show interfaces trunk
Port        Mode             Encapsulation  Status        Native vlan
Gi0/0       on               802.1q         trunking      1
. . .
```

###### Default VLAN vs. native VLAN

People often confuse the default VLAN and the native VLAN:

|                  | Default VLAN                                    | Native VLAN                                                                                   |
| :--------------- | :---------------------------------------------- | :-------------------------------------------------------------------------------------------- |
| Applies to       | Access ports                                    | Trunk ports                                                                                   |
| What it is       | The VLAN access ports are assigned to by default | The VLAN that untagged frames received on the trunk are assigned to. Frames in this VLAN are forwarded untagged. |
| Default value    | VLAN 1                                          | VLAN 1                                                                                        |
| Can you change it | No                                             | Yes, per port                                                                                 |

###### Configuring the native VLAN

To configure the native VLAN of a trunk port, use `switchport trunk native vlan vlan-id`:

```text
SW1(config-if)# switchport trunk native vlan 30
SW1(config-if)# show interfaces trunk
Port        Mode             Encapsulation  Status        Native vlan
Gi0/0       on               802.1q         trunking      30
. . .
```

You configure the native VLAN per port. A switch with multiple trunk ports can have a different native VLAN on each port.

##### Native VLAN mismatch

You can configure a different native VLAN on each end of a link, but it's a misconfiguration. Make sure the native VLAN matches on both ends of the link.

In this example, SW1 G0/0's native VLAN is 10 and SW2 G0/0's native VLAN is 30:

1. PC1 sends a frame to PC10.
2. SW1 forwards the frame untagged to SW2.
3. SW2 receives the untagged frame and assigns it to VLAN 30, SW2 G0/0's native VLAN.
4. The destination connects to SW2 G0/1, an access port in VLAN 10. SW2 can't forward the frame to its proper destination.

When traffic crosses from one VLAN to another like this, it's called **VLAN hopping**.

##### Disabling the native VLAN

The native VLAN was developed to accommodate devices that don't support 802.1Q tagging, such as hubs. Using it can make the network vulnerable to security exploits, so best practice is to disable the native VLAN on trunk ports.

You can't actually disable the native VLAN. Instead, configure an unused VLAN (not the default, VLAN 1) as the native VLAN. This is equivalent to disabling it. For example, configure `switchport trunk native vlan 999` on the G0/0 ports of SW1 and SW2 to make VLAN 999, an unused VLAN, the native VLAN.

{{< admonition "Remember" tip >}}
As a security best practice, configure an unused VLAN (not the default, VLAN 1) as the native VLAN on your trunk ports.
{{< /admonition >}}

### Inter-VLAN routing

**Inter-VLAN routing** is routing between subnets in a LAN that's segmented with VLANs.

#### One router interface per VLAN

```text
R1(config)# interface g0/0
R1(config-if)# ip address 172.16.1.1 255.255.255.192
R1(config-if)# no shutdown
R1(config-if)# interface g0/1
R1(config-if)# ip address 172.16.1.65 255.255.255.192
R1(config-if)# no shutdown
R1(config-if)# interface g0/2
R1(config-if)# ip address 172.16.1.129 255.255.255.192
R1(config-if)# no shutdown
 
SW1(config)# interface g0/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# interface g0/2
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# interface g0/3
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 30
```

#### Router on a stick

**Router on a stick (ROAS)** is an inter-VLAN routing method that uses a trunk link between a switch and a router. You divide a single physical router interface into multiple virtual **subinterfaces**, each with its own IP address. Subinterfaces send and receive tagged frames, like a trunk port on a switch.

A router's physical interface and its subinterfaces all share the same MAC address. When a frame arrives on the physical interface, the router uses the frame's VLAN tag, not its destination MAC address, to decide which subinterface the frame is for.

##### Configuring ROAS

###### Switch side

Configure SW1 G0/1 as a trunk port, allow only the necessary VLANs, and change the native VLAN to an unused VLAN:

```text
SW1(config)# interface g0/1
SW1(config-if)# switchport trunk encapsulation dot1q
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk allowed vlan 10,20,30
SW1(config-if)# switchport trunk native vlan 999
```

###### Router side

1. Enable the physical interface with `no shutdown`.
2. Enter subinterface configuration mode with `interface`, followed by the interface name, a period, and a number that identifies the subinterface, such as `interface g0/0.10`.
3. Associate the subinterface with a VLAN with `encapsulation dot1q vlan-id`.
4. Configure the IP address.

```text
R1(config)# interface g0/0
R1(config-if)# no shutdown
R1(config-if)# interface g0/0.10
R1(config-subif)#
```

```text
R1(config-subif)# encapsulation dot1q 10
R1(config-subif)# ip address 172.16.1.1 255.255.255.192
```

{{< admonition "Subinterface number vs. VLAN ID" note >}}
The subinterface number (the `.10` in G0/0.10) doesn't have to match the VLAN ID. It only identifies the subinterface. `encapsulation dot1q` tells the router which VLAN to associate with the subinterface. Still, match the two numbers. There's no reason not to.
{{< /admonition >}}

Configure two more subinterfaces, one for VLAN 20 and one for VLAN 30, then confirm with `show ip interface brief`. The G0/0 interface itself has no IP address. The three subinterfaces have IP addresses, and they send and receive traffic through the physical G0/0 interface:

```text
R1(config-subif)# interface g0/0.20
R1(config-subif)# encapsulation dot1q 20
R1(config-subif)# ip address 172.16.1.65 255.255.255.192
R1(config-subif)# interface g0/0.30
R1(config-subif)# encapsulation dot1q 30
R1(config-subif)# ip address 172.16.1.129 255.255.255.192
R1(config-subif)# do show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         unassigned      YES manual up                    up
GigabitEthernet0/0.10      172.16.1.1      YES manual up                    up
GigabitEthernet0/0.20      172.16.1.65     YES manual up                    up
GigabitEthernet0/0.30      172.16.1.129    YES manual up                    up
. . .
```

##### Configuring the native VLAN with ROAS

You can configure the native VLAN on the router's side of the connection in two ways:

| Method                    | Configuration                                                                                       |
| :------------------------ | :-------------------------------------------------------------------------------------------------- |
| `native` keyword          | Use `encapsulation dot1q vlan-id native` on the appropriate subinterface                            |
| Physical interface        | Configure the native VLAN's IP address on the physical interface, not a subinterface                |

###### Using the native keyword

This example configures VLAN 10 as the native VLAN by adding `native` to `encapsulation dot1q`:

```text
R1(config)# interface g0/0
R1(config-if)# no shutdown
R1(config-if)# interface g0/0.10
R1(config-subif)# encapsulation dot1q 10 native
R1(config-subif)# ip address 172.16.1.1 255.255.255.192
R1(config-subif)# interface g0/0.20
R1(config-subif)# encapsulation dot1q 20
R1(config-subif)# ip address 172.16.1.65 255.255.255.192
R1(config-subif)# interface g0/0.30
R1(config-subif)# encapsulation dot1q 30
R1(config-subif)# ip address 172.16.1.129 255.255.255.192
```

###### Using the physical interface

This example doesn't configure a subinterface for VLAN 10. It configures the native VLAN's IP address on G0/0 itself. VLAN 10 doesn't need `encapsulation dot1q`, but the subinterfaces of the non-native VLANs (VLANs 20 and 30) still need it:

```text
R1(config)# interface g0/0
R1(config-if)# no shutdown
R1(config-if)# ip address 172.16.1.1 255.255.255.192
R1(config-if)# interface g0/0.20
R1(config-subif)# encapsulation dot1q 20
R1(config-subif)# ip address 172.16.1.65 255.255.255.192
R1(config-subif)# interface g0/0.30
R1(config-subif)# encapsulation dot1q 30
R1(config-subif)# ip address 172.16.1.129 255.255.255.192
```

#### Multilayer switching

A **multilayer switch** (also called a **Layer 3 switch**) is a switch that can also route packets: a switch with a built-in router. It's the third option for inter-VLAN routing and perhaps the most popular, although ROAS is common as well.

##### Inter-VLAN routing with SVIs

Multilayer switches route between VLANs with virtual interfaces called **switch virtual interfaces (SVIs)**. Each SVI is an interface on the switch's built-in router. Hosts in each VLAN use their VLAN's SVI IP address as their default gateway.

To configure inter-VLAN routing with SVIs:

1. Enable IP routing with `ip routing`.
2. Create each SVI with `interface vlan vlan-id`.
3. Configure an IP address on the SVI, like on a router interface.

Unlike a router subinterface number, the `vlan-id` in `interface vlan` is significant. It specifies which VLAN the SVI is associated with.

```text
SW1(config)# ip routing
SW1(config)# interface vlan 10
SW1(config-if)# ip address 172.16.1.1 255.255.255.192
SW1(config-if)# interface vlan 20
SW1(config-if)# ip address 172.16.1.65 255.255.255.192
SW1(config-if)# interface vlan 30
SW1(config-if)# ip address 172.16.1.129 255.255.255.192
```

On some switches, SVIs are administratively disabled by default. In that case, use `no shutdown` to enable each SVI.

SW1 can now route packets in the LAN. Like a router, SW1 inserts connected and local routes into its routing table for each SVI, so you don't need static routes:

```text
SW1# show ip route
. . .
      172.16.0.0/16 is variably subnetted, 6 subnets, 2 masks
C        172.16.1.0/26 is directly connected, Vlan10
L        172.16.1.1/32 is directly connected, Vlan10
C        172.16.1.64/26 is directly connected, Vlan20
L        172.16.1.65/32 is directly connected, Vlan20
C        172.16.1.128/26 is directly connected, Vlan30
L        172.16.1.129/32 is directly connected, Vlan30
```

##### SVI up/up requirements

Like a physical interface, an SVI must be up/up to function (the Status and Protocol columns in `show ip interface brief`):

```text
SW1# show ip interface brief | include Vlan 
Vlan10                 172.16.1.1      YES manual up                    up
Vlan20                 172.16.1.65     YES manual up                    up
Vlan30                 172.16.1.129    YES manual up                    up
```

Use this list to troubleshoot an SVI that won't reach up/up:

1. The VLAN associated with the SVI must exist on the switch (created with `vlan vlan-id`).
2. The switch must have at least one of the following in an up/up state:
   - An access port associated with the VLAN (with `switchport access vlan`)
   - A trunk port that allows the VLAN (with `switchport trunk allowed vlan`)
3. The VLAN must be enabled (no `shutdown` applied).
4. The SVI must be enabled (no `shutdown` applied).

These commands break the requirements, and all three SVIs go up/down:

```text
SW1(config)# no vlan 10
SW1(config)# interface g1/3
SW1(config-if)# shutdown
SW1(config-if)# interface g0/0
SW1(config-if)# switchport trunk allowed vlan remove 30
SW1(config-if)# vlan 20
SW1(config-vlan)# shutdown
SW1(config-vlan)# do show ip interface brief | include Vlan
Vlan10                 172.16.1.1      YES manual up                    down
Vlan20                 172.16.1.65     YES manual up                    down
Vlan30                 172.16.1.129    YES manual up                    down
```

| SVI    | Commands                                                                                      | Requirement broken                                                        |
| :----- | :-------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------ |
| Vlan10 | `no vlan 10`                                                                                  | 1: The VLAN no longer exists on the switch                                |
| Vlan20 | `vlan 20`, then `shutdown`                                                                    | 3: The VLAN is disabled                                                   |
| Vlan30 | `shutdown` on G1/3, then `switchport trunk allowed vlan remove 30` on G0/0                    | 2: No up/up access port in VLAN 30 and no up/up trunk port that allows it |

##### Using routed ports for external connectivity

Hosts in the LAN must also reach external networks, such as the internet or another LAN in the corporate network. A common way to provide external connectivity is a routed port on a multilayer switch. A **routed port** is a physical port on a multilayer switch configured to function like a router interface.

To configure a routed port:

1. Enter interface configuration mode.
2. Run `no switchport`.
3. Configure an IP address, like on a router interface.

```text
SW1(config)# interface g0/1
SW1(config-if)# no switchport
SW1(config-if)# ip address 172.16.1.193 255.255.255.252
SW1(config-if)# do show ip route
. . .
      172.16.0.0/16 is variably subnetted, 8 subnets, 3 masks
C        172.16.1.0/26 is directly connected, Vlan10
L        172.16.1.1/32 is directly connected, Vlan10
C        172.16.1.64/26 is directly connected, Vlan20
L        172.16.1.65/32 is directly connected, Vlan20
C        172.16.1.128/26 is directly connected, Vlan30
L        172.16.1.129/32 is directly connected, Vlan30
C        172.16.1.192/30 is directly connected, GigabitEthernet0/1
L        172.16.1.193/32 is directly connected, GigabitEthernet0/1
```

SW1 G0/1 is now a routed port with an IP address, and SW1 added connected and local routes for it. SW1 still can't forward packets outside the LAN. It needs a route (or routes) to external destinations. Like on a router, you can configure static routes on a multilayer switch or use a dynamic routing protocol.

This example configures a static default route on SW1, with R1's IP address as the next hop:

```text
SW1(config)# ip route 0.0.0.0 0.0.0.0 172.16.1.194
```
