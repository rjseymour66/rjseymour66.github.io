---
title: "Time savers"
# linkTitle: ""
weight: 100
# description:
---

## `less` to `vim`

When you are viewing a file with `less`, just press `v` to open the file in your default editor. When you are finished, save in the editor and you are sent back to `less`.

Make sure you set your default editor either in your shell or `.bashrc`:

```bash
VISUAL=vi               # .bashrc
EDITOR=vi

export EDITOR=vi        # shell env
```

## Edit files that contain a given string

Use `grep` with command substitution to edit with vim all files that contain a specific string:
- cycle through the files in vim with `:bn` (forward) and `:bp` (backwards) 
```bash
vim $(grep -l <string> *)                                   # search pwd
vim $(grep -lr <string> *)                                  # search dir tree
vim $(find . -type f -print0 | xargs -0 grep -l <string>)   # faster search for larger dir tree
```

## Creating empty files

Create them with `touch` or `echo`. Prefer `touch`:

```bash
touch file{0..1000}.txt
echo -n > empty             # -n option omits newline char
```

## Process file one line at a time

Use this loop to read a file, one line at a time:
- `cat file` sends the contents of file to stdout
- `while read line` is a while loop
  - `read` takes input from stdin
  - `line` is the variable

```bash
cat file | while read line; do
    # do something
    echo "$line"
done
```

