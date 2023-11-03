---
title: "Grid"
weight: 60
description: >
  Working with CSS grid.
---

## Explicit and implicit grid

The `grid-template-{rows|columns}` settings explicitly define grid tracks to contain elements. If you add content and there is not a grid cell available, the grid implicitly defines new grid tracks to contain the new content. Similarly, if you define only the `grid-template-columns` and `gap` value, the grid will implicitly define rows based on the content within each block element.

The `grid-template-{rows|columns}` sizing that defines the explicit grid tracks does not apply to the implicit grid tracks. To define implicit grid sizing, use `grid-auto-{rows|columns}`:

```css
.container {
  display: grid;
  grid-template-columns: 50px 50px;
  grid-template-rows: 50px 50px;
  grid-auto-rows: 50px;
}
```

The previous ruleset says "create a layout with two columns that are 50px wide, and two rows that are 50px tall. If we add any content, add a new row to the grid that is 50px tall." The `auto` rows use the explicit column definition.

Because of the naturual document flow, you almost always want to define implicit rows, not columns. If you need to define implicit columns, apply `grid-auto-flow: column;` to tell the page to add implicit columns, then define `grid-auto-columns`:

```css
.container {
  display: grid;
  grid-template-columns: 50px 50px;
  grid-template-rows: 50px 50px;
  grid-auto-flow: column;
  grid-auto-columns: 50px;
}
```

`grid-template` is a shorthand for `grid-template-rows` and `grid-template-columns`:

```css
.container {
  display: grid;
  /* grid-template: rows / columns; */
  grid-template: 50px 50px / 50px 50px 50px;
}
```

## Gap

Grid gap is the space (gutter) between the rows and columns. Define it with `column-gap` or `row-gap`. You can use the shorthand property `gap: column-row column-gap;`



## Grid positioning

- [CSS Tricks Complete Guide to CSS Grid](https://css-tricks.com/snippets/css/complete-guide-grid/)
- [Inspect CSS grid layouts](https://developer.chrome.com/docs/devtools/css/grid/)

If you do not explicitly place items in the grid, the grid uses its auto-placement rules to add items to the first available cells in the grid.

### Grid lines

You can position items in a grid with grid lines. Grid lines are created implicitly when you define the grid tracks. When you define a grid with `grid-template-{row|column}`, you are defining the grid tracks. The grid track is the space between two grid lines.

Grid lines are numbered starting at 1 from left to right and top to bottom. If you have a 3x3 grid, there are 4 vertical grid lines because you have to count the border grid line. There are also negative lines that start at -1 and run opposite of the default line numbering. This numbering scheme addresses only the explicit grid---it does not take into account any rows or columns added with the implicit grid.

### Grid cells

A grid cell is the intersection of a row and a column---a (row, column) coordinate. To calculate the grid lines, think about an item in the middle of a 3x3 grid. an item in the middle lies in a row track 2 (between lines 2 and 3) and then column track 2 (between lines 2 and 3).

The following grid item properties let you position items in the grid using grid lines:

- `grid-column-start`
- `grid-column-end`
- `grid-row-start`
- `grid-row-end`

You can omit the `*-end` values if the item only spans one column or row, because that is the grid default. Otherwise, you can use these other shorthand versions:

- `grid-column: val / val` which is `grid column: <start> / <end>`
- `grid-row` which is `grid-row: <start> / <end>`

For example, the following rule uses both positive and negative grid line numbers to span the entire grid:

```css
.container {
  grid-column: 1 / -1;
}
```

### grid-area

Another shorthand property is `grid-area`. You can use grid lines to position elements, or you can use named grid template areas.

Here is an example of the grid lines:

```css
.area {
  grid-area: <row-start> / <col-start> / <row-end> / <col-end>;
}
```

Here is an example of template areas:

```css
.container {
  display: inline-grid;
  grid-template: 40px 40px 40px 40px 40px / 40px 40px 40px 40px 40px;
  background-color: lightblue;
  grid-template-areas:
    "living-room living-room living-room living-room living-room"
    "living-room living-room living-room living-room living-room"
    "bedroom bedroom bathroom kitchen kitchen"
    "bedroom bedroom bathroom kitchen kitchen"
    "closet closet bathroom kitchen kitchen";
}

#living-room {
  grid-area: living-room;
}
```

To indicate empty cells, use a `.` character:

```css
.container {
  display: inline-grid;
  grid-template: 40px 40px 40px 40px 40px / 40px 40px 40px 40px 40px;
  background-color: lightblue;
  grid-template-areas:
    "living-room living-room living-room living-room living-room"
    "living-room living-room living-room living-room living-room"
    "bedroom     bedroom     bathroom     kitchen     kitchen"
    "bedroom     bedroom     bathroom     kitchen     kitchen"
    "closet      closet      .            .           .";
}
```

### Gap

Add `column-gap` or `row-gap` to the grid container ruleset:

```css
.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, 100px);
  column-gap: 20px;
  row-gap: 1em;
}
```

The shorthand for these properties is `gap: <row-gap> <col-gap>`:

```css
.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, 100px);
  gap: 1em 20px;
}
```

### Spanning

The `span` keyword lets you specify the starting grid line and then the number of tracks that you want the grid area to span:

```css
.box1 {
  /* spans 1 col by default */
  grid-column: 1;
  /* spans from line 1 to 4 */
  grid-row: 1 / span 3;
}
```

## Dynamic rows and columns

Use `minmax()` and `clamp()` to make dynamically-sized rows and columns.

- `repeat(num, size)` to size rows or columns
- `fr` is fractional unit, a dynamic unit that distributes any space that is remaining on the grid.

You can use `repeat()` multiple times in the same rule:

```css
.container {
  ...
  grid-template-columns: repeat(2, 2fr) repeat(3, 1fr);
}
```
This creates 5 total columns, where the first two are twice the size of the last three.


### min() and max()

Use these functions to size the rows and columns:
```css
.grid-container {
  grid-template-rows: repeat(2, min(200px, 50%));
  grid-template-columns: repeat(5, max(120px, 15%));
}
```

When you set a `min()` size, you are essentially setting the maximum size. The previous row sizes are either 200px or 50% of the grid height, whichever value is the smallest. So, the rows are 50% of the grid size, at the most.

The `max()` function creates columns that are at least 120px or 15% of the grid's horizontal space.

### minmax()

`minmax()` is eaiser to reason with than `min()` and `max()` functions because the first value is the min, and the second is the max. You can only use it with the following properties:

- `grid-template-columns`
- `grid-template-rows`
- `grid-auto-columns`
- `grid-auto-rows`

The following ruleset creates columns that are 150px at the smallest and can dynamically grow up to 200px, at maximum:

```css
.grid-container {
  grid-template-rows: repeat(2, 1fr);
  grid-template-columns: repeat(5, minmax(150px, 200px));
}
```

### clamp()

`clamp()` lets you define an ideal size until the grid column or row hits the minimum or maximum thresholds:

```css
/* clamp(min, ideal, max) */

.grid-container {
  grid-template-columns: repeat(5, clamp(150px, 20%, 200px));
}
```

### autofit

`autofit` returns the highest positive integer without overflowing the grid. 


Lets the grid create the number of rows or columns based on the grid size. This helps create responsive grids. For example, the following grid has columns that are 150px when there is enough space for more than one, or the column is 1fr:

```css
.grid-container {
  width: 500px;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
}
```
When the previous rule is applied, the browser does the following:
1. The browser determines the highest number of grid items it can fit in the width, which is the minimum from the `minmax()` function.
2. After it creates the max number of columns (3 columns), it resizes the columns to their max, which is `1fr`.

### autofill

This behaves similarly to `autofit` until there are fewer items than can fill up the the grid row or column once:

```css
.grid-container {
  ...
  grid-template-rows: repeat(2, 1fr);
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
}
```

Where `autofit` uses the column's max size after it adds all columns, `autofill` uses the minimum size. The columns will grow to their max until space for a new column is available, then each column snaps back to its minimum size.