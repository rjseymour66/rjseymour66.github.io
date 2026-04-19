+++
title = 'tmux'
date = '2026-04-19T00:00:00-04:00'
weight = 40
draft = false
+++

tmux is a terminal multiplexer: it lets you run multiple terminal sessions inside a single window, split that window into panes, and detach from sessions without killing them. The session persists on the server until you explicitly destroy it—even after you close your terminal or lose an SSH connection.

## Installation

Install tmux from your distribution's package manager:

```bash
# Debian/Ubuntu
sudo apt install tmux

# RHEL/Fedora
sudo dnf install tmux
```

Confirm the installed version:

```bash
tmux -V
```

## Core concepts

tmux organizes work into three layers:

| Layer | Description |
| :---- | :---------- |
| **Session** | A collection of windows. You attach to and detach from sessions. |
| **Window** | A single screen within a session. Equivalent to a browser tab. |
| **Pane** | A rectangular division of a window. Each pane runs its own shell. |

A tmux **server** runs in the background and manages all sessions. When you run `tmux`, it starts the server (if not already running) and creates your first session.

## Session management

Start a new named session:

```bash
tmux new-session -s mysession
```

List active sessions:

```bash
tmux ls
```

Attach to a session by name:

```bash
tmux attach-session -t mysession
```

| Key binding | Action |
| :---------- | :----- |
| `Prefix d` | Detach from the current session |
| `Prefix s` | Open the interactive session list |
| `Prefix $` | Rename the current session |
| `Prefix (` | Switch to the previous session |
| `Prefix )` | Switch to the next session |

Kill a session from the command line:

```bash
tmux kill-session -t mysession
```

## Window management

| Key binding | Action |
| :---------- | :----- |
| `Prefix c` | Create a new window |
| `Prefix n` | Move to the next window |
| `Prefix p` | Move to the previous window |
| `Prefix 0–9` | Jump to a window by its index number |
| `Prefix ,` | Rename the current window |
| `Prefix w` | Open the interactive window list |
| `Prefix &` | Close the current window (prompts for confirmation) |

## Pane management

| Key binding | Action |
| :---------- | :----- |
| `Prefix %` | Split the current pane vertically (side by side) |
| `Prefix "` | Split the current pane horizontally (top and bottom) |
| `Prefix ←→↑↓` | Move focus to the pane in that direction |
| `Prefix z` | Zoom the current pane to fill the window; press again to unzoom |
| `Prefix x` | Close the current pane (prompts for confirmation) |
| `Prefix {` | Swap the current pane with the one above it |
| `Prefix }` | Swap the current pane with the one below it |
| `Prefix q` | Display pane index numbers briefly |
| `Prefix Ctrl+←→↑↓` | Resize the current pane one cell at a time |
| `Prefix Space` | Cycle through the built-in pane layouts |

## Key bindings

The **prefix key** is the key combination you press before every tmux command. The default is `Ctrl+b`. To send a literal `Ctrl+b` to the running program, press it twice.

Two bindings are useful at any time:

| Key binding | Action |
| :---------- | :----- |
| `Prefix ?` | List all current key bindings |
| `Prefix :` | Open the tmux command prompt |

## Copy mode

Copy mode lets you scroll through terminal output and copy text without a mouse. tmux supports vi key bindings in copy mode.

Enable vi keys in `~/.tmux.conf`:

```bash
set-window-option -g mode-keys vi
```

Enter and exit copy mode:

| Key binding | Action |
| :---------- | :----- |
| `Prefix [` | Enter copy mode |
| `q` | Exit copy mode |

Navigate in copy mode using standard vi motion keys:

| Key | Action |
| :-- | :----- |
| `h` `j` `k` `l` | Move left, down, up, right |
| `w` / `b` | Jump forward / backward one word |
| `gg` / `G` | Jump to the top / bottom of the scrollback buffer |
| `/` | Search forward |
| `?` | Search backward |
| `n` / `N` | Jump to the next / previous search match |

Select and copy text:

| Key | Action |
| :-- | :----- |
| `Space` | Start a selection |
| `Enter` | Copy the selection and exit copy mode |

Paste the copied text:

| Key binding | Action |
| :---------- | :----- |
| `Prefix ]` | Paste the most recent buffer |

## `.tmux.conf`

tmux reads `~/.tmux.conf` at startup. Changes to this file do not apply to running sessions automatically.

Apply changes to the current session without restarting:

```bash
tmux source-file ~/.tmux.conf
```

Or from within tmux: `Prefix :` then `source-file ~/.tmux.conf`.

Common configuration options:

```bash
# Remap the prefix to Ctrl+a
unbind C-b
set -g prefix C-a
bind C-a send-prefix

# Enable mouse support
set -g mouse on

# Increase scrollback buffer
set -g history-limit 10000

# Start window and pane indices at 1 instead of 0
set -g base-index 1
setw -g pane-base-index 1

# Use vi keys in copy mode
setw -g mode-keys vi

# Remove the escape-key delay (important for vim users)
set -s escape-time 0
```

## Status bar customization

The status bar runs along the bottom of the tmux window. You can change its colors and content.

```bash
# Status bar colors
set -g status-bg colour235
set -g status-fg white

# Left and right content
set -g status-left "[#S] "
set -g status-right "#H | %Y-%m-%d %H:%M"

# Refresh interval in seconds
set -g status-interval 5
```

Common status bar variables:

| Variable | Value |
| :------- | :---- |
| `#S` | Session name |
| `#W` | Current window name |
| `#H` | Hostname |
| `#(cmd)` | Output of a shell command |

## Plugins

tmux Plugin Manager (TPM) installs and manages tmux plugins from GitHub.

**Install TPM:**

```bash
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

Add the following to the bottom of `~/.tmux.conf`:

```bash
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'

run '~/.tmux/plugins/tpm/tpm'
```

Reload your config, then install the plugins:

```bash
tmux source-file ~/.tmux.conf
```

Then from within tmux:

| Key binding | Action |
| :---------- | :----- |
| `Prefix I` | Install plugins |
| `Prefix U` | Update plugins |
| `Prefix Alt+u` | Remove unused plugins |

Recommended plugins:

| Plugin | Purpose |
| :----- | :------ |
| `tmux-plugins/tmux-sensible` | A set of sane defaults that most users agree on |
| `tmux-plugins/tmux-resurrect` | Save and restore sessions across reboots |
| `tmux-plugins/tmux-continuum` | Automatically save sessions on an interval |
| `tmux-plugins/tmux-yank` | Copy to the system clipboard from copy mode |

To add a plugin, append a `set -g @plugin` line for it and press `Prefix I`.

## Scripting with tmux

You can drive tmux non-interactively from shell scripts.

Create a detached session:

```bash
tmux new-session -d -s mysession
```

Check whether a session exists before creating it:

```bash
if ! tmux has-session -t mysession 2>/dev/null; then
    tmux new-session -d -s mysession
fi
```

Send a command to a session:

```bash
tmux send-keys -t mysession "htop" Enter
```

This pattern is useful for provisioning a standard workspace on login or in a dotfiles script.

## Recommended setup

The following `~/.tmux.conf` consolidates the settings from the sections above into a working starting configuration. Copy it as-is, then adjust to taste.

Install TPM first:

```bash
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

Then write `~/.tmux.conf`:

```bash
# Prefix
unbind C-b
set -g prefix C-a
bind C-a send-prefix

# General
set -g mouse on
set -g history-limit 10000
set -s escape-time 0

# Indexing: start windows and panes at 1
set -g base-index 1
setw -g pane-base-index 1

# Copy mode
setw -g mode-keys vi

# Status bar
set -g status-bg colour235
set -g status-fg white
set -g status-left "[#S] "
set -g status-right "#H | %Y-%m-%d %H:%M"
set -g status-interval 5

# Plugins
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'tmux-plugins/tmux-yank'

run '~/.tmux/plugins/tpm/tpm'
```

Apply the config and install the plugins:

```bash
tmux source-file ~/.tmux.conf
```

Then from within tmux, press `Prefix I` to install the plugins.
