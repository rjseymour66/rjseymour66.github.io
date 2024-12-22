---
title: "Syntax"
linkTitle: "Syntax"
# weight: 30
# description:
---


## Symbol commands

| Command | Description |
|:--------|:------------|
| `( )` | Runs command in a subshell. |
| `(( ))` | Evaluates and assigns value to a variable. |
| `$(( ))` | Evaluates enclosed expression. |
| `[ ]` |  |
| `<` `>` | String comparison |
| `$( )` | Command substitution. Execute a command and substitue it in the place of the `$()` syntax. |
| `command` | Command substitution. Execute a command and substitue it in the place of the `$()` syntax. |

## Displaying messages

Use `echo` to display messages to the command line:

```bash
# no quotes
echo This is a message without a single quote

# output blank line
echo 

# quotes bc contains single quote (')
echo "This ain't a message without a single quote"
```

## Quotes

Single quotes read metacharacters literally.

Double quotes read metacharacters literally, except for:
- `$`
- `\`
- backtick (\`)

```bash
#!/bin/bash

BASH_VAR="Bash script"

# single quotes
echo $BASH_VAR

echo '$BASH_VAR "$BASH_VAR"'

# double quotes
echo "It's $BASH_VAR and \"BASH_VAR\" using backticks: `date` or $(date)"
```

### Quotes and tics

Backticks, double-, and single-quotes have different use cases:
- single (`''`): When you want to use the literal text in the variable or command statement and do not want character or command substution.
- double (`""`): Allow white space, and character or command substitution.
- backticks (\` \`): Execute a command or script and have its ouput substituted.

### ANSI-C standard characters

| Character | Value |
|:---:|:---:|
| `\a`  | alert (bell)  |
| `\b`  | backspace  |
| `\e`  | escape char  |
| `\f`  | form feed |
| `\n`  | newline  |
| `\r`  | carriage return  |
| `\t`  | horizontal tab  |
| `\v`  | vertical tab  |
| `\\`  | backslash  |
| `\`\`  | single quote  |
| `\nnn`  | octal value of chars  |
| `\xnn`  | hex value of chars  |

```bash
#!/bin/bash

# as a example we have used \n as a new line, \x40 is hex value for @
# and \56 is octal value for .
echo $'web: www.linuxconfig.org\nemail: web\x40linuxconfig\56org'
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

Variables set aside memory locations to temporarily store information so you can recall that information later in a script or program by referencing the variable name.

| Command | Description |
|---|---|
| `set` | Display all global variables. |
| `printenv` | Display local variables set for the session. |
| `export` | Mark a variable as exportable, which means that any child processes spawned from your shell can use it. |
| `env` | Lets you run a script and modify the env vars internal to the script without affecting the system environment variables. |


### Environment variables

Env vars track specific system information:

```bash
set
BASH=/bin/bash
BASHOPTS=checkwinsize:cmdhist:complete_fullquote:expand_aliases:extglob:extquote:force_fignore:globasciiranges:histappend:interactive_comments:login_shell:progcomp:promptvars:sourcepath
BASH_ALIASES=()
BASH_ARGC=([0]="0")
BASH_ARGV=()
BASH_CMDS=()
BASH_COMPLETION_VERSINFO=([0]="2" [1]="11")
BASH_LINENO=()
BASH_REMATCH=()
BASH_SOURCE=()
...
# script
cat envvars.sh 
#!/bin/bash
# display user information from the system
echo User info for userid: $USER
echo UID: $UID
echo HOME: $HOME

# output
./envvars.sh 
User info for userid: linuxuser
UID: 1001
HOME: /home/linuxuser
```

### User variables


A variable is a user-defined value that points to data. Define variables with the following syntax:
- User variables let you store your own data in your shell scripts. To assign values, use `=` with no spaces.

```bash
var1=10
var2=23.45
var3=testing
var4="This has spaces in it"

# script
cat user_vars.sh 
#!/bin/bash
# testing variables
days=10
guest=Katie
echo $guest checked in $days ago

# output
./user_vars.sh 
Katie checked in 10 ago

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

### Exit status

`$?`

When a script ends, it returns its _exit status_ to the parent shell to tell us whether or not the script completed successfully.

```bash
# successful command
who
linuxuser pts/0        2024-04-24 14:51 (10.0.2.2)

# exit status
echo $?
0

# unsuccessful command
./tester.sh
-bash: ./tester.sh: No such file or directory

# exit status
echo $?
127
```

By default, a successful script exits with a `0` status. You can change this with the `exit` command. Change the status if you want to indicate specific errors by changing the exit status value. This helps with debugging:

```bash
# create new shell
/bin/bash

# reassign exit
exit 123
exit

# verify reassignment
echo $?
123
```

### Text manipulation

globbing
: Lets you use wildcard characters to search for multiple files and directories in your script.

parameter expansion
: Placing braces around a variable name (`{test1}`) lets you specify a substring value from the variable based on offset and length.

  [Parameter expansion like a pro](https://www.cyberciti.biz/tips/bash-shell-parameter-substitution-2.html)

`read`
: Lets you read text files line by line, then you can process using standard text manipulation tools.

regular expressions
: Find and replace specific strings in files.


## Arrays

```bash
#!/bin/bash
# Array with 4 els
ARRAY=( 'Debian Linux' 'Redhat Linux' Ubuntu Linux )

# get num of els in array
ELEMENTS=${#ARRAY[@]}

# echo elements in array
for (( i=0;i<$ELEMENTS;i++ )); do
	echo ${ARRAY[${i}]}
done

# OUTPUT
Debian Linux
Redhat Linux
Ubuntu
Linux
```

### Read from file into array

```bash
#!/bin/bash
# Declare array
declare -a ARRAY

# Link fd 10 with stdin
exec 10<&0
# Replace stdin with file supplied as first arg
exec < $1

let count=0
while read LINE; do
	ARRAY[$count]=$LINE
	((count++))
done

echo Number of elements: ${#ARRAY[@]}
# echo array content
echo ${ARRAY[@]}
# restore stdin from fd 10 and close 10
exec 0<&10 10<&-
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
## Command substitution

Lets you assign the output of a command to a user variable in the shell script:

```bash
var=$(command)

# assign
today=$(date) 
me=$(who)

# use vars
echo $today echo $me
Sat Apr 27 12:13:07 PM EDT 2024 echo linuxuser pts/0 2024-04-27 12:06 (10.0.2.2)
```

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

# function params
#!/bin/bash
# Declare funcs in any order
function funcB {
	echo funcB
}

function funcA {
	echo $1
}

funcA "Function A!"
funcB
```
## Control flow

The following sections describe basic control flow syntax.

### if...then...

End the `if` block with `fi` (_if_ spelled backwards):

```bash
if [ test_command ]; then       # if...then
    # command
fi

# -----------------
if [ test_command ]; then       # if...then...else
    # comma
else
    # command
fi

# -----------------
if [ test_command ]; then       # if...then...elif...(else)
    # command
elif
    # command
else
    # command
fi

# syntax
if [ <condition> ]
then
    <commands>
fi

# comparing numbers
if [ $1 -eq $2 ]
then
	echo "Both values are equal"
	exit
fi

if [ $1 -gt $2 ]
then
	echo "The first val is greater than the second val"
	exit
fi


if [ $1 -lt $2 ]
then
	echo "The first val is less than the second val"
	exit	
fi
```

### Condition tests

#### Numeric

| Test | Description |
|---|---|
| n1 -eq n2 | n1 is equal to n2 |
| n1 -ge n2 | n1 is greater than or equal to n2 |
| n1 -gt n2 | n1 is greater to n2 |
| n1 -le n2 | n1 is less than or equal to n2 |
| n1 -lt n2 | n1 is less than n2 |
| n1 -ne n2 | n1 is not equal to n2 |

#### String

| Test | Description |
|---|---|
| str1 = str2 | str1 is the same as str2 |
| str1 != str2 | str1 is not the same as str2 |
| str1 < str2 | str1 is less than str2 |
| str1 > str2 | str1 is greater than str2 |
| -n str1 | str1 has length greater than zero |
| -z str1 | str1 has length of zero |

#### File

| Test | Description |
|---|---|
| -b file | file exists and is a block file |
| -d file | checks if file exists and is a character file |
| -d file | file exists and is a dir |
| -e file | file exists |
| -f file | file exists and is a file |
| -G file | file exists and the default groups is the same as the current user |
| -g file | file exists and is set-group-id |
| -k file | sticky bit |
| -L file | file exists and is a symbolic link |
| -O file | file exists and is owned by current user |
| -r file | file exists and is readable |
| -S file | file exists and is a socket |
| -s file | file exists and is not empty |
| -w file | file exists and is writable |
| -x file | file exists and is executable |
| file1 -nt file2 | file1 is newer than file2 |
| file1 -ot file2 | file1 is older than file2 |

### Loops

Indicate that a loop is complete with the `done` keyword:

```bash

# for...in
for <variable> in <series>; do
    <commands>
done

# while
while test_condition_is_true
do
    # commands
done

# until
#!/bin/bash
COUNT=0
until [ $COUNT -gt 5 ]; do
	echo Value of count is: $COUNT
	let COUNT=COUNT+1
done

# iterate through the files in the Home folder
for file in $(ls | sort); do
	if [ -d $file ]
	then
		echo "$file is a directory"
	fi
	if [ -f $file ]
	then
		echo "$file is a file"
	fi
done

# list every file or dir in /var
for f in $(ls /var/ ); do
	echo $f
done
```

### while loop with input

```bash
#!/bin/bash
# locate and replace spaces in file names
DIR="."

# STDOUT of find command is STDIN to while loop
find $DIR -type f | while read file; do
	if [[ "$file" = *[[:space:]]* ]]; then
		mv "$file" $(echo $file | tr ' ' '_')
	fi
done
```

### case statement

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

# example syntax
case <variable> in
<pattern1>) commands1;;
<pattern2> | <pattern3>) commands2;; # list more than one patterns on a line
*) default <commands>;;
esac

# check current user
case $USER in
su | linuxuser)
	echo "Welcome $USER"
	echo "Please enjoy your visit";;
testing)
	echo "Special testing account";;
admin)
	echo "Don't forget to log off when you're done";;
*)
	echo "Sorry, you're not allowed here!";;
esac
```

### Control keywords

You can use the following keywords when you want to control loop execution apart from the conditions:

`break`
: Terminates the execution of a loop.

`continue`
: Passes control of a loop to the next iteration.

`exit`
: Exits the entire script.

`return`
: Sends back data, result, or return code to the calling script.


## Math

`let`
: You can evaluate equations in place and no need to reference vars with `$`. It is equivalent to `(())`.

`declare -i VAR`
: Declares `VAR` as an integer.

`$[ expression ]` or `$(( expression ))`
: Arithmetic expansion. You can perform arithmetic without declaring variables with `let` or `declare`.

```bash
let SUM=$1+$2
declare -i SUM2
echo $(($1 + $2))

# floating point rounding
floating=3.3446
echo $(printf %.0f $floating)
3
```

### bc

Program that lets you perform floating-point calculations:

```bash
# standard usage
bc
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type \`warranty\'. 
12 * 5.4
64.8
3.14 * (4 + 5)
28.26
quit

# quiet mode, no copyright info
bc -q
3.44 / 5
0

# set scale to set num of digits to right of decimal
scale=4
3.44 / 5
.6880
quit

# use bc in script
# variable=$(echo "options; express" | bc)
var1=$(echo "scale=4; 3.44 / 5" | bc)
echo $var1
.6880
```