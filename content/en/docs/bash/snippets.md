---
title: "Snippets"
# weight: 10
description: >
  Expressions or code blocks that complete specific tasks.
---

## Text

### Uppercase or lowercase

Use the `tr` (translate) command to change the case of a character set:

```bash
uppercase=$(echo $var | tr '[a-z]' '[A-Z]')

lowercase=$(echo $var | tr '[A-Z]' '[a-z]')
```

Another option is the `typeset` command. You can use the `-u` and `-l` options to ensure a variable always evaluates in upper or lowercase, respectively:

```bash
$ typeset -u my_name
$ my_name='Sally'
$ echo $my_name 
SALLY


$ typeset -l last_name
$ last_name='Ride'
$ echo $last_name 
ride
```

> You must set the `typset` value first.