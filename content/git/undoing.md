---
title: "Undoing changes"
weight: 50
description: >
  Amend commits, unstage files, and discard working tree changes.
---

Git gives you several ways to undo work at different stages of the workflow.
The right tool depends on whether the change is in your working tree, in the
staging area, or already committed. The sections below cover the most common
recovery operations.

## Amending the last commit

### git commit --amend

`--amend` replaces the most recent commit with a new one. Git creates a fresh
commit object (with a new SHA-1 hash) that incorporates any staged changes
alongside the previous content. It is best reserved for local commits that
have not been pushed to a shared branch, because rewriting a published commit
forces anyone who has based work on it to reconcile their history.

The following example corrects a commit message after the fact:

```bash
# original commit
$ git log --oneline
07c2519 (HEAD -> master) amend setup

# amend with a new message
$ git commit --amend -m 'updated amend commit'
[master 9a22606] updated amend commit
 2 files changed, 132 insertions(+), 2 deletions(-)

# the old commit is replaced
$ git log --oneline
9a22606 (HEAD -> master) updated amend commit
```

To add a forgotten file to the previous commit without changing the message,
stage the file first and then amend:

```bash
# you forgot to include forgot.file in the last commit
$ git add forgot.file
$ git commit --amend --no-edit
```

`--no-edit` reuses the existing commit message so the editor does not open.

## Unstaging a file

### git restore --staged

`git restore --staged <file>` removes a file from the staging area but leaves
your working tree changes intact. Git prints this command as a hint in
`git status` output whenever a file is staged:

```bash
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   README.md
        new file:   unstage.md

# unstage README.md but keep the edits on disk
$ git restore --staged README.md
```

## Discarding working tree changes

### git restore

`git restore <file>` discards uncommitted changes in the working tree and
reverts the file to its state in the last snapshot. This operation is
irreversible: any unsaved edits are permanently lost.

```bash
# discard all changes to README.md since the last commit
$ git restore README.md
```

To discard all working tree changes at once, pass `.`:

```bash
git restore .
```

## Choosing the right undo tool

The table below maps each situation to the correct command:

| Situation | Command |
|:---|:---|
| Wrong commit message or forgot to stage a file | `git commit --amend` |
| File staged but should not be in the next commit | `git restore --staged <file>` |
| Working tree edits you want to throw away | `git restore <file>` |
| Changes you want to save but not commit yet | `git stash` (see [Basics](../basics)) |
