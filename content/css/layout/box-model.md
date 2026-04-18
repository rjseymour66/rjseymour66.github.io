+++
title = 'Box Model'
date = '2025-08-06T10:11:27-04:00'
weight = 20
draft = false
+++

The box model refers to the parts of an element and the size they contribute to the element size. Each element on the page is made of four overlapping rectangles:
- **content area**: innermost rectangle where the contents of the element reside.
- **padding area**: content area plus any padding. Top and bottom padding on inline elements does not contribute to the height of the container element.
- **border area**: padding area plus any border.
- **margin area**: outermost rectangle that contains the border area plus any margins. Top and bottom margin on inline elements does not contribute to the height of the container element.

{{< admonition "`outline` property" note >}}
The `outline` property is not part of the box model, so it does not contribute to the element's size. It is placed outside the border and overlaps the margin.
{{< /admonition >}}

## Universal border box

By default, the box model uses `box-sizing: content-box;` for each element. This means that when you set the width of an element, it sets the width on the content area only. Any padding, border, or margin are _added_ to the content width, so elements with `content-box` are much larger than `border-box`.

You should prefer `box-sizing: border-box;` setting for each element to make its size more predictable. This combines the content, padding, and border areas to determine the size of the element. Margin is still outside the box.

To apply `border-box` sizing to all elements, including pseudo-elements, place the following ruleset to the top of your stylesheet :

```css
*,
*::before,
*::after {
  box-sizing: border-box;
}
```

## Setting global box sizing

You can set `border-box` on the entire document and still use `content-box` on specific components. Set the root element to `border-box` and inherit that value with the universal border box sizing pattern. Then, you can override the box sizing on other elements as needed:

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