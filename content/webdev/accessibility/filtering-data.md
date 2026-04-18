---
title: "Filtering data"
# linkTitle: ""
weight: 110
# description:
---

Consider a product listing page where users can filter by brand, price range, and availability. When a user applies a filter, the page updates with new results. A sighted user sees the list change. A screen reader user hears nothing unless you explicitly notify them. This page covers how to build accessible filter interfaces.

To make a form a landmark, add `role="form"` and give it an accessible name with `aria-label="<name>"`.

## Types of filters

There are two kinds of form filters:

*Interactive*
: Updates immediately when a user selects a filter. Provides fast feedback, but requires a partial page reload on every change.

*Batch*
: Users select several options before submitting. Reduces page reloads but may yield zero results if the user combines too many filters.

## Client-side scripting

Server-side rendering causes a full page reload, so the browser announces the new page title and the screen reader user knows the results changed. Client-side rendering updates the DOM without a page load, which provides only visual feedback. You must notify screen reader users explicitly.

You have two options to notify users about filter results:

*Move focus to the results region*
: Add `tabindex="-1"` to the results container and call `focus()` after the update. The screen reader announces the region label and then reads the first item.

*Output the results count to a live region*
: A live region (`role="status"`) announces its text content automatically whenever it changes. This approach does not interrupt the user's current position in the page.

Choose focus movement when the results are the primary next action and the user should start reading immediately. Choose a live region when you want to notify the user without interrupting their current context, such as when applying a filter from a sidebar while the results are in a separate column.

Here is the HTML:

- `tabindex="-1"` makes the results container focusable by JavaScript without adding it to the tab order.
- The `<div>` is labeled as a region and given an accessible name by its heading.

```html
<div id="results" role="region" aria-labelledby="results_heading" tabindex="-1">
    <h2 id="results_heading">Results</h2>
    <div role="status">Showing 40 of 40 records</div>
    <ol class="list">
        <li><strong>Rubber Soul</strong><br>Beatles</li>
        <li><strong>Let It Be</strong><br>Beatles</li>
        <li><strong>Help</strong><br>Beatles</li>
    </ol>
</div>
```


Here is the JavaScript. The two `finishQuery` implementations are alternatives. Choose one and delete the other:

```js
const form = document.querySelector('form');
const results = document.querySelector('#results');
const list = document.querySelector('ol');
const liveRegion = document.querySelector('[role="status"]');
let records, filtered;

// Option 1: Move focus to the results region
// Use this when the user should immediately start reading the new results.
function finishQuery() {
    results.focus();
}

// Option 2: Announce the result count via a live region
// Use this when you want to notify without moving focus away from the filters.
// (Remove Option 1 if you choose this approach.)
//
// function finishQuery() {
//     const total = records.length;
//     const found = filtered.length;
//     liveRegion.textContent = `Showing ${found} of ${total} records`;
// }

// Create a list of the results
function showResults() {
    list.innerHTML = "";
    for (let i = 0; i < filtered.length; i++) {
        const record = filtered[i];
        const item = document.createElement('li');
        const title = document.createElement('strong');
        title.textContent = `${record.title} (${record.year})`;
        list.append(title, record.artist);
        list.append(item);
    }
}

// Filter list with user input
function filterForm(e) {
    e.preventDefault();

    const formData = new FormData(form);

    filtered = records.filter(record => {
        const artist = formData.get('artist');
        const countries = formData.getAll('country');
        const shipping = formData.getAll('shipping');

        if (artist && record.artist !== artist) {
            return;
        }

        if (countries.length && !countries.includes(record.country)) {
            return;
        }

        if (shipping.length && !shipping.includes(record.shipping)) {
            return;
        }

        return true;
    });

    showResults();
    finishQuery();
}

// Call a db. Example schema below
async function getRecords() {
    // JSON response example

    [
        {
            "artist": "Beatles",
            "title": "Let It Be",
            "year": 1969,
            "country": "GB",
            "format": ["LP", "CD"],
            "shipping": "eu"
        },
        // ...
    ];
}

// Fetch data and add the event listener to the form
getRecords().then(data => {
    records = data;
    filtered = data;
    form.addEventListener('submit', filterForm);
});
```

### Pagination

Pagination manages large result sets by dividing them across pages. Follow these practices:

- Wrap the pagination list in a `<nav>` element to create a landmark.
- Apply `aria-current="page"` to the active page link. This tells screen reader users which page they are on, the same way `aria-current="page"` works in a navigation menu.
- Add a skip link at the beginning of the results region so users can jump directly to pagination without tabbing through all the results.

```html
<nav class="pagination" aria-labelledby="pagination_heading">
    <h2 id="pagination_heading">Select Page</h2>
    <ol>
        <li><a href="#" aria-current="page">1</a></li>
        ...
    </ol>
</nav>
```

### Sorting results

The following HTML applies a live region to announce sort changes. When the user selects a sort option, JavaScript updates the live region text and the screen reader announces it automatically:

```html
<fieldset id="sorting">
    <legend>Sort by</legend>
        <div>
            <input type="radio" name="sorting" id="sorting_artist" checked="checked">
            <label for="sorting_artist">Artist</label>
            <input type="radio" name="sorting" id="sorting_date">
            <label for="sorting_date">Date</label>
        </div>
</fieldset>
<div id="live-region-sorting" hidden="hidden">Sorted by [type]</div>
```
