---
title: "CPU and memory"
weight: 190
---

You need to understand your CPU hardware:
- Number of cores
- huperthreading
- cache sizes

```bash
cat /proc/cpuinfo
processor	: 0
vendor_id	: GenuineIntel
cpu family	: 6
model		: 158
model name	: Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
stepping	: 10
microcode	: 0xf4
cpu MHz		: 2600.000
cache size	: 12288 KB
physical id	: 0
siblings	: 12
core id		: 0
cpu cores	: 6
apicid		: 0
initial apicid	: 0
fpu		: yes
fpu_exception	: yes
cpuid level	: 22
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid ept_ad fsgsbase tsc_adjust sgx bmi1 avx2 smep bmi2 erms invpcid mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp sgx_lc md_clear flush_l1d arch_capabilities
vmx flags	: vnmi preemption_timer invvpid ept_x_only ept_ad ept_1gb flexpriority tsc_offset vtpr mtf vapic ept vpid unrestricted_guest ple pml ept_mode_based_exec
bugs		: cpu_meltdown spectre_v1 spectre_v2 spec_store_bypass l1tf mds swapgs itlb_multihit srbds mmio_stale_data retbleed gds
bogomips	: 5199.98
clflush size	: 64
cache_alignment	: 64
address sizes	: 39 bits physical, 48 bits virtual
power management:

processor	: 1
vendor_id	: GenuineIntel
...
```

### uptime

Tells the amount of time that a system has been up, and also the _load averages_. Load averages are the average amount of processes waiting for or using the CPU.
- Single core processor - load avg 2 means that a process is using the CPU while another waits

```bash
# 1, 5, 15 min load average
uptime
 20:40:35 up 4 days, 22:47,  1 user,  load average: 0.71, 0.75, 0.77
```
### sar

System activity reporter. Helps you view CPU performance over time. Available in `sysstat` package.
- Uses data stored by the `sadc` program in the `/var/log/sa/` directory

```bash
# start sysstat collection
systemctl start sysstat
systemctl enable sysstat
Synchronizing state of sysstat.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable sysstat
Created symlink /etc/systemd/system/multi-user.target.wants/sysstat.service → /lib/systemd/system/sysstat.service.
Created symlink /etc/systemd/system/sysstat.service.wants/sysstat-collect.timer → /lib/systemd/system/sysstat-collect.timer.
Created symlink /etc/systemd/system/sysstat.service.wants/sysstat-summary.timer → /lib/systemd/system/sysstat-summary.timer.

# run sar
$ sar -u
Linux 5.15.0-105-generic (precision-5540) 	04/23/2024 	_x86_64_	(12 CPU)

08:48:27 PM  LINUX RESTART	(12 CPU)
# run sar every 2 seconds, 30 times
$ sar -u 2 30
```

## Memory

Processes temporarily store data in RAM because it is easier and faster to access than disk. For example, there is _disk buffering_, which is when data is read from disk and stored in a location called a _buffer cache_. This lets the computer read data from memory, not disk.

The kernel maintains shared memory areas that allow multiple programs to read and write.

```bash
# view system memory
cat /proc/meminfo 
MemTotal:       32465788 kB
MemFree:        10995932 kB
MemAvailable:   22344756 kB
Buffers:         2053268 kB
Cached:          7098176 kB
SwapCached:            0 kB
Active:          4530664 kB
...
```

### ipcs

Shows information on IPC facilities.

```bash
# view shared active memory segments
ipcs -m

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status      
0x00d61fe8 0          postgres   600        56         6                       
0x0052e2c1 1          postgres   600        56         6                       
0x0052e6a9 2          postgres   600        56         6                       
0x00000000 131109     ryanseymou 600        524288     2          dest         
0x00000000 98346      ryanseymou 600        524288     2          dest  
```

## Swapping

Memory is divided into chunks called _pages_. Swapping occurs when the system takes the memory for an idle process and copies it to disk to make room for active processes. This section of disk is called:
- swap space
- swap
- virtual memory

When the idle process is active again, the memory is copied from disk back into memory.

Swap space is either a file or a disk partition, called a swap partition. This is added to the `/etc/fstab` config file.


### vmstat

View disk I/O spcific to swapping, and blocks in and blocks out to the device:

```bash
vmstat
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 10863292 2055396 10472992    0    0    18    32   32   14  3  2 95  0  0

```

### swapon 

View if the swap space is a file or disk partition:

```bash
# file swap space
swapon -s
Filename				Type		Size		Used		Priority
/swapfile               file		67108860	0		    -2

# partition swap space
swapon -s
Filename				Type		Size	Used	Priority
/dev/sda2             	partition	1048572	0	    -2
```

### mkswap

Makes a swap partition.


```bash
# make swap partition
mkswap /dev/<partition>

# view partition details
blkid /dev/<partition>

# view current free disk space
free -h
              total        used        free      shared  buff/cache   available
Mem:          3.6Gi       177Mi       3.0Gi       8.0Mi       353Mi       3.2Gi
Swap:         1.0Gi          0B       1.0Gi


# activate swap partition
swapon /dev/<partition>

# verify disk space
free -h
              total        used        free      shared  buff/cache   available
Mem:          3.6Gi       177Mi       3.0Gi       8.0Mi       353Mi       3.2Gi
Swap:         2.0Gi          0B       2.0Gi
```

Changing swap priority:

```bash
# disengage swap space
swapoff /dev/<partition>

# change the priority
swapon -p 0 /dev/<partition>


# verify priority
swapon -s
Filename				Type		Size	Used	Priority
/dev/sda2               partition	1048572	0	    0
```

## OOM killer

The kernel always overcommits memory to various processes. The out of memory killer (OOM killer) assigns a score to processes that use memory and then kills processes with the highest score until the system is using a safe amount of memory. It automatically assignes a low score to the following:
- kernel
- root
- crucial processes

You can modify the OOM killer with the following parameters:
- `vm.panic_on`
- `kernel.panic`
- `vm.overcommit_memory`
- `overcommit_ratio`