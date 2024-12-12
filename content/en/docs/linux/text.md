---
title: "Text tools and processing"
linkTitle: "Text"
# weight: 1000
# description:
---


## Read entire file

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

### pr

Display file with special formatting.

```bash
pr [OPTION]... [FILE]...
-n # display file in n columns
-l <n> # default lines displayed
-m # merge multiple files into one
-s <c> # change default column separator to c
-t # no file header or trailer
-w <n> # change default page width to n

pr -tl 15 filename.txt

# great for comparing files (merging output)
pr -mtl 15 file1.txt file2.txt
```

## Read portion of text files

### grep

Search for text patterns or a single string in a file.

Uses regex, a pattern template that you define for a utility like grep that filters text. Basic regex (BREs) include `.*` and `^r`. Extended regex (EREs) let you specify two words or character sets to match with the `|` char.

```bash
grep [OPTIONS] PATTERN [FILE...]
-c # display record count that match PATTERN
-d action # read, skip, or recurse on dirs
-E # Extended regex
-F # ignore special chars
-i # ignore case
-o # display match only, not full line text
-r # recursive
-R # recursive, including symbolic links
-v # match only files that do not match PATTERN
-w # find a word surrounded by whitespace or puncutation

# find root in passwd file, case-insensitive
grep -i root /etc/passwd

# search for 'hosts:' in all /etc/ files, skipping dirs (-d skip)
grep -d skip hosts: /etc/*

# double filter with grep
grep "authenticating" /var/log/auth.log | grep "root"

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

# begin with either l or o
grep -E '^(l|o)' greptest.txt 

# begins w letter between l and u, inclusive
grep -E '^[l-u]' greptest.txt

# egrep is equal to grep -e
$ egrep "(daemon|s).*nologin" /etc/passwd

# fgrep (fixed strings) - doesn't recognize special characters
grep -F o$ greptest.txt 
o$technix

# search dir recursively for string (plain text only)
grep -R -i "PermitRootLogin" /etc/*

grep -w <word>                  # search for one word              
grep -w <word1> <word2> <dir>   # search for multiple words in the dir

grep -o <word> <dir>            # display only the matching word, not entire line of text
grep -w "[A-Z]+{5,}" <file>     # search a file for an all caps word at least 5 chars long                 


```

### head

Display first lines of a file (10 by default).

```bash
head [OPTION]... [FILE]...

# these two are equivalent
head -n 2 file.txt
head -2 file.txt

# display all file lines except the last 40
head -n -40 file.txt
```

### tail 

Display last lines of a file (10 by default) and watching log files:

```bash
tail [OPTION]... [FILE]...

# these are equivalent
tail -n 2 file.txt
tail -2 file.txt

# begin display at line 40 of file
tail -n +40 file.txt

# Very useful to watch log files with `-f` (follow) option:
tail -f /var/log/auth.log
```

## Processing text

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
-f 10-      # display field 10 to end of line
-s          # display only records that have the delimiter
-z          # change ASCII EOL char to NUL

# display fields 1 and 7, delimited by colon
cut -d ":" -f 1,7 /etc/passwd

# display chars 1-5 from file
cut -c 1-5 /etc/passwd
```

### uniq

## Formatting text

### sort 

Sorts a file's data. It does not change the file, only displays sorted output.

```bash
sort [OPTION... [FILE]...
-b # ignore leading white space
-c # check if file is already sorted
-f # ignore case
-k n1 # sort file using data in n1, space is default delimiter
-M # month sort, must be JAN, FEB, etc format
-n # numerical sort
-o file # create new sorted file named file
-r # reverse sort
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
| `2>&1` | Redirects STDERR to same dest as STDOUT |  |
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

## Stream editors

A stream editor modifies text that is passed to it from a file or output from a pipeline. The stream editor makes text changes as the text "streams" through the editor utility.

### sed

[Handy one-liners for SED](https://edoras.sdsu.edu/doc/sed-oneliners.html)

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

### awk and gawk

[AWK one-liner collection](http://tuxgraphics.org/~guido/scripts/awk-one-liner.html)

`awk` is from the UNIX days, `gawk` is `awk` that was rewritten for GNU. If you use `awk` on a modern distribution, it calls `gawk`.

`gawk` is more powerful than `sed` and can do the following:
- define vars
- use arithmetic and string operators
- use loops and logic
- create formatted reports from large datasets

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

# print last field in each line in file.txt
awk -F: '{ print $NF }' file.txt

# print every line and delete 2nd field in file.txt
 awk '{ $2 = ""; print }' file.txt

# gawk in a file 
cat script.gawk 
{if ($4=="cake.")
    {$4="donuts"; print $0}
else if ($5=="cake.")
    {$5="donuts"; print $0}}

gawk -f script.gawk cake.txt 
```