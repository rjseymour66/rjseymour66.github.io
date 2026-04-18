---
title: "Configuration"
weight: 20
description: >
  Set up your identity, editor, and preferences with git config.
---

Before you commit anything, git needs to know who you are. Your name and email
are embedded in every commit you create. Git reads configuration from three
scopes, applying the most specific scope last:

| Scope | File | Flag |
|:---|:---|:---|
| System | `/etc/gitconfig` | `--system` |
| User | `~/.gitconfig` or `~/.config/git/config` | `--global` |
| Repository | `<project>/.git/config` | (none) |

Repository settings override user settings, and user settings override system
settings.

## Install git

On Debian and Ubuntu systems, install git with the following command:

```bash
sudo apt install git-all
```

## Set your identity

Set your name and email address globally so every repository on your machine
uses them:

```bash
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```

To override your identity for a single repository, run the same commands
without the `--global` flag from inside the repository directory:

```bash
git config user.name "John Doe"
git config user.email johndoe@example.com
```

## Set your editor

Git opens a text editor when you write a commit message or resolve a conflict.
Set your preferred editor with the following command:

```bash
git config --global core.editor emacs
```

## Set the default branch name

By default, git names the first branch `master`. Change it to `main` with
the following command:

```bash
git config --global init.defaultBranch main
```

## Common settings

### Line endings

On Linux and macOS, set `core.autocrlf` to `input` so git converts
Windows-style CRLF line endings to LF when you commit, but does not convert
on checkout:

```bash
git config --global core.autocrlf input
```

On Windows, set it to `true` so git converts LF to CRLF on checkout and back
to LF on commit:

```bash
git config --global core.autocrlf true
```

### Color output

Git disables color when it detects that output is not going to a terminal.
Force color on with the following command:

```bash
git config --global color.ui auto
```

### Credential caching

When you connect over HTTPS, git asks for your username and password on every
push. Cache credentials in memory for 15 minutes with the following command:

```bash
git config --global credential.helper cache
```

To keep credentials cached for longer, pass a timeout in seconds:

```bash
git config --global credential.helper 'cache --timeout=3600'
```

On macOS, `osxkeychain` stores credentials permanently in the system keychain:

```bash
git config --global credential.helper osxkeychain
```

## View your configuration

List all configuration values currently in effect:

```bash
$ git config --list
user.name=linuxuser
user.email=linuxuser@example.com
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true
```

To see each value alongside the file that set it, add `--show-origin`:

```bash
$ git config --list --show-origin
file:/etc/gitconfig   diff.astextplain.textconv=astextplain
file:~/.gitconfig     user.name=John Doe
file:~/.gitconfig     user.email=johndoe@example.com
```

To check a single value, pass the key name:

```bash
$ git config user.name
John Doe
```

## Inspect config files directly

You can read the raw config files with `cat`. The global config stores
user-level settings:

```bash
$ cat ~/.gitconfig
[user]
    name = linuxuser
    email = linuxuser@example.com
```

The local config stores repository-level settings:

```bash
$ cat .git/config
[core]
    repositoryformatversion = 0
    filemode = true
    bare = false
    logallrefupdates = true
```

## Removing a configuration value

Pass `--unset` to remove a specific key from a config file:

```bash
git config --global --unset core.editor
```

## Get help

View the full manual for any git command with any of the following:

```bash
git help <verb>
git <verb> --help
man git-<verb>
```

For example, to open the `git config` manual page:

```bash
git help config
```

## Aliases

Aliases let you create short names for long or frequently typed git commands.
Define them with `git config --global alias.<name> <command>`:

```bash
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.co checkout
```

You can also alias multi-word commands. This example creates a `last` alias
that shows the most recent commit:

```bash
git config --global alias.last 'log -1 HEAD'
```

Running `git last` is then equivalent to running `git log -1 HEAD`:

```bash
$ git last
commit 871eb64b61414d9633b8056ddb3d66e77609f8a6 (HEAD, tag: v1.0.2.1)
Author: Jeff Welling <jeff.welling@gmail.com>
Date:   Sun Apr 3 16:50:10 2011 -0400

    Bumped version to 1.0.2.1
```

A widely used alias combines `--oneline`, `--graph`, `--all`, and `--decorate`
into a single `lg` command for a readable branch overview:

```bash
git config --global alias.lg \
  'log --oneline --graph --all --decorate'
```
