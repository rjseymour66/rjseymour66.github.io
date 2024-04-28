---
title: "Linux+"
weight: 40
---

## Basics

### Output redirection

Lets you redirect the output of a command from the monitor to another device, such as a file.

### Script format

The _shebang_ (`#!`) signals to the OS which shell to run the shell script with.
- If you don't specify the shell, the system runs the script with the default shell defined in your user account in `/etc/passwd`

## Advanced

### Displaying messages

Use `echo` to display messages to the command line:

```bash
# no quotes
echo This is a message without a single quote

# output blank line
echo 

# quotes bc contains single quote (')
echo "This ain't a message without a single quote"
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

User variables let you store your own data in your shell scripts. To assign values, use `=` with no spaces:

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
```
### Args and positional vars

Pass command-line arguments to the shell with positional variables:

```bash
# script
cat positional_args.sh 
#!/bin/bash
# Positional vars
echo $1 checked in $2 days ago

# output
./positional_args.sh Ricky 8
Ricky checked in 8 days ago
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

## Writing scripts

### Command substitution

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

### Math

Bash only evaluates values as text strings. To perform math, you have to place the equation in brackets:

```bash
# only integers
result=$[ 25 * 5 ]
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

### if statement

```bash
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

#### Numeric condition tests

| Test | Description |
|---|---|
| n1 -eq n2 | checks if n1 is equal to n2 |
| n1 -ge n2 | checks if n1 is greater than or equal to n2 |
| n1 -gt n2 | checks if n1 is greater to n2 |
| n1 -le n2 | checks if n1 is less than or equal to n2 |
| n1 -lt n2 | checks if n1 is less than n2 |
| n1 -ne n2 | checks if n1 is not equal to n2 |

#### String condition tests

| Test | Description |
|---|---|
| str1 = str2 | checks if str1 is the same as str2 |
| str1 != str2 | checks if str1 is not the same as str2 |
| str1 < str2 | checks if str1 is less than str2 |
| str1 > str2 | checks if str1 is greater than str2 |
| -n str1 | checks if str1 has length greater than zero |
| -z str1 | checks if str1 has length of zero |

#### File condition tests

| Test | Description |
|---|---|
| -d file | checks if file exists and is a dir |
| -e file | checks if file exists |
| -f file | checks if file exists and is a file |
| -r file | checks if file exists and is readable |
| -s file | checks if file exists and is not empty |
| -w file | checks if file exists and is writable |
| -x file | checks if file exists and is executable |
| -O file | checks if file exists and is owned by current user |
| -G file | checks if file exists and the default groups is the same as the current user |
| file1 -nt file2 | checks if file1 is newer than file2 |
| file1 -ot file2 | checks if file1 is older than file2 |

### case statement

write a `case` statement instead of writing multiple `if` condition tests:

```bash
# syntax
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

### Loops

#### for loop

Iterates through every element in a series, such as files in a directory or lines in a text document:

```bash
# syntax
for <variable> in <series>; do
    <commands>
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
```

#### while loop

A `while` loop keeps looping as long as the condition specified evaluates true. The condition in a `while` loop is the same as the `if` statement:

```bash
# syntax
while [<condition> ] ; do
    <commands>
done

# example
number=$1
factorial=1
while [ $number -gt 0 ] ; do
	factorial=$[ $factorial * $number ]
	number=$[ $number - 1 ]
done
echo The factorial of $1 is $factorial
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