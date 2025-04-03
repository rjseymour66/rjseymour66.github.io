---
title: "Brash one-liners"
linkTitle: "Brash"
weight: 80
# description:
---

A brash one-liner is a complicated bash command that solves a business problem. Use this formula to build them:
1. Create a command that solves a piece of the puzzle. The first command that you build should output the initial data that you want the complete command to work on.
2. Run the command and check the output
3. Recall the command from history and revise as needed
4. Repeat steps 2 and 3 until it works




Here is a complicated command that renames a set of .jpg files:

```bash
paste <(echo {1..10}.jpg | sed 's/ /\n/g') \
      <(echo {0..9}.jpg | sed 's/ /\n/g') \
| sed 's/^/mv /' \
| bash
```

- `echo {1..10}.jpg` generates a list of .jpg files
- `sed 's/ /\n/g'` replaces spaces with newline characters
- `paste` prints both commands side-by-side, and the process substitution lets `paste` read the commands as files
- `sed 's/^/mv /'` prepends `mv` to the beginning of each line to create a mv command
- `bash` executes the constructed `mv` commands

## Testing tools

- Command history
- `echo` to test expressions
- `echo` or `ls` to test destructive commands like `rm`, `mv`, or `cp`
  - `echo` prints the command that you would run
  - `ls`  to replace `rm` to list the files that you would remove
- `tee` to view intermediate results. Add `tee outputfile` as a step in a pipeline to view what the command produces at that point in the pipeline

## Insert a filename into a sequence

1. Generate the command arguments as lists on stdout
2. Print the lists side by side with `paste` and command substitution
3. Prepend a command name with `sed` by replace the beginning-of-line character with a program name and space, like "mv "
4. Pipe the results to bash

```bash
paste <(seq -w 10 -1 3 | sed 's/\(.*\)/ch\1.ascii.doc/') \      # 1 and 2
      <(seq -w 11 -1 4 | sed 's/\(.*\)/ch\1.ascii.doc/') \      # 1
      | sed 's/^/mv /'                                          # 3
      | bash                                                    # 4
```

## Checking matched pairs of files


These solutions are equivalent:

```bash
diff <(ls *.jpg | sed 's/\.[^.]*$//') <(ls *.txt | sed 's/\.[^.]*$//') \
| grep '^[<>]' \
| awk '/^</{print $2 ".jpg"} /^>/{print $2 ".txt"}'
```

```bash
ls -1 $(ls *.{jpg,txt} \
| sed 's/\.[^.]*$//' \
| uniq -c \
| awk '/^ *1 /{print $2 "*"}')
```

## Generating CDPATH from $HOME

You can add this to your `.bashrc` file to create the `CDPATH` variable:

```bash
echo 'CDPATH=$HOME' \                                               # start the assignment string
     $(cd && ls -d */ | sed -e 's@^@$HOME/@g' -e 's@/$@@') ..\      # (1) get all dirs in $HOME, (2) rm final '/'
     | tr ' ' ':'                                                   # sub spaces for colons
```

## Generating test files

Generate 1,000 text files with random text. It selects words randomly from a large text file and creates 1,000 smaller files with random contents and lengths:

1. Randomly shuffly the dictionary file
2. Select a random number of lines from the dictionary file
3. Create an output file to hold the results
4. Run the solution 1,000 times

```bash
yes 'shuf -n $RANDOM -o $(pwgen -N1 10).txt /usr/share/dict/words' \    # yes prints the command infinitely
| head -n 1000 \                                                        # head limits the output 
| bash                                                                  # bash executes the output
```

## Generating empty files


The simplest way to generate 1,000 empty files is with brace expansion:

```bash
touch file{0..9}{0..9}.txt file{10..1000}.txt
```

For more interesting filenames with the dictionary:

```bash
grep '^[a-z]*$' /usr/share/dict/words \     # grep for lowercase words only
| shuf \                                    # shuffle the output
| head -n1000 \                             # output only 1000 words
| xargs -I {} touch {}.txt                  # run xargs on all output with the touch command and the -I option
```