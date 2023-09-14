---
title: "Box model"
linkTitle: "Box model"
weight: 2
description: >
  Notes about basic Box model.
---

## Document flow

_Normal document flow_ is how inline and block elements behave on in the HTML document.

_Inline elements_ flow along with the text of the page and will line wrap if they reach the end of their container.

_Block elements_ have a line break before and below them. The user-agent stylesheet sets block elements to `display: block;`.

### Height and width

Contents fill the width of their container, then line wrap if thats their behavior. 

The width of a parent element determines the width of its children, but the height of a parent element is determined by the heights of its children.

## Building a layout

> When you are creating a layout, start with the larger elements, working 'outside in'. This lets you make sure your larger container elements are in the right place before you start working with smaller elements.

### Double-container pattern

Because containers are block elements and block elements expand to fill the widths of their containers, you have to restrict the size of the page's main container. You can do this with the _double-container pattern_.

This pattern uses two containers to restrict the width of the content in the inner container. The `body` is usually the outer container---you do not have to apply width to the body. So, you apply width and margins to the inner container:

```css
.container {
    max-width: var(--container-width);
    margin: 0 auto;
}
```

> Setting the left and right `margin` to `auto` centers the content because it makes the margins expand as much as necessary to fill the remaining width available in the outer container.
> This does not work for the top and bottom margin.

You can create this pattern using the logical properties:

```css
.container {
    max-width: var(--container-width);
    margin: auto;
}
```

## Logical properties

Logical properties are a way to write inline and block styles that are consistent when you apply it to other languages.



## Setting global box sizing

The following sets the box sizing on the entire document, while allowing you to set box sizing differently on specific components, if necessary:

```css
:root {
    box-sizing: border-box;
}

*,
*::before,
*::after {
    box-sizing: inherit;
}

.special-component {
    box-sizing: content-box;
}
```