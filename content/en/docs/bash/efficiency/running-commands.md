---
title: "Running commands"
linkTitle: ""
weight: 70
# description:
---

## List techniques

A sequence of commands on a single command line.

### Conditional lists

Each command depends on the success or failure of the previous command. This depends on the exit code. An exit code of `0` means success, and non-zero means failure:

```bash
echo $?             # get exit code
```
Use the `&&` and `||` operators to create conditional lists:
- `&&`: runs the following command only if the first succeeds. Good for chaining git commands.
- `||`: runs the following command only if the first fails. Good if you want to exit a script if a command fails:

```bash
git add . && git commit -m 'commit message' && git push         # run only if all commands succeed
cd dir || mkdir dir                                             # if cd fails, make dir
cd dir || exit 1                                                # exit script with error if cannot access dir
cd dir || mkdir dir && cd dir || echo "cd dir failed"           # combined operators
```


### Unconditional lists

Separated with a `;` only--commands run one after the other, regardless of success or failure:
- good for ad hoc commands
- same as typing the commands individually and pressing enter
- only the last exit code is assigned to the `$?` shell variable

```bash
sleep 7200; cp -a ~/important-files /mnt/backup-drive
```

## Substitution techniques

Automatically replace the text of a command with other text.

### Command substitution

A command is replaced by its output:
- runs in a subshell, so accepts aliases
- in scripts, commonly used to store output of command in variable


```bash
$(<command>)                                                # syntax
`command`                                                   # old syntax - DO NOT USE
mv $(grep -l "text match" *.txt) dest-dir                   # substitutes a list of files to move
echo Today is $(echo $(date +%A) | tr a-z A-Z)!             # nest sub commands
MATCHES=$(grep -l "text match" *.txt)                       # assign output to var
```

### Process substitution

A command is replaced by its output, but as if its stored in a file:
- transform output into a file so you can run commands that only read from disk to read from stdin
- helpful to compare files, bc `diff` can only compare files

Process substitution runs a command and associates its output with a file descriptor:

```bash
echo <(ls -l)
/dev/fd/63          # file descriptor

set _o posix        # (if necessary) enable process substition

<(command)          # syntax

diff <(ls *.jpg | sort -n) <(seq 1 1000 | sed 's/$/.jpg/') \    # compare contents of one dir with test vals generated on the fly and output as a file
> | grep '>' \                                                  # only output file lines
> | cut -c3-                                                    # display the filenames only
```

## Command-as-string techniques

Construct a string piece-by-piece and then run the string as a command:

### Argument to `bash`

When you run a command with `bash`, it creates a child process with its own environment, including current working directory and variables.
- Changes to the shell in a `bash` command do not affect your present shell
- Helpful with `sudo` commands that use redirection, because the shell runs the redirection command after evaluating the entire command

```bash
bash -c "<command>"                 # syntax

sudo bash -c 'echo "new log file" > /var/log/custom.log'        # sudo redirection, otherwise would only run echo as sudo
[sudo] password for user: 

cat /var/log/custom.log 
new log file
```

### Piping to `bash`

`bash` reads anything from stdin, so you can pipe commands to `bash`:
- good when you need to run multiple similar commands in a row

These steps are going to move every file in a directory into another directory whose first letter corresponds to the first letter in the file name. For example, files `apple` and `aardvark` move to the `a/` directory. The general process is as follows:

1. Print the commands by manipulating strings
2. View the resuls with `less`
3. Pipe the results to `bash`

Here are the steps:

1. Create the subdirectories:
   ```bash
   mkdir {a..z}
   ```
2. Create the regex for the `sed` command you will use in `mv`. Begin by capturing the first character of the filename as expression #1 `(\1)` for `sed`:
   ```bash
   ^\(.\)
   ```
3. Get the rest of the filename as expression #2 `(\2)`:
   ```bash
   \(.*\)$
   ```
4. Connect the two:
   ```bash
   ^\(.\)\(.*\)$
   ```
5. Create the `mv` command, passing the sed expressions as the filename:
   ```bash
   mv \1\2 \1           # mv apple a
   ```
6. Check the full command with `less`:
   ```bash
   ls -1 ??* | sed 's/^\(.\)\(.*\)$/mv \1\2 \1/' | less
   ```
7. When satisfied, pipe it to `bash` to execute:
   ```bash
   ls -1 ??* | sed 's/^\(.\)\(.*\)$/mv \1\2 \1/' | bash          # ??* makes sure the file is 2 chars long so it doesn't collide with the filenames
   ```

### Remote commands with SSH

You can execute a single command by passing it after the `ssh` command:
- quicker than logging in, executing command, logging out
- quote or escape special chars and redirection that need to be evaluated by remote shell
- you can pipe commands to `ssh` like you would `bash`
  - suppress pseudo-terminal output with `-T` option
  - suppress welcome text by processing with `bash`

```bash
ssh user@host <command>                     # syntax

ssh user@host ls > output                   # output file on local machine
ssh user@host "ls > output"                 # output file on remote machine
echo "ls > output" | ssh user@host          # pipe to SSH
echo "ls > output" | ssh -T user@host       # pipe to SSH, suppress pseudo-terminal info
echo "ls > output" | ssh user@host bash     # pipe to SSH, suppress welcome text
```

### xargs

Helps you construct and run multiple similar commands. It accepts two inputs:
1. List of whitespace-separated strings in stdin
2. Incomplete command template that is missing some arguments

`xargs` merges the list of strings and the command template create and run complete commands.
- with `find`, `xargs` is usually a cleaner option than `-exec`
- `-n`: controls now many arguments are appended by `xargs` onto each generated command
- `I`: controls where the input strings appear in the generated command. Follow `-I` by any arbitrary string to make that string the placeholder. Seems to be like the `{}` in `find`'s `-exec`.
  Using `-I` limits `xargs` arguments to 1.
- `-0`: changes the input separator from whitespace to the null character (ASCII zero, `\0`).
  With `find`, protects against special characters in input. Pass `-print0` to `find` to separate input strings with nulls.
  `ls` cannot cleanly change space to nulls, so you can create an alias: `ls0="find . -maxdepth=1 -print0"`

```bash
find . -type f -name \*.py -print0 | xargs -0 wc -l     # print num of lines in all pythong files in pwd

# -n option
ls
apple  banana  cantaloupe  date  watermelon

ls | xargs -n1 echo                             # one option at a time
apple
banana
cantaloupe
date
watermelon

ls | xargs -n2 echo                             # two options at a time
apple banana
cantaloupe date
watermelon
```

## Process-control techniques

### Background commands

Immediately return the prompt and execute the command out of sight (in the background). Good for commands that take a long time to complete. Commands that occupy the shell are called _foreground_ commands. A shell can run one foreground command, and any number of background commands.

Background commands are part of the shell's job control:
- A job is a shell's unit of work, its a single instance of a command running in the shell
- Each job has an ID that is a positive integer, or job ID
- A good use of background jobs is to suspend your text editor with `CTRL+Z`while you go back to the shell to run commands. 

To run a command in the background, append an ampersand to the end of the command:

```bash
<command> &                                     # syntax
<command1> & <command2> & <command3> &          # multiple bg commands
```
#### Job control commands

| Command    | Description                                 |
| ---------- | ------------------------------------------- |
| `jobs`     | View shell's jobs                           |
| `CTRL + Z` | Suspend a running job                       |
| `bg`       | Run current suspended job in the background |
| `bg %n`    | Move suspended job _n_ into the background  |
| `fg %n`    | Move background job _n_ into the foreground |
| `kill %n`  | Kill background job _n_                     |


### Explicit subshells

Launch these in the middle of a combined command. Enclose the command in parentheses (`()`) to run in a child shell:
- useful if you need to change dirs in a pipeline command

```bash
(cd /usr/local)                     # doesn't change parent shell pwd
```


### Process replacement

Supersedes the parent shell. Use with `exec`, but I couldn't find any practical examples for what I do!