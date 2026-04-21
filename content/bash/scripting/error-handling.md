---
title: "Error handling"
weight: 60
description: >
  Exit status, exit codes, traps, signal handling, getopts, and process management.
---

## Exit status

Every command returns an *exit status*: `0` means success, and any non-zero value means failure. The special variable `$?` holds the exit status of the most recent command:

```bash
ls /etc/hosts
echo $?    # 0 (success)

ls /nonexistent
echo $?    # 2 (failure)
```

### Common exit codes

Exit codes are not globally standardized, but many tools follow conventions set by the C standard library and the Advanced Bash-Scripting Guide:

| Code    | Meaning                                      | Common source                                                                      |
| :------ | :------------------------------------------- | :--------------------------------------------------------------------------------- |
| `0`     | Success                                      | Any command that completed without error                                           |
| `1`     | General error                                | Catchall for miscellaneous failures (`grep` with no match, logic errors)           |
| `2`     | Misuse of shell built-in or invalid argument | `ls` on a nonexistent path, missing required argument                              |
| `126`   | Command found but not executable             | File exists but lacks execute permission                                           |
| `127`   | Command not found                            | Typo in command name, or command not in `PATH`                                     |
| `128`   | Invalid argument to `exit`                   | `exit 3.14` or `exit -1`                                                           |
| `128+N` | Fatal signal N                               | `130` = interrupted with Ctrl-C (signal 2), `137` = killed with SIGKILL (signal 9) |
| `255`   | Exit status out of range                     | Script called `exit` with a value greater than 255                                 |

Codes `130` and `137` are the ones you encounter most often in practice. `130` means the user pressed Ctrl-C; `137` means the process was force-killed, usually by the OOM killer or a `kill -9`.

### Setting a script's exit code

Call `exit` with a number to terminate the script and return that code to the caller. Any value from `0` to `255` is valid:

```bash
exit 0    # success
exit 1    # general failure
exit 2    # invalid argument or usage error
```

The caller reads the exit code from `$?` and decides what to do next. The caller might be another script, a CI pipeline, or a shell. A non-zero code causes `&&` chains to stop and `set -e` scripts to abort.

Establish a consistent convention at the top of each script so the codes are easy to audit:

```bash
readonly E_SUCCESS=0
readonly E_ARGS=2
readonly E_NOTFOUND=3

if [ $# -lt 1 ]; then
    echo "Usage: $0 <config-file>" >&2
    exit $E_ARGS
fi

if [ ! -f "$1" ]; then
    echo "Error: '$1' not found" >&2
    exit $E_NOTFOUND
fi

# main logic
exit $E_SUCCESS
```

If a script ends without an explicit `exit`, it returns the exit status of its last command. Always call `exit` explicitly so the return value is intentional and not an accident of whatever ran last.

## Traps and signals

When a process receives a signal, it can run a *trap*: a handler function that cleans up before the process exits. List all available signals with `kill -l`.

The most common trap cleans up temporary files when a script is interrupted:

```bash
TMPFILE=$(mktemp)

function cleanup {
    rm -f "$TMPFILE"
    echo "Cleaned up temporary files"
}

trap cleanup EXIT INT TERM

# the rest of the script can write to $TMPFILE safely
```

`EXIT` runs when the script exits for any reason. `INT` handles Ctrl+C. `TERM` handles `kill` commands.

A trap that catches Ctrl+C during a loop and exits gracefully:

```bash
trap 'echo "Interrupted -- exiting cleanly"; exit 1' INT

for host in "${HOSTS[@]}"; do
    echo "Scanning $host"
    nmap "$host"
done
```

## getopts for CLI argument parsing

`getopts` parses command-line flags in a standard format. The option string lists accepted flags. A colon after a flag means it takes an argument:

```bash
#!/bin/bash
# Usage: ./script.sh -c /dest -i -r /source

while getopts 'c:irR' opt; do
    case "${opt}" in
    c)
        COPY=YES
        DESTDIR="$OPTARG"
        ;;
    i)
        CASEMATCH='-i'
        ;;
    [Rr])
        RECURSIVE=YES
        ;;
    *)
        echo "Usage: $0 [-c destdir] [-i] [-r] source" >&2
        exit 2
        ;;
    esac
done

# shift parsed args so $1 now refers to the first non-flag argument
shift $(( OPTIND - 1 ))
SOURCE="$1"
```

After `getopts` processes all flags, `shift $(( OPTIND - 1 ))` resets the argument list so positional parameters start at the first non-flag argument.

## Process management

### ps command options

The following table shows common `ps` command options:

| Command   | Output                                    |
| :-------- | :---------------------------------------- |
| `ps`      | Processes running in the current terminal |
| `ps -f`   | Full listing for the current user         |
| `ps -ef`  | Full listing of all user processes        |
| `ps -A`   | All processes including kernel processes  |
| `ps aux`  | Wide listing with CPU and memory usage    |
| `ps auxw` | Wide listing sorted by CPU percentage     |

A script that checks whether a specific process is running and restarts it if not:

```bash
#!/bin/bash
SERVICE="nginx"

if ! pgrep -x "$SERVICE" > /dev/null; then
    echo "$SERVICE is not running, starting it" | tee -a /var/log/watchdog.log
    systemctl start "$SERVICE"
fi
```
