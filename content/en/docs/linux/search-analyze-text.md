---
title: "Searching and analyzing text"
weight: 40
description: >
  How to Search and analyze text.
---

## Processing text files

**_text file record_**
: Single file line that ends in a newline feed (ASCII character LF). Check this w `cat -e` command to display LF chars as `$`. Change this with `cut -z`.

**_text file record delimiter_**
: One or more chars that create a boundary between different data items within a record.

### cut

Extracts small data sections of a file. Does not edit any files, it copies data and shows it to you:

```bash
cut OPTION... [FILE]...
-c nlist    # Display record chars in nlist (-c 2-8)
-b blist    # Display record bytes in blist
-d          # set delimiter, tab is default
-f flist    # display records fields in flist (-f 1,5)
-s          # display only records that have the delimiter
-z          # change ASCII EOL char to NUL

# display fields 1 and 7, delimited by colon
cut -d ":" -f 1,7 /etc/passwd
# display chars 1-5 from file
cut -c 1-5 /etc/passwd
```

### grep

Uses regex, a pattern template that you define for a utility like grep that filters text. Basic regex (BREs) include `.*` and `^r`. Extended regex (EREs) let you specify two words or character sets to match with the `|` char.

```bash
grep [OPTIONS] PATTERN [FILE...]
-c # display record count that match PATTERN
-d action # read, skip, or recurse on dirs
-E # Extended regex
-i # ignore case
-R,-r # recursive
-v # match only files that do not match PATTERN

# BREs

grep daemon.*nologin /etc/passwd
# any line with root
grep root /etc/passwd
root:x:0:0:root:/root:/bin/bash
# any line that begins with root
grep ^root /etc/passwd
root:x:0:0:root:/root:/bin/bash
# any line that does not end in nologin
grep -v nologin$ /etc/passwd

# EREs

# begin with either root or debus
$ grep -E "^root|^dbus" /etc/passwd

# egrep is equal to grep -e
$ egrep "(daemon|s).*nologin" /etc/passwd
```

## Formatting text

### sort 

Sorts a file's data. It does not change the file, only displays sorted output.

```bash
sort [OPTION... [FILE]...
-c # check if file is already sorted
-f # ignore case
-k n1 # sort file using data in n1, space is default delimiter
-M # month sort, must be JAN, FEB, etc format
-n # numerical sort
-o file # create new sorted file named file
-r # reverse sort
```

### cat 

```bash
cat [OPTION]... [FILE]...
-A # show all, equal to vaT
-E # display $ for new line feed (LF)
-n # add number to text file lines
-s # squeeze blank, don't display mult. blank lines
-T # display ^I for tab
-v # use ^ or M- for nonprinting characters. helps
   # find issues (other encodings) when processing text files 
```

### printf

Format printing for a single line of text.

```bash
printf FORMAT [ARGUMENT]...

printf "%s\n" "This is a string"
This is a string

printf "%.2f\n" "32.08294393"
32.08
```

### wc

Get the word count in a file:

```bash
wc [OPTION]... [FILE]...
-c # byte count
-L # byte count of longest line
-l # line count
-m # char count
-w # word count

# num of lines, words, bytes
wc random.txt 
5  9 51 random.txt

# check line length. config files are usually ~72-75 bytes long
wc -L /etc/nsswitch.conf 
75 /etc/nsswitch.conf
```

## Redirecting input and output

- In Linux, everything is a file, including the output process.
- Each file object is identified with a _file descriptor_, which is an integer that classifies a process's open files.

### Redirection operators

General rules:
- `>` is STDOUT
- `<` is STDIN
- `2>` is STDERR
- `&>` is STDOUT & STDERR

| Operator | Description | Example |
|----------|-------------|---------|
| `>` | STDOUT redirect | `grep nologin$ /etc/passwd > file.txt` |
| `>>` | STDOUT redirect append | `grep nologin$ /etc/passwd >> file.txt` |
| `2>` | STDERR | `grep -d skip hosts: /etc/* 2> /dev/null` |
| `2>>` | STDERR redirect append | `grep -d skip hosts: /etc/* 2> /dev/null` |
| `&>` | STDOUT and STDERR redirect |  |
| `<` | STDIN redirect | `tr " " "," < file.txt ` |
| `<>` | STDIN and STDOUT redirect | ...? |

### STDOUT

89,76,100,92,68,84,73


- The file descriptor for output is 1.
- Output is also called STDOUT
- By default, STDOUT directs output to your current terminal, which is represented by the `/dev/tty` file

### STDERR

- File descriptor for STDERR is 2
- By default, STDERR directs output to your current terminal, which is represented by the `/dev/tty` file

### STDIN

- File descriptor for STDIN is 0
- `tr` requires STDIN redirect

## Piping commands

- Redirects STDOUT, STDIN, and STDERR on the command line
- Any command in the pipeline has its STDOU redirected as STDIN for the next command

### tee

- Keep copy of command output to send to STDIN and view in STDOUT
- Based on a tee pipe fitting--the water flow is sent in multiple directions
  - You can save output to a file _and_ view it in STDOUT

```bash
grep /bin/bash$ /etc/passwd | tee BashUsers.txt
```

## Here documents

Also called 'here text' or 'heredoc'.

- Redirects multiple items into a command
- `<<` followed by keyword (often `EOF`)
  - The keyword indicates that data entry is complete

```bash
sort << EOF
> dog
> cat
> fish
> EOF
cat
dog
fish

cat << EOF > cat.txt
> This 
> is
> a
> here document
> EOF

cat cat.txt 
This 
is
a
here document
```

## Creating command lines

### xargs

Accepts STDIN from other commands as arguments to the xa`rgs argument:

```bash
# find all empty files in /tmp and ask permission to remove them
find tmp -size 0 | xargs -p /usr/bin/rm
```

### Shell expansion

Expand commands by wrapping them in parentheses and preceding it with a `$`. This syntax doesn't display the output to STDOUT, it executes the command and replaces it with its output:

```bash
# find empty files in tmp/ dir
ls $(find tmp -size 0)
tmp/EmptyFile1.txt  tmp/EmptyFile2.txt  tmp/EmptyFile3.txt
```

## Editing text files

### vim

vim loads a file into a memory buffer and lets you edit. There are 3 standard modes:
- **command mode**: Normal mode. You can move around the buffer area with the keyboard.
- **insert mode**: Edit or entry mode. Press `I` or `i`, and `--Insert--` will appear in the message area. Leave by pressing `Esc` key.
- **ex mode**: Colon command bc every command is preceded by a colon.

[vim cheat sheet](https://vimsheet.com/)

## Stream editors

A stream editor modifies text that is passed to it from a file or output from a pipeline. The stream editor makes text changes as the text "streams" through the editor utility.

### sed

Short for "stream editor", it edits a stream of text data based on commands either entered into the command line or stored in a text file:
1. Reads one line at a time from the input stream (STDIN by default)
2. Matches input text with supplied editor commands
3. Modifies the input text with editor commands
4. Outputs modified text to STDOUT
5. Reads the next line, and repeats

```bash
sed [OPTINS] [SCRIPT]... [FILENAME]
-e script # add script to processing. script is on command line
-f script # add script to processing. script is a file
-r # use extended regex in script



# s (substitute) command. everything after sed is [SCRIPT]
echo "i like cake" | sed 's/cake/pie/'

# doesn't modify all 'cake' occurrences
echo "i like cake and more cake" | sed 's/cake/pie/'
i like pie and more cake
# global modify
echo "i like cake and more cake" | sed 's/cake/pie/g'
i like pie and more pie

# replace all cake occurences in cake.txt in STDOUT only
sed 's/cake/donuts/' cake.txt 

# -e lets you use multiple scripts w sed. separate commands with ";"
sed -e 's/cake/donuts/ ; s/like/love/' cake.txt

# use a file with sed commands:
cat script.sed 
s/cake/donuts/
s/like/love/
# use -f command
sed -f script.sed cake.txt 
```

### gawk

`gawk` is more powerful than `sed` and can do the following:
- define vars
- use arithmetic and string operators
- use loops and logic
- create formatted reports from large datasets

`awk` is from the UNIX days, `gawk` is `awk` that was rewritten for GNU. If you use `awk` on a modern distribution, it calls `gawk`.

Easier to put `gawk` commands in a file:

```bash
gawk [OPTIONS] [PROGRAM]... [FILENAME]
-F d # specify delimiter
-f file # use file for gawk scripts
-s # sandbox mode

# $0 is entire line, $1 is first data field, etc
echo 'hello world' | gawk '{print $0}'
hello world
echo 'hello world' | gawk '{print $1}'
hello
echo 'hello world' | gawk '{print $2}'
world

# print first data field
gawk '{print $1}' cake.txt 
# read cake.txt, if data field is 'cake' change to 'donuts' and print the line
gawk '{if ($4 == "cake.") {$4="donuts"; print $0}}' cake.txt 

gawk -F : '{print $1}' /etc/passwd

# gawk in a file 
cat script.gawk 
{if ($4=="cake.")
    {$4="donuts"; print $0}
else if ($5=="cake.")
    {$5="donuts"; print $0}}

gawk -f script.gawk cake.txt 
```