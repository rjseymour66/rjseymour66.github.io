+++
title = 'Document flow'
date = '2025-08-06T10:11:21-04:00'
weight = 10
draft = false
+++

## Definitions

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
  - `<div>` is a generic block element that you can use as a container.


## Building a layout

When you are creating a layout, start with the larger elements, and work from the outside inward. This ensures that your larger container elements are in the right place before you start working with smaller elements.

### Double-container pattern

Because containers are block elements and block elements expand to fill the widths of their containers, you have to restrict the size of the page's main container. You can do this with the _double-container pattern_.

This pattern uses an inner and outer container to restrict the width of the content in the inner container. Apply width and margin to the inner container. The `<body>` is usually the outer container---you do not have to apply width to the body.

{{< admonition "<body> container" tip >}}
If the container is the `<body>`, you can add width and margin to the `<body>` element. Make sure you add any background color to the `<html>` element so there is no white background past your `<body>` margins.
{{< /admonition >}}

The following examples implement the double container pattern with two divs:

```html
<div class="outer">
    <div class="inner">
        <!-- content -->
    </div>
</div>
```

The CSS lets the outer container expand to `800px` and the inner container expand to `300px`:

```css
.outer-container {
  --container-width: 800px;
  max-inline-size: var(--container-width);
}

.inner-container {
  --container-width: 300px;
  max-inline-size: var(--container-width);
  margin-inline: 0 auto;
}
```
- `max-inline-size` (or `max-width`) controls how large the element can grow on large viewports. On smaller screens, the element can shrink below this size. So, `inner-container` cannot get larger than `300px`.
- `margin-inline: 0 auto;` is the easiest way to center content. It makes the left and right margins expand as much as necessary to fill the remaining width available in the outer container. This does not work for the top and bottom margin.

### Double container with `min()`

You can also set up the double container with the `min()` function. The `min()` function always applies the smaller of the two values. For example:

```css
.container {
  --max-width: 1110px;
  --side-padding: 1rem;

  width: min(var(--max-width), 100% - var(--side-padding) * 2)
}
```
This sets the width to either `1110px` or `100% - var(--side-padding) * 2`, which is 100% width of the parent container minus `2rem` of padding (`1rem` on either side of the `.container` element). When the container is larger than 100% of its parent plus `2rem`, it applies `1110px` to the width. When the container is smaller than `1110px`, it displays the other `min()` setting, which leaves 1rem of padding on either side.

### Spacing container elements (lobotomized owl)

This pattern helps you manage margin for block or inline-block elements. It targets any element that immediately follows another element, so you don't have to create a ruleset for each unique element in the container.

`stack` is a class applied to a container that you want to vertically space all child elements. The child combinator (`>`)  limits the lobotmized owl to elements that exist in a container with `class="stack"`:

```css
.stack > * + * {
    margin-top: 1.5em;
}

/* psuedo-class equivalent */
.stack > :not(:first-child) {
    margin-top: 1.5em;
}
```

### Multi-column layout module

This module lets content flow naturally between multiple columns, much like how columns work in MS Word. There are two strategies to define the layout:
- `column-width`: The browser creates as many columnss of that width as possible.
- `column-count`: The browser creates equal-sized columns of the specified number of columns.

Here is the HTML with a list of 12 items and a block quote:

```html
<article class="columns">
    <ul>
        <li>1-list item</li>
        <li>2-list item</li>
        ...
        <li>12-list item</li>
    </ul>
    <blockquote>This is going to span all columns if I apply <code>column-span: all;</code> to the blockquote.</blockquote>
</article>
```

Here is an example using `column-count`:

```css
.columns {
  column-count: 4;
  column-rule: 3px solid $grey-500;
  column-gap: 2rem;
}
```

Here is an example using `column-width`:

```css
.columns {
  column-width: 12rem;
  column-rule: 3px solid $grey-500;
  column-gap: 2rem;
}
```

If you want an element to span multiple columns, use the `column-span` property. It accepts `all` or `none`:

```css
.columns > blockquote {
    column-span: all;
}
```

## Logical properties

[CSS logical properties and values (MDN)](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_logical_properties_and_values)

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

### Margin and padding shorthand

| Logical name     | Description                                      |
| :--------------- | :----------------------------------------------- |
| `margin-inline`  | Set `start` (`left`) and `end` (`right`) margin  |
| `margin-block`   | Set `start` (`top`) and `end` (`bottom`) margin  |
| `padding-inline` | Set `start` (`left`) and `end` (`right`) padding |
| `padding-block`  | Set `start` (`top`) and `end` (`bottom`) padding |
| `border-inline`  | Set `start` (`left`) and `end` (`right`) border  |
| `border-block`   | Set `start` (`top`) and `end` (`bottom`) border  |


## Width and height

The main prinicples of width and height are as follows:
- The width of a parent element determines the width of its children. Contents fill the width of their container, then line wrap if thats their behavior.
- The height of a parent element is determined by the heights of its children. The height of a block-size container is determined by the height of its contents.

### Overflow

It is typically bad practice to add heights to block-sized containers. The normal document flow has a constrained width and unlimited height---width is determined by device size, but you can scroll on a web page forever.

You don't want to set the height because then you have to deal with _overflow_. Overflow happens when the container is not large enough for all content, so it renders outside the container. If you get overflow, control it with the following properties:
- `visible`: Default. You can see overflowing content.
- `hidden`: Overflowing content is clipped, and you have to add JS to enable scrolling.
- `clip`: Overflowing content is clipped, and you cannot add scrolling with JS.
- `scroll`: Adds scrollbars to the container, even if the content doees not overflow.
- `auto`: Adds scrollbars only when there is overflow.

{{< admonition "" tip >}}
Generally, you should prefer `auto`.
{{< /admonition >}}

You can control horizontal overflow with `overflow-x`, and vertical overflow with `overflow-y`. These properties support the same values as `overflow`.

### line-height

Use `line-height` to change the height that an inline element contributes to its container. To make padding and margin affect its container, the element has to be a block element. To change an inline element to a block element, use `display: inline-block;`.

### Percentage-based heights

Never use a percentage for heights. The browser ignores percentage rules set on heights because it cannot compute the value. When the browser calculates an element's height, it looks to that element's container to calculate the percentage. However, the height of a container is determined by the height of its contents, resulting in a circular definition that the browser cannot resolve.


{{< admonition "Adding height to containers" tip >}}
- To make a container fill the screen, use `100vh` or `100svh`.
- To create columns of equal height, use [flexbox](./flexbox.md) or [grid](./grid.md).
{{< /admonition >}}

### min-height and max-height

{{< admonition "Sizing sibling elements" tip >}}
If you want to size sibling elements equally, you probably want to use flexbox.
{{< /admonition >}}

> 

`min-height` and `max-height` are useful if you need an element to be at least or at most a specific size. Use relative units like `em` or `rem`.

If you set `min-height` on a container, the container will be at least that height, and grows larger when the screen size allows. When the screen is large enough, the browser fills the container with any content as the container height grows. `min-height` is more useful because it doesn't result in overflow problems.

`max-height` allows the container to grow to the specified size, then its contents overflow.

## Margins

### Negative margins

Negative margins make elements overlap or stretch wider than their containers. Negative margin moves the element to that side. For example, if you set `margin-left: -10rem;`, the element will move 10rem to the left.

{{< admonition "Placing elements" tip >}}
Avoid placing elements with negative margins as they might affect event listeners on interactive elements.

Instead, use the `position` property when placing elements.
{{< /admonition >}}

You can calculate negative margin with the `calc()` function. This ruleset places an 1/3 of an image above its containing block using a negative `margin-top` and `calc()`:

```css
img.portrait {
  ...
  margin-top: calc(-1 * var(--image-size) / 3);
}
```

### Collapsed margins

Top and bottom margins combine to form a single margin. This is called _collapsing_. By default, the user agent stylesheet adds top and bottom margin to block elements. For example, it adds `1em` top and bottom for `<p>` elements.

{{< admonition "Note" note >}}
The size of the collapsed margin is the size of the largest of the two joined margins.
{{< /admonition >}}

Multiple margins can collapse. in the following example, the bottom of the `h2`, top of the `div` and top of the `p` elements all collapse:

```html
<h2>Heading</h2>
<div>
    <p>Collapse</p>
</div>
```

### Troubleshooting collapsed margins

{{< admonition "Padding" tip >}}
If you have an issue with whitespace above or below an element, remove the margin and add padding to define the space you need.
{{< /admonition >}}

Margins don't collapse outside a container. In other words, if you have top margin set on a child element, it applies that margin outside the container. This can cause issues if there is a background color or image.

You can solve by this in the following ways:
- Add padding to the child element and set top and bottom margin to 0. This lets the padding define the space between the elements.
- Apply `overflow: auto` (or any value other than `visible`) to the container to prevent margins inside the container from collapsing with those outside the container. This is often the least intrusive solution.
- To prevent two margins from collapsing, add a border or padding between them.

Margins do not collapse under the following circumstances:
- If the container is inline-block, floated, or has an absolute or fixed position.
- When the elements are part of a flex or grid layout.
- Table-cell elements don’t have a margin, so they won’t collapse. This also applies to table-row and most other table display types. Exceptions are table, table-inline, and table-caption.