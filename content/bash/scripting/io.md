---
title: "Input and output"
weight: 30
description: >
  Output formatting, quoting, user input, here documents, and command substitution.
---

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

A *here document* supplies multi-line input to a command without requiring a separate file. Everything between the opening and closing delimiter is treated as stdin. The delimiter can be any word. `EOF` is conventional but not required.

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

Use a quoted delimiter here to prevent variable expansion. Config files typically contain literal values, not shell expressions.

## Command substitution

The `$()` syntax tells bash to run the command inside the parentheses in a subshell, then replace the entire expression with that command's stdout. Use it to assign output to a variable, embed it in a string, or pass it as an argument to another command.

```bash
TODAY=$(date +%Y-%m-%d)
DISK_USAGE=$(df -h / | awk 'NR==2 {print $5}')
echo "Disk usage on $TODAY: $DISK_USAGE"
```

Nest command substitutions to build complex single-line expressions:

```bash
echo "Today is $(echo $(date +%A) | tr a-z A-Z)!"    # Today is MONDAY!
```
