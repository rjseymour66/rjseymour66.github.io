---
title: "Introducing the shell"
linkTitle: "Shell"
weight: 20
# description:
---

The shell is a user interface between you and the operating system. The most common is the bash shell (Born Again SHell).

The shell performs a number of tasks in a way that the builtins don't know they happen before they are executed, such as:
- evaluates special characters for pattern matching or globbing (wildcards)
- handles pipes
- redirects stdin and stdout

The term _shell_ can reference a few things:
1. The concept of the linux shell, as in "run the command in the shell"
2. A specific instance of the shell running on your machine

## Pattern matching

Pattern matching or globbing are when you use the asterisk (`*`) character to match any character pattern.
- if there are no matches, the shell tries to match the pattern literally
- the shell always evalutates the pattern before passing the result to the command

```bash
grep one *.txt          # * matches any number of characters
grep one on?.txt        # ? matches any one character
grep one file[12345]    # match a single character within the set bracket []
grep one file[1-5]      # range syntax, equivalent to [12345]
grep one file*[0246]    # combination syntax, match any even number
ls *.doc                # no match, tries to match literally
ls: cannot access '*.doc': No such file or directory
```

## Variables

A running shell can define variables that store values. Each variable has a name and a value.

Print a variable value with `printenv` or by prepending the variable name with a `$`:
- When you use the `$` syntax, the shell evaluates the variable before passing it to the command

```bash
printenv HOME   # /home/elinux
printenv USER   # elinux
echo $HOME      # /home/elinux
echo $USER      # elinux
```

### Defining variables

There are _pre-defined_ and _user-defined_ variables. Variables like `HOME` and `USER` are pre-defined by the shell. By convention, they are uppercase.

User-defined variables work the same as pre-defined vars:
- Do not put a space on either side of the equals sign

```bash
name=value
$name

dev=$HOME/Development/
cd $dev
~/Development
```

## Aliases

An alias is a name that stands in for a command, much like a variable is a name that stands in for a value:
- called _shadowing_ because an alias command takes precendence over the built-in command. The alias "shadows" the original command

```bash
alias                   # list all aliases in current shell
alias ll                # get value of ll
unalias ll              # delete ll alias

alias ll="ls -lh"       # create alias
ll
total 36K
-rw-rw-r-- 1 elinux elinux  325 Mar 25 18:45 animals.txt
-rw-rw-r-- 1 elinux elinux   93 Mar 27 13:29 grades
...
```

## Redirecting input and output

Input redirection is when the shell redirects stdin to come from a file rather than the keyboard.
- there are three unique streams: stdin, stdout, and stderr

```bash
wc < file.txt                       # input redirection
wc < file.txt > output              # redirect input and output
cp nothing file.txt 2> errors.log   # redirect stderr
cp nothing file.txt 2>> errors.log  # append stderr
cp nothing file.txt &> errors.log   # redirect stdout and stderr
```