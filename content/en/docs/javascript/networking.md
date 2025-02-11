---
title: "Networking"
# linkTitle: ""
weight: 60
description:
---

All webpages are loaded with HTTP or HTTPS requests, and JS exposes its own APIs so you can use networking in your app.

## fetch()

Easy-to-use, promise-based HTTP and HTTPS requests with features that supports any HTTP use case:
- Replaces the XMLHttpRequest API, which is sometimes called "XHR".

All `fetch()` requests are three-step process:
1. Call `fetch()` with the URL of the content you want to retrieve
2. Step 1 asynchronously returns a response object, so you need to use methods on the response object to get the response body
   - This requires two `.then()` calls or two `await` expressions
3. Process the response body however you need to


`fetch()` can accept a url and request properties in a few different ways that include raw strings, an Options object, and a Request object:
- `fetch('url-string')`
- `fetch(URL obj)`
- `fetch(URL obj, {Options-obj-props})`
- `fetch({Request-obj})`

```js
// --- single arg version --- //
let url = 'https://jsonplaceholder.typicode.com/posts/3';

fetch(url)                                      // 1
    .then(response => response.json())          // 2
    .then(json => console.log(json.title));     // 3


// --- Options object version --- //
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


// --- multiple arg version ex 1 --- //
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

### Request object

You can create a Request object that includes the URL and an an object of request properties:


```js
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

When you send a POST request, you usually send the request body as a set of name/value pairs (a JSON object):
- you can also send the values as a `URLSearchParams()` object with the content type set to `application/x-www-form-urlencoded`

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

| **Property**       | **Description** |
|-------------------|----------------|
| `method`         | HTTP method (`GET`, `POST`, `PUT`, etc.) |
| `url`            | The request URL |
| `headers`        | A `Headers` object containing request headers |
| `body`           | The request body (only for methods like `POST`, `PUT`) |
| `bodyUsed`       | `true` if the request body has already been read |
| `referrer`       | The referrer URL (or `"about:client"` if not set) |
| `referrerPolicy` | The policy for referrer information (e.g., `"no-referrer"`) |
| `mode`           | Request mode (`"cors"`, `"no-cors"`, `"same-origin"`) |
| `credentials`    | How credentials (cookies/auth) are handled (`"omit"`, `"same-origin"`, `"include"`) |
| `cache`         | Cache mode (`"default"`, `"no-store"`, `"reload"`, etc.) |
| `redirect`       | Redirect mode (`"follow"`, `"manual"`, `"error"`) |
| `integrity`      | Subresource Integrity (SRI) hash |
| `keepalive`      | `true` if the request should outlive the page |
| `signal`         | `AbortSignal` to cancel the request |


### URL API

Pg 320

### Headers

Response Headers are in a Headers object
- header names are case-insensitive
- `has()`: Boolean, tests for presence of a header
- `get()`: value of a header 

```js
// iterate through all headers
let logResponseHeaders = responseObject => {
    for (let [name, value] of responseObject.headers) {
        console.log(`Header "${name}": ${value}`);
    }
};
```

If you specify a request body (PUT or POST), the browser automatically adds a few headers:
- "Content-Length" header
- "Content-type" header that defines the content type as `text-plain; charset=UTF-8`
- If you send HTML or JSON, add the correct `text/html` or `application/json` content type header

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

You can add path parameters to a URL:

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

You can either use `fetch()` with multiple arguments (url string or object and a Headers object), or you can create a `Request` object and pass it to `fetch()`:
- `.set()` method takes two args: a header name and header value


```js
let authHeaders = new Headers();                                                // create Headers obj
authHeaders.set('Authorization', `Basic ${btoa(`${username}:${password}`)}`);   // .set() method

fetch('https://jsonplaceholder.typicode.com/posts',
    { headers: authHeaders })
    .then(response => response.json())
    .then(json => console.log(json.body));
```

### Response object

`fetch()` returns a Promise that resolves to a Response object:
- resolves the Promise when the response starts to arrive - at least the status and headers are available, but maybe not the body
- Only rejects the Promise when one of these occur:
  - user computer is offline
  - server is unresponsive
  - URL hostname does not exist
- For these reasons, you should always include a `.catch()` clause with `fetch()`

Here is an example with error handling:
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

| Property    | Description |
|:-------------|:-------------|
| `ok`        | `true` if status is 200–299 |
| `status`    | HTTP status code (e.g., `200`, `404`) |
| `statusText` | HTTP status message (e.g., `"OK"`) |
| `headers`   | Response headers (`Headers` object) |
| `url`       | Final URL after redirects |
| `redirected` | `true` if redirected |
| `type`      | Response type (`"cors"`, `"basic"`, `"opaque"`, etc.) |
| `body`      | Raw response body (`ReadableStream`) |
| `bodyUsed`  | `true` if the response body has been read - e.g. if you used `.json()` or `.text()` to read the body |

#### Response object methods

There are multiple methods that return the Response body:
- `json()`: returns response as a parsed JSON object
- `text()`: returns response as a string of text
- `arrayBuffer()`: good for binary data - this returns a Promise that resolves to an ArrayBuffer, which you can turn into a DataView object to read the binary data
- `blob()`: returns a Blob object - *B*inary *L*arge *Ob*ject. Good for large amounts of binary data
  - browser might stream the resp to a temporary file then return a Blob that represents that temp file
  - you can't randomly access Blob data
  - Use the Blob to create a URL that refers to it with `URL.createObjectURL()` or use FileReader API to get the Blob contents as a string or ArrayBuffer
- `formData()`: returns Promise that resolves to a FormData() object. Use for bodies encoded in "multi-part/form-data" format.
  - Common in POST requests.

##### Streaming response bodies

You can also stream the response body, which is a ReadableStream object:
- good if there is processing you can do on chunks of the resp body as they arrive over the network
- also good to show users a progress bar for download progress
- if `response.bodyUsed` returns `false`, you can call `getReader()` on `response.body`, then `read()` method to read the chunks of text
- `read()` returns a Promise with `done` and `value` properties
  - `done` returns true when you reach the end of the stream
  - `value` contains the next chunk of data or `undefined` when complete
- Avoid using the streaming API with raw Promises - use `async` and `await`  