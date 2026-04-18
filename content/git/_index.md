---
title: "git"
linkTitle: "git"
weight: 10
description: >
  Concepts, commands, and workflows for working with git.
---

Git is a *distributed version control system* (DVCS). When you clone a repository,
you receive a complete copy of the project and its full history. Any client can
restore a corrupted server from its local copy.

Rather than storing differences between file versions, git stores *snapshots* of
the entire project at each commit. When a file has not changed between commits,
git stores a reference to the previously saved version rather than duplicating it.
Each snapshot is identified by a SHA-1 hash: a 40-character string derived from
the file contents. If the contents change, the hash changes, so git always knows
when data has been modified.

## Repositories

A *local repository* is a full copy of the project stored on your machine. It
holds the complete tree and commit history. A *remote repository* is the same
structure hosted elsewhere: a cloud service, an on-site server, or another
developer's machine.

## Three states

Git tracks every file in one of three states:

*modified*
: You have changed the file but have not committed it to the database yet.

*staged*
: You have marked the modified file to go into the next commit snapshot.

*committed*
: The data is safely stored in your local database.

## Three sections

A git project has three main sections:

*working tree*
: A single checkout of one version of the project. Git pulls files out of the
compressed database in the git directory and places them on disk for you to
work with.

*staging area*
: A file inside the `.git` directory that records what goes into the next commit.
It is also called the *index*. When you run `git add`, git writes checksums,
timestamps, and filenames into `.git/index` and compresses the file contents
into `.git/objects/` as a *blob*.

*git directory*
: Where git stores the metadata and object database for the project. This is
what gets copied when you clone a repository. Key contents include:

  - `objects/` — compressed file snapshots (blobs) and commit metadata
  - `index` — the staging area file
  - `refs/` — pointers to commits (branches and tags)
  - `HEAD` — a pointer to the currently checked-out branch
  - `COMMIT_EDITMSG` — the message from the most recent commit

## General workflow

Every change you track in git follows the same three steps:

1. Modify files in the working tree.
2. Stage the changes you want to include.
3. Commit, which takes the staged snapshot and stores it in the git directory.

The diagram below shows how files move between states:

```
Untracked       Unmodified              Modified             Staged
    |                |                      |                   |
    | Add the file -------------------------------------------> |
    |                | Edit file -->        |                   |
    |                |                      | Stage file -----> |
    | <--Remove file |                      |                   |
    |                | <-------------------------------- Commit |
```
