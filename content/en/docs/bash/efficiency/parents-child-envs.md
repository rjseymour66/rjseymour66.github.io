---
title: "Parent, children, and environments"
linkTitle: "Environments"
weight: 60
# description:
---

- implemented workflows and processes to improve documentatino accuracy, including custom Jira fields, review protocols, and Confluence documentation.
- Designed Jira dashboard for live documentation updates
- Converted API documentation from Markdown to Open API Specification.
- Reviewed documentation tools and presented findings to Senior management
- 

The shell is just another program on your computer. It displays a prompt, reads a command from stdin, and evaluates and runs the command.

When you log in, Linux runs your _login shell_. You see this if your log into a headless machine. On GUI machines, it runs in the background until you open a terminal.
- `bash` is a program like any other
- all available shells are stored in `/etc/shells`
- `SHELL` env var defines the current default shell
- run `bash` to start a new instance of the shell. `exit` to close it and return to the original shell.

## Parent and child processes

> Every time you run a command in Linux, you create a child process. The child process runs in its own environment, copied from the parent. This is true for EVERY command, including simple commands like `ls`.

Any program--including a shell--that invokes another program is the parent process. The invoked program is its child:
- each process has its own environment, such as shell vars
- child processes copy their parent's environment
- when you run a script, the script runs in a child process, so you can `cd` all over the filesystem and not change the directory in your parent process
  - `cd` is a builtin, which is why you can change directories in the parent process. Otherwise, it would be impossible
- each command in a pipeline launches its own child process

## Environment variables

Env vars are variables that are automatically copied from a parent shell to each child that it invokes, forming the shell's environment:
- `export` creates an env var and tells the shell to copy the var to all child processes
- local variables are the vars that you create in the shell
- env vars ARE NOT global variables. Changes to the env vars in child shells do not affect the parent vars
  - env vars like HOME seem global, but they are set by your login shell, which is the parent to all shells. Also, env vars like these are rarely changed
  - you can set local vars in config files like `.bashrc`, so they appear to be copied from shell to shell

```bash
printenv | sort                 # view sorted envars

MY_VAR=10                       # create local var
printenv MY_VAR                 # ...
export MY_VAR                   # convert to ENV VAR
printenv MY_VAR
10

export MY_VAR2=20               # create and export single command
```

## Child shells vs subshells

A child shell is a partial copy of its parent. It doesn't receive local variables or aliases. A subshell is a complete copy of its parent and includes all variables, aliases, functions, etc.
- scripts can't access aliases because they run in a child shell
- to create a subshell, enclose the command in parentheses

```bash
(ls)                            # run ls in subshell
(alias)                         # lists all parent aliases
```

## Configuring your environment

When a shell starts, it runs a series of configuration files to prepare the environment:
- System-wide configuration files are found in `/etc`.
- Individual config files are found in the home dir
- You should store your customized config files on GitHub

There are several types of config files:
- **Startup files**: Apply to your login shell, they run automatically when you login. You can set and export env vars, but don't set aliases because they are not copied to any child shells.
  Your distro provides one of these files, so 
- **Initialization (init) files**: Run for every shell instance that is not a login shell, including your terminal and any shell scripts. You can set variables or define and alias here.
- **Cleanup files**: Run right before login shell exits.


You should have your personal startup file (i.e. `.bash_profile`) source your personal initialization file (`.bashrc`). Add this command to the startup file to make that happen:
- `source` is a builtin. This is required because if it were just a program you run, it would run in a child shell and would not affect the environment

```bash
if [ -f "$HOME/.bashrc" ]
then
    source "$HOME/.bashrc"
fi
```

### File summary

| File type | Invoked by            | System-wide location                | Personal file location                                           |
| :-------- | :-------------------- | :---------------------------------- | :--------------------------------------------------------------- |
| Startup   | Login shell           | `/etc/profile`                      | `$HOME/.bash_profile`<br>`$HOME/.bash_login`<br>`$HOME/.profile` |
| Init      | Nonlogin shells       | `/etc/bashrc`<br>`/etc/bash.bashrc` | `$HOME/.bashrc`                                                  |
| Cleanup   | Login shell (on exit) | `/etc/bash.bash_logout`             | `$HOME/.bash_logout`                                             |