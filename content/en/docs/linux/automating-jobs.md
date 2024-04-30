---
title: "Automating jobs"
weight: 220
---

## Background mode

Run scripts and programs in the background so you can continue to work in your shell terminal.

### Terminal session

Use an ampersand (`&`) to run a process in background mode. It separates the command from the bash shell and runs it as a separate background process. The process runs to completion, or until the terminal session ends.

- `[n]`: job number
- `nnnnn`: PID of the job

```bash
/for_loop.sh &
[1] 43785

# displays Done when complete
[1]+  Done                    ./for_loop.sh

# first job
./sleep1.sh &
[1] 43993

# second job
./sleep2.sh &
[2] 43995

# third job
./sleep3.sh &
[3] 43999

# view processes
ps au | grep sleep
linuxus+   43993  0.0  0.0   9972  3456 pts/0    S    09:56   0:00 /bin/bash ./sleep1.sh
linuxus+   43994  0.0  0.0   8372  1920 pts/0    S    09:56   0:00 sleep 200
linuxus+   43995  0.0  0.0   9972  3456 pts/0    S    09:56   0:00 /bin/bash ./sleep2.sh
linuxus+   43996  0.0  0.0   8372  1920 pts/0    S    09:56   0:00 sleep 200
linuxus+   43999  0.0  0.0   9972  3456 pts/0    S    09:56   0:00 /bin/bash ./sleep3.sh
linuxus+   44000  0.0  0.0   8372  1920 pts/0    S    09:56   0:00 sleep 200
linuxus+   44003  0.0  0.0   9212  2560 pts/0    S+   09:56   0:00 grep --color=auto sleep

# view running jobs
jobs
[1]   Running                 ./sleep1.sh &
[2]-  Running                 ./sleep2.sh &
[3]+  Running                 ./sleep3.sh &
```

### nohup

`nohup` ("no hangup") runs a process in the background that can live beyond the current session. For example, you can run a job and then exit the terminal, and the job will continue to run to completion.

It works by running another command that blocks any `SIGHUP` signals that are sent to the process. To maintain any output that would normally be written to the terminal, it creates a file named `nohup.out` in the current working directory:

```bash
# shell 1
nohup ./looper_nohup.sh &
[1] 2575816
nohup: ignoring input and appending output to 'nohup.out'


# shell 2
cat nohup.out 
Count:	10
Count:	9
Count:	8
Count:	7
Count:	6
Count:	5
Count:	4
Count:	3
Count:	2
Count:	1
```

## Sending signals

You can control operations running in the foreground with key combinations:

**CTRL + C**
: Sends signal interrupt (SIGINT)

**CTRL + D**
: Sends end-of-file (EOF) to shell. Helpful when entering a data stream. Without a data stream, it terminates the shell.

**CTRL + Z**
: Sends a signal terminal stop (SIGTSTP) to stop (pause) a process without terminating it. This leaves the program in memory, so it can pick up where it left off.
  
  ```bash
  # start process  
  sleep 33
  ^Z
  [1]+  Stopped                 sleep 33

  # bash shell warns you if you try to exit
  $ exit
  exit
  There are stopped jobs.

  # view processes
  $ ps au
  USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
  linuxus+    4478  0.0  0.0 162388  6088 tty2     Ssl+ Apr18   0:00 /usr/libexec/gdm-wayland-session env GNOME_SHELL_SESS
  linuxus+    4481  0.0  0.0 223040 15244 tty2     Sl+  Apr18   0:00 /usr/libexec/gnome-session-binary --session=ubuntu
  ...
  linuxus+ 2586354  0.1  0.0  16488  9504 pts/3    Ss   21:31   0:00 bash
  linuxus+ 2586665  0.0  0.0   8372  1004 pts/3    T    21:31   0:00 sleep 33
  linuxus+ 2586976  0.0  0.0  14592  4296 pts/3    R+   21:32   0:00 ps au

  # kill process
  $ kill -9 2586665
  [1]+  Killed                  sleep 33
  ```

## Job control

Job control is starting, stopping, killing, and resuming jobs.
- `+` sign is the _default_ job
- `-` sign is the will become default job when the current default job completes processing

```bash
jobs
-l # list PID of process along w/job number
-n # lists only jobs that have changed status since last notification from the shell
-p # lists only the PIDs of the jobs
-r # lists only running jobs
-s # lists only stopped jobs

# ./job_control.sh 
This is job control program 2614902
Loop #10
^Z
[1]+  Stopped                 ./job_control.sh

# start second process
$ ./job_control.sh > job_control.sh.out &
[2] 2615448

# view both processes
$ jobs
[1]+  Stopped                 ./job_control.sh
[2]-  Running                 ./job_control.sh > job_control.sh.out &

# view all jobs
jobs -l
[1]  2614902 Stopped                 ./job_control.sh
[2]- 2619268 Stopped                 ./job_control.sh
[3]+ 2619396 Stopped                 ./job_control.sh

# kill one job
$ kill -9 2614902
[1]   Killed                  ./job_control.sh

# kill two jobs 
$ kill -9 2619268 2619396
[2]-  Killed                  ./job_control.sh
[3]+  Killed                  ./job_control.sh
```

### bg

Runs a job in the background:

```bash
# all jobs stopped
jobs
[1]   Stopped                 ./job_control.sh
[2]-  Stopped                 ./job_control.sh
[3]+  Stopped                 ./job_control.sh

# run job 1 in background
bg 1
[1] ./job_control.sh &
Loop \#9

# view jobs
jobs
[1]   Running                 ./job_control.sh &
[2]-  Stopped                 ./job_control.sh
[3]+  Stopped                 ./job_control.sh

```

### fg

Runs a job in the foreground. This means that the job takes over the shell:

```bash
# view jobs
jobs -l
[1]+ 2634046 Stopped                 ./job_control.sh

# run job 1 in foreground
$ fg 1
./job_control.sh
Loop #9
Loop #8
```

## Run scripts at preset time

### at

Lets you specify a time when the linux system runs a script. You have to submit each job that you want to run, you don't schedule recurring jobs:
- The system adds the job to a queue with directions about when the shell should run the job.
- The `atd` daemon runs in the background (starting at boot) and checks the queue for jobs to run
  - `/var/spool/at` contains the job queue
  - There are 26 different job queues available for different priority levels, using lowercase `a` - `z`
- time accepts following formats:
  - 10:15
  - 10:15 p.m.
  - now, noon, midnight, or teatime (4 p.m.)
  - MMDDYY, MM/DD/YY, DD.MM.YY
  - Jul 4 or Dec 25
  - Now + 25 minutes
  - 10:15 p.m. tomorrow
  - 22:15 tomorrow
  - 10:15 + 7 days
- 

```bash
at [-f <filename>] <time>

# check pending jobs
atq
1	Tue Apr 30 22:20:00 2024 a rseymour

# delete pending job
atrm 1

# verify deleted
atq
```

### cron

`cron` lets you schedule jobs that need to run on a regular basis.
- runs in the background and checks _cron tables_ for jobs that are scheduled to run
  - all users have their own cron tables
  - handle tables with `crontab`
  - when you add entry to cron table, shell opens vi editor
- runs with the user account that submitted the job

```bash
# format
min hour dayofmonth month dayofweek command

# 10:15am each day
15 10 * * * /full/path/to/program.sh

# 4:15pm every Monday (0 - Sun, 6 - Sat)
15 16 * * 1 /full/path/to/program.sh

# 12 noon first day of each month
00 12 1 * * /full/path/to/program.sh

# list existing cron table
crontab -l
no crontab for linuxuser

crontab -l
# Edit this file to introduce tasks to be run by cron.
... 
# m h  dom mon dow   command
54 22 * * 1 /home/linuxuser/cron_echo.sh > cron.out


# add entry
crontab -e
(opens cron table in vi)

# delete entry
crontab -r

# verify deleted
crontab -l
no crontab for linuxuser
```

### Systemd timers

Use the systemd startup method to schedule jobs:

[Systemd timers explained](https://coady.tech/systemd-timer-vs-cron/)