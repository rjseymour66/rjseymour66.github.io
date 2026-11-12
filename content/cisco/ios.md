+++
title = 'IOS CLI'
date = '2026-07-07T22:31:18-04:00'
weight = 10
draft = false
+++

Cisco IOS organizes configuration into a hierarchy of modes, described below. Use `exit` to move up one level and `end` (or `Ctrl+Z`) to return directly to Privileged EXEC from any configuration mode.

## Modes

Cisco IOS has the following modes:

| Mode                                          | Prompt             | Description                                                                                                                                         |
| :-------------------------------------------- | :----------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------- |
| User EXEC mode                                | `>`                | The first mode you reach after connecting to the device. It gives limited, read-only access to device status.                                       |
| Privileged EXEC mode, also called enable mode | `#`                | Reached by running `enable` from User EXEC mode. It gives full read access to device state and is required before you can enter configuration mode. |
| Global configuration mode                     | `(config)#`        | Reached by running `configure terminal` from Privileged EXEC mode. Device-wide settings, like the hostname and routing protocols, live here.        |
| Interface configuration mode                  | `(config-if)#`     | A sub-mode of global config that controls a specific physical or logical port, for example its speed, IP address, or shutdown state.                |
| Line configuration mode                       | `(config-line)#`   | A sub-mode of global config that controls an access line used to connect to the device: the console port, VTY lines (SSH/Telnet), or the AUX port.  |
| Router configuration mode                     | `(config-router)#` | A sub-mode of global config that controls a specific routing protocol process, such as OSPF.                                                        |

### Enter mode

Each mode has its own command to enter it, run from the mode one level above.

| Mode                    | Command                                                       |
| :---------------------- | :------------------------------------------------------------ |
| User EXEC               | *(connect)*                                                   |
| Privileged EXEC         | `enable`                                                      |
| Global configuration    | `configure terminal`                                          |
| Interface configuration | `interface` *type* *port-number*                              |
| Line configuration      | `line console 0`<br>`line vty` *first* *last*<br>`line aux 0` |
| Router configuration    | `router ospf` *process-id*                                    |

### User EXEC

Use this mode for quick, non-privileged checks, like confirming the device is reachable, without risking any configuration changes.

User EXEC is the least-privileged mode in the IOS command hierarchy. It lets you run basic commands that display the device's configuration and status.

{{< admonition "CCNA tip" tip >}}
Learning the available `show` commands and how to interpret their output is a major part of studying for the CCNA.
{{< /admonition >}}

| Command  | Description                 |
| :------- | :-------------------------- |
| `enable` | Enters Privileged EXEC mode |
| `quit`   | Closes the current session  |

### Privileged EXEC

Use this mode when you need full visibility into device state, such as the running configuration or hardware details, or when you need to run operational commands like `reload` and `copy` before entering configuration mode.

- Privileged EXEC gives unlimited access to the available `show` commands and to many other commands that control device features.
- Privileged EXEC also lets you enter global configuration mode and change the configuration. Protect it with a password. For details, see [Privileged EXEC passwords](#privileged-exec-passwords).
- Neither EXEC mode changes the configuration. Both modes only let you view the device's status and configuration and run operational commands, such as restarting the device, saving the configuration, and moving and deleting files.

| Command                              | Description                                    |
| :----------------------------------- | :--------------------------------------------- |
| `disable`                            | Returns to User EXEC mode                      |
| `configure terminal`                 | Enters global configuration mode               |
| `no debug all`                       | Disables all active debug sessions             |
| `undebug all`                        | Alias for `no debug all`                       |
| `reload`                             | Reboots the device                             |
| `copy running-config startup-config` | Saves running config to NVRAM                  |
| `write`                              | Alias for `copy running-config startup-config` |
| `write memory`                       | Alias for `copy running-config startup-config` |
| `copy startup-config running-config` | Loads startup config into running config       |
| `show running-config`                | Displays the current running configuration     |
| `show startup-config`                | Displays the saved startup configuration       |
| `write erase`                        | Erases the startup configuration from NVRAM    |
| `erase startup-config`               | Alias for `write erase`                        |
| `erase nvram:`                       | Erases all NVRAM contents                      |
| `show mac address-table`             | Displays the MAC address table (switches only) |
| `quit`                               | Closes the current session                     |

### Global configuration

Use this mode to change settings that apply to the whole device, such as the hostname and passwords, or as your entry point into a more specific sub-mode like an interface, line, or routing process.

- Configure a unique hostname on each device so the devices are easy to identify.
- To undo a configuration command, put `no` in front of it.
- To run an EXEC mode command from a configuration mode, put `do` in front of it. You don't have to return to Privileged EXEC first.

| Command                        | Description                                                                                 |
| :----------------------------- | :------------------------------------------------------------------------------------------ |
| `line console 0`               | Enters line config mode for the console; subsequent commands apply only to the console line |
| `line vty` *first* *last*      | Enters line config mode for remote access lines                                             |
| `interface` *type port-number* | Enters interface config mode for the specified port                                         |
| `router ospf` *process-id*     | Enters router config mode for an OSPF process                                               |
| `hostname` *name*              | Sets the device hostname                                                                    |
| `enable password` *password*   | Sets the enable password (legacy)                                                           |
| `enable secret` *password*     | Sets the enable secret, stored as a hash                                                    |
| `service password-encryption`  | Encrypts current and future passwords with type 7 encryption                                |
| `no` *command*                 | Undoes a configuration command                                                              |
| `do` *command*                 | Runs an EXEC mode command from a configuration mode                                         |
| `exit`                         | Moves up one level in the config hierarchy                                                  |
| `end`                          | Returns directly to Privileged EXEC                                                         |
| `Ctrl+Z`                       | Runs the partially typed command, then returns to Privileged EXEC                           |
| `Ctrl+C`                       | Returns to Privileged EXEC without running the partially typed command                      |

{{< admonition "Ctrl+Z vs. Ctrl+C" warning >}}
If you press `Ctrl+Z` in the middle of typing a command, the device runs the typed command before it returns to Privileged EXEC. This is equivalent to pressing Enter and then running `end`. `Ctrl+C` doesn't run the command. It only returns you to Privileged EXEC.
{{< /admonition >}}

### Interface configuration

Use this mode to change settings scoped to one physical or logical port, such as its IP address, speed, or shutdown state.

| Command         | Description                                         |
| :-------------- | :-------------------------------------------------- |
| `speed` *value* | Hard-sets interface speed; disables autonegotiation |
| `exit`          | Moves up one level in the config hierarchy          |
| `end`           | Returns directly to Privileged EXEC                 |
| `Ctrl+Z`        | Returns directly to Privileged EXEC                 |

### Line configuration

Use this mode to control how administrators connect to the device itself, such as setting console or VTY passwords, not how the device forwards traffic.

| Command               | Description                                               |
| :-------------------- | :-------------------------------------------------------- |
| `login`               | Tells IOS to perform simple password checking on the line |
| `password` *password* | Sets the plain-text password for the line                 |
| `exit`                | Moves up one level in the config hierarchy                |
| `end`                 | Returns directly to Privileged EXEC                       |
| `Ctrl+Z`              | Returns directly to Privileged EXEC                       |

### Router configuration

Use this mode to enable or tune a specific routing protocol instance, such as advertising networks into OSPF.

| Command  | Description                                |
| :------- | :----------------------------------------- |
| `exit`   | Moves up one level in the config hierarchy |
| `end`    | Returns directly to Privileged EXEC        |
| `Ctrl+Z` | Returns directly to Privileged EXEC        |

## Help commands

IOS provides context-sensitive help so you don't have to memorize every command and keyword.

| Syntax                 | Description                                                                   |
| :--------------------- | :---------------------------------------------------------------------------- |
| `?`                    | Lists every command available in the current EXEC or configuration mode       |
| *command* ` ?`         | Lists the keywords or arguments that *command* accepts                        |
| *com*`?`               | Lists every command that starts with *com* (no space before `?`)              |
| *com*`Tab`             | Completes the command if only one command starts with *com*                   |
| *com*`Enter`           | Runs the command if only one command starts with *com*                        |
| *command* *parm*`?`    | Lists the keywords for *command* that start with *parm* (no space before `?`) |
| *command* *parm*`Tab`  | Completes *parm* if it uniquely matches one keyword                           |
| *command* *parm1* ` ?` | Lists the keywords available after *parm1*                                    |

`<cr>` in help output means carriage return, which refers to the Enter key. It means you can press Enter to run the command as typed. For example, `show clock` is a valid command on its own, even though the `detail` keyword is available.

## Keyboard shortcuts

| Shortcut | Description                                              |
| :------- | :------------------------------------------------------- |
| `Ctrl+A` | Moves the cursor to the beginning of the current command |
| `Ctrl+E` | Moves the cursor to the end of the current command       |
| `Ctrl+U` | Deletes all characters to the left of the cursor         |

## Filter show output

A pipe (`|`) filters the output of a `show` command. The `include` keyword after a pipe shows only the lines that include the characters you specify:

```bash
R1# show running-config | include enable
enable password ccna
```

## Device access

You can access the IOS CLI in two ways:

- **Console:** Connect a PC or laptop to the device's console port with a console cable.
- **Network:** Connect to the device over the network with a protocol like Telnet or Secure Shell (SSH).

### Console ports

The console port type depends on the device model. USB Mini-B and RJ45 console ports are common across many Cisco router and switch models.

To connect to an RJ45 console port, you must use a **rollover cable**. A rollover cable reverses the pin order from one end to the other:

| End A pin | End B pin |
| :-------- | :-------- |
| 1         | 8         |
| 2         | 7         |
| 3         | 6         |
| 4         | 5         |
| 5         | 4         |
| 6         | 3         |
| 7         | 2         |
| 8         | 1         |

### Terminal emulator settings

When you connect a terminal emulator on a PC to a device's console port, configure these settings:

| Setting          | Cisco default | Description                                                                     |
| :--------------- | :------------ | :------------------------------------------------------------------------------ |
| **Speed**        | 9600 bps      | The rate at which data is sent                                                  |
| **Data bits**    | 8             | The number of bits used for each character of text sent to the device           |
| **Stop bits**    | 1             | Sent after every character so the receiving device can detect the end of it     |
| **Parity**       | None          | An extra bit sent with each character for error detection                       |
| **Flow control** | None          | Handles cases where a device sends data faster than the receiver can process it |

## Configuration files

IOS stores the device configuration in two text files. The files are stored in different hardware memory and serve different purposes:

|                | running-config                               | startup-config                                     |
| :------------- | :------------------------------------------- | :------------------------------------------------- |
| **Purpose**    | Determines the device's current operation    | Loaded by the device when it boots                 |
| **Stored in**  | Random-access memory (RAM)                   | Nonvolatile RAM (NVRAM), a special type of RAM     |
| **View with**  | `show running-config`                        | `show startup-config`                              |

When the device boots, it copies the contents of startup-config to running-config in RAM.

### Save the configuration

To save changes to running-config, copy running-config to startup-config. Otherwise, the device loses those changes when it reboots. It boots with the last saved startup-config, or with a factory-default configuration if no startup-config exists. A new device that boots for the first time has no startup-config file until you run one of these commands:

- `write`
- `write memory`
- `copy running-config startup-config`

### Restore the factory-default configuration

To return a device to its factory-default configuration, erase startup-config with one of these commands, then restart the device with `reload`:

- `write erase`
- `erase nvram:`
- `erase startup-config`

## Privileged EXEC passwords

Privileged EXEC mode lets a user enter global configuration mode and change the device configuration. Configure a password to prevent unauthorized users from accessing privileged EXEC mode.

### Enable password

The **enable password** is a password you must enter to access privileged EXEC mode. The enable password is a legacy feature.

```bash
R1(config)# enable password ccna
R1(config)# exit
R1# disable
R1> enable
Password:
R1#
```

To verify the password, filter the running configuration:

```bash
R1# show running-config | include enable
enable password ccna
```

#### Encrypt passwords

To improve the security of the enable password, run `service password-encryption` in global configuration mode. The command encrypts all current passwords on the device and all passwords you configure in the future. It uses **type 7** encryption.

```bash
R1(config)# service password-encryption
R1(config)# do show running-config | include enable
enable password 7 0307580507
```

To undo the encryption, run `no service password-encryption`:

- Currently encrypted passwords stay encrypted.
- Future passwords aren't encrypted.

### Enable secret

The **enable secret** is a more secure password. IOS stores it as a hash, rather than as encrypted ciphertext. **Hashing** is a one-way function. You can't reverse a hash to recover the original password.

```bash
R1(config)# enable secret cisco
R1(config)# do show running-config | include enable
enable secret 9 $9$emuJQV5sVZCY8v$INbrp9XrtfWHieMubzYt7N640m4KXDIqKg/a6SHY9lU
enable password 7 0307580507
```

- If you configure both the enable password and the enable secret, only the enable secret works. The enable password stays in the configuration, but you can't use it to access privileged EXEC mode.
- The hashing algorithm is **scrypt** (pronounced S-crypt), also called **type 9**. The `9` before the hash indicates the type.
- On many older devices, the default algorithm is Message Digest 5 (MD5), also called **type 5**. Type 5 isn't as secure as type 9, so use type 9.
