---
title: "Controlling and managing processes"
linkTitle: "Processes"
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
  

| Field | Description |
|:--|:--|
| `PID` | Process ID |
| `TTY` | The TTY that the process is tied to | 
| `STAT` | Status code - the current state of the process. D (sleep), Z (defunct), T (stopped), S (interruptible sleep), R (run queue) | 
| `TIME` | Total amount of CPU time consumed by this process | 
| `COMMAND` | Command being run by the process | 

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
ps aux | grep <keyword>             # search processes for <keyword> - commonly used
ps u -C <keyword>                   # show processes that have <keyword>
ps aux --sort=-pcpu                 # sort processes using most CPU
ps aux --sort=-pcpu | head -5       # show top 5 processes using most CPU since boot
ps aux --sort=-pmem | head -5       # show top 5 processes using most memory since boot

```