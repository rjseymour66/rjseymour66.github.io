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

| Classic Name                 | Logical name                |
| :--------------------------- | :-------------------------- |
| `width`                      | `inline-size`               |
| `height`                     | `block-size`                |
| `margin-top`                 | `margin-block-start`        |
| `margin-bottom`              | `margin-block-end`          |
| `margin-left`                | `margin-inline-start`       |
| `margin-right`               | `margin-inline-end`         |
| `text-align: left`           | `text-align: start`         |
| `text-align: right`          | `text-align: end`           |
| `border-top-left-radius`     | `border-start-start-radius` |
| `border-top-right-radius`    | `border-start-end-radius`   |
| `border-bottom-left-radius`  | `border-end-start-radius`   |
| `border-bottom-right-radius` | `border-end-end-radius`     |

### New logical properties

There are some logical properties that make possible settings that were not possible with the classic naming convention:

| Logical name     | Description                                      |
| :--------------- | :----------------------------------------------- |
| `margin-inline`  | Set `start` (`left`) and `end` (`right`) margin  |
| `margin-block`   | Set `start` (`top`) and `end` (`bottom`) margin  |
| `padding-inline` | Set `start` (`left`) and `end` (`right`) padding |
| `padding-block`  | Set `start` (`top`) and `end` (`bottom`) padding |
| `border-inline`  | Set `start` (`left`) and `end` (`right`) border  |
| `border-block`   | Set `start` (`top`) and `end` (`bottom`) border  |


## Element height

The height of a block-size container is determined by the height of its contents.

It is typically bad practice to add heights to block-sized containers. The normal document flow has a constrained width and unlimited height---width is determined by device size, but you can scroll on a web page forever.

You don't want to set the height because then you have to deal with _overflow_. Overflow happens when the container is not large enough for all content, so it renders outside the container. If you get overflow, control it with the following properties:
- `visible`: Default. You can see overflowing content.
- `hidden`: Overflowing content is clipped, and you have to add JS to enable scrolling.
- `clip`: Overflowing content is clipped, and you cannot add scrolling with JS.
- `scroll`: Adds scrollbars to the container, even if the content doees not overflow.
- `auto`: Adds scrollbars only when there is overflow.

> Generally, you should prefer `auto`.

You can control horizontal overflow with `overflow-x`, and vertical overflow with `overflow-y`. These properties support the same values as `overflow`.

Use `line-height` to change the height that an inline element contributes to its container. To make padding and margin affect its container, the element has to be a block element. To change an inline element to a block element, use `display: inline-block;`.

### Percentage-based heights

Don't use percents for heights of elements in the normal doc flow---when you use percents for heights, the browser looks to that element's container to determine calculate the percentage. The height of that container is determined by the height of its children, which is a circular definition. The browser can't resolve this, and it ignores percentage rules.

Common mistakes when trying to add height to a container with percentages:
- Making a container fill the screen by setting `height: 100%` on an element and its ancestors. Instead, use `height: 100vh`.
- Creating columns of equal height. Use flexbox or grid.


### min-height and max-height

> If you want to size sibling elements equally, you probably want to use flexbox.

`min-height` and `max-height` are useful if you need an element to be at least or at most a specific size. Use relative units like `em` or `rem`.

If you set `min-height` on a container, the container will be at least that height, and it can grow larger when the screen size allows. When the screen is large enough, the browser fills the container with any content as the container height grows. `min-height` is more useful because it doesn't result in overflow problems.

`max-height` allows the container to grow to the specified size, then its contents overflow.

## Negative margins

Negative margins make elements overlap or stretch wider than their containers. Negative margin moves the element to that side. For example, if you set `margin-left: -10rem;`, the element will move 10rem to the left.

> Try to avoid negative margins as they might affect event listeners on interactive elements.
> 
> Prefer the `position` property when placing elements.

## Collapsed margins

Top and bottom margins combine to form a single margin. This is called _collapsing_. By default, the user agent stylesheet adds top and bottom margin to block elements. For example, 1em top and bottom for `p` elements.

**The size of the collapsed margin is the size of the largest of the two joined margins.**

Multiple margins can collapse. in the following example, the bottom of the `h2`, top of the `div` and top of the `p` elements all collapse:

```html
<h2>Heading</h2>
    <div>
        <p>Collapse</p>
    </div>
```

#### Problems with collapsing margins

Margins don't collapse outside a container. If you have an issue trying to get rid of whitespace above or below an element, **remove the margin and add padding to define the space you need.**

When a container element has a background and no top or bottom margin, and it contains a child element that has margins, the margins collapse to the outside---this looks like the container uses the child elements margin. When there is a background, this makes it look strange.

You can solve by this in the following ways:
- **Add padding to the child element and set top and bottom margin to 0. This lets the padding define the space between the elements.**
- Applying `overflow: auto` (or any value other than visible) to the container prevents margins inside the container from collapsing with those outside the container. This is often the least intrusive solution.
- Adding a border or padding between two margins stops them from collapsing.
- Margins won’t collapse to the outside of a container that is an inline-block, that is floated, or that has an absolute or fixed position.
- When using a flexbox or grid layout, margins won’t collapse between elements that are part of the flex layout.
- Elements with a table-cell display don’t have a margin, so they won’t collapse. This also applies to table-row and most other table display types. Exceptions are table, table-inline, and table-caption.

## Spacing container elements (lobotomized owl)

This pattern helps you manage margin for block or inline-block elements. It targets any element that immediately follows another element, so you don't have to create a ruleset for each unique element in the container.

`stack` is a class applied to a container that you want to vertically space all child elements. The child combinator (`>`)  limits the lobotmized owl to elements that exist in a container with `class="stack"`:

```css
.stack > * + * {
    margin-top: 1.5em;
}

/* equivalent to this */
.stack > :not(:first-child) {
    margin-top: 1.5em;
}
```

## Images

To change the size of an image without changing its proportions, set the `width` to the size you want, and set the `height` to `auto`:

```css
img {
  height: auto;
  width: val;
}
```

Setting the height and width on an image also helps the browser calculate the image size while it loads the other content. This might prevent strange rendering behavior during page loads.

### object-position

Works with `object-fit`, which tells the browser to calculate the optimum size of the image based on the dimensions provided so that it does not distort. `object-position` changes where the image is positioned inside the container to manipulate which part of the image is clipped.