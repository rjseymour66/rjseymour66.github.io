---
title: "Commit history"
weight: 40
description: >
  Browse, filter, and format the commit log with git log.
---

`git log` is your primary tool for understanding what happened in a repository
and when. You might run it to find which commit introduced a bug, to see what
a colleague pushed while you were away, or to verify your own work before
opening a pull request. It lists commits in reverse chronological order by
default.

[Full list of common options](https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History#log_options)

## Quick reference

The table below summarizes the most common `git log` flags:

| Flag | Purpose |
|:---|:---|
| `-p` / `--patch` | Show the full diff for each commit |
| `--stat` | Show insertion and deletion counts per file |
| `--oneline` | One line per commit: short hash and subject |
| `--graph` | Draw an ASCII branch-and-merge graph |
| `--all` | Include commits from all branches |
| `--decorate` | Show branch and tag names next to commits |
| `-n <number>` | Limit output to the most recent N commits |
| `--since=<date>` | Show commits after a date |
| `--until=<date>` | Show commits before a date |
| `--author=<name>` | Filter by author name or email |
| `--grep=<pattern>` | Filter by commit message text |
| `-S <string>` | Show commits that added or removed a string |
| `--no-merges` | Exclude merge commits |
| `--follow -- <file>` | Follow a file through renames |

## Viewing diffs and statistics

### Show the patch for each commit

Pass `-p` or `--patch` to print the full diff introduced by each commit:

```bash
$ git log -p
commit ca82a6dff817ec66f44342007202690a93763949 (HEAD -> master, origin/master)
Author: Scott Chacon <schacon@gmail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number

diff --git a/Rakefile b/Rakefile
index a874b73..8f94139 100644
--- a/Rakefile
+++ b/Rakefile
@@ -5,7 +5,7 @@ require 'rake/gempackagetask'
-    s.version   =   "0.1.0"
+    s.version   =   "0.1.1"
```

Limit output to the last N commits by combining `-p` with a count. This
example shows the patch for only the most recent commit:

```bash
git log -p -1
```

The `-n` flag works the same way for any subcommand:

```bash
git log -n 5          # show the last 5 commits
```

### Show abbreviated statistics

Pass `--stat` to print a short summary of insertions and deletions for each
commit without the full diff:

```bash
$ git log --stat
commit a11bef06a3f659402fe7563abf99ad00de2209e6
Author: Scott Chacon <schacon@gmail.com>
Date:   Sat Mar 15 10:31:28 2008 -0700

    first commit

 README           |  6 ++++++
 Rakefile         | 23 +++++++++++++++++++++++
 lib/simplegit.rb | 25 +++++++++++++++++++++++++
 3 files changed, 54 insertions(+)
```

## Formatting output

### --oneline

`--oneline` is the fastest way to scan history. It prints one commit per line
with the abbreviated hash and the subject:

```bash
$ git log --oneline
ca82a6d (HEAD -> master, origin/master) changed the version number
085bb3b removed unnecessary test code
a11bef0 first commit
```

`--oneline` is shorthand for `--pretty=oneline --abbrev-commit`.

### --pretty presets

`--pretty` changes the format of each log entry. The built-in presets are:

| Preset | Output |
|:---|:---|
| `oneline` | Hash and subject on one line |
| `short` | Hash, author, and subject |
| `full` | Hash, author, committer, and subject |
| `fuller` | Same as `full` plus author and commit dates |

The following examples show each preset:

```bash
# short: hash, author, subject
$ git log --pretty=short
commit ca82a6dff817ec66f44342007202690a93763949 (HEAD -> master)
Author: Scott Chacon <schacon@gmail.com>

    changed the version number

# full: adds committer
$ git log --pretty=full
commit ca82a6dff817ec66f44342007202690a93763949 (HEAD -> master)
Author: Scott Chacon <schacon@gmail.com>
Commit: Scott Chacon <schacon@gmail.com>

    changed the version number

# fuller: adds author and commit dates
$ git log --pretty=fuller
commit ca82a6dff817ec66f44342007202690a93763949 (HEAD -> master)
Author:     Scott Chacon <schacon@gmail.com>
AuthorDate: Mon Mar 17 21:52:11 2008 -0700
Commit:     Scott Chacon <schacon@gmail.com>
CommitDate: Fri Apr 17 21:56:31 2009 -0700

    changed the version number
```

### Custom format

`--pretty=format:"..."` builds a custom output line using format specifiers.
The most useful specifiers are:

| Specifier | Output |
|:---|:---|
| `%H` | Full commit hash |
| `%h` | Abbreviated commit hash |
| `%an` | Author name |
| `%ae` | Author email |
| `%ar` | Author date, relative (for example, "3 days ago") |
| `%ad` | Author date, absolute |
| `%s` | Subject (first line of commit message) |
| `%d` | Ref names (branch and tag labels) |

[Full list of format specifiers](https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History#pretty_format)

The following example prints the abbreviated hash, author name, relative date,
and subject:

```bash
$ git log --pretty=format:"%h - %an, %ar : %s"
ca82a6d - Scott Chacon, 16 years ago : changed the version number
085bb3b - Scott Chacon, 16 years ago : removed unnecessary test code
a11bef0 - Scott Chacon, 16 years ago : first commit
```

### Branch and merge graph

Combine `--oneline`, `--graph`, `--all`, and `--decorate` to draw a full
picture of every branch and merge in the repository. This is one of the most
useful combinations for understanding a project's branching history:

```bash
$ git log --oneline --graph --all --decorate
* 4e3af98 (HEAD -> feature) add login page
* 2b1c673 add user model
| * ca82a6d (main) changed the version number
| * 085bb3b removed unnecessary test code
|/
* a11bef0 first commit
```

Without `--all`, git shows only the history reachable from the current branch.
`--decorate` adds branch and tag labels to each commit.

## Filtering commits

### By date

Pass `--since` or `--until` to bound the output to a time range. Both
relative and absolute formats work:

```bash
# commits from the last two weeks
git log --since=2.weeks

# commits on a specific day
git log --since="2008-01-15" --until="2008-01-16"
```

### By author

Pass `--author` to filter commits by name or email. The value is a regular
expression, so partial matches work:

```bash
git log --author="Scott Chacon"
git log --author="@gmail.com"    # everyone with a Gmail address
```

### By commit message

Pass `--grep` to show only commits whose message matches a pattern. This is
useful for finding all commits related to a ticket or feature:

```bash
# find all commits mentioning "authentication"
git log --grep="authentication"

# combine with --all-match to require multiple patterns simultaneously
git log --grep="fix" --grep="auth" --all-match
```

### By string in the diff (pickaxe)

Pass `-S` with a string to show only commits that added or removed that exact
string in the code. This is the fastest way to find when a function was
introduced or deleted:

```bash
git log -S function_name
```

### By commit range

The `branch1..branch2` syntax shows commits reachable from `branch2` but not
from `branch1`. This is the standard way to preview what a feature branch adds
before merging it:

```bash
# see what feature adds on top of main
git log main..feature

# see what main has that feature does not (commits you need to catch up on)
git log feature..main
```

### By file or path

Pass `--` followed by a path to show only commits that touched that file or
directory:

```bash
git log -- path/to/file
```

To follow a file through renames, add `--follow`:

```bash
git log --follow -- path/to/file
```

### Exclude merge commits

Pass `--no-merges` to omit merge commits from the output:

```bash
git log --no-merges
```
