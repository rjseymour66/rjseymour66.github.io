---
title: "Optimizing performance"
linkTitle: "Performance"
weight: 200
---


## Monitor processes 

Each running program is a _process_ that can run in the foreground, display output on a sonsole or GUI, or run in the background.
- Each process gets a process ID (PID), and linux uses the PID to manage how the process uses CPU and memory.

When linux boots, it starts the `init` process
- Core of the system, it runs scripts that start all other processes running on the system


### pgrep

Returns the PID for the currently running process that matches the pattern:

```bash
pgrep system*
1
213
260
522
524
715
785
...

# find process
pgrep sleep
1675989
1680227

# Send hangup signal
$ sudo kill -SIGHUP 1680227

# verify process is no longer running
$ pgrep sleep
1675989

# view in other terminal
sleep 1000
Hangup
```

### pidof

Returns the PID of the specified process:

```bash
pidof systemd
5276 2687
```

### ps

View currently running processes. Accepts different kinds of params:
- preceded by dash (Unix-style)
- no dash (BSD-style)
- double dash (GNU-style)

```bash
# ps columns
UID          PID    PPID  C STIME TTY          TIME CMD
```

UID
: user ID that is running the program

PID
: process ID

PPID
: process ID of the parent process

C
: processor utilization over the lifetime of the process

STIME
: system time when the process started

TTY
: terminal device that started the process

TIME
: cumulative CPU time required to run the process

CMD
: name of the program that started the process
  
[<program>]
: In the `CMD` column means that the process is is currently swapped out from physical memory into virtual memory (swap space) on the hard drive. 

  These processes are _sleeping_, which usually means that they are waiting for an event. When an event triggers, the kernel sends the process a signal, and it reacts depending on which sleep mode it is in:
  - **interruptible sleep**: receives the signal and immediately wakes up
  - **uninterruptible sleep**: only wakes up based on an external event (ex: hardware available). It saves other signals and acts on them when it wakes up.

  A _zombie process_ is when a process ended but its parent hasn't acknowledged the termination signal because it is sleeping.



```bash
# processes in current user shell
ps
    PID TTY          TIME CMD
 362858 pts/0    00:00:00 bash
1554908 pts/0    00:00:00 ps

# every process running on the system
ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 Apr18 ?        00:00:14 /sbin/init splash
root           2       0  0 Apr18 ?        00:00:00 [kthreadd]
root           3       2  0 Apr18 ?        00:00:00 [rcu_gp]
root           4       2  0 Apr18 ?        00:00:00 [rcu_par_gp]
root           5       2  0 Apr18 ?        00:00:00 [slub_flushwq]
root           6       2  0 Apr18 ?        00:00:00 [netns]
root           8       2  0 Apr18 ?        00:00:00 [kworker/0:0H-events_highpri]
...
```

### top

Displays process information in real time. The top section includes general system information:


```bash
top - 09:47:32 up 5 days, 11:54,  1 user,  load average: 0.75, 0.52, 0.48       # general sys info
Tasks: 376 total,   1 running, 375 sleeping,   0 stopped,   0 zombie            # general process info
%Cpu(s):  0.8 us,  1.2 sy,  0.2 ni, 97.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st # CPU info 
MiB Mem :  31704.9 total,  10716.0 free,   8790.9 used,  12198.0 buff/cache     # physical memory
MiB Swap:  65536.0 total,  65536.0 free,      0.0 used.  21815.3 avail Mem      # swap memory

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND                      
   7972 linuxus+  20   0 6294616   3.8g   3.7g S   7.9  12.2 100:58.48 VBoxHeadless                 
   4565 linuxus+  20   0 6446072 372728 147248 S   4.6   1.1  39:17.97 gnome-shell                  
 652102 linuxus+  20   0 2788404 593496 537340 S   2.0   1.8  18:25.31 VBoxHeadless                 
   5781 linuxus+  20   0  582208  78284  44020 S   1.7   0.2   2:16.89 gnome-terminal-              
    314 root     -51   0       0      0      0 S   1.0   0.0   2:14.13 irq/51-SYNA2393              
   1702 mysql     20   0 2376580 399892  36116 S   1.0   1.2   9:49.43 mysqld          
```

#### CPU details

| Symbol | Category | Description |
|---|---|---|
| us | User | amount of time the CPU spends running application code |
| sy | System | amount of time the CPU spends working with sys resources |
| ni | Nice | amount of time CPU spends running low-priority processes |
| id | Idle | amount of time CPU is not busy |
| wa | Waiting | (`iowait`) amount of time CPU spends waiting for disk or network ops to complete |
| hi | Hardware interrupt | amount of time CPU spends processing hardware interrupts |
| si | Software interrupt | amount of time CPU spends processing software interrupts |

#### Process columns

PID
: process ID

USER
: process owner's username

PR
: priority of process

NI
: nice value of process

VIRT
: amount of virtual memory (swap space) used by the process

RES
: amount of physical memory used by the process

SHR
: amount of memory that the process is sharing with other processes

S
: Process status:
  - D: interruptible sleep
  - R: running
  - S: sleeping
  - T: traced or stopped
  - Z: zombie

%CPU
: share of CPU time that the process is using

%MEM
: share of available physical memory that the process is using

TIME+
: total CPU time that the process has used since starting

COMMAND
: command-line name of the process

#### Interactive commands

`top` sorts processes based on `%CPU` value by default. There are a lot of options, so here are articles explaining a few:

- [man page](https://man7.org/linux/man-pages/man1/top.1.html#4._INTERACTIVE_Commands)
- [Unleash top](https://medium.com/@extio/unleashing-the-power-of-the-linux-top-command-a-comprehensive-guide-d26ce09fa71a)

### htop

Improved version of `top`:

[Rocky Linux docs](https://docs.rockylinux.org/gemstones/htop/)


## Manage processes

### nice

By default, all processes have the same priority to obtain CPU time and memory resources. `nice` starts an application with a non-default priority setting:

```bash
# lower val = higher priority
nice -n <val> <command>
```

### renice

By default, all processes have the same priority to obtain CPU time and memory resources. `renice` changes the priority of a process that's already running:

```bash
renice <priority> [-p pids] [-u users] [-g groups]

# example
renice 15 -p 3178
```

## Stopping processes

There are 2 commands that send a process signal to running processes: `kill` and `pkill`.

### Process signals

Processes communicate with each other using _process signals_, which are predefined messages that processes recognize and may choose to ignore or act on.

| Number | Name | Description |
|---|---|---|
| 1 | `SIGHUP` | Hangs up |
| 2 | `SIGINT` | Interrupts |
| 3 | `SIGQUIT` | Stops running |
| 9 | `SIGKILL` | Unconditionally terminates |
| 11 | `SIGSEGV` | Segments violation |
| 15 | `SIGTERM` | Terminates if possible |
| 17 | `SIGSTOP` | Stops unconditionally but doesn't terminate |
| 18 | `SIGTSTP` | Stops or pauses but continues to run in the bg |
| 19 | `SIGCONT` | Resumes execution after STOP or TSTP |

### kill

Sends a signal process based on PID. By default, sends a `SIGTERM` signal, but you can specify the signal with the `-s` option and should send them in the following order:
- `TERM`
- `SIGINT` or `SIGHUP`
- `SIGKILL` (most forecful)

There is no output, so you have to send use the `ps` or `top` command to verify that the command stopped.

```bash
kill <PID>

# send SIGTERM
kill 3980

# specify the signal
kill -s SIGHUP 3980

# specify the signal shorthand
kill -SIGHUP 3980
```

### pkill

Stop processes using the name instead of PID. Accepts wildcard characters, too:

```bash
sudo pkill http*
```



```bash

```