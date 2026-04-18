---
title: "Text processing"
weight: 30
description: >
  Filtering, transforming, and extracting data from text files with grep, sed, awk, and the core text tools.
---

Most system data on Linux is stored as plain text. Log files record application events, configuration files control behavior, CSV exports carry database contents, and `/proc` files expose kernel state. The command line provides a set of composable tools for filtering, transforming, and extracting data from these files without requiring a database query language or spreadsheet software.

The real power is in combining these tools in pipelines. Each command in a pipeline does one thing and does it well. A two-minute investigation of a production issue often comes down to three or four of these commands chained together. This page covers the tools you will reach for most frequently.

## Viewing and counting text

### wc

`wc` counts lines, words, and characters in a file or from stdin. A newline counts as one character.

The following table shows the common options:

| Option | Counts |
|:-------|:-------|
| `-l`   | Lines only |
| `-w`   | Words only |
| `-c`   | Characters only |

Count lines in a log file to check whether it has grown since last night:

```bash
wc -l /var/log/auth.log             # 1274 /var/log/auth.log
wc -w /etc/nginx/nginx.conf         # word count of a config file
ls -1 /etc | wc -l                  # number of entries in /etc
```

### head

`head` prints the first 10 lines of a file. Specify a different number with `-n`:

```bash
head -20 /var/log/syslog            # first 20 lines
ls /bin | head -5                   # first 5 filenames in /bin
head -3 access.log | wc -w          # count words in the first 3 lines
```

### tail

`tail` prints the last 10 lines of a file. Pass `-f` to follow a log file in real time as new lines are written:

```bash
tail -f /var/log/nginx/error.log                    # live log watching
tail -n 50 /var/log/syslog                          # last 50 lines
tail -n +3 /etc/hosts                               # everything starting at line 3
head -10 /var/log/auth.log | tail -3                # lines 8, 9, and 10
```

### cat and tac

`cat` (concatenate) prints files in order from top to bottom. `tac` prints them in reverse, from bottom to top:

```bash
cat file1.txt file2.txt > combined.txt
tac /var/log/apache2/access.log | head -20    # most recent 20 requests
```

`tac` is useful for logs that are in chronological order but cannot be reversed with `sort -r`, because the sort key appears mid-line.

## Extracting columns and fields

### cut

`cut` extracts columns from a line. Specify columns by field number (tab-delimited by default) or by character position.

The following table shows the main options:

| Option        | Behavior |
|:--------------|:---------|
| `-fN`         | Print field N (tab-delimited) |
| `-f2,4`       | Print fields 2 and 4 |
| `-f2-4`       | Print fields 2 through 4 |
| `-cN`         | Print character N |
| `-c2-4`       | Print characters 2 through 4 |
| `-d,`         | Change the field delimiter to `,` |

Extract the username field from `/etc/passwd`:

```bash
cut -d: -f1 /etc/passwd             # first field, colon-delimited
cut -d: -f1,7 /etc/passwd           # usernames and shells
cut -f1-3 report.tsv                # first three tab-delimited columns
```

## Combining text

### paste

`paste` combines files side by side, separating columns with a tab by default:

```bash
paste usernames.txt emails.txt                      # tab-delimited
paste -d, names.txt scores.txt                      # comma-delimited
paste -d "\n" list1.txt list2.txt                   # interleave lines from two files
```

### diff

`diff` compares two files and prints their differences. The output notation uses `<` for lines only in the first file and `>` for lines only in the second:

```bash
diff /etc/hosts /etc/hosts.backup
1c1                                 # line 1 in file 1 differs from line 1 in file 2
< 192.168.1.100 web-01
---
> 192.168.1.101 web-01
```

Filter the output to see only the changed lines without the context markers:

```bash
diff config.current config.backup | grep '^[<>]'
diff config.current config.backup | grep '^[<>]' | cut -c3-     # remove the < > prefix
```

## Transforming text

### tr

`tr` translates characters: it takes two sets of characters and replaces each character in the first set with the corresponding character in the second set. Pass `-d` to delete characters instead.

Convert a colon-delimited string to newlines for readable output:

```bash
echo $PATH | tr ':' '\n'
```

Remove all spaces from a string:

```bash
echo "hello world" | tr -d ' '     # helloworld
```

Convert uppercase to lowercase:

```bash
echo "SERVER-01" | tr '[A-Z]' '[a-z]'    # server-01
```

### rev

`rev` reverses the characters on each line. This is useful when you need to extract the last field from lines that have varying numbers of columns. For example, extract the last word from each line regardless of how many words precede it:

```bash
rev /etc/shells | cut -d'/' -f1 | rev       # extract shell name from each path
```

## Sorting and deduplication

### sort

`sort` reorders lines in ascending alphabetical order by default.

The following table shows common options:

| Option | Behavior |
|:-------|:---------|
| `-r`   | Reverse (descending) order |
| `-n`   | Numeric sort |
| `-nr`  | Numeric descending |
| `-u`   | Remove duplicate lines |
| `-f`   | Ignore case |
| `-k N` | Sort by field N (whitespace-delimited) |
| `-t ,` | Change field delimiter to `,` |
| `-o`   | Write output to the specified file |

Sort a log file by the third column (for example, HTTP status code):

```bash
sort -k9 -n access.log              # sort by status code (field 9)
sort -k3 -rn report.txt             # sort by field 3, numeric descending
cut -f3 data.tsv | sort -nr         # sort the third column numerically
```

Sort by characters 4 and 5 of field 2 (useful for date fields):

```bash
sort -k 2.4,2.5 filename.txt
```

### uniq

`uniq` removes adjacent duplicate lines. Always `sort` first so that duplicates are adjacent.

The following table shows common options:

| Option | Behavior |
|:-------|:---------|
| `-c`   | Prefix each line with the count of occurrences |
| `-d`   | Print only lines that appear more than once |
| `-u`   | Print only lines that appear exactly once |
| `-i`   | Ignore case when comparing |
| `-f N` | Skip the first N fields before comparing |

Find the most common HTTP status codes in an access log:

```bash
awk '{print $9}' /var/log/apache2/access.log | sort | uniq -c | sort -rn | head -10
```

Find the grade that appears most often in a grades file:

```bash
cut -f1 grades | sort | uniq -c | sort -nr | head -1 | cut -c9
```

## Pattern matching with grep

`grep` searches files for lines that match a pattern and prints each matching line. It is the most frequently used filtering tool on the command line.

```bash
grep [OPTIONS] PATTERN [FILE...]
```

The following table shows common options:

| Option  | Behavior |
|:--------|:---------|
| `-c`    | Print a count of matching lines instead of the lines themselves |
| `-E`    | Enable extended regular expressions (ERE) |
| `-f`    | Read patterns from a file |
| `-i`    | Ignore case |
| `-l`    | Print only filenames of files with at least one match |
| `-n`    | Print the line number before each match |
| `-o`    | Print only the matching text, not the full line |
| `-q`    | Silent mode -- exit status only, no output |
| `-r`    | Recursively search directories |
| `-R`    | Recursive search, following symbolic links |
| `-v`    | Invert match: print lines that do not match |
| `-w`    | Match whole words only |
| `-P`    | Enable Perl-compatible regular expressions |

Search for failed SSH login attempts in the auth log:

```bash
grep "Failed password" /var/log/auth.log
grep -c "Failed password" /var/log/auth.log     # count failures
grep -i "error" /var/log/syslog                  # case-insensitive
grep -R "PermitRootLogin" /etc/ssh/             # recursive config search
grep -v "^#" /etc/nginx/nginx.conf              # non-comment lines only
```

Double-filter to find root login failures specifically:

```bash
grep "authenticating" /var/log/auth.log | grep "root"
```

### Anchors and patterns

Pattern anchors control where in the line a match must occur:

```bash
grep ^root /etc/passwd          # lines that begin with "root"
grep nologin$ /etc/passwd       # lines that end with "nologin"
grep -v '^$' file.txt           # non-blank lines (match blank lines and invert)
grep '......' file.txt          # lines with at least 6 characters
grep daemon.*nologin /etc/passwd
```

### Extended regular expressions

Pass `-E` (or run `egrep`) to enable extended regex, which adds support for `|`, `+`, `?`, `{`, and `(`:

```bash
grep -E "^root|^dbus" /etc/passwd               # lines beginning with root or dbus
grep -E '^(web|app|db)-[0-9]+' /etc/hosts       # match server naming patterns
egrep "(daemon|nobody).*nologin" /etc/passwd
```

### Regex reference

The following table describes regex metacharacters:

| Metacharacter | Matches |
|:--------------|:--------|
| `.`           | Any single character except newline |
| `*`           | Zero or more of the preceding character |
| `+`           | One or more of the preceding character (ERE) |
| `?`           | Zero or one of the preceding character (ERE) |
| `^`           | Start of line |
| `$`           | End of line |
| `[abc]`       | Any character in the set |
| `[^abc]`      | Any character not in the set |
| `[a-z]`       | Any character in the range |
| `\`           | Escape the following character |

The following table describes POSIX character classes for use inside `[[ ]]`:

| Class        | Matches |
|:-------------|:--------|
| `[:alnum:]`  | Alphanumeric characters |
| `[:alpha:]`  | Alphabetic characters |
| `[:digit:]`  | Digits |
| `[:lower:]`  | Lowercase letters |
| `[:upper:]`  | Uppercase letters |
| `[:space:]`  | Whitespace including line breaks |
| `[:punct:]`  | Punctuation |
| `[:xdigit:]` | Hexadecimal digits |

### Perl regex shortcuts

These shortcuts require the `-P` flag:

| Shortcut | Matches |
|:---------|:--------|
| `\s`     | Any whitespace |
| `\S`     | Any non-whitespace |
| `\d`     | Any digit |
| `\D`     | Any non-digit |
| `\w`     | Word character (letter, digit, underscore) |
| `\W`     | Non-word character |

### Quantifiers

Quantifiers control how many times the preceding expression must match:

```bash
grep -E 'T{5}'    file   # T appears exactly 5 times consecutively
grep -E 'T{3,6}'  file   # T appears 3 to 6 times
grep -E 'T{5,}'   file   # T appears 5 or more times
```

### Back references

A back reference lets you refer to a previous capture group. This example matches any HTML opening and closing tag pair where the tag names match:

```bash
egrep '<([a-zA-Z]*)>.*</\1>' file.html
```

The `\1` means "whatever was matched inside the first set of parentheses."

## Stream editing with sed

`sed` transforms text by applying a sequence of instructions called a *sed script* to each line of input. The most common script replaces one string with another:

```bash
sed 's/regexp/replacement/' input-file
```

The `s` command replaces the first occurrence on each line. Append `g` to replace all occurrences:

```bash
echo "one one two" | sed 's/one/yes/g'          # yes yes two
```

The following table shows common sed commands:

| Script                     | Effect |
|:---------------------------|:-------|
| `s/old/new/`               | Replace first occurrence per line |
| `s/old/new/g`              | Replace all occurrences per line |
| `s/old/new/i`              | Case-insensitive replacement |
| `s/old/new/gI`             | Global case-insensitive replacement |
| `2s/old/new/`              | Replace only on line 2 |
| `s/old/new/3`              | Replace only the third occurrence on each line |
| `-i`                       | Modify the file in place |
| `d`                        | Delete matching lines |
| `/pattern/d`               | Delete lines matching a pattern |
| `nd`                       | Delete line number n |
| `-n 'Np'`                  | Print only line N |
| `-n '/pattern/p'`          | Print lines matching a pattern |
| `y/abc/xyz/`               | Translate characters (like `tr`) |

Replace a hostname in a configuration file:

```bash
sed -i 's/old-server.internal/new-server.internal/g' /etc/app/config.conf
```

Delete comment lines and blank lines from a config file:

```bash
sed -e '/^#/d' -e '/^$/d' /etc/nginx/nginx.conf
```

Print a specific line range:

```bash
sed -n '5,10p' /var/log/syslog      # print lines 5 through 10
```

### Subexpressions

Subexpressions let you capture and rearrange parts of a matched string. Wrap the part you want to capture in `\(` and `\)`, then reference it in the replacement with `\1`, `\2`, and so on.

For example, reformat a date string from `YYYY-MM-DD` to `DD/MM/YYYY`:

```bash
echo "2025-03-29" | sed 's/\([0-9]*\)-\([0-9]*\)-\([0-9]*\)/\3\/\2\/\1/'
# 29/03/2025
```

Rename image files by moving a version number from the end of the name to before the extension:

```bash
sed "s/image\.jpg\.\([1-3]\)/image\1.jpg/"
```

### Long regex patterns

Break a long regex into named shell variables for readability:

```bash
areacode='\([0-9]*\)'
state='\([A-Z][A-Z]\)'
city='\([^@]*\)'

regexp="${areacode}@${state}@${city}@"
replacement='\1\t\2\t\3\n'

sed "s/$regexp/$replacement/g" data.txt
```

## Report generation with awk

`awk` processes structured text files and generates reports. It reads each line, splits it into fields, and runs your *awk program* against each line. Fields are referenced as `$1`, `$2`, and so on. `$0` is the entire line and `$NF` is the last field.

```bash
awk '{print $2}' /etc/hosts                     # print second column of hosts file
awk -F: '{print $1}' /etc/passwd                # usernames, colon-delimited
df | awk 'FNR>1 {print $4}'                     # available space, skip header
```

### Filtering with patterns

Run an awk program only on lines that match a pattern:

```bash
awk '/ERROR/' /var/log/app.log                  # lines containing ERROR
awk '!/^#/' /etc/hosts                          # non-comment lines
awk '$9 >= 500' access.log                      # HTTP 5xx errors (field 9 is status code)
awk 'NR>=10 && NR<=20' file.txt                 # lines 10 through 20
```

### BEGIN and END blocks

`BEGIN` runs before processing any lines. `END` runs after all lines have been processed. Both are useful for printing headers, footers, and computed summaries:

```bash
awk -F'\t' \
'BEGIN {print "Recent entries:"} \
$3~/^2025/{print $4, "(" $3 ").", "\"" $2 "\""} \
END {print "End of report"}' \
data.tsv
```

Sum a column and print the total:

```bash
seq 1 100 | awk '{s+=$1} END {print "Total:", s}'     # Total: 5050
```

### Arrays and loops in awk

`awk` arrays act as hash maps indexed by any string key. This makes them useful for counting occurrences:

```bash
awk '{counts[$9]++} END {for (code in counts) print counts[code], code}' access.log
```

The previous command counts HTTP status codes across an access log. `counts[$9]++` increments the count for each status code value in field 9. The `END` block prints each status code and its count.

Find duplicate files by checksum:

```bash
md5sum *.jpg | awk '{counts[$1]++} END {for (key in counts) print counts[key], key}' | sort -rn
```

### awk cheat sheet

The following table shows common awk programs:

| Program | Description |
|:--------|:------------|
| `'{print $1}'` | Print first column |
| `'{print $1, $3}'` | Print columns 1 and 3 |
| `'NR==3'` | Print line 3 |
| `'NR>=2 && NR<=4'` | Print lines 2 to 4 |
| `'/pattern/'` | Print lines matching pattern |
| `'!/pattern/'` | Print lines not matching pattern |
| `'{print NR, $0}'` | Print lines with line numbers |
| `'$2 > 50'` | Print lines where field 2 is greater than 50 |
| `'{sum+=$1} END {print sum}'` | Sum field 1 |
| `'{sum+=$1} END {print sum/NR}'` | Average of field 1 |
| `'BEGIN {FS=","} {print $1}'` | Comma-delimited: print first field |
| `'{print toupper($1)}'` | Convert first field to uppercase |
| `'{sub(/old/, "new"); print}'` | Replace first occurrence per line |
| `'{gsub(/old/, "new"); print}'` | Replace all occurrences per line |
| `'{print $NF}'` | Print last field |

## Generating text

### date

`date` prints the current date and time in any format you specify. Format strings begin with `+` and contain `%` sequences:

The following table shows common format strings:

| Format string       | Example output           | Description |
|:--------------------|:-------------------------|:------------|
| `%Y-%m-%d`          | 2025-03-29               | ISO 8601 date |
| `%d-%m-%Y`          | 29-03-2025               | Day-Month-Year |
| `%m/%d/%Y`          | 03/29/2025               | Month/Day/Year (US) |
| `%A, %B %d, %Y`     | Saturday, March 29, 2025 | Full date with names |
| `%H:%M:%S`          | 14:30:15                 | 24-hour time |
| `%Y-%m-%d %H:%M:%S` | 2025-03-29 14:30:15      | Full timestamp |
| `%s`                | 1743456615               | Unix epoch seconds |

Generate a timestamped filename for a log archive:

```bash
tar -czf "backup_$(date +%Y-%m-%d).tar.gz" /var/www/html
```

### seq

`seq` prints a sequence of numbers. The basic form is `seq LOW HIGH`. Add a third argument between them to set the step:

```bash
seq 1 5             # 1 2 3 4 5
seq 1 2 10          # 1 3 5 7 9 (odd numbers)
seq 10 -1 1         # 10 9 8 ... 1 (countdown)
seq 0 0.5 2         # 0 .5 1.0 1.5 2.0
seq -w 1 5          # 1 2 3 4 5 with zero-padded width
seq -s, 1 5         # 1,2,3,4,5 (comma-separated)
```

### Brace expansion

Brace expansion generates sequences of numbers or letters directly in the shell without a separate program:

```bash
echo {1..5}                     # 1 2 3 4 5
echo {1..10..2}                 # 1 3 5 7 9 (step of 2)
echo {a..z}                     # a b c ... z
echo {A..Z} | tr -d ' '         # ABCDEFGHIJKLMNOPQRSTUVWXYZ (no spaces)
echo {A..Z} | tr ' ' '\n'       # one letter per line
```

Brace expansion is useful for creating numbered files or directories in bulk:

```bash
mkdir -p logs/{jan,feb,mar,apr,may,jun,jul,aug,sep,oct,nov,dec}
touch report_{2023..2025}.txt
```

### yes

`yes` prints the same line repeatedly until killed. By default it prints `y`. Pipe it to a command that requires interactive confirmation, or pipe it to `head` to generate repeated test input:

```bash
yes | apt-get install -y package-name    # confirm all prompts automatically
yes "test data" | head -100              # generate 100 lines of test data
```

## Checksums and duplicate detection

### md5sum and sha1sum

`md5sum` computes a 32-character hash (checksum) of a file's contents. Files with identical contents produce identical checksums. `sha1sum` produces a 40-character hash and is more collision-resistant:

```bash
md5sum /etc/hosts                       # hash the file
sha1sum deployment.tar.gz              # verify a downloaded archive
```

Verify a file's integrity by comparing against a published checksum:

```bash
sha1sum -c checksums.txt               # -c reads a file of hash: filename pairs
```

### Detecting duplicate files

This pipeline finds duplicate files in the current directory by comparing checksums. Walk through each step to understand how it builds:

Step 1: compute the checksum for all `.txt` files:

```bash
md5sum *.txt | cut -d' ' -f1
```

Step 2: sort so that identical checksums are adjacent:

```bash
md5sum *.txt | cut -d' ' -f1 | sort
```

Step 3: count occurrences of each checksum:

```bash
md5sum *.txt | cut -d' ' -f1 | sort | uniq -c
```

Step 4: sort by count descending so duplicates appear first:

```bash
md5sum *.txt | cut -d' ' -f1 | sort | uniq -c | sort -rn
```

Step 5: filter out non-duplicates (count of 1):

```bash
md5sum *.txt | cut -d' ' -f1 | sort | uniq -c | sort -rn | grep -v "^      1 "
```

Once you have a checksum for a duplicate, find the filenames:

```bash
md5sum *.txt | grep "5bbf5a52328e7439ae6e719dfe712200" | cut -d' ' -f3
```

## Real-world pipeline example: Apache log analysis

An Apache access log has fields for IP address, timestamp, HTTP method, path, status code, and bytes transferred. This section shows how to investigate a spike in 404 errors.

Filter only 404 lines for today's date:

```bash
grep "$(date +%d/%b/%Y)" /var/log/apache2/access.log | grep ' 404 '
```

Find the top 15 paths generating 404 errors:

```bash
grep ' 404 ' /var/log/apache2/access.log | awk '{print $7}' | sort | uniq -c | sort -rn | head -15
```

Find the IP addresses making the most requests that result in 404 errors:

```bash
grep ' 404 ' /var/log/apache2/access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -10
```

Check whether a single IP is making an unusually high number of 401 errors (failed authentication):

```bash
grep ' 401 ' /var/log/apache2/access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -5
```

## Leveraging text files for recurring tasks

When data is structured and stored in a text file, you can write commands once and rerun them on updated data. The general process is:

1. Identify the business problem that involves data.
2. Store the data in a plain text file in a consistent format.
3. Write Linux commands to process the file and solve the problem.
4. Capture the commands in a script so they are easy to repeat.

A practical example: check domain expiration dates from a list. The following script queries each domain's registrar with `whois` and extracts the expiry date:

```bash
#!/bin/bash
expdate=$(date \
            --date "$(whois "$1" \
                | grep 'Registry Expiry Date:' \
                | awk '{print $4}')" \
            +'%Y-%m-%d')
echo "$expdate $1"
```

Call that script from a loop that reads from a file of domain names:

```bash
while read -r domain; do
    ./check-expiry "$domain"
    sleep 5
done < domains.txt
```
