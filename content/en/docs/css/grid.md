---
title: "Grid"
linkTitle: "Grid"
weight: 4
description: >
  Notes about basic Grid and Grid properties.
---

Grid lets you create 2-D layouts. It works from the layout in.

Grid containers behave like block elements, it fills 100% of the available width.

You can use the `fr` unit, which stands for `fraction unit`. This is analogous to `flex-grow` in flexbox.

`grid-gap` defines the amount of space to add to the gutter between each grid cell. If you define this, it lies atop the grid lines.

### Terminology

- grid container: The element that you apply `display: grid` to
- grid item: A child of the grid container.
- grid line: The line between the columns and rows.
- grid cell: Area between grid lines.
- grid track: The space between two adjacent grid lines.
- grid area: The total space surrounded by four grid lines.

### Grid container properties

Apply the following properties to the grid container:

```css
.container {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    grid-template-rows: min-content 1fr min-content;
    grid-template-areas:
        ". header     header    ."
        ". nav        nav       ."
        ". content    content   ."
        ". footer     footer    .";
    /* shorthand for *-columns, *-rows, and *-areas */
    grid-template: 
        [row1-start] "header header header" 25px [row1-end]
        [row2-start] "footer footer footer" 25px [row2-end];
    column-gap: 2rem;
    row-gap: 2rem;
    /* shorthand for row-gap and column-gap*/
    gap: 2rem 2rem;
    justify-items: start;
    align-items: center;
    /* align-items/justify-items */
    place-items: center;
    justify-content: space-around;
    align-content: end;
    /* shorthand for align-content and justify-content props */
    place-content: end space-around;
    /* size of any implicit grid tracks */
    grid-auto-columns: 60px 60px;
    /* places items in the grid automatically */
    grid-auto-flow: column;
    grid: 100px 300px / auto-flow 200px;
}
```

### Grid item properties

Apply the following properties to the grid items:

```css
.grid-item {
    grid-column-start: <number> | <name> | span <number> | span <name> | auto;
    grid-column-end: <number> | <name> | span <number> | span <name> | auto;
    grid-row-start: <number> | <name> | span <number> | span <name> | auto;
    grid-row-end: <number> | <name> | span <number> | span <name> | auto;
    /* shortnad for previous props */
    grid-column: <start-line> / <end-line> | <start-line> / span <value>;
    grid-row: <start-line> / <end-line> | <start-line> / span <value>;
    grid-area: header;
    justify-self: start;
    align-self: center;
    place-self: stretch;
}
```

### Sizing functions and keywords

- `fr`: fractional unit
- `min-content`: minimum size of the content.
- `max-content`: maximum size of the content.
- `minmax(min, max)`: sets the min and max for the element.
- `repeat(num, size)`: creates `num` rows or columns of `size` width

### Grid areas

If you don't want to count grid lines to place items, use grid areas. You apply `grid-template` on the grid container, and then the `grid-area` property on the child elements:

```css
.container {
    display: grid;
    grid-template-areas:
        "title  title"
        "nav    nav"
        "main   aside1"
        "main   aside2";
    grid-template-columns: 2fr 1fr;
    grid-template-rows: repeat(4, auto);
    grid-gap: 1.5em;
    max-width: 1080px;
    margin: 0 auto;
}

header {
    grid-area: title;
}

nav {
    grid-area: nav;
}
...
```

### Implicit grid

Implicit grid tracks have a size of `auto`, which means that they grow to the size of the grid item contents. Use `grid-auto-rows` or `grid-auto-columns` to set a size for implicit grid items.