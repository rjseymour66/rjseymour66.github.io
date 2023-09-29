---
title: "Grid"
linkTitle: "Grid"
weight: 4
description: >
  Notes about basic Grid and Grid properties.
---

Grid lets you create 2-D layouts. It works from the layout in.

Like flexbox, grid has two levels that create a hierarchy:
- grid container
- grid items

Grid containers behave like block elements, it fills 100% of the available width.

## Flexbox vs Grid

Flexbox is one-dimensional---you cannot align items in one row with the next. The size of flexbox items is determined by their contents. In other words, flexbox works from the content out, meaning that you can arrange flex items in a row or column, but you do not have to explicitly set their size.

Use flexbox for things like navbars.

Grids is two-dimensional---you can align items vertically and horizontally. Grid works from the layout in. You begin by defining a layout and placing items within that structure. If a grid item's contents grow beyond its grid track, then it affects other items in the grid.

### When to use what?

If you need to align items in two dimensions, use grid.

If you need to align items in one direction (like navs), use flexbox.

## Columns and rows

Define the column and rows in the grid container definition. You can create a layout with the following properties:
- `grid-template-columns`
- `grid-template-rows`

For example, the following ruleset defines three rows and columns of equal width and height:

```css
.grid {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  grid-template-rows: 1fr 1fr;
  gap: 0.5em;
}
```

### `fr`

The `fr` unit, which stands for `fraction unit`, is analogous to `flex-grow` in flexbox. You can mix and match `fr` units with other units, such as `px`.

### `auto`

To create columns or rows that grow according to their contents, use the `auto` value. For example, the following ruleset creates four columns that are sized based on their contents:

```css
.grid {
  grid-template-columns: repeat(4, auto);
}
```

### `gap`

`gap` defines the amount of space to add to the gutter between each grid cell. You can provide two values to `gap`, where the first is row gap and the second is column gap.


### `repeat()`

You could also define the rows and columns with the `repeat()` function:

```css
.grid {
  ...
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(2, 1fr);
  ...
}
```

### `span`



### Terminology

- grid container: The element that you apply `display: grid` to
- grid item: A child of the grid container.
- grid line: The line between the columns and rows.
- grid cell: Single space on the grid where horizontal and vertical tracks overlap.
- grid track: The space between two adjacent grid lines. This is a different way to say _row_ or _column_. A horizontal track is a row, and vertical is column.
- grid area: The total space surrounded by four grid lines---a rectangle of one or more grid cells.

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

### Grid lines

You can place grid items in the grid container using the grid lines and the following properties:
- `grid-column`
- `grid-row`

If you have a grid that has 4 columns, and you want a grid item to span the first three columns, apply the following rule:
```css
.grid-item {
    grid-column: 1 / 4;
}
```

### Grid areas

If you don't want to count grid lines to place items, use grid areas. You apply `grid-template` on the grid container, and then assign the area with the `grid-area` property on the child elements:

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
To leave a cell empty, use a period. The following example creates an empty cell in the center:

```css
.grid-container {
    grid-template-areas:
        "title tile right"
        "left  .    left"
        "footer footer footer";
}
```

### Implicit grid

Implicit grid tracks have a size of `auto`, which means that they grow to the size of the grid item contents. Use `grid-auto-rows` or `grid-auto-columns` to set a size for implicit grid items.