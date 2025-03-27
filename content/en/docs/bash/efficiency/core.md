---
title: "Core commands"
linkTitle: "Core"
weight: 10
# description:
---

The Linux command line uses lots of small apps that do a single thing with minimal features or options. You can combine these commands with pipes (`|`) to perform complex tasks. You do this by connecting the stdout of one command with the stdin of another. Any command that contains pipes is called a _pipeline_.

## Commands

### ls


`ls` behaves differently depending on its stdout. If stdout is a screen, it formats the output for human consumption. When its stdout is redirected, it produces a single column output::

```bash
# stdout is console
ls /bin
'['                          dbus-send                            grub-script-check       migratepages                       pwd                     sg_reset_wp                      tnftp
 aa-enabled                  dbus-update-activation-environment   grub-syslinux2cfg       migrate-pubring-from-classic-gpg   pwdx                    sg_rmsn                          toe
 aa-exec                     dbus-uuidgen                         gsettings               migspeed                           py3clean                sg_rtpg                          top
...

# stdout redirected
ls /bin | cat
[
aa-enabled
aa-exec
aa-features-abi


ls -1                   # force single column
ls -C                   # force 1 line
```

Keep this in mind when working with pipelines:
```bash
ls                                      # outputs 1 line
animals.txt  myfile  myfile2  test.py
ls | wc -l                              # outputs 4 lines in pipeline
4

ls -C | wc -l                           # force 1 line with redirect
1
```


### wc

Prints the number of lines, words, and characters in a file:
- `/n` counts as a character

```bash
wc animals.txt 
  7  51 325 animals.txt         # 7 lines, 51 words, 325 chars
wc -l animals.txt               # lines only
wc -w animals.txt               # words only
wc -c animals.txt               # characters only

ls -1 | wc -l                   # count number of files in cwd
wc animals.txt | wc -w          # count number of words in wc output
      1       1       2         # /n counts as a char!
```

### head

Prints the first 10 lines of a file. Specify the number of lines with `-n[N]` or `-[N]`:
- If the file is shorter than the line number you specify, it prints the whole file.
- Good to reduce the number of output lines

```bash
head -3 file.txt | wc -w        # count words in first 3 lines of file.txt
ls /bin | head -5               # first 5 filenames in /bin
```

### cut

Prints one or more columns from a file. Specify the column with one of these syntaxes:
- Field syntax: `-fN`, `-fN-N`. Useful when fields are tab-delimited strings. Specify column (`-f2,4`) or range (`-f2-4`)
- Character syntax: `-cN`. Specify with commas (`-c2,3,4`) or range (`-c2-4`)

Change the delimited from string to a different character with `-d[delimiter]`

```bash
# field syntax
cut -f2 file.txt                # print the second column
cut -f2,3 file.txt              # print second and third column
cut -f1-3 file.txt              # print first through third column

# character syntax
cut -c1,2,3 file.txt            # print first three chars
cut -c1-3 file.txt              # print first three chars

# change delimiter
cut -d, -f1                     # change delimiter to comma, print first field
```

### grep

Search for text patterns or a single string in a file.

Uses regex, a pattern template that you define for a utility like grep that filters text. Basic regex (BREs) include `.*` and `^r`. Extended regex (EREs) let you specify two words or character sets to match with the pipe (`|`) character:

```shell
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

grep -i root /etc/passwd                                    # find root in passwd file, case-insensitive
grep -d skip hosts: /etc/*                                  # search for 'hosts:' in all /etc/ files, skipping dirs (-d skip)
grep "authenticating" /var/log/auth.log | grep "root"       # double filter with grep
grep daemon.*nologin /etc/passwd

grep root /etc/passwd                                       # any line with root
root:x:0:0:root:/root:/bin/bash

grep ^root /etc/passwd                                      # any line that begins with root
root:x:0:0:root:/root:/bin/bash

grep -v nologin$ /etc/passwd                                # any line that does not end in nologin

ls -l /usr/lib | cut -c1 | grep d | wc -l                   # number of dirs in /usr/lib

### --- EREs --- ###

grep -E "^root|^dbus" /etc/passwd                           # begin with either root or debus
grep -E '^(l|o)' greptest.txt                               # begin with either l or o
grep -E '^[l-u]' greptest.txt                               # begins w letter between l and u, inclusive
egrep "(daemon|s).*nologin" /etc/passwd                     # egrep is equal to grep -e
grep -F o$ greptest.txt                                     # fgrep (fixed strings) - doesn't recognize special characters
o$technix
grep -R -i "PermitRootLogin" /etc/*                         # search dir recursively for string (plain text only)

grep -w <word>                                              # search for one word              
grep -w <word1> <word2> <dir>                               # search for multiple words in the dir

grep -o <word> <dir>                                        # display only the matching word, not entire line of text
grep -w "[A-Z]+{5,}" <file>                                 # search a file for an all caps word at least 5 chars long
```

### sort

Reorders the lines of a file in ascending order, by default. You can also sort:
- `-r`: descending alpha
- `-n`: numerical ascending
- `-nr`: numerical descending 

```bash
sort file.txt               # ascending alpha
sort -r file.txt            # descending alpha
sort -n file.txt            # ascending numeric
sort -nr file.txt           # descending numeric
cut -f3 file.txt | sort -nr # sort column 3 by descending numeric
```

### uniq

Detects repeated, adjacent lines in a file and removes repeats by default. For example, if you have a file that contains multiple lines of equal value, `uniq` only displays the value once:

- `cut` lets you filter lines by field or character to find unique values
- `sort` gets non-adjacent values in order

**The `-c` option prepends 6 empty characters to the line.**

```bash
cat letters 
A
A
B
B
A                   # not adjacent to other As
C
C
C
uniq letters 
A
B
A                   # left A here because its not adjacent
C

uniq -c letters     # count occurrences
      2 A
      2 B
      1 A
      3 C


# pipeline example
cat grades
C	Geraldine
B	Carmine
A	Kayla
A	Sophia
B	Haresh
C	Liam
B	Elijah
B	Emma
A	Olivia
D	Noah
F	Ava

cut -f1 grades | sort | uniq -c | sort -nr | head -1 | cut -c9
B
```

### md5sum

Examines a file's contents and computes a 32-character string called a checksum. Checksums for files with the same contents are equal, otherwise they are unique.

In this example, `one.txt` and `one-too.txt` have the same contents, so they have the same checksum. `two.txt` has different contents, so it has a different checksum:

```bash
cat one.txt && cat one-too.txt && cat two.txt 
one     # one.txt
one     # one-too.txt
two     # two.text

md5sum one.txt && md5sum one-too.txt && md5sum two.txt 
5bbf5a52328e7439ae6e719dfe712200  one.txt
5bbf5a52328e7439ae6e719dfe712200  one-too.txt
c193497a1a06b2c72230e6146ff47080  two.txt
```

## Detecting duplicate files



```bash
# 1. First, get the checksum for all files with one of these options

md5sum *.txt | cut -c1-32               # character syntax (checksum is 32 chars long)
f6957bacbc9fc247f9c50f5b92702f53
5bbf5a52328e7439ae6e719dfe712200
5bbf5a52328e7439ae6e719dfe712200
c193497a1a06b2c72230e6146ff47080

md5sum *.txt | cut -d' ' -f1            # field syntax with changed delimiter
f6957bacbc9fc247f9c50f5b92702f53
5bbf5a52328e7439ae6e719dfe712200
5bbf5a52328e7439ae6e719dfe712200
c193497a1a06b2c72230e6146ff47080

# 2. Sort so dupes are adjacent

md5sum *.txt | cut -d' ' -f1 | sort
5bbf5a52328e7439ae6e719dfe712200
5bbf5a52328e7439ae6e719dfe712200
c193497a1a06b2c72230e6146ff47080
f6957bacbc9fc247f9c50f5b92702f53

# 3. Get unique checksums with a count:

md5sum *.txt | cut -d' ' -f1 | sort | uniq -c
      2 5bbf5a52328e7439ae6e719dfe712200
      1 c193497a1a06b2c72230e6146ff47080
      1 f6957bacbc9fc247f9c50f5b92702f53

# 4. Sort again to put dupes at top:

md5sum *.txt | cut -d' ' -f1 | sort | uniq -c | sort -nr
      2 5bbf5a52328e7439ae6e719dfe712200
      1 f6957bacbc9fc247f9c50f5b92702f53
      1 c193497a1a06b2c72230e6146ff47080

# 5. Use grep -v to omit checksums that begin with "1":

md5sum *.txt | cut -d' ' -f1 | sort | uniq -c | sort -nr | grep -v "      1"
      2 5bbf5a52328e7439ae6e719dfe712200

```
Now, you can `grep` for the a file with a specific checksum:

```bash
md5sum *.txt | grep "5bbf5a52328e7439ae6e719dfe712200"
5bbf5a52328e7439ae6e719dfe712200  one-too.txt
5bbf5a52328e7439ae6e719dfe712200  one.txt

# grep for the filename:
md5sum *.txt | grep "5bbf5a52328e7439ae6e719dfe712200" | cut -d' ' -f3      # field syntax
one-too.txt
one.txt


$ md5sum *.txt | grep "5bbf5a52328e7439ae6e719dfe712200" | cut -c35-        # character syntax
one-too.txt
one.txt
```