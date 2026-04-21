---
title: "Variables"
weight: 20
description: >
  User-defined and environment variables, parameter expansion, arrays, and arithmetic.
---

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

`unset` removes a variable from the current shell environment. After unsetting, any reference to the variable returns an empty string. With `set -u` enabled, bash treats the reference as an error instead:

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

Removing an element leaves a gap in the index. The remaining elements do not shift. If your code iterates by index, account for the missing slot. Iterating with `"${SERVERS[@]}"` skips empty slots automatically.

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

Every token must be a separate argument separated by spaces. This includes numbers, operators, and variables. Operators that are also shell metacharacters (`*`, `(`, `)`) must be escaped or quoted:

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
