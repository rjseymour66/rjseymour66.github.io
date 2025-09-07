---
title: "Controlling and managing processes"
linkTitle: "xProcesses"
weight: 60
# description:
---

## Jobs

```bash
CTRL + z            # send current process into the backgroud
jobs                # view all jobs running in bg
fg                  # open most recent job
fg 1                # open job 1
<program> &         # start <program> in the bg
```

## ps

View processes currently running on your system:
- Process ID (PID) differentiates each process from the next on your server
  - Server only knows PID
  

| Field     | Description                                                                                                                 |
| :-------- | :-------------------------------------------------------------------------------------------------------------------------- |
| `PID`     | Process ID                                                                                                                  |
| `TTY`     | The TTY that the process is tied to                                                                                         |
| `STAT`    | Status code - the current state of the process. D (sleep), Z (defunct), T (stopped), S (interruptible sleep), R (run queue) |
| `TIME`    | Total amount of CPU time consumed by this process                                                                           |
| `COMMAND` | Command being run by the process                                                                                            |

- Interruptible sleep means that the process is waiting for input and cannot handle new signals
- defunct is a zombie, waiting for the parent process to clean it up
- TTY is teletypewriter - a terminal that accepts input and manages the output
  - Historically, when users connected to large mainframe computers with a keyboard and monitor
  - Now, its our terminal that is connected to a machine locally or with SSH
  - Most processes run on a TTY
  - `pts/0` is a virtual (pseudo) terminal, and `pts` is a designation assigned to a terminal emulator


```bash
ps                                  # list of processes run by user
pidof <name>                        # get PID of named process
ps a                                # list processes for all users
ps au                               # list processes with usernames
ps aux                              # list all processes for all users with usernames, and 
                                    # include daemons or bg processes, not limited to TTY processes
$ ps -ef                            # every process from parent shell back to init
ps aux | grep <keyword>             # search processes for <keyword> - commonly used
ps u -C <keyword>                   # show processes that have <keyword>
ps aux --sort=-pcpu                 # sort processes using most CPU
ps aux --sort=-pcpu | head -5       # show top 5 processes using most CPU since boot
ps aux --sort=-pmem | head -5       # show top 5 processes using most memory since boot
```

## pstree

```bash
# visualize parent/child processes (-p displays PIDs)
$ pstree -p
systemd(1)─┬─ModemManager(789)─┬─{ModemManager}(805)
           │                   ├─{ModemManager}(806)
           │                   └─{ModemManager}(813)
           ├─cron(857)
           ├─dbus-daemon(717)
           ├─login(866)───bash(982)
           ├─multipathd(364)─┬─{multipathd}(376)
           │                 ├─{multipathd}(377)
           │                 ├─{multipathd}(378)
           │                 ├─{multipathd}(379)
           │                 ├─{multipathd}(380)
           │                 └─{multipathd}(381)
...
```

## Change process priority

You can change the priority of running processes:
- Not very common today because of RAM available in modern machines
  - Some data processing and deep learning processes might require process priority changes
- nice values range from `-20` to `19`
- `nice` sets priority of processes when you start them
- The _higher_ the nice value, the _lower_ the priority
- `renice` changes priority of running processes
- You can't change the niceness of a process that you do not own
- Requires `sudo` to lower the nicencess of a process

> Think of the `nice` (`NI`) value as how nice a process is to other processes on the system, so the higher the niceness value, the nicer the process and the lower the priority.
> 
> lower the niceness value, the nicer the process and the and higher the priority. So, `10` is a higher priority than a process running `NI` of `20`.

### nice

Set the priority of a new process:

```bash
nice -n <num> <program>
```

### renice

Renice changes priority of running processes.

By default, processes begin with a nice value of `80`. If you use `renice` with the `-n 10` argument, then you are adding 10 to the default NI value, resulting in 90. If you again run `renice -n 5 -p <PID>`, then you are lowering the current NI value by 5, resulting in 85. 

```bash
ps -l                               # view priority and nice cols
renice -n <num> -p <PID>
renice -n <num> <process-name>


# --- Example --- #
# 1. View current PRI and NI
ps -l
F S   UID     PID    PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
0 S  1000    1093    1091  0  80   0 -  2254 do_wai pts/0    00:00:03 bash
0 T  1000   12047    1093  0  80   0 -  8981 do_sig pts/0    00:00:00 vim
0 R  1000   12196    1093 99  80   0 -  2729 -      pts/0    00:00:00 ps

# 2. Change priority with renice
renice -n 10 -p 12047
12047 (process ID) old priority 0, new priority 10

# 3. Verify the new PRI NI values
ps -l
F S   UID     PID    PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
0 S  1000    1093    1091  0  80   0 -  2254 do_wai pts/0    00:00:03 bash
0 T  1000   12047    1093  0  90  10 -  8981 do_sig pts/0    00:00:00 vim
0 R  1000   12205    1093 99  80   0 -  2729 -      pts/0    00:00:00 ps
```

## Misbehaving processes

Use the `kill` command to send a process a `SIGTERM (15)` signal to stop gracefully:
- a signal is sent to a process by the kernel, a process, or manually
  - signals tell the process about a request or a change
  - learn more at `man 7 signal`
- `SIGKILL (9)` force closes a process
  - Can have harmful effects, but probably not likely
  - Use case: zombie process. Might even need a reboot bc not assigned a CPU.

```bash
kill <PID>              # send SIGTERM
killall <process>       # send SIGTERM to all processes that match <process>

kill -9 <PID>           # send SIGKILL
killall -9 <process>    # not recommended
```

## Managing system processes

Daemons run processes in the background and typically start at boot:
- When we install an app to run as a service, the system confiugures it to start at boot
- Services are managed by the init system, or PID 1 - `systemd`
- "service", "daemon", and "unit" are synonymous
- Manage units with `systemctl`
- Check `vendor preset` in `Loaded` line of `systemctl status <unit>` to see if unit starts automatically after installation

### systemctl

```bash
systemctl                       # lists all available units
systemctl list-units            # same as above
systemctl status <unit>         # check status of service, enabled, recent logs

systemctl start <unit>          # starts the unit
systemctl stop <unit>           # stops the unit
systemctl restart <unit>        # restarts the unit (stops then starts)
systemctl reload <unit>         # activate new configurations without bringing unit down (ex: Apache)
                                # NOTE: not all units have 'reload' option

systemctl enable <unit>         # tell unit to start when server boots
systemctl disable <unit>        # tell unit not to start when server boots
systemctl enable --now ssh      # enable a unit and also start it
```

## init

```bash
ps -ef | grep init              # view init
file /sbin/init                 # init is symlink to systemd
```

## Scheduling with cron

`cron` can run a process at a specific time:
- Each user has their own cron configurations, called a `crontab`
- `crontab` is a list of cron jobs, one per line
- cron jobs require fully-qualified commands, such as `/usr/bin/apt`
  - find path with `which`
- `root` can schedule system-wide tasks

```bash
crontab -l                      # view your crontab
sudo crontab -u <user> -l       # view <user>'s crontab

crontab -e                      # edit crontab (create new cronjob)
EDITOR=vim crontab -e           # set default editor for crontab actions

# --- Example crontab entries --- #
3 0 * * 4 /usr/local/bin/backup.sh      # run cleanup.sh at 12:03 AM each Thursday
* 0 * * * /usr/bin/apt update           # run apt update at midnight each night
```