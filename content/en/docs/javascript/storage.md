---
title: "Storage"
# linkTitle: ""
weight: 70
description:
---

Web apps can store data locally on the user's browser - called client-side storage:
- Like browser memory
- segregated by origin - data from one site can't read data stored from another
- Web apps can choose the lifetime of the data: temporarily until the browser exits, or permanently so it is avaiable much later
- All client-side storage is unencrypted

Forms of client-side storage:
- **Web Storage**: `localStorage` and `sessionStorage` objects that map string keys to string values
- **Cookies**: Old mechanism for storing small amounts of data. Cookies are always transmitted to the server with every HTTP request
- **IndexedDB**: asynchronous API to an object database that supports indexing

## localStorage and sessionStorage

`localStorage` and `sessionStorage` are properties of the Window object, and they are Storage objects:
- Storage object property values must be strings
- Properties of a Storage object persis between page reloads
- You create an arbitrary property name during assignment

```js
localStorage.myPropertyName = 'This is anything';
console.log(Object.keys(localStorage));             // is iterable
```


Because it only stores strings, you have to encode and decode the data yourself:

```js
// storing strings
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

// JSON
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

### Properties and methods

| Property           | Description |
|--------------------|-------------|
| `length`          | Returns the number of key-value pairs stored in the storage object. |
| `setItem(key, value)` | Stores a key-value pair in the storage object. If the key already exists, it updates the value. |
| `getItem(key)`    | Retrieves the value associated with the specified key. Returns `null` if the key does not exist. |
| `removeItem(key)` | Deletes the specified key and its associated value from storage. |
| `clear()`         | Removes all key-value pairs from storage. |
| `key(index)`      | Returns the key at the specified index within the storage object. Returns `null` if the index is out of bounds. |


### Storage lifetime and scope

`localStorage` is permanent and is not deleted until the user's device deletes it or it is deleted through the API:
- scoped to document origin, which means that all scripts with the same origin share one `localStorage` object
  - they can read and overwrite the data
- scoped by browser - Chrome and Firefox do not share `localStorage` data

`sessionStorage` lives as long as the window or browser tab is open:
- if the tab is restored, then the `sessionStorage` is restored too
- scoped to the document origin, just like `localStorage`
- also scoped per window - if you have 2 tabs open, each tab has its own `sessionStorage` that cannot read or write to each other

### Storage events

When `localStorage` changes, the browser fires a 'storage' event on all Window objects except the Window object that made the change
- If tab A makes a change to `localStorage`, then tab B will receive a 'storage' event
- register a handler for a storage event with `window.onstorage` or `window.addEventListener('storage', func(){...}))`
- a sort of broadcast mechanism that alerts all windows of a user preference or change
- 'storage' event listener is useful if the user changes to dark mode - the other tabs with the app open can change too

Event properties for a storage event:

| Property       | Description |
|---------------|-------------|
| `key`         | The key in `localStorage` that was changed. Returns `null` if `.clear()` was called. |
| `oldValue`    | The previous value of the key before the change. Returns `null` if the key was newly added. |
| `newValue`    | The new value assigned to the key. Returns `null` if the key was removed. |
| `url`         | The URL of the document where the storage change happened. |
| `storageArea` | The `Storage` object (`localStorage` or `sessionStorage`) where the change occurred. |


## Cookies

Small amount of named data stored by the browser and associated with a page or website:
- client-side is `cookie` object on the Document object
- designed for server-side programming
- extension of HTTP protocol
- avaialble to both the browser and server, so server-side scripts can read and write cookie data to use in the browser


### Properties
| Property     | Description |
|-------------|-------------|
| `document.cookie` | Gets or sets cookies associated with the current document. Returns all cookies as a single string. |

### Methods
| Method         | Description |
|---------------|-------------|
| `setCookie(name, value, options)` | Creates or updates a cookie with a specified name, value, and optional attributes. |
| `getCookie(name)` | Retrieves the value of a specific cookie by name. |
| `deleteCookie(name)` | Deletes a cookie by setting its expiration date in the past. |

### Attributes
| Attribute      | Description |
|---------------|-------------|
| `expires`     | Sets the expiration date of the cookie (UTC format). If not set, the cookie is a session cookie. |
| `max-age`     | Specifies the lifetime of the cookie in seconds. |
| `path`        | Defines the URL path for which the cookie is valid. Defaults to the current page’s path. |
| `domain`      | Specifies the domain the cookie belongs to. Defaults to the current domain. |
| `secure`      | Restricts the cookie to HTTPS connections only. |
| `HttpOnly`    | Prevents client-side JavaScript from accessing the cookie. Only sent to the server. |
| `SameSite`    | Controls whether cookies are sent with cross-site requests (`Strict`, `Lax`, `None`). |

