---
title: "Processes"
linkTitle: "Processes"
# weight: 1000
# description:
---

## ps

```bash
# every process from parent shell back to init
$ ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 03:04 ?        00:00:05 /sbin/init
root           2       0  0 03:04 ?        00:00:00 [kthreadd]
root           3       2  0 03:04 ?        00:00:00 [pool_workqueue_release]
root           4       2  0 03:04 ?        00:00:00 [kworker/R-rcu_g]
root           5       2  0 03:04 ?        00:00:00 [kworker/R-rcu_p]
root           6       2  0 03:04 ?        00:00:00 [kworker/R-slub_]
root           7       2  0 03:04 ?        00:00:00 [kworker/R-netns]
root          10       2  0 03:04 ?        00:00:00 [kworker/0:0H-events_highpri]
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

## init

```bash
# view init
$ ps -ef | grep init
root           1       0  0 03:04 ?        00:00:05 /sbin/init

# init is symlink to systemd
$ file /sbin/init
/sbin/init: symbolic link to ../lib/systemd/systemd
```

## kill and killall

```bash
# kill a single process
kill <pid>

# kill all instances of a program
killall <process-name>
```

## Priority

Not used very often...

Set priority if a process is taking up too much CPU, but you can't kill it because it is a critical service.

### nice and renice

View the `nice` value with the `NI` column in the `top` command.

Set `nice` value between `-20` and `19`.
- Higher the value, the _nicer_ the process is about giving up system resources to other processes.

`renice` changes the `nice` value after its started.

```bash
# high nice value
nice -15 /scripts/myscript.sh

# low nice value
nice --15 /scripts/myscript.sh
```