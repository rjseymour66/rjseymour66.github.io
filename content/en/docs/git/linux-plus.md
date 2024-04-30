---
title: "Linux+ git"
weight: 20
---

Working directory
: Where all program files are created, modified, and reviewed.

Staging area
: Also called the _index_, its located on the same system and the working directory. Register files to the staging area with `git add`.

  There is a hidden subdirectory, `.git` that is created with `git init`. When you add a file to the staging area, git creates or updates the file information in `.git/index`. This includes checksums, timestamps, and filenames.

  git compresses the file and stores the compressed file as an object (_blob_) in `.git/objects/`. If it is a modified file that is already tracked, git stores the modifications as a new object so that there are versions of each iteration.

Local repo
: Project tree and commit information (`git commit`) is stored in `.git/objects/`. A commit is also called a _snapshot_.

Remote repo
: Same as local repo, but in a different location (cloud, on-site server, etc).


## git setup

```bash
# view config with CLI
git config --list
user.name=linuxuser
user.email=linuxuser@example.com
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true

# view global config
cat /home/linuxuser/.gitconfig 
[user]
	name = linuxuser
	email = linuxuser@example.com

# view local config
cat .git/config 
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
```

### .git

```bash
# .git contents
ls .git
branches        config       HEAD   index  logs     packed-refs
COMMIT_EDITMSG  description  hooks  info   objects  refs

# view last commit msg 
cat .git/COMMIT_EDITMSG 
linux file in git

```