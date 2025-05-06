---
title: "Presenting tabular data"
linkTitle: "Tables"
weight: 110
# description:
---

Good article: [Grids Part 1: To grid or not to grid](https://sarahmhigley.com/writing/grids-part1/)


Some screen readers have shortcuts that jump directly to a table and announce the number of columns and rows:
- Add a `<caption>` element to label the table. The SR announces the caption when it announces the cols and rows
- If the table is nested in a `<figure>`, use a `<figcaption>` to label the table
- If you have a table cell in the body that labels the row, use the `<th>` element, even though you aren't in the table head. `<th>` elements have implicit labels
- Try to avoid spanning table headers with `rowspan` and `colspan`. SRs don't support them very well.


## Scrollable tables

To make a table scrollable:
- Wrap it in a `<div>` and add the scroll styles to the div. You can't add scroll styles directly to a table.
- Add `tabindex="0"` to make the div focusable
- Anything focusable needs an accessible name, so use `aria-labelledby="<caption-value>"` to reference the caption


## Sorting tables

```html
<div class="visually-hidden" role="status"></div>
    <table>
    <caption>Scores Group A
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
    </caption>
</table>
```


Here is the CSS:
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

Here is the JS and a brief description of each function:
- `getRows(cell, rows)`: Gets all values of the current column and saves them in an array
- `updateButton(cell)`: Puts the `aria-sort` attribute on the column header of sorted column, removes it from other column if present
- `sortRows(rows)`: Sorts and reorders rows
- `updateLiveRegion()`: Updates the live region, and clears it after one second. This tells the screen reader that sorting was successful.

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