---
title: "Navigation example"
linkTitle: "Navigation example"
weight: 9
description: >
  Sample styling sheet for a navigation menu.
---

## Nav styling

When creating menu items:
- Apply padding to the internal `<a>` tags to provide more clickable surface area.
- Use `display: block;`. This allows its parent to derive the height from their padding, not line height.

Horizontal nav styling:

```css
.ul-class {
    display: flex;
    margin: 0;
    padding: .5em;
    background-color: #5f4b44;
    list-style-type: none;
    border-radius: .2em;
}

.ul-class > li {
    margin-top: 0;    
}

.ul-class > li > a {
    display: block;
    padding: .5em 1em;
    background-color: #cc6b5a;
    color: #fff;
    text-decoration: none;
}

.ul-class > li + li {
    margin-left: 1.5em;
}
```