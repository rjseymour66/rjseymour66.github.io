---
title: "Script structure"
weight: 10
description: >
  Shebang, running scripts, execution flags, and debugging techniques.
---

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
