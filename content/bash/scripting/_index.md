---
title: "Writing bash scripts"
weight: 20
description: >
  Script structure, variables, control flow, functions, arrays, math, and error handling.
---

Scripts turn a manual sequence of steps into a repeatable, automated process. A script that backs up a database, sends an alert when disk space runs low, or renames a thousand files eliminates repetitive work and removes the chance of human error from routine operations. When something does go wrong, a well-written script produces clear exit codes and error messages that make the failure easy to diagnose.

This page covers everything you need to write reliable bash scripts: from the first line of the file to argument parsing, control flow, math, and signal handling.

## Script structure

A bash script is a plain text file with execute permissions. The first line, called the *shebang*, tells the operating system which interpreter to run:

```bash
#!/bin/bash
```

An alternative form locates the bash executable via `PATH`, which improves portability across systems where bash lives in a non-standard location:

```bash
#!/usr/bin/env bash
```

Store scripts that you want available system-wide in `/usr/local/bin`. Store personal scripts in `~/bin` and add that directory to your `PATH`.

## Running a script

Before you can run a script, set the execute permission with `chmod`:

```bash
chmod +x my_script.sh
./my_script.sh
```

The `./` prefix is required because the current directory is not in `PATH` by default. This is a deliberate security decision: it prevents a malicious file named `ls` or `cp` from running in place of the real command.

To run a script in a specific shell without setting execute permissions, pass it as an argument:

```bash
bash my_script.sh
```

## Execution flags

Pass flags between `bash` and the script name to change how the shell runs the script. You can combine flags into a single argument:

```bash
bash -xv script.sh      # trace + verbose
bash -xvn script.sh     # trace + verbose + syntax check only
```

The following table lists the most useful flags:

| Flag          | Name             | Effect                                                                                               |
| :------------ | :--------------- | :--------------------------------------------------------------------------------------------------- |
| `-n`          | No execute       | Parse the script and check syntax without running any commands                                       |
| `-x`          | Xtrace           | Print each command after expansion, prefixed with `+`, before executing it                           |
| `-v`          | Verbose          | Print each line as bash reads it, before expansion                                                   |
| `-e`          | Exit on error    | Exit immediately when any command returns a non-zero status                                          |
| `-u`          | Unset as error   | Treat references to unset variables as errors and exit                                               |
| `-o pipefail` | Pipeline failure | Return the exit status of the first failed command in a pipeline, not the last                       |
| `-i`          | Interactive      | Run the shell in interactive mode, loading `~/.bashrc` and enabling job control                      |
| `-r`          | Restricted       | Run in restricted mode. See [Restricted shell]({{< relref "/bash/shell-basics#restricted-shell" >}}) |
| `-s`          | Stdin            | Read commands from stdin; any remaining arguments become positional parameters                       |

### -n: check syntax without running

Use `-n` to catch syntax errors before you run a script in production. Bash parses the entire file and reports errors but executes nothing:

```bash
bash -n script.sh
script.sh: line 12: syntax error near unexpected token `fi'
```

`-n` does not catch runtime errors such as a missing file or an undefined variable. It only validates that the script parses correctly.

### -x: trace execution

`-x` prints each command after the shell expands it, prefixed with `+`. This shows you exactly what bash runs, including the values of any variables:

```bash
bash -x deploy.sh
+ DEST=/var/www/html
+ rsync -av ./dist/ /var/www/html
+ systemctl reload nginx
```

Use `-x` when a script produces unexpected results and you need to see the expanded commands rather than the source lines.

### -v: verbose output

`-v` prints each line as bash reads it, before variable expansion or substitution. This is useful for tracing logic errors in scripts that use complex expansions:

```bash
bash -v script.sh
for file in "$@"; do
+ for file in report.txt notes.txt
```

### -e and -u: fail fast

`-e` exits the script as soon as any command fails. `-u` exits when the script references a variable that has not been set. Together they catch two of the most common silent failures in shell scripts:

```bash
bash -eu deploy.sh
```

You can also set these inside the script itself with `set`, which is the more common pattern for production scripts:

```bash
#!/bin/bash
set -eu
```

### -o pipefail: catch pipeline errors

By default, a pipeline returns the exit status of its last command, even if an earlier command failed. `-o pipefail` changes this so the pipeline fails if any command in it fails:

```bash
bash -o pipefail script.sh
```

Combine it with `-e` and `-u` for the strictest error handling:

```bash
bash -euo pipefail script.sh
```

Or set it at the top of the script:

```bash
#!/bin/bash
set -euo pipefail
```

## Debugging

Add the `-x` flag to the shebang to print each command as the shell executes it. The shell prefixes each command with `+`:

```bash
#!/bin/bash -x
```

For example, running a script with debugging enabled produces output like this:

```bash
+ DIR=.
+ find . -type f
+ read file
+ [[ ./config.sh = *[[:space:]]* ]]
```

You can also enable debugging for a specific section of a script by wrapping it:

```bash
set -x          # enable debugging
mv "$old" "$new"
set +x          # disable debugging
```

Two other useful options to add to scripts are `set -e`, which exits immediately on any error, and `set -u`, which treats unset variables as errors. Together, they catch many common mistakes:

```bash
#!/bin/bash
set -eu
```

## Displaying output

`echo` prints a message followed by a newline. `printf` gives you more control over formatting:

```bash
echo "Backup started"
echo                    # print a blank line

printf "%-15s %8d\n" "$hostname" "$count"
```

## Quoting

Single quotes, double quotes, and backticks have different effects on how the shell evaluates content inside them:

- *Single quotes* (`''`): Every character is literal. The shell does not expand variables or interpret escape sequences.
- *Double quotes* (`""`): Allow variable expansion (`$`), command substitution (`` ` ` `` or `$()`), and escape sequences (`\`).
- *Backticks* (`` ` ` ``): Execute a command and substitute its output. Prefer `$()` instead -- it is easier to read and supports nesting.

For example, compare the three behaviors:

```bash
BASH_VAR="hello"
echo '$BASH_VAR'               # $BASH_VAR (literal)
echo "$BASH_VAR"               # hello (expanded)
echo "Today is $(date +%A)"    # Today is Monday (command substituted)
```

### ANSI-C escape characters

When you need special characters in a string, use the `$'...'` syntax to enable ANSI-C escape sequences:

| Sequence | Value                      |
| :------: | :------------------------- |
|   `\n`   | Newline                    |
|   `\t`   | Horizontal tab             |
|   `\r`   | Carriage return            |
|   `\a`   | Bell (alert)               |
|   `\b`   | Backspace                  |
|   `\\`   | Backslash                  |
|  `\xnn`  | Hex value of a character   |
|  `\nnn`  | Octal value of a character |

For example, embed a newline and a hex character in a string:

```bash
echo $'Line one\nLine two'
echo $'email: user\x40example.com'    # \x40 is the hex code for @
```

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

| Code | Meaning | Common source |
| :--- | :------ | :------------ |
| `0` | Success | Any command that completed without error |
| `1` | General error | Catchall for miscellaneous failures (`grep` with no match, logic errors) |
| `2` | Misuse of shell built-in or invalid argument | `ls` on a nonexistent path, missing required argument |
| `126` | Command found but not executable | File exists but lacks execute permission |
| `127` | Command not found | Typo in command name, or command not in `PATH` |
| `128` | Invalid argument to `exit` | `exit 3.14` or `exit -1` |
| `128+N` | Fatal signal N | `130` = interrupted with Ctrl-C (signal 2), `137` = killed with SIGKILL (signal 9) |
| `255` | Exit status out of range | Script called `exit` with a value greater than 255 |

Codes `130` and `137` are the ones you encounter most often in practice. `130` means the user pressed Ctrl-C; `137` means the process was force-killed, usually by the OOM killer or a `kill -9`.

### Setting a script's exit code

Call `exit` with a number to terminate the script and return that code to the caller. Any value from `0` to `255` is valid:

```bash
exit 0    # success
exit 1    # general failure
exit 2    # invalid argument or usage error
```

The caller — another script, a CI pipeline, or a shell — reads the exit code from `$?` and decides what to do next. A non-zero code causes `&&` chains to stop and `set -e` scripts to abort.

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

Scripts exit with the status of their last command by default. Control this explicitly with the `exit` command to signal specific error conditions:

```bash
#!/bin/bash
if [ ! -d "$1" ]; then
    echo "Error: directory '$1' does not exist" >&2
    exit 1
fi
# continue processing...
exit 0
```

Writing errors to stderr (`>&2`) keeps them separate from normal output and makes them visible even when stdout is redirected.

## Variables

### User-defined variables

Define a variable with `=` and no spaces on either side. Reference it with `$`:

```bash
days=10
guest="Alice"
logfile="/var/log/app.log"

echo "$guest checked in $days days ago"
echo "Watching $logfile"
```

### Environment variables

Run `export` to make a variable available to child processes. Scripts launched from your terminal can then read it:

```bash
export BACKUP_DIR="/mnt/backups"
./run_backup.sh    # script reads $BACKUP_DIR
```

Run `set` to display all global variables. Run `printenv` to display only exported environment variables.

### Unset a variable

`unset` removes a variable from the current shell environment. After unsetting, any reference to the variable returns an empty string — unless you have `set -u` enabled, in which case bash treats it as an error:

```bash
tmpdir="/tmp/build"
echo "$tmpdir"    # /tmp/build

unset tmpdir
echo "$tmpdir"    # (empty)
```

Use `unset` to release a sensitive value after you no longer need it, or to ensure a script does not inherit a variable that a parent process may have set:

```bash
unset DB_PASSWORD          # clear credential after use
unset -f my_function       # remove a function definition
```

Pass `-v` to remove a variable explicitly (the default), or `-f` to remove a function.

### Local variables in functions

Variables defined inside a function are global by default. Declare them with `local` to restrict their scope:

```bash
function report {
    local count=0
    count=$(grep -c "ERROR" "$1")
    echo "Found $count errors in $1"
}
```

## Parameter expansion

Parameter expansion is the mechanism bash uses to read, modify, and substitute variable values. The `${}` syntax tells bash to expand the parameter named inside the braces.

{{< admonition "Expansion syntax" tip >}}
You can print a variable name with either `${VAR}` or `$VAR`. Prefer the `${VAR}` syntax because it makes code less prone to interpretation.
{{< /admonition >}}

Braces are optional for simple variable references, but required when the variable name is immediately followed by characters that would otherwise be parsed as part of the name:

```bash
NAME="world"
echo "$NAME"          # world
echo "${NAME}wide"    # worldwide  — braces delimit the variable name
echo "$NAMEwide"      # empty — bash looks for NAMEwide, which is unset
```

### Default and assignment values

Use these forms to handle unset or empty variables without `if` blocks:

| Syntax            | Behavior                                                               |
| :---------------- | :--------------------------------------------------------------------- |
| `${VAR:-default}` | Returns `default` if `VAR` is unset or empty                           |
| `${VAR:=default}` | Sets `VAR` to `default` and returns it if unset or empty               |
| `${VAR:?message}` | Exits with `message` as the error if `VAR` is unset or empty           |
| `${VAR:+value}`   | Returns `value` if `VAR` is set and non-empty; otherwise returns empty |

```bash
LOGDIR=${LOG_DIR:-/var/log}                     # fall back to /var/log
: ${TMPDIR:=/tmp}                               # set TMPDIR if not already set
echo "${CONFIG:?Config file is required}"       # exit with error if CONFIG is empty
EXTRA=${DEBUG:+--verbose}                       # add --verbose only if DEBUG is set
```

### String length

`${#VAR}` returns the number of characters in the value:

```bash
FILE="report.pdf"
echo ${#FILE}    # 10
```

### Substrings

`${VAR:offset:length}` extracts a substring starting at `offset` (zero-indexed). Omit `length` to extract to the end of the string:

```bash
DATE="2024-03-15"
echo ${DATE:0:4}    # 2024
echo ${DATE:5:2}    # 03
echo ${DATE:8}      # 15
```

### Pattern removal

Use these to strip prefixes or suffixes without calling `sed` or `awk`:

| Syntax            | Behavior                                               |
| :---------------- | :----------------------------------------------------- |
| `${VAR#pattern}`  | Removes the shortest match of `pattern` from the left  |
| `${VAR##pattern}` | Removes the longest match of `pattern` from the left   |
| `${VAR%pattern}`  | Removes the shortest match of `pattern` from the right |
| `${VAR%%pattern}` | Removes the longest match of `pattern` from the right  |

```bash
FILE="/var/log/app.log"
echo ${FILE##*/}      # app.log   — strip everything up to the last /
echo ${FILE%/*}       # /var/log  — strip everything after the last /
echo ${FILE%.log}     # /var/log/app  — strip the .log suffix
```

### Substitution

`${VAR/old/new}` replaces the first match of `old` with `new`. Use `//` to replace all matches:

```bash
PATH_STR="/usr/local/bin:/usr/bin:/bin"
echo ${PATH_STR/bin/BIN}     # /usr/local/BIN:/usr/bin:/bin  — first match only
echo ${PATH_STR//bin/BIN}    # /usr/local/BIN:/usr/BIN:/BIN  — all matches
```

### Case conversion (Bash 4+)

| Syntax     | Effect                        |
| :--------- | :---------------------------- |
| `${VAR^}`  | Uppercase the first character |
| `${VAR^^}` | Uppercase all characters      |
| `${VAR,}`  | Lowercase the first character |
| `${VAR,,}` | Lowercase all characters      |

```bash
NAME="alice"
echo ${NAME^}     # Alice
echo ${NAME^^}    # ALICE
```

## Text manipulation

Three built-in expansion features let you manipulate string values without calling external tools:

*Globbing* lets you match multiple filenames with wildcard patterns in your script.

*Parameter expansion* lets you extract substrings, apply default values, and transform variable content. For example, `${VAR#pattern}` removes the shortest match of `pattern` from the left, and `${VAR%pattern}` removes from the right.

*String slicing* lets you remove or replace substrings:

```bash
STRING="user|admin|root"
FIRST=${STRING%%|*}        # removes everything to the right of the first |: "user"
REST=${STRING#*|}          # removes the first field and |: "admin|root"
CLEAN=${STRING//|/,}       # replaces all | with ,: "user,admin,root"
```

### Case conversion

Convert a string between upper and lowercase with `tr`:

```bash
upper=$(echo "$var" | tr '[a-z]' '[A-Z]')
lower=$(echo "$var" | tr '[A-Z]' '[a-z]')
```

The `typeset` command enforces case at assignment time. The value is always stored in the specified case regardless of what you assign:

```bash
typeset -u HOSTNAME_UPPER
HOSTNAME_UPPER="web-01"
echo $HOSTNAME_UPPER    # WEB-01

typeset -l env_name
env_name="PRODUCTION"
echo $env_name          # production
```

## Arrays

Bash arrays store multiple values indexed by number:

```bash
SERVERS=("web-01" "web-02" "db-01" "cache-01")

echo ${SERVERS[0]}          # web-01 (first element)
echo ${#SERVERS[@]}         # 4 (number of elements)
echo ${SERVERS[@]}          # web-01 web-02 db-01 cache-01 (all elements, separate words)
echo ${SERVERS[*]}          # web-01 web-02 db-01 cache-01 (all elements, single string)

for server in "${SERVERS[@]}"; do
    ssh "$server" uptime
done
```

### Delete array elements

Use `unset` to remove a single element by index or to delete the entire array:

```bash
SERVERS=("web-01" "web-02" "db-01" "cache-01")

unset SERVERS[1]            # remove web-02
echo ${SERVERS[@]}          # web-01 db-01 cache-01

unset SERVERS               # delete the entire array
echo ${SERVERS[@]}          # (empty)
```

Removing an element leaves a gap in the index — the remaining elements do not shift. If your code iterates by index, account for the missing slot. Iterating with `"${SERVERS[@]}"` skips empty slots automatically.

To remove an element and repack the array into contiguous indices, reassign it:

```bash
SERVERS=("${SERVERS[@]}")
```

### Reassign element values

Assign a new value to an element by referencing its index directly:

```bash
SERVERS=("web-01" "web-02" "db-01" "cache-01")

SERVERS[1]="web-99"
echo ${SERVERS[@]}    # web-01 web-99 db-01 cache-01
```

Assign to an index beyond the current length to append an element:

```bash
SERVERS[4]="backup-01"
echo ${SERVERS[@]}    # web-01 web-99 db-01 cache-01 backup-01
```

To append without tracking the length, use `+=`:

```bash
SERVERS+=("monitor-01")
echo ${SERVERS[@]}    # web-01 web-99 db-01 cache-01 backup-01 monitor-01
```

### Associative arrays (Bash 4.0+)

Associative arrays store values indexed by string keys:

```bash
declare -A PORTS
PORTS["http"]=80
PORTS["https"]=443
PORTS["ssh"]=22

echo ${PORTS["https"]}          # 443

for service in "${!PORTS[@]}"; do   # ${!array[@]} gets the keys
    echo "$service: ${PORTS[$service]}"
done
```

### Reading a file into an array

Read a file line by line into an array with `readarray`. The `-t` flag strips the trailing newline from each element:

```bash
readarray -t LINES < /etc/hosts

for line in "${LINES[@]}"; do
    echo "$line"
done
```

## User input

Read input from the user with the `read` command:

```bash
#!/bin/bash
echo -n "Enter the target host: "
read TARGET_HOST

echo -n "Enter username: "
read -s USERNAME    # -s suppresses echoing (useful for passwords)

echo "Connecting to $USERNAME@$TARGET_HOST"
```

Read multiple values in one command, where `read` assigns each word to the corresponding variable:

```bash
read HOST PORT <<< "db.internal 5432"
echo "$HOST on port $PORT"
```

### -r: read raw input

By default, `read` treats backslashes as escape characters. A trailing `\` continues the line, and sequences like `\t` are interpreted. Pass `-r` to treat backslashes as literal characters.

Pair it with `IFS=` to prevent `read` from stripping leading and trailing whitespace. Together they preserve every character in the line exactly as it appears in the file:

```bash
while IFS= read -r line; do
    echo "$line"
done < input.txt
```

**Real-world use case:** parsing `/etc/passwd` to extract every user's home directory. Home directory paths are safe here, but a file owned by an attacker or user could contain backslash sequences that silently mangle the data without `-r`:

```bash
while IFS=: read -r username _ _ _ _ homedir shell; do
    echo "$username: $homedir ($shell)"
done < /etc/passwd
```

`IFS=:` splits each line on `:` instead of whitespace, assigning each field to a variable. `_` discards fields you don't need. `-r` ensures a path like `/home/user\name` is never mangled.

## Here documents

A *here document* supplies multi-line input to a command without requiring a separate file. Everything between the opening and closing delimiter is treated as stdin. The delimiter can be any word — `EOF` is conventional but not required.

### Writing a login banner

Use a here document to write a multi-line login banner to `/etc/motd` in a single command. This is common in provisioning scripts that need to stamp every new server with a standard message:

```bash
cat << EOF > /etc/motd
Welcome to $(hostname)
Last updated: $(date)
Unauthorized access is prohibited.
EOF
```

Because the opening delimiter is unquoted, bash expands `$(hostname)` and `$(date)` on the local machine before writing the file. The resulting `/etc/motd` contains the resolved values, not the expressions.

### Running multi-line commands over SSH

Use a quoted delimiter to send a block of commands to a remote shell without copying a script file first. This is useful for one-off health checks or remote setup steps:

```bash
ssh user@remote bash << 'EOF'
set -eu
echo "Running on $(hostname)"
df -h /
uptime
EOF
```

Quoting the opening delimiter (`'EOF'`) tells the local shell to pass the body as literal text. The remote shell receives and expands `$(hostname)` on its end, so you see the remote hostname rather than the local one.

If you forget the quotes, the local shell expands the variables before sending anything over SSH. That is rarely what you want and produces subtle bugs that are hard to trace.

### Writing root-owned config files

When a script needs to write to a path owned by root, pipe the here document through `sudo tee` instead of redirecting with `>`. Redirection runs as the current user, so it fails on protected paths. `tee` runs as root and handles the write:

```bash
sudo tee /etc/app/config.conf << 'EOF'
[server]
host = 0.0.0.0
port = 8080
log_level = warn
EOF
```

Use a quoted delimiter here to prevent variable expansion — config files typically contain literal values, not shell expressions.

## Command substitution

The `$()` syntax tells bash to run the command inside the parentheses in a subshell, then replace the entire expression with that command's stdout. Use it anywhere you need the output of a command as a value—in a variable assignment, inside a string, or as an argument to another command.

```bash
TODAY=$(date +%Y-%m-%d)
DISK_USAGE=$(df -h / | awk 'NR==2 {print $5}')
echo "Disk usage on $TODAY: $DISK_USAGE"
```

Nest command substitutions to build complex single-line expressions:

```bash
echo "Today is $(echo $(date +%A) | tr a-z A-Z)!"    # Today is MONDAY!
```

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

The `case` statement matches a value against patterns and runs the corresponding commands. It is the bash equivalent of `switch` in other languages. End each case block with `;;` and the default case with `*`:

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

## Loops

End every loop with `done`.

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

The following table describes loop control keywords:

| Keyword    | Effect                                               |
| :--------- | :--------------------------------------------------- |
| `break`    | Exits the loop immediately                           |
| `continue` | Skips to the next iteration                          |
| `exit`     | Exits the entire script with an optional status code |
| `return`   | Returns from a function with an optional status code |

## Math

### Integer arithmetic

The `let` command evaluates an arithmetic expression:

```bash
let SUM=$1+$2
let RESULT=COUNT*2
```

Double parentheses `(( ))` provide arithmetic evaluation without needing `$` to reference variables:

```bash
(( result = COUNT * i / maxcount ))
echo $result
```

Arithmetic expansion `$(( ))` evaluates an expression and substitutes the result:

```bash
echo $(( $1 + $2 ))
echo $(( 2 ** 10 ))    # 1024
```

Declare a variable as an integer with `declare -i` to prevent non-integer assignments:

```bash
declare -i total
total=0
total+=5
```

Declare local integers inside functions:

```bash
function calculate {
    local -i result
    result=$(( $1 * $2 ))
    echo $result
}
```

### Arithmetic with expr

`expr` evaluates an expression and prints the result to stdout. It predates `$(( ))` and `let`, but you still encounter it in older scripts and portable POSIX sh code. Capture its output with command substitution:

```bash
result=$(expr 5 + 3)
echo $result    # 8
```

`expr` supports the following operators:

| Operator | Operation                        |
| :------- | :------------------------------- |
| `+`      | Addition                         |
| `-`      | Subtraction                      |
| `\*`     | Multiplication (must escape `*`) |
| `/`      | Integer division                 |
| `%`      | Modulo                           |

Every token — numbers, operators, and variables — must be a separate argument separated by spaces. Operators that are also shell metacharacters (`*`, `(`, `)`) must be escaped or quoted:

```bash
expr 10 + 2          # 12
expr 10 - 2          # 8
expr 10 \* 2         # 20
expr 10 / 3          # 3 (truncates)
expr 10 % 3          # 1

count=7
expr $count + 1      # 8
```

`expr` also returns an exit status: `0` if the result is non-zero, `1` if the result is zero or the expression is invalid. This makes it useful in older scripts as a counter in a `while` loop:

```bash
i=1
while [ $i -le 5 ]; do
    echo "iteration $i"
    i=$(expr $i + 1)
done
```

{{< admonition "Prefer \$(( )) in new scripts" tip >}}
`$(( ))` is faster than `expr` (no subprocess), handles operator precedence, and does not require escaping `*`.
{{< /admonition >}}

### Floating-point arithmetic with bc

Bash integer arithmetic truncates decimals. For floating-point calculations, pipe to `bc`:

```bash
result=$(echo "scale=4; 3.14159 * 2.5 * 2.5" | bc)
echo "Area: $result"
```

The `scale` variable sets the number of decimal places. Run `bc` interactively:

```bash
bc -q        # quiet mode, no copyright notice
3.44 / 5
0
scale=4
3.44 / 5
.6880
quit
```

A real-world example: calculate the percentage of disk space used:

```bash
USED=$(df / | awk 'NR==2 {print $3}')
TOTAL=$(df / | awk 'NR==2 {print $2}')
PCT=$(echo "scale=1; $USED / $TOTAL * 100" | bc)
echo "Disk usage: ${PCT}%"
```

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
