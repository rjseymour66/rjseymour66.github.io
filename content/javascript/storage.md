---
title: "Storage"
# linkTitle: ""
weight: 100
description:
---

Browsers provide several APIs for storing data locally on the user's device. This is called *client-side storage*. All client-side storage is segregated by origin: data stored by one site cannot be read by another. All client-side storage is also unencrypted, so you should never store sensitive data such as passwords or authentication tokens in any client-side storage mechanism.

Web apps control how long data persists. Some storage lasts only until the browser tab closes. Other storage remains until the user clears it or the app removes it through the API.

Browser apps have three client-side storage options:

- *Web Storage*: `localStorage` and `sessionStorage` objects that map string keys to string values
- *Cookies*: A mechanism for storing small amounts of named data. The browser transmits cookies to the server with every HTTP request, making them suitable for session management.
- *IndexedDB*: An asynchronous API for a structured object database that supports indexing and complex queries.

## localStorage and sessionStorage

`localStorage` and `sessionStorage` are properties of the `Window` object. Both implement the `Storage` interface: all property values must be strings, and stored data persists between page reloads. You can assign to storage using an arbitrary property name:

```js
localStorage.myPropertyName = 'This is anything';
console.log(Object.keys(localStorage));             // is iterable
```

Because `Storage` only accepts strings, you must encode non-string values before storing them and decode them after retrieval. The following example demonstrates encoding numbers, dates, and objects for storage:

```js
// storing numbers
let x = 10;
console.log(typeof (x));                // number
localStorage.x = x;
console.log(typeof (localStorage.x));   // string

let y = parseInt(localStorage.x);       // convert back to number

// storing dates
localStorage.rightNow = new Date().toUTCString();
let oldDate = localStorage.rightNow;
console.log(typeof (oldDate));                      // string
let parsedDate = new Date(Date.parse(oldDate));
console.log(typeof (parsedDate));                   // object (Date object)

// storing objects as JSON
let jsonObj = {
    key: "value",
    boolish: true,
    test: "tester",
    age: 41
};

localStorage.json = JSON.stringify(jsonObj);
let data = JSON.parse(localStorage.json);

console.log(localStorage.json);                 // string
console.log(data);                              // JSON obj
```

### Real-world example: typed localStorage wrapper

`localStorage` only stores strings, so every non-string value requires serialization before writing and deserialization after reading. A thin wrapper around `localStorage` handles this automatically and provides a consistent API for all value types:

```js
const store = {
    get(key, fallback = null) {
        try {
            const raw = localStorage.getItem(key);
            return raw !== null ? JSON.parse(raw) : fallback;
        } catch {
            return fallback;    // handles malformed JSON gracefully
        }
    },
    set(key, value) {
        localStorage.setItem(key, JSON.stringify(value));
    },
    remove(key) {
        localStorage.removeItem(key);
    },
    update(key, updater, fallback = null) {
        this.set(key, updater(this.get(key, fallback)));
    },
};

// Booleans, numbers, arrays, objects — all work transparently
store.set('darkMode', true);
const isDark = store.get('darkMode', false);    // boolean, not string

store.set('cart', []);
store.update('cart', items => [...items, { id: 1, qty: 2 }]);
const cart = store.get('cart', []);
```

### Real-world example: shopping cart

The following example implements a persistent shopping cart that survives page refreshes and loads instantly from `localStorage` on the next visit:

```js
const CartStore = (() => {
    const KEY = 'shopping_cart';
    let items = JSON.parse(localStorage.getItem(KEY)) ?? [];

    const save = () => localStorage.setItem(KEY, JSON.stringify(items));

    return {
        getItems: () => [...items],

        add(product) {
            const existing = items.find(i => i.id === product.id);
            if (existing) {
                existing.qty++;
            } else {
                items.push({ ...product, qty: 1 });
            }
            save();
        },

        remove(productId) {
            items = items.filter(i => i.id !== productId);
            save();
        },

        total() {
            return items.reduce((sum, i) => sum + i.price * i.qty, 0);
        },

        clear() {
            items = [];
            save();
        },
    };
})();

CartStore.add({ id: 'shoe-42', name: 'Trail Runner', price: 89.99 });
CartStore.add({ id: 'sock-m',  name: 'Merino Socks', price: 14.99 });
console.log(`Total: $${CartStore.total().toFixed(2)}`);  // Total: $104.98
```

### Properties and methods

The `Storage` interface exposes the following properties and methods:

| Property | Description |
|:---|:---|
| `length` | Returns the number of key-value pairs stored in the storage object. |
| `setItem(key, value)` | Stores a key-value pair. If the key already exists, it updates the value. |
| `getItem(key)` | Returns the value for the specified key, or `null` if the key does not exist. |
| `removeItem(key)` | Deletes the specified key and its associated value. |
| `clear()` | Removes all key-value pairs from storage. |
| `key(index)` | Returns the key at the specified index, or `null` if the index is out of bounds. |


### Lifetime and scope

`localStorage` data persists until the user clears browser storage or the app removes it through the API. It is scoped to the document origin: all scripts sharing the same origin read from and write to the same `localStorage` object. `localStorage` is also scoped to the browser itself. Chrome and Firefox maintain separate stores and do not share data.

`sessionStorage` lasts only as long as the browser window or tab is open. If the user restores a closed tab, the browser restores its `sessionStorage` as well. Like `localStorage`, it is scoped to the document origin. Unlike `localStorage`, it is also scoped per window: each tab maintains its own separate `sessionStorage` that other tabs cannot read or write.

### Storage events

When `localStorage` changes, the browser fires a `storage` event on every `Window` object except the one that made the change. If tab A writes to `localStorage`, tab B receives a `storage` event. This makes the `storage` event a broadcast mechanism: when one tab updates a user preference, all other open tabs can respond immediately.

Attach a `storage` event handler with `window.onstorage` or `window.addEventListener('storage', handler)`. A dark mode toggle is a typical scenario: when the user switches themes in one tab, other open tabs can update their appearance without a page refresh.

### Real-world example: syncing dark mode across tabs

The `storage` event fires in every tab *except* the one that made the change. Attaching a listener keeps UI state consistent when the user has multiple tabs open:

```js
const THEME_KEY = 'theme';

function applyTheme(theme) {
    document.documentElement.dataset.theme = theme;   // CSS can use [data-theme="dark"]
}

// Apply saved theme on load
applyTheme(localStorage.getItem(THEME_KEY) ?? 'light');

// Toggle theme in the current tab
document.querySelector('#theme-toggle').addEventListener('click', () => {
    const next = document.documentElement.dataset.theme === 'dark' ? 'light' : 'dark';
    localStorage.setItem(THEME_KEY, next);
    applyTheme(next);
});

// Keep all other open tabs in sync
window.addEventListener('storage', (e) => {
    if (e.key === THEME_KEY && e.newValue) {
        applyTheme(e.newValue);
    }
});
```

The `storage` event object exposes the following properties:

| Property | Description |
|:---|:---|
| `key` | The key in `localStorage` that was changed. Returns `null` if `.clear()` was called. |
| `oldValue` | The previous value of the key before the change. Returns `null` if the key was newly added. |
| `newValue` | The new value assigned to the key. Returns `null` if the key was removed. |
| `url` | The URL of the document where the storage change occurred. |
| `storageArea` | The `Storage` object (`localStorage` or `sessionStorage`) where the change occurred. |


## Cookies

*Cookies* are small pieces of named data the browser stores and associates with a specific website. Originally designed for server-side programming, cookies are an extension of the HTTP protocol: the browser transmits them to the server with every HTTP request, and server-side scripts can set them in response headers. Client-side JavaScript can also read and write cookies through `document.cookie`, unless a cookie carries the `HttpOnly` attribute, which restricts it to server access only.

Configure a cookie's lifetime and scope by appending attributes to the cookie string when writing it.

`document.cookie` returns all cookies for the current page as a single string of semicolon-separated name/value pairs. To access an individual value, split the string and decode each pair. The following function parses all accessible cookies into a `Map`:

```js
document.cookie = 'name=value';         // set a cookie

// set multiple cookies
document.cookie = 'sessionId=123456789';
document.cookie = 'username=ricky';

let getCookies = () => {
    let cookies = new Map();
    let all = document.cookie;
    console.log(all);
    let list = all.split("; ");
    console.log(list);
    for (let cookie of list) {
        if (!cookie.includes("=")) continue;
        let p = cookie.indexOf('=');
        let name = cookie.substring(0, p);
        let value = cookie.substring(p + 1);
        value = decodeURIComponent(value);
        cookies.set(name, value);
    }
    return cookies;
};

getCookies();
```

Cookies marked with the `HttpOnly` attribute are not accessible from JavaScript. The server sets them and only the server can read them. The following example reads all JavaScript-accessible cookies and extracts a specific value:

```js
let cookie = decodeURIComponent(document.cookie);
let cookieList = cookie.split(";");
for (let i = 0; i < cookieList.length; i++) {
  let c = cookieList[i];
  if (c.charAt(0) == " ") {
    c = c.trim();
  }
  if (c.startsWith("name")) {
    alert(c.substring(5, c.length));
  }
}
```

### Lifetime and scope

By default, cookies are *session cookies*: they persist for the life of the browser session and disappear when the user closes the window. Adding a `max-age` attribute tells the browser how long to keep a cookie, in seconds. The browser stores persistent cookies in a file and deletes them when they expire.

The `path` and `domain` attributes control which pages can access a cookie. By default, a cookie is scoped to the document origin and path. A cookie set at `/first/second/cookies` is accessible to that path and any sub-path such as `/first/second/cookies/subdir`, but not to `/first/second/`. In most cases, the default path scoping is sufficient. Set `path=/` to make a cookie accessible to every page in the domain. Set the `domain` attribute to share a cookie across subdomains. For example, setting `domain=example.com` makes the cookie available on `docs.example.com`, `shop.example.com`, and any other subdomain.

Set the `secure` attribute to restrict a cookie to HTTPS connections only.

Browsers enforce the following limits on cookie storage:

- 300 cookies in total
- 20 cookies per web server
- 4 KB per cookie

### Storing cookies

To associate a cookie with the document, assign to the `cookie` property on the `Document` object. Cookie names and values cannot contain semicolons, commas, or whitespace, so encode values with `encodeURIComponent()` and decode them with `decodeURIComponent()`.

Append attributes to the cookie string to configure its behavior:

- To set any attribute, append `; attribute=value`
- To set the `secure` flag, append `; secure`
- To update a cookie's lifetime, reset `max-age` with a new value
- To delete a cookie, set `max-age=0`

The following example includes a `setCookie` helper that handles encoding and `max-age` calculation, and a `deleteCookie` function:

```js
document.cookie = 'test=value'

// set a cookie with an optional expiration in days
let setCookie = (name, value, daysToLive) => {
    let cookie = `${name}=${encodeURIComponent(value)}`;
    if (daysToLive !== null) {
        cookie += `; max-age=${daysToLive * 60 * 60 * 24}`;
    }
    document.cookie = cookie;
};

// delete a cookie
let deleteCookie = (name) => {
    document.cookie = `${name}=""; max-age=0`;
};
```

### Properties

The `Document` object exposes the following cookie property:

| Property | Description |
|:---|:---|
| `document.cookie` | Gets or sets cookies associated with the current document. Returns all cookies as a single string. |

### Attributes

Cookie attributes configure lifetime, scope, and security. Append them to the cookie string when writing:

| Attribute | Description |
|:---|:---|
| `expires` | Sets the expiration date of the cookie in UTC format. If not set, the cookie is a session cookie. |
| `max-age` | Specifies the lifetime of the cookie in seconds. |
| `path` | Defines the URL path for which the cookie is valid. Defaults to the current page's path. |
| `domain` | Specifies the domain the cookie belongs to. Defaults to the current domain. |
| `secure` | Restricts the cookie to HTTPS connections only. |
| `HttpOnly` | Prevents client-side JavaScript from accessing the cookie. Only sent to the server. |
| `SameSite` | Controls whether cookies are sent with cross-site requests (`Strict`, `Lax`, `None`). |

## IndexedDB

*IndexedDB* is a transactional, NoSQL database built into the browser. It stores structured data, including files and blobs, and supports indexed queries for efficient lookup. Unlike `localStorage`, IndexedDB is not limited to string values and can handle large amounts of structured data. It is intended for offline-capable applications and is often combined with Service Workers to enable full offline functionality.

IndexedDB organizes data into *object stores*, which are equivalent to tables in a relational database. Each record is identified by a *key*, and you can create *indexes* on any property to search by values other than the key. All operations are asynchronous and must run inside a *transaction*.

The raw IndexedDB API is event-based and verbose. Most developers prefer the `idb` library, which provides a Promise-based interface over the same API without adding abstraction overhead.

The following example opens a database, creates an object store for tasks with a status index, and adds a record:

```js
const request = indexedDB.open('todo-app', 1);

// Create the object store on first open or version upgrade
request.onupgradeneeded = (event) => {
    const db = event.target.result;
    if (!db.objectStoreNames.contains('tasks')) {
        const store = db.createObjectStore('tasks', { keyPath: 'id', autoIncrement: true });
        store.createIndex('status', 'status', { unique: false });
    }
};

request.onsuccess = (event) => {
    const db = event.target.result;

    // Add a task inside a transaction
    const tx = db.transaction('tasks', 'readwrite');
    const store = tx.objectStore('tasks');
    store.add({ text: 'Buy groceries', status: 'open' });

    tx.oncomplete = () => console.log('Task saved');
    tx.onerror = (e) => console.error('Transaction failed:', e.target.error);
};

request.onerror = (event) => {
    console.error('Failed to open database:', event.target.error);
};
```
