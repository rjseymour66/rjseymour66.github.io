---
title: "ARIA attributes"
# linkTitle: "ARIA attributes"
weight: 10
# description:
---

| ARIA Attribute     | Description                                                            | Use Case                                               | Example Snippet                                                          |
| ------------------ | ---------------------------------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------ |
| `aria-label`       | Provides an accessible label for an element                            | When no visible label is available                     | `<button aria-label="Close"></button>`                                   |
| `aria-labelledby`  | References another element that labels this one                        | For complex widgets or grouped labels                  | `<div aria-labelledby="section-title">`                                  |
| `aria-describedby` | References text that describes the element                             | Add supplemental description                           | `<input aria-describedby="hint">`                                        |
| `aria-hidden`      | Hides an element from assistive tech                                   | Hide decorative elements                               | `<span aria-hidden="true">★</span>`                                      |
| `aria-expanded`    | Indicates whether a section or control is expanded                     | Toggle sections like accordions or menus               | `<button aria-expanded="false">Menu</button>`                            |
| `aria-pressed`     | Indicates the pressed state of a toggle button                         | Toggle buttons                                         | `<button aria-pressed="false">Bold</button>`                             |
| `aria-checked`     | Indicates the checked state of a checkbox/radio                        | Custom checkbox widgets                                | `<div role="checkbox" aria-checked="true"></div>`                        |
| `aria-selected`    | Indicates the selected state of an item in a set                       | Tabs, selectable lists                                 | `<li aria-selected="true">Tab 1</li>`                                    |
| `aria-disabled`    | Marks an element as disabled (not interactive)                         | Disable elements not using native `disabled`           | `<div aria-disabled="true">Save</div>`                                   |
| `aria-controls`    | References the element that is controlled                              | Button controlling a menu or dialog                    | `<button aria-controls="menu1">Open Menu</button>`                       |
| `aria-live`        | Announces content changes dynamically                                  | Alerts, chat updates                                   | `<div aria-live="polite">New message</div>`                              |
| `role`             | Defines the role of an element                                         | Markup custom widgets or landmarks                     | `<div role="dialog">...</div>`                                           |
| `aria-current`     | Indicates the current item within a set                                | Current page in nav                                    | `<a aria-current="page" href="/about">About</a>`                         |
| `aria-modal`       | Indicates whether a dialog is modal                                    | Accessibility for modals                               | `<div role="dialog" aria-modal="true">`                                  |
| `aria-busy`        | Signals loading state                                                  | For areas loading content                              | `<div aria-busy="true">Loading...</div>`                                 |
| `aria-haspopup`    | Indicates the element has a popup (like a menu, dialog, listbox, etc.) | Use with buttons or links that trigger menus or popups | `<button aria-haspopup="menu" aria-controls="dropdown">Options</button>` |
