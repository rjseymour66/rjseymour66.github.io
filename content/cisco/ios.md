+++
title = 'IOS commands'
date = '2026-07-07T22:31:18-04:00'
weight = 20
draft = false
+++

Cisco IOS organizes configuration into a hierarchy of modes, described below. Use `exit` to move up one level and `end` (or `Ctrl+Z`) to return directly to Privileged EXEC from any configuration mode.

## Modes

Cisco IOS has the following modes:

| Mode | Prompt | Description |
|:---|:---|:---|
| User EXEC mode | `>` | The first mode you reach after connecting to the device. It gives limited, read-only access to device status. |
| Privileged EXEC mode, also called enable mode | `#` | Reached by running `enable` from User EXEC mode. It gives full read access to device state and is required before you can enter configuration mode. |
| Global configuration mode | `(config)#` | Reached by running `configure terminal` from Privileged EXEC mode. Device-wide settings, like the hostname and routing protocols, live here. |
| Interface configuration mode | `(config-if)#` | A sub-mode of global config that controls a specific physical or logical port, for example its speed, IP address, or shutdown state. |
| Line configuration mode | `(config-line)#` | A sub-mode of global config that controls an access line used to connect to the device: the console port, VTY lines (SSH/Telnet), or the AUX port. |
| Router configuration mode | `(config-router)#` | A sub-mode of global config that controls a specific routing protocol process, such as OSPF. |

### Enter mode

Each mode has its own command to enter it, run from the mode one level above.

| Mode | Command |
|:---|:---|
| User EXEC | *(connect)* |
| Privileged EXEC | `enable` |
| Global configuration | `configure terminal` |
| Interface configuration | `interface` *type* *port-number* |
| Line configuration | `line console 0`<br>`line vty` *first* *last*<br>`line aux 0` |
| Router configuration | `router ospf` *process-id* |

### User EXEC

Use this mode for quick, non-privileged checks, like confirming the device is reachable, without risking any configuration changes.

| Command | Description |
|:---|:---|
| `enable` | Enters Privileged EXEC mode |
| `quit` | Closes the current session |

### Privileged EXEC

Use this mode when you need full visibility into device state, such as the running configuration or hardware details, or when you need to run operational commands like `reload` and `copy` before entering configuration mode.

| Command | Description |
|:---|:---|
| `disable` | Returns to User EXEC mode |
| `configure terminal` | Enters global configuration mode |
| `no debug all` | Disables all active debug sessions |
| `undebug all` | Alias for `no debug all` |
| `reload` | Reboots the device |
| `copy running-config startup-config` | Saves running config to NVRAM |
| `copy startup-config running-config` | Loads startup config into running config |
| `show running-config` | Displays the current running configuration |
| `show startup-config` | Displays the saved startup configuration |
| `write erase` | Erases the startup configuration from NVRAM |
| `erase startup-config` | Alias for `write erase` |
| `erase nvram:` | Erases all NVRAM contents |
| `show mac address-table` | Displays the MAC address table (switches only) |
| `quit` | Closes the current session |

### Global configuration

Use this mode to change settings that apply to the whole device, such as the hostname, or as your entry point into a more specific sub-mode like an interface, line, or routing process.

| Command | Description |
|:---|:---|
| `line console 0` | Enters line config mode for the console; subsequent commands apply only to the console line |
| `line vty` *first* *last* | Enters line config mode for remote access lines |
| `interface` *type port-number* | Enters interface config mode for the specified port |
| `router ospf` *process-id* | Enters router config mode for an OSPF process |
| `hostname` *name* | Sets the device hostname |
| `exit` | Moves up one level in the config hierarchy |
| `end` | Returns directly to Privileged EXEC |
| `Ctrl+Z` | Returns directly to Privileged EXEC |

### Interface configuration

Use this mode to change settings scoped to one physical or logical port, such as its IP address, speed, or shutdown state.

| Command | Description |
|:---|:---|
| `speed` *value* | Hard-sets interface speed; disables autonegotiation |
| `exit` | Moves up one level in the config hierarchy |
| `end` | Returns directly to Privileged EXEC |
| `Ctrl+Z` | Returns directly to Privileged EXEC |

### Line configuration

Use this mode to control how administrators connect to the device itself, such as setting console or VTY passwords, not how the device forwards traffic.

| Command | Description |
|:---|:---|
| `login` | Tells IOS to perform simple password checking on the line |
| `password` *password* | Sets the plain-text password for the line |
| `exit` | Moves up one level in the config hierarchy |
| `end` | Returns directly to Privileged EXEC |
| `Ctrl+Z` | Returns directly to Privileged EXEC |

### Router configuration

Use this mode to enable or tune a specific routing protocol instance, such as advertising networks into OSPF.

| Command | Description |
|:---|:---|
| `exit` | Moves up one level in the config hierarchy |
| `end` | Returns directly to Privileged EXEC |
| `Ctrl+Z` | Returns directly to Privileged EXEC |

## Help commands

IOS provides context-sensitive help so you don't have to memorize every command and keyword.

| Syntax | Description |
|:---|:---|
| `?` | Lists every command available in the current mode |
| *command* ` ?` | Lists the keywords or arguments that *command* accepts |
| *com*`?` | Lists every command that starts with *com* (no space before `?`) |
| *command* *parm*`?` | Lists the keywords for *command* that start with *parm* (no space before `?`) |
| *command* *parm*`Tab` | Completes *parm* if it uniquely matches one keyword |
| *command* *parm1* ` ?` | Lists the keywords available after *parm1* |
