---
title: "Presenting tabular data"
linkTitle: "Tables"
weight: 110
# description:
---

Good article: [Grids Part 1: To grid or not to grid](https://sarahmhigley.com/writing/grids-part1/)

Tables appear in dashboards, leaderboards, financial reports, and comparison pages. When a screen reader encounters a table, it announces the number of columns and rows before the user reads any data. This orientation depends on correct table markup. Some screen readers also provide shortcuts to jump directly to a table.

Follow these practices for all tables:

- Add a `<caption>` element to label the table. The screen reader announces the caption along with the column and row count, giving users immediate context.
- If the table is nested in a `<figure>`, apply a `<figcaption>` to label it instead.
- If a table cell in the `<tbody>` labels its row, apply the `<th>` element even though it is outside the table header section. The `<th>` element carries an implicit label role.
- Avoid spanning headers with `rowspan` and `colspan`. Screen reader support for spanned headers is inconsistent.


## Scrollable tables

You cannot apply scroll styles directly to a `<table>`. To make a table scrollable, follow these steps:

1. Wrap the table in a `<div>` and apply the scroll styles to the `<div>`.
2. Add `tabindex="0"` to the `<div>` to make it keyboard-focusable, so keyboard users can scroll the container.
3. Add `aria-labelledby="<caption-id>"` to the `<div>` to give it an accessible name. Anything focusable must have one.


## Sorting tables

The following example shows a sortable table with an accessible live region that announces when sorting changes. A sighted user sees the arrow icons change. A screen reader user needs the live region announcement to know the sort order changed.

Here is the HTML. The `<caption>` provides the table's accessible name. The sort buttons in each column header contain decorative SVG arrows hidden with `aria-hidden`:

```html
<div class="visually-hidden" role="status"></div>
<table>
    <caption>Scores Group A</caption>
    <thead>
        <tr>
            <th><button class="sort">Name
            <svg width="13" viewBox="0 0 126 171" aria-hidden="true">
                <path d="M62.7 3.9 6 70l114-.5z"/>
                <path d="M63 166.5 6 100.6h114z"/>
            </svg>
            </button></th>
            <th><button class="sort">Score
            <svg width="13" viewBox="0 0 126 171" aria-hidden="true">
                <path d="M62.7 3.9 6 70l114-.5z"/>
                <path d="M63 166.5 6 100.6h114z"/>
            </svg>
            </button></th>
            <th>Country</th>
        </tr>
    </thead>
    <tbody>
        <tr>
        <td>Michael</td>
        <td>27</td>
        <td>America</td>
        </tr>
        <tr>
        <td>Robert</td>
        <td>7</td>
        <td>Croatia</td>
        </tr>
    </tbody>
</table>
```

The CSS resets the sort button styles and fills the active arrow path based on the `aria-sort` attribute. The `aria-sort` attribute on the column header communicates the current sort direction to screen readers, so users hear "Name, ascending" rather than just "Name":

```css
.sort {
  all: unset;
  display: flex;
  gap: 0.4rem;
  align-items: center;
}

.sort path {
  fill: transparent;
  stroke: currentColor;
  stroke-width: 12;
}

[aria-sort="ascending"] path:first-child {
  fill: currentColor;
}

[aria-sort="descending"] path:last-child {
  fill: currentColor;
}
```

Here is the JavaScript. Each function handles one responsibility:

- `getRows(cell, rows)`: Gets all values of the current column and saves them in an array for sorting.
- `updateButton(cell)`: Puts the `aria-sort` attribute on the sorted column header and removes it from any other sorted column.
- `sortRows(rows)`: Sorts and reorders the table rows in place.
- `updateLiveRegion()`: Announces the sort result to screen readers and clears the announcement after one second to avoid stale announcements.

```js
const table = document.querySelector('table');
const liveRegion = document.querySelector('[role="status"]');
let toSort;
let direction = 'ascending';

table.addEventListener('click', e => {
    const button = e.target.closest('thead button');

    if (button) {
        const cell = button.parentNode;
        const tbody = table.querySelector('tbody');
        const rows = tbody.querySelectorAll('tr');

        toSort = [];
        getRows(cell, rows);
        updateButton(cell);
        sortRows(rows);
        updateLiveRegion();
    }
});

const getRows = (cell, rows) => {
    const index = [...cell.parentNode.children].indexOf(cell);

    for (let i = 0; i < rows.length; i++) {
        const row = rows[i];
        const cells = row.querySelectorAll('td');

        toSort.push([cells[index].innerText, row.cloneNode(true)]);
    }
};

const sortRows = rows => {
    toSort.sort(function (a, b) {
        const comp = a[0].localeCompare(b[0], "en", { numeric: true });
        return comp;
    });

    if (direction === "descending") {
        toSort.reverse();
    }

    for (let i = 0; i < rows.length; i++) {
        const row = rows[i];
        row.parentNode.replaceChild(toSort[i][1], row);
    }
};

const updateButton = cell => {
    const sortedColumn = table.querySelector('[aria-sort]');
    if (sortedColumn && sortedColumn !== cell) {
        sortedColumn.removeAttribute('aria-sort');
    }

    direction = cell.getAttribute('aria-sort') === 'ascending' ? 'descending' : 'ascending';
    cell.setAttribute('aria-sort', direction);
};

const updateLiveRegion = () => {
    liveRegion.textContent = `Sorted ${direction}`;

    setTimeout(() => {
        liveRegion.textContent = ``;
    }, 1000);
};
```
