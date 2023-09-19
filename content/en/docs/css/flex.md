---
title: "Flexbox"
linkTitle: "Flexbox"
weight: 3
description: >
  Notes about basic Flexbox and Flex properties.
---

## Flex

Flexbox is short for Flexible Box Layout, and can help you arrange elements in a row or column, vertically center items, and make elements equal height. 


Apply `display: flex` to an element to turn it into a _flex container_, and all of its children become _flex items_. By default, flex items are the same height, and the height is determined by their content.

> If you use `display: inline-flex`, all elements become flex items, but they are inline elements instead of block.
>
> You will rarely use `inline-flex`.

## Flex items

When arranged in a row, flex items are placed horizontally along the _main axis_, and vertically along the _cross-axis_. Column layouts have a vertical main axis and horizontal cross axis.

### Size

The `flex` property controls the size of the flex items along the main axis.

By default, flex is set to `flex: 0 1 auto`. It consists of three properties: `flex-grow`, `flex-shrink`, and `flex-basis`:
- `flex-basis`: Defines a starting size of an element. Set this like you would set a width, with px, ems, percentages.
  
  By default, this is set to `auto`. When set to `auto`, the browser checks if there is a `width` declared on the element. If there is a `width`, the browser uses the `width` setting. If there is no `width`, the browser determines the size by the item's contents. If you have a `width` set, but `flex-basis` is set to anything other than `auto`, then the browser ignores the `width` setting.
- `flex-grow`: When the browser calculates the `flex-basis`, it does not necessarily fill the width of the flex container. Any remaining space is consumed according to the `flex-grow` setting on each flex item.
  
  If `flex-grow` is set to 0, the item will not grow beyond its `flex-basis` value. If it is set to any number other than 0, then the items grow until the space is used.

  The `flex-grow` value acts as a weight. For example, if you set item A to `1` and item B to `2`, then item B will take up twice as much of the remaining space as item A.
- `flex-shrink`: Indicates whether it should shrink to prevent overflow in the event that the flex items' initial size is larger than the flex container.
  
  `flex-shrink: 0;` means that the item does not shrink. Items with a higher value shrink more than items with a lower value.

The shorthand `flex` property is usually all you need. For example, the following styles creates two flex items, one twice as gig as the other, with a gap:

```css
.flex {
    display: flex;
    gap: var(--gap-size);
}

.column-main {
    flex: 2;
}

.column-sidebar {
    flex: 1;
}
```
## Old notes

Flexbox allows you to define 1-D layouts. It works from the content out. Use it for rows or columns of similar elements. You don't need to set the size, because that is determined by the content.





Flex items align side by side, left to right, in a row. Use `flex-direction` to change this.

A flex container takes up 100% of its available width, while the height is determined natually by its contents. The `line-height` of the text inside each flex item is what determines the height of each item.



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