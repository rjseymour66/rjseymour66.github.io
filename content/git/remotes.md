---
title: "Remotes"
weight: 60
description: >
  Add, inspect, fetch from, and push to remote repositories.
---

A *remote* is a version of your repository hosted at another location: a
service like GitHub or GitLab, an on-site server, or even another directory on
the same machine. Remotes let teams share work. You push commits to share
your changes and fetch or pull to receive changes from others.

## Managing remotes

### git remote

`git remote` lists the remotes configured for the current repository. The
default name git assigns to the server you cloned from is `origin`:

```bash
$ git remote
origin
```

Pass `-v` for the full fetch and push URLs:

```bash
$ git remote -v
origin  https://github.com/schacon/ticgit (fetch)
origin  https://github.com/schacon/ticgit (push)
```

### Adding a remote

`git remote add <name> <url>` registers a new remote. The name is the
shorthand you use in push and fetch commands:

```bash
# HTTPS
git remote add origin https://github.com/username/repo.git

# SSH (preferred for passwordless authentication)
git remote add origin git@github.com:username/repo.git
```

### Inspecting a remote

`git remote show <name>` reports the remote's URLs, tracked branches, and
the local branches configured for push and pull:

```bash
$ git remote show origin
* remote origin
  Fetch URL: https://github.com/schacon/ticgit
  Push  URL: https://github.com/schacon/ticgit
  HEAD branch: master
  Remote branches:
    master tracked
  Local branches configured for 'git pull':
    master merges with remote master
  Local refs configured for 'git push':
    master pushes to master (up to date)
```

### Renaming and removing a remote

Rename a remote with `git remote rename`. All tracking references update
automatically:

```bash
$ git remote rename origin upstream
$ git remote
upstream
```

Remove a remote with `git remote remove`. This deletes all associated
tracking branches and configuration:

```bash
$ git remote remove upstream
$ git remote
(empty)
```

## Receiving changes

### git fetch

`git fetch <remote>` downloads all commits, branches, and tags from the
remote that you do not have locally. It does not merge anything into your
working tree. The downloaded data lands in remote-tracking branches such as
`origin/main`, which you can inspect before deciding to integrate:

```bash
git fetch origin
```

Fetch from all configured remotes at once:

```bash
git fetch --all
```

### git pull

`git pull` fetches from the tracked remote branch and immediately merges the
result into your current branch. It is equivalent to `git fetch` followed by
`git merge`:

```bash
git pull
```

On teams that prefer a linear history, configure pull to rebase instead of
merge. This replays your local commits on top of the incoming changes rather
than creating a merge commit:

```bash
# set rebase as the default pull behavior
git config --global pull.rebase true

# or pass it once per pull
git pull --rebase
```

## Sharing changes

### git push

`git push <remote> <branch>` uploads your local commits to the remote branch.
Two conditions must be true for a push to succeed: you must have write access,
and no one must have pushed commits to that branch since your last fetch. If
the remote has commits you do not have, fetch and integrate them first:

```bash
# push the current branch to origin
git push origin main
```

### Setting an upstream branch

Pass `-u` (or `--set-upstream`) the first time you push a new branch. Git
records the association so future `git push` and `git pull` commands know
which remote branch to target without arguments:

```bash
git push -u origin feature-login
```

After setting the upstream, pushing is just:

```bash
git push
```

### Deleting a remote branch

Pass `--delete` to remove a branch from the remote. This is typically done
after a feature branch has been merged:

```bash
git push origin --delete feature-login
```
