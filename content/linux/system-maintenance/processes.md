+++
title = 'Processes'
date = '2025-09-07T18:49:03-04:00'
weight = 60
draft = false
+++



## Jobs

Use job control to run and manage multiple processes from a single shell session:

```bash
CTRL + z            # suspend the current process and send it to the background
jobs                # list all background jobs
fg                  # bring the most recent job to the foreground
fg 1                # bring job 1 to the foreground
bg 1                # resume job 1 in the background
<program> &         # start <program> directly in the background
```

## ps

Use `ps` to list processes running on your system.

{{< admonition "Process ID" "note" >}}
Each process has a unique **Process ID (PID)**. The kernel identifies and tracks processes by PID only.
{{< /admonition >}}

| Field     | Description                                                                                                                                           |
| :-------- | :---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `PID`     | Process ID                                                                                                                                            |
| `TTY`     | Terminal the process is tied to                                                                                                                       |
| `STAT`    | Status code: the current state of the process. `D` (uninterruptible sleep), `Z` (defunct), `T` (stopped), `S` (interruptible sleep), `R` (run queue) |
| `TIME`    | Total CPU time consumed by this process                                                                                                               |
| `COMMAND` | Command the process is running                                                                                                                        |

The `STAT` column uses single-letter codes to show a process's current state. `S` (interruptible sleep) means the process is waiting for an event and can be interrupted by signals. `D` (uninterruptible sleep) means the process is waiting (usually on I/O) and cannot be interrupted. `Z` (defunct) marks a zombie process waiting for its parent to clean it up.

`TTY` stands for teletypewriter, historically a physical terminal connected to a mainframe. Today it's the terminal connected to your machine locally or over SSH. `pts/0` is a pseudo-terminal (a virtual terminal assigned by a terminal emulator):

```bash
ps                                  # list processes for the current user
pidof <name>                        # get the PID of a named process
ps a                                # list processes for all users
ps au                               # list processes with usernames
ps aux                              # list all processes, including daemons not tied to a TTY
ps -ef                              # every process, from init down
ps aux | grep <keyword>             # search processes by keyword
ps u -C <keyword>                   # show processes matching keyword
ps aux --sort=-pcpu                 # sort by CPU usage (highest first)
ps aux --sort=-pcpu | head -5       # show the 5 processes using the most CPU
ps aux --sort=-pmem | head -5       # show the 5 processes using the most memory
```

## pstree

Use `pstree` to visualize running processes as a tree, showing parent/child relationships at a glance. The `-p` flag includes each process's PID:

```bash
pstree -p
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

Linux assigns each process a priority that determines how much CPU time the scheduler gives it. Process priority changes are most common in CPU-intensive workloads like data processing or machine learning jobs.

`nice` values range from `-20` (highest priority) to `19` (lowest priority). The higher the nice value, the more a process yields to others. `nice` sets the priority of a process when you start it. `renice` changes the priority of a process that is already running. You cannot change the niceness of a process you do not own, and you need `sudo` to lower the nice value (increase priority) of any process.

### nice

Pass `-n` with a value from `-20` to `19` to set the priority when starting a process:

```bash
nice -n <num> <program>
```

### renice

`renice` changes the priority of a process that is already running. The `-n` flag takes an absolute niceness (`NI`) value, replacing the current value rather than adjusting it. New processes start with a default NI of `0`:

```bash
renice -n <num> -p <PID>
renice -n <num> <process-name>
```

In this example, vim is running with PID `12047` at the default NI of `0`. Raising its NI to `10` reduces its CPU scheduling priority.

1. Check the current `PRI` and `NI` values with `ps -l`.

   ```bash
   ps -l
   F S   UID     PID    PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
   0 S  1000    1093    1091  0  80   0 -  2254 do_wai pts/0    00:00:03 bash
   0 T  1000   12047    1093  0  80   0 -  8981 do_sig pts/0    00:00:00 vim
   0 R  1000   12196    1093 99  80   0 -  2729 -      pts/0    00:00:00 ps
   ```

2. Set the NI value for the vim process (PID `12047`) to `10`.

   ```bash
   renice -n 10 -p 12047
   12047 (process ID) old priority 0, new priority 10
   ```

3. Confirm the updated `PRI` and `NI` values.

   ```bash
   ps -l
   F S   UID     PID    PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
   0 S  1000    1093    1091  0  80   0 -  2254 do_wai pts/0    00:00:03 bash
   0 T  1000   12047    1093  0  90  10 -  8981 do_sig pts/0    00:00:00 vim
   0 R  1000   12205    1093 99  80   0 -  2729 -      pts/0    00:00:00 ps
   ```

## Misbehaving processes

The `kill` command sends signals to processes. A signal is a notification from the kernel, another process, or a user that tells a process to take action. `SIGTERM (15)` asks a process to stop gracefully. `SIGKILL (9)` forces it to stop immediately, bypassing any cleanup handlers. Reserve it for processes that do not respond to `SIGTERM`, such as zombie processes. For a full signal reference, run `man 7 signal`:

```bash
kill <PID>              # send SIGTERM
killall <process>       # send SIGTERM to all processes matching <process>

kill -9 <PID>           # send SIGKILL
killall -9 <process>    # not recommended
```

## Managing system processes

Daemons run in the background and typically start at boot. When you install an app to run as a service, the system configures it to start automatically. The terms `service`, `daemon`, and `unit` are interchangeable.

`systemd` is the init system (PID 1) responsible for managing these services. Manage units with `systemctl`. To check whether a unit starts automatically after installation, look for the `vendor preset` field in the `Loaded` line of `systemctl status <unit>`.

### systemctl

The following commands cover the most common unit management tasks:

```bash
systemctl                       # list all available units
systemctl list-units            # same as above
systemctl status <unit>         # show status, enabled state, and recent logs

systemctl start <unit>          # start the unit
systemctl stop <unit>           # stop the unit
systemctl restart <unit>        # stop then start the unit
systemctl reload <unit>         # reload configuration without stopping the unit (not supported by all units)

systemctl enable <unit>         # start the unit automatically at boot
systemctl disable <unit>        # do not start the unit at boot
systemctl enable --now ssh      # enable a unit and start it immediately
```

## init

`init` is the first process the kernel starts at boot, always assigned PID 1. On modern Linux systems, `/sbin/init` is a symlink to `systemd`. These commands let you confirm that relationship:

```bash
ps -ef | grep init              # find the init process
file /sbin/init                 # show what init points to
```

## Scheduling with cron

`cron` runs commands on a schedule. Each user has a personal schedule file called a `crontab` (cron table), where each line defines one job. Cron jobs require fully-qualified command paths. Find the path for a command with `which`. `root` can also schedule system-wide tasks.

Each crontab entry follows this format:

```
<minute> <hour> <day-of-month> <month> <day-of-week> <command>
```

An asterisk (`*`) in any field means "every." Values range from `0–59` for minutes, `0–23` for hours, `1–31` for day of month, `1–12` for month, and `0–7` for day of week (0 and 7 both represent Sunday):

```bash
crontab -l                              # view your crontab
sudo crontab -u <user> -l               # view another user's crontab

crontab -e                              # open your crontab in the default editor
EDITOR=vim crontab -e                   # open your crontab in vim for this invocation

# --- Example crontab entries --- #
3 0 * * 4 /usr/local/bin/backup.sh      # run backup.sh at 12:03 AM each Thursday
0 0 * * * /usr/bin/apt update           # run apt update once at midnight each night
```