---
title: "Common workflows"
weight: 15
description: >
  End-to-end workflows for solo work, team collaboration, and server
  administration, from beginner to advanced.
---

The other pages in this section document individual commands. This page
combines those commands into complete workflows: the sequence of steps you
actually run to get something done. Each workflow states when you would
reach for it and why, then walks through the commands in order.

## Beginner: working alone

These are the workflows you repeat every day on a project with no
collaborators. They build directly on the commands in
[Basics](../basics) and [Commit history](../history).

### The daily solo loop

Use this loop any time you are the only person touching a repository. It
keeps your history readable even without the pressure of a team review
process.

```bash
git status                # see what changed
git add -p                # stage one logical change at a time
git commit -m 'add input validation to signup form'
git log --oneline         # confirm the commit landed as expected
```

### Clone a repository

Reach for this workflow the first time you touch an existing project,
whether it is a public repository or one a colleague handed you.

```bash
git clone https://github.com/username/project.git
cd project

# make a change, then review it before staging
git diff
git add README.md
git commit -m 'fix typo in installation instructions'
git push
```

If the clone has no remote write access yet, see
[Create and connect a remote](#create-and-connect-a-remote)
below for the server-side half of this workflow.

### Undo a recent mistake

Use this workflow immediately after you notice a problem with your most
recent commit, before you do anything else. Catching the mistake early
keeps the fix simple.

```bash
# wrong commit message, nothing else needs to change
git commit --amend -m 'add rate limiting to login endpoint'

# committed to main instead of a feature branch
git branch feature/rate-limiting   # create a branch at the current commit
git reset --hard HEAD~1            # move main back one commit
git checkout feature/rate-limiting # your commit now lives here instead
```

See [Undoing changes](../undoing) for the full set of recovery commands,
including `git restore` for uncommitted edits.

## Beginner: server side

These workflows set up the remote half of a repository. Run them once per
project, typically right after you decide a repository needs to be shared.

### Create and connect a remote

Do this the moment a solo project needs a backup off your machine, or
before inviting anyone else to contribute.

```bash
# on GitHub or GitLab: create an empty repository through the web UI first,
# then connect your local repository to it
git remote add origin git@github.com:username/project.git
git push -u origin main
```

`-u` records the upstream branch, so later pushes and pulls need no
arguments. See [Remotes](../remotes) for `fetch`, `pull`, and the full
`remote` command set.

### Enable branch protection

Apply this as soon as a second contributor joins the project, before their
first pull request. It stops accidental direct pushes to `main` while the
team is still small.

1. Open the repository's **Settings** on GitHub or GitLab.
2. Select **Branches** and add a protection rule for `main`.
3. Require a pull request before merging, and block force-pushes.

At this stage, a single required reviewer is usually enough. The
[Intermediate: server side](#intermediate-server-side)
section below covers required status checks and multiple reviewers.

## Intermediate: working with others

These workflows appear once you are merging your work against other
people's changes instead of working in isolation. They draw on
[Branching](../branching) and [Rebasing](../rebasing).

### Rebase a feature branch before merging

Use this workflow for any change that takes more than a few minutes,
once you are collaborating with at least one other contributor on the
same branch. It keeps your branch's history linear and makes the
eventual review easier to read.

```bash
git checkout main
git pull
git checkout -b feature/password-reset

# work and commit as usual, then catch up with main before opening a pull request
git fetch origin
git rebase origin/main

git push -u origin feature/password-reset
```

Open the pull request after the push. See
[Keeping a branch up to date](../branching#keeping-a-branch-up-to-date) for
the merge-versus-rebase tradeoff this workflow is built on.

### Resolve a rebase conflict

This happens whenever a commit you are replaying changes the same lines
that `main` already changed. Resolve it the same way you would resolve a
merge conflict, one commit at a time.

```bash
$ git rebase origin/main
CONFLICT (content): Merge conflict in auth.go
error: could not apply 2b1c673... add password reset endpoint

# edit the conflicted file, removing the <<<<<<< ======= >>>>>>> markers
git add auth.go
git rebase --continue
```

Repeat for each conflicting commit. If the rebase becomes unmanageable,
run `git rebase --abort` to return to the state before you started. See
[Handling conflicts during a rebase](../rebasing#handling-conflicts-during-a-rebase)
for the full explanation.

### Bisect a regression

Reach for `git bisect` when you know a bug exists now, know a commit
where it did not exist, but have no idea which of the commits in between
introduced it. Bisect finds that commit with a binary search instead of a
manual review of every change.

```bash
git bisect start
git bisect bad HEAD              # the current commit has the bug
git bisect good v1.4.0           # this known-good tag does not

# git checks out a commit roughly halfway between the two; test it, then report back
git bisect good                  # or: git bisect bad
```

Git repeats the checkout-and-report cycle, narrowing the range each time,
until it identifies the exact commit that introduced the bug. Run
`git bisect reset` afterward to return to your original branch.

### Cherry-pick a hotfix

Use `git cherry-pick` when a fix needs to land on a branch other than the
one it was committed on, most often when a bug fix made on `main` also
needs to ship on a maintained release branch.

```bash
git log --oneline main            # find the commit hash of the fix
git checkout release/v2.1
git cherry-pick a7f92c1
```

Cherry-pick replays that single commit onto your current branch as a new
commit with a new hash, the same mechanism rebase uses to replay an entire
branch. If the pick conflicts, resolve it and run
`git cherry-pick --continue`, just as you would during a rebase.

## Intermediate: server side

These workflows let the server enforce rules automatically instead of
relying on contributors to remember them.

### Client and server hooks

A **hook** is a script git runs automatically at a specific point in the
commit or push process. Reach for a client-side hook to catch problems
before they leave your machine, and a server-side hook to enforce rules
no client can bypass.

A client-side `pre-commit` hook runs before a commit is created, and can
block it:

```bash
# .git/hooks/pre-commit (must be executable)
#!/bin/sh
golangci-lint run
```

A server-side `pre-receive` hook runs on the remote when it receives a
push, before any ref updates. It can reject the entire push, which makes
it the right place for rules a client-side hook cannot guarantee were run:

```bash
# hooks/pre-receive on the server (must be executable)
#!/bin/sh
while read oldrev newrev refname; do
  if git log "$oldrev..$newrev" --format=%s | grep -qi '^wip'; then
    echo "reject: WIP commit messages are not allowed on this branch"
    exit 1
  fi
done
```

### Trigger CI on push and PR

Set this up once a project has automated tests worth running. Configure
your CI provider to listen for the two events that matter most: a push to
any branch, and a pull request opened against `main`.

```yaml
# example: GitHub Actions trigger
on:
  push:
  pull_request:
    branches: [main]
```

Every push to a feature branch runs the suite early. The pull-request
trigger runs it again against the latest `main`, which catches conflicts
between your branch and other work that merged after your last push.

### Require reviews and status checks

Expand on
[Enable branch protection](#enable-branch-protection) once
the team and the CI pipeline have both grown. Require the CI check to
pass and require at least one approving review before a pull request can
merge, so neither a human nor an automated check alone can let a bad
change through.

1. Open the branch protection rule for `main`.
2. Add the CI job as a required status check.
3. Set required approvals to at least one, and enable **dismiss stale
   approvals** so a new push invalidates an old sign-off.

## Advanced: working alone

These workflows solve problems that only show up on long-lived branches,
large repositories, or after something has already gone wrong locally.

### Clean up history with interactive rebase

Use this before opening a pull request from a branch that has been open
for days and accumulated messy "WIP" or "fix review comment" commits. A
clean history lets the reviewer follow the story of the change instead of
the story of how you wrote it.

```bash
git fetch origin
git rebase -i origin/main
```

Squash and reorder commits in the editor that opens. See
[Interactive rebase](../rebasing#interactive-rebase) for the full command
reference, including `squash`, `fixup`, and `drop`.

### Parallel work with worktrees

Reach for `git worktree` when you need to work on two branches at once,
for example reviewing a pull request while you have uncommitted changes
on your own feature branch. It avoids the stash-switch-unstash cycle
entirely.

```bash
git worktree add ../project-review feature/other-branch
cd ../project-review    # a second working directory, same repository

# when finished
cd ../project
git worktree remove ../project-review
```

Each worktree has its own working directory and index, but they share the
same commit history and object database.

### Recover a bad reset with reflog

Use this immediately after a `git reset --hard` or a rebase removes
commits you needed. `git reflog` records every movement of `HEAD`, even
ones a normal `git log` can no longer reach.

```bash
$ git reflog
a7f92c1 HEAD@{0}: reset: moving to HEAD~1
2b1c673 HEAD@{1}: commit: add password reset endpoint

# recover the commit the reset removed
git checkout 2b1c673
git branch recovered-work
```

Git keeps reflog entries for about 90 days by default, which gives you a
wide window to recover from almost any local mistake.

### Submodule vs. subtree

Decide between these two approaches when one repository needs to include
another repository's code. Reach for a submodule when contributors need
to track the embedded project's own commit history independently. Reach
for a subtree when you want the embedded code to behave like a normal
part of your repository, with no extra commands required to clone it.

```bash
# submodule: a pointer to a specific commit in another repository
git submodule add git@github.com:username/library.git vendor/library
git submodule update --init --recursive

# subtree: the other repository's files, merged directly into your history
git subtree add --prefix vendor/library git@github.com:username/library.git main --squash
```

## Advanced: server side

These workflows cover running the server side of git yourself, rather
than relying on a hosted service.

### Self-host a git server

Reach for a self-hosted server when policy requires code to stay on
infrastructure you control. The simplest option is a bare repository
over SSH; for a team that wants a web UI, pull requests, and CI,
install a self-managed platform such as Gitea or GitLab instead.

```bash
# on the server: create a bare repository
ssh git@server 'git init --bare /srv/git/project.git'

# on a client: clone and push to it like any other remote
git clone git@server:/srv/git/project.git
git push origin main
```

A **bare repository** has no working tree. It stores only the git
directory, which is all a server needs since no one edits files directly
on it.

### Control access over SSH

Set this up before adding contributors to a self-hosted server, so each
person's access is scoped to what they need. Generate keys with the
steps in [Best practices](../best-practices#2-generate-an-ssh-key-and-add-it-to-your-git-host),
then restrict what each key can do on the server side.

```bash
# in the server's authorized_keys file, restrict a key to one command
command="git-shell -c \"$SSH_ORIGINAL_COMMAND\"",no-port-forwarding ssh-ed25519 AAAA... deploy-key
```

A **deploy key** is an SSH key tied to a single repository instead of a
person, typically used by automated systems that need read or write
access without a human's full account privileges.

### Mirror and back up a server

Run this on a schedule once a self-hosted server holds work that cannot
be recreated from anywhere else. A mirror clone tracks every branch and
tag, not just the ones a normal clone follows.

```bash
# initial mirror clone, including all refs
git clone --mirror git@server:/srv/git/project.git project-backup.git

# refresh it later, for example from a cron job
cd project-backup.git
git remote update
```

Store the mirror on separate storage from the primary server, so a
single hardware failure cannot take out both copies.

### Scale a monorepo

Reach for these two features once a repository grows large enough that a
full `git clone` and checkout take an impractical amount of time, which
is common on a monorepo containing many teams' projects.

```bash
# partial clone: skip downloading file contents until they are needed
git clone --filter=blob:none git@server:monorepo.git

# sparse-checkout: only populate the working tree with the paths you need
cd monorepo
git sparse-checkout init --cone
git sparse-checkout set services/billing libs/shared
```

Partial clone reduces what the initial clone downloads. Sparse-checkout
reduces what lands in your working tree. Most large-monorepo setups use
both together.

## Collaboration models

These are the higher-level patterns a team chooses once, which then shape
every workflow above. Pick the model that matches your team's size and
release process, not the other way around.

### Fork-and-PR vs. shared repo

Choose **fork-and-PR** for open-source projects or any team where most
contributors should not have write access to the main repository.
Contributors push to their own fork and open a pull request back to the
original repository.

Choose the **shared-repo feature branch** model, covered in
[Managing a software application](../best-practices#managing-a-software-application),
for a team where every contributor already has write access. It skips
the fork step and pushes feature branches directly to the shared
repository.

```bash
# fork-and-PR: push to your own fork, not the upstream repository
git remote add origin git@github.com:your-username/project.git
git remote add upstream git@github.com:original-owner/project.git
git push origin feature/password-reset
```

### Trunk-based development vs. GitFlow

Choose **trunk-based development** for a continuously deployed
application, where short-lived feature branches merge into `main`
multiple times a day. [Managing a software application](../best-practices#managing-a-software-application)
already documents GitHub Flow, a lightweight form of trunk-based
development built on feature branches and pull requests.

Choose **GitFlow** instead for a project that ships versioned releases on
a schedule, such as desktop or embedded software. GitFlow adds long-lived
`develop` and `release` branches on top of `main`, which gives you a
place to stabilize a release without blocking ongoing feature work:

```bash
main:     A-------------------M        (released versions only)
develop:   \---B---C-----E----/        (integration branch)
                   \     /
feature:            D---/              (short-lived feature branch)
```

### Code review gates and merge queues

Add a required review, covered in
[Require reviews and status checks](#require-reviews-and-status-checks),
once more than one person touches a codebase. Add a **merge queue** once
your team outgrows that: it serializes merges to a protected branch,
re-testing each pull request against the latest `main` immediately before
it merges, so two passing pull requests can never combine into a broken
`main`.

### Commit message and PR conventions

Agree on a commit message convention before the team's history grows
large enough that an inconsistent log becomes hard to search.
[Commit early and often](../best-practices#commit-early-and-often) already
covers the `<type>: <summary>` format this convention is built on;
apply it consistently across the team, and use the same `type` values in
pull request titles so both are searchable with the same terms.
