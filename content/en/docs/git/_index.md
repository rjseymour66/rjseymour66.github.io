---
title: "git"
linkTitle: "git"
# weight: 40
description: >
  Notes about basic git.
---

Git is a distributed version control system (DVCS) that stores a complete copy of a repository on your local machine. To track files, it stores snapshots of each file in a compressed database. Git tracks each of these files using a SHA-1 hash checksum--each time you commit a new or modified file to the database, git creates a checksum. If you do not change a file during a version, git copies the unchanged file into the new version.

# States and directories

Git uses 3 stages: 
- committed. Stored in your local db
- modified. Changed, but not committed to the local db
- staged. Marked a modified file to go into the next commit snapshot

Git uses 3 main sections of the git project:
- .git directory, the object and metadata database of the project. This is the cloned repo
- working directory, a checkout of one version of the .git directory. git checks files out of the .git database and places them in a directory in your filesystem so you can work on them
- staging area (also called the "index"), stores info about what is going into the next commit

# Configuration

git configuration values are stored in the `.gitconfig` file, located in one of the following places:
- `/etc/gitconfig`: system-wide git configurations
- `~/.gitconfig`: user git configurations. Set these values with the `--global` option
- `project/.git/config`: local project repository configuration

Return git configuration information with the following command:

```
$ git config --list
```