---
title: "Best practices"
weight: 70
description: >
  Idioms, conventions, and structural patterns for readable, reliable bash scripts.
---

Writing a bash script that works once is straightforward. Writing one that works reliably, survives edge cases, and can be understood six months later is harder. The practices on this page address the most common sources of fragile, hard-to-maintain scripts.

## Structure a script consistently

A well-structured script is easier to read, debug, and extend. Use this layout as a starting point:

```bash
#!/usr/bin/env bash
set -euo pipefail

# --- Constants -----------------------------------------------------------
readonly SCRIPT_NAME=$(basename "$0")
readonly LOG_FILE="/var/log/myapp.log"

# --- Functions -----------------------------------------------------------
usage() {
    cat <<EOF
Usage: $SCRIPT_NAME [OPTIONS] <target>

Options:
  -v    Enable verbose output
  -h    Show this help message
EOF
    exit 0
}

log() {
    echo "$(date +%T) $*" | tee -a "$LOG_FILE"
}

# --- Main ----------------------------------------------------------------
main() {
    local target=${1:-}
    [[ -z "$target" ]] && { echo "$SCRIPT_NAME: target required" >&2; usage; }

    log "Starting run for $target"
    # ... work here ...
    log "Done"
}

main "$@"
```

This layout puts all logic inside functions and calls `main "$@"` at the end. Keeping logic in functions means you can source the script in tests without executing anything, and you can call individual functions in isolation.

## Set safety options at the top

Add `set -euo pipefail` immediately after the shebang. These three options catch a large class of bugs:

| Option        | Effect                                                         |
| :------------ | :------------------------------------------------------------- |
| `-e`          | Exit immediately when any command returns a non-zero status    |
| `-u`          | Treat unset variables as errors                                |
| `-o pipefail` | A pipeline fails if any command in it fails, not just the last |

Without `pipefail`, a command like `grep "error" /var/log/app.log | wc -l` succeeds even if `grep` finds nothing and returns exit code 1.

When you need a command to fail without aborting the script, use `||` to handle the failure explicitly:

```bash
grep "ERROR" "$LOGFILE" || true          # ignore non-zero exit from grep
count=$(wc -l < "$FILE") || count=0      # fall back to 0 on failure
```

## Mark constants as readonly

Declare values that should never change with `readonly`. This prevents accidental reassignment and makes intent clear:

```bash
readonly CONFIG_DIR="/etc/myapp"
readonly MAX_RETRIES=5
readonly SCRIPT_NAME=$(basename "$0")
```

## Naming conventions

Consistent naming makes variable scope visible at a glance:

| Convention            | Use for                                            |
| :-------------------- | :------------------------------------------------- |
| `UPPER_CASE`          | Global variables, constants, environment variables |
| `lower_case`          | Local variables inside functions                   |
| `_leading_underscore` | Private functions not meant to be called directly  |

```bash
readonly DB_HOST="db.internal"       # global constant

function connect {
    local host=$1                    # local to the function
    local port=${2:-5432}
    psql -h "$host" -p "$port"
}
```

## Always quote variable expansions

An unquoted variable undergoes word splitting and glob expansion. This causes silent, intermittent bugs that are hard to reproduce:

```bash
# Dangerous: breaks if $FILE contains spaces or glob characters
cp $FILE /backup/

# Safe: the value is treated as a single token
cp "$FILE" /backup/
```

Quote every variable expansion unless you specifically want word splitting. If you do want word splitting, use `"${array[@]}"` instead. It expands each element as a separately quoted word, which is the correct form when passing an array of arguments.

## `[[ ]]` instead of `[ ]` for conditionals

`[[ ]]` is a bash built-in that handles edge cases more safely than the POSIX `[ ]` test command:

```bash
# [ ] fails or behaves oddly with empty variables, spaces, and operators
if [ $count -gt 0 ]; then ...      # breaks if count is unset

# [[ ]] handles these safely
if [[ $count -gt 0 ]]; then ...    # works even if count is empty
if [[ $filename == *.log ]]; then  # glob matching works natively
if [[ $input =~ ^[0-9]+$ ]]; then  # regex matching with =~
```

Use `(( ))` for arithmetic comparisons, which is cleaner than `-eq` and friends:

```bash
if (( retries > MAX_RETRIES )); then
    echo "Too many retries" >&2
    exit 1
fi
```

## Write errors to stderr

Separate errors from normal output by writing to stderr with `>&2`. This keeps error messages visible even when stdout is redirected to a file or pipe, and it makes pipelines work correctly:

```bash
if [[ ! -f "$CONFIG" ]]; then
    echo "$SCRIPT_NAME: config file not found: $CONFIG" >&2
    exit 1
fi
```

Include the script name in error messages so the source is obvious when the script is called from another script or a cron job.

## Meaningful exit codes

Exit with `0` on success and a non-zero code on failure. Exit code `1` is a general error; codes `2` and above can signal specific failure types. Document them in your `usage` function if callers need to distinguish them:

```bash
readonly E_USAGE=2
readonly E_NO_CONFIG=3
readonly E_NETWORK=4

[[ $# -lt 1 ]] && { usage; exit $E_USAGE; }
[[ ! -f "$CONFIG" ]] && { echo "Config not found" >&2; exit $E_NO_CONFIG; }
```

## Clean up with trap

Use `trap` to clean up temporary files and lock files whether the script exits normally, fails, or is interrupted:

```bash
readonly TMPFILE=$(mktemp)
readonly LOCKFILE="/var/run/myapp.lock"

cleanup() {
    rm -f "$TMPFILE" "$LOCKFILE"
}
trap cleanup EXIT

# Script can now write to $TMPFILE freely — cleanup always runs
```

`EXIT` fires on any exit. Add `INT` and `TERM` if you need to handle Ctrl+C or `kill` separately.

## Functions with a single responsibility

Each function should do one thing. Functions that do too much are hard to test and reuse. Prefer functions that return a status code and communicate results through variables or stdout:

```bash
# Returns 0 if the service is up, 1 if it is down
is_running() {
    local service=$1
    pgrep -x "$service" > /dev/null
}

# Prints the primary IP address
get_ip() {
    hostname -I | awk '{print $1}'
}

if is_running nginx; then
    echo "nginx is up at $(get_ip)"
fi
```

Declare every variable inside a function as `local`. Without `local`, the variable leaks into the global scope and can silently overwrite another variable with the same name.

## Validate input early

Check all required arguments and preconditions at the start of `main`, before doing any real work. This fails fast and produces clear error messages instead of cryptic failures deep in the script:

```bash
main() {
    local source_dir=${1:-}
    local dest_dir=${2:-}

    [[ -z "$source_dir" || -z "$dest_dir" ]] && {
        echo "Usage: $SCRIPT_NAME <source> <dest>" >&2
        exit 2
    }
    [[ ! -d "$source_dir" ]] && {
        echo "$SCRIPT_NAME: not a directory: $source_dir" >&2
        exit 1
    }
    [[ ! -w "$dest_dir" ]] && {
        echo "$SCRIPT_NAME: not writable: $dest_dir" >&2
        exit 1
    }

    # ... proceed with validated inputs
}
```

## Common pitfalls

### Don't parse `ls`

`ls` output is designed for human eyes, not programmatic parsing. Filenames can contain spaces, newlines, or special characters that break word splitting. Use globs or `find` instead:

```bash
# Fragile: breaks on filenames with spaces
for file in $(ls /var/log/*.log); do ...

# Safe: the shell expands the glob directly
for file in /var/log/*.log; do ...

# Safe: find handles all filenames correctly
find /var/log -name "*.log" -print0 | while IFS= read -r -d '' file; do ...
```

### Don't use backticks for command substitution

Backtick syntax is harder to read and cannot be nested without escaping. Use `$()` instead:

```bash
# Hard to read and nest
result=`echo \`date\``

# Clear and nestable
result=$(echo $(date))
```

### Don't ignore `read` failures

When reading a file with a `while read` loop, the last line may be silently skipped if the file does not end with a newline. Guard against this:

```bash
while IFS= read -r line || [[ -n "$line" ]]; do
    echo "Line: $line"
done < "$FILE"
```

`IFS=` prevents `read` from stripping leading and trailing whitespace. `-r` prevents backslash interpretation.

### Prefer built-ins over subshells

Every `$(...)` call spawns a subshell, which has measurable overhead inside tight loops. Replace external commands with parameter expansion where possible:

```bash
# Spawns a subshell and an external process on every iteration
for file in "${FILES[@]}"; do
    base=$(basename "$file")
    ...
done

# Pure bash — no subshell
for file in "${FILES[@]}"; do
    base=${file##*/}
    ...
done
```

Similarly, avoid `echo "$var" | grep` when `[[ $var =~ pattern ]]` does the same thing without a subshell.

## Use `local -r` for function constants

Combine `local` and `readonly` inside functions to lock down values that should not change:

```bash
function backup_dir {
    local -r src=$1
    local -r dest="/backup/$(date +%Y%m%d)"
    rsync -a "$src" "$dest"
}
```

## Add a usage function to every script

A `usage` function documents the script's interface and gives users a fast way to get help. Call it when the user passes `-h` or provides invalid arguments:

```bash
usage() {
    cat <<EOF
Usage: $(basename "$0") [OPTIONS] <file>

Processes <file> and writes results to stdout.

Options:
  -n <count>   Number of lines to process (default: all)
  -v           Verbose output
  -h           Show this help
EOF
}

while getopts 'n:vh' opt; do
    case $opt in
        n) COUNT=$OPTARG ;;
        v) VERBOSE=1 ;;
        h) usage; exit 0 ;;
        *) usage; exit 2 ;;
    esac
done
```

## Test with `bash -n` before running

`bash -n` checks syntax without executing the script. Run it before deploying or sharing a script:

```bash
bash -n my_script.sh
```

Run with `bash -x` to trace execution line by line when debugging unexpected behavior. Wrap individual sections in `set -x` / `set +x` to limit the trace to the part that matters.

## Where to store scripts

Store personal scripts in `~/bin` or `~/.local/bin`. Both are conventional locations for user scripts and keep them separate from system binaries. Use `/usr/local/bin` for scripts that every user on the system should be able to run.

| Directory        | Use for                                      |
| :--------------- | :------------------------------------------- |
| `~/bin`          | Personal scripts, available only to you      |
| `~/.local/bin`   | Personal scripts (XDG convention, preferred on modern distros) |
| `/usr/local/bin` | System-wide scripts available to all users   |

Create `~/bin` if it does not exist, then make the script executable:

```bash
mkdir -p ~/bin
cp my_script.sh ~/bin/my_script
chmod +x ~/bin/my_script
```

## Adding scripts to PATH

`PATH` is a colon-separated list of directories bash searches when you run a command by name. Adding your script directory to `PATH` lets you run your scripts from anywhere without typing the full path.

Add `~/bin` to `PATH` in `~/.bashrc` so it is available in every new terminal session:

```bash
export PATH="$HOME/bin:$PATH"
```

Reload your shell to apply the change immediately:

```bash
source ~/.bashrc
```

Confirm bash can find the script:

```bash
which my_script
```

On Ubuntu 20.04 and later, `~/.local/bin` is added to `PATH` automatically if it exists. Create the directory and it will be available in your next login session without any changes to `.bashrc`.

## Common aliases

Define aliases in `~/.bashrc` so they are available in every interactive session.

### System administration

```bash
# Navigation
alias ..='cd ..'                # go up one directory
alias ...='cd ../..'            # go up two directories
alias ll='ls -lah'              # long list with hidden files, human-readable sizes
alias la='ls -A'                # list all files except . and ..

# Safety
alias rm='rm -i'                # prompt before deleting
alias cp='cp -i'                # prompt before overwriting
alias mv='mv -i'                # prompt before overwriting
alias mkdir='mkdir -pv'         # create parent directories as needed, print each one created

# System information
alias df='df -h'                # disk usage in human-readable units
alias du='du -h'                # directory sizes in human-readable units
alias free='free -h'            # RAM and swap in human-readable units
alias ports='ss -tulnp'         # show all listening TCP/UDP ports with associated PIDs
alias ps='ps aux'               # all running processes with user, CPU, and memory stats

# Logs
alias syslog='sudo tail -f /var/log/syslog'     # stream the system log in real time
alias authlog='sudo tail -f /var/log/auth.log'  # stream login and authentication events

# Convenience
alias grep='grep --color=auto'                        # highlight matches in output
alias reload='source ~/.bashrc'                       # apply .bashrc changes without restarting the shell
alias myip='curl -s ifconfig.me'                      # print your public IP address
alias update='sudo apt update && sudo apt upgrade -y' # fetch and install all available updates
```

### Penetration testing

```bash
# Nmap
alias nmap='nmap -v'                    # enable verbose output on every scan
alias nmapq='nmap -T4 -F'              # fast scan of the 100 most common ports
alias nmapf='nmap -sV -sC -O -A'       # full scan: service versions, default scripts, OS detection
alias nmapall='nmap -p- -T4'           # scan all 65535 ports at aggressive timing

# Listeners
alias listen='nc -lvnp 4444'           # open a netcat listener on port 4444 for reverse shells

# Web
alias myip='curl -s ifconfig.me'       # print your public IP address
alias pubip='curl -s ipinfo.io/ip'     # alternate public IP lookup
alias headers='curl -sI'              # print HTTP response headers only, no body

# Recon
alias hosts='cat /etc/hosts'           # print local hostname-to-IP mappings
alias routes='ip route show'           # print the kernel routing table
alias arp='arp -a'                     # print the ARP cache showing known hosts on the local network
```
