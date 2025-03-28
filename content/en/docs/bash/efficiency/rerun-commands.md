---
title: "Rerunning commands"
# linkTitle: ""
weight: 30
# description:
---

## History

Lists all commands previously run in the shell:
- `HISTSIZE` env var determines how many commands are saved, 500 by default.
- Set custom `HISTSIZE` in `.bashrc`. `-1` is unlimited.
- Commands are saved unevaluted. For example, `echo $HOME`
- `HISTCONTROL` lets you manage which commands are saved
- `HISTCONTROL=ignoredups` does not save identical, consecutive commands.
- Each shell has its own, separate history
- `HISTFILE` is where the shell writes history, usually `$HOME/.bash_history`
  - When a shell exits, it writes history to `HISTFILE`
  - When a new shell starts, it loads `HISTFILE` as its history

```bash
history                         # print all shell history
history 5                       # print last 5 commands
history | less                  # pipe to less
history | sort -nr | less       # earliest to latest
history | grep -w cd            # print commands with 'cd'
history -c                      # clear history for current shell
```

### History expansion

Lets you access shell history with special expressions:

```bash
!!                      # most recent command
!grep                   # most recent command that began with grep
!?grep?                 # most recent command that contained grep
!8                      # run 8th command in history
!-2                     # run command executed 2 commands ago
!-2:p                   # print but do not run command executed 2 commands ago
!$                      # final arg in previous command
!*                      # all args in previous command

# alternative to alias="rm -i"
ls <file-pattern>       # confirm returns files to delete
rm !$                   # delete files with final arg in previous command
```

### Incremental search

Lets you type a few characters of a command and the rest appears instantly, like autocomplete:
- **CTRL + R** starts the interactive prompt, and it cycles through all matching options
- **CTRL + G** or **CTRL + C** exits the interactive prompt
- **Enter** executes the command

### Fixing mistakes

```bash
^wrong^right            # correct error in previous command (caret syntax)
!!:s/wrong/right/       # substitution with history expansion
```

### vim- or emacs-style editing

A powerful way to edit a command line is with vim or emacs commands. By default, the shell uses emacs:
- press **Esc** to enter editor-style mode
- change style with `set -o <editor>`. Add to `.bashrc` to make it permanent

```bash
set -o vi               # use vi style
set -o emacs            # use emacs style
```