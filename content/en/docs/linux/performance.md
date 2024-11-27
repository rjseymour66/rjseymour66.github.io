---
title: "Systems performance"
linkTitle: "Performance"
# weight: 1000
# description:
---

There are four core elements of each system:
1. CPU
2. Memory--RAM, physical, and virtual
3. Storage devices
4. Network load management

## CPUs

CPUs wait for instructions, perform computations, then return answers.

### Find number of processors

```bash
# get CPU count
cat /proc/cpuinfo | grep processor
processor	: 0
processor	: 1
```

## CPU load

There are two main indicators:

CPU load
: Number of currently active and queued processes that the CPU is handling, as a percentage of the total CPU capacity. Measure this metric over time to give an accurate description.

CPU utilization
: Measure of time that a CPU is not idle.
  - `1` on a single-core machine is full capacity.
  - `1.25` on single-core means CPU is at capacity and 25% of processes are waiting.
  - `4` on four-core machine is full capacity.  
  - 75% utilization is when you start seeing performance degradation.

### Load average

```bash
# current time, time since last boot, # logged in users, last min, 5 mins, 15mins
uptime                              
 09:58:16 up 7 days, 14:43,  1 user,  load average: 2.44, 2.19, 2.01

```

### Manage CPU load

```bash
top

```

### Simulate CPU load

Use `yes` to mimic high CPU loads. It writes the word "yes" to the STDOUT repeatedly:

```bash
yes > /dev/null &
[1] 5887

# kill the process
killall yes
```