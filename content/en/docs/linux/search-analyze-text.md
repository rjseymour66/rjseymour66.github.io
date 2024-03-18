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
-E # Exended regex
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

## Redirecting output

