---
title: "Tagging"
weight: 70
description: >
  Mark releases and significant points in history with git tag.
---

Tags mark specific commits as significant, most commonly to identify release
points such as `v1.0.0` or `v2.3.1`. Unlike branches, tags do not move when
new commits are added. Git supports two types: *lightweight* tags, which are
simple pointers to a commit, and *annotated* tags, which are full objects
stored in the git database with their own metadata and optional signature.

Prefer annotated tags for releases. They carry the tagger's name, email,
date, and a message, and they can be signed with GPG. Lightweight tags are
better suited for temporary or local markers.

## Listing tags

List all tags in the repository:

```bash
$ git tag
v1.2
v1.4
v1.4-lw
```

Filter by pattern with `-l`:

```bash
$ git tag -l "v1.*"
v1.2
v1.4
v1.4-lw
```

## Annotated tags

### Creating an annotated tag

Pass `-a` to create an annotated tag and `-m` to supply the message inline.
If you omit `-m`, git opens your editor:

```bash
git tag -a v1.4 -m 'my version 1.4'
```

`git show <tag>` displays the tag object followed by the commit it points to:

```bash
$ git show v1.4
tag v1.4
Tagger: rseymour <rseymour@opentext.com>
Date:   Mon Apr 15 09:30:00 2024 -0400

my version 1.4

commit 3ecedf1e2e6554581fc298926090a10b1e89bfcd (HEAD -> master, tag: v1.4)
Author: rseymour <rseymour@opentext.com>
Date:   Fri Apr 12 08:56:35 2024 -0400

    add to commit
...
```

## Lightweight tags

### Creating a lightweight tag

Create a lightweight tag by passing only the tag name with no flags. It stores
just the commit checksum in a file with no additional metadata:

```bash
git tag v1.4-lw
```

`git show` on a lightweight tag outputs only the commit, not a tag object:

```bash
$ git show v1.4-lw
commit 3ecedf1e2e6554581fc298926090a10b1e89bfcd (HEAD -> master, tag: v1.4-lw, tag: v1.4)
Author: rseymour <rseymour@opentext.com>
Date:   Fri Apr 12 08:56:35 2024 -0400

    add to commit
...
```

## Tagging an older commit

Pass a commit hash to tag a point in history that has already passed. This is
useful when you forgot to tag at release time:

```bash
# tag commit ff612a9 as v1.2
$ git tag -a v1.2 ff612a9 -m 'tag older commit'

$ git show v1.2
tag v1.2
Tagger: rseymour <rseymour@opentext.com>
Date:   Mon Apr 15 09:36:55 2024 -0400

tag older commit

commit ff612a95b393dc8b8f36ada8394d16b265c31611 (tag: v1.2)
Author: rseymour <rseymour@opentext.com>
Date:   Thu Apr 11 09:09:41 2024 -0400

    rm rmfile
```

## Sharing tags

Git does not push tags automatically. You must push them explicitly.

Push a single tag:

```bash
git push origin v1.4
```

Push all tags (both lightweight and annotated):

```bash
git push origin --tags
```

Push only annotated tags, which is the preferred option for releases:

```bash
git push origin --follow-tags
```

## Deleting tags

Delete a tag locally:

```bash
$ git tag -d v1.4-lw
Deleted tag 'v1.4-lw' (was 3ecedf1)
```

Delete a tag from the remote:

```bash
git push origin --delete v1.4-lw
```

## Checking out a tag

`git checkout <tag>` switches the repository to the state the tag points to.
This puts the repository in *detached HEAD* state: HEAD points directly to a
commit rather than to a branch. Any commits you create in this state belong to
no branch and are reachable only by their hash:

```bash
$ git checkout v1.0.2.1
Note: switching to 'v1.0.2.1'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

HEAD is now at 871eb64 Bumped version to 1.0.2.1
```

If you need to make changes at a tagged point — for example, to apply a bug
fix to an older release — create a branch first:

```bash
git checkout -b hotfix/v1.0.3 v1.0.2.1
```

This gives the new commits a home on a branch where they remain reachable.
