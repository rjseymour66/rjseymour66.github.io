---
title: "Branching"
weight: 80
description: >
  Create, switch, merge, and delete branches. Resolve conflicts.
---

A *branch* in git is a lightweight movable pointer to a commit. Creating a
branch does not copy files. Git simply writes a 41-byte file containing the
SHA-1 hash the branch points to. This makes branching nearly instant,
regardless of repository size.

Branches let you isolate work. You can develop a feature, fix a bug, or run
an experiment on a branch without touching the code on `main`. When the work
is ready, you bring it back by merging.

## How branches work

Every repository starts with one branch. By convention it is named `main`
(or `master` in older repositories). `HEAD` is a special pointer that tracks
which branch you are currently on:

```
A---B---C   (main, HEAD)
```

When you create a branch, git adds a new pointer at the same commit. `HEAD`
stays on the original branch:

```
A---B---C   (main, HEAD)
            ^
            (feature)
```

When you switch to the new branch and commit, the feature pointer advances.
`main` stays where it was:

```
A---B---C   (main)
         \
          D---E   (feature, HEAD)
```

## Managing branches

### git branch

List all local branches. The branch with `*` is the one `HEAD` points to:

```bash
$ git branch
* main
  feature/login
  fix/timeout
```

Pass `-a` to include remote-tracking branches:

```bash
$ git branch -a
* main
  feature/login
  remotes/origin/main
  remotes/origin/feature/login
```

Pass `-v` to see the last commit on each branch:

```bash
$ git branch -v
* main          ca82a6d update readme
  feature/login 2b1c673 add login form
```

### Create a branch

Create a branch at the current commit without switching to it:

```bash
git branch feature/user-auth
```

### git switch and git checkout

`git switch` is the modern command for changing branches. `git checkout` does
the same and is still widely used:

```bash
# switch to an existing branch
git switch feature/user-auth
git checkout feature/user-auth   # equivalent

# create a branch and switch to it in one step
git switch -c feature/user-auth
git checkout -b feature/user-auth   # equivalent
```

### Delete a branch

Delete a branch after its work has been merged. Git refuses to delete a branch
with unmerged commits unless you force it:

```bash
git branch -d feature/user-auth     # safe: refuses if unmerged
git branch -D feature/user-auth     # force: deletes regardless
```

## Merging

Merging integrates the history of one branch into another. Switch to the
branch you want to merge *into* (usually `main`), then run `git merge`:

```bash
git checkout main
git merge feature/user-auth
```

Git uses one of two strategies depending on the state of the branches.

### Fast-forward merge

A fast-forward merge happens when the target branch has not diverged from the
source. `main` simply moves its pointer forward to the tip of the feature
branch. No new commit is created:

```
Before:                        After git merge feature:

A---B---C  (main)              A---B---C---D---E  (main, HEAD)
         \
          D---E  (feature)
```

```bash
$ git merge feature/login
Updating ca82a6d..2b1c673
Fast-forward
 login.go | 42 +++++++++++++++++++++
 1 file changed, 42 insertions(+)
```

### Three-way merge

A three-way merge happens when both branches have diverged from a common
ancestor. Git identifies three commits: the common ancestor, the tip of the
target branch, and the tip of the source branch. It combines them into a new
*merge commit* that has two parents:

```
Before:                        After git merge feature:

A---B---C---F  (main)          A---B---C---F---G  (main, HEAD)
         \                              \       /
          D---E  (feature)               D---E
```

```bash
$ git merge feature/login
Merge made by the 'ort' strategy.
 login.go | 42 +++++++++++++++++++++
 1 file changed, 42 insertions(+)
```

## Resolving conflicts

A conflict occurs when two branches modify the same lines in the same file.
Git cannot decide automatically which version to keep, so it pauses the merge
and asks you to resolve it.

```bash
$ git merge feature/login
Auto-merging auth.go
CONFLICT (content): Merge conflict in auth.go
Automatic merge failed; fix conflicts and then commit the result.
```

Git marks the conflicting region in the file:

```
<<<<<<< HEAD
func authenticate(user string) bool {
    return checkPassword(user)
=======
func authenticate(user, token string) bool {
    return checkToken(user, token)
>>>>>>> feature/login
```

The section between `<<<<<<< HEAD` and `=======` is the version on your
current branch. The section between `=======` and `>>>>>>>` is the incoming
version. Edit the file to produce the result you want, removing all three
marker lines:

```go
func authenticate(user, token string) bool {
    return checkToken(user, token)
}
```

After editing every conflicted file, stage them and complete the merge:

```bash
git add auth.go
git commit
```

To abandon a merge in progress and return to the state before you ran
`git merge`:

```bash
git merge --abort
```

## Keeping a branch up to date

While you work on a feature branch, `main` may receive new commits from other
contributors. Two approaches bring those commits into your branch.

### Merge main into your branch

This preserves the exact history of both branches but adds a merge commit:

```bash
git checkout feature/login
git merge main
```

### Rebase onto main

Rebasing replays your commits on top of the latest `main`, producing a linear
history. See [Rebasing](../rebasing) for a full explanation. For a long-running
feature branch, rebasing is usually the cleaner choice:

```bash
git fetch origin
git rebase origin/main
```

## Viewing branch relationships

`git log --oneline --graph --all --decorate` shows the full branch and merge
history as an ASCII graph:

```bash
$ git log --oneline --graph --all --decorate
*   g7f3a1b (HEAD -> main) Merge branch 'feature/login'
|\
| * 2b1c673 (feature/login) add token validation
| * 4e3af98 add login handler
* | ca82a6d update readme
|/
* a11bef0 initial commit
```
