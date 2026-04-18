---
title: "Basics"
weight: 30
description: >
  Initialize and clone repositories, stage changes, and commit snapshots.
---

These commands cover the day-to-day cycle of working with git: starting or
cloning a repository, tracking files, reviewing what has changed, and saving
snapshots. Every git workflow runs through this loop repeatedly.

## Initialize or clone a repository

### git init

`git init` creates a new `.git` directory in the current folder and begins
tracking it as a git repository:

```bash
git init
```

### git clone

`git clone` copies an existing repository to your machine. It creates a new
directory, initializes `.git` inside it, pulls down all project data, and
checks out a working copy of the latest version. All files start as tracked
and unmodified:

```bash
git clone https://path/to/repo
```

## Tracking and staging files

Files in the working tree are either *tracked* or *untracked*:

*tracked*
: The file appeared in the last snapshot, or you have already staged it. Git
knows about this file.

*untracked*
: Git sees the file but it was not in the last snapshot and is not in the
staging area.

### git status

`git status` reports the state of every file in the working tree and staging
area:

```bash
$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        README
        hello.go

nothing added to commit but untracked files present (use "git add" to track)
```

For a compact summary, pass `-s`. The output has two columns: the left column
shows the staging area status and the right column shows the working tree
status:

```bash
$ git status -s
 M README           # modified in working tree, not yet staged
MM Rakefile         # staged and then modified again
A  lib/git.rb       # newly added to staging
M  lib/simplegit.rb # modified and staged
?? LICENSE.txt      # untracked
```

### git add

`git add` begins tracking a new file or stages a modified one. Pass a file
name, a directory (git adds all files recursively), or a pattern:

```bash
git add README hello.go
```

After adding, the files appear under "Changes to be committed":

```bash
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   README
        new file:   hello.go
```

Three additional forms of `git add` cover common situations:

```bash
git add .        # stage all changes in the current directory and below
git add -u       # stage modifications and deletions, but not new files
git add -p       # interactively choose which changes to stage
```

`git add -p` (*patch mode*) is one of the most useful habits to build. It
walks you through each changed hunk in a file and asks whether to stage it.
This lets you commit one logical change at a time even when your working tree
contains several unrelated edits:

```bash
$ git add -p
diff --git a/main.go b/main.go
@@ -10,6 +10,10 @@ func main() {
+   // TODO: remove debug logging
+   log.Println("starting server")
Stage this hunk [y,n,q,a,d,s,?]?
```

The most common responses are:

| Key | Action |
|:---|:---|
| `y` | Stage this hunk |
| `n` | Skip this hunk |
| `s` | Split the hunk into smaller pieces |
| `q` | Quit without staging further hunks |
| `?` | Show all options |

## Ignoring files

### .gitignore

A `.gitignore` file tells git which files and directories to leave untracked.
Place it in the root of your repository. A collection of language-specific
templates is available at the
[GitHub gitignore repository](https://github.com/github/gitignore).

The rules below show the most common patterns:

```bash
# ignore all .a files
*.a

# track lib.a even though .a files are ignored above
!lib.a

# ignore TODO only in the root directory, not in subdirectories
/TODO

# ignore all files in any directory named build
build/

# ignore doc/notes.txt but not doc/server/arch.txt
doc/*.txt

# ignore all .pdf files under doc/ at any depth
doc/**/*.pdf
```

## Reviewing changes

### git diff

`git diff` shows the exact lines that changed. It uses a pager for long
output. Three common forms cover the most frequent use cases:

| Command | What it shows |
|:---|:---|
| `git diff` | Changes in the working tree that are not yet staged |
| `git diff --staged` | Changes staged for the next commit |
| `git diff --cached` | Same as `git diff --staged` |

The following example shows a working-tree diff after modifying a staged file:

```bash
$ git diff
diff --git a/README b/README
index e8798b5..e471777 100644
--- a/README
+++ b/README
@@ -165,4 +165,41 @@ Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   README
        new file:   hello.go
...
```

To preview what goes into the next commit, run the staged diff:

```bash
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

## Committing

### git commit

`git commit` records a snapshot of everything in the staging area. Running it
without flags opens your configured editor so you can write a message:

```bash
git commit
```

Pass `-v` to include the diff in the editor window as a reference:

```bash
git commit -v
```

Pass `-m` to write the message inline without opening an editor:

```bash
$ git commit -m 'first commit'
[master (root-commit) 0016653] first commit
 2 files changed, 261 insertions(+)
 create mode 100644 README
 create mode 100644 hello.go
```

The output shows the branch name, the SHA-1 checksum, the commit message, and
a summary of what changed.

### git commit -a

The `-a` flag automatically stages every tracked file before committing,
skipping the `git add` step. It does not stage new untracked files:

```bash
$ git commit -am 'second commit'
[master 454f4d3] second commit
 1 file changed, 27 insertions(+)
```

### git show

`git show` displays the metadata and diff for a single commit. Without
arguments it shows the most recent commit:

```bash
$ git show
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gmail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number

diff --git a/Rakefile b/Rakefile
...
```

Pass a commit hash, branch name, or tag to inspect a specific point in
history:

```bash
git show 085bb3b          # by abbreviated hash
git show main             # tip of the main branch
git show v1.4             # a tagged release
```

## Removing and renaming files

### git rm

`git rm` removes a file from both the working directory and the staging area.
The removal is staged and takes effect on the next commit.

The following example walks through the full removal sequence:

```bash
# delete the file from the filesystem
$ rm PROJECTS.md

# git sees it as deleted but not yet staged
$ git status
On branch master
Changes not staged for commit:
        deleted:    PROJECTS.md

# stage the removal
$ git rm PROJECTS.md
rm 'PROJECTS.md'

# commit to stop tracking the file
$ git commit -m 'deleted PROJECTS.md'
[master 3108c75] deleted PROJECTS.md
 1 file changed, 0 insertions(+), 0 deletions(-)
 delete mode 100644 PROJECTS.md
```

To remove a file from the staging area while keeping it on disk, pass
`--cached`. This is useful when you accidentally staged a file you want to
keep locally but not commit:

```bash
$ git rm --cached PROJECTS.md
rm 'PROJECTS.md'

$ git status
On branch master
Untracked files:
        PROJECTS.md
```

To remove all `.log` files from a directory, pass a glob pattern:

```bash
git rm log/\*.log
```

### git mv

`git mv` renames a file and stages the rename in one step. Without it, git
would see the original file as deleted and the new file as untracked:

```bash
$ git mv move.txt move.md

$ git status
On branch master
Changes to be committed:
        renamed:    move.txt -> move.md

$ git commit -m 'rename'
[master 98bf571] rename
 1 file changed, 0 insertions(+), 0 deletions(-)
 rename move.txt => move.md (100%)
```

## Saving work in progress

### git stash

`git stash` saves your uncommitted changes to a temporary stack and restores a
clean working tree. It is useful when you need to switch branches or pull
updates without committing unfinished work.

The typical stash workflow has three steps:

```bash
# save current changes to the stash
git stash

# your working tree is now clean — switch branches, pull, or investigate
git checkout main
git pull

# return to your feature branch and restore your changes
git checkout feature-branch
git stash pop
```

`git stash pop` applies the most recent stash and removes it from the stack.
To apply a stash without removing it, run `git stash apply` instead.

List all stashes with `git stash list`. Each entry is numbered from zero:

```bash
$ git stash list
stash@{0}: WIP on feature: 2b1c673 add user model
stash@{1}: WIP on feature: a11bef0 first commit
```

To apply a specific stash, pass its identifier:

```bash
git stash apply stash@{1}
```

To discard a stash you no longer need:

```bash
git stash drop stash@{0}
```

By default, `git stash` saves only tracked files. Pass `-u` to include
untracked files as well:

```bash
git stash -u
```
