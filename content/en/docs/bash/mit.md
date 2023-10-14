---
title: "MIT"
linkTitle: "MIT"
weight: 30
description: >
  MIT missing semseter
---

## Variables 

```bash
# declare variable. Be mindful of spaces.
$ foo=bar

# define strings
echo "Hello"
echo 'Hello'

# output of command into a variable
$ foo=$(pwd)
$ echo $foo
/home/ryanseymour/Development
$ echo "We are in $(pwd)"
We are in /home/ryanseymour/Development
```

## Process substituion

```bash
$ cat <(ls) <(ls..)
```

## Create strings

```bash
# This works
echo "Value is $foo"
# single quotes do not work!
echo 'Value is $foo'
```

## Keywords

`$1` is equivalent to argv in other languages. Its how you access the first variable.
`$0` is the name of the script
`$_` will give you the last argument of the previous command
`!!` is replaced with the last command that you executed. use this with `sudo` a lot.
`$?` gets you the error code from the previous command

```bash
$ echo "Hello"
Hello
$ echo $?
0

# to stderr
$ grep foobar mcd.sh
grep: mcd.sh: No such file or directory
$ echo $?
2

# success
$ true
$ echo $?
0

$ false || echo "oops fail"
oops fail

$ echo $?
0
# failure
$ false
$ echo $?
1

$ false && echo "oops fail"
$ echo $?
1
```

```bash
mkdir/new
... Permission denied
sudo !!
```

## Functions

This function creates a directory and then cds into it:

```bash
mcd () {
    mkdir -p "$1"
    cd "$1"
}
```

## Conditionals

For test conditionals, run `man test`.

## Globbing

Globbing is finding things with `*` (I think?):
```bash
$ ls *.sh

$ touch {foo,bar}/{a..j}
$ ls foo/
a  b  c  d  e  f  g  h  i  j
$ ls bar
a  b  c  d  e  f  g  h  i  j

$ rm foo/c
rm: remove regular empty file 'foo/c'? y

$ diff <(ls foo) <(ls bar)
2a3
> c

```

## Shebang 

Run the `env` command with `python` as a variable and then run the 
```bash
#!/usr/bin/env python

# OR

#!/bin/bash
```

```bash
$ find . -name src -type d
```