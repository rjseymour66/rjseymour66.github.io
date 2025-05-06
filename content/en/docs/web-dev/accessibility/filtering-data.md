---
title: "Filtering data"
# linkTitle: ""
weight: 110
# description:
---

To make a form a landmark, use the `role="form"` and add an accessible name with `aria-label="<name>"`.

## Types of filters

There are two kinds of form filters:
- Interactive: Updates when a user clicks a filter. Gets immediate feedback, but there is a performance hit because parts of the page must reload.
- Batch: Users select several options before submitting the form, but it might yield zero results.

## Client-side scripting

Server-side rendering requires a full page load, so users are aware of changes to the page. Client-side rendering changes the DOM, which provides only visual feedback.

You have two options to notify users about the filter results:
- Move focus from the submit button to the region. Add `tabindex="-1"` to the HTML and use the `focus()` method
- Output the results to a live region. The screen reader will announce changes to a live region.

Here is the HTML:
- `tabindex="-1"` makes it focusable by JS
- The `<div>` is labeled as a region and labeled by the heading

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


Here is the basic JS. There are two implementations of the `finishQuery()` function so you can either focus with JS or output the list to a live region:

```js
const form = document.querySelector('form');
const results = document.querySelector('#results');
const list = document.querySelector('ol');
const liveRegion = document.querySelector('[role="status"]');
let records, filtered;

// Option 1: Focus region with JS
function finishQuery() {
    results.focus()
}

// Option 2: Output to live region
function finishQuery() {
    const total = records.length;
    const found = filtered.length;
    liveRegion.textContent = `Showing ${found} or ${total} records`;
}

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

// Fetch data add the event listener to the form
getRecords().then(data => {
    records = data;
    filtered = data;
    form.addEventListener('submit', filterForm);
});
```

### Pagination

Pagination helps you manage lots of results data. Some tips:
- wrap the pagination list in a `<nav>` element to create a landmark
- use `aria-current="page"` on the current page
- add a skip link a the beginning of the results region so users can access the pagination instead of cycling through the results first

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

Here is an example of a live region for the sorting results:

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