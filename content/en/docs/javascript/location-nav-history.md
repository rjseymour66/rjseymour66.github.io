---
title: "Location, navigation, and history"
linkTitle: "Location, nav, history"
weight: 50
description:
---

## Location

The Location object is represented by the `location` property on the Window and Document object, and it represents the current URL of the document displayed in the window:
- `window.location` is the same as `document.location`
- `document.URL` is the URL string
- provides an API for loading new documents in the window
- `href` property and `toString()` method return the URL
- `hash` returns the fragment identifier
- `search` returns the part of the URL that starts with a `?` - the query string

`URL` objects have a `searchParams` property that is a parsed representation of the `search` property
- Location object does not have the `searchParams` property - you need to make a URL object out of the location object and then you can parse the query parameters:

```js
// http://127.0.0.1:5500/index.html?q=test
let url = new URL(window.location);         // URL object  
let query = url.searchParams.get("q");      // test
```

### Loading new documents

You can change the document that the browser loads by assigning `window.location` a new URL string or fragment:
- `window.location = 'http://www.google.com';` - absolute
- `window.location = 'headings.html';` - relative
- `location = '#section1';` - fragment
- `replace()` method takes a string URL and loads a new page, and then also replaces the calling document in the browser's history.
  - Use case: if the user's browser doesn' not support features on your JS page, you can use `replace('https://static-site.com')` to replace it with a static version.
- `reload()` method makes the document reload

### Browsing history