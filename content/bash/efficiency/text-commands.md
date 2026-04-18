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

### tr

Translates characters into other characters:
- takes two sets of characters: translates the first set into the second set
- `-d`: deletes whitespace and tabs

```bash
echo one two      three | tr -d ' '     # onetwothree
```

### rev

Reverses characters on a line. Useful if you need to extract field in a file where each line has a different number of columns. For example, if a file is a list of names, and you need the last name, the number of words on each line varies:

```bash
rev file | cut -d ' ' -f1 | rev
```

### awk

Transforms lines of text from files or stdin into any other text, using a sequence of instructions called an _awk program_:
- `-f` option accepts awk program files
- always enclose the awk program in single or double quotes when provided on the command line
- `-F` changes the input separator

```bash
awk <program> <input-files>
awk -f <program-file1> -f <program-file2> <input-files>
awk 'FNR<=7' nums.file                  # print first 7 lines of file
echo "one two" | awk '{print $2, $1}'   # two one (switch cols)

awk '{print $NF}' file                  # print last word on each line

awk '/^word/{print $4}' file            # if the line begins with 'word', print the fourth field

# if field 3 starts with 201, perform awk program on that line
awk -F'\t' '$3~/^201/{print $4, "(" $3 ").", "\"" $2 "\""}' animals.txt

awk -F'\t' \    # same command with intro and ending
'BEGIN {print "Recent books:"} \
$3~/^201/{print $4, "(" $3 ").", "\"" $2 "\""} \
END {print "For more books, search the web"}' \
animals.txt 

Recent books:
...
For more books, search the web

seq 1 100 | awk '{s+=$1} END {print s}'     # sum numbers 1 - 100
```

#### Arrays and loops

An array in `awk` stores a collection of values. Each value in the array is an element, which consists of the array name and stored value: `counts["f44323234453400"]`. The syntax to access an element is a key. (It seems to act more like a map than an array.)

```bash
md5sum *.jpg | awk '{counts[$1]++}'   # store all md5sums in array, where the element is count[<md5sum>] and key is the number of occurrences
```

The for loop steps through an array and processes each element in sequence. It uses this syntax:

`for (var in array) <action> array[var]`

If this is used in the same command that creates the array, run the for loop after an `END` instruction so it operates on the completed array:

```bash
md5sum *.jpg \
| awk '{counts[$1]++} \
END {for (key in counts) print counts[key] " " key}'
3 5bbf5a52328e7439ae6e719dfe712200
1 c193497a1a06b2c72230e6146ff47080
1 febe6995bad457991331348f7b9c85fa
```

#### awk cheat sheet 

| Command                                                          | Description                                     | Example                                                          |
| ---------------------------------------------------------------- | ----------------------------------------------- | ---------------------------------------------------------------- |
| `awk '{print $1}' file`                                          | Print first column                              | `awk '{print $1}' data.txt`                                      |
| `awk '{print $1, $3}' file`                                      | Print multiple columns                          | `awk '{print $1, $3}' data.txt`                                  |
| `awk 'NR==3' file`                                               | Print third line                                | `awk 'NR==3' data.txt`                                           |
| `awk 'NR>=2 && NR<=4' file`                                      | Print lines 2 to 4                              | `awk 'NR>=2 && NR<=4' data.txt`                                  |
| `awk '/pattern/' file`                                           | Print lines matching a pattern                  | `awk '/error/' log.txt`                                          |
| `awk '!/pattern/' file`                                          | Print lines NOT matching a pattern              | `awk '!/error/' log.txt`                                         |
| `awk '{print NR, $0}' file`                                      | Print line numbers                              | `awk '{print NR, $0}' data.txt`                                  |
| `awk '{print NF}' file`                                          | Print number of fields in each line             | `awk '{print NF}' data.txt`                                      |
| `awk '$2 > 50' file`                                             | Print lines where 2nd column is greater than 50 | `awk '$2 > 50' data.txt`                                         |
| `awk '{sum+=$1} END {print sum}' file`                           | Sum the first column                            | `awk '{sum+=$1} END {print sum}' data.txt`                       |
| `awk '{sum+=$1} END {print sum/NR}' file`                        | Average of first column                         | `awk '{sum+=$1} END {print sum/NR}' data.txt`                    |
| `awk 'BEGIN {FS=","} {print $1}' file`                           | Change field separator to comma                 | `awk 'BEGIN {FS=","} {print $1}' data.csv`                       |
| `awk -F: '{print $1}' /etc/passwd`                               | Use `:` as delimiter                            | `awk -F: '{print $1}' /etc/passwd`                               |
| `awk '{print toupper($1)}' file`                                 | Convert first column to uppercase               | `awk '{print toupper($1)}' data.txt`                             |
| `awk '{print tolower($1)}' file`                                 | Convert first column to lowercase               | `awk '{print tolower($1)}' data.txt`                             |
| `awk '{sub(/old/, "new"); print}' file`                          | Replace first occurrence in each line           | `awk '{sub(/error/, "fixed"); print}' log.txt`                   |
| `awk '{gsub(/old/, "new"); print}' file`                         | Replace all occurrences in each line            | `awk '{gsub(/error/, "fixed"); print}' log.txt`                  |
| `awk 'BEGIN {print "Header"} {print} END {print "Footer"}' file` | Add header and footer                           | `awk 'BEGIN {print "Start"} {print} END {print "End"}' data.txt` |



### sed

Transforms text from files or stdin into other text with a sequence of instructions called a _sed script_ (which is a string):
- most common `sed` script replaces strings with another string
  `s/regexp/replacement/`
  - replaces the first occurence only. Use `g` global option to replace all occurences
- `-e` lets you use a sequence of scripts on a single input file
- `-f` lets you use sed scripts stored in files
- replace the `/` with any other character, such as `_` 
```bash
sed <script> <input-files>
sed -e <script1> -e <script2> -e <script3> <input-files>
sed -f <script-file1> -f <script-file2> -f <script-file3> <input-files>

echo Ricky Henderson | sed 's/Henderson/Ricardo/'   # Ricky Ricardo

sed 's/.* //' celebrities               # replace all chars up to last space with nothing
sed 7q nums.file                        # print first 7 lines of file
sed 's/\.jpg/\.png/'                    # replace .jpg by .png
echo Case Insensitive | sed 's/case/how/i'  # case insensitive
echo one one two | sed 's/one/yes/g'    # global replace
```

#### Deletion

You can delete by line number, or with regex matches:

```bash
seq 1 10 | sed 5d                       # delete 5th line
seq 1 10 | sed '/[2468]/d'              # delete even numbers
```

#### Subexpressions

Lets you manipulate and change strings:
- each subexpression is numbered `\1`, `\2`, and so on

1. Create a regex that matches what you want to change
2. Isolate with parenteses and backslashes (`(\` and `)\`) the portion of the string that you want to manipulate

```bash
image\.jpg\.[1-3]                               # 1. create reges
image\.jpg\.\([1-3]\)                           # 2. isolate the portion you want to manipulate
sed "s/image\.jpg\.\([1-3]\)/image\1.jpg/"      # 3. move the string with \1
```

#### Long regex

If you have a very long regex that is hard to decipher, you can break it up into parts and store each part in a shell variable:
- Make sure you use single quotes for vars so the shell doesn't evalutate special characters

```bash
s/\([0-9]*\)@\([A-Z][A-Z]\)@\([^@*\])@/\1\t\2\t\3\n/g    # starting regex

areacode='\([0-9]*\)'                                    # variables
state='\([A-Z][A-Z]\)'
cities='\([^@*\])'

regexp="$areacode@$state@$cities@"

replacement='/\1\t\2\t\3\n/'                              # replacement string

sed "s/$regexp/$replacement/g"                            # sed command
```

#### Cheatsheet

| Command                        | Description                                 | Example                                        |
| ------------------------------ | ------------------------------------------- | ---------------------------------------------- |
| `s/find/replace/`              | Replace first occurrence in a line          | `sed 's/foo/bar/' file.txt`                    |
| `s/find/replace/g`             | Replace all occurrences in a line           | `sed 's/foo/bar/g' file.txt`                   |
| `s/find/replace/i`             | Case-insensitive replacement                | `sed 's/foo/bar/i' file.txt`                   |
| `s/find/replace/gI`            | Global case-insensitive replacement         | `sed 's/foo/bar/gI' file.txt`                  |
| `s/find/replace/gw output.txt` | Save changes to another file                | `sed 's/foo/bar/gw output.txt' file.txt`       |
| `2s/find/replace/`             | Replace only in the second line             | `sed '2s/foo/bar/' file.txt`                   |
| `s/old/new/3`                  | Replace only the third occurrence in a line | `sed 's/foo/bar/3' file.txt`                   |
| `-i`                           | Modify file in place                        | `sed -i 's/foo/bar/g' file.txt`                |
| `^`                            | Match beginning of line                     | `sed 's/^Hello/Hi/' file.txt`                  |
| `$`                            | Match end of line                           | `sed 's/world!$/earth!/' file.txt`             |
| `d`                            | Delete line(s)                              | `sed '/delete this line/d' file.txt`           |
| `nd`                           | Delete specific line number                 | `sed '3d' file.txt` (deletes line 3)           |
| `-n 'Np'`                      | Print only specific lines                   | `sed -n '2p' file.txt` (prints line 2)         |
| `-n '/pattern/p'`              | Print lines matching a pattern              | `sed -n '/error/p' file.txt`                   |
| `/pattern/d`                   | Delete lines matching pattern               | `sed '/error/d' file.txt`                      |
| `y/abc/xyz/`                   | Translate characters                        | `sed 'y/abc/xyz/' file.txt`                    |
| `N`                            | Merge next line with current                | `sed 'N' file.txt`                             |
| `:label` & `b label`           | Looping with labels                         | `sed ':a; /pattern/{s/foo/bar/; ba}' file.txt` |

