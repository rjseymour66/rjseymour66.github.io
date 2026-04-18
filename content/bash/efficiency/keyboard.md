---
title: "Keyboard efficiency"
linkTitle: "Keyboard"
weight: 90
# description:
---

## Browsers

| Command  | Shortcut                  |
| -------- | ------------------------- |
| CTRL+N   | Open new window.          |
| CTRL+T   | Open new tab              |
| CTRL+W   | Close tab                 |
| CTRL+Tab | Cycle through browser tab |
| CTRL+L   | Jump to address bar       |


Open browsers from the shell:
- create aliases for these to simplify

```bash
# open home page redirect all diagnostic output to /dev/null
firefox &> /dev/null & 
google-chrome &> /dev/null &

# open web page in new tab
firefox https://example.com
google-chrome https://example.com

# open in new window
firefox --new-window https://example.com
google-chrome --new-window https://example.com

# open private window
firefox --private-window https://example.com
google-chrome --incognito https://example.com
```

## HTML with curl and wget

These tools download web pages and other web content:
- `curl` prints output to stdout
- `wget` saves output to a file

```bash
curl https://example.com
wget https://example.com
```

## Clipboard control

> SHIFT + INSERT will paste into the terminal!

In Linux, copy and paste operations are part of a mechanism called _X selections_. There are two selection types:
- **clipboard**: standard, familiar clipboard. You copy, it is stored on the clipboard, then you paste content from the clipboard.
- **primary selection**: in certain applications, selected content is written to the primary selection, even if you don't explicitly copy it. For example, when you highlight text in a window, it is automatically written to the primary selection.