---
title: "Functions"
weight: 50
description: >
  Defining functions, variable scope, positional parameters, and return values.
---

## Functions

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
