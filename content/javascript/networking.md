---
title: "Networking"
# linkTitle: ""
weight: 90
description:
---

Browsers load every web page over HTTP or HTTPS, and JavaScript provides APIs that let you make those same requests from your own code. The `fetch()` API handles HTTP requests, the URL API constructs and parses URLs, and the WebSocket and Server-Sent Events APIs maintain persistent server connections.

## fetch()

`fetch()` is a promise-based API for making HTTP and HTTPS requests. It replaces the older *XMLHttpRequest* (XHR) API and supports every standard HTTP method and use case.

Every `fetch()` call follows the same three-step pattern:

1. Call `fetch()` with the URL of the resource you want to retrieve.
2. When `fetch()` resolves, call a method on the Response object to read the response body. This step is asynchronous and requires a second `.then()` call or a second `await` expression.
3. Process the response body.

`fetch()` accepts a URL and optional request properties in three forms:

- `fetch(urlString)`: a plain string URL for simple GET requests
- `fetch(urlString, options)`: a string URL plus an options object for configuring the method, headers, body, and more
- `fetch(request)`: a `Request` object that bundles the URL and options together

The following example fetches a blog post by ID and logs its title:

```js
let url = 'https://jsonplaceholder.typicode.com/posts/3';

fetch(url)                                      // 1. call fetch with a URL
    .then(response => response.json())          // 2. read the response body
    .then(json => console.log(json.title));     // 3. process the data
```

You can also pass the URL and options as separate arguments. The following example creates a new post by sending a JSON body:

```js
fetch('https://jsonplaceholder.typicode.com/posts', {
  method: 'POST',
  body: JSON.stringify({
    title: 'Post title!',
    body: 'Lorem ipsum odor amet, consectetuer adipiscing elit. Feugiat habitasse sodales efficitur ornare mollis parturient. Vehicula lobortis quisque ultricies magnis vulputate habitant curae porta mi. Ultrices egestas orci class elit dictum.',
    userId: 1,
  }),
  headers: {
    'Content-type': 'application/json; charset=UTF-8',
  },
})
  .then((response) => response.json())
  .then((json) => console.log(json));
```

Alternatively, bundle the URL and options into a `Request` object and pass it to `fetch()`:

```js
let postUrl = 'https://jsonplaceholder.typicode.com/posts';
let request = new Request(postUrl, {
    method: 'POST',
    body: JSON.stringify({
        title: 'Post title!',
        body: 'Lorem ipsum odor amet, consectetuer adipiscing elit. Feugiat habitasse sodales efficitur ornare mollis parturient. Vehicula lobortis quisque ultricies magnis vulputate habitant curae porta mi. Ultrices egestas orci class elit dictum.',
        userId: 1,
    }),
    headers: {
        'Content-type': 'application/json; charset=UTF-8',
    },
});

fetch(request)
    .then(response => response.json())
    .then(json => console.log(json));
```

### Request object

A `Request` object bundles a URL and its configuration into a single value. This is useful when you want to define a request once and pass it to `fetch()` later, or when you need to inspect or clone the request before sending it.

The following example builds a `Request` object for a GET call and passes it to `fetch()`:

```js
let url = 'https://jsonplaceholder.typicode.com/posts/1';
let request = new Request(url, {
    method: 'GET',
    headers: {
        'Content-type': 'application/json; charset=UTF-8'
    },
});

fetch(request)
    .then(resp => resp.json())
    .then(json => console.log(json));
```

#### POST requests

When you send a POST request, you usually send the request body as a JSON object. You can also send values as a `URLSearchParams` object with the `Content-Type` header set to `application/x-www-form-urlencoded`.

The following example handles a registration form submission, sending the name and email fields to an API endpoint:

```js
document.getElementById('userForm').addEventListener('submit', async function (event) {
    event.preventDefault(); // Prevent page reload

    // Create URLSearchParams object and append form data
    const formData = new URLSearchParams();
    formData.append('name', document.getElementById('name').value);
    formData.append('email', document.getElementById('email').value);

    try {
        // Send the form data to the REST API
        const response = await fetch('https://example.com/api/users', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/x-www-form-urlencoded'
            },
            body: formData.toString() // Convert URLSearchParams to string
        });

        // Handle response
        if (response.ok) {
            document.getElementById('responseMessage').textContent = 'User successfully added!';
        } else {
            document.getElementById('responseMessage').textContent = 'Failed to add user.';
        }
    } catch (error) {
        console.error('Error:', error);
        document.getElementById('responseMessage').textContent = 'An error occurred.';
    }
});
```

#### Request object properties

The options object accepted by both `fetch()` and `new Request()` supports the following properties:

| Option | Description |
|:---|:---|
| `method` | HTTP method (GET, POST, etc.) |
| `headers` | Custom headers (e.g., `Authorization`, `Content-Type`) |
| `body` | Request body (used in `POST`, `PUT`) |
| `mode` | Cross-origin mode (`cors`, `same-origin`, `no-cors`) |
| `credentials` | Handles cookies (`omit`, `same-origin`, `include`) |
| `cache` | Caching behavior (`default`, `no-cache`, `reload`, `force-cache`, `only-if-cached`) |
| `redirect` | How redirects are handled (`follow`, `error`, `manual`) |
| `referrer` | Referrer information |
| `referrerPolicy` | Controls how referrer info is sent |
| `integrity` | Subresource integrity check |
| `keepalive` | Keeps the request alive after page unload |
| `signal` | Allows request cancellation (`AbortController`) |


### URL API

The `URL` class parses a URL string into its components and lets you read or modify each part. It correctly handles URL encoding and decoding, which makes it preferable to the legacy `escape()` and `unescape()` functions. The `origin` property is read-only. All other properties are read-write.

The `URL` API is supported in all modern browsers. Internet Explorer does not support it.

The following example shows the components available on a parsed `URL` object:

```js
url.href                // 'https://example.com:8080/path/name?q=term&key=value#fragment'
url.origin              // https://example.com:8080
url.protocol            // https:
url.host                // example.com:8080
url.hostname            // example.com
url.port                // 8080
url.pathname            // /path/name
url.search              // ?q=term&key=value
url.hash                // #fragment
url.toString()          // https://example.com:8080/path/name?q=term&key=value#fragment
```

HTTP requests can encode multiple form field values in the query portion of a URL. The `url.search` property returns the full query string, including the leading `?`. The `url.searchParams` property provides a dedicated API for reading and modifying individual key/value pairs. Avoid the legacy `escape()` and `unescape()` functions for URL encoding. Prefer `encodeURI()` and `decodeURI()` when you must encode manually, or rely on the `URL` object, which handles encoding automatically.

The following example demonstrates the `searchParams` API:

```js
let url = new URL('https://example.com/search');
url.searchParams.append('key', 'value');            // add new key/value pair
url.searchParams.set('key', 'new-value');           // change value for 'key'
url.searchParams.get('key')                         // return value for 'key'
url.searchParams.has('key')                         // Boolean, whether 'key' exists
url.searchParams.append('opts', 'extra-values');    // add new key/value pair
url.searchParams.append('opts', 'more-values');     // add another pair with the same key
url.searchParams.getAll('opts')                     // return all values for 'opts'
url.searchParams.sort()                             // sort all pairs by key name
url.searchParams.delete('opts');                    // delete all pairs with 'opts' key
```

To build a query string from scratch, create a `URLSearchParams` object, append your parameters, and assign it to `url.search`:

```js
let url = new URL('https://example.com/search');
let params = new URLSearchParams();
params.append('one', 'value');
params.append('key', 'pair');
url.search = params;
```


### Headers

Response headers are stored in a `Headers` object. Header names are case-insensitive. The `has()` method returns a Boolean indicating whether a header is present, and `get()` returns its value.

The following example iterates over all headers in a response and logs each name/value pair:

```js
// iterate through all headers
let logResponseHeaders = responseObject => {
    for (let [name, value] of responseObject.headers) {
        console.log(`Header "${name}": ${value}`);
    }
};
```

When you include a request body in a `PUT` or `POST` request, the browser automatically adds two headers:

- `Content-Length`
- `Content-Type`, set to `text/plain; charset=UTF-8` by default

If you send HTML or JSON, override the default by setting the correct `Content-Type` header (`text/html` or `application/json`). The following example sends a JSON body with the appropriate content type:

```js
let postUrl = 'https://jsonplaceholder.typicode.com/posts';

fetch(postUrl, {
    method: 'POST',
    body: JSON.stringify({
        title: 'My title',
        body: 'This is the request body!',
        userId: 1,
    }),
    headers: {
        'Content-Type': 'application/json; charset=UTF-8',
    }
})
    .then(resp => resp.json())
    .then(json => console.log(json));
```

### Set request parameters

You can build a URL with query parameters dynamically by setting values on `url.searchParams`. This is useful when your API accepts filter or search criteria as query string parameters, such as fetching all posts for a specific user.

The following example constructs the URL `https://jsonplaceholder.typicode.com/posts?userId=1` and fetches the matching results:

```js
let postByUser = (userId) => {
    let url = new URL('https://jsonplaceholder.typicode.com/posts');
    url.searchParams.set('userId', userId);
    fetch(url)
        .then(response => response.json())
        .then(data => console.log(data));
};

postByUser(1);
```

### Set request headers

You can set custom request headers in two ways: pass a `Headers` object as the `headers` property of the `fetch()` options argument, or bundle headers inside a `Request` object. The `Headers` object's `.set()` method takes a header name and a value.

The following example sets an `Authorization` header for HTTP Basic authentication before fetching a list of posts:

```js
let authHeaders = new Headers();                                                // create Headers obj
authHeaders.set('Authorization', `Basic ${btoa(`${username}:${password}`)}`);   // .set() method

fetch('https://jsonplaceholder.typicode.com/posts',
    { headers: authHeaders })
    .then(response => response.json())
    .then(json => console.log(json.body));
```

### Response object

`fetch()` returns a Promise that resolves to a `Response` object. The Promise resolves as soon as the response starts to arrive. At that point, the status code and headers are available, but the body may not be fully received yet.

The Promise only rejects in these cases:

- The user's computer is offline
- The server is unresponsive
- The URL hostname does not exist

Always include a `.catch()` clause with `fetch()` to handle these network failures.

### The fetch gotcha: 4xx and 5xx do not reject

`fetch()` only rejects its Promise when the network fails entirely (offline, DNS failure, server unreachable). A `404 Not Found` or `500 Internal Server Error` response *resolves* the Promise. You must check `response.ok` yourself:

```js
// WRONG — logs the 404 body as if the request succeeded
fetch('/api/missing')
    .then(res => res.json())
    .then(data => console.log(data));

// CORRECT — check ok before reading the body
async function getUser(id) {
    const res = await fetch(`/api/users/${id}`);
    if (!res.ok) {
        throw new Error(`HTTP ${res.status}: ${res.statusText}`);
    }
    return res.json();
}
```

`response.ok` is `true` for status codes 200–299 only.

The following example adds content-type checking alongside the status check:

```js
fetch(url)
    .then(response => {
        let contentType = response.headers.get('Content-Type');
        if (response.ok && contentType.includes('application/json')) {
            return response.json();
        } else {
            throw new Error(`Unexpected response status ${response.status} or content type`);
        }
    })
    .then(data => console.log(data.body))                               // when resp body resolves, log it
    .catch(error => console.log('Error while fetching post', error));   // handle error
```

#### Response object properties

The `Response` object exposes the following properties:

| Property | Description |
|:---|:---|
| `ok` | `true` if status is 200–299 |
| `status` | HTTP status code (e.g., `200`, `404`) |
| `statusText` | HTTP status message (e.g., `"OK"`) |
| `headers` | Response headers (`Headers` object) |
| `url` | Final URL after redirects |
| `redirected` | `true` if redirected |
| `type` | Response type (`"cors"`, `"basic"`, `"opaque"`, etc.) |
| `body` | Raw response body (`ReadableStream`) |
| `bodyUsed` | `true` if the response body has been read (for example, by calling `.json()` or `.text()`) |

#### Response object methods

The `Response` object provides the following methods for reading the response body:

| Method | Description |
|:---|:---|
| `json()` | Returns the response body as a parsed JSON object. |
| `text()` | Returns the response body as a string. |
| `arrayBuffer()` | Returns a Promise that resolves to an `ArrayBuffer`. Appropriate for binary data, which you can wrap in a `DataView` to read. |
| `blob()` | Returns the response body as a *Binary Large Object* (Blob), suited for large binary payloads. The browser may stream the response to a temporary file and return a Blob representing it. You cannot randomly access Blob data. Work with the contents via `URL.createObjectURL()` or the `FileReader` API. |
| `formData()` | Returns a Promise that resolves to a `FormData` object. Handles bodies encoded as `multipart/form-data`, a common format in POST requests. |

##### Streaming response bodies

Instead of waiting for the entire response body to arrive, you can read it as a `ReadableStream` and process chunks as they come in over the network. This is useful when you want to handle large responses incrementally or track download progress for the user.

Before reading the stream, confirm that `response.bodyUsed` is `false`. Then call `getReader()` on `response.body` to get a reader, and call `read()` on the reader to receive chunks. Each call to `read()` returns a Promise that resolves to an object with two properties:

- `done`: `true` when the stream is exhausted
- `value`: the next chunk of data, or `undefined` when complete

Avoid the streaming API with raw Promises. Prefer `async` and `await` for readable control flow.

The following example streams a large text response, logging each chunk and tracking the total bytes received:

```js
const response = await fetch('/api/large-file');

if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
}

const reader = response.body.getReader();
const decoder = new TextDecoder();
let received = 0;

while (true) {
    const { done, value } = await reader.read();

    if (done) break;

    received += value.length;
    console.log(`Received ${received} bytes`);
    console.log(decoder.decode(value, { stream: true }));
}
```

### Uploading files

To upload a file, add an `<input type="file">` element to your HTML and attach a `change` event listener. The input's `files` property exposes a `FileList` where each item is a `File` object. To send the file to a server, append it to a `FormData` object and pass that as the `fetch()` request body. This pattern is common for profile photo uploads, document attachments, and other user-submitted binary content.

The process follows four steps:

1. Get the uploaded file from `fileInput.files[0]`.
2. Create a `FormData` object.
3. Append the file to the `FormData` object.
4. Send the request with `fetch()`.

The following example listens for a file selection, validates that a file was chosen, and sends it to an upload endpoint:

```js
let fileInput = document.querySelector('#myfile');

fileInput.addEventListener('change', (e) => {
    if (fileInput.files.length === 0) {             // handle no file when form submitted
        message.textContent = "Upload a file!";
        return;
    }

    const file = fileInput.files[0];                // get uploaded file

    let formData = new FormData();                  // create new FormData obj
    formData.append(file.name, file);               // add uploaded file to formData obj

    fetch('path/to/upload', {                       // send file w/fetch
        method: "POST",
        body: formData
    });
});
```

### Cross-origin requests

An *origin* is the combination of a URL's protocol, host, and port, such as `https://example.com:8080`. A *same-origin request* is one where the requesting page and the target server share the same origin. By default, browsers block *cross-origin requests*, which target a server with a different origin than the page making the request.

*Cross-Origin Resource Sharing* (CORS) enables safe cross-origin requests. When you make a cross-origin `fetch()` call, the browser automatically adds an `Origin` header that you cannot override. This tells the target server where the request originated. The server's response must include an `Access-Control-Allow-Origin` header that permits the requesting origin. If that header is absent, `fetch()` rejects the Promise.

### Aborting a request

Abort a `fetch()` call when it takes too long or when a user action (e.g., clicking "Cancel") triggers a cancellation. An `AbortController` manages this: its read-only `signal` property is an `AbortSignal` object you pass to `fetch()`. Calling `abort()` on the controller cancels the in-flight request and rejects the Promise with an `AbortError`.

1. Create an `AbortController` object.
2. Pass `controller.signal` as the `signal` property in the `fetch()` options object.
3. Call `controller.abort()` when you want to cancel the request, for example inside a `setTimeout()` callback or a button click handler.

The following example cancels a fetch request if it does not complete within 5 seconds:

```js
const controller = new AbortController();
const timeoutId = setTimeout(() => controller.abort(), 5000); // 5-second timeout

fetch(url, {
    method: 'GET',
    signal: controller.signal // Attach the AbortController signal
})
    .then(response => response.json())
    .then(json => console.log('Success:', json))
    .catch(error => {
        if (error.name === 'AbortError') {
            console.error('Request timed out');
        } else {
            console.error('Fetch error:', error);
        }
    })
    .finally(() => clearTimeout(timeoutId));
```

### Retry with exponential backoff

Transient failures, such as a momentarily overloaded server, and rate-limit responses (`429`) often succeed on a retry. Exponential backoff doubles the wait time between attempts to avoid overwhelming a struggling server:

```js
async function fetchWithRetry(url, options = {}, maxRetries = 3, baseDelay = 500) {
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
        try {
            const res = await fetch(url, options);

            if (res.ok) return res;

            // Retry on rate limit or server error — not on client errors (4xx)
            if (res.status === 429 || res.status >= 500) {
                if (attempt === maxRetries) throw new Error(`HTTP ${res.status} after ${maxRetries} retries`);
                const delay = baseDelay * 2 ** (attempt - 1);   // 500ms, 1s, 2s…
                await new Promise(r => setTimeout(r, delay));
                continue;
            }

            throw new Error(`HTTP ${res.status}`);  // 4xx — do not retry
        } catch (err) {
            if (attempt === maxRetries) throw err;
            await new Promise(r => setTimeout(r, baseDelay * 2 ** (attempt - 1)));
        }
    }
}

// Usage
try {
    const res = await fetchWithRetry('/api/reports', { method: 'GET' });
    const data = await res.json();
} catch (err) {
    console.error('Request failed after all retries:', err.message);
}
```

### Centralized API wrapper

Instead of repeating headers, base URLs, and `response.ok` checks on every call, wrap `fetch` in a single function. Every call site becomes one line:

```js
const api = (() => {
    const BASE = 'https://api.example.com/v1';

    async function request(method, path, body = null) {
        const token = localStorage.getItem('auth_token');

        const res = await fetch(`${BASE}${path}`, {
            method,
            headers: {
                'Content-Type': 'application/json',
                ...(token && { 'Authorization': `Bearer ${token}` }),
            },
            body: body ? JSON.stringify(body) : null,
        });

        if (!res.ok) {
            // Try to read an error message from the response body
            const err = await res.json().catch(() => ({ message: res.statusText }));
            throw new Error(err.message ?? `HTTP ${res.status}`);
        }

        return res.status === 204 ? null : res.json();  // 204 No Content has no body
    }

    return {
        get:    (path)        => request('GET',    path),
        post:   (path, body)  => request('POST',   path, body),
        put:    (path, body)  => request('PUT',    path, body),
        patch:  (path, body)  => request('PATCH',  path, body),
        delete: (path)        => request('DELETE', path),
    };
})();

// Clean call sites — no headers, no ok checks, no base URL
const users = await api.get('/users');
const created = await api.post('/users', { name: 'Alice', role: 'admin' });
await api.delete('/users/42');
```

## Server-sent events

Web apps sometimes need the server to push notifications to the client without the client polling repeatedly. HTTP is a request-response protocol, so server-push requires a special approach. The *EventSource* API handles it by opening a persistent HTTP connection and receiving a continuous stream of events from the server. If the connection drops, the browser reconnects automatically.

The following example connects to a live notifications endpoint and logs each incoming event:

```js
const source = new EventSource('/api/notifications');

source.addEventListener('message', (event) => {
    console.log('New notification:', event.data);
});

source.addEventListener('error', () => {
    console.error('SSE connection lost — browser will reconnect automatically');
});

// Close the connection when it is no longer needed
source.close();
```

## WebSockets

WebSockets let JavaScript in the browser exchange text and binary messages with a server in both directions, unlike Server-Sent Events, which only flow from server to client. The WebSocket protocol starts with an HTTP handshake: the browser sends an `Upgrade: websocket` header, and once the server accepts, the connection switches to the WebSocket protocol. Both sides can then send messages freely. The server must be configured to handle WebSocket connections.

WebSocket URLs use the `ws://` scheme for unencrypted connections and `wss://` for connections over TLS.

The following example opens a WebSocket connection to a chat server, joins a room, and handles incoming messages:

```js
const socket = new WebSocket('wss://chat.example.com/room/42');

socket.addEventListener('open', () => {
    console.log('Connected to chat server');
    socket.send(JSON.stringify({ type: 'join', username: 'alice' }));
});

socket.addEventListener('message', (event) => {
    const message = JSON.parse(event.data);
    console.log(`${message.username}: ${message.text}`);
});

socket.addEventListener('close', () => {
    console.log('Disconnected from chat server');
});

socket.addEventListener('error', (event) => {
    console.error('WebSocket error:', event);
});
```
