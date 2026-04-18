---
title: "Best practices"
weight: 10
description: >
  Professional workflows for software projects, sysadmin files, and
  environment setup.
---

This page covers three real-world git workflows: managing a software
application as part of a team, tracking system administration files, and
configuring a new workstation from scratch. Each workflow builds on the
commands covered in the other pages in this section.

## Environment setup

Run these steps once on any new machine before working with git.

### 1. Install git and set your identity

```bash
sudo apt install git-all

git config --global user.name "Your Name"
git config --global user.email you@example.com
git config --global init.defaultBranch main
git config --global core.autocrlf input    # Linux/macOS
git config --global pull.rebase true       # keep history linear on pull
git config --global color.ui auto
```

### 2. Generate an SSH key and add it to your git host

SSH authentication is faster and more secure than HTTPS with a password.
Generate a key pair and copy the public key to GitHub or GitLab:

```bash
# generate an ed25519 key (recommended over RSA)
ssh-keygen -t ed25519 -C "you@example.com"

# start the SSH agent and load the key
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# print the public key to copy into GitHub -> Settings -> SSH Keys
cat ~/.ssh/id_ed25519.pub
```

Test the connection after adding the key:

```bash
ssh -T git@github.com
# Hi username! You've successfully authenticated.
```

### 3. Set up a global .gitignore

A global `.gitignore` keeps editor files, OS artifacts, and secrets out of
every repository without editing each project's own `.gitignore`:

```bash
# create the file
touch ~/.gitignore_global

# tell git to use it
git config --global core.excludesfile ~/.gitignore_global
```

Common entries to add:

```
# macOS
.DS_Store

# Linux
*~

# editors
.vscode/
.idea/
*.swp

# secrets (never commit these)
.env
.env.local
*.pem
*.key
```

### 4. Add useful aliases

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.lg 'log --oneline --graph --all --decorate'
git config --global alias.last 'log -1 HEAD'
```

---

## Managing a software application

This workflow, known as *GitHub Flow*, is the standard approach for teams
working on a continuously deployed application. Every change moves through
a feature branch and a pull request before reaching `main`.

```
main:    A---B-----------M---N      (protected, always deployable)
                  \     /
feature:           C---D            (your work in isolation)
```

### Daily workflow

#### Start each session

Pull the latest changes from `main` before creating a branch. This reduces
the chance of conflicts later:

```bash
git checkout main
git pull
```

#### Create a branch for each unit of work

Name branches so the purpose is clear from the name alone:

```bash
# feature
git checkout -b feature/user-authentication

# bug fix
git checkout -b fix/login-timeout

# documentation update
git checkout -b docs/api-reference
```

#### Commit early and often

Small, focused commits are easier to review and easier to revert if something
breaks. Write the commit message as a short imperative sentence describing
what the commit does, not what you did:

```bash
git add -p                          # stage only what belongs in this commit
git commit -m 'add JWT validation middleware'
```

A useful commit message structure:

```
<type>: <short summary under 72 characters>

<optional body explaining why, not what>
```

Common types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`.

#### Keep your branch up to date

If `main` has moved ahead while you were working, rebase your branch onto it
rather than merging. This keeps the feature branch history linear and makes
the eventual merge clean:

```bash
git fetch origin
git rebase origin/main
```

#### Push and open a pull request

```bash
git push -u origin feature/user-authentication
```

Then open a pull request on GitHub or GitLab. Keep pull requests small enough
to review in one sitting. A good rule of thumb is under 400 lines changed.

#### After the pull request merges

Delete the local and remote branches to keep the repository clean:

```bash
git checkout main
git pull
git branch -d feature/user-authentication
git push origin --delete feature/user-authentication
```

---

## Tracking system administration files

Sysadmins regularly edit files in `/etc`, `~/.config`, and home directories.
Tracking these in git gives you a history of every change, a way to roll back
a misconfiguration, and a record of who changed what and when.

### Option 1: Track a single config directory

If your changes are confined to one directory, initialize git inside it:

```bash
cd /etc/nginx
sudo git init
sudo git add nginx.conf sites-available/
sudo git commit -m 'initial nginx configuration'
```

Push to a private remote so the history is backed up and accessible from other
machines:

```bash
sudo git remote add origin git@github.com:username/nginx-config.git
sudo git push -u origin main
```

### Option 2: Track dotfiles with a bare repository

A *bare repository* is a git database without a working tree. This technique
lets you track files anywhere in your home directory without nesting `.git`
inside `~`:

```bash
# initialize a bare repo in a hidden directory
git init --bare ~/.dotfiles

# create an alias so you do not have to type the flags every time
echo "alias dotfiles='git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'" \
  >> ~/.bashrc
source ~/.bashrc

# hide untracked files so git status is not noisy
dotfiles config --local status.showUntrackedFiles no

# add your configuration files
dotfiles add ~/.bashrc ~/.vimrc ~/.ssh/config
dotfiles commit -m 'initial dotfiles'
dotfiles remote add origin git@github.com:username/dotfiles.git
dotfiles push -u origin main
```

To restore your environment on a new machine:

```bash
git clone --bare git@github.com:username/dotfiles.git ~/.dotfiles
alias dotfiles='git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'
dotfiles checkout
```

### Protecting secrets

Never commit passwords, API keys, or private keys. Add a global ignore pattern
for common secret file names (see Environment setup above) and audit staged
files before every commit:

```bash
git diff --staged    # review exactly what is going into the commit
```

If you accidentally commit a secret, treat it as compromised immediately:
revoke and rotate the credential. Removing it from history with
`git filter-branch` or `git filter-repo` does not protect it if the commit
was ever pushed to a remote.

### Recording changes to system files

When you edit a system file, commit with a message that explains why the
change was made, not just what changed. Future you and your colleagues will
thank you:

```bash
# vague: describes what, not why
git commit -m 'update sshd_config'

# clear: explains the reason
git commit -m 'disable password auth, require SSH keys per security audit'
```

---

## Recovering from common mistakes

| Mistake | Recovery |
|:---|:---|
| Committed to `main` instead of a branch | `git branch feature/oops`, `git reset --hard HEAD~1`, switch to the new branch |
| Committed a secret | Revoke the credential immediately, then remove with `git filter-repo` |
| Pushed a bad commit to a shared branch | `git revert <hash>` creates a new commit that undoes it without rewriting history |
| Lost commits after a reset | `git reflog` shows every HEAD movement; check out the commit to recover it |
| Merge conflict you cannot resolve | `git merge --abort` cancels the merge and restores the pre-merge state |
