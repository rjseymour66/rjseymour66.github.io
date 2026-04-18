---
title: "Tables"
# linkTitle: ""
weight: 100
# description:
---

> Table styling: https://estelle.github.io/CSS/tables/#slide1

Displays data in rows and columns. Good when you want to present data that should be:

- compared
- calculated
- sorted
- cross-referenced

## Table elements

All elements are wrapped in `<table>`, which has an implicit ARIA role of `table`:

- if the table has a selection state or lets the user rearrange data, use `role="grid"`
- if rows or cols can be expanded and collapsed, use `role="treegrid"`

These elements all have table rows (`<tr>`) that can have table headers (`<th>`) and table data (`<td>`):

- `<thead>`
- `<tbody>`
- `<tfoot>`

Here is a list of elements in the order they can appear in a `<table>` element:

- `<caption>`:
- `<colgroup>`: Might contain nested
- `<thead>`: Made
- `<tbody>`:
- `<tfoot>`:

### Table caption

`<caption>` is the preferred way to name a table. Alternatively, you an use `aria-label` or `aria-labelledby` to provide an accessible name to the table.

Position the caption with the CSS proprety `caption-side`. You can only set it at the top or bottom.

If you need to provide a long description of the table:

1. summarize it in the caption
2. write a summary paragraph that precedes the table
3. associate the two with the `aria-describedby` attribute

### Data sectioning

The data portion of a table consists of three sections:

- zero or more `<thead>`
- `<tbody>`
- `<tfoot>`

These don't help accessibility, but have usability benefits:

- sticky headers
- tbody contents can scroll

### Table content

All row content is in a table row element (`<tr>`). This can hold these elements:

- `<th>` if the parent element is a `<thead>`. user agents display this as bold. has implicit ARIA roles of `columnheader` and `rowheader`.
- `<td>` if the parent element is `<tbody>`

There are attributes to add padding between cells, within cells, add borders, and text alignment:

- `border-collapse`: controls the cellpadding and cellspacing, which is the space between the cell content and its border
- `border-spacing`: controls space between borders of adjacent cells.

  This has no effect if `border-collpase: collapse;` is set. If `border-collpase: separate;` is set, you can hide empty cells completely with `empty-cells: hide;`

### Merging cells

Merge cells with these attributes:

- `colspan`
- `rowspan`

### Styling tables

Tables are not responsive by default. Here are details about CSS, aria, and tables: https://adrianroselli.com/2018/02/tables-css-display-properties-and-aria.html

`<colgroup>` groups `<col>` elements together, directly under the `<caption>` element:

```html
<table>
  <caption>
    Table Caption
  </caption>
  <colgroup>
    <col />
  </colgroup>
  <thead></thead>
</table>
```

Here is the order of precedence of table elements in styling:

1. table
2. column groups
3. columns
4. row groups
5. rows
6. cells

So, if the `<table>` element and `<colgroup>` element both have a background color, the `<colgroup>` styles are on top.

To add stripes to a table:

```css
tbody tr:nth-of-type(odd) {
  background-color: rgba(0 0 0 / 0.1);
}
```
