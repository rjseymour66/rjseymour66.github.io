+++
title = 'Switches'
date = '2026-07-07T22:58:21-04:00'
weight = 30
draft = false
+++

## Ethernet LAN

An **Ethernet LAN** is a group of devices connected at Layer 2, sharing the same broadcast domain. A LAN switch forwards frames based on destination MAC addresses. When a frame arrives, the switch records the source MAC address and the ingress port, building a **MAC address table**. When forwarding, the switch looks up the destination MAC. If it finds a match, it forwards the frame out only that port (**filtering**). If no match exists, it floods the frame out all ports except the one it arrived on (**unknown unicast flooding**).

## Interfaces

**Interface configuration** mode controls the behavior of individual switch ports. Use `interface` to enter a single port or `interface range` to apply the same settings to multiple ports at once. From there, you set **speed**, **duplex**, **shutdown state**, and a **description** to document port purpose. Prefixing any command with `no` returns that setting to its default. Use `default interface` to reset all settings on a port at once.

### Status and statistics

Every interface reports two independent status fields: **line status** and **line protocol status**. Line status reflects Layer 1. It tells you whether the interface detects a physical signal and hasn't been administratively shut down. Line protocol status reflects Layer 2. It tells you whether the interface is successfully exchanging keepalives or negotiating the data-link protocol with the device on the other end. Because the two fields are independent, an interface can have a healthy physical connection but still fail to bring up its line protocol, or the reverse.

View both fields with these commands:

- `show interfaces` displays "*interface* is *line status*, line protocol is *protocol status*" for one interface or for all interfaces.
- `show interfaces description` displays every interface's Status and Protocol columns in one table.

Use these two commands instead of `show interfaces status`, which collapses both fields into a single Status column and can obscure which layer is actually failing.

| Line status           | Protocol status | Interface status          | Root cause                                                                                                                                                                                                                                              |
| :-------------------- | :-------------- | :------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| up                    | up              | connected                 | Normal operation. Layer 1 has a valid signal and Layer 2 keepalives or protocol negotiation succeeded.                                                                                                                                                  |
| administratively down | down            | disabled                  | An administrator ran `shutdown` on the interface. Line protocol always reports down when the interface is administratively down.                                                                                                                        |
| down                  | down            | notconnect                | No signal at Layer 1. Common causes: unplugged or damaged cable, missing or mismatched SFP, the far-end port shut down or unplugged, or a hardware failure on either end.                                                                               |
| up                    | down            | connected (protocol down) | Layer 1 has a valid signal but Layer 2 isn't up. Common causes: keepalive or encapsulation mismatch on a serial link, a brief window right after `no shutdown` while negotiation completes, or the far-end device not yet running its Layer 2 protocol. |
| down (err-disabled)   | down            | err-disabled              | A protection feature (BPDU Guard, port security violation, UDLD, storm control) forced the port into a hardware-disabled state. Clear it with `shutdown` then `no shutdown`, or configure errdisable recovery.                                          |

### Counters

A **counter** is a running tally the interface keeps of packet and error events, accumulated since boot or since you last ran `clear counters`. `show interfaces` *id* displays every counter for a single interface, alongside the interface's negotiated speed, duplex, and encapsulation.

```text
s6#show interfaces gigabitEthernet 0/2
GigabitEthernet0/2 is up, line protocol is up (connected)
  ...
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     956 packets input, 193351 bytes, 0 no buffer
     Received 956 broadcasts, 0 runts, 0 giants, 0 throttles
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored, 0 abort
     0 watchdog, 0 multicast, 0 pause input
     0 input packets with dribble condition detected
     2357 packets output, 263570 bytes, 0 underruns
     0 output errors, 0 collisions, 10 interface resets
     0 babbles, 0 late collision, 0 deferred
     0 lost carrier, 0 no carrier
     0 output buffer failures, 0 output buffers swapped out
```

- `runts` counts frames shorter than the minimum valid Ethernet frame size of 64 bytes. A nonzero count usually points to a duplex mismatch or a collision fragment.
- `giants` counts frames longer than the maximum size the interface expects. A nonzero count usually points to a jumbo frame or MTU mismatch, or a faulty NIC on the far end.
- `input errors` is the total count of inbound frames the interface discarded for any error condition. It's the sum of runts, giants, CRC errors, frame errors, overruns, and ignored frames.
- `CRC` counts frames that failed the cyclic redundancy check, meaning the frame was corrupted in transit. This points to a Layer 1 problem, such as a bad cable, a bad connector, or electrical interference.
- `frame` counts frames with a framing error, where the frame didn't end on a clean byte boundary and also failed the CRC check. Like CRC errors, this usually traces back to a duplex mismatch or electrical noise.
- `packets output` is the total number of packets the interface has transmitted since the counters were last cleared.
- `output errors` is the total count of frames the interface failed to transmit successfully. It's the sum of underruns, collisions, interface resets, babbles, late collisions, and lost or no carrier events.
- `collisions` counts how many times the interface detected a collision while transmitting. Collisions are normal on a half-duplex link, but any nonzero count on a full-duplex interface signals a duplex mismatch.
- `late collision` counts collisions detected after the interface had already transmitted the first 64 bytes of the frame. A late collision almost always means a duplex mismatch or a cable run longer than the Ethernet distance limit.

### Autogen, speed, duplex

Set an interface to autonegotiate speed and duplex, then confirm the negotiated values.

```text
S4#config t
Enter configuration commands, one per line.  End with CNTL/Z.
S4(config)#interface gigabitEthernet 0/1
S4(config-if)#speed auto
S4(config-if)#duplex auto
S4(config-if)#exit
S4#show interfaces status
```

- `S4#config t` (short for `configure terminal`) enters global configuration mode.
- `Enter configuration commands, one per line. End with CNTL/Z.` is IOS's standard confirmation that you're now in configuration mode. You don't type it.
- `S4(config)#interface gigabitEthernet 0/1` enters interface configuration mode for Gi0/1. IOS accepts a space or no space between the interface type and its slot/port number.
- `S4(config-if)#speed auto` enables speed autonegotiation instead of a hard-coded speed.
- `S4(config-if)#duplex auto` enables duplex autonegotiation instead of a hard-coded duplex mode.
- `S4(config-if)#exit` returns to global configuration mode.
- `S4#show interfaces status` confirms the speed and duplex the interface actually negotiated.

You can also find the negotiated speed and duplex in the output of `show interfaces` *id*.

### Auto-MDIX

**Auto-MDIX** (automatic medium-dependent interface crossover) lets an interface detect whether the attached cable is straight-through or crossover, then automatically swaps its internal transmit and receive pairs to match. This removes the need to pick the correct cable type when connecting a switch to another switch, a router, or a PC. Auto-MDIX is enabled by default on most Cisco switch interfaces, but it only takes effect when the interface is also set to autonegotiate speed and duplex.

```text
s6#show interfaces gigabitEthernet 0/2 status
Port      Name               Status       Vlan       Duplex  Speed Type
Gig0/2                       connected    1          a-full  a-100010/100/1000BaseTX

s6#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
s6(config)#int gi
s6(config)#int gigabitEthernet 0/2
s6(config-if)#no mdix auto
s6(config-if)#exit
```

- `s6#show interfaces gigabitEthernet 0/2 status` displays the interface's current status, VLAN, duplex, and speed before making the change. `a-full` and `a-1000` show that duplex and speed were autonegotiated rather than set manually.
- `s6#conf t` (short for `configure terminal`) enters global configuration mode.
- `Enter configuration commands, one per line. End with CNTL/Z.` is IOS's standard confirmation that you're now in configuration mode. You don't type it.
- `s6(config)#int gi` is incomplete. `gi` uniquely abbreviates `GigabitEthernet`, but without a slot/port number IOS has no specific interface to enter.
- `s6(config)#int gigabitEthernet 0/2` enters interface configuration mode for Gi0/2.
- `s6(config-if)#no mdix auto` disables auto-MDIX on this interface, so it no longer automatically corrects for a mismatched cable. You'd need the correct straight-through or crossover cable instead.
- `s6(config-if)#exit` returns to global configuration mode.

### Add a description

An interface description documents what's actually plugged into a port, so anyone troubleshooting later doesn't have to trace cables or ask around. Set one on every uplink and trunk at minimum, so `show interfaces status` and `show running-config` tell you the far-end device, its port, and the link's purpose at a glance.

Use a consistent format across your network, for example `Link to <device>, port <their-interface>, <purpose>` for switch-to-switch or router uplinks. For WAN circuits, include the circuit ID and provider. For access ports, name the connected device, user, or wall jack instead. Note autonegotiation or a hard-set speed/duplex when it matters for troubleshooting, as in the example below. IOS caps descriptions at 240 characters, and the description is purely cosmetic: it documents the port but doesn't affect switching, routing, or negotiation.

```text
s6#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
s6(config)#int g
s6(config)#int g 0/2
s6(config-if)#description Link to S5, using autonegotiation
s6(config-if)#exit
s6(config)#exit
s6#
%SYS-5-CONFIG_I: Configured from console by console

s6#show int g0/2 status
Port      Name               Status       Vlan       Duplex  Speed Type
Gig0/2    Link to S5, using  connected    1          a-full  a-100010/100/1000BaseTX
```

- `s6#conf t` (short for `configure terminal`) enters global configuration mode.
- `Enter configuration commands, one per line. End with CNTL/Z.` is IOS's standard confirmation that you're now in configuration mode. You don't type it.
- `s6(config)#int g` is incomplete. `g` is ambiguous between `GigabitEthernet` and other interface types starting with `g`, so IOS has no single interface to enter.
- `s6(config)#int g 0/2` is still incomplete. IOS didn't reject it outright, but the abbreviation alone doesn't identify an interface type.
- `s6(config-if)#description Link to S5, using autonegotiation` sets the interface description. IOS takes the rest of the line as-is, including spaces, so there's no need to quote it.
- `s6(config-if)#exit` returns to global configuration mode.
- `s6(config)#exit` returns to Privileged EXEC mode.
- `%SYS-5-CONFIG_I: Configured from console by console` is unsolicited console output logging that a configuration change was made from the console, not a command.
- `s6#show int g0/2 status` confirms the change. The Name column is truncated to fit the table, showing only `Link to S5, using` even though the full description is longer.

### Range commands

Use `interface range` to apply the same command to many interfaces in one pass instead of repeating it in each interface's own configuration mode. It's the practical way to configure a block of unused ports, a stack of access ports that all need the same VLAN and description, or any group of interfaces that share a setting.

```text
s6#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
s6(config)#interface range f0/10 - 24
s6(config-if-range)#description Interface not in use
s6(config-if-range)#^Z
s6#
%SYS-5-CONFIG_I: Configured from console by console

s6#show int status
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/1                        notconnect   1          auto    auto  10/100BaseTX
...
Fa0/11    Interface not in u notconnect   1          auto    auto  10/100BaseTX
Fa0/12    Interface not in u notconnect   1          auto    auto  10/100BaseTX
Fa0/13    Interface not in u notconnect   1          auto    auto  10/100BaseTX
Fa0/14    Interface not in u notconnect   1          auto    auto  10/100BaseTX
Fa0/15    Interface not in u notconnect   1          auto    auto  10/100BaseTX
Fa0/16    Interface not in u notconnect   1          auto    auto  10/100BaseTX
Fa0/17    Interface not in u notconnect   1          auto    auto  10/100BaseTX
Fa0/18    Interface not in u notconnect   1          auto    auto  10/100BaseTX
Fa0/19    Interface not in u notconnect   1          auto    auto  10/100BaseTX
Fa0/20    Interface not in u notconnect   1          auto    auto  10/100BaseTX
Fa0/21    Interface not in u notconnect   1          auto    auto  10/100BaseTX
Fa0/22    Interface not in u notconnect   1          auto    auto  10/100BaseTX
Fa0/23    Interface not in u notconnect   1          auto    auto  10/100BaseTX
Fa0/24    Interface not in u notconnect   1          auto    auto  10/100BaseTX
Gig0/1                       notconnect   1          auto    auto  10/100/1000BaseTX
Gig0/2    Link to S5, using  connected    1          a-full  a-100010/100/1000BaseTX
```

- `s6#conf t` (short for `configure terminal`) enters global configuration mode.
- `Enter configuration commands, one per line. End with CNTL/Z.` is IOS's standard confirmation that you're now in configuration mode. You don't type it.
- `s6(config)#interface range f0/10 - 24` enters interface range configuration mode for Fa0/10 through Fa0/24 at once. Spaces around the `-` are optional.
- `s6(config-if-range)#description Interface not in use` applies the description to every interface in the range, not just one.
- `s6(config-if-range)#^Z` (`Ctrl+Z`) returns directly to Privileged EXEC mode from any configuration mode, skipping the intermediate `exit` steps.
- `%SYS-5-CONFIG_I: Configured from console by console` is unsolicited console output logging that a configuration change was made from the console, not a command.
- `s6#show int status` confirms the change across all interfaces. Fa0/10 through Fa0/24 now show the shared description, while Fa0/1 through Fa0/9 and Gi0/1 have none, and Gi0/2 still shows the individual description set earlier since `interface range` only touched the Fa0/10-24 range.

### Shutdown and bring up

Administratively disable an interface with `shutdown` when you need to take a port down for maintenance, isolate a suspected problem, or lock down a port you're not ready to hand out yet, without losing any of its existing configuration. `no shutdown` brings it back up when you're done.

```text
s6#config t
Enter configuration commands, one per line.  End with CNTL/Z.
s6(config)#int g0/2
s6(config-if)#shutdown

s6#show interface g0/2 status
Port      Name               Status       Vlan       Duplex  Speed Type
Gig0/2    Link to S5, using  disabled 1          auto    auto  10/100/1000BaseTX

s6#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
s6(config)#int g0/2
s6(config-if)#no shutdown
```

- `s6#config t` (short for `configure terminal`) enters global configuration mode.
- `Enter configuration commands, one per line. End with CNTL/Z.` is IOS's standard confirmation that you're now in configuration mode. You don't type it.
- `s6(config)#int g0/2` enters interface configuration mode for Gi0/2.
- `s6(config-if)#shutdown` administratively disables the interface. Unlike a cable fault or the far-end device being down, this is a state you set yourself and it persists until you clear it.
- `s6#show interface g0/2 status` confirms the interface is down. The Status column shows `disabled`, distinct from `notconnect` (no cable) or `connected`.
- `s6#conf t` re-enters global configuration mode.
- `s6(config)#int g0/2` re-enters interface configuration mode for Gi0/2.
- `s6(config-if)#no shutdown` administratively re-enables the interface, so it can come back up once a cable is connected and link is detected.

### Restore defaults

Prefix a command with `no` to remove that setting and return it to its default, rather than typing the default value back in yourself. This works for the settings covered above: speed, duplex, description, and shutdown state.

```text
s6#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
s6(config)#int g0/2
s6(config-if)#no speed
s6(config-if)#no duplex
s6(config-if)#no description
s6(config-if)#no shutdown
```

- `s6#conf t` (short for `configure terminal`) enters global configuration mode.
- `Enter configuration commands, one per line. End with CNTL/Z.` is IOS's standard confirmation that you're now in configuration mode. You don't type it.
- `s6(config)#int g0/2` enters interface configuration mode for Gi0/2.
- `s6(config-if)#no speed` removes the configured speed and returns the interface to speed autonegotiation.
- `s6(config-if)#no duplex` removes the configured duplex and returns the interface to duplex autonegotiation.
- `s6(config-if)#no description` clears the interface description.
- `s6(config-if)#no shutdown` returns the interface to its default administratively enabled state.

### Duplex mismatch

A **duplex mismatch** is a link where one end runs full-duplex and the other runs half-duplex, usually because one side is hard-set while the other is left on autonegotiation. Under IEEE 802.3, a side that doesn't detect autonegotiation on the far end falls back to half-duplex, even if the hard-set peer is actually full-duplex.

A half-duplex interface can't send and receive at once, so it uses CSMA/CD (carrier sense multiple access with collision detection) to arbitrate the wire: listen before sending, then keep listening through the slot time, the first 64 bytes of its own frame. The full-duplex peer skips all of this. It assumes exclusive use of the wire and sends whenever it wants, so it can key up mid-way through the half-duplex side's frame. The half-duplex interface senses that incoming energy while still transmitting, reads it as a collision, and responds with standard CSMA/CD: stop sending, transmit a 32-bit jam signal, then back off and retry.

A collision only counts as normal if it's detected inside that 64-byte slot time. Since the full-duplex peer keeps transmitting well past that window, the half-duplex side often doesn't detect the collision until after its own 64 bytes are already out, and IOS logs it as a late collision instead. A nonzero, climbing `late collision` counter, especially on an interface set to full-duplex that shouldn't see collisions at all, is one of the clearest signs of a mismatch. Check it with `show interfaces` *id* on both ends, and fix any mismatch by matching the settings or returning both sides to `duplex auto`.

### Commands

| Command                                                      | Mode             | Description                                                       |
| :----------------------------------------------------------- | :--------------- | :---------------------------------------------------------------- |
| `default interface` *interface-id*                           | Global config    | Resets all interface settings to defaults                         |
| `description` *text*                                         | Interface config | Adds a text description to the interface                          |
| `duplex [auto\|full\|half]`                                  | Interface config | Sets the duplex mode or enables autonegotiation                   |
| `interface` *type* *port-number*                             | Global config    | Enters interface config mode for a specific port                  |
| `interface range` *type* *port-number* `-` *end-port-number* | Global config    | Enters interface config mode for a range of ports                 |
| `no description`                                             | Interface config | Removes the interface description                                 |
| `no duplex`                                                  | Interface config | Removes the configured duplex and returns to default              |
| `[no] mdix auto`                                             | Interface config | Enables or disables automatic MDI-X crossover detection           |
| `no shutdown`                                                | Interface config | Enables an administratively disabled interface                    |
| `no speed`                                                   | Interface config | Removes the configured speed and returns to default               |
| `show interfaces` *id*                                       | Privileged EXEC  | Displays detailed statistics for a specific interface             |
| `show interfaces [` *type* *number* `]`                      | Privileged EXEC  | Displays detailed statistics for all interfaces or a specific one |
| `show interfaces` *id* `counters`                            | Privileged EXEC  | Displays traffic counters for an interface                        |
| `show interfaces description`                                | Privileged EXEC  | Displays interface descriptions and operational status            |
| `show interfaces status`                                     | Privileged EXEC  | Displays status, duplex, and speed for all interfaces             |
| `show interfaces [` *type* *number* `] status`               | Privileged EXEC  | Displays status for all interfaces or a specific one              |
| `show interfaces vlan` *number*                              | Privileged EXEC  | Displays status and statistics for a VLAN SVI                     |
| `shutdown`                                                   | Interface config | Administratively disables the interface                           |
| `speed [10\|100\|1000\|auto]`                                | Interface config | Sets the interface speed or enables autonegotiation               |

## MAC address table

| Command                                                   | Description                                                                                                            |
| :-------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------- |
| `clear mac address-table dynamic`                         | Clears dynamic entries. Optionally filter by `vlan` *vlan-number*, `interface` *interface-id*, or `address` *mac-addr* |
| `show mac address-table`                                  | Displays all MAC address table entries                                                                                 |
| `show mac address-table aging-time`                       | Displays the MAC address aging timer                                                                                   |
| `show mac address-table count`                            | Displays the number of entries per VLAN                                                                                |
| `show mac address-table dynamic`                          | Displays only dynamically learned entries                                                                              |
| `show mac address-table dynamic address` *mac-addr*       | Filters dynamic entries by MAC address                                                                                 |
| `show mac address-table dynamic interface` *interface-id* | Filters dynamic entries by interface                                                                                   |
| `show mac address-table dynamic vlan` *vlan-id*           | Filters dynamic entries by VLAN                                                                                        |

## Configuration files

A switch stores configuration and code across four types of memory, each with different persistence characteristics.

IOS works with two configuration files. The **running configuration** is the configuration currently active on the device, held in RAM. The **startup configuration** is the configuration saved in NVRAM that IOS loads into RAM as the running-config every time the device boots. Until you save your changes, they exist only in the running-config.

| Memory | Volatile?                               | Stores                                                     | Role in configuration                                 |
| :----- | :-------------------------------------- | :--------------------------------------------------------- | :---------------------------------------------------- |
| RAM    | Yes                                     | Running configuration, routing table, ARP cache            | Holds the active config. Lost on reload unless saved  |
| NVRAM  | No                                      | Startup configuration                                      | Loaded into RAM as the running-config at boot         |
| Flash  | No                                      | IOS image (and often the startup-config and license files) | Supplies the IOS image loaded into RAM at boot        |
| ROM    | No (not rewritable in normal operation) | Bootstrap program, ROM Monitor (ROMmon), mini-IOS          | Runs POST and loads the IOS image from Flash into RAM |

**RAM** holds the running-config. Because RAM is volatile, any change you make in configuration mode exists only in RAM until you save it. Run `show running-config` to view it.

**NVRAM** holds the startup-config. Running `copy running-config startup-config` copies RAM's running-config into NVRAM, committing your changes so they survive a reload. If you reload without saving, IOS reloads NVRAM's startup-config into RAM, discarding any unsaved changes.

**Flash memory** holds the IOS image itself, the switch's file system for boot images and, on some platforms, backup configuration and license files. Use `dir flash:` to view its contents.

**ROM** holds the bootstrap program and ROMmon, the low-level code that runs power-on self-test (POST), locates the IOS image in Flash, and loads it into RAM before handing off to the startup-config in NVRAM.

### Startup configuration

| Command                              | Mode            | Description                                                           |
| :----------------------------------- | :-------------- | :-------------------------------------------------------------------- |
| `copy running-config startup-config` | Privileged EXEC | Saves the running configuration to NVRAM as the startup configuration |
| `copy startup-config running-config` | Privileged EXEC | Merges the startup configuration into the running configuration       |
| `erase nvram:`                       | Privileged EXEC | Erases all NVRAM contents                                             |
| `erase startup-config`               | Privileged EXEC | Erases the startup configuration from NVRAM                           |
| `show startup-config`                | Privileged EXEC | Displays the saved startup configuration                              |
| `write erase`                        | Privileged EXEC | Alias for `erase startup-config`                                      |

### Running configuration

| Command                                         | Mode            | Description                                                |
| :---------------------------------------------- | :-------------- | :--------------------------------------------------------- |
| `show running-config`                           | Privileged EXEC | Displays the current running configuration                 |
| `show running-config all`                       | Privileged EXEC | Displays running config including default values           |
| `show running-config interface` *type* *number* | Privileged EXEC | Displays running config for a specific interface           |
| `show running-config \| begin line vty`         | Privileged EXEC | Displays running config starting from the VTY line section |

## Setting passwords

Unsecured console and VTY lines leave the device open to anyone who can reach it physically or over the network. IOS offers two authentication models: **line password** authentication (`login` + `password`) applies a single shared password to the line, while **local authentication** (`login local` + `username secret`) requires per-user credentials stored on the device. For remote access, always use **SSH** instead of Telnet. Telnet sends credentials in plaintext. SSH encrypts the session.

### Enable secret

Set a hashed password on Privileged EXEC mode so only authorized users can move past User EXEC and reach device configuration.

```text
configure terminal
enable secret <password>
```

### Console line password

Require a password on the console port so physical access to the device isn't enough to reach User EXEC mode on its own.

```text
line console 0
password <password>
login
exit
```

### VTY line password

Require a password on the VTY lines so remote sessions can't reach User EXEC mode without authenticating. `line vty 0 15` covers all 16 VTY lines, and `transport input all` permits both Telnet and SSH. Restrict it to `ssh` in production to avoid sending credentials in plaintext.

```text
line vty 0 15
password <password>
login
transport input all
end
```

### Username password

Create a local user account with a hashed password, so you have credentials ready before you enable local authentication on a line.

```text
configure terminal
username <name> secret <password>
```

### Local username login

Require per-user credentials instead of one shared line password, so each administrator authenticates with their own account and you get individual accountability for access. Start by creating the local user account.

```text
configure terminal
username <name> secret <password>
```

#### Console

Configure the console line to authenticate against the local user database instead of a shared line password. `line con 0` is a valid abbreviation for `line console 0`, entered from global configuration mode. Once `login local` is set, the line no longer authenticates with a shared password, so `no password` clears out any password left over from a previous configuration.

```text
line con 0
login local
no password
```

#### Telnet (VTY)

Configure the VTY lines the same way, so remote sessions authenticate against local user accounts instead of a shared line password.

```text
line vty 0 15
login local
no password
transport input all
```

`transport input all` permits both Telnet and SSH on the VTY lines. Restrict it to `ssh` in production, since Telnet sends credentials in plaintext.

### Transport

Enter `transport input` from line configuration mode to control which remote access protocols a line accepts. Use it to lock a line down to SSH only in production, to permit Telnet temporarily in a lab, or to disable remote access to a line entirely.

| Command                      | Description                                       |
| :--------------------------- | :------------------------------------------------ |
| `transport input all`        | Permits both Telnet and SSH                       |
| `transport input none`       | Disables all remote access protocols on the line  |
| `transport input ssh`        | Permits SSH only                                  |
| `transport input telnet`     | Permits Telnet only                               |
| `transport input telnet ssh` | Permits both Telnet and SSH (equivalent to `all`) |

| Command                                    | Mode          | Description                                               |
| :----------------------------------------- | :------------ | :-------------------------------------------------------- |
| `crypto key generate rsa modulus` *bits*   | Global config | Generates RSA keys for SSH (512-2048 bits)                |
| `hostname` *name*                          | Global config | Sets the device hostname, required for SSH key generation |
| `ip domain name` *fqdn*                    | Global config | Sets the domain name, required for SSH key generation     |
| `ip ssh version 2`                         | Global config | Forces SSHv2 only                                         |
| `line console 0`                           | Global config | Enters line config mode for the console port              |
| `line vty` *first-vty* *last-vty*          | Global config | Enters line config mode for VTY lines (remote access)     |
| `login`                                    | Line config   | Enables line password authentication                      |
| `login local`                              | Line config   | Enables local user account authentication                 |
| `password` *password*                      | Line config   | Sets the plain-text line password                         |
| `transport input [telnet\|ssh\|all\|none]` | Line config   | Restricts allowed remote access protocols on the line     |
| `username` *name* `secret` *password*      | Global config | Creates a local user with a hashed password               |

## SSH and security

SSH requires a hostname, a domain name, and an RSA key pair before it can run. The example below sets the hostname, sets the domain name, generates the RSA keys, and forces SSH version 2.

```text
Switch>en
Switch#config t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#hostname SW3
SW3(config)#ip domain name example.com
SW3(config)#crypto key generate rsa
The name for the keys will be: SW3.example.com
Choose the size of the key modulus in the range of 360 to 4096 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.

How many bits in the modulus [512]: 1024
% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]

SW3(config)#ip ssh version 2
```

- `en` (short for `enable`) moves from User EXEC to Privileged EXEC mode.
- `config t` (short for `configure terminal`) enters global configuration mode.
- `hostname SW3` sets the device hostname, which changes the prompt and becomes part of the RSA key name.
- `ip domain name example.com` sets the domain name. Combined with the hostname, it forms the key name `SW3.example.com`.
- `crypto key generate rsa` generates the RSA key pair SSH uses to encrypt sessions. IOS prompts for a modulus size (in bits) since none was given on the command line. Larger moduli are more secure but take longer to generate. 1024 bits is the practical minimum for SSHv2.
- `ip ssh version 2` forces the device to use SSHv2 only. SSHv1 has known vulnerabilities and shouldn't be enabled.

Enabling SSH only makes the device *capable* of running the protocol. It doesn't make the VTY lines accept it. Apply this to actually allow SSH sessions in:

```text
SW3(config)#line vty 0 15
SW3(config-line)#login local
SW3(config-line)#transport input all
SW3(config-line)#exit
SW3(config)#username ryan secret password
```

`line vty 0 15` enters line config mode for all 16 VTY lines. `login local` authenticates incoming sessions against the local username database instead of a shared line password, so each admin needs their own account (see [Local username login](#local-username-login)). `transport input all` permits both Telnet and SSH on the line. Without it, or with `transport input none`, the VTY lines reject incoming connections regardless of protocol. `exit` returns to global configuration mode.

Without this step, the `crypto key`/`ip ssh version 2` work only prepares the device to speak SSH. The VTY lines still control whether any remote session, SSH or Telnet, gets in at all, and how it authenticates.

`username ryan secret password` creates the local account that `login local` checks against. It has to exist before anyone can authenticate. Without it, `login local` rejects every login attempt because there are no local accounts to match against. `secret` hashes the password, so store it as `secret`, never `password`, for any account used for login.

```text
SW3(config)#line vty 0 15
SW3(config-line)#transport input ssh
SW3(config-line)#exit
SW3#show ip ssh
SSH Enabled - version 2.0
Authentication timeout: 120 secs; Authentication retries: 3
SW3#show ssh
%No SSHv2 server connections running.
%No SSHv1 server connections running.
```

`transport input ssh` narrows the VTY lines from `all` to SSH only, so Telnet is no longer accepted even though it was permitted earlier. `exit` returns to global configuration mode. `show ip ssh` verifies the SSH configuration itself: it confirms SSH is enabled, running version 2.0, and reports the authentication timeout and retry limit. `show ssh` shows active SSH sessions rather than configuration, so it reports no connections here because no client has connected yet.

### Commands

| Command                        | Description                                     |
| :----------------------------- | :---------------------------------------------- |
| `show crypto key mypubkey rsa` | Displays the device's RSA public key            |
| `show ip ssh`                  | Displays SSH version and configuration settings |
| `show ssh`                     | Displays active SSH sessions                    |

## IPv4 configuration

Give the switch a management IP address on a VLAN interface (SVI) and a default gateway, so you can reach it over the network instead of only through the console.

```text
SW3>enable
SW3#config t
Enter configuration commands, one per line.  End with CNTL/Z.
SW3(config)#interface vlan 1
SW3(config-if)#ip address 192.168.1.200 255.255.255.0
SW3(config-if)#no shutdown

SW3(config-if)#
%LINK-3-UPDOWN: Interface Vlan1, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up
exit
SW3(config)#ip default-gateway 192.168.1.1
```

- `SW3>enable` moves to Privileged EXEC mode.
- `SW3#config t` (short for `configure terminal`) enters global configuration mode.
- `Enter configuration commands, one per line. End with CNTL/Z.` is IOS's standard confirmation that you're now in configuration mode. You don't type it.
- `SW3(config)#interface vlan 1` enters interface configuration mode for VLAN 1's SVI, the logical Layer 3 interface used to manage the switch.
- `SW3(config-if)#ip address 192.168.1.200 255.255.255.0` assigns that interface a static management address.
- `SW3(config-if)#no shutdown` administratively enables the interface, since VLAN SVIs are down by default.
- `%LINK-3-UPDOWN: Interface Vlan1, changed state to down` is unsolicited console output confirming the link-layer state change, not a command.
- `%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up` is unsolicited console output confirming the line protocol came up once at least one active port belongs to VLAN 1.
- `exit` returns to global configuration mode.
- `SW3(config)#ip default-gateway 192.168.1.1` sets the gateway the switch itself uses for its own management traffic.

### DHCP

Configure the management interface to obtain its IP address automatically from a DHCP server instead of a static assignment.

```text
SW3>enable
SW3#config t
Enter configuration commands, one per line.  End with CNTL/Z.
SW3(config)#interface vlan 1
SW3(config-if)#ip address dhcp
SW3(config-if)#no shutdown
SW3#show dhcp lease
Temp IP addr: 0.0.0.0 for peer on Interface: Vlan1
Temp sub net mask: 0.0.0.0
   DHCP Lease server: 0.0.0.0 , state: Selecting
   DHCP Transaction id: 127A054E
   Lease: 0 secs,  Renewal: 0 secs,  Rebind: 0 secs
Temp default-gateway addr: 0.0.0.0
   Next timer fires after: 00:00:00
   Retry count: 7  Client-ID:cisco-0004.9A51.6206-Vlan
   Client-ID hex dump: 636973636F2D303030342E394135312E
                       63230362D566C616E
   Hostname: SW3
SW3#show interfaces vlan 1
Vlan1 is up, line protocol is up
  Hardware is CPU Interface, address is 0004.9a51.6206 (bia 0004.9a51.6206)
  MTU 1500 bytes, BW 100000 Kbit, DLY 1000000 usec,
     reliability 255/255, txload 1/255, rxload 1/255
  ...
SW3#show ip default-gateway
```

- `SW3>enable` moves to Privileged EXEC mode.
- `SW3#config t` (short for `configure terminal`) enters global configuration mode.
- `Enter configuration commands, one per line. End with CNTL/Z.` is IOS's standard confirmation that you're now in configuration mode. You don't type it.
- `SW3(config)#interface vlan 1` enters interface configuration mode for VLAN 1's SVI.
- `SW3(config-if)#ip address dhcp` configures the interface to request an IPv4 address from a DHCP server instead of using a static address.
- `SW3(config-if)#no shutdown` administratively enables the interface so it can send the DHCP request.
- `SW3#show dhcp lease` displays the client's current DHCP lease. This output shows `state: Selecting` with every address field at `0.0.0.0`, meaning the switch had just sent a `DHCPDISCOVER` and hadn't received an offer yet. A completed lease shows the assigned address, lease server, and non-zero lease timers.
- `SW3#show interfaces vlan 1` confirms the SVI is `up, line protocol is up`, independent of whether DHCP has finished assigning an address.
- `SW3#show ip default-gateway` displays the default gateway currently in use, whether it came from DHCP or was set manually with `ip default-gateway`.

### Commands

| Command                                      | Mode             | Description                                             |
| :------------------------------------------- | :--------------- | :------------------------------------------------------ |
| `interface vlan` *number*                    | Global config    | Enters interface config for the management VLAN (SVI)   |
| `ip address` *ip-addr* *subnet-mask*         | Interface config | Assigns a static IP address                             |
| `ip address dhcp`                            | Interface config | Configures the interface to obtain an address from DHCP |
| `ip default-gateway` *addr*                  | Global config    | Sets the default gateway for management traffic         |
| `ip name-server` *server-ip-1* *server-ip-2* | Global config    | Configures DNS server addresses                         |
| `show dhcp lease`                            | Privileged EXEC  | Displays DHCP lease information obtained by the switch  |
| `show ip default-gateway`                    | Privileged EXEC  | Displays the configured default gateway                 |

## History

Use these commands to control how many previous commands IOS remembers, and to review that history for the current session.

| Command                     | Description                                                                 |
| :-------------------------- | :-------------------------------------------------------------------------- |
| `history size` *x*          | Sets the command history buffer size for a line, persisting across sessions |
| `show history`              | Displays the command history for the current session                        |
| `terminal history size` *x* | Temporarily sets the command history buffer size for the current session    |

## Web UI

The web UI provides a browser-based alternative to the CLI for basic device management. Disable the plaintext HTTP server in favor of HTTPS, and require the same local-authentication model used for CLI logins. Cisco recommends disabling the HTTP server, since it transmits credentials and management traffic in plaintext.

| Command                                              | Description                                                            |
| :--------------------------------------------------- | :--------------------------------------------------------------------- |
| `ip http authentication local`                       | Requires web UI logins to authenticate against the local user database |
| `ip http secure-server`                              | Enables the HTTPS web server                                           |
| `no ip http server`                                  | Disables the insecure HTTP web server                                  |
| `username` *name* `privilege 15 password` *password* | Creates a local user with full (level 15) privilege for web UI login   |

## Other configuration

| Command                    | Mode          | Description                                           |
| :------------------------- | :------------ | :---------------------------------------------------- |
| `enable secret` *password* | Global config | Sets a hashed password for privileged EXEC access     |
| `hostname` *name*          | Global config | Sets the device hostname                              |
| `logging synchronous`      | Line config   | Prevents log messages from interrupting command input |
| `[no] logging console`     | Global config | Enables or disables log messages to the console       |

## VLANs

Normal switching logic forwards frames using only MAC addresses and interfaces: a frame arrives on one interface, the switch looks up the destination MAC address in its MAC address table, and forwards the frame out the interface that address maps to.

A **VLAN** (virtual LAN) changes this. A VLAN lets a subset of a switch's ports act as their own separate Ethernet LAN, even though every port sits on the same physical switch. To forward a frame, the switch first identifies which VLAN it belongs to, then looks up the destination MAC address only among the entries learned on that VLAN, and forwards the frame only out ports that belong to that same VLAN.

### Concepts

A LAN is all the devices that share the same **broadcast domain**. When any device sends a broadcast frame, every other device in that domain receives a copy of it, whether or not the frame is actually relevant to them. By default, a switch treats all of its interfaces as one broadcast domain, so a broadcast sent on any port reaches every other port.

VLANs let you divide a single switch into multiple broadcast domains instead of just one. Splitting hosts into separate VLANs limits how many devices receive any given broadcast frame, which cuts down the effort devices waste processing broadcasts that don't concern them.

### Trunking

A **trunk** is a single link, usually between two switches or between a switch and a router, that carries traffic for more than one VLAN at the same time. Trunking lets a switch forward frames from multiple VLANs over that single physical connection by adding a small header to each Ethernet frame. Without trunking, you'd need a separate physical link per VLAN to connect two switches, which doesn't scale as the number of VLANs grows.

A **link** is the direct Layer 1 connection between two adjacent interfaces, whether that's copper, fiber, or a wireless bridge. In the simplest case, a link is just a single cable, but it can also be a bundle of multiple physical cables acting as one logical connection, for example an EtherChannel. What matters for trunking isn't the medium itself, but that both ends' interfaces are configured to expect tagged, multi-VLAN traffic over it.

To keep frames from different VLANs distinguishable on that shared link, the sending switch adds a VLAN tag to each frame before forwarding it out the trunk. This **tagging** inserts a small header into the frame identifying which VLAN it belongs to. IEEE 802.1Q is the standard tagging format. Cisco's older proprietary alternative, ISL, is now considered legacy.

The **VLAN ID** is the number carried inside that tag, a value from 1 to 4094, that identifies the specific VLAN a frame belongs to. The switch on the receiving end of the trunk reads the VLAN ID to determine which broadcast domain the frame came from, then forwards it using only the MAC table entries and ports that belong to that VLAN.

#### 802.1Q and ISL

**802.1Q** is the IEEE open standard for VLAN tagging, and it's now the format nearly every vendor uses. It tags a frame by inserting a 4-byte field directly into the existing Ethernet header, carrying the VLAN ID and an 802.1p priority value for QoS. 802.1Q also defines a *native VLAN*, the one VLAN on a trunk whose frames are sent untagged. Both ends of a trunk must agree on the native VLAN, or frames can end up delivered to the wrong VLAN.

**ISL** (Inter-Switch Link) is Cisco's older, proprietary alternative. Rather than tagging the existing frame, it wraps the whole frame in a new 26-byte header and a new trailer, adding more overhead and forcing the switch to recompute the frame check sequence. ISL has no native VLAN, every frame is tagged, and Cisco has removed ISL support from most current switch platforms in favor of 802.1Q.

VLAN IDs span two ranges. The **normal range** covers 1 to 1005 and is stored locally in `vlan.dat`.

| Range     | Port            | Description                                                                           |
| :-------- | :-------------- | :------------------------------------------------------------------------------------ |
| 1         | Ethernet        | Default VLAN. Every port belongs to it until reassigned. Can't be deleted or renamed. |
| 2-1001    | Ethernet        | User-configurable normal-range VLANs.                                                 |
| 1002-1005 | FDDI/Token Ring | Reserved for legacy FDDI and Token Ring media. Can't be deleted.                      |

The **extended range**, 1006 to 4094, exists for larger deployments such as service providers and isn't saved to `vlan.dat`.

**VTP** (VLAN Trunking Protocol) is a Cisco-proprietary protocol that synchronizes the VLAN database across every switch in a VTP domain, so you create a VLAN once instead of on every switch. A switch runs as a server (creates and advertises VLANs), a client (accepts advertisements but can't make changes), or transparent (ignores the domain's database but still forwards advertisements). VTP versions 1 and 2 only synchronize normal-range VLANs and rely on a revision number that can let a stale switch overwrite the domain's database. Version 3 adds extended-range support and stronger safeguards. VTP pruning is an optional feature that stops trunks from flooding traffic for VLANs no downstream switch actually uses.

### Administrative mode

An interface's **administrative mode** is the switchport mode you (or the platform default) configured it to run, as opposed to its *operational mode*, which is the mode the interface actually settled into after negotiation. The two don't always match: an interface administratively set to negotiate a trunk can still end up operating as a plain access port if the far end never agrees to trunk.

That negotiation runs over **DTP** (Dynamic Trunking Protocol), a Cisco-proprietary protocol switches use to decide whether a link becomes a trunk, based on the administrative mode configured on each end. A trunk configured with `switchport mode trunk` is a *static* or *configured* trunk: it trunks unconditionally, without waiting on the far end to agree. A trunk that forms through `dynamic desirable` or `dynamic auto` is a *negotiated* trunk: whether it actually becomes a trunk depends on what the interface on the other end is set to.

Cisco recommends disabling DTP negotiation on most ports. A port left in `dynamic auto` or `dynamic desirable` will negotiate a trunk with anything that sends it the right DTP frames, including a rogue switch or a host running switch-spoofing software, handing an attacker access to every VLAN the trunk carries. Add `switchport nonegotiate` to a statically configured `access` or `trunk` port to stop it from sending or responding to DTP altogether.

| Command option      | Description                                                                                                 |
| :------------------- | :------------------------------------------------------------------------------------------------------------ |
| `access`              | Always operates as an access port. Never trunks, regardless of what the far end negotiates.                 |
| `trunk`               | Always operates as a trunk, unconditionally. Still sends DTP frames by default, encouraging the far end to also trunk. |
| `dynamic desirable`   | Actively sends DTP frames to negotiate a trunk. Becomes a trunk if the far end is set to `trunk`, `dynamic desirable`, or `dynamic auto`. |
| `dynamic auto`        | Passively waits for the far end to initiate negotiation. Becomes a trunk only if the far end is set to `trunk` or `dynamic desirable`. |

Two interfaces both left on `dynamic auto` never trunk. Neither side initiates DTP negotiation, so the link stays an access port even though both ends are technically willing to trunk if asked. This is the platform default on many Catalyst switches, and it's exactly what the following output shows.

```text
S5#show interfaces gigabitEthernet 0/1 switchport
Name: Gig0/1
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: static access
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: native
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 1 (default)
Voice VLAN: none
Administrative private-vlan host-association: none
Administrative private-vlan mapping: none
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk private VLANs: none
Operational private-vlan: none
Trunking VLANs Enabled: All
Pruning VLANs Enabled: 2-1001
Capture Mode Disabled
Capture VLANs Allowed: ALL
Protected: false
Unknown unicast blocked: disabled
Unknown multicast blocked: disabled
Appliance trust: none
```

- `Administrative Mode: dynamic auto` is the configured mode. This interface is willing to trunk, but only if the far end asks first.
- `Operational Mode: static access` is what the interface actually settled into. Since nothing initiated negotiation, it defaulted to a plain access port.
- `Administrative Trunking Encapsulation: dot1q` is the encapsulation this interface would use if it became a trunk. `Operational Trunking Encapsulation: native` confirms it isn't trunking right now, so no tagging is actually happening.
- `Negotiation of Trunking: On` confirms DTP is active on this interface, even though it hasn't resulted in a trunk yet.
- `Access Mode VLAN: 1 (default)` is the VLAN this port uses in its current access-mode state.
- `Trunking Native Mode VLAN: 1 (default)` is the native VLAN this port would use if it does become a trunk.
- `Pruning VLANs Enabled: 2-1001` is the range of VLANs eligible for VTP pruning on this trunk, the normal range minus VLAN 1 and the reserved 1002-1005 VLANs.

Change the interface to `dynamic desirable` to have it actively request a trunk instead of waiting.

```text
S5#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S5(config)#interface g
S5(config)#interface gigabitEthernet 0/1
S5(config-if)#switchport mode dynamic desirable

S5(config-if)#
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to up
^Z
S5#
%SYS-5-CONFIG_I: Configured from console by console

S5#show interfaces giga
S5#show interfaces gigabitEthernet 0/1 switchport
Name: Gig0/1
Switchport: Enabled
Administrative Mode: dynamic desirable
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 1 (default)
Voice VLAN: none
Administrative private-vlan host-association: none
Administrative private-vlan mapping: none
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk private VLANs: none
Operational private-vlan: none
Trunking VLANs Enabled: All
Pruning VLANs Enabled: 2-1001
Capture Mode Disabled
Capture VLANs Allowed: ALL
Protected: false
Unknown unicast blocked: disabled
Unknown multicast blocked: disabled
Appliance trust: none
S5#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Gig0/1      desirable    n-802.1q       trunking      1

Port        Vlans allowed on trunk
Gig0/1      1-1005

Port        Vlans allowed and active in management domain
Gig0/1      1,2,3

Port        Vlans in spanning tree forwarding state and not pruned
Gig0/1      1,2,3
```

- `S5(config)#interface g` is an incomplete abbreviation. IOS can't tell which interface type you mean from `g` alone, so it rejects the command and returns to the same prompt.
- `S5(config-if)#switchport mode dynamic desirable` sets the interface to actively negotiate a trunk with DTP instead of waiting for the far end.
- The three `%LINEPROTO-5-UPDOWN` messages record the line protocol briefly flapping (up, down, up) while DTP renegotiates the link. This is normal and expected when a mode change triggers a new trunk negotiation.
- `S5#show interfaces giga` is another incomplete abbreviation, rejected the same way.
- `Operational Mode: trunk` in the second `show interfaces gigabitEthernet 0/1 switchport` output confirms the negotiation succeeded and the interface is now trunking.
- `Operational Trunking Encapsulation: dot1q` now matches the administrative setting, confirming frames are actually being tagged.
- `S5#show interfaces trunk` only lists interfaces currently trunking. Gig0/1 appearing here at all, with `Status: trunking`, is direct confirmation the trunk is up. `Mode: desirable` and `Encapsulation: n-802.1q` show it got there through negotiation (the `n-` prefix means negotiated, not statically configured) rather than a hard-coded `switchport mode trunk`.
- The VLANs allowed on the trunk (`1-1005`) are every normal-range VLAN by default, while the VLANs active in the management domain (`1,2,3`) are only the ones that actually exist on the switch, VLAN 1 plus the two you created earlier.

How you configure a port depends on what's connected to it, not on one universal template.

An end-user device like a PC or a printer only ever needs one VLAN, so lock the port to access mode and turn off negotiation entirely:

```text
S5(config)#interface fastEthernet 0/5
S5(config-if)#switchport mode access
S5(config-if)#switchport access vlan 10
S5(config-if)#switchport nonegotiate
```

An IP phone needs two VLANs: its own voice traffic, and a separate data VLAN passed through to a PC plugged into the phone's second port. `switchport voice vlan` puts the phone on its own tagged VLAN, while `switchport access vlan` still controls the PC's untagged traffic:

```text
S5(config)#interface fastEthernet 0/6
S5(config-if)#switchport mode access
S5(config-if)#switchport access vlan 10
S5(config-if)#switchport voice vlan 110
S5(config-if)#switchport nonegotiate
```

A host that needs more than one VLAN, for example a virtualization host running VMs across several VLANs, needs a trunk instead, restricted to only the VLANs it actually uses:

```text
S5(config)#interface gigabitEthernet 0/2
S5(config-if)#switchport mode trunk
S5(config-if)#switchport trunk allowed vlan 10,20
S5(config-if)#switchport nonegotiate
```

An uplink to another switch is also a trunk, but instead of a narrow VLAN list, it usually carries every VLAN in use across both switches:

```text
S5(config)#interface gigabitEthernet 0/1
S5(config-if)#switchport mode trunk
S5(config-if)#switchport nonegotiate
```

In every case, `switchport nonegotiate` disables DTP on the port, so its mode only ever changes when someone reconfigures it directly.

### Configure (Long)

Create a VLAN, name it, and assign access ports to it, then confirm the change. You'd do this to separate a group of devices, such an accounting department, into their own broadcast domain, isolating their traffic from the rest of the network without needing a dedicated physical switch.

```text
S5#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                Fa0/5, Fa0/6, Fa0/7, Fa0/8
                                                Fa0/9, Fa0/10, Fa0/11, Fa0/12
                                                Fa0/13, Fa0/14, Fa0/15, Fa0/16
                                                Fa0/17, Fa0/18, Fa0/19, Fa0/20
                                                Fa0/21, Fa0/22, Fa0/23, Fa0/24
                                                Gig0/1, Gig0/2
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
S5#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S5(config)#vlan 2
S5(config-vlan)#name Accounting-vlan
S5(config-vlan)#exit
S5(config)#interface range fastEthernet 0/13 - 14
S5(config-if-range)#switchport access vlan 2
S5(config-if-range)#switchport mode access
S5(config-if-range)#end
S5#
%SYS-5-CONFIG_I: Configured from console by console
show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                Fa0/5, Fa0/6, Fa0/7, Fa0/8
                                                Fa0/9, Fa0/10, Fa0/11, Fa0/12
                                                Fa0/15, Fa0/16, Fa0/17, Fa0/18
                                                Fa0/19, Fa0/20, Fa0/21, Fa0/22
                                                Fa0/23, Fa0/24, Gig0/1, Gig0/2
2    Accounting-vlan                  active    Fa0/13, Fa0/14
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active
```

- `S5#show vlan brief` displays the current VLAN database: every VLAN's ID, name, status, and assigned ports.
- The first output shows only the VLANs that exist by default: VLAN 1, with every access port still assigned to it, plus the four reserved legacy VLANs (1002-1005) that can't be deleted.
- `S5#conf t` enters global configuration mode.
- `S5(config)#vlan 2` creates VLAN 2 and enters VLAN configuration mode for it.
- `S5(config-vlan)#name Accounting-vlan` assigns a descriptive name to VLAN 2.
- `S5(config-vlan)#exit` returns to global configuration mode.
- `S5(config)#interface range fastEthernet 0/13 - 14` enters interface range configuration mode for Fa0/13 and Fa0/14, using the full interface name.
- `S5(config-if-range)#switchport access vlan 2` assigns both interfaces to VLAN 2 as access ports.
- `S5(config-if-range)#switchport mode access` explicitly sets the port mode to access.
- `S5(config-if-range)#end` returns directly to privileged EXEC mode.
- `%SYS-5-CONFIG_I: Configured from console by console` is IOS's unsolicited log message confirming a change was made from the console.
- The second `show vlan brief` confirms the change: VLAN 2 now appears as `Accounting-vlan` with Fa0/13 and Fa0/14 listed under its Ports column, and those two interfaces no longer appear under VLAN 1.

```text
S5#show running-config 
Building configuration...

Current configuration : 1176 bytes
!
version 15.0
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname S5
!
!
!
!
!
!
spanning-tree mode pvst
spanning-tree extend system-id
!
interface FastEthernet0/1
!
...
interface FastEthernet0/13
 switchport access vlan 2
 switchport mode access
!
interface FastEthernet0/14
 switchport access vlan 2
 switchport mode access
...
interface Vlan1
 no ip address
 shutdown
...
!
end

S5#show vlan id 2

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
2    Accounting-vlan                  active    Fa0/13, Fa0/14

VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
2    enet  100002     1500  -      -      -        -    -        0      0
```

- `S5#show running-config` displays the switch's active configuration. Under `interface FastEthernet0/13` and `interface FastEthernet0/14`, the `switchport access vlan 2` and `switchport mode access` lines confirm both ports are configured for VLAN 2. Every other interface has no VLAN-specific configuration and stays on the default VLAN. The rest of the output is the switch's baseline configuration and isn't specific to VLAN 2.
- `S5#show vlan id 2` displays detail for a single VLAN instead of the whole database. The top table repeats the same summary as `show vlan brief`: VLAN 2 is active and named `Accounting-vlan`, with Fa0/13 and Fa0/14 assigned to it.
- The second table adds detail you rarely need day to day: the media type (`enet` for Ethernet), the VLAN's MTU, and legacy Token Ring/FDDI fields that only apply to VLANs 1002-1005.

### Configure (short)

You don't have to create and name a VLAN before assigning ports to it. Run `switchport access vlan` *id* against a VLAN ID that doesn't exist yet, and IOS creates it for you, using a default name instead of one you chose.

```text
S5#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S5(config)#interface range fastEthernet 0/15 - 16
S5(config-if-range)#switchport access vlan 3
% Access VLAN does not exist. Creating vlan 3
S5(config-if-range)#exit
S5(config)#exit
S5#
%SYS-5-CONFIG_I: Configured from console by console

S5#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                Fa0/5, Fa0/6, Fa0/7, Fa0/8
                                                Fa0/9, Fa0/10, Fa0/11, Fa0/12
                                                Fa0/17, Fa0/18, Fa0/19, Fa0/20
                                                Fa0/21, Fa0/22, Fa0/23, Fa0/24
                                                Gig0/1, Gig0/2
2    Accounting-vlan                  active    Fa0/13, Fa0/14
3    VLAN0003                         active    Fa0/15, Fa0/16
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active
```

- `S5#conf t` enters global configuration mode.
- `S5(config)#interface range fastEthernet 0/15 - 16` enters interface range configuration mode for Fa0/15 and Fa0/16, using the full interface name.
- `S5(config-if-range)#switchport access vlan 3` assigns both interfaces to VLAN 3, which doesn't exist yet.
- `% Access VLAN does not exist. Creating vlan 3` is IOS's confirmation that it created VLAN 3 automatically since no VLAN with that ID existed.
- `S5(config-if-range)#exit` returns to global configuration mode.
- `S5(config)#exit` returns to privileged EXEC mode.
- `%SYS-5-CONFIG_I: Configured from console by console` is IOS's unsolicited log message confirming a change was made from the console.
- `S5#show vlan brief` confirms VLAN 3 exists, with Fa0/15 and Fa0/16 assigned to it. Since no name was given, IOS assigned the default name `VLAN0003`, the VLAN ID zero-padded to four digits.

### VLAN Trunking Protocol (VTP)

**VTP** (VLAN Trunking Protocol) is a Cisco-proprietary protocol that synchronizes the VLAN database across every switch in a VTP domain. Configure a VLAN once on a VTP server switch, and VTP advertises that change over trunk links to every other switch in the domain, so you don't have to create the same VLAN by hand on each one. A switch participates in this domain in one of three modes: server (creates and advertises VLANs), client (accepts advertisements but can't make changes), or transparent (ignores the domain's database).

Because every server-mode switch can add, rename, or delete VLANs for the whole domain, an unexpected VTP change (or a switch with a stale but higher configuration revision number) can wipe out VLANs across every switch at once. Setting a switch to VTP transparent mode disables this behavior. A transparent switch doesn't apply VLAN changes it receives and doesn't advertise its own, though it still forwards VTP advertisements out its trunk ports for other switches to use, and it can keep its own local VLANs independent of the domain.

```text
S5#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S5(config)#vtp mode transparent
Setting device to VTP TRANSPARENT mode.
S5(config)#end
```

- `S5(config)#vtp mode transparent` takes the switch out of the shared VTP domain.
- `Setting device to VTP TRANSPARENT mode.` confirms IOS applied the change. From here, this switch's VLAN configuration is entirely local, and it neither sends nor accepts VLAN database updates.

### Voice VLAN

A **PBX** (private branch exchange) is a private telephone switch that connects an organization's internal phone extensions to each other and out to the public phone network. Traditionally, a PBX ran on its own dedicated telephony wiring and hardware, completely separate from the data network.

**IP telephony** replaces that dedicated wiring by carrying voice as digitized packets over the same Ethernet network used for data. Instead of a hardware PBX, call control runs as software (like Cisco Unified Communications Manager), and phones become ordinary IP endpoints on the network.

A Cisco IP phone accomplishes this with a built-in three-port switch. One internal port connects to the phone's own circuitry that handles the call. A second port, the uplink, runs out over a patch cable and Ethernet UTP cabling back to the wiring closet switch. A third port, on the back of the phone, lets you plug in a PC with a short patch cable. The phone forwards the PC's traffic out its uplink port alongside its own, so a single Ethernet drop at the desk serves both the phone and the PC.

Because phones and PCs share that one drop, put them in separate VLANs: a **voice VLAN** for the phone's traffic, and the regular access VLAN for the PC's. Voice traffic is latency and jitter-sensitive in a way ordinary data traffic isn't, so keeping it on its own VLAN lets the network identify, prioritize, and troubleshoot it separately from everything else on the link.

Splitting the traffic this way means the single link between the switch and the phone has to carry two VLANs at once, even though the switchport is still configured as an access port. The phone handles this by tagging its own voice frames with an **802.1Q** header carrying the voice VLAN ID, while it passes the PC's frames through untagged on the access VLAN. The switch reads the tag to sort voice frames into the voice VLAN, and treats anything untagged as belonging to the access VLAN, all over the same physical cable.

```text
S5#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S5(config)#vlan 10
S5(config-vlan)#vlan 11
S5(config-vlan)#interface range Fast
S5(config-vlan)#interface range Fa0/1 - 4
S5(config-if-range)#switchport mode access
S5(config-if-range)#switchport access vlan 10
S5(config-if-range)#switchport voice vlan 11
S5(config-if-range)#^Z
S5#
%SYS-5-CONFIG_I: Configured from console by console

S5#show interfaces Fas
S5#show interfaces FastEthernet 0/4 switchport
Name: Fa0/4
Switchport: Enabled
Administrative Mode: static access
Operational Mode: down
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: native
Negotiation of Trunking: Off
Access Mode VLAN: 10 (VLAN0010)
Trunking Native Mode VLAN: 1 (default)
Voice VLAN: 11
Administrative private-vlan host-association: none
Administrative private-vlan mapping: none
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk private VLANs: none
Operational private-vlan: none
Trunking VLANs Enabled: All
Pruning VLANs Enabled: 2-1001
Capture Mode Disabled
Capture VLANs Allowed: ALL
Protected: false
Unknown unicast blocked: disabled
Unknown multicast blocked: disabled
Appliance trust: none
```

- `S5(config)#vlan 10` creates VLAN 10, the data VLAN for PCs, and enters VLAN configuration mode for it.
- `S5(config-vlan)#vlan 11` creates VLAN 11, the voice VLAN, directly from VLAN configuration mode. IOS accepts `vlan` *id* from within that mode, not just from global configuration.
- `S5(config-vlan)#interface range Fast` is an incomplete abbreviation. IOS can't tell which interface type you mean from `Fast` alone, so it rejects the command and returns to the same prompt.
- `S5(config-vlan)#interface range Fa0/1 - 4` enters interface range configuration mode for Fa0/1 through Fa0/4. IOS moves directly there from VLAN configuration mode without needing an `exit` first.
- `S5(config-if-range)#switchport mode access` sets these ports to access mode.
- `S5(config-if-range)#switchport access vlan 10` assigns VLAN 10 as the access VLAN, carrying the PC's untagged traffic.
- `S5(config-if-range)#switchport voice vlan 11` assigns VLAN 11 as the voice VLAN, telling any Cisco IP phone connected here to tag its traffic with that VLAN ID.
- `S5#show interfaces Fas` is another incomplete abbreviation, rejected the same way.
- `Administrative Mode: static access` and `Operational Mode: down` confirm the port is configured for access mode, but nothing is currently plugged into it.
- `Access Mode VLAN: 10 (VLAN0010)` confirms the data VLAN. `Voice VLAN: 11` confirms the voice VLAN, both applied to the same physical port.

### Troubleshooting

When a device can't reach something it should, confirm its VLAN configuration before looking further up the network.

#### Confirm correct VLAN access

| Command                                                | Description                                                                                            |
| :------------------------------------------------------ | :--------------------------------------------------------------------------------------------------- |
| `show interfaces` *type* *num* `switchport`              | Confirms a single interface's administrative and operational mode, along with its access and voice VLAN assignments |
| `show interfaces status`                                 | Lists every interface's link status, VLAN, duplex, and speed in one table, so you can spot a port on the wrong VLAN at a glance |
| `show mac address-table`                                 | Confirms which VLAN a device's MAC address was actually learned on, verifying it's really communicating on the VLAN you expect |
| `show vlan`                                              | Displays the full VLAN database with the same detail as `show vlan brief`, plus additional per-VLAN fields |
| `show vlan brief`                                        | Displays every VLAN's ID, name, status, and assigned ports in one summary table                        |
| `show vlan id` *num*                                     | Narrows the VLAN database to a single VLAN                                                             |

#### VLANs active and defined

These commands are the best way to confirm a switch actually knows a particular VLAN exists, and if it does, to determine the VLAN's shutdown state.

```text
S5#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/1, Gi0/2
10   Sales                            active    Fa0/1, Fa0/2
20   Engineering                      suspend   Fa0/3, Fa0/4
30   Marketing                        active
```

The Status column is what you're after:

| Status    | Meaning                                                                                                          |
| :-------- | :----------------------------------------------------------------------------------------------------------------- |
| active    | The VLAN exists and is forwarding traffic normally.                                                             |
| suspend   | The VLAN exists but is suspended, so it stops forwarding traffic until you re-enable it.                        |
| act/unsup | The VLAN is active, but this switch's hardware doesn't support it (for example, a VLAN type the platform can't process). |

To check a single VLAN instead of the whole table, use `show vlan id` *num*.

`show vlan brief` only covers the VLAN itself at Layer 2. It says nothing about that VLAN's SVI (switch virtual interface) at Layer 3, so check that separately with `show ip interface brief`.

```text
S5#show ip interface brief

Interface              IP-Address      OK? Method Status                Protocol
Vlan10                 10.1.10.1       YES manual up                    up
Vlan20                 10.1.20.1       YES manual administratively down  down
```

A Status of `administratively down` means someone ran `shutdown` on that SVI directly. A Status of `down` without "administratively" means the SVI itself hasn't been shut down, but it has no active ports in its VLAN, or the VLAN is suspended. Suspending a VLAN takes its SVI's line protocol down too, even though nobody administratively shut down the SVI.

`show interfaces status` shows the effect at the port level.

```text
S5#show interfaces status

Port      Name               Status       Vlan       Duplex  Speed  Type
Fa0/1                        connected    10         a-full  a-100  10/100BaseTX
Fa0/3                        inactive     20         auto    auto   10/100BaseTX
```

An `inactive` port here isn't administratively shut down. It means the VLAN assigned to the port doesn't exist or has been suspended, leaving the port with nothing to forward to. Cross-check with `show vlan brief` to confirm the VLAN's state, or `show running-config interface` *id* to rule out the port itself being shut down.

Use `show vlan brief` for VLAN (Layer 2) state, `show ip interface brief` for SVI (Layer 3) state, and `show interfaces status` to see the effect at the port level.

#### Mismatched trunk states

The most common trunk misconfiguration is leaving both ends of a link on `switchport mode dynamic auto`. Since `dynamic auto` only waits for the far end to initiate DTP negotiation and never initiates it, two interfaces both left on `dynamic auto` never trunk. Each side is technically willing to trunk, but neither ever asks, so the link falls back to a plain access port even though you intended it as a trunk.

`show interfaces` *type* *num* `switchport` on each end of the suspect link tells you exactly what's happening. Check the Administrative Mode field on both interfaces: if both report `dynamic auto`, that confirms the mismatch.

```text
S5#show interfaces fastEthernet 0/13 switchport
Name: Fa0/13
Switchport: Enabled
Administrative Mode: static access
Operational Mode: down
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: native
Negotiation of Trunking: Off
Access Mode VLAN: 2 (Accounting-vlan)
Trunking Native Mode VLAN: 1 (default)
Voice VLAN: none
Administrative private-vlan host-association: none
Administrative private-vlan mapping: none
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk private VLANs: none
Operational private-vlan: none
Trunking VLANs Enabled: All
Pruning VLANs Enabled: 2-1001
Capture Mode Disabled
Capture VLANs Allowed: ALL
Protected: false
Unknown unicast blocked: disabled
Unknown multicast blocked: disabled
Appliance trust: none
```

This particular example isn't the mismatch. `Administrative Mode: static access` means Fa0/13 was deliberately configured for access mode, on VLAN 2 (Accounting-vlan), and `Negotiation of Trunking: Off` confirms DTP isn't even running on it. `Operational Mode: down` here just means nothing is currently connected.

Contrast that with a genuine `dynamic auto` mismatch: both interfaces would show `Negotiation of Trunking: On`, since DTP is active on a dynamic port, while `Operational Mode` still reads `static access` because neither side ever triggered a successful negotiation. Seeing negotiation turned on but the port still stuck in access mode on both ends of the link is the signature of this specific misconfiguration. Fix it by setting at least one end to `dynamic desirable`, or configure both ends statically with `switchport mode trunk`.

#### Supported VLANs on trunk

A trunk being up doesn't mean every VLAN is actually forwarding across it. You'll want to identify exactly which VLANs a trunk port currently carries, because a VLAN's frames can fail to cross an otherwise healthy trunk for several unrelated reasons: the VLAN was never allowed on that trunk, it's allowed but doesn't actually exist (or is suspended) on this switch, or spanning tree is blocking that VLAN on this port to prevent a loop.

`switchport trunk allowed vlan` controls the first of those. By default, a trunk allows every normal-range VLAN. Narrow that list with `switchport trunk allowed vlan` *vlan-list* to set it outright, `switchport trunk allowed vlan add` *vlan-list* to add to the existing list, `switchport trunk allowed vlan remove` *vlan-list* to remove from it, or `switchport trunk allowed vlan except` *vlan-list* to allow everything except the VLANs you name. Restricting this list to only the VLANs a link actually needs limits how far a broadcast, or an attacker on a hopped VLAN, can reach.

```text
S5#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Gig0/1      desirable    n-802.1q       trunking      1

Port        Vlans allowed on trunk
Gig0/1      1-1005

Port        Vlans allowed and active in management domain
Gig0/1      1,2,3,10,11,20,30,31,40

Port        Vlans in spanning tree forwarding state and not pruned
Gig0/1      1,2,3,10,11,20,30,31,40
```

Each block narrows the previous one, and together they show exactly where a missing VLAN would drop out:

- The first block summarizes the trunk itself: `desirable` and `n-802.1q` show the mode and encapsulation were negotiated rather than statically configured, and `trunking` confirms the link is up as a trunk on native VLAN 1.
- `Vlans allowed on trunk` (`1-1005`) lists every VLAN `switchport trunk allowed vlan` permits, whether or not those VLANs actually exist on this switch.
- `Vlans allowed and active in management domain` narrows that list to VLANs that are both allowed and actually defined and active in the VLAN database. A VLAN allowed here but missing from this line either hasn't been created or is suspended.
- `Vlans in spanning tree forwarding state and not pruned` narrows it once more to the VLANs actually forwarding traffic right now. A VLAN missing from this line, despite appearing in the line above, is either blocked by spanning tree on this port or has been removed by VTP pruning.

#### Mismatched native VLANs

`switchport trunk native vlan` *id* changes a trunk's native VLAN away from the default, VLAN 1. A trunk's native VLAN is the one VLAN whose frames cross the link untagged, without an 802.1Q header identifying which VLAN they belong to.

That's exactly what causes problems when the two ends of a trunk disagree on which VLAN is native. An untagged frame carries no VLAN information of its own, so the receiving switch has no way to know which VLAN the sender intended. It simply assumes the frame belongs to whatever VLAN it has configured as native. If the two switches disagree, a frame sent as native VLAN 10 on one end can arrive and get treated as native VLAN 1 on the other, leaking traffic between VLANs that were supposed to stay isolated.

IOS can detect this on its own. Each switch advertises its configured native VLAN over CDP, and if the two sides don't match, IOS logs a native VLAN mismatch warning instead of silently misforwarding traffic. `show interfaces trunk` also lists each port's Native vlan column, so comparing that value on both ends of a trunk is the fastest way to confirm a mismatch. Fix it by setting `switchport trunk native vlan` to the same ID on both sides.

