---
title: "Leveraging text files"
linkTitle: "Text files"
weight: 80
# description:
---

If data is formatted and stored in a text file, you can construct commands to maniuplate it. Here are the general steps:
1. Notice the business problem that involves data
2. Store the data in a text file in a convenient format
3. Write Linux commands to process the files and solve the problem
4. (Optional) Capture the commands in scripts, aliases, or functions so they are simpler to run

## Finding files

The `find` command takes time to search the entire filesystem. You can keep a hidden file in your /home dir that lists the locations of all files in your /home dir tree:

```bash
find $HOME -print > $HOME/.ALLFILES                 # create file with all files

#!/bin/bash                                         # script to search hidden file
# $@ means 'all args provided to the script'
grep "$@" $HOME/.ALLFILES

mv ff.sh /usr/local/bin                             # move script to dir in PATH
```

## Check domain expiration

You can create a script that queries a domain registrar with `whois` and uses `date`, `grep`, and `awk` to extract and format the date:

```bash
#!/bin/bash
expdate=$(date \
            --date $(whois "$1" \
                | grep 'Registry Expiry Date:' \
                | awk '{print $4}') \
                +'%Y-%m-%d')
echo "$expdate $1"
```

You can call this script in another script that loops through a file and checks the expiration date for all domains listed:

```bash
#!/bin/bash
cat domains.txt | while read domain; do
	./check-expiry "$domain"
	sleep 5
done
```