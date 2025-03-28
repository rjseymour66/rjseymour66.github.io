---
title: "Producing, isolating, combining, and transforming text"
linkTitle: "Text commands"
weight: 50
# description:
---

## Producing text

### date

Prints dates and times in various formats. Formatting starts with plus sign (`+`) and uses special expressions `%X`:

| Format String       | Example Output           | Description                          |
| ------------------- | ------------------------ | ------------------------------------ |
| `%Y-%m-%d`          | 2025-03-29               | ISO 8601 (Year-Month-Day)            |
| `%d-%m-%Y`          | 29-03-2025               | Day-Month-Year (common in Europe)    |
| `%m/%d/%Y`          | 03/29/2025               | Month/Day/Year (common in the US)    |
| `%Y/%m/%d`          | 2025/03/29               | Year/Month/Day                       |
| `%A, %B %d, %Y`     | Saturday, March 29, 2025 | Full weekday, month name, day, year  |
| `%a, %b %d`         | Sat, Mar 29              | Abbreviated weekday and month        |
| `%H:%M:%S`          | 14:30:15                 | 24-hour time (HH:MM:SS)              |
| `%I:%M:%S %p`       | 02:30:15 PM              | 12-hour time with AM/PM              |
| `%Y-%m-%d %H:%M:%S` | 2025-03-29 14:30:15      | Full timestamp in ISO 8601 format    |
| `%s`                | 1743456615               | Unix timestamp (seconds since epoch) |


### seq

Prints a  sequence of numbers:
- requires 2 args, low and high value of sequence
- if you add a third arg, it is `seq <low> <increment> <high>`
- negative number to produce descending sequence
- decimal increment for floating point numbers
- `-s` option changes the separator. By default, separated by newline
- `-w` makes each value the same width with leading zeroes

```bash
seq <low> [<increment>] <high>

seq 1 5             # 1 to 5
seq 1 2 10          # 1 to 10, increments of 2
seq 2 2 20          # 2 to 20, evens only
seq 10 -1 -10       # countdown from 10 to -10
seq 2.1 .01 2.5     # 2.10 to 2.50, increments of .01
seq -sx 1 10        # 1x2x3x4x5x6x7x8x9x10
seq -w 2 2 100      # 2 to 100, evens 002 - 100
```

### Brace expansion

Shell feature that prints a sequence of numbers or characters:
- numbers and letters
- produces a single line separated by a space
- change piping output with `tr` (translate)
- separate the low and high values with 2 dots `..`
- add a third value as the increment: `{x..y..z}`

```bash
echo {1..5}                 # 1 2 3 4 5
echo {1..10..2}             # 1 3 5 7 9
echo {2..10..2}             # 2 4 6 8 10
echo {10..-10..-1}          # 10 9 8...-8 -9 -10
echo {a..z}                 # a b c d...w x y z
echo {A..Z..2}              # A C E G I K M O Q S U W Y
echo {A..Z} | tr -d ' '     # ABCDEFGHIJKLMNOPQRSTUVWXYZ
echo {A..Z} | tr ' ' '\n'   # replace delimiter
A
B
...
Z
alias nth="echo {A..Z} | tr -d ' ' | cut -c"    # find nth letter of alphabet
```

### find

Prints file paths recursively:
- pipe to `sort` to list alpha
- use `-exec echo rm {}` for safe deletes


`exec` executes a linux command for each file path in the output:
1. Create the `find` command
2. Append `-exec` and then the command to execute. In the command, substitue `{}` for the file path
3. End with quoted or escaped semicolon: `";"` or `\;`


```bash
find /etc -type f -name "*.conf"                        # quote patterns
find /etc -type f -name "*.conf" -exec ls -l {} \;      # run 'ls -l' on each file that matches the name pattern
find /etc -type f -name "*.conf" -ls                    # built-in option

# safe deletes
find ~/toolbox/ -type f -name "*~" -exec echo rm {} \;  # 1. test run
find ~/toolbox/ -type f -name "*~" -delete              # 2. execute with built-in option
```


### yes

Prints the same line repeatedly. Prints `y` by default:
- can pipe to commands that require you answer "yes" to interactive prompts
- pipe to `head` to print string specific number of times, especially when generating test files

```bash
yes 'some string' | head -10
```

## Isolating text

Primarily use these commands:
- `grep`: prints lines that match a string
- `cut`: prints columns from a file
- `head`: prints first lines of file
- `tail`: prints last lines of file

### grep and regex

```bash
grep -l his dir/*           # find all files that contain 'his' in dir
grep '^[A-Z]' file          # all lines that begin with capital
grep -v '^$' file           # nonblank lines (find blank lines and ignore)
grep 'one\|two' file        # either one or two
grep '......' file          # at least 6 chars long
grep '<.*>' file            # less than before greater than, such as HTML code
grep 'w\.' file             # w followed by period

# fixed strings
fgrep w\. file              # ignore regex
grep -F w\. file            # ignore regex

# match against set of strings
cut -d: -f7 /etc/passwd | sort -u | grep -f /etc/shells -F  # match shells in passwd file w shells on computer
cat first | sort -u | grep -f second                        # simpler version
```

With regex:

| Syntax        | Example       | Description                                       |
| :------------ | :------------ | :------------------------------------------------ |
| `.`           | `...`         | Any single character except newline               |
| `X*`          | `_*`          | Zero or more occurrences of `_`                   |
| [characters]  | [aeiouAEIOU]  | Any characters in the set                         |
| [^characters] | [^aeiouAEIOU] | Any non-vowel                                     |
| [c1-c2]       | [0-4]         | Any characters in the given range                 |
| [^c1-c2]      | [0-4]         | Any characters not in the given range             |
| E1\| E2       | [one\|two]    | Either of two expressions (either 'one' or 'two') |
| E1\| E2       | [0-4]         | Any characters not in the given range             |


### tail

Prints last 10 lines of file, can change number of lines like `head`:

```bash
tail -n+3 frost                 # print starting at the third line
tail +3 frost                   # print starting at the third line

head -4 tempfile | tail -2      # print 3rd and 4th lines in file
```
### awk

Can extract columns from a file in ways that `cut` cannot:
- use when columns have arbitrarily sized delimiters
- refer to each field with `$<fieldNum>`
  - multi-digit columns, use parentheses: `$(11)`
- final field: `$NF` (number of fields)
- entire line: `$0`
- use commas to include whitespace in output

```bash
awk '{print $2}' /etc/hosts                         # print second column
echo one two three four | awk '{print $2 $3}'       # twothree
echo one two three four | awk '{print $2, $3}'      # two three

df | awk '{print $4}'
df | awk 'FNR>1 {print $4}'                         # print only line numbers greater than 1
echo first:::::second | awk '-F':* '{print $2}'     # change delimiter. Prints 'second'
```


## Combining text

### cat

`cat` is short for "concatenate", so it combines files from top to bottom:

```bash
cat file1 file2 file3
```

### tac

Combines files bottom-to-top. Its the opposite of `cat`, which is why its `cat` backwards:
- great for processing files that are in chronological order but not reversible with `sort -r`, such as a web server log


```bash
tac /var/log/apache2/access.log.1           # reverse web server log
```

### paste

Combines files side-by-side in columns separated by a tab character:
- `-d` to change delimiter

```bash
paste title-words1 title-words2             # tab-delimited
paste -d, title-words1 title-words2         # comma-delimited
paste -d "\n" title-words1 title-words2     # interleave lines from two or more files
```

### diff

Interleaves text from two files by printing their differences:
- `1c1` means "line 1 in file 1 is different than line 1 in file 2"
- `2a3` means "file 2 has a third line not present in file 1"


```bash
diff file1 file2
1c1                                         # line 1 in file1 is different than line 2 in file2
< Linux is all about efficiency.            # line from file1
---                                         # separator
> MacOS is all about efficiency.            # line from file2
2a3                                         # addition: file2 has third line not present in file1                          
> Have a nice day.

diff file1 file2 | grep '^[<>]'             # grep for just the differences
< Linux is all about efficiency.
> MacOS is all about efficiency.
> Have a nice day.

diff file1 file2 | grep '^[<>]' | cut -c3-  # same grep, remove leading </>
Linux is all about efficiency.
MacOS is all about efficiency.
Have a nice day.
```

## Transforming text