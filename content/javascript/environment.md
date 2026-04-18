---
title: "Programming environment"
linkTitle: "Environment"
weight: 20
description:
---

## Links

- [DOM tree and nodes](https://www.digitalocean.com/community/tutorials/understanding-the-dom-tree-and-nodes#html-terminology)
- [Javascript DOM tutorial](https://www.javascripttutorial.net/javascript-dom/)
- [Eloquent Javascript "The Document Object Model"](https://eloquentjavascript.net/14_dom.html)
- [DOM Enlightenment](https://domenlightenment.com/)

## HTML <script\> tags

The `<script>` tag tells the browser where to find JavaScript and how to load
it. You can write JavaScript inline between opening and closing `<script>` tags,
but the more common approach is to reference an external file with the `src`
attribute. Even when you specify `src`, the closing `</script>` tag is still
required. External scripts separate content from behavior, can be shared across
multiple HTML files, and are downloaded once and retrieved from cache on
subsequent page loads. The `src` attribute accepts any URL, so you can also load
scripts hosted on other servers. JavaScript files use the `.js` extension.

### `type` attribute

The `type` attribute has two practical uses:

- **Declare a module:** Mark the script as an ES module so the browser applies
  module scoping and import rules.
- **Embed hidden data:** Include structured data in the page without rendering
  it. This is common in server-rendered apps that need to pass data to the
  client without a separate HTTP request. For example:

```html
<script type="application/json" id="config">
  { "userId": 42, "theme": "dark" }
</script>
```

```js
const config = JSON.parse(document.getElementById('config').textContent);
```

Historically, developers set `type="application/javascript"`, but that value
has been deprecated. Omit the attribute unless you are declaring a module.

#### Modules

Without a bundler that combines all modules into a single file, declare your
entry point with `type="module"` so the browser resolves its dependencies:

```html
<script src="index.js" type="module"></script>
```

This loads the top-level module and fetches all dependent modules automatically.

### `async` and `defer`

When the HTML parser encounters a `<script>` tag, it stops parsing and runs
the script immediately. This is called *synchronous* or *blocking* script
execution. JavaScript originally required this behavior because `document.write()`
was the only way to inject content during page load. Today, `document.write()`
is considered bad practice, and blocking execution slows page loads unnecessarily.

You can attach the `defer` or `async` boolean attributes to any external script
to let the parser continue while the script downloads:

```html
<script defer src="analytics.js"></script>
<script async src="widget.js"></script>
```

Both attributes signal to the browser that the page does not rely on
`document.write()`. Their behavior differs in when the script executes:

- **`defer`:** Executes the script after the document is fully parsed. Scripts
  run in document order, making `defer` the right choice for your own
  application code that needs to query or modify the DOM.
- **`async`:** Downloads the script without blocking parsing and executes it
  as soon as it is available, regardless of document order. Use `async` for
  independent third-party scripts like analytics or ad widgets that do not
  depend on your application code.
- **Precedence:** If both attributes are present, `async` takes precedence.

Module scripts defer by default. Adding `async` to a module causes it to
execute as soon as the module and all its dependencies finish loading,
regardless of document order.

If you cannot add `defer` or `async`, placing the `<script>` tag at the bottom
of `<body>` produces equivalent behavior. Reserve `defer` and `async` for
scripts that must load in `<head>`.

## DOM

The *Document Object Model* (DOM) is the API for working with the Document
object, which represents the HTML page displayed in the browser.

HTML documents consist of nested elements arranged in a tree. The DOM mirrors
this structure by representing each HTML element as a JavaScript object and
each string of text as a text object. Both types are instances of classes that
extend the base `Node` class. The DOM API lets you query and traverse these
nodes using familial terminology: parent, child, sibling, descendant, and
ancestor.

The DOM API also lets you create new `Element` and `Text` nodes and insert them
relative to existing elements, making it the primary tool for dynamic content
manipulation.

JavaScript provides a dedicated class for each HTML tag type. For example,
`<body>` maps to `HTMLBodyElement` and `<table>` maps to `HTMLTableElement`. For
each occurrence of a tag in the document, the browser creates an instance of the
corresponding class, called an *element object*. Each element object exposes
properties that map to the tag's HTML attributes. Some classes also expose
additional properties that have no direct HTML attribute equivalent.

The following example illustrates all three of these concepts using an
`<input>` element:

```js
const input = document.querySelector('input[name="email"]');

console.log(input instanceof HTMLInputElement); // true
console.log(input.type);     // "email" — maps to the HTML `type` attribute
console.log(input.validity); // ValidityState — no direct HTML attribute equivalent
```

## BOM

The *Browser Object Model* (BOM) is the API that lets JavaScript communicate
with the browser itself, outside the document. It includes four core objects:

- **Window:** The global browser window and the root of the BOM
- **History:** The browser's session history for the current tab
- **Navigator:** Information about the browser and the device running it
- **Location:** The URL of the current page

To inspect all BOM properties, pass `window` to `console.dir()`. You can then
access any property with dot notation:

```js
console.dir(window)       // inspect all BOM properties
window.history.length     // number of entries in the session history
window.document           // the DOM document for the current page
window.history.go(-1)     // go back one page in history
```

### Window `navigator`

The `navigator` object contains information about the browser, including its
name, version, and the operating system running it. It is globally available,
so you can reference it without the `window.` prefix:

```js
console.dir(window.navigator)
// navigator is globally available
console.dir(navigator)
```

### Window `location` object

The `location` object holds the URL of the current page. You can read individual
URL components or assign a new value to navigate the browser to a different page:

```js
console.dir(window.location)
// location is globally available
console.dir(location)
```

For example, assigning `location.href` redirects the user immediately:

```js
location.href = '/login'; // redirects to the login page
```

## Global object

Each browser tab has exactly one global object. All JavaScript code running in
that tab shares it. JavaScript's standard library is defined on the global
object, and it serves as the entrypoint for core web APIs such as `document`
and `fetch()`.

In web browsers, the global object is also the `window` object, which
represents the current browser window. The best practice is to access global
properties with the `window.` prefix to make the scope explicit. For example,
`window.innerWidth` returns the width of the browser viewport in pixels.

## Namespaces

In a *module*, every constant, variable, function, and class is private by
default. To share code between modules, you must explicitly export it and import
it in the consuming module.

In a non-module script, all scripts on the page share a single namespace, which
creates the risk of naming conflicts. Declarations made with `var` and `function`
become properties of the global object, making them callable with the `window.`
prefix. For example:

```js
var greeting = 'hello';
function sayHi() { return greeting; }

console.log(window.greeting); // "hello"
console.log(window.sayHi());  // "hello"
```

ES6 declarations (`let`, `const`, and `class`) do not create properties on the
global object, but naming conflicts can still occur across scripts. Prefer
modules to avoid this problem entirely.

## Program execution

A JavaScript program consists of all JavaScript code in or referenced from a
document that shares a global `Window` object. Non-module scripts also share a
top-level namespace. An `<iframe>` has its own `Window` and `Document` objects,
making it a separate program. If the containing page and the embedded document
are served from the same origin, the two programs can communicate with each
other.

### First phase

The first phase loads and runs all JavaScript content. The browser processes
inline and external `<script>` elements in document order, accounting for
`defer` and `async` attributes, running each script from top to bottom. This
phase typically completes in under a second. Many scripts in this phase do
nothing except define functions and classes, or register event handlers and
callbacks for the second phase.

The following steps describe the first phase in detail:

1. The browser creates a `Document` object and begins parsing the page, adding
   `Element` objects and `Text` nodes as it encounters HTML.
   `document.readyState` is `"loading"`.
2. The HTML parser executes any `<script>` tags without `defer`, `async`, or
   `type="module"` immediately. These scripts can call `document.write()`, but
   most just register event handlers.
3. When the parser encounters an `async` `<script>`, it downloads the script in
   the background and continues parsing. Do not call `document.write()` in an
   async script.
4. `document.readyState` changes to `"interactive"`.
5. `defer` scripts execute in document order. They have full access to the
   parsed document. Do not call `document.write()` in deferred scripts. Any
   remaining `async` scripts may also execute at this point.
6. The browser fires the `DOMContentLoaded` event on the `Document` object,
   marking the transition to the second phase. Some `async` scripts may still
   be running.
7. The document is fully parsed, but may still be waiting on images and other
   resources. Once all resources finish loading, `document.readyState` changes
   to `"complete"` and the browser fires a `load` event on the `Window` object.
8. The second phase begins. Event handlers are invoked asynchronously in
   response to user and browser events.

### Second phase

The second phase is asynchronous and event-driven. In response to events, the
browser executes the handlers and callbacks registered during the first phase.
This phase continues for as long as the document is open. Events that trigger
callbacks include:

- Mouse clicks and keystrokes
- Network activity
- Document resource loading
- Elapsed time (timers)
- JavaScript errors

The first two events of the second phase are `DOMContentLoaded` and `load`.
Both serve as starting signals for JavaScript initialization. The following
table summarizes the key page lifecycle events:

| **Event** | **When it fires** | **Use case** |
| --- | --- | --- |
| `DOMContentLoaded` | After HTML is parsed, before full page load | Run JS that does not depend on images or CSS |
| `load` | After the entire page (CSS, images, etc.) loads | Initialize app after all resources load |
| `pageshow` | Similar to `load`, also fires on back/forward cache | Detect when page is restored from cache |
| `beforeunload` | When the user is about to leave | Show warnings or save data |
| `unload` | When the page is closing | Clean up resources (for example, logs, API calls) |
| `visibilitychange` | When the page is hidden or becomes visible | Pause or resume background tasks |

The following example registers handlers for both primary events:

```js
document.addEventListener('DOMContentLoaded', () => console.log('DOM ready'));
window.addEventListener('DOMContentLoaded', () => console.log('DOM ready (window)'));

// document doesn't have the `load` event — only window does
window.addEventListener('load', () => console.log('page fully loaded'));
```

### Real-world example: initializing UI components

`DOMContentLoaded` is the right place to attach event listeners and query the
DOM. Use `load` only when you depend on images or stylesheets being available:

```js
document.addEventListener('DOMContentLoaded', () => {
    const form = document.querySelector('#signup-form');
    const status = document.querySelector('#status-msg');

    form.addEventListener('submit', async (e) => {
        e.preventDefault();
        const email = form.elements['email'].value.trim();

        if (!email.includes('@')) {
            status.textContent = 'Enter a valid email address.';
            return;
        }

        status.textContent = 'Submitting...';
        try {
            const res = await fetch('/api/signup', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ email }),
            });
            if (!res.ok) throw new Error('Server error');
            status.textContent = 'Success! Check your inbox.';
        } catch {
            status.textContent = 'Something went wrong. Try again.';
        }
    });
});
```

### Threading model

JavaScript is single-threaded. Only one task executes at a time, which
eliminates locks, deadlocks, and race conditions. No two event handlers ever
run simultaneously. Because the browser cannot respond to user input while a
script is executing, long-running scripts block the UI. Keep individual tasks
short.

*Web workers* provide a controlled form of concurrency. A worker runs in a
background thread and can perform compute-heavy tasks without freezing the UI.
Workers cannot access the DOM and do not share state with other workers or the
main thread. They communicate through asynchronous message events:

```js
// main.js
const worker = new Worker('worker.js');
worker.postMessage({ data: largeArray });
worker.onmessage = (e) => console.log('Result:', e.data.result);

// worker.js
self.onmessage = (e) => {
    const result = heavyComputation(e.data.data);
    self.postMessage({ result });
};
```

## Input/Output

JavaScript reads from several browser-provided inputs:

- **Document:** The HTML page itself, accessed through the DOM API
- **User input:** Mouse clicks, keyboard events, and text entry
- **Current URL:** Available as `document.URL`
- **HTTP cookies:** Readable and writable via `document.cookie`. Cookies are
  typically managed server-side, but JavaScript can read and write them in
  the browser.
- **`navigator` object:** Provides information about the browser and device,
  including:
  - `navigator.userAgent`
  - `navigator.language`
  - `navigator.hardwareConcurrency`
- **`screen` object:** Provides information about the user's display, including:
  - `screen.width`
  - `screen.height`

The `navigator` and `screen` objects behave like environment variables: they
expose read-only context about the runtime that your code can inspect but not
change. For example, you can read `navigator.language` to serve localized
content or check `navigator.hardwareConcurrency` to decide how many Web Workers
to spawn:

```js
if (navigator.language.startsWith('fr')) {
    loadContent('fr');
}

const workerCount = Math.min(navigator.hardwareConcurrency, 4);
```

JavaScript produces output primarily by modifying the DOM. Console output is
available but intended for debugging, not user-facing content.

## Errors

JavaScript programs do not crash the way native applications do. When an error
occurs, the program stops executing the current task, logs the error to the
console, and continues running. You can assign handler functions to properties
on the `window` object to intercept errors programmatically, which is most
useful for telemetry and error reporting services.

Assign a function to `window.onerror` to catch uncaught runtime errors and
unexpected failures:

```js
window.onerror = function(message, source, line, col, error) {
    reportToTelemetry({ message, source, line, col, error });
};
```

To catch unhandled Promise rejections, assign to `window.onunhandledrejection`
or register a listener with `addEventListener`:

```js
window.onunhandledrejection = (e) => console.error('Unhandled rejection:', e.reason);

window.addEventListener('unhandledrejection', function(e) {
    reportToTelemetry({ reason: e.reason });
});
```

## Web security

### Restrictions

JavaScript runs in a security sandbox with two core restrictions. First, it
cannot read or write to the filesystem. Second, it cannot access
general-purpose networks. JavaScript can only communicate over HTTP requests
and WebSockets.

### Same-origin policy

A JavaScript script can only read properties of windows and documents that
share the same *origin*. An origin is the combination of protocol, host, and
port from the URL that loaded the document. The origin is determined by the
document the script is embedded in, not the script's own URL. If Host A serves
a page that loads a script from Host B, that script's origin is Host A.

Each of the following conditions produces a different origin:

- A different web server
- A different scheme (for example, `http` versus `https`)
- A different port on the same server

`<iframe>` elements cannot read properties of the page hosting them.

The same-origin policy also applies to HTTP requests. By default, JavaScript
can only make requests to the server that loaded the document. To communicate
with a different server, you must use one of two mechanisms.

**`document.domain`:** When a site spans multiple subdomains (for example,
_docs.example.com_, _support.example.com_, and _example.com_), scripts on one
subdomain may need to access properties on another. A script with the
_docs.example.com_ origin can set `document.domain` to `example.com` to share
access with the other subdomains.

**Cross-Origin Resource Sharing (CORS):** Lets a server declare which origins
it will accept requests from. The browser adds an `Origin` header to
cross-origin requests, and the server responds with an
`Access-Control-Allow-Origin` header to permit or deny access.

### Cross-site scripting

*Cross-site scripting* (XSS) occurs when an attacker injects HTML or JavaScript
into your website. It is called "cross-site" because the attack involves more
than one site: the attacker's site tricks users into triggering code that runs
in the context of your site, which can then read cookie data or log keystrokes.

If you dynamically generate content from user input, you must sanitize that
input. Without sanitization, an attacker can embed a `<script>` tag in a form
field and have it execute in other users' browsers.

Two prevention approaches are available:

- Remove HTML tags from all untrusted data before inserting it into the DOM.
- Display untrusted content inside an `<iframe>` with the `sandbox` attribute
  set. The `sandbox` attribute disables scripting and other potentially
  dangerous browser behaviors.

## Event loop

JavaScript is a single-threaded language. Only one thing can happen at a time:
tasks must wait for previously executing tasks to complete.

The single executor is called the *event loop*. The event loop executes all
JavaScript work. Even though JavaScript is single-threaded, it achieves
concurrency with the *call stack* and the *callback queue*.

### Call stack and callback queue

The call stack is a queue of all actions pending execution. The event loop
constantly monitors the call stack and completes pending tasks, one by one,
from the top of the stack.

When you use a callback, JavaScript outsources the callback task to the
browser's web API. When the callback completes, it moves into the callback
queue. When the call stack is empty, the event loop checks the callback queue
for pending work. If callbacks are waiting, they execute one by one from the
top of the queue. After each callback runs, the event loop checks the call
stack again before executing the next callback.

### Microtasks vs macrotasks

The callback queue has two tiers with different priorities:

- **Macrotasks** (`setTimeout`, `setInterval`, I/O, UI events): one runs per
  event loop turn.
- **Microtasks** (Promises, `queueMicrotask`, `MutationObserver`): the entire
  microtask queue drains before the next macrotask begins.

```js
console.log('1: sync');

setTimeout(() => console.log('4: macrotask'), 0);

Promise.resolve().then(() => console.log('3: microtask'));

console.log('2: sync');

// Output:
// 1: sync
// 2: sync
// 3: microtask    ← all microtasks run before the next macrotask
// 4: macrotask
```

This explains a common surprise: `await` resumes as a microtask, so code after
`await` runs before any pending `setTimeout` callbacks. This applies even to
`setTimeout(..., 0)`:

```js
async function run() {
    console.log('A');
    await Promise.resolve();
    console.log('C');   // microtask — runs before D
}

setTimeout(() => console.log('D'), 0);  // macrotask
run();
console.log('B');

// Output: A, B, C, D
```

Knowing this order matters when you are sequencing async work: prefer Promises
over `setTimeout` when you need something to run as soon as the current call
stack is clear.
