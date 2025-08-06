---
title: "Box Model"
linkTitle: "xBox Model"
weight: 50
# description:
---

## Box model 

The box model refers to the parts of an element and the size they contribute to the element size. Each element on the page is made of four overlapping rectangles:
- **content area**: innermost rectangle where the contents of the element reside.
- **padding area**: content area plus any padding. Top and bottom padding on inline elements does not contribute to the height of the container element.
- **border area**: padding area plus any border.
- **margin area**: outermost rectangle that contains the border area plus any margins. Top and bottom margin on inline elements does not contribute to the height of the container element.

> The `outline` property is not part of the box model, so it does not contribute to the element's size. It is placed outside the border and overlaps the margin.


### Universal border box sizing

By default, the box model uses `box-sizing: content-box;` for each element. This means that when you set the width of an element, it sets the width on the content area only. Any padding, border, or margin are _added_ to the content width, so elements with `content-box` are much larger than `border-box`.

Prefer the `box-sizing: border-box;` setting for each element. This combines the content, padding, and border areas to determine the size of the element. Margin is still outside the box.

Place the following ruleset to the top of your stylesheet:

```css
*,
::before,
::after {
  box-sizing: border-box;
}
```

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