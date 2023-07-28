---
title: "Flexbox"
linkTitle: "Flexbox"
weight: 3
description: >
  Notes about basic Flexbox and Flex properties.
---

## Flex

Flexbox allows you to define 1-D layouts. It works from the content out. Use it for rows or columns of similar elements. You don't need to set the size, because that is determined by the content.

Applying `display: flex` to an element turns it into a _flex container_, and all of its children become _flex items_. By default, flex items are the same height.

Flex items align side by side, left to right, in a row. Use `flex-direction` to change this.

A flex container takes up 100% of its available width, while the height is determined natually by its contents. The `line-height` of the text inside each flex item is what determines the height of each item.

Flex items are placed horizontally along the _main axis_, and vertically along the _cross-axis_.

Margins between flex items do not collapse like in the regular document flow.

Styling with flexbox involves the following:
- Identify the flex container and apply `flex: display;`
- Set `flex-direction`, if necessary.
- Declare margins or `flex` values for the flex items to control their sizes.

### Flex container properties

Apply the following properties to the flex container:

```css
.flex-container {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    /* flex-flow is shorthand for flex-direction and flex-wrap */
    flex-flow: column wrap;
    /* justify-content uses the main axis. Think about setting text. */
    justify-content: space-between;
    /* align-* props use the cross-axis */
    align-items: flex-start;
    align-content: stretch;
    gap: 1rem;
    row-gap: 1rem;
    column-gap: 1rem;
}
```

### Flex item properties

Apply the following properties to the flex container:

```css
.flex-item {
    order: 3;
    flex-grow: 1;
    flex-shrink: 2;
    flex-basis: auto;
    flex: 1;
    /* uses cross-axis */
    align-self: flex-start;
}

```
### flex: property
Use the `flex` property to to allow flex items to grow to fill its container size, and to size flex items relative to each other:
```css
.column-one {
    flex: 1;
}

.column-two {
    flex: 2;
}
```

In the previous example, `column-one` is 1/3 the size of the screen width, and `column-two` is 2/3 its size.

`flex` property is short for the following:
- `flex-grow`: Set to `0` or `1`:
  - `0`: The flex item will not grow past its `flex-basis`
  - `1` or non-zero: The flex item grows until all of the remaining space is used up. The item takes up more space relative to the `flex-grow` value set on other items. For example, setting to `2` makes it consume twice the space that a sibling element set to `1` consumes.
- `flex-shrink`: Default: 1. Determines if the element shrinks to prevent overflow. Set to `0` to prevent the item from shrinking.
- `flex-basis`: Default: 0%. Sets the starting point for the size of the element before the remaining space is distributed. The initial size is `auto`, which means it checks for a declared width and uses that, or it uses the element's contents for sizing.

#### Holy Grail layout:

The first and third columns have a fixed width of 200px, and the center column grows to fill all the remaining space:

 ```css
.column-one {
    flex: 0 0 200px;
}

.column-two {
    flex: 1;
}

.column-three {
    flex: 0 0 200px;
}
```

### Flex direction

Swaps the direction of the main-axis and the cross-axis. 

For `flex-direction` is a row, the `flex` property applies to the width of the container. When `flex-direction` is column, the `flex` property applies to the height of the container.