---
title: "Control flow"
weight: 40
description: >
  Arguments, conditionals, condition tests, case statements, and loops.
---

## Arguments and positional parameters

Shell arguments are referenced by number, prefixed with `$`. The following table describes the special variables:

| Variable        | Value                                                     |
| :-------------- | :-------------------------------------------------------- |
| `$0`            | Name of the script                                        |
| `$1`, `$2`, ... | First, second, and subsequent arguments                   |
| `$#`            | Number of arguments passed                                |
| `$*`            | All arguments as a single string                          |
| `$@`            | All arguments as separate words                           |
| `"$*"`          | All arguments joined into one string, separated by spaces |
| `"$@"`          | All arguments as individually quoted strings              |

A practical example that processes all arguments passed to a script:

```bash
#!/bin/bash
echo "Script: $0"
echo "Arguments: $#"
echo "First arg: $1"

for arg in "$@"; do
    echo "Processing: $arg"
done
```

### Default values

Provide fallback values for arguments that may not be set:

```bash
PATTERN=${1:-"PDF document"}    # use first arg, or default to "PDF document"
STARTDIR=${2:-.}                 # use second arg, or default to current directory
```

## Control flow

### if / elif / else

End every `if` block with `fi`. The condition can be a command (any exit status), a test in `[ ]`, or a compound test in `[[ ]]`:

```bash
if [ -f "$CONFIG" ]; then
    echo "Config found: $CONFIG"
elif [ -d "$CONFIG" ]; then
    echo "Error: $CONFIG is a directory, not a file" >&2
    exit 1
else
    echo "Config not found, using defaults"
fi
```

Use `[[ ]]` for pattern matching and string comparisons, which handles special characters more safely:

```bash
if [[ $HOSTNAME == web-* ]]; then
    echo "This is a web server"
fi
```

### Commands as conditions

Any command can serve as an `if` condition. The shell runs the command and tests its exit code: `0` is true, any non-zero value is false.

```bash
if touch testfile; then
    echo "Created testfile"
else
    echo "Could not create testfile" >&2
fi
```

Use it whenever a command's success or failure is all you need:

```bash
if mkdir -p /var/app/logs; then
    echo "Log directory ready"
fi

if grep -q "ERROR" /var/log/app.log; then
    echo "Errors found in log"
fi
```

Negate a condition with `!`:

```bash
if ! pgrep -x nginx > /dev/null; then
    echo "nginx is not running" >&2
    exit 1
fi
```

For simple one-liners, `&&` and `||` are shorter alternatives. `&&` runs the second command only if the first succeeds; `||` runs it only if the first fails:

```bash
mkdir -p /var/app/logs && echo "Directory ready"
pgrep -x nginx > /dev/null || echo "nginx is not running" >&2
```

### Condition tests

#### Numeric comparisons

The following table shows numeric test operators:

| Test        | True when                         |
| :---------- | :-------------------------------- |
| `n1 -eq n2` | n1 equals n2                      |
| `n1 -ne n2` | n1 does not equal n2              |
| `n1 -gt n2` | n1 is greater than n2             |
| `n1 -ge n2` | n1 is greater than or equal to n2 |
| `n1 -lt n2` | n1 is less than n2                |
| `n1 -le n2` | n1 is less than or equal to n2    |

For arithmetic comparisons, `(( ))` is more natural and does not require the `-eq` syntax:

```bash
if (( disk_usage > 90 )); then
    echo "Critical: disk almost full"
fi
```

#### String comparisons

The following table shows string test operators:

| Test           | True when                |
| :------------- | :----------------------- |
| `str1 = str2`  | str1 equals str2         |
| `str1 != str2` | str1 does not equal str2 |
| `str1 < str2`  | str1 sorts before str2   |
| `str1 > str2`  | str1 sorts after str2    |
| `-n str1`      | str1 is non-empty        |
| `-z str1`      | str1 is empty            |

#### File tests

The following table shows file test operators:

| Test        | True when                                    |
| :---------- | :------------------------------------------- |
| `-e file`   | File exists                                  |
| `-f file`   | File exists and is a regular file            |
| `-d file`   | File exists and is a directory               |
| `-r file`   | File exists and is readable                  |
| `-w file`   | File exists and is writable                  |
| `-x file`   | File exists and is executable                |
| `-s file`   | File exists and is not empty                 |
| `-L file`   | File exists and is a symbolic link           |
| `-O file`   | File exists and is owned by the current user |
| `f1 -nt f2` | f1 is newer than f2                          |
| `f1 -ot f2` | f1 is older than f2                          |

A real-world example combining file and string tests:

```bash
#!/bin/bash
LOCKFILE="/var/run/backup.lock"

if [ -f "$LOCKFILE" ]; then
    echo "Backup already running (lock file exists)" >&2
    exit 1
fi

touch "$LOCKFILE"
trap "rm -f $LOCKFILE" EXIT    # clean up lock file when the script exits
```

### case statements

The `case` statement evaluates an EXPRESSION once and compares it against each PATTERN in order. When a PATTERN matches, its commands run and the statement exits. If no PATTERN matches, nothing runs.

```bash
case EXPRESSION in
    PATTERN)
        commands
        ;;
    PATTERN | PATTERN)
        commands
        ;;
    *)
        default commands
        ;;
esac
```

Each block ends with `;;`. The catch-all `*)` matches anything and must come last. `esac` closes the statement.

#### Pattern types

| Pattern | Matches |
| :------ | :------ |
| `word` | The exact string `word` |
| `word1 \| word2` | Either `word1` or `word2` |
| `[Yy]` | Any single character listed in brackets |
| `*.log` | Any string ending in `.log` (glob) |
| `??` | Any two-character string |
| `*` | Anything; use as the catch-all default |

Match a specific environment and fall through to a default on unknown input:

```bash
case "$ENVIRONMENT" in
    production)
        DB_HOST="db.prod.internal"
        LOG_LEVEL="error"
        ;;
    staging)
        DB_HOST="db.staging.internal"
        LOG_LEVEL="warn"
        ;;
    development|dev)
        DB_HOST="localhost"
        LOG_LEVEL="debug"
        ;;
    *)
        echo "Unknown environment: $ENVIRONMENT" >&2
        exit 1
        ;;
esac
```

#### Command dispatch

`case` is the standard pattern for routing subcommands in a CLI tool. The EXPRESSION is the subcommand argument; each PATTERN is a command name:

```bash
case "$1" in
    start)
        systemctl start myapp
        ;;
    stop)
        systemctl stop myapp
        ;;
    restart)
        systemctl restart myapp
        ;;
    status)
        systemctl status myapp
        ;;
    help|-h|--help)
        usage
        ;;
    *)
        echo "Unknown command: $1" >&2
        usage
        exit 2
        ;;
esac
```

## Loops

Every loop uses a keyword for the loop type, the condition, then a `;do (...) done` clause.

### for loops

Iterate over a list of values:

```bash
for server in web-01 web-02 web-03; do
    echo "Deploying to $server"
    ssh "$server" "./deploy.sh"
done
```

Iterate over all files in a directory:

```bash
for file in $(ls /var/log/*.log | sort); do
    if [ -f "$file" ]; then
        echo "$file: $(wc -l < $file) lines"
    fi
done
```

Use C-style syntax for numeric loops:

```bash
for (( i=0; i < 10; i++ )); do
    echo "Attempt $i"
done
```

### while loops

Run while a condition is true:

```bash
RETRIES=0
MAX_RETRIES=5

while (( RETRIES < MAX_RETRIES )); do
    if ./health_check.sh; then
        echo "Service is healthy"
        break
    fi
    (( RETRIES++ ))
    echo "Attempt $RETRIES failed, retrying in 10 seconds"
    sleep 10
done
```

Read a file line by line with a while loop:

```bash
while read -r LINE; do
    echo "Processing: $LINE"
done < /etc/hosts
```

A while loop that renames files containing spaces by replacing spaces with underscores:

```bash
find /data -type f | while read -r file; do
    if [[ "$file" = *[[:space:]]* ]]; then
        mv "$file" "$(echo "$file" | tr ' ' '_')"
    fi
done
```

### until loops

Run until a condition becomes true (the opposite of `while`):

```bash
COUNT=0
until [ "$COUNT" -gt 5 ]; do
    echo "Count is: $COUNT"
    (( COUNT++ ))
done
```

### Loop control keywords

| Keyword    | Effect                                               |
| :--------- | :--------------------------------------------------- |
| `break`    | Exits the loop immediately                           |
| `continue` | Skips to the next iteration                          |
| `exit`     | Exits the entire script with an optional status code |
| `return`   | Returns from a function with an optional status code |

### break

Use `break` when you've found what you were looking for and continuing would waste work. It exits the loop immediately and resumes execution after `done`.

Search a list of servers for the first one that responds, then stop:

```bash
ACTIVE_SERVER=""

for server in web-01 web-02 web-03; do
    if ping -c1 -q "$server" &>/dev/null; then
        ACTIVE_SERVER="$server"
        break    # no need to check the rest
    fi
done

if [[ -n "$ACTIVE_SERVER" ]]; then
    echo "Deploying to $ACTIVE_SERVER"
else
    echo "No servers available" >&2
    exit 1
fi
```

`break` also works in `while` loops to exit once a condition is met, useful when polling for a process to start or a file to appear:

```bash
echo "Waiting for nginx..."
while true; do
    if pgrep -x nginx &>/dev/null; then
        echo "nginx is up"
        break
    fi
    sleep 2
done
```

### continue

Use `continue` to skip a specific iteration and move on to the next one. It keeps the loop running but avoids processing items that don't meet a condition.

Process log files but skip any that are empty:

```bash
for logfile in /var/log/*.log; do
    if [[ ! -s "$logfile" ]]; then
        continue    # skip empty files
    fi
    echo "Processing $logfile ($(wc -l < "$logfile") lines)"
    grep "ERROR" "$logfile" >> /tmp/errors.log
done
```

Skip hosts that don't respond instead of aborting the whole loop:

```bash
for host in "${HOSTS[@]}"; do
    if ! ping -c1 -q "$host" &>/dev/null; then
        echo "Skipping unreachable host: $host" >&2
        continue
    fi
    ssh "$host" "./deploy.sh"
done
```
