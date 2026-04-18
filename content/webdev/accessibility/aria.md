---
title: "ARIA attributes"
# linkTitle: "ARIA attributes"
weight: 10
# description:
---

*Accessible Rich Internet Applications* (ARIA) is a set of HTML attributes that communicate the role, state, and properties of user interface elements to assistive technologies. Before reaching for ARIA, always check whether a native HTML element already conveys the same meaning. A `<button>` is always preferable to `<div role="button">`, and a `<nav>` is always preferable to `<div role="navigation">`.

## Rules of ARIA use

The W3C defines five rules for ARIA. Apply them in order:

1. **Do not add ARIA if a native HTML element or attribute already provides the semantics or behavior you need.** For example, apply `<button>` instead of `<div role="button">`. Native elements carry built-in keyboard support, focus management, and screen reader announcements that ARIA cannot fully replicate.

2. **Do not change native semantics unless you have no other option.** For example, do not add `role="heading"` to a `<button>`. If you need a heading, apply the appropriate `<h1>` through `<h6>` element.

3. **All interactive ARIA controls must be keyboard accessible.** If you create a custom widget with a role like `slider` or `menu`, keyboard users must be able to operate it without a mouse. This means handling `keydown` and `keyup` events in addition to `click`.

4. **Do not apply `role="presentation"` or `aria-hidden="true"` to a focusable element.** Hiding a focusable element from the accessibility tree while leaving it in the tab order creates a broken experience. For example, a close button hidden with `aria-hidden="true"` is still reachable by keyboard, but the screen reader announces nothing when the button receives focus.

5. **All interactive elements must have an accessible name.** An icon button with no visible label must still provide a name through `aria-label`, `aria-labelledby`, or visible text content.

For the full specification, see the [W3C notes on ARIA use](https://www.w3.org/TR/using-aria/#NOTES).

## Cheatsheet

The following table lists common ARIA attributes, what they communicate, and when to apply them:

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
