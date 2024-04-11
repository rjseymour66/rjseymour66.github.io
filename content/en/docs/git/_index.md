---
title: "git"
linkTitle: "git"
# weight: 40
description: >
  Notes about basic git.
---

Git is a distributed version control system (DVCS) that stores a complete copy of a repository on your local machine. To track files, it stores snapshots of each file in a compressed database. Git tracks each of these files using a SHA-1 hash checksum--each time you commit a new or modified file to the database, git creates a checksum. If you do not change a file during a version, git copies the unchanged file into the new version.

## States and directories

Git uses 3 stages: 
- committed. Stored in your local db
- modified. Changed, but not committed to the local db
- staged. Marked a modified file to go into the next commit snapshot

Git uses 3 main sections of the git project:
- .git directory, the object and metadata database of the project. This is the cloned repo
- working directory, a checkout of one version of the .git directory. git checks files out of the .git database and places them in a directory in your filesystem so you can work on them
- staging area (also called the "index"), stores info about what is going into the next commit

## Configuration

git configuration values are stored in the `.gitconfig` file, located in one of the following places:
- `/etc/gitconfig`: system-wide git configurations
- `~/.gitconfig`: user git configurations. Set these values with the `--global` option
- `project/.git/config`: local project repository configuration

Return git configuration information with the following command:

```
$ git config --list
```

## Getting started

- Distributed Version Control System (DVCS): clients fully mirror the repo, including its full history.
  - Any client can restore a corrupted server

### git 

- Thinks about data as a "stream of snapshots" (snapshots == commit) of a mini filesystem
- During each commit, git takes a snapshot of what the files look like and stores a reference to the snapshot 
  - If file hasn't changed, git doesn't store the file again. It links to the previous identical file that it already stored.
- Everything is local bc you have the entire project on your workstation
- Integrity - everything is stored as a checksum, so it always knows when information changes
  - SHA-1 hash uses 40 char string
- Adds data - generally only adds data, hard to do anything that is not undoable

### Three states 

modified
: Changed file but not committed to the db yet

staged
: marked a modified file in its current version to go into the next commit snapshot

committed
: data is stored in local database

### Three sections of git project 

working tree
: single checkout of one version of the project. Its files are pulled out of the compressed db in the git directory and placed on disk for you to work with.

staging area (index)
: A file in the git directory that stores info about what goes into the next commit. 

git directory (.git)
: Where git stores the metadata and object db for the project. This is what's copied when you clone a repo.


General workflow:
1. Modify files in working tree 
2. Add changes to staging area 
3. Commit, which stores a snapshot of staging area and stores in git directory


## Setup 

### Install

```bash
sudo apt install git-all
```

### git config

Use `git config` to set config variables for your git install. Stored here in reverse order of precedence:
- `/etc/gitconfig`: Applys to every user on system.
  - Add `--system` option to `git config`
- `~/.gitconfig` or `~/.config/git/config`: User options. 
  - Add `--global` option to apply changes to all repos for your user account
- `/<git-project>/.git/config`: Specific to the repo

```bash
# get help
$ git help <verb>
$ git <verb> --help
$ man git-<verb>

# open git config man page in browser
$ git help config

# view all settings
$ git config --list
diff.astextplain.textconv=astextplain
filter.lfs.clean=git-lfs clean -- %f
filter.lfs.smudge=git-lfs smudge -- %f
...

# list all settings and where they are stored
$ git config --list --show-origin
file:C:/Program Files/Git/etc/gitconfig diff.astextplain.textconv=astextplain
file:C:/Program Files/Git/etc/gitconfig filter.lfs.clean=git-lfs clean -- %f

# set username and address
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com

# set username and address for specific project
$ git config user.name "John Doe"
$ git config user.email johndoe@example.com

# change default editor
$ git config --global core.editor emacs

# set default branch name
$ git config --global init.defaultBranch main

# check specific value
$ git config user.name
<username>
```

## git basics 

### Getting git repo

```bash
# start repo, creates .git directory
$ git init

# clone repo
# creates dir, inits /.git, pulls down all data, checks out working copy of latest version
# all files tracked and unmodified
$ git clone https://path/to/repo
```

### Recording changes 

Files have two states:

tracked
: In the last snapshot, or newly staged files. Files that git knows about.

untracked
: git sees a file not in last snapshot or in staging area.

```
Untracked       Unmodified              Modified             Staged 
    |                |                      |                   |
    | Add the file -------------------------------------------> |
    |                | Edit file -->        |                   |
    |                |                      |                   |
    |                |                      | Stage file -----> |
    | <--Remove file |                      |                   |
    |                | <-------------------------------- Commit |
```

Clean working directory 
: No tracked files are modified 

```bash 
# get file status
$ git status
On branch master

No commits yet

Untracked files:    # lists untracked files
  (use "git add <file>..." to include in what will be committed)
        README
        hello.go

nothing added to commit but untracked files present (use "git add" to track)

# add this content (files, dirs) to start tracking
# if directory, adds all files in dir recursively
$ git add README hello.go

# added files are now tracked and staged to be committed
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   README
        new file:   hello.go

# short, less verbose status
# contains 2 columns: staging area status  working tree status
$ git status -s
 M README               # modified, but not staged
MM Rakefile             # modified, staged, modified again
A  lib/git.rb           # added to staging
M  lib/simplegit.rb     # modified and staged
?? LICENSE.txt          # untracked
```

### .gitignore

[GitHub gitignore files](https://github.com/github/gitignore)

Basic rules:

```bash 
# ignore all .a files
*.a

# but do track lib.a, even though you're ignoring .a files above
!lib.a

# only ignore the TODO file in the current directory, not subdir/TODO
/TODO

# ignore all files in any directory named build
build/

# ignore doc/notes.txt, but not doc/server/arch.txt
doc/*.txt

# ignore all .pdf files in the doc/ directory and any of its subdirectories
doc/**/*.pdf
```

### git diff 

Shows exactly what you changed, uses pager to show long files:

- `git diff`: Changes that are unstaged, or changes in the working directory.
- `git diff --staged`: Changes on staged files
- `git diff --cached`: Same as `git diff --staged`

```bash
# check status
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   README
        new file:   hello.go

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README

# compare working dir with staging area
$ git diff
warning: LF will be replaced by CRLF in README.
The file will have its original line endings in your working directory
diff --git a/README b/README
index e8798b5..e471777 100644
--- a/README
+++ b/README
@@ -165,4 +165,41 @@ Changes to be committed:
   (use "git rm --cached <file>..." to unstage)
         new file:   README
         new file:   hello.go
...

# see what is going in next commit
$ git diff --staged
diff --git a/README b/README
new file mode 100644
index 0000000..e8798b5
--- /dev/null
+++ b/README
@@ -0,0 +1,168 @@
+## Getting started
...
```

### git commit

Commits files in the staging area:

```bash
# open default editor to enter commit msg
$ git commit

# show diff in default editor
$ git commit -v

# inline commit message
$ git commit -m 'first commit'
[master (root-commit) 0016653] first commit     # branch committed to, SHA-1 checksum
 2 files changed, 261 insertions(+)             # files changed, stats about changes
 create mode 100644 README
 create mode 100644 hello.go
```

### git commit -a 

This command skips the staging area (skips `git add <file | dir>`). The `-a` option automatically stages every file that is already tracked before the commit:

```bash 
# view unstaged files
$ gs
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README

no changes added to commit (use "git add" and/or "git commit -a")

# skip staging area and provide commit message
$ git commit -am 'second commit'
[master 454f4d3] second commit
 1 file changed, 27 insertions(+)
```

### git rm

Removes file from staging and working directory:

```bash
##########################
# delete a tracked file
$ rm PROJECTS.md

# verify its deleted
$ git status
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        deleted:    PROJECTS.md

no changes added to commit (use "git add" and/or "git commit -a")

# remove file from working dir
$ git rm PROJECTS.md
rm 'PROJECTS.md'

# file is ready to delete
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    PROJECTS.md

# commit changes, file is no longer tracked
$ git commit -m 'deleted PROJECTS.md'
[master 3108c75] deleted PROJECTS.md
 1 file changed, 0 insertions(+), 0 deletions(-)
 delete mode 100644 PROJECTS.md

##########################
# rm from staging area 
$ git rm --cached PROJECTS.md
rm 'PROJECTS.md'

# file is no longer in staging area
$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        PROJECTS.md

nothing added to commit but untracked files present (use "git add" to track)


# rm all files in /log with .log extension
$ git rm log/\*.log
```

### git mv

Rename a file in git. Otherwise, it deletes the file w/the old name, then tracks the new file name:

```bash
# change from *.txt -> *.md
$ git mv move.txt move.md

# verify change
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        renamed:    move.txt -> move.md

# commit
$ git commit -m 'rename'
[master 98bf571] rename;
 1 file changed, 0 insertions(+), 0 deletions(-)
 rename move.txt => move.md (100%)

# filename is changed
$ ls
hello.go  move.md  PROJECTS.md  README.md
```

## Commit history