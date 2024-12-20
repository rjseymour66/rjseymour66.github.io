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

## Basic commands

```bash
echo "write this to the console"
sleep <num>     # sleep for <num> seconds. <num>m or <num>h for min, hours
clear           # clears the screen, good for the beginning of scripts
$word           # access the value of var named "word"
```

## Arguments

Shell arguments are 0-indexed, and are always preceded with the `$` symbol. For example, `$0` is the shell itself, and `$1` is the first argument passed to the shell.

You can process all CLI args at once with the following commands. Notice the different behavior when each command is wrapped in double quotes (`""`):

`$*`
: Specifies all command arguments.

`$@`
: Specifies all command arguments.

`"$*"`
: Takes the entire argument list as one argument, with a space between each value.

`"$@"`
: Takes the entire argument list as separates it into individual arguments.


> Look into the `shift` command.

```bash
#!/bin/bash

# echo args to the shell
echo $1 $2 $3 ' -> echo $1 $2 $3'

# store args from command line in array
args=("$@")

# echo args to shell
echo ${args[0]} ${args[1]} ${args[2]} ' -> args=("$@"); echo ${args[0]} ${args[1]} ${args[2]}'

# use $@ to print out all args
echo $@ ' -> echo $@'

# print number of args
echo Number of arguments passed: $# ' -> echo Number of arguments passed: $#'

```

## Variables

A variable is a user-defined value that points to data. Define variables with the following syntax:

```bash
var='value to assign'

# access with '$'
echo $var
print $var
cat $var_file

# global vs local vars
#!/bin/bash
#
VAR="global var"

function bash {
	local VAR="local var"
	echo $VAR
}

echo $VAR
bash
```

## User input

```bash
#!/bin/bash

echo -e "Hi, please type the word: \c"
read word

echo "The word you entered is: $word"
echo -e "Can you please enter two words? "
read word1 word2
echo "Here is your input \"$word1\" \"$word2\""
echo -e "How do you feel about bash scripting?"

# read stores reply into the default build-in var $REPLY
read
echo "You said $REPLY, I'm glad to hear that! "
echo -e "What are your favorite colors ? "

# -a reads into an array
read -a colors
echo "My favorite colors are also ${colors[0]}, ${colors[1]}, ${colors[2]}:-)"
```

## Symbol commands

| Command | Description |
|:--------|:------------|
| `( )` | Runs command in a subshell. |
| `(( ))` | Evaluates and assigns value to a variable. |
| `$(( ))` | Evaluates encloded expression. |
| `[ ]` |  |
| `<` `>` | String comparison |
| `$( )` | Command substitution. Execute a command and substitue it in the place of the `$()` syntax. |
| `command` | Command substitution. Execute a command and substitue it in the place of the `$()` syntax. |


### Quotes and tics

Backticks, double-, and single-quotes have different use cases:
- single (`''`): When you want to use the literal text in the variable or command statement and do not want character or command substution.
- double (`""`): Allow white space, and character or command substitution.
- backticks (\` \`): Execute a command or script and have its ouput substituted.

## Functions

There are a number of ways to define functions in bash. You should use the function keyword and no parentheses to emphasize that Bash functions do not accept paramters. Additionally, you can put the curly braces on separate lines. Make sure that you define a function before you can use it:

```bash
# signature options:
function Helper() {}
function Helper {}
Helper () {}

# preferred
function func_name {
    # commands
}
```
## Control flow

The following sections describe basic control flow syntax.

### if...then...

End the `if` block with `fi` (_if_ spelled backwards):

```bash

# if...then

if [ test_command ]
then
    # command
fi

# if...then...else

if [ test_command ]
then
    # command
else
    # command
fi

# if...then...elif...(else)

if [ test_command ]
then
    # command
elif
    # command
else
    # command
fi
```

### Loops

Indicate that a loop is complete with the `done` keyword:

```bash

# for...in
for loop_var in arg_list
do
    # commands
done

# while
while test_condition_is_true
do
    # commands
done

# until
until test_condition_is_true
do
    # commands
done
```

### Case statement

The `case` statement is the Bash equivalent to the `switch` statement in other languages. Indicate the end of command execution for each case with double semi-colons (`;;`). The default ("fall-through") case is indicated with the wildcard (`*`) character:

```bash
case $var in

match_1)
    # commands for match_1
    ;;
match_2)
    # commands for match_2
    ;;
match_3)
    # commands for match_3
    ;;
*)
    # commands for default (no match)
    ;;
esac
```

### Control keywords

You can use the following keywords when you want to control loop execution apart from the conditions:

break
: Terminates the execution of a loop.

continue
: Passes control of a loop to the next iteration.

exit
: Exits the entire script.

return
: Sends back data, result, or return code to the calling script.

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