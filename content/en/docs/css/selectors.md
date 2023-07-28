---
title: "Selectors"
linkTitle: "Selectors"
weight: 6
description: >
  Notes about basic Selectors.
---

## Selectors

parent > child: `>` is a descendant selector that targets an element that is child of `parent`.

## Input elements

The following selects all `input` elements that are not checkboxes and not radio buttons:

```css
input:not([type:checkbox]):not([type:radio]) {
    display: block;
    width: 100%;
    margin-top: 0;
}
```

For input elements, the width is determined by the size attribute, which is the number of characters it should contain without scrolling. Use the `width` attribute to force a specific width value.

