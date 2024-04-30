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
  ryansey+    4478  0.0  0.0 162388  6088 tty2     Ssl+ Apr18   0:00 /usr/libexec/gdm-wayland-session env GNOME_SHELL_SESS
  ryansey+    4481  0.0  0.0 223040 15244 tty2     Sl+  Apr18   0:00 /usr/libexec/gnome-session-binary --session=ubuntu
  ryansey+  190137  0.0  0.0  16488  9600 pts/1    Ss   Apr19   0:00 bash
  ryansey+  651009  0.0  0.0  16488  9572 pts/2    Ss   Apr20   0:00 bash
  ryansey+ 1766607  0.0  0.0  17352  9828 pts/2    S+   Apr24   0:00 ssh -p 2223 rseymour@localhost
  ryansey+ 1812565  0.0  0.0  16488  9584 pts/0    Ss+  Apr24   0:00 bash
  ryansey+ 2362947  0.0  0.0  17360  9048 pts/1    S+   Apr27   0:00 ssh -p 2222 linuxuser@localhost
  ryansey+ 2586354  0.1  0.0  16488  9504 pts/3    Ss   21:31   0:00 bash
  ryansey+ 2586665  0.0  0.0   8372  1004 pts/3    T    21:31   0:00 sleep 33
  ryansey+ 2586976  0.0  0.0  14592  4296 pts/3    R+   21:32   0:00 ps au

  # kill process
  $ kill -9 2586665
  [1]+  Killed                  sleep 33
  ```

## Job control