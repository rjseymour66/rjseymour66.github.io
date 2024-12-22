---
title: "Basics"
linkTitle: "Basics"
weight: 30
description: >
  Overview of key terms, control structures, etc.
---

Scripts are regular text files with execute permissions set.
- When the execute perm is set, the system will look for the shebang
- If it finds the shebang, it runs the script with `/bin/bash`
- For python, use `#!/usr/bin/env python3`

Store scripts in `/usr/local/bin`.

## Running a script

Before you can run a script, you have to set the execution bit on it with `chmod`:

```bash
$ chmod +x script_name
$ ./script_name
```

The cwd is not in your `$PATH` for security and practical reasons:
- Malware can be named the same as a common command, such as `ls`
- You might have files in your `pwd` with names that conflict with other executables


## Here documents

A _here document_ provides values to a script without user action. It is a multiline string or file literal that sends input streams to other commands and programs.

The here document must contain data in the exact format that the script expects, or it fails. For example, if there are extra spaces, the script fails.

I need to revisit this. For much better examples, see the [Advanced Bash scripting guide](https://tldp.org/LDP/abs/html/here-docs.html).

## Traps (termination signals)

When a program terminates before its supposed to, the computer sends an exit symbol. This is called a trap. You can view them all with `kill -l`:

```bash
$ kill -l
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 ...

#!/bin/bash

# trap command
trap bashtrap INT

# clear screen
clear

# bash trap function is executed when CTRL-C is pressed:
# bash prints message => Executin bash trap subroutine
bashtrap() {
	echo "CTRL+C Detected! ...executing bash trap!"
}

# for loop from 1/10 to 10/10
for a in `seq 1 10`; do
	echo "$a/10 to Exit."
	sleep 1
done
echo "Exit Bash Trap example!"

```

## Processes

The following table shows common `ps` commands:

| Command | Description |
|:--------|:------------|
| ps | Currently running processes for the user. |
| ps -f | Full list of users currently running processes. |
| ps -ef | Full list of processes, excluding kernel processs. |
| ps -A | All user and kernel processes. |
| ps -Kf | Full kernel processes. |
| ps auxw | Wide listing sorted by percentage of CPU usage. |