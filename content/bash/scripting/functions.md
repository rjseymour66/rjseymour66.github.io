---
title: "Functions"
weight: 50
description: >
  Defining functions, variable scope, positional parameters, and return values.
---

Functions group commands under a name so you can call them multiple times. Define a function before calling it:

```bash
function check_disk {
    local threshold=$1
    local usage
    usage=$(df -h / | awk 'NR==2 {print $5}' | tr -d '%')
    if (( usage > threshold )); then
        echo "WARNING: disk usage at ${usage}%" >&2
        return 1
    fi
    echo "Disk usage OK: ${usage}%"
    return 0
}

check_disk 80
```

Inside a function, positional parameters (`$1`, `$2`, ...) refer to the function's own arguments, not the script's arguments. `$0` still refers to the script name. `$#` is the number of arguments passed to the function.

Functions return a status code. To return a value other than a status, store it in a variable:

```bash
function get_ip {
    local RESULT
    RESULT=$(hostname -I | awk '{print $1}')
    echo "$RESULT"
}

MY_IP=$(get_ip)
echo "My IP: $MY_IP"
```

## Arguments

Functions receive arguments as positional parameters: `$1` for the first argument, `$2` for the second, and so on. `$#` holds the count. Assign arguments to named local variables at the top of the function so the code is self-documenting:

```bash
create_user() {
    local username=$1
    local group=$2

    useradd -m -G "$group" "$username"
    echo "Created $username in $group"
}

create_user alice developers
```

### Function params vs. script params

Inside a function, positional parameters refer to the function's own arguments, not the script's. `$1` is the first argument passed to the function call, not to the script. `$0` always refers to the script name regardless of where it appears:

```bash
#!/usr/bin/env bash

deploy() {
    local env=$1             # "staging" — the function's $1
    echo "Deploying to $env"
    echo "Script: $0"        # still the script name, not the function name
}

# Script called as: ./deploy.sh production
echo "Script arg: $1"        # production — the script's $1
deploy staging               # function receives its own separate $1
```

Output:

```
Script arg: production
Deploying to staging
Script: ./deploy.sh
```

To use the script's positional parameters inside a function, pass them explicitly:

```bash
process_all() {
    local item
    for item in "$@"; do
        echo "Processing $item"
    done
}

process_all "$@"    # forward all script arguments to the function
```

### Default values

Use parameter expansion to make arguments optional:

```bash
backup_dir() {
    local src=$1
    local dest=${2:-/tmp/backup}    # defaults to /tmp/backup if not supplied

    cp -r "$src" "$dest"
    echo "Backed up $src to $dest"
}

backup_dir /etc/app              # uses default dest
backup_dir /etc/app /mnt/backup  # uses supplied dest
```

### Validating required arguments

Check that required arguments are present before doing any work:

```bash
send_alert() {
    local message=$1
    local recipient=$2

    if [[ -z "$message" || -z "$recipient" ]]; then
        echo "send_alert: message and recipient are required" >&2
        return 1
    fi

    mail -s "Alert" "$recipient" <<< "$message"
}
```

### Iterating over all arguments

Use `"$@"` to process every argument the function receives:

```bash
ping_hosts() {
    local host
    for host in "$@"; do
        if ping -c1 -q "$host" &>/dev/null; then
            echo "$host: up"
        else
            echo "$host: unreachable" >&2
        fi
    done
}

ping_hosts web-01 web-02 db-01
```

`"$@"` expands each argument as a separately quoted word, preserving arguments that contain spaces.

## Returning values

### Return codes

`return` exits the current function and sets its exit status. Use `0` for success and a non-zero value for failure. The caller reads the status from `$?`:

```bash
is_even() {
    local n=$1
    (( n % 2 == 0 ))    # sets exit status: 0 if true, 1 if false
}

if is_even 4; then
    echo "4 is even"
fi

is_even 7
echo $?    # 1
```

`return` without an argument returns the exit status of the last command that ran in the function.

### Returning data with stdout

Functions cannot return strings directly. Print the value with `echo` and capture it with command substitution:

```bash
get_hostname() {
    echo "$(hostname -s)"
}

HOST=$(get_hostname)
echo "Running on $HOST"
```

Because command substitution runs the function in a subshell, variables set inside the function are not visible to the caller. Use this pattern when you only need the printed output.

### Returning data with a named variable

To return multiple values, or to avoid a subshell, write the result into a caller-supplied variable name using a nameref (`local -n`, Bash 4.3+):

```bash
get_load() {
    local -n _result=$1    # _result is an alias for the variable named in $1
    _result=$(uptime | awk '{print $NF}')
}

get_load load_avg
echo "Load: $load_avg"
```

`local -n` makes `_result` a nameref: reads and writes to `_result` affect the variable whose name was passed as `$1`. Prefix the internal name with `_` to avoid a collision if the caller passes a variable with the same name.
