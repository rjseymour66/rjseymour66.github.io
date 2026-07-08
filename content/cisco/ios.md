+++
title = 'IOS commands'
date = '2026-07-07T22:31:18-04:00'
weight = 20
draft = false
+++

Cisco IOS organizes configuration into a hierarchy of modes. *Privileged EXEC* mode (indicated by `#`) is the entry point for viewing device state and entering configuration. Running `configure terminal` drops you into *global configuration* mode, where device-wide settings like hostname and routing protocols live. From global config, you enter sub-modes to configure specific resources. *Interface configuration* mode controls physical and logical ports, where you set properties like speed, IP addresses, and shutdown state. *Line configuration* mode controls the access lines used to connect to the device, including the console port, VTY (SSH/Telnet), and AUX port. Use `exit` to move up one level and `end` to return directly to privileged EXEC.

1. *Privileged EXEC* (`#`): view and manage device state, enter configuration
2. *Global configuration* (`(config)#`): device-wide settings: hostname, routing, AAA, VLANs
3. *Interface configuration* (`(config-if)#`): per-port settings: IP, speed, shutdown state
4. *Line configuration* (`(config-line)#`): console, VTY, and AUX access settings
5. *Router configuration* (`(config-router)#`): routing protocol settings: OSPF, EIGRP, BGP

## Configuration commands

| Command | Mode | Description |
|:---|:---|:---|
| `line console 0` | Global config | Enters line config mode for the physical console port |
| `login` | Line config | Requires password authentication on the line |
| `password` *password* | Line config | Sets the plain-text password for the line |
| `interface` *type port-number* | Global config | Enters interface config mode for the specified port |
| `speed` *value* | Interface config | Hard-sets interface speed; disables autonegotiation |
| `hostname` *name* | Global config | Sets the device hostname |
| `exit` | Any | Moves up one level in the config hierarchy |
| `end` | Any | Returns directly to privileged EXEC from any config mode |
| `Ctrl+Z` | Any | Returns directly to privileged EXEC from any config mode |

## EXEC commands

EXEC mode is the command-line layer above configuration. *User EXEC* mode (indicated by `>`) provides limited read-only access and is the first mode you reach after connecting. Running `enable` elevates you to *Privileged EXEC* mode (indicated by `#`), where you can view device state, manage configuration files, reload the device, and control the session. Unlike configuration commands, EXEC commands do not modify the running configuration.

| Command | Mode | Description |
|:---|:---|:---|
| `no debug all` | Privileged EXEC | Disables all active debug sessions |
| `undebug all` | Privileged EXEC | Alias for `no debug all` |
| `reload` | Privileged EXEC | Reboots the device |
| `copy running-config startup-config` | Privileged EXEC | Saves running config to NVRAM |
| `copy startup-config running-config` | Privileged EXEC | Loads startup config into running config |
| `show running-config` | Privileged EXEC | Displays the current running configuration |
| `write erase` | Privileged EXEC | Erases the startup configuration from NVRAM |
| `erase startup-config` | Privileged EXEC | Alias for `write erase` |
| `erase nvram:` | Privileged EXEC | Erases all NVRAM contents |
| `quit` | Any EXEC | Closes the current session |
| `show startup-config` | Privileged EXEC | Displays the saved startup configuration |
| `enable` | User EXEC | Enters Privileged EXEC mode |
| `disable` | Privileged EXEC | Returns to User EXEC mode |
| `configure terminal` | Privileged EXEC | Enters global configuration mode |
| `show mac address-table` | Privileged EXEC | Displays the MAC address table (switches only) |
