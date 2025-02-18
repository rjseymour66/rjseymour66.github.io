---
title: "Document flow and layout"
# linkTitle: "CSS in Depth"
weight: 40
# description:
---

## Document flow

Normal document flow
: Default layout behavior of elements on the page - how inline and block elements behave on in the HTML document.

Inline elements
: Flow along with the text of the page and will line wrap if they reach the end of their container.
  - Ex: `<b>`, `<a>`, `<span>`
  - _inline-block_ elements use margin and padding for their height, but behave like inline elements.
  - _span_ is a generic inline element that you can use to group text and other inline elements.

Block elements
: Appear on their own lines and fill the width of the container. Have a line break before and below them. The user-agent stylesheet sets block elements to `display: block;`.
  - Ex: `<p>`, `<div>`, and `<header>`
  - _div_ is a generic block element that you can use as a container.


### Height and width

Contents fill the width of their container, then line wrap if thats their behavior. 
- The width of a parent element determines the width of its children
- The height of a parent element is determined by the heights of its children.


## Building a layout

When you are creating a layout, start with the larger elements, working 'outside in'. This lets you make sure your larger container elements are in the right place before you start working with smaller elements.

### Double-container pattern

Because containers are block elements and block elements expand to fill the widths of their containers, you have to restrict the size of the page's main container. You can do this with the _double-container pattern_.

This pattern uses two containers to restrict the width of the content in the inner container. The `body` is usually the outer container---you do not have to apply width to the body. So, you apply width and margins to the inner container.

Use a custom property for the width so you can apply it to other elements that use the double-container pattern:

```css
.container {
    max-width: var(--container-width);
    margin: 0 auto;
}
```
- `max-width` lets the elements shrink below this size on smaller screens but will never get larger than 1080px on larger viewports.
- `margin: 0 auto` is the easiest way to center content. It makes the left and right margins expand as much as necessary to fill the remaining width available in the outer container.
- This does not work for the top and bottom margin.

Here is the double-container using new [logical properties](#logical-properties):

```css
.container {
    max-inline-size: var(--container-width);
    margin-inline: auto;
}
```

### Double container with `min()`

You can also set up the double container with the `min()` function. The `min()` function always applies the smaller of the two values to the HTML. For example:

```css
.container {
  --max-width: 1110px;
  --side-padding: 1rem;

  width: min(var(--max-width), 100% - var(--side-padding) * 2)
}
```
This example sets the container width to either 1110px or 100% of the screen, with 1rem of padding on either side. On a large monitor, the container displays at 1110px. When the display is smaller than 1110px, it displays the other `min()` setting, which leaves 1rem of padding on either side.


## Logical properties

[CSS logical properties and values](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_logical_properties_and_values)

Logical properties are a way to write inline and block styles that are consistent when you apply it to other languages. In general:
- Replace `left` with `inline-start` and `right` with `inline-end`
- `top` with `block-start` and `bottom` with `block-end`

| Classic Name | Logical name |
|:-------------|:-------------|
| `width` | `inline-size` |
| `height` | `block-size` |
| `margin-top` | `margin-block-start` |
| `margin-bottom` | `margin-block-end` |
| `margin-left` | `margin-inline-start` |
| `margin-right` | `margin-inline-end` |
| `text-align: left` | `text-align: start` |
| `text-align: right` | `text-align: end` |
| `border-top-left-radius` | `border-start-start-radius` |
| `border-top-right-radius` | `border-start-end-radius` |
| `border-bottom-left-radius` | `border-end-start-radius` |
| `border-bottom-right-radius` | `border-end-end-radius` |

### New logical properties

There are some logical properties that make possible settings that were not possible with the classic naming convention:

| Logical name | Description |
|:---|:---|
| `margin-inline`  | Set `start` (`left`) and `end` (`right`) margin |
| `margin-block`   | Set `start` (`top`) and `end` (`bottom`) margin |
| `padding-inline` | Set `start` (`left`) and `end` (`right`) padding |
| `padding-block`  | Set `start` (`top`) and `end` (`bottom`) padding |
| `border-inline`  | Set `start` (`left`) and `end` (`right`) border |
| `border-block`   | Set `start` (`top`) and `end` (`bottom`) border |