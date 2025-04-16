---
title: "Grid"
# linkTitle: "CSS in Depth"
weight: 80
# description:
---

Officially titled the Grid Layout Module.

Grid lets you create 2-D layouts of columns and rows. It works from the layout in. If you make an element a grid, it creates a grid row for each child element until you explicitly place the elements in a different grid structure.

Like flexbox, grid has two levels that create a hierarchy:

- grid container
- grid items

Grid containers behave like block elements, it fills 100% of the available width.

## Flexbox vs Grid

- If you need to align items in two dimensions, use grid.
- If you need to align items in one direction (like navs), use flexbox.

Flexbox is one-dimensional---you cannot align items in one row with the next. The size of flexbox items is determined by their contents. In other words, flexbox works from the content out, meaning that you can arrange flex items in a row or column, but you do not have to explicitly set their size.

Use flexbox for things like navbars.

Grids is two-dimensional---you can align items vertically and horizontally. Grid works from the layout in. You begin by defining a layout and placing items within that structure. If a grid item's contents grow beyond its grid track, then it affects other items in the grid.

## Terminology

Grid container
: The element that you apply `display: grid` to

Grid item
: A child of the grid container.

Grid line
: The line between the columns and rows.

Grid cell
: Single space on the grid where horizontal and vertical tracks overlap.

Grid track
: The space between two adjacent grid lines. This is a different way to say _row_ or _column_. A horizontal track is a row,
and vertical is column.
Grid area
: The total space surrounded by four grid lines---a rectangle of one or more grid cells.

## Grid container properties

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
  grid-template:                    /* shorthand for *-columns, *-rows, and *-areas */
    [row1-start] "header header header" 25px [row1-end]
    [row2-start] "footer footer footer" 25px [row2-end];
  column-gap: 2rem;
  row-gap: 2rem;
  gap: 2rem 2rem; /* shorthand for row-gap and column-gap*/
  justify-items: start;
  align-items: center; /* align-items/justify-items */
  place-items: center;
  justify-content: space-around;
  align-content: end;
  place-content: end space-around; /* shorthand for align-content and justify-content props */
  grid-auto-columns: 60px 60px; /* size of any implicit grid tracks */
  grid-auto-flow: column; /* places items in the grid automatically */
  grid: 100px 300px / auto-flow 200px;
}
```

## Grid item properties

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

## Grid positioning

- [CSS Tricks Complete Guide to CSS Grid](https://css-tricks.com/snippets/css/complete-guide-grid/)
- [Inspect CSS grid layouts](https://developer.chrome.com/docs/devtools/css/grid/)

If you do not explicitly place items in the grid, the grid uses its auto-placement rules to add items to the first available cells in the grid.

### Grid lines

You can position items in a grid with grid lines. Grid lines are created implicitly when you define the grid tracks. When you define a grid with `grid-template-{row|column}`, you are defining the grid tracks. The grid track is the space between two grid lines.

Grid lines are numbered starting at 1 from left to right and top to bottom. If you have a 3x3 grid, there are 4 vertical grid lines because you have to count the border grid line. There are also negative lines that start at -1 and run opposite of the default line numbering. This numbering scheme addresses only the explicit grid---it does not take into account any rows or columns added with the implicit grid.

You can place grid items in the grid container using the grid lines and the following properties:

- `grid-column`
- `grid-row`

If you have a grid that has 4 columns, and you want a grid item to span the first three columns, apply the following rule:

```css
.grid-item {
  grid-column: 1 / 4;
}
```

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

### Grid area

If you don't want to count grid lines to place items, use grid areas. You apply `grid-template` on the grid container, and then assign the area with the `grid-area` property on the child elements:

```css
main {
  display: grid;
  grid-template-columns: repeat(2, minmax(auto, 1fr)) 250px;
  grid-template-areas:
    "header header header"
    "content content author"
    "content content aside"
    "plays . aside"
    "footer footer footer";
  gap: 20px;
}

@media (min-width: 955px) {
  main {
    grid-template-columns: repeat(3, 1fr);
    grid-template-areas:
      "header header header"
      "content author aside"
      "content plays aside"
      "footer footer footer";
    gap: 20px;
  }
}

header { grid-area: header; }
article { grid-area: content; }
aside { grid-area: aside; }
.author-details { grid-area: author; }
.plays { grid-area: plays; }
footer { grid-area: footer; }
```

To leave a cell empty, use a period. Because the cell doesn't have a name, content cannot be assigned to it. The following example creates an empty cell in the center:

```css
.grid-container {
  grid-template-areas:
    "title tile right"
    "left  .    left"
    "footer footer footer";
}
```

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

## Sizing columns and rows

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

### Sizing functions and keywords

- `fr`: fractional unit
- `min-content`: minimum size of the content.
- `max-content`: maximum size of the content.
- `minmax(min, max)`: sets the min and max for the element.
- `repeat(num, size)`: creates `num` rows or columns of `size` width

### fr

The `fr` unit, which stands for `fraction unit`, is analogous to `flex-grow` in flexbox. You can mix and match `fr` units with other units, such as `px`.

### auto

To create columns or rows that grow according to their contents, use the `auto` value. For example, the following ruleset creates four columns that are sized based on their contents:

```css
.grid {
  grid-template-columns: repeat(4, auto);
}
```

### gap

`gap` defines the amount of space to add to the gutter between each grid cell. You can provide two values to `gap`, where the first is row gap and the second is column gap.

### repeat()

You could also define the rows and columns with the `repeat()` function. It takes the number of columns or rows to create, and the size of each column or row:

```css
.grid {
  ...
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(2, 1fr);
  ...
}
```

### span

Tells the browser the number of grid tracks that the item should span. You won't specify a specific start or end row, so the browser uses its placement algorithm to place the element on the page - this is usually where it logically falls in the HTML structure:

```css
header,
nav {
  grid-column: 1 / 3;
  grid-row: span 1;
}
```

Here, the header and the nav are the first two grid items on the page, so the header is placed on the first row, nav placed on the second row, and they are both placed within the first and third grid columns.

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

Special value for the `repeat()` function. Lets the grid create the number of rows or columns based on the grid size. This helps create responsive grids.

`autofit` returns the highest positive integer without overflowing the grid. For example, the following grid has columns that are 150px when there is enough space for more than one, or the column is 1fr:

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

Special value for the `repeat()` function. The browser places as many tracks onto the grid as it can without violating the restriction set by the size that you define:

```css
.grid-container {
  ...
  grid-template-rows: repeat(2, 1fr);
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
}
```

This behaves similarly to `autofit` until there are fewer items than can fill up the the grid row or column once. Where `autofit` uses the column's max size after it adds all columns, `autofill` uses the minimum size. The columns will grow to their max until space for a new column is available, then each column snaps back to its minimum size.

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

### Implicit grid

Implicit grid tracks have a size of `auto`, which means that they grow to the size of the grid item contents. Use `grid-auto-rows` or `grid-auto-columns` to set a size for implicit grid items.

When an item doesn't fit in the row, the browser displays it on the next row and leaves a blank area. `grid-auto-flow: dense;` tells the browser to fill gaps in the grid so there are no blank spaces, even if it has to change the display order or some grid items:

```css
.portfolio {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  grid-auto-rows: 1fr;
  gap: 1em;
  grid-auto-flow: dense;
}
```

This ruleset creates columns at least 200px wide, and rows of equal size.

## Subgrid

> Not widely supported. Only in Firefox, Safari, and Chrome.

New feature that lets you place a grid within a grid and then position the inner grid's items on the grid lines of the parent grid. Confusing so I am not very confident.
