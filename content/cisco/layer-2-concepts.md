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

| Field                              | Size    | Purpose                                                                                                                                       |
| :--------------------------------- | :------ | :-------------------------------------------------------------------------------------------------------------------------------------------- |
| **Tag Protocol Identifier (TPID)** | 16 bits | Always `0x8100`. It sits where the EtherType field normally is. When the switch sees `0x8100` here, it knows the frame is tagged with 802.1Q. |
| **Tag Control Information (TCI)**  | 16 bits | Second half of the tag. Contains the PCP, DEI, and VID subfields.                                                                             |
| **Priority Code Point (PCP)**      | 3 bits  | Marks frames as higher or lower priority. Used for Quality of Service (QoS).                                                                  |
| **Drop Eligible Indicator (DEI)**  | 1 bit   | Indicates frames that can be dropped if the network is congested. Also used for QoS.                                                          |
| **VLAN Identifier (VID)**          | 12 bits | Indicates which VLAN the frame is in. This is the most important field.                                                                       |

The VID is 12 bits, so there are 4,096 VLANs in total (2<sup>12</sup> = 4,096).

##### ISL

Before IEEE 802.1Q, Cisco developed **Inter-Switch Link (ISL)** to tag frames over trunk links.

|                | 802.1Q      | ISL                                             |
| :------------- | :---------- | :---------------------------------------------- |
| Developed by   | IEEE        | Cisco (proprietary)                             |
| Switch support | Not limited | Cisco switches only                             |
| Status         | Current     | Deprecated, not supported on new Cisco switches |

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

| Column        | Value      | Meaning                                                             |
| :------------ | :--------- | :------------------------------------------------------------------ |
| Mode          | `on`       | G0/0 is manually configured as a trunk with `switchport mode trunk` |
| Encapsulation | `802.1q`   | Set by `switchport trunk encapsulation dot1q`                       |
| Status        | `trunking` | Expected, because G0/0 is manually configured in trunk mode         |
| Native vlan   | `1`        | See [The native VLAN](#the-native-vlan)                             |

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

| Keyword  | Effect                                                           |
| :------- | :--------------------------------------------------------------- |
| `WORD`   | Replaces the list with the VLANs you specify, such as `10,20,30` |
| `add`    | Adds VLANs to the current list                                   |
| `remove` | Removes VLANs from the current list                              |
| `all`    | Allows all VLANs (the default)                                   |
| `none`   | Allows no VLANs. The port can't forward or receive any traffic.  |
| `except` | Allows all VLANs except the VLANs you specify                    |

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

|                   | Default VLAN                                     | Native VLAN                                                                                                      |
| :---------------- | :----------------------------------------------- | :--------------------------------------------------------------------------------------------------------------- |
| Applies to        | Access ports                                     | Trunk ports                                                                                                      |
| What it is        | The VLAN access ports are assigned to by default | The VLAN that untagged frames received on the trunk are assigned to. Frames in this VLAN are forwarded untagged. |
| Default value     | VLAN 1                                           | VLAN 1                                                                                                           |
| Can you change it | No                                               | Yes, per port                                                                                                    |

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

| Method             | Configuration                                                                        |
| :----------------- | :----------------------------------------------------------------------------------- |
| `native` keyword   | Use `encapsulation dot1q vlan-id native` on the appropriate subinterface             |
| Physical interface | Configure the native VLAN's IP address on the physical interface, not a subinterface |

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

| SVI    | Commands                                                                   | Requirement broken                                                        |
| :----- | :------------------------------------------------------------------------- | :------------------------------------------------------------------------ |
| Vlan10 | `no vlan 10`                                                               | 1: The VLAN no longer exists on the switch                                |
| Vlan20 | `vlan 20`, then `shutdown`                                                 | 3: The VLAN is disabled                                                   |
| Vlan30 | `shutdown` on G1/3, then `switchport trunk allowed vlan remove 30` on G0/0 | 2: No up/up access port in VLAN 30 and no up/up trunk port that allows it |

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

## Dynamic Trunking Protocol and VLAN Trunking Protocol

**Dynamic Trunking Protocol (DTP)** is a Cisco-proprietary protocol that lets Cisco switches automatically determine the operational mode of their ports. Neighboring switches send each other DTP messages that advertise their port's administrative mode.

### Administrative and operational modes

| Mode           | Definition                                                         |
| :------------- | :----------------------------------------------------------------- |
| Administrative | The port's configured mode, set with the `switchport mode` command |
| Operational    | The mode the port operates in: access or trunk                     |

If you manually configure a port as an access port or trunk port, its administrative and operational modes are identical:

| Administrative mode      | Operational mode   |
| :----------------------- | :----------------- |
| `switchport mode access` | Always access port |
| `switchport mode trunk`  | Always trunk port  |

### Why you should disable DTP

DTP was developed to streamline switch deployment by requiring less manual configuration of port modes. DTP turned out to be a security vulnerability, so best practice today is generally to disable DTP on switch ports.

### DTP negotiation

The `switchport mode` command has two more options: `dynamic auto` and `dynamic desirable`. These administrative modes don't specify which mode the port operates in. They tell the switch to use DTP to determine the port's operational mode. By default, a port in either mode operates as an access port. If the connected switches both agree, they form a trunk link.

**DTP negotiation** is the process of exchanging DTP messages and agreeing on an operational mode.

#### Viewing administrative and operational modes

To view a port's administrative and operational modes, run `show interfaces interface-name switchport`.

#### dynamic auto

`dynamic auto` is the default administrative mode for all Cisco switch ports. A port in this mode uses DTP to negotiate its operational mode, but it doesn't actively try to form a trunk link with its neighbor. By default, two connected Cisco switches don't form a trunk. The connection stays in access mode:

```text
SW1# show interfaces g0/0 switchport
Name: Gig0/0
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: static access
. . .
```

#### dynamic desirable

A port in `dynamic desirable` mode actively attempts to form a trunk. After you change the administrative mode of SW1 G0/0 to `dynamic desirable`, its operational mode is trunk:

```text
SW1# show interfaces g0/0 switchport
Name: Gig0/0
Switchport: Enabled
Administrative Mode: dynamic desirable
Operational Mode: trunk
. . .
```

### Switch port administrative modes

| Mode                | Description                                                                                                                                                                                           | Port sends DTP messages? |
| :------------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------- |
| `access`            | Manually configures an access port. Operational mode is always access.                                                                                                                                | No                       |
| `trunk`             | Manually configures a trunk port. Operational mode is always trunk.                                                                                                                                   | Yes                      |
| `dynamic auto`      | Uses DTP to negotiate its operational mode but doesn't actively try to form a trunk with its neighbor.<br><br>Forms a trunk if connected to a port in `trunk` or `dynamic desirable` mode.            | Yes                      |
| `dynamic desirable` | Uses DTP to negotiate its operational mode and actively tries to form a trunk with its neighbor.<br><br>Forms a trunk if connected to a port in `trunk`, `dynamic auto`, or `dynamic desirable` mode. | Yes                      |

#### Operational mode for each combination

Know the resulting operational mode of each combination of administrative modes:

| Administrative modes  | `access` | `trunk` | `dynamic desirable` | `dynamic auto` |
| :-------------------- | :------- | :------ | :------------------ | :------------- |
| **Access**            | access   | invalid | access              | access         |
| **Trunk**             | invalid  | trunk   | trunk               | trunk          |
| **Dynamic desirable** | access   | trunk   | trunk               | trunk          |
| **Dynamic auto**      | access   | trunk   | trunk               | access         |


Access + trunk isn't a valid combination. One of two things happens:

- The switches detect the mismatch and block the link.
- Traffic on the link is limited to the trunk port's native VLAN and the access port's VLAN, because both are untagged.

#### Trunk encapsulation negotiation

If a switch supports both 802.1Q and ISL, its default setting is `switchport trunk encapsulation negotiate`:

| Switch A    | Switch B         | Result                                                |
| :---------- | :--------------- | :---------------------------------------------------- |
| `negotiate` | `negotiate`      | ISL                                                   |
| `negotiate` | `dot1q` or `isl` | The `negotiate` side agrees to the same encapsulation |
| `dot1q`     | `isl`            | Encapsulation mismatch: a misconfiguration            |

If you encounter a switch that supports both 802.1Q and ISL, manually configure 802.1Q with `switchport trunk encapsulation dot1q`.

### Disabling DTP

DTP was developed to streamline switch deployment by letting switches automatically determine the operational mode of their ports.

Disable DTP for two reasons:

- Its security implications
- To reduce unnecessary traffic (DTP messages) in the LAN

You can disable DTP in two ways:

| Method                                        | Command                  |
| :-------------------------------------------- | :----------------------- |
| Manually configure the port as an access port | `switchport mode access` |
| Explicitly disable DTP                        | `switchport nonegotiate` |

To verify the first method:

1. Run `show interfaces g0/0 switchport`. The output shows `Negotiation of Trunking: On`, which means the port is sending DTP messages.
2. Configure G0/0 as an access port.
3. Check again. Trunk negotiation is now `Off`.

```text
SW1# show interfaces g0/0 switchport
Name: Gi0/0
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: static access
. . .
Negotiation of Trunking: On
. . .
SW1# configure terminal
SW1(config)# interface g0/0
SW1(config-if)# switchport mode access
SW1(config-if)# do show interfaces g0/0 switchport
Name: Gi0/0
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
. . .
Negotiation of Trunking: Off
. . .
```

Use the second method, `switchport nonegotiate`, to ensure that the port never sends DTP messages, regardless of its current mode (access or trunk). In this example, G0/0 is a trunk with negotiation `On` until you run `switchport nonegotiate`:

```text
SW1(config)# interface g0/0
SW1(config-if)# switchport mode trunk
SW1(config-if)# do show interfaces g0/0 switchport
Name: Gi0/0
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
. . .
Negotiation of Trunking: On
. . .
SW1(config-if)# switchport nonegotiate
SW1(config-if)# do show interfaces g0/0 switchport
Name: Gi0/0
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
. . .
Negotiation of Trunking: Off
. . .
```

{{< admonition "Best practice" tip >}}
Even if you manually configure a port in access mode, also run <code>switchport nonegotiate</code> to ensure the port never sends DTP messages.
{{< /admonition >}}

### VLAN Trunking Protocol

**VLAN Trunking Protocol (VTP)** is another Cisco-proprietary protocol that plays a role in VLAN configuration on Cisco switches. VTP lets switches automatically synchronize their **VLAN database**, the file that stores information about the VLANs that exist on the switch.

With VTP, switches in a LAN send each other messages about the VLANs in their VLAN database. Each switch synchronizes its database to the latest version.

| Fact               | Detail                                                                                                                |
| :----------------- | :-------------------------------------------------------------------------------------------------------------------- |
| VLAN database file | `vlan.dat` in flash memory                                                                                            |
| View flash memory  | `dir flash:` or `show flash:`                                                                                         |
| Benefit            | Saves time and reduces human error by propagating VLAN changes without manually configuring every switch              |
| Vendor support     | Cisco-proprietary like DTP. Runs only on Cisco switches, so it can't synchronize VLANs with another vendor's switches |

#### VTP synchronization

This causes SW1 to increment the **VTP revision number**, a number that tracks the latest version of the VLAN database. The revision number starts at 0. The switch updates it each time the VLAN database changes, such as when a VLAN is created, deleted, or renamed.

The **VTP domain** is the group of switches in a LAN that share the same VTP domain name. By default, switches have no VTP domain name, and VTP isn't active:

- You can configure VLANs on the switch, but it doesn't send VTP messages to other switches in the LAN.
- A switch without a VTP domain name is in **domain NULL**.

`show vtp status` shows an empty VTP domain name and a configuration revision of 0:

```text
SW1# show vtp status
VTP Version capable             : 1 to 3
VTP version running             : 1
VTP Domain Name                 :
VTP Pruning Mode                : Disabled
VTP Traps Generation            : Disabled
Device ID                       : 5254.0008.8000
Configuration last modified by 0.0.0.0 at 4-25-23 03:25:46
Local updater ID is 0.0.0.0 (no valid interface found)
 
Feature VLAN:
--------------
VTP Operating Mode                : Server
Maximum VLANs supported locally   : 1005
Number of existing VLANs          : 5
Configuration Revision            : 0
MD5 digest                        : 0x57 0xCD 0x40 0x65 0x63 0x59 0x47 0xBD 
                                    0x56 0x9D 0x4A 0x3E 0xA5 0x69 0x35 0xBC
```

To configure a VTP domain name, run `vtp domain domain-name`. The switch then sends VTP messages to other switches, and all switches without a VTP domain name adopt the new name.

In this example, SW1 joins domain Manning and creates VLANs 2, 3, and 4. SW2 and SW3 adopt the domain name and synchronize to the same VLAN count and revision number:

```text
SW1(config)# vtp domain Manning
Changing VTP domain name from NULL to Manning
SW1(config)# vlan 2
SW1(config-vlan)# vlan 3
SW1(config-vlan)# vlan 4
SW1(config-vlan)# end
SW1# show vtp status
. . .
VTP Domain Name                 : Manning
. . .
Number of existing VLANs          : 8
Configuration Revision            : 3
SW2# show vtp status
. . .
VTP Domain Name                 : Manning
. . .
Number of existing VLANs          : 8
Configuration Revision            : 3
SW3# show vtp status
. . .
VTP Domain Name                 : Manning
. . .
Number of existing VLANs          : 8
Configuration Revision            : 3
```

#### VTP modes

Configure the VTP mode with `vtp mode mode`. Switches in server and client mode actively participate in VTP and synchronize their VLAN databases to match each other. Switches in transparent and off mode don't.

| Mode        | Create, modify, and delete VLANs | Advertise changes | Synchronize database | Description                                                                                                                                                                                                     |
| :---------- | :------------------------------- | :---------------- | :------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Server      | Yes                              | Yes               | Yes                  | The default mode. Advertises changes to its VLAN database and synchronizes its VLAN database when it receives an advertisement with a higher revision number.                                                   |
| Client      | No                               | Yes               | Yes                  | Otherwise behaves the same as a server.                                                                                                                                                                         |
| Transparent | Yes                              | No                | No                   | Doesn't advertise changes to its own VLAN database or synchronize its VLAN database with others. Doesn't directly participate in the VTP domain, but forwards VTP messages between switches in the same domain. |
| Off         | Yes                              | No                | No                   | Doesn't advertise changes to its own VLAN database or synchronize its VLAN database with others. Doesn't participate in VTP at all.                                                                             |

Cisco recommends server mode for all switches in the domain if they have sufficient memory resources. Any modern switch has sufficient resources to store VLAN information, so you can safely leave all switches in VTP server mode.

If the LAN doesn't use VTP, run `vtp mode off` to disable VTP on all switches.

#### VTP versions

VTP has three versions, and version 1 is the default:

```text
SW1# show vtp status
VTP Version capable             : 1 to 3
VTP version running             : 1
. . .
```

| Fact               | Detail                                                                         |
| :----------------- | :----------------------------------------------------------------------------- |
| Configure version  | `vtp version version`                                                          |
| Preferred version  | Version 3 improves on versions 1 and 2. Always prefer it if you use VTP.       |
| `vtp primary` mode | Privileged EXEC mode, although most VTP commands are global configuration mode |

In this example, you enable VTP version 3 on SW1 and try to create VLAN 6:

1. Run `vtp version 3`.
2. Run `vlan 6`. The command fails because SW1 isn't the primary server for the VLAN database.
3. Run `vtp primary` to make SW1 the primary server.
4. Run `vlan 6` again. SW1 creates VLAN 6.

```text
SW1(config)# vtp version 3
SW1(config)# vlan 6
VTP VLAN configuration not allowed when device is not 
the primary server for vlan database.
SW1(config)# do vtp primary
This system is becoming primary server for feature vlan
No conflicting VTP3 devices found.
Do you want to continue? [confirm]
SW1(config)# vlan 6
SW1(config-vlan)# exit
```

A VTP domain has only one primary server. If you run `vtp primary` on another switch in the domain, that switch becomes the primary server and the first one reverts to being a secondary server.

##### Extended range VLANs

| Range          | VLANs     | Availability                                                                                                                                                            |
| :------------- | :-------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Normal range   | 1–1005    | Available in old versions of Cisco IOS                                                                                                                                  |
| Extended range | 1006–4094 | Reserved in old IOS versions for internal use by applications on the switch. You couldn't create them or assign ports to them. A later IOS version made them available. |

Today, all Cisco switches support both normal-range and extended-range VLANs. VTP version 3 added the ability to create and propagate extended-range VLANs.

#### Is VTP dangerous?

The danger of VTP is that a newly connected switch can overwrite the VLAN database of all switches in the domain.

{{< admonition "Remember" warning >}}
One recommended procedure is to reset a switch's VTP revision number to 0 before you connect it to the network. Use one of the three methods in the following table.
{{< /admonition >}}

| Method                                                            | Versions      |
| :---------------------------------------------------------------- | :------------ |
| Change the VTP domain name to a different one, then back again    | Not specified |
| Change the VTP mode to transparent, then back to server or client | 1 and 2 only  |
| Change the VTP mode to off, then back to server or client         | 1 and 2 only  |

The primary server mechanism in version 3 eliminates this risk. Switches synchronize only to the primary server. Even if you connect a new switch with a higher revision number, switches in the LAN don't synchronize to it. With version 3, you don't need to fear VTP, and it can be a great tool for automating some network configuration workflows.


## Spanning Tree Protocol

**Spanning Tree Protocol (STP)** runs on all Cisco switches by default. It solves a significant problem in LANs: Layer 2 loops, where frames loop around the network indefinitely.

### The need for STP

| Loop type                   | Prevention                                                                                                                                                                                                    |
| :-------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Layer 3 loop (routing loop) | Each router decrements the Time-to-Live (TTL) field when it forwards a packet. When the TTL reaches 0, the router drops the packet. This prevents packets from looping indefinitely after a misconfiguration. |
| Layer 2 loop                | The Ethernet header has no TTL field. If a loop occurs between switches, no mechanism prevents frames from looping around the LAN indefinitely.                                                               |

Layer 2 loops are inevitable in a LAN with multiple paths between any two nodes, because switches flood **BUM traffic** (broadcast, unknown unicast, and multicast) frames.

Multiple paths between hosts are an example of redundancy, which is desirable in a network. **Redundancy** means having additional network devices and connections beyond the minimum necessary for communication.

Layer 2 loops cause two main problems:

| Problem                  | Description                                                                                                                        |
| :----------------------- | :--------------------------------------------------------------------------------------------------------------------------------- |
| **Broadcast storm**      | Enough looping frames accumulate to consume so many network resources (device CPU or link bandwidth) that the network is unusable. |
| **MAC address flapping** | A switch learns the same MAC address repeatedly on separate ports.                                                                 |

In the MAC address flapping example, SW1 constantly updates the MAC address table entry for PC1's MAC address between multiple ports. SW1 doesn't know which port PC1 is actually connected to, so PC1 can't receive frames.

### How STP works

STP prevents Layer 2 loops by blocking redundant connections. It leaves only a single active path between any two nodes in a LAN.

In the example topology, SW3's G0/1 port is now in the blocking state:

| Port state     | Behavior                                                                                    |
| :------------- | :------------------------------------------------------------------------------------------ |
| **Blocking**   | Doesn't forward frames and doesn't process received frames, except for STP-related messages |
| **Forwarding** | Forwards and receives frames as normal. All other ports are in this state.                  |

The SW2 G0/1 to SW3 G0/1 link is unused. Network traffic doesn't pass over disabled links, but they're available to take over if one of the active links fails.

A **spanning tree** is a concept from graph theory:

| Term     | Meaning                                                                   |
| :------- | :------------------------------------------------------------------------ |
| Graph    | A structure that models relationships between objects (also called nodes) |
| Tree     | A subgraph in which any two nodes are connected by exactly one path       |
| Spanning | The tree includes all nodes. It spans across all nodes.                   |

### STP algorithm

The **STP algorithm** is the process STP uses to create a loop-free topology. It has three main steps:

1. Root bridge election
2. Root port selection
3. Designated port selection

{{< admonition "Bridge vs. switch" note >}}
In the context of STP, bridge and switch are synonymous.
{{< /admonition >}}

#### Root bridge election

The **root bridge** is the central point of reference for the STP topology. In later steps, all other switches ensure that they have exactly one active path to the root bridge.

Switches elect the root bridge by sharing STP **Bridge Protocol Data Unit (BPDU)** messages with each other. Switches send BPDUs every 2 seconds. BPDUs contain various STP-related information. Two pieces matter for the root bridge election:

- The switch's own **bridge identifier (BID)**, a number that uniquely identifies the switch in the LAN
- The BID of the switch it believes is the root bridge

##### The BID

The switch that sends the **superior BPDU** becomes the root bridge of the LAN. The superior BPDU has superior parameters according to the STP algorithm. That means the BPDU with the numerically lowest My BID field.

The BID is a 64-bit number:

<div class="bitfield-wrap">
<table class="merged bitfield">
  <thead>
    <tr><th colspan="5">Bridge priority (16 bits)</th><th rowspan="2">MAC address (48 bits)</th></tr>
    <tr><th colspan="4">Configurable priority value</th><th>Extended System ID</th></tr>
  </thead>
  <tbody>
    <tr><td>32768</td><td>16384</td><td>8192</td><td>4096</td><td>VLAN ID</td><td>Identifies the switch as a whole</td></tr>
  </tbody>
</table>
</div>

| Part                        | Description                                                                                                          |
| :-------------------------- | :------------------------------------------------------------------------------------------------------------------- |
| Configurable priority value | First part of the bridge priority. By default, the most significant bit is set to 1, which is equivalent to 0d32768. |
| **Extended System ID**      | Second part of the bridge priority. Equal to the VLAN ID.                                                            |
| MAC address                 | 48 bits                                                                                                              |

Cisco switches run a proprietary version of STP called **Per-VLAN Spanning Tree Plus (PVST+)**. In PVST+, switches run a separate STP instance for each VLAN, so they create a separate spanning tree for each VLAN. The benefit is that different links can be disabled in different VLANs, which balances traffic over all links.

##### Why include the VLAN ID in the bridge priority?

The STP standard (IEEE 802.1D) specifies that each switch must have a unique BID. Combining the bridge priority with the switch's MAC address achieves this. Even if all switches in the LAN have the same bridge priority, MAC addresses are unique, so each switch gets a unique BID.

For example, a switch runs two STP instances (VLAN 1 and VLAN 2). It has the default priority value of 32768 and MAC address 5254.000f.adab:

| VLAN | Bridge priority   | BID                  |
| :--- | :---------------- | :------------------- |
| 1    | 32768 + 1 = 32769 | 32769:5254.000f.adab |
| 2    | 32768 + 2 = 32770 | 32770:5254.000f.adab |

The bridge priority is written in decimal and the MAC address in hexadecimal (as usual). A colon often separates the two.

##### Comparing BIDs

The MAC address in the second part of the BID isn't the MAC of any of the switch's ports. It's a separate MAC address that identifies the switch as a whole.

For example, which of these BIDs is numerically lower and therefore superior?

| Switch | BID                  |
| :----- | :------------------- |
| SW1    | 32769:5254.000f.adab |
| SW2    | 32769:5254.0013.cf9a |
| SW3    | 32769:5254.0016.5d5e |
| SW4    | 32769:5254.001d.d23a |

Although you divide the BID into parts, it's just a 64-bit number. The bits written on the left (the bridge priority) are the most significant, so compare them first:

1. Compare the bridge priorities. All four switches have the same bridge priority of 32769, so compare the MAC addresses to break the tie.
2. Compare the most significant MAC digits first. The first six digits of all four MAC addresses (the OUI) are the same: 5254.00.
3. Compare the next digit. It's 1 for SW2, SW3, and SW4, but 0 for SW1. SW1 has the lowest BID of the four, so it's the root bridge.

Confirm this with `show spanning-tree` on SW1:

```text
SW1# show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     5254.000f.adab
             This bridge is the root    
. . .
  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     5254.000f.adab
. . .
```

##### Comparing the bridge priority

A switch connected to the router that end hosts use to access external networks is a good choice for root bridge. Frames should have an efficient path to the router without passing through too many switches.

In this example, you configure SW3 as the root bridge and lower SW4's priority so it functions as a **secondary root bridge**. The secondary root bridge takes over as root if the root bridge malfunctions, because it has the lowest BID of the remaining switches.

To configure a switch's bridge priority, run `spanning-tree vlan vlan-id priority priority-value`. A value that isn't a multiple of 4096 fails:

```text
SW3(config)# spanning-tree vlan 1 priority 20000
% Bridge Priority must be in increments of 4096.
% Allowed values are:
  0     4096  8192  12288 16384 20480 24576 28672
  32768 36864 40960 45056 49152 53248 57344 61440
```

You can configure the priority only in increments of 4096. The bridge priority field is 16 bits, but only the four most significant bits make up the configurable priority value: the bits with values of 0d32768, 0d16384, 0d8192, and 0d4096. 4096 is the value of the least significant bit you can change.

SW3 gets priority 24576 and SW4 gets 28672. SW4's output shows SW3 (24577) as root:

```text
SW3(config)# spanning-tree vlan 1 priority 24576

SW4(config)# spanning-tree vlan 1 priority 28672
SW4(config)# do show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    24577
             Address     5254.0016.5d5e
. . .
  Bridge ID  Priority    28673  (priority 28672 sys-id-ext 1)
             Address     5254.001d.d23a
. . .
```

You can also configure the bridge priority with `spanning-tree vlan vlan-id root {primary | secondary}`:

| Keyword     | Effect                                                                                                                                                                                                                                                                                 |
| :---------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `secondary` | Sets the priority to 28672, one increment of 4096 under the default of 32768.                                                                                                                                                                                                          |
| `primary`   | Sets the priority to 24576, two increments of 4096 under the default.<br><br>If 24576 isn't enough to make the switch the root bridge (that is, the current root bridge's priority is 24576), sets the priority to the highest multiple of 4096 that makes the switch the root bridge. |

These commands work if all other switches in the LAN have the default priority of 32768. The switch with `primary` becomes the root bridge, and the switch with `secondary` is next in line if the root bridge fails.

These commands aren't recommended, for two reasons:

- `secondary` doesn't guarantee that the switch is next in line to become the root bridge if the current root bridge fails. Another non-root switch could have a priority value lower than 28672.
- `primary` fails in some situations. It can't set the switch's priority to 0.

{{< admonition "Best practice" tip >}}
To ensure a switch is the root bridge, run <code>spanning-tree vlan vlan-id priority 0</code>. The only way to take over the root is to run the same command on another switch with a lower MAC address, and therefore a lower BID.
{{< /admonition >}}

#### Root port selection

After the root bridge election, each non-root switch selects one of its ports as its **root port**: the port with the best path to the root bridge. The switch calculates this from information in the BPDUs it receives from its neighbors.

The switch selects the root port by comparing these parameters, in order of priority:

1. Lowest **root cost**, which measures the port's proximity to the root bridge
2. Lowest neighbor BID
3. Lowest neighbor port ID

A port's root cost indicates how efficient the path to the root bridge is through that port. A lower value is better.

STP port cost values:

| Speed    | Cost |
| :------- | :--- |
| 10 Mbps  | 100  |
| 100 Mbps | 19   |
| 1 Gbps   | 4    |
| 10 Gbps  | 2    |

A port's root cost is the total cost of the ports leading toward the root bridge, not just the cost of the individual port. The switch uses each parameter only when the previous one ties:

| Parameter               | Root port                                                                                                                          |
| :---------------------- | :--------------------------------------------------------------------------------------------------------------------------------- |
| Lowest root cost        | The port with the lowest root cost                                                                                                 |
| Lowest neighbor BID     | If multiple ports have the same root cost, the port connected to the neighbor with the lowest BID                                  |
| Lowest neighbor port ID | If ports have the same root cost and connect to the same neighbor, the port connected to the neighbor port with the lowest port ID |

##### Lowest root cost

After the root bridge election, the root bridge is the only switch that generates new BPDUs. Other switches receive those BPDUs, update some information in them, and forward them to their neighbors.

When a non-root switch forwards a BPDU, it adds the cost of the port it received the BPDU on.

##### Lowest neighbor bridge ID

The receiving switch can use the neighbor's BID as a tiebreaker when it decides its root port. The port connected to the neighbor with the lowest BID becomes the switch's root port.

##### Lowest neighbor port ID

The neighbor port ID is the final tiebreaker for root port selection.

{{< admonition "Neighbor's port ID, not local" warning >}}
The tiebreaker uses the neighbor's port IDs, not the local switch's port ID. To decide SW1's root port, compare the port IDs of SW3's ports that connect to SW1.
{{< /admonition >}}

The **port ID** uniquely identifies each port on the switch. Like the BID, it has two parts:

| Part                        | Value                                                  |
| :-------------------------- | :----------------------------------------------------- |
| Configurable priority value | 128 by default                                         |
| Sequential number           | 1 for the first port, 2 for the second port, and so on |

The `Prio.Nbr` column of `show spanning-tree` shows each port ID:

```text
SW3# show spanning-tree
. . .
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Desg FWD 4         128.1    P2p
Gi0/1               Desg FWD 4         128.2    P2p
Gi0/2               Desg FWD 4         128.3    P2p
```

To influence root port selection, configure a port's priority value (the first part of the port ID) with `spanning-tree vlan vlan-id port-priority priority-value` in interface configuration mode.

To decide which port has a lower port ID, you can just compare the port names. G0/0 has a lower port ID than G0/1, because 0 is lower than 1.

#### Designated port selection

The final step of the STP algorithm is to select designated ports:

| Port role           | State      | Direction                        |
| :------------------ | :--------- | :------------------------------- |
| **Root port**       | Forwarding | Points toward the root bridge    |
| **Designated port** | Forwarding | Points away from the root bridge |

Every port on the root bridge is a designated port, because they all point away from the root bridge.

Each segment in the LAN must have exactly one designated port. The exact meaning of **segment** varies, but here a segment is a link between switches. The switches select designated ports with these parameters, in order of priority:

1. The port on the switch with the lowest root cost becomes designated.
2. The port on the switch with the lowest BID becomes designated.

{{< admonition "What is a segment?" note >}}
A Layer 1 segment is an electrical connection between devices and is equivalent to a collision domain. This is the meaning of segment in these notes. Two connected switches are another example of a Layer 1 segment.

A Layer 3 segment is equivalent to a subnet. Layer 2 and Layer 3 segments usually have a one-to-one relationship (one subnet per VLAN), but a single VLAN can include multiple subnets.
{{< /admonition >}}

The root bridge election and root port selection already identify some designated ports:

- All ports on the root bridge are designated.
- All ports connected to a root port are designated.

Each remaining segment must have one designated port. The other ports are non-designated. **Non-designated ports** are in the blocking state. This is how STP prevents loops.

In the example, SW1 G0/0 connects to SW3 G0/1, a designated port. A segment can have only one designated port, so SW1 G0/0 is a non-designated port.

##### Port on the switch with lowest root cost

Root cost is the first parameter that decides which side of each remaining link becomes designated. The port on the switch with the lowest root cost becomes designated, and the other port becomes non-designated.

{{< admonition "Watch the wording" warning >}}
The rule is "the port on the switch with the lowest root cost becomes designated," not "the port with the lowest root cost becomes designated." You compare each switch's root cost through its root port, not the cost of each port whose role is being decided.
{{< /admonition >}}

##### Port on the switch with lowest bridge ID

If root costs tie, the switches compare their BIDs. The port on the switch with the lowest BID becomes designated, and the port on the other switch becomes non-designated.

#### Summary

The STP algorithm:

1. Root bridge election (one per LAN)
   1. Lowest BID
2. Root port selection (one per switch, excluding the root bridge)
   1. Lowest root cost
   2. Lowest neighbor BID
   3. Lowest neighbor port ID
3. Designated port selection (one per segment)
   1. Port on the switch with the lowest root cost
   2. Port on the switch with the lowest BID

### STP port states and timers

Root, designated, and non-designated are the STP **port roles**. In addition to the three roles, STP has multiple **port states**.

| State      | Behavior                                   | Ports in this state in a stable LAN |
| :--------- | :----------------------------------------- | :---------------------------------- |
| Forwarding | Active. Can forward and receive frames.    | Root and designated ports           |
| Blocking   | Disabled. Can't forward or receive frames. | Non-designated ports                |

A port also goes through some transitional states to prepare to forward frames. Timers govern how long the port spends in each state.

#### STP port states

STP has four main port states: blocking, listening, learning, and forwarding.

You might also hear of a fifth state, **disabled**. A disabled port isn't operational. For example, it's disabled with the `shutdown` command or isn't connected to another device. STP isn't active on such a port, so it's usually not included as an STP port state.

| State              | Forward frames? | Learn MAC addresses? | Stable or transitional?   |
| :----------------- | :-------------- | :------------------- | :------------------------ |
| Blocking (`BLK`)   | No              | No                   | Stable (non-designated)   |
| Listening (`LIS`)  | No              | No                   | Transitional              |
| Learning (`LRN`)   | No              | Yes                  | Transitional              |
| Forwarding (`FWD`) | Yes             | Yes                  | Stable (root, designated) |

When a port is first enabled, for example when you connect it to another device, it enters the **listening** state:

- The port can only send and receive STP BPDUs.
- The port doesn't forward any regular data frames.
- The switch doesn't learn MAC addresses from frames that arrive on the port.
- The switch decides the port's role: root, designated, or non-designated.
- The state is transitional. The port should stay in it for a maximum of 15 seconds.

To see the listening state, disable SW2's G0/1 port, reenable it, and run `show spanning-tree`. `LIS` in the `Sts` column indicates the listening state:

```text
SW2(config)# interface g0/1
SW2(config-if)# shutdown
SW2(config-if)# no shutdown
SW2(config-if)# do show spanning-tree
. . .
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Altn BLK 4         128.1    P2p 
Gi0/1               Root LIS 4         128.2    P2p
```

{{< admonition "Altn role" note >}}
<code>Altn</code> means alternate, which is equivalent to the non-designated role. Alternate is a port role introduced in Rapid STP.
{{< /admonition >}}

After the listening state, the port's role decides its next state:

| Role               | Next state | When             |
| :----------------- | :--------- | :--------------- |
| Non-designated     | Blocking   | Immediately      |
| Root or designated | Learning   | After 15 seconds |

In the **blocking** state, the port is effectively disabled:

- The port doesn't forward frames.
- Its only job is to listen for BPDUs and react if the network changes.
- `show ip interface brief` still shows its status as up/up.
- `show spanning-tree` shows its status as `BLK`.

The **learning** state is similar to the listening state, with one difference: the port starts learning MAC addresses when it receives frames. This state prepares the port to start forwarding traffic. Like the listening state, it's transitional.

After 15 seconds in the learning state, a root or designated port transitions to the **forwarding** state. A forwarding port is a fully operational switch port that can forward traffic.

STP has **converged** when all switches in the network have decided their ports' roles and all ports are in a blocking or forwarding state. The LAN is then stable.

#### STP timers

Three timers govern how STP operates:

| Timer         | Purpose                                                                                     | Duration               |
| :------------ | :------------------------------------------------------------------------------------------ | :--------------------- |
| Hello         | How often BPDUs are sent                                                                    | 2 seconds              |
| Forward delay | The length of the listening and learning states                                             | 15 seconds (per state) |
| Max age       | How long a switch waits to change the STP topology after it stops receiving BPDUs on a port | 20 seconds             |

The root bridge's timers apply to all switches in the LAN. For example, the root bridge's hello timer dictates how often BPDUs are sent in the LAN. All BPDUs originate from the root bridge, and the other switches forward them out of their designated ports.

The **hello timer** determines how often BPDUs are sent. By default, it's 2 seconds, so BPDUs are sent every 2 seconds.

The **forward delay timer** determines the length of the listening and learning states. By default, it's 15 seconds: 15 seconds for listening and 15 seconds for learning. A newly enabled port takes a total of 30 seconds before it can forward frames (except BPDUs).

Each switch port spends a set time in each state before it transitions to another state. This lets the switch be absolutely sure it won't cause a loop when it transitions a port to the forwarding state. In a LAN with redundant connections, preventing loops is very important, because a loop can bring down a LAN in a matter of seconds.

The **max age timer** determines how long a switch waits to change the STP topology after it stops receiving BPDUs on a port. By default, it's 20 seconds. With the default hello timer of 2 seconds, a port can miss 10 BPDUs before the switch recalculates the STP topology, for example by electing a new root bridge and recalculating port roles.

STP can take up to 50 seconds to move a blocking port into the forwarding state:

| Stage           | Time       |
| :-------------- | :--------- |
| Max age timer   | 20 seconds |
| Listening state | 15 seconds |
| Learning state  | 15 seconds |
| **Total**       | **50 seconds** |

If the hardware failure makes SW1's G0/1 port totally nonoperational (down/down state), SW1 reacts immediately. It doesn't wait for the max age timer.

### PortFast and BPDU Guard

Cisco switches include a suite of optional STP features, sometimes called the **STP toolkit**, that can speed up STP convergence and improve stability. The toolkit includes PortFast and BPDU Guard.

Switch ports connected to devices that don't use STP, such as PCs, are always designated ports, so there's no risk of a Layer 2 loop. However, because STP is timer-based, a connected device must wait 30 seconds before it can access the network, when the switch port enters the forwarding state.

#### PortFast

**PortFast** is an optional STP feature that moves a switch port immediately to the forwarding state, bypassing the listening and learning states.

| Scope                             | Command                             | Mode                    |
| :-------------------------------- | :---------------------------------- | :---------------------- |
| A specific port                   | `spanning-tree portfast`            | Interface configuration |
| All access ports (not trunk ports) | `spanning-tree portfast default`    | Global configuration    |

When you enable PortFast on a port, IOS warns you about bridging loops and notes that PortFast takes effect only in non-trunking mode:

```text
SW1(config)# interface g1/0
SW1(config-if)# spanning-tree portfast
%Warning: portfast should only be enabled on ports connected to a single
 host. Connecting hubs, concentrators, switches, bridges, etc... to this
 interface when portfast is enabled, can cause temporary bridging loops.
 Use with CAUTION
%Portfast has been configured on GigabitEthernet1/0 but will only 
 have effect when the interface is in a non-trunking mode.
```

{{< admonition "End hosts only" warning >}}
PortFast puts switch ports in the forwarding state immediately, bypassing the listening and learning states. Enable it only on ports intended for end hosts. Don't connect switches to PortFast-enabled ports.
{{< /admonition >}}

#### BPDU Guard

**BPDU Guard** is another optional STP feature that disables a switch port if it receives a BPDU. Enable it on all PortFast-enabled ports.

PortFast-enabled ports should connect only to end hosts, which don't send BPDUs. If a user carelessly connects another switch to a port meant for end hosts, BPDU Guard disables the port. This prevents the new switch from affecting the STP topology, for example by becoming the new root bridge.

| Scope                       | Command                                  | Mode                    |
| :-------------------------- | :--------------------------------------- | :---------------------- |
| A specific port             | `spanning-tree bpduguard enable`         | Interface configuration |
| All PortFast-enabled ports  | `spanning-tree portfast bpduguard default` | Global configuration  |

```text
SW4(config)# interface g0/0
SW4(config-if)# spanning-tree portfast
SW4(config-if)# spanning-tree bpduguard enable
```

If a port with BPDU Guard enabled receives a BPDU from another switch, the port enters an **error-disabled** state:

```text
%SPANTREE-2-BLOCK_BPDUGUARD: Received BPDU on port Gi0/0 
with BPDU Guard enabled. Disabling port.
%PM-4-ERR_DISABLE: bpduguard error detected on Gi0/0, 
putting Gi0/0 in err-disable state
```

To reenable an error-disabled port:

1. Solve the problem that caused the error. For example, disconnect the switch from the PortFast and BPDU Guard-enabled port.
2. Run `shutdown` on the port.
3. Run `no shutdown` on the port to reset it.

{{< admonition "Remember" tip >}}
Enable PortFast only on ports meant for end hosts, and enable BPDU Guard on all PortFast-enabled ports. You can use only PortFast or only BPDU Guard, but best practice is to use both features together.
{{< /admonition >}}
