---
title: "The bash shell"
weight: 10
description: >
  How bash processes commands, manages data streams, and maintains the environment.
---

When you log in to a Linux system, open a terminal, or connect to a remote server, you interact with bash. Bash is the program that reads your commands, evaluates them, and passes them to the operating system. Understanding how bash works at this level makes the difference between a user who types commands and a practitioner who can construct pipelines, diagnose unexpected behavior, and write reliable scripts.

This page covers the core mechanics: how bash classifies and finds commands, how data flows between programs, how the environment shapes every command you run, and how the shell connects remote machines into a single workflow.

## Commands, built-ins, and keywords

Not all commands work the same way. Bash divides executable items into three categories:

- *Files*: Executable programs stored on disk, such as `grep`, `find`, or `python3`. Bash finds these by searching the directories listed in `PATH`.
- *Built-ins*: Commands built into bash itself, such as `cd` and `pwd`. They run inside the current shell process rather than launching a new one.
- *Keywords*: Part of the bash language, such as `if`, `for`, and `while`. The shell interprets them directly.

Run `type` to identify which category a command belongs to:

```bash
type -t cat      # file
type -t cd       # builtin
type -t if       # keyword
```

Run `compgen` to list all available items of a given type:

```bash
compgen -c    # all available commands
compgen -b    # built-ins only
compgen -k    # keywords only
```

## Standard streams

Every process on Linux has three streams connected to it by default:

| Stream   | File descriptor | Default source or destination |
| :------- | :-------------- | :---------------------------- |
| `stdin`  | 0               | Keyboard input                |
| `stdout` | 1               | Terminal (standard output)    |
| `stderr` | 2               | Terminal (error messages)     |

These streams let programs work together without knowing anything about each other. A program reads from stdin and writes to stdout. The shell connects programs by wiring one program's stdout to the next program's stdin.

## Redirecting streams

Redirection changes where a stream goes without modifying the program. The shell evaluates all redirection operators before running the command.

The following table shows common redirection operators:

| Operator | Effect                                                  |
| :------- | :------------------------------------------------------ |
| `>`      | Write stdout to a file. Creates or overwrites the file. |
| `>>`     | Append stdout to a file                                 |
| `<`      | Read stdin from a file instead of the keyboard          |
| `2>`     | Write stderr to a file                                  |
| `2>>`    | Append stderr to a file                                 |
| `&>`     | Write stdout and stderr to a file                       |
| `&>>`    | Append stdout and stderr to a file                      |

Redirect command output to a file for later review:

```bash
ls -la /etc > /tmp/etc_listing.txt
```

Redirect only error messages to a log file, discarding normal output:

```bash
cp -r /data /backup 2> /tmp/copy_errors.log
```

Send both stdout and stderr to the same file. This useful when running unattended jobs or when you don't want to output any information to the screen, but you want to review it later:

```bash
find /home -name "*.conf" &> /tmp/find_results.txt
```

To discard output entirely, redirect to `/dev/null`:

```bash
find / -name "authorized_keys" 2>/dev/null
```

### The tee command

`tee` writes output to both stdout and a file simultaneously. This lets you inspect output in the terminal while also saving it for later analysis:

```bash
grep "ERROR" /var/log/syslog | tee /tmp/errors.txt | wc -l
```

The previous command filters errors from syslog, saves them to `/tmp/errors.txt`, and counts the matching lines in one step.

To append to an existing file rather than overwrite it, pass the `-a` flag:

```bash
./deploy.sh 2>&1 | tee -a /var/log/deploys.log
```

## Pipes and pipelines

A *pipeline* is a sequence of commands connected by the pipe operator (`|`). The shell connects the stdout of each command to the stdin of the next. Each command in a pipeline runs as its own child process in parallel.

Find the ten most recently modified files in `/var/log`:

```bash
ls -lt /var/log | head -10
```

Count the number of unique IP addresses in an Apache access log:

```bash
awk '{print $1}' /var/log/apache2/access.log | sort | uniq | wc -l
```

Find the most common HTTP status codes returned today:

```bash
grep "$(date +%d/%b/%Y)" /var/log/apache2/access.log | awk '{print $9}' | sort | uniq -c | sort -rn
```

The key principle behind pipelines is that each command does one thing well. Combining small, focused tools lets you solve complex problems without writing a script.

## Pattern matching

*Globbing* lets you match filenames with wildcard patterns. The shell expands the pattern into a list of matching filenames before passing them to the command. The command itself never sees the pattern -- it receives the expanded list.

The following table shows common glob patterns:

| Pattern  | Matches                                  |
| :------- | :--------------------------------------- |
| `*`      | Any number of characters, including none |
| `?`      | Any single character                     |
| `[abc]`  | Any single character in the set          |
| `[a-z]`  | Any single character in the range        |
| `[!abc]` | Any single character not in the set      |

List all `.conf` files in `/etc`:

```bash
ls /etc/*.conf
```

Match files with a single-character extension:

```bash
ls source.?    # matches source.c but not source.cpp
```

Match all even-numbered log files:

```bash
ls /var/log/app.log.*[02468]
```

If a pattern matches nothing, bash passes the literal string to the command. For example, `ls *.doc` fails with a "no such file" error if no `.doc` files exist in the current directory.

## Variables

The shell stores values in *variables*. Reference a variable by prepending its name with `$`. The shell expands the variable before passing the result to the command:

```bash
echo $HOME      # /home/youruser
echo $USER      # youruser
```

### Defining variables

Define a variable with `=` and no spaces on either side of the sign:

```bash
logdir=/var/log
ls "$logdir"
```

Quoting a variable with double quotes prevents word splitting and glob expansion on its value, which is important when a value contains spaces:

```bash
target_dir="/var/log/my app"
ls "$target_dir"
```

### Environment variables

*Environment variables* are variables that the shell copies to every child process it creates. They form the shell's environment.

Common environment variables include:

| Variable       | Description                                                                    |
| :------------- | :----------------------------------------------------------------------------- |
| `HOME`         | Path to your home directory                                                    |
| `USER`         | Your username                                                                  |
| `PATH`         | Colon-separated list of directories searched for executables                   |
| `SHELL`        | Path to your default shell                                                     |
| `BASH_VERSION` | Version string of the running bash instance (for example, `5.1.16(1)-release`) |
| `BASHPID`      | Process ID of the current bash shell                                           |
| `HOSTNAME`     | Hostname of the current machine                                                |
| `OSTYPE`       | Operating system type (for example, `linux-gnu` or `darwin`)                   |
| `HISTFILE`     | Path to your command history file                                              |
| `EDITOR`       | Your preferred text editor                                                     |
| `RANDOM`       | Returns a pseudo-random integer between 0 and 32767 each time it is referenced |
| `UID`          | Numeric user ID of the current user                                            |

Print all environment variables sorted alphabetically:

```bash
printenv | sort
```

Run `export` to promote a local variable to an environment variable, making it available to child processes. This is how you pass configuration values to scripts or subprocesses:

```bash
export DB_HOST=db.internal
export DB_PORT=5432
./run_migrations.sh    # the script can read DB_HOST and DB_PORT
```

## PATH

`PATH` is a colon-separated list of directories. When you run a command, bash searches these directories in order and runs the first match. Print each directory on its own line:

```bash
echo $PATH | tr : "\n"
```

If a command exists in multiple directories, bash runs the first one it finds. You can override a system program by placing your own version in a directory that appears earlier in `PATH`. A common pattern is to add a personal `bin` directory at the front:

```bash
export PATH=$HOME/bin:$PATH
```

Run `which` to confirm which version of a program bash will run:

```bash
which python3    # /usr/bin/python3
```

Run `type` to identify built-ins, aliases, functions, and programs by name:

```bash
type cd        # cd is a shell builtin
type ll        # ll is aliased to `ls -lh`
```

## CDPATH

`CDPATH` works like `PATH`, but for the `cd` command. When you change directories, bash searches the directories in `CDPATH` for a match. This is useful when you work frequently inside a deep directory tree.

For example, define `CDPATH` to include your home directory and its parent:

```bash
CDPATH=$HOME:..
```

After this, `cd projects` changes to `$HOME/projects` from anywhere on the filesystem. Adding `..` to `CDPATH` lets you move to sibling directories without typing `../sibling`.

Define `CDPATH` in your `.bashrc` to make it permanent:

```bash
CDPATH=$HOME:$HOME/projects:..
```

When you change to a directory via `CDPATH`, bash prints the absolute path to confirm where you landed.

## Aliases

An *alias* is a shortcut for a command or command sequence. Bash evaluates aliases before searching `PATH`, so an alias with the same name as a real command takes precedence. This is called *shadowing*:

```bash
alias                     # list all current aliases
alias ll                  # show the value of a specific alias
alias ll="ls -lh"         # define an alias
unalias ll                # remove an alias
```

A common pattern is making destructive commands safer:

```bash
alias rm="rm -i"          # prompt before every deletion
alias cp="cp -i"          # prompt before overwriting
```

To bypass an alias and run the original command, prefix it with a backslash:

```bash
\rm file.txt    # runs the real rm, not the alias
```

Define aliases in `.bashrc` to make them available in every interactive shell.

## Quotes and backslashes

Single quotes, double quotes, and backslashes each change how the shell evaluates special characters.

The following table summarizes their behavior:

| Quoting style | Effect                                                                        |
| :------------ | :---------------------------------------------------------------------------- |
| `'single'`    | Every character is literal, including `$` and `\`                             |
| `"double"`    | Most characters are literal. The shell still expands `$`, `\`, and backticks. |
| `\backslash`  | Treats the immediately following character as literal                         |

Compare single and double quote behavior:

```bash
name="Alice"
echo '$name'     # $name  (literal, not expanded)
echo "$name"     # Alice  (variable expanded)
```

A backslash at the end of a line continues the command on the next line, which is useful for readability in long pipelines:

```bash
find /var/log -name "*.log" -mtime -1 \
    -exec grep "CRITICAL" {} \; \
    > /tmp/critical_today.txt
```

## Parent and child processes

Every command you run creates a *child process* that inherits a copy of the parent shell's environment. This has a practical consequence: a script can run `cd` and change directories freely, but when the script exits, your terminal stays in its original directory. The script's directory change only affects its own child process.

Built-ins like `cd` and `export` are exceptions. They run inside the current shell process, which is why `cd` actually changes your working directory and `export` actually modifies your environment.

### Subshells

A *subshell* is a complete copy of the parent shell, including its variables, aliases, and functions. Enclose commands in parentheses to run them in a subshell:

```bash
(cd /var/log && grep -r "ERROR" .)
```

The previous command changes to `/var/log` and greps inside it, but your terminal remains in its current directory when the subshell exits.

### Child shells vs. subshells

A *child shell* is a partial copy of its parent. It inherits environment variables but not local variables or aliases. Scripts run in child shells, which is why scripts cannot access aliases defined in your terminal.

A subshell is a full copy of the parent. To confirm this, run `alias` inside parentheses and it will list all aliases from the parent:

```bash
(alias)    # lists all parent shell aliases
```

## Shell configuration files

When bash starts, it runs configuration files to prepare the environment. Which files it runs depends on whether the shell is a *login shell* or a *non-login shell*.

The following table shows the relevant configuration files:

| File type | When it runs         | System-wide location    | Personal location               |
| :-------- | :------------------- | :---------------------- | :------------------------------ |
| Startup   | Login shell          | `/etc/profile`          | `~/.bash_profile`, `~/.profile` |
| Init      | All non-login shells | `/etc/bash.bashrc`      | `~/.bashrc`                     |
| Cleanup   | Login shell exit     | `/etc/bash.bash_logout` | `~/.bash_logout`                |

Add `PATH` customizations, `CDPATH`, and exported variables to `~/.bash_profile`. Add aliases, functions, and local variable definitions to `~/.bashrc`. Have `~/.bash_profile` source `~/.bashrc` so that interactive terminal sessions pick up both:

```bash
if [ -f "$HOME/.bashrc" ]; then
    source "$HOME/.bashrc"
fi
```

Storing your personal configuration files in a version-controlled repository (such as a dotfiles repo on GitHub) makes it easy to reproduce your environment on a new machine.

## Restricted shell

Start bash with `-r` (or run it as `rbash`) to launch a restricted shell. A restricted shell limits what users can do, making it useful for locked-down accounts or controlled execution environments:

```bash
bash -r
rbash
```

The restricted shell blocks the following actions:

- Changing directories with `cd`
- Modifying `PATH`, `SHELL`, `ENV`, or `BASH_ENV`
- Running commands that contain a `/` in the name or path
- Redirecting output with `>`, `>>`, `&>`, or `<>`
- Using `exec` to replace the shell process

Restrictions take effect after bash processes any startup files (`~/.bash_profile`, `~/.bashrc`). A user cannot turn off restricted mode with `set +r` once the shell is running.

To give a user a restricted shell as their login shell, set their shell to `rbash` in `/etc/passwd`:

```bash
sudo usermod -s /bin/rbash username
```

## Control operators

Control operators tell bash how to sequence and connect commands. They are not commands themselves — bash evaluates them as part of parsing the command line.

| Operator | Name            | Effect                                                                               |
| :------- | :-------------- | :----------------------------------------------------------------------------------- |
| `&&`     | AND             | Run the next command only if the preceding command succeeded (exit status 0)         |
| `\|`     | Pipe            | Connect stdout of the preceding command to stdin of the next                         |
| `\|\|`   | OR              | Run the next command only if the preceding command failed (non-zero exit status)     |
| `;`      | Sequencer       | Run the next command after the preceding command finishes, regardless of exit status |
| `;;`     | Case terminator | End a `case` clause                                                                  |
| `(` `)`  | Subshell        | Group commands and run them in a subshell                                            |

### &&: conditional AND

`&&` runs the right-hand command only if the left-hand command exits with status 0. This is the standard pattern for chaining dependent steps:

```bash
make && make install
git add . && git commit -m "update config"
```

If `make` fails, `make install` does not run.

### |: pipe

`|` connects the stdout of the left command to the stdin of the right command. Each command runs as its own process in parallel:

```bash
ps aux | grep nginx
cat /var/log/syslog | grep ERROR | wc -l
```

### ||: conditional OR

`||` runs the right-hand command only if the left-hand command fails. Use it to provide a fallback or to print an error message:

```bash
ping -c1 8.8.8.8 || echo "Network unreachable"
mkdir /tmp/work || exit 1
```

### ;: sequencer

`;` runs commands in sequence, one after the other, regardless of whether the previous command succeeded. It is equivalent to pressing Enter between commands:

```bash
apt update; apt upgrade; apt autoremove
```

Use `;` when the commands are independent and you do not need to check exit status between them. Use `&&` when each step depends on the one before it.

### ;;: case terminator

`;;` ends a clause inside a `case` statement. Bash jumps to `esac` after executing the matching clause:

```bash
case "$1" in
    start)  systemctl start myapp ;;
    stop)   systemctl stop myapp ;;
    *)      echo "Usage: $0 start|stop" ;;
esac
```

### ( ): subshell grouping

Enclosing commands in parentheses runs them in a subshell. The subshell inherits the parent's environment but any changes — directory changes, variable assignments — do not affect the parent:

```bash
(cd /tmp && tar -xzf archive.tar.gz)    # parent shell stays in its original directory
(export DEBUG=1; ./run_tests.sh)        # DEBUG is set only for this subshell
```

Use a subshell when you need to isolate side effects from the rest of the script.

## Job control

Job control is bash's system for managing multiple commands running in the same terminal session. Each command you run is a *job*. By default, jobs run in the foreground and you wait for them to finish before you get your prompt back. Job control lets you push long-running commands to the background so your terminal stays free, check what is running, and pull jobs back when you need them.

### &: background execution

Appending `&` to a command starts it in the background. Bash prints the job number in brackets and the process ID, then returns the prompt immediately:

```bash
./long_backup.sh &
[1] 48732
```

The job number (`[1]`) is how bash tracks the job within the session. Background jobs still write to your terminal by default. Redirect their output to keep your terminal clean:

```bash
./long_backup.sh > backup.log 2>&1 &
```

Start a development web server in the background while you continue working in the same terminal:

```bash
python3 -m http.server 8080 &
[1] 52301
```

### jobs: listing background jobs

`jobs` lists every job currently managed by the shell, along with its status and the command that started it:

```bash
jobs
[1]-  Running     ./long_backup.sh &
[2]+  Running     python3 -m http.server 8080 &
```

The `+` marks the *current job*, which is the one `fg` and `bg` act on by default. The `-` marks the previous job. Status is one of `Running`, `Stopped`, or `Done`.

Pass `-l` to include the PID alongside each job:

```bash
jobs -l
[1]-  48732 Running     ./long_backup.sh &
[2]+  52301 Running     python3 -m http.server 8080 &
```

### fg: bringing a job to the foreground

`fg` moves a background job to the foreground. Without an argument, it resumes the current job:

```bash
fg
```

To bring a specific job to the foreground, pass its job number prefixed with `%`:

```bash
fg %2
```

You started a log monitoring command in the background, finished other work, and now want to watch it:

```bash
tail -f /var/log/nginx/access.log &
[1] 53100

# ... do other work ...

fg %1
tail -f /var/log/nginx/access.log
```

Once a job is in the foreground, press `Ctrl+C` to stop it or `Ctrl+Z` to suspend it and send it back to the stopped state.

### Ctrl+Z and bg: suspending and resuming

`Ctrl+Z` suspends the foreground job and returns the prompt. The job is paused, not killed:

```bash
./long_backup.sh
^Z
[1]+  Stopped     ./long_backup.sh
```

Run `bg` to resume the stopped job in the background:

```bash
bg %1
[1]+ ./long_backup.sh &
```

This is useful when you started a command in the foreground and realize mid-run that you want your terminal back without losing the work already done.

### nohup: surviving logout

When you close a terminal or disconnect from SSH, bash sends the HUP (hangup) signal to all jobs in the session. By default that kills them. `nohup` runs a command that ignores HUP, so it keeps running after you log out.

Run a long job that must survive a disconnection:

```bash
nohup ./long_backup.sh &
[1] 58423
nohup: ignoring input and appending output to 'nohup.out'
```

By default `nohup` redirects output to `nohup.out` in the current directory. Redirect to a specific file to keep things organized:

```bash
nohup ./long_backup.sh > /var/log/backup.log 2>&1 &
```

Check progress by tailing the log file:

```bash
tail -f /var/log/backup.log
```

`nohup` is the standard choice when you need a process to keep running after you close your SSH session. For processes that need to survive reboots or run on a schedule, use a systemd service instead.

## Remote commands with SSH

You can run a single command on a remote machine by passing it to `ssh` without starting an interactive session. This is useful for automation, remote monitoring, and scripted maintenance:

```bash
ssh user@host ps aux                            # list running processes on the remote machine
ssh user@host ls /etc > local_output.txt        # save remote output to a local file
ssh user@host "ls /etc > /tmp/output.txt"       # save output on the remote machine
ssh user@host bash < ./deploy.sh                # run a local script on the remote machine without copying it first
```

To suppress the pseudo-terminal warning when piping commands to SSH, pass the `-T` flag:

```bash
echo "df -h" | ssh -T user@host
```

To redirect on the remote machine rather than locally, quote or escape the redirection operator so the local shell does not evaluate it:

```bash
ssh user@host ps \> /tmp/ps.out    # the > is evaluated by the remote shell
```
