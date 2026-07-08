+++
title = 'Switches'
date = '2026-07-07T22:58:21-04:00'
weight = 30
draft = false
+++

## Ethernet LAN

An *Ethernet LAN* is a group of devices connected at Layer 2, sharing the same broadcast domain. A LAN switch forwards frames based on destination MAC addresses. When a frame arrives, the switch records the source MAC address and the ingress port, building a *MAC address table*. When forwarding, the switch looks up the destination MAC. If it finds a match, it forwards the frame out only that port (*filtering*). If no match exists, it floods the frame out all ports except the one it arrived on (*unknown unicast flooding*).

## EXEC commands

Use these Privileged EXEC commands to verify switch state without modifying configuration. They cover the MAC address table, interface status, SSH settings, DHCP leases, running configuration, and session history. Run them during initial setup to confirm your configuration took effect, and during troubleshooting to isolate where a problem exists.

### MAC address table

| Command | Mode | Description |
|:---|:---|:---|
| `show mac address-table` | Privileged EXEC | Displays all MAC address table entries |
| `show mac address-table dynamic` | Privileged EXEC | Displays only dynamically learned entries |
| `show mac address-table dynamic vlan` *vlan-id* | Privileged EXEC | Filters dynamic entries by VLAN |
| `show mac address-table dynamic address` *mac-addr* | Privileged EXEC | Filters dynamic entries by MAC address |
| `show mac address-table dynamic interface` *interface-id* | Privileged EXEC | Filters dynamic entries by interface |
| `show mac address-table count` | Privileged EXEC | Displays the number of entries per VLAN |
| `show mac address-table aging-time` | Privileged EXEC | Displays the MAC address aging timer |
| `clear mac address-table dynamic` | Privileged EXEC | Clears dynamic entries; optionally filter by `vlan` *vlan-number*, `interface` *interface-id*, or `address` *mac-addr* |

### Interfaces

| Command | Mode | Description |
|:---|:---|:---|
| `show interfaces` *id* `counters` | Privileged EXEC | Displays traffic counters for an interface |
| `show interfaces status` | Privileged EXEC | Displays status, duplex, and speed for all interfaces |
| `show interfaces vlan` *number* | Privileged EXEC | Displays status and statistics for a VLAN SVI |
| `show interfaces [` *type* *number* `] status` | Privileged EXEC | Displays status for all interfaces or a specific one |
| `show interfaces [` *type* *number* `]` | Privileged EXEC | Displays detailed statistics for all interfaces or a specific one |
| `show interfaces description` | Privileged EXEC | Displays interface descriptions and operational status |

### Running configuration

| Command | Mode | Description |
|:---|:---|:---|
| `show running-config` | Privileged EXEC | Displays the current running configuration |
| `show running-config \| begin line vty` | Privileged EXEC | Displays running config starting from the VTY line section |
| `show running-config interface` *type* *number* | Privileged EXEC | Displays running config for a specific interface |
| `show running-config all` | Privileged EXEC | Displays running config including default values |

### SSH and security

| Command | Mode | Description |
|:---|:---|:---|
| `show crypto key mypubkey rsa` | Privileged EXEC | Displays the device's RSA public key |
| `show ip ssh` | Privileged EXEC | Displays SSH version and configuration settings |
| `show ssh` | Privileged EXEC | Displays active SSH sessions |

### IP and DHCP

| Command | Mode | Description |
|:---|:---|:---|
| `show dhcp lease` | Privileged EXEC | Displays DHCP lease information obtained by the switch |
| `show ip default-gateway` | Privileged EXEC | Displays the configured default gateway |

### Session

| Command | Mode | Description |
|:---|:---|:---|
| `terminal history size` *x* | Privileged EXEC | Temporarily sets the command history buffer size for the session |
| `show history` | Privileged EXEC | Displays the command history for the current session |

## Interface configuration

*Interface configuration* mode controls the behavior of individual switch ports. Use `interface` to enter a single port or `interface range` to apply the same settings to multiple ports at once. From there, you set *speed*, *duplex*, *shutdown state*, and a *description* to document port purpose. Prefixing any command with `no` returns that setting to its default. Use `default interface` to reset all settings on a port at once.

| Command | Mode | Description |
|:---|:---|:---|
| `interface` *type* *port-number* | Global config | Enters interface config mode for a specific port |
| `interface range` *type* *port-number* `-` *end-port-number* | Global config | Enters interface config mode for a range of ports |
| `shutdown` | Interface config | Administratively disables the interface |
| `no shutdown` | Interface config | Enables an administratively disabled interface |
| `speed [10\|100\|1000\|auto]` | Interface config | Sets the interface speed or enables autonegotiation |
| `duplex [auto\|full\|half]` | Interface config | Sets the duplex mode or enables autonegotiation |
| `description` *text* | Interface config | Adds a text description to the interface |
| `no speed` | Interface config | Removes the configured speed and returns to default |
| `no duplex` | Interface config | Removes the configured duplex and returns to default |
| `no description` | Interface config | Removes the interface description |
| `default interface` *interface-id* | Global config | Resets all interface settings to defaults |
| `[no] mdix auto` | Interface config | Enables or disables automatic MDI-X crossover detection |

## Login Security commands

Unsecured console and VTY lines leave the device open to anyone who can reach it physically or over the network. IOS offers two authentication models: *line password* authentication (`login` + `password`) applies a single shared password to the line, while *local authentication* (`login local` + `username secret`) requires per-user credentials stored on the device. For remote access, always use *SSH* instead of Telnet. Telnet sends credentials in plaintext; SSH encrypts the session.

| Command                                    | Mode          | Description                                               |
| :----------------------------------------- | :------------ | :-------------------------------------------------------- |
| `line console 0`                           | Global config | Enters line config mode for the console port              |
| `line vty` *first-vty* *last-vty*          | Global config | Enters line config mode for VTY lines (remote access)     |
| `login`                                    | Line config   | Enables line password authentication                      |
| `password` *password*                      | Line config   | Sets the plain-text line password                         |
| `login local`                              | Line config   | Enables local user account authentication                 |
| `username` *name* `secret` *password*      | Global config | Creates a local user with a hashed password               |
| `crypto key generate rsa modulus` *bits*   | Global config | Generates RSA keys for SSH (512-2048 bits)                |
| `transport input [telnet\|ssh\|all\|none]` | Line config   | Restricts allowed remote access protocols on the line     |
| `ip domain name` *fqdn*                    | Global config | Sets the domain name, required for SSH key generation     |
| `hostname` *name*                          | Global config | Sets the device hostname, required for SSH key generation |
| `ip ssh version 2`                         | Global config | Forces SSHv2 only                                         |

## IPv4 configuration

| Command                                      | Mode             | Description                                             |
| :------------------------------------------- | :--------------- | :------------------------------------------------------ |
| `interface vlan` *number*                    | Global config    | Enters interface config for the management VLAN (SVI)   |
| `ip address` *ip-addr* *subnet-mask*         | Interface config | Assigns a static IP address                             |
| `ip address dhcp`                            | Interface config | Configures the interface to obtain an address from DHCP |
| `ip default-gateway` *addr*                  | Global config    | Sets the default gateway for management traffic         |
| `ip name-server` *server-ip-1* *server-ip-2* | Global config    | Configures DNS server addresses                         |

## Other configuration

| Command                    | Mode          | Description                                           |
| :------------------------- | :------------ | :---------------------------------------------------- |
| `hostname` *name*          | Global config | Sets the device hostname                              |
| `enable secret` *password* | Global config | Sets a hashed password for privileged EXEC access     |
| `history size` *length*    | Line config   | Sets the command history buffer size for the line     |
| `logging synchronous`      | Line config   | Prevents log messages from interrupting command input |
| `[no] logging console`     | Global config | Enables or disables log messages to the console       |

