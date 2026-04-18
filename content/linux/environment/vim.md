+++
title = 'vim'
date = '2025-08-18T10:04:38-04:00'
weight = 20
draft = false
+++


vim is a modal text editor available on virtually every Linux system. It operates in distinct modes: normal mode for navigation and commands, and insert mode for typing text. Press `Esc` to return to normal mode at any time.

{{< admonition "vim.nox" note >}}
`vim.nox` is a minimal vim build for server environments. It includes scripting language support but omits the GUI components of full vim.
{{< /admonition >}}

## .vimrc

`.vimrc` is your personal vim configuration file, stored in your home directory. Settings here apply every time you open vim. It is part of the broader [dot file](../setup#dot-files) system for managing shell and tool configuration. A system-wide configuration file also exists at `/etc/vimrc`.

For example, enable line numbers on startup:

```bash
# ~/.vimrc
set number
```

Start vim without loading `.vimrc` to test settings in isolation:

```bash
vim -u NONE
```

## Basic Navigation

Navigate in normal mode without entering insert mode:

| Command    | Description                                    |
| :--------- | :--------------------------------------------- |
| `h`        | Move left                                      |
| `j`        | Move down                                      |
| `k`        | Move up                                        |
| `l`        | Move right                                     |
| `w`        | Move to the beginning of the next word         |
| `3w`       | Move to the 3rd word to the right              |
| `b`        | Move to the beginning of the previous word     |
| `3b`       | Move to the 3rd word to the left               |
| `e`        | Move to the end of the current word            |
| `0`        | Move to the beginning of the line              |
| `$`        | Move to the end of the line                    |
| `gg`       | Go to the first line of the file               |
| `G`        | Go to the last line of the file                |
| `Ctrl + u` | Move up half a screen                          |
| `Ctrl + d` | Move down half a screen                        |
| `(`        | Move to the beginning of the previous sentence |
| `)`        | Move to the beginning of the next sentence     |

## Editing

Commands for entering insert mode, deleting, yanking, and pasting text:

| Command    | Description                                         |
| :--------- | :-------------------------------------------------- |
| `i`        | Insert mode at the cursor                           |
| `I`        | Insert mode at the beginning of the line            |
| `a`        | Append mode after the cursor                        |
| `A`        | Append mode at the end of the line                  |
| `o`        | Open a new line below the current line              |
| `O`        | Open a new line above the current line              |
| `r`        | Replace the character under the cursor              |
| `x`        | Delete the character under the cursor               |
| `X`        | Delete the character before the cursor              |
| `dd`       | Delete the current line                             |
| `99dd`     | Delete 99 lines                                     |
| `dw`       | Delete from the cursor to the end of the word       |
| `3wd`      | Delete the 3 words to the right                     |
| `db`       | Delete the word to the left                         |
| `3db`      | Delete the 3 words to the left                      |
| `d$`       | Delete from the cursor to the end of the line       |
| `d0`       | Delete from the cursor to the beginning of the line |
| `das`      | Delete the current sentence                         |
| `yy`       | Copy (yank) the current line                        |
| `3yy`      | Copy three lines starting from the cursor           |
| `yw`       | Copy (yank) the current word                        |
| `yiw`      | Copy the word under the cursor                      |
| `y$`       | Copy (yank) to the end of the line                  |
| `y^`       | Copy from the cursor to the start of the line       |
| `y%`       | Copy to the matching bracket (`()`, `{}`, `[]`)     |
| `p`        | Paste after the cursor                              |
| `5p`       | Paste 5 times                                       |
| `P`        | Paste before the cursor                             |
| `u`        | Undo                                                |
| `Ctrl + r` | Redo                                                |

## Visual Mode

Visual mode lets you select text before applying a command. Three selection types are available:

| Command    | Description                                   |
| :--------- | :-------------------------------------------- |
| `v`        | Start visual mode (character selection)       |
| `V`        | Start visual line mode                        |
| `vjjj`     | Highlight the current line and 3 lines below  |
| `vjjjyy`   | Highlight and copy three lines                |
| `Ctrl + v` | Start visual block mode                       |
| `y`        | Yank (copy) the selected text                 |
| `d`        | Delete the selected text                      |
| `>`        | Indent the selected text                      |
| `<`        | Un-indent the selected text                   |

## Search and Replace

Search and replace commands work in normal mode:

| Command          | Description                                                |
| :--------------- | :--------------------------------------------------------- |
| `/pattern`       | Search forward for pattern                                 |
| `?pattern`       | Search backward for pattern                                |
| `n`              | Repeat the search in the same direction                    |
| `N`              | Repeat the search in the opposite direction                |
| `:%s/old/new/g`  | Replace all occurrences of old with new in the entire file |
| `:%s/old/new/gc` | Replace all occurrences with confirmation                  |

## Working with Multiple Files

vim loads each file into a buffer. Switch between buffers without closing the editor:

| Command       | Description               |
| :------------ | :------------------------ |
| `:e filename` | Open a file               |
| `:w`          | Save the current file     |
| `:w filename` | Save as filename          |
| `:q`          | Quit vim                  |
| `:wq`         | Save and quit             |
| `:q!`         | Quit without saving       |
| `:bn`         | Go to the next buffer     |
| `:bp`         | Go to the previous buffer |
| `:bd`         | Delete a buffer           |

## Useful Commands

Runtime settings and shell integration commands:

| Command             | Description                             |
| :------------------ | :-------------------------------------- |
| `:set nu`           | Show line numbers                       |
| `:set nonu`         | Hide line numbers                       |
| `:syntax on`        | Enable syntax highlighting              |
| `:syntax off`       | Disable syntax highlighting             |
| `:set paste`        | Enable paste mode                       |
| `:set nopaste`      | Disable paste mode                      |
| `:set tabstop=4`    | Set tab width to 4 spaces               |
| `:set expandtab`    | Convert tabs to spaces                  |
| `:set shiftwidth=4` | Set indentation width to 4 spaces       |
| `:! <command>`      | Run a shell command without leaving vim |

## Split windows

Split the editor to view multiple files at once. Use `Ctrl+w` twice to cycle between panes:

| Command             | Description                        |
| :------------------ | :--------------------------------- |
| `:sp /path/to/file` | Split horizontally and open a file |
| `:vs /path/to/file` | Split vertically and open a file   |
| `Ctrl+w` `Ctrl+w`   | Switch between open panes          |

## Exiting Vim

Exit commands work from normal mode. Press `Esc` first if you are in insert mode:

| Command       | Description         |
| :------------ | :------------------ |
| `:w`          | Save the file       |
| `:q`          | Quit vim            |
| `:wq` or `ZZ` | Save and quit       |
| `:q!`         | Quit without saving |