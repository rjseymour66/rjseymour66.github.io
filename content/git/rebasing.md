---
title: "Rebasing"
weight: 90
description: >
  Replay commits onto a new base to produce a clean, linear history.
---

Rebasing is one of the two ways to integrate changes from one branch into
another. Merging preserves the full history of both branches, including when
they diverged and when they came back together. Rebasing replays your commits
on top of another branch, as if you had started your work from that point.
The result is a linear history with no merge commits.

Choose rebasing when you want a clean, readable history on a feature branch
before merging it into `main`. Choose merging when you want to preserve the
exact record of how work developed in parallel.

## How rebase works

Suppose `main` and `feature` have diverged. Both have commits the other does
not:

```
Before rebase:

main:    A---B---C
                  \
feature:           D---E   (HEAD)
```

`git rebase main` replays commits `D` and `E` onto `C`. Git finds the common
ancestor (`B`), takes each commit on `feature` that is not on `main`, and
applies them one at a time on top of the tip of `main`. Each replayed commit
gets a new SHA-1 hash because its parent has changed:

```
After git rebase main (run from feature branch):

main:    A---B---C
                  \
feature:           D'--E'  (HEAD)
```

`D'` and `E'` contain the same changes as `D` and `E`, but they are new
commit objects with different hashes. The original `D` and `E` are
abandoned.

After the rebase, the merge from `main` into `feature` is a simple
fast-forward — no merge commit required:

```
After git checkout main && git merge feature:

main:    A---B---C---D'--E'  (HEAD)
```

## Basic rebase

Switch to the branch you want to rebase, then pass the branch you want to
rebase onto:

```bash
git checkout feature
git rebase main
```

Or in one step from any branch:

```bash
git rebase main feature
```

If the rebase succeeds, update `main` with a fast-forward merge:

```bash
git checkout main
git merge feature
```

## Handling conflicts during a rebase

When a replayed commit conflicts with the target branch, git pauses and asks
you to resolve it — the same process as a merge conflict:

```bash
$ git rebase main
Auto-merging auth.go
CONFLICT (content): Merge conflict in auth.go
error: could not apply 2b1c673... add token validation
hint: Resolve all conflicts manually, then run "git rebase --continue"
```

1. Open the conflicted file and resolve the markers.
2. Stage the resolved file:

```bash
git add auth.go
```

3. Continue replaying the remaining commits:

```bash
git rebase --continue
```

To skip the current commit entirely (use with care — you lose that change):

```bash
git rebase --skip
```

To abandon the rebase and return to the state before you started:

```bash
git rebase --abort
```

## The golden rule of rebasing

**Never rebase commits that have been pushed to a shared branch.**

When you rebase, git replaces existing commits with new ones. If a teammate
has based work on the original commits, their branch now diverges from yours
in confusing ways. They must perform a complex merge to reconcile the two
histories, and older versions of commits may appear duplicated in the log.

The rule applies to any branch others are working from: `main`, `develop`,
shared release branches, or any branch you pushed to a remote and told a
colleague about. Rebasing is safe on branches that exist only on your local
machine or on a remote branch that only you use.

```
                       !! DO NOT rebase C and D after pushing !!

Remote main:  A---B---C---D
                            \
Colleague:                   E---F   (their work based on D)

You rebase:   A---B---C'--D'        (new hashes -- D is gone from your perspective)

Colleague now has:
              A---B---C---D---E---F  (their view, still based on old D)
                        \
                         C'--D'     (your rebased commits)

Result: duplicate commits, confusing history, broken collaboration
```

If you have already pushed and still need to rebase, coordinate with your
team, force-push with `--force-with-lease` (safer than `--force`), and have
everyone re-clone or reset their local branches.

## Interactive rebase

Interactive rebase (`-i`) opens an editor listing the commits to be replayed.
You can reorder, edit, squash, or drop individual commits before they land on
the target branch. It is the standard tool for cleaning up a messy feature
branch before opening a pull request.

```bash
# interactively rebase the last 4 commits
git rebase -i HEAD~4
```

Git opens an editor with a list of commits, oldest first:

```
pick 4e3af98 add login handler
pick 2b1c673 add token validation
pick a7f92c1 fix typo in comment
pick 3d8b041 WIP: half-finished session logic
```

Change the verb on each line to control what happens to that commit:

| Command | Action |
|:---|:---|
| `pick` | Keep the commit as-is |
| `reword` | Keep the commit but edit its message |
| `edit` | Pause after applying so you can amend the commit |
| `squash` | Combine with the previous commit, merging messages |
| `fixup` | Combine with the previous commit, discarding this message |
| `drop` | Remove the commit entirely |

### Squashing work-in-progress commits

A common use is collapsing several "WIP" or "fix typo" commits into one clean
commit before merging:

```
Before:                            After squash:

pick 4e3af98 add login handler     pick 4e3af98 add login handler
squash 2b1c673 add token valid.    squash 2b1c673 add token validation
fixup a7f92c1 fix typo             (dropped into previous)
drop 3d8b041 WIP: half-finished    (removed entirely)
```

The result is one commit with the combined diff of the first two, and a
message you write in the editor that opens after saving the rebase plan.

### Rewriting a commit message

Change `pick` to `reword` for any commit whose message you want to fix. Git
pauses at that commit and opens the editor:

```
reword 4e3af98 add login handler
pick   2b1c673 add token validation
```

### Splitting a commit

Change `pick` to `edit` to pause at a commit. Then reset it to unstaged,
stage the pieces separately, and commit each one:

```bash
git reset HEAD~           # unstage the commit, keep changes in working tree
git add auth.go
git commit -m 'add login handler'
git add token.go
git commit -m 'add token validation'
git rebase --continue
```

## Real-world scenarios

### Clean up a feature branch before a pull request

You have been working on a feature for two days and your branch has 12 commits
including several "WIP" and "fix review comment" entries. Before opening the
pull request, squash them into a few logical commits:

```bash
git rebase -i origin/main
```

In the editor, squash related commits and drop the noise. The reviewer sees a
clean history that tells the story of the change, not the story of how you
wrote it.

### Keep a long-running branch current

Your feature branch has been open for a week while `main` has received 20
new commits. Rebase to incorporate the latest changes without a merge commit:

```bash
git fetch origin
git rebase origin/main
```

If conflicts arise, resolve them commit by commit as git pauses. When the
rebase completes, your commits sit cleanly on top of the latest `main`.

### Fix a commit deep in history

You realize a commit from five commits ago has a bug. Use interactive rebase
to pause at that commit and amend it:

```bash
git rebase -i HEAD~5
# change 'pick' to 'edit' on the target commit
```

Git pauses after applying the commit. Fix the bug, stage the change, and
amend:

```bash
git add .
git commit --amend --no-edit
git rebase --continue
```

## Rebase vs. merge: when to use each

| Situation | Use |
|:---|:---|
| Keeping a feature branch current with `main` | Rebase |
| Integrating a finished feature into `main` | Either (team preference) |
| Preserving the exact timeline of parallel work | Merge |
| Cleaning up commits before a pull request | Interactive rebase |
| Commits already pushed to a shared branch | Merge (never rebase) |
