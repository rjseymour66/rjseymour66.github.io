+++
title = 'Switches'
date = '2026-07-07T22:58:21-04:00'
weight = 30
draft = false
+++

## Ethernet LAN

An *Ethernet LAN* is a group of devices connected at Layer 2, sharing the same broadcast domain. A LAN switch forwards frames based on destination MAC addresses. When a frame arrives, the switch records the source MAC address and the ingress port, building a *MAC address table*. When forwarding, the switch looks up the destination MAC. If it finds a match, it forwards the frame out only that port (*filtering*). If no match exists, it floods the frame out all ports except the one it arrived on (*unknown unicast flooding*).

## Interface configuration

*Interface configuration* mode controls the behavior of individual switch ports. Use `interface` to enter a single port or `interface range` to apply the same settings to multiple ports at once. From there, you set *speed*, *duplex*, *shutdown state*, and a *description* to document port purpose. Prefixing any command with `no` returns that setting to its default. Use `default interface` to reset all settings on a port at once.

| Command                                                      | Mode             | Description                                             |
| :----------------------------------------------------------- | :--------------- | :------------------------------------------------------ |
| `interface` *type* *port-number*                             | Global config    | Enters interface config mode for a specific port        |
| `interface range` *type* *port-number* `-` *end-port-number* | Global config    | Enters interface config mode for a range of ports       |
| `shutdown`                                                   | Interface config | Administratively disables the interface                 |
| `no shutdown`                                                | Interface config | Enables an administratively disabled interface          |
| `speed [10\|100\|1000\|auto]`                                | Interface config | Sets the interface speed or enables autonegotiation     |
| `duplex [auto\|full\|half]`                                  | Interface config | Sets the duplex mode or enables autonegotiation         |
| `description` *text*                                         | Interface config | Adds a text description to the interface                |
| `no speed`                                                   | Interface config | Removes the configured speed and returns to default     |
| `no duplex`                                                  | Interface config | Removes the configured duplex and returns to default    |
| `no description`                                             | Interface config | Removes the interface description                       |
| `default interface` *interface-id*                           | Global config    | Resets all interface settings to defaults               |
| `[no] mdix auto`                                             | Interface config | Enables or disables automatic MDI-X crossover detection |
| `show interfaces status`                                     | Privileged EXEC  | Displays status, duplex, and speed for all interfaces    |
| `show interfaces` *id*                                       | Privileged EXEC  | Displays detailed statistics for a specific interface    |
| `show interfaces` *id* `counters`                             | Privileged EXEC  | Displays traffic counters for an interface               |
| `show interfaces vlan` *number*                               | Privileged EXEC  | Displays status and statistics for a VLAN SVI            |
| `show interfaces [` *type* *number* `] status`                | Privileged EXEC  | Displays status for all interfaces or a specific one     |
| `show interfaces [` *type* *number* `]`                       | Privileged EXEC  | Displays detailed statistics for all interfaces or a specific one |
| `show interfaces description`                                 | Privileged EXEC  | Displays interface descriptions and operational status   |

## MAC address table

| Command                                                   | Description                                                                                                            |
| :-------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------- |
| `show mac address-table`                                  | Displays all MAC address table entries                                                                                 |
| `show mac address-table dynamic`                          | Displays only dynamically learned entries                                                                              |
| `show mac address-table dynamic vlan` *vlan-id*           | Filters dynamic entries by VLAN                                                                                        |
| `show mac address-table dynamic address` *mac-addr*       | Filters dynamic entries by MAC address                                                                                 |
| `show mac address-table dynamic interface` *interface-id* | Filters dynamic entries by interface                                                                                   |
| `show mac address-table count`                            | Displays the number of entries per VLAN                                                                                |
| `show mac address-table aging-time`                       | Displays the MAC address aging timer                                                                                   |
| `clear mac address-table dynamic`                         | Clears dynamic entries; optionally filter by `vlan` *vlan-number*, `interface` *interface-id*, or `address` *mac-addr* |

## Configuration files

A switch stores configuration and code across four types of memory, each with different persistence characteristics.

IOS works with two configuration files. The *running configuration* is the configuration currently active on the device, held in RAM. The *startup configuration* is the configuration saved in NVRAM that IOS loads into RAM as the running-config every time the device boots. Until you save your changes, they exist only in the running-config.

| Memory | Volatile?                               | Stores                                                     | Role in configuration                                 |
| :----- | :-------------------------------------- | :--------------------------------------------------------- | :---------------------------------------------------- |
| RAM    | Yes                                     | Running configuration, routing table, ARP cache            | Holds the active config; lost on reload unless saved  |
| NVRAM  | No                                      | Startup configuration                                      | Loaded into RAM as the running-config at boot         |
| Flash  | No                                      | IOS image (and often the startup-config and license files) | Supplies the IOS image loaded into RAM at boot        |
| ROM    | No (not rewritable in normal operation) | Bootstrap program, ROM Monitor (ROMmon), mini-IOS          | Runs POST and loads the IOS image from Flash into RAM |

*RAM* holds the running-config. Because RAM is volatile, any change you make in configuration mode exists only in RAM until you save it. Run `show running-config` to view it.

*NVRAM* holds the startup-config. Running `copy running-config startup-config` copies RAM's running-config into NVRAM, committing your changes so they survive a reload. If you reload without saving, IOS reloads NVRAM's startup-config into RAM, discarding any unsaved changes.

*Flash memory* holds the IOS image itself, the switch's file system for boot images and, on some platforms, backup configuration and license files. Use `dir flash:` to view its contents.

*ROM* holds the bootstrap program and ROMmon, the low-level code that runs power-on self-test (POST), locates the IOS image in Flash, and loads it into RAM before handing off to the startup-config in NVRAM.

### Startup configuration

| Command                              | Mode            | Description                                                           |
| :----------------------------------- | :-------------- | :-------------------------------------------------------------------- |
| `show startup-config`                | Privileged EXEC | Displays the saved startup configuration                              |
| `copy running-config startup-config` | Privileged EXEC | Saves the running configuration to NVRAM as the startup configuration |
| `copy startup-config running-config` | Privileged EXEC | Merges the startup configuration into the running configuration       |
| `erase startup-config`               | Privileged EXEC | Erases the startup configuration from NVRAM                           |
| `write erase`                        | Privileged EXEC | Alias for `erase startup-config`                                      |
| `erase nvram:`                       | Privileged EXEC | Erases all NVRAM contents                                             |

### Running configuration

| Command                                         | Mode            | Description                                                |
| :---------------------------------------------- | :-------------- | :--------------------------------------------------------- |
| `show running-config`                           | Privileged EXEC | Displays the current running configuration                 |
| `show running-config \| begin line vty`         | Privileged EXEC | Displays running config starting from the VTY line section |
| `show running-config interface` *type* *number* | Privileged EXEC | Displays running config for a specific interface           |
| `show running-config all`                       | Privileged EXEC | Displays running config including default values           |

## Setting passwords

Unsecured console and VTY lines leave the device open to anyone who can reach it physically or over the network. IOS offers two authentication models: *line password* authentication (`login` + `password`) applies a single shared password to the line, while *local authentication* (`login local` + `username secret`) requires per-user credentials stored on the device. For remote access, always use *SSH* instead of Telnet. Telnet sends credentials in plaintext; SSH encrypts the session.

### Enable secret

Set a hashed password on Privileged EXEC mode so only authorized users can move past User EXEC and reach device configuration.

```bash
configure terminal
enable secret <password>
```

### Console line password

Require a password on the console port so physical access to the device isn't enough to reach User EXEC mode on its own.

```bash
line console 0
password <password>
login
exit
```

### VTY line password

Require a password on the VTY lines so remote sessions can't reach User EXEC mode without authenticating. `line vty 0 15` covers all 16 VTY lines, and `transport input all` permits both Telnet and SSH; restrict it to `ssh` in production to avoid sending credentials in plaintext.

```bash
line vty 0 15
password <password>
login
transport input all
end
```

### Username password

Create a local user account with a hashed password, so you have credentials ready before you enable local authentication on a line.

```bash
configure terminal
username <name> secret <password>
```

### Local username login

Require per-user credentials instead of one shared line password, so each administrator authenticates with their own account and you get individual accountability for access. Start by creating the local user account.

```bash
configure terminal
username <name> secret <password>
```

#### Console

Configure the console line to authenticate against the local user database instead of a shared line password. `line con 0` is a valid abbreviation for `line console 0`, entered from global configuration mode. Once `login local` is set, the line no longer authenticates with a shared password, so `no password` clears out any password left over from a previous configuration.

```bash
line con 0
login local
no password
```

#### Telnet (VTY)

Configure the VTY lines the same way, so remote sessions authenticate against local user accounts instead of a shared line password.

```bash
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
| `transport input telnet ssh` | Permits both Telnet and SSH (equivalent to `all`) |
| `transport input none`       | Disables all remote access protocols on the line  |
| `transport input telnet`     | Permits Telnet only                               |
| `transport input ssh`        | Permits SSH only                                  |

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

## SSH and security

SSH requires a hostname, a domain name, and an RSA key pair before it can run. The example below sets the hostname, sets the domain name, generates the RSA keys, and forces SSH version 2.

```bash
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
- `ip domain name example.com` sets the domain name; combined with the hostname, it forms the key name `SW3.example.com`.
- `crypto key generate rsa` generates the RSA key pair SSH uses to encrypt sessions. IOS prompts for a modulus size (in bits) since none was given on the command line; larger moduli are more secure but take longer to generate. 1024 bits is the practical minimum for SSHv2.
- `ip ssh version 2` forces the device to use SSHv2 only. SSHv1 has known vulnerabilities and shouldn't be enabled.

Enabling SSH only makes the device *capable* of running the protocol; it doesn't make the VTY lines accept it. Apply this to actually allow SSH sessions in:

```bash
SW3(config)#line vty 0 15
SW3(config-line)#login local
SW3(config-line)#transport input all
SW3(config-line)#exit
SW3(config)#username ryan secret password
```

`line vty 0 15` enters line config mode for all 16 VTY lines. `login local` authenticates incoming sessions against the local username database instead of a shared line password, so each admin needs their own account (see [Local username login](#local-username-login)). `transport input all` permits both Telnet and SSH on the line; without it, or with `transport input none`, the VTY lines reject incoming connections regardless of protocol. `exit` returns to global configuration mode.

Without this step, the `crypto key`/`ip ssh version 2` work only prepares the device to speak SSH. The VTY lines still control whether any remote session, SSH or Telnet, gets in at all, and how it authenticates.

`username ryan secret password` creates the local account that `login local` checks against. It has to exist before anyone can authenticate; without it, `login local` rejects every login attempt because there are no local accounts to match against. `secret` hashes the password, so store it as `secret`, never `password`, for any account used for login.

```bash
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

```bash
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
- `Enter configuration commands, one per line. End with CNTL/Z.` is IOS's standard confirmation that you're now in configuration mode; you don't type it.
- `SW3(config)#interface vlan 1` enters interface configuration mode for VLAN 1's SVI, the logical Layer 3 interface used to manage the switch.
- `SW3(config-if)#ip address 192.168.1.200 255.255.255.0` assigns that interface a static management address.
- `SW3(config-if)#no shutdown` administratively enables the interface, since VLAN SVIs are down by default.
- `%LINK-3-UPDOWN: Interface Vlan1, changed state to down` is unsolicited console output confirming the link-layer state change, not a command.
- `%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up` is unsolicited console output confirming the line protocol came up once at least one active port belongs to VLAN 1.
- `exit` returns to global configuration mode.
- `SW3(config)#ip default-gateway 192.168.1.1` sets the gateway the switch itself uses for its own management traffic.

### DHCP

Configure the management interface to obtain its IP address automatically from a DHCP server instead of a static assignment.

```bash
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
- `Enter configuration commands, one per line. End with CNTL/Z.` is IOS's standard confirmation that you're now in configuration mode; you don't type it.
- `SW3(config)#interface vlan 1` enters interface configuration mode for VLAN 1's SVI.
- `SW3(config-if)#ip address dhcp` configures the interface to request an IPv4 address from a DHCP server instead of using a static address.
- `SW3(config-if)#no shutdown` administratively enables the interface so it can send the DHCP request.
- `SW3#show dhcp lease` displays the client's current DHCP lease. This output shows `state: Selecting` with every address field at `0.0.0.0`, meaning the switch had just sent a `DHCPDISCOVER` and hadn't received an offer yet; a completed lease shows the assigned address, lease server, and non-zero lease timers.
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

| Command                     | Description                                                       |
| :--------------------------- | :------------------------------------------------------------------ |
| `show history`               | Displays the command history for the current session                |
| `terminal history size` *x* | Temporarily sets the command history buffer size for the current session |
| `history size` *x*          | Sets the command history buffer size for a line, persisting across sessions |

## Web UI

The web UI provides a browser-based alternative to the CLI for basic device management. Disable the plaintext HTTP server in favor of HTTPS, and require the same local-authentication model used for CLI logins. Cisco recommends disabling the HTTP server, since it transmits credentials and management traffic in plaintext.

| Command                                              | Description                                                            |
| :--------------------------------------------------- | :--------------------------------------------------------------------- |
| `no ip http server`                                  | Disables the insecure HTTP web server                                  |
| `ip http secure-server`                              | Enables the HTTPS web server                                           |
| `ip http authentication local`                       | Requires web UI logins to authenticate against the local user database |
| `username` *name* `privilege 15 password` *password* | Creates a local user with full (level 15) privilege for web UI login   |

## Other configuration

| Command                    | Mode          | Description                                           |
| :------------------------- | :------------ | :---------------------------------------------------- |
| `hostname` *name*          | Global config | Sets the device hostname                              |
| `enable secret` *password* | Global config | Sets a hashed password for privileged EXEC access     |
| `logging synchronous`      | Line config   | Prevents log messages from interrupting command input |
| `[no] logging console`     | Global config | Enables or disables log messages to the console       |

