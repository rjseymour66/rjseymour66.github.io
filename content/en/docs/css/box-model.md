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


https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_logical_properties_and_values

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

## Box model 

Each element on the page is made of four overlapping rectangles:
- content area: innermost rectangle where the contents of the element reside.
- padding area: content area plus any padding. 
- border area: padding area plus any border.
- margin area: outermost rectangle that contains the border area plus any margins.

> The `outline` property is not part of the box model, so it does not contribute to the element's size. It is placed outside the border and overlaps the margin.

By default, the box model uses `box-sizing: content-box;` for each element. This means that when you set the width of an element, it sets the width on the content area only. Any padding, border, or margin are _added_ to the content width.

Prefer the `box-sizing: border-box;` setting for each element. This combines the content, padding, and border areas to determine the size of the element. Margin is still outside the box. Place the following ruleset to the top of your stylesheet:

```css
*,
::before,
::after {
  box-sizing: border-box;
}
```



## Padding and margin

### Inline elements

Use `line-height` to change the height that an inline element contributes to its container.

### Block elements 

To make padding and margin affect its container, the element has to be a block element. To change an inline element to a block element, use `display: inline-block;`.

### Negative margins 

Negative margins make elements overlap or stretch wider than their containers.

> Try to avoid negative margins as they might affect event listeners on interactive elements.
> 
> Prefer the `position` property when placing elements.

Negative right margins make block elements wider on the right side. 


### Collapsed margins

Top and bottom margins combine to form a single margin. This is called _collapsing_. By default, the user agent stylesheet adds top and bottom margin to block elements. For example, 1em top and bottom for `p` elements.

The size of the collapsed margin is the size of the largest of the two joined margins.

#### Problems with collapsing margins 

When a container element has a background and no top or bottom margin, and it contains a child element that has margins, the margins collapse to the outside---this looks like the container uses the child elements margin. When there is a background, this makes it look strange.

You can solve by this in the following ways:
- Add padding to the child element and set top and bottom margin to 0. This lets the padding define the space between the elements.
- Applying `overflow: auto` (or any value other than visible) to the container prevents margins inside the container from collapsing with those outside the container. This is often the least intrusive solution.
- Adding a border or padding between two margins stops them from collapsing.
- Margins won’t collapse to the outside of a container that is an inline-block, that is floated (chapter 12), or that has an absolute or fixed position (chapter 6).
- When using a flexbox or grid layout, margins won’t collapse between elements that are part of the flex layout (chapters 4 and 5).
- Elements with a table-cell display don’t have a margin, so they won’t collapse. This also applies to table-row and most other table display types. Exceptions are table, table-inline, and table-caption.

## Container heights

It is typically bad practice to add heights to block-sized containers. The normal document flow has a constrained width and unlimited height---width is determined by device size, but you can scroll on a web page forever. The height of a block-size container is determined by the height of its contents. 

You don't want to set the height because then you have to deal with _overflow_. Overflow happens when the container is not large enough for all content, so it renders outside the container. If you get overflow, control it with the following properties:
- `visible`: Default. You can see overflowing content.
- `hidden`: Overflowing content is clipped, and you have to add JS to enable scrolling.
- `clip`: Overflowing content is clipped, and you cannot add scrolling with JS.
- `scroll`: Adds scrollbars to the container, even if the content doees not overflow.
- `auto`: Adds scrollbars only when there is overflow.

> Generally, you should prefer `auto`.

### Percentage-based heights 

Don't use percents for heights of elements in the normal doc flow---when you use percents for heights, the browser looks to that element's container to determine calculate the percentage. The height of that container is determined by the height of its children, which is a circular definition. The browser can't resolve this, and it ignores percentage rules.

### min- and max-height 

> If you want to size sibling elements equally, you probably want to use flexbox.

`min-height` and `max-height` are useful if you need an element to be at least or at most a specific size. If you set `min-height` on a container, the container will be at least that height, and it can grow larger when the screen size allows. When the screen is large enough, the browser fills the container with any content as the container height grows.

`max-height` allows the container to grow to the specified size, then its contents overflow.

### Spacing elements in a container 

When you need to evenly space elements in a container, you need to make sure that the padding of the container and child element margins interact properly. You can set margins on elements that share a class with the `+` adjacent sibling combinator, but you have to add a ruleset for each unique element in the container. The solution is the _lobotomized owl_ rule:

```css
.stack > * + * {
  margin-block-start: 1.5em;
  /* margin-top: 1.5em; */
}
```

This selector says, _any element with class `stack`, apply `1.5em` top margin to any block element that immediately follows another element. This is equivalent to the following:

```css
.stack > :not(:first-child) {
  margin-block-start: 1.5em;
}
```