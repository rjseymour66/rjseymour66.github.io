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


### Lifetime and scope

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

- https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies
- https://javascript.info/cookie

Small amount of named data stored by the browser and associated with a page or website:
- client-side is `cookie` object on the Document object
- designed for server-side programming
- extension of HTTP protocol
- avaialble to both the browser and server, so server-side scripts can read and write cookie data to use in the browser
- specify the lifetime and scope of a cookie with attributes (specially formatted strings) on the 


Set cookies by setting `cookie` property of the document:
- list of name/value pairs delimited by semi-colon and space
- `split()` method breaks it into pairs
- After you extract the cookie, you have to decode the cookie

```js
document.cookie = 'name=value';         // set a cookie

// func to get cookies
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

### Lifetime and scope

Cookies are transient - they persist for the life of the browser session and disappear when the user exits the window:
- `max-age` attribute lets browser to know how long to store cookies
  - browser stores cookies in a file and deleted when they expire
- `path` and `domain` attributes configure the scope
  - scoped by origin and document path
  - cookies are available in any directory or subdirectory that it exists in: `/first/second/cookies` and `/first/second/cookies/subdir` can view the same cookies, but `first/second/` cannot
  - You usually want this default visibility
  - Set `path` with a web server URL to share cookies across a website
  - Ex: Set to `/` to make it avaialble to any page in the domain
  - Set the `domain` attribute to make cookies across the entire domain, including subdomains.
  - Ex: Set to `example.com` to make cookies available across `docs.example.com`, `shop.example.com`, etc
- `secure` attribute to `true` to send cookies via HTTPS only

Limitations on browser storage:
- 300 cookies in total
- 20 cookies per web server
- 4KB per cookie

### Storing cookies

To associate a cookie with the document, just set the `cookie` property on the Document object:
- use `encodeUIRComponent()` to encode because cannot include semicolons, commas, or whitespace
  - decode with `decodeUIRComponent()`
- to set an attribute, append `; attribute=value`
- to set `secure`, just appened `;secure`
- change lifetime of a cookie by reseting `max-age`
- to delete a cookie, set `max-age=0`

```js
document.cookie = 'test=value'

// create a cookie function
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

| Property     | Description |
|-------------|-------------|
| `document.cookie` | Gets or sets cookies associated with the current document. Returns all cookies as a single string. |


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

## IndexedDB

https://javascript.info/indexeddb

A database built into the browser that is usually too much for standard apps:
- intended for offline apps
- often combined with ServiceWorkers and other technologies

