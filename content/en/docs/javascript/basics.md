---
title: "Web programming basics"
linkTitle: "Basics"
weight: 10
description:
---

## HTML <script\> tags

The `<script>` tag includes the JS in the HTML page so the browser can execute it.
- JS can be inline between the `<script>` and `</script>` tags
- More common to use the `src` attribute in the `<script>` tag to specify the URL of a JS file. Still requires the `</script>` closing tag
  - Separates content and behavior
  - Create a single JS page that can be reused for multiple HTML files
  - JS file is only downloaded once and subsequently can be retreived from the cache
  - `src` takes an arbitrary URL - you can import JS code that is exported by other web servers
- JS files use `.js` extension

### `type` attribute

There are two reasons to use the `type` attribute:
- specify the script as a module
- embed data into the web page without displaying it

Historically, people used `type="application/javascript"`, but that has been deprecated for a long time.

#### Modules

If you are using modules and NOT using a code bundler that combines all modules into a single JS file, you have to load the top-level JS file with the `type=module` attribute:

```html
<script src="index.js" type="module"></script>
```
This loads the top-level module, which loads all dependent modules.

### `async` and `defer`

When the HTML parser encounters a `<script>` tag, it has to run the script before it continues to make sure it doesn't output any HTML
- Called _synchronous_ or _blocking_ script execution
- This is because when JS first came out, there was no API for manipulating the DOM - JS had to generate content while the document was loading, using the `document.write()` method to inject text into the HTML doc.
- `document.write()` is bad style
- This slows down page loads

Can include boolean `defer` or `async` attributes:

```html
<script defer src="index.js"></script>
<script async src="index.js"></script>
```

These attributes tell the browser that the HTML document does not use `document.write()`, so it can continue parsing the HTML doc:
- `defer`: Execute the script after the document is fully loaded and parsed and is ready to be manipulated
- `async`: Execute the script ASAP, but do not block parsing to download the script
- If both are present, `async` takes precedence over `defer`
- Scripts run in the order that they are listed in the HTML doc, so async scripts might run out of order
- Module scripts:
  - run as `defer`, by default
  - If you add `async`, the code executes as soon as the module and its dependencies are loaded
- You can just put the `<script>` tag at the end of the doc to get the same behavior as `defer` and `async`
  - Use these if you have to load scripts in the head

## DOM

Document Object Model is the API for working with the Document object
- Document object represents the HTML doc displayed in the browser
- HTML documents contain nested HTML elements formed in a tree
- DOM API mirrors tree structure of HTML doc
  - For each HTML element, there is a JS object
  - For each string of text, there is a text object
  - These elements are classes, and they are subclasses of the Node class
  - JS can query and traverse the Nodes with the DOM API
  - Node trees use familial language - parent, child, sibling, descendant, ancestors
- DOM API can create Element and Text nodes and insert them in relation to other Element objects
- There is a JS class for each HTML tag type
  - Ex: HTMLBodyElement class, HTMLTableElement class
  - There is an instance of the JS class - called a JS element object - for each occurrence of the HTML tag in the document
  - Each JS element object has properties that correspond to HTML tag attributes
  - Some JS classes define attributes that are not available on the HTML tag

## Global object

There is one global object per browser or tab, and all JS code running in the window or tab shares the same global object:
- JS's standard library is defined on the global object, and it is the entrypoint for some web APIs, such as `document` and `fetch()`
- In web browsers, the global object is also the `window` object, which represents the current web browser window
  - Best practice to use `window.` prefix when calling the global object. Ex: `window.innerWidth`


## Namespaces

Modules: constants, variables, funcs, and classes defined in a module are private and need to be explicitly exported, and then can be imported by another module

Non-modules: All scripts share a namespace and can share vars, funcs, etc. 
- Be careful with naming conflicts
- `var` and `function` declarations create shared global object properties. This means that you can invoke them with `window.<function>`
- ES6 `let`, `const`, and `class` do not create properties on the global object, but still be mindful of namespaces

## Program execution

A JS program is all JS code in or referenced from a document that shares a global Window object
- non-module scripts also share a top-level namespace
- an `<iframe>` has a different global Window object and Document object, so its a separate program
  - If the container and contained document are on the same server, they can communicate with each other

### First phase

Load JS content:
- Document content is loaded
- This stage should take less than a second
- Code in inline and external `<script>` elements are run in the order they appear in the document, taking into account `defer` and `async` attributes.
  - Each script is run from top to bottom
- Some scripts just define functions and classes for the second phase
  - Ex: Register event handlers or callbacks

Detailed breakdown:
1. Browser creates a Document object and parses the web page. Adds Element objects and Text nodes as it parses the HTML.
   - `document.readyState` is `loading`
2. HTML parser adds to the document any `<script>` tags without `defer`, `async`, or modules. Can use `document.write()` to maniupulate the DOM, but these scripts generally just register event handlers
3. If the HTML parser encounters an `async` `<script>` tag, it downloads the script and continues parsing the document.
   Do not use the `document.write()` method with this event
4. `document.readyState` changes to `interactive`
5. `defer` scripts are executed in the order they are encountered in the document. They have access to the complete document - but do NOT use `document.write()`. Async scripts might also be executed.
6. `DOMContentLoaded` event is fired on the Document object. This begins the transition to phase 2. `async` scripts might still be executed.
7. Document is completely loaded, but might be waiting on images or other content. After all content is loaded and `async` scripts are loaded `document.readyState` is changed to `complete` and the browser fires a `load` event on the Window object.
8. Completely in second phase, event handlers are invoked asynchronously.

### Second phase

Asychronous and event-driven:
- In response to events, the browser executes event handlers and callbacks that were registed in the first phase
- This phase lasts as long as the document is displayed in the browser
- Event examples:
  - mouse clicks, keystrokes
  - network activity
  - document resource loading
  - elapsed time
  - errors in JS code

First events to occur are `DOMContentLoaded` and 'load' events:
- These events are used as a trigger or starting signal for JS actions like registering handlers on the `load` event.

| **Event**           | **When It Fires**  | **Use Case**    |
|---------------------|:-------------------|:----------------|
| `DOMContentLoaded`  | After HTML is parsed, before full page load | Run JS that doesn’t depend on images or CSS  |
| `load`             | After the entire page (CSS, images, etc.) loads | Initialize app after all resources load     |
| `pageshow`        | Similar to `load`, also fires on back/forward cache | Detect when page is restored from cache    |
| `beforeunload`     | When the user is about to leave | Show warnings or save data                 |
| `unload`           | When the page is closing       | Clean up resources (e.g., logs, API calls) |
| `visibilitychange` | When the page is hidden or visible | Pause/resume background tasks             |

```js
window.addEventListener('DOMContentLoaded', () => alert('DOMContentLoaded'));
document.addEventListener('DOMContentLoaded', () => alert('DOMContentLoaded'));

// document doesn't have the `load` event, only window does
window.addEventListener('load', () => alert('DOMContentLoaded'));
```

### Threading model

JS is single-threaded:
- there are no locks, deadlocks, race conditions
- no two event handlers can execute at the same time
- the browser does not respond to user input when scripts and event handlers are executing, so you can't write code that runs too long

Web worker - a controlled form of concurrency
- background thread that can perform tasks without freezing the UI
- can't access the document, does not share its state with other workers or main thread
- communicates with the main thread and other workers through asynchronous message events

## Input/Output

JS takes the following inputs:
- Document, which JS accesses with the DOM API
- User input - mouse clicks, keyboard, text, etc
- URL of document being displayed, avaialble with `document.URL`
- HTTP cookie req header with `document.cookie`. Cookies are usually server side, but JS can read/write them in browser
- Global `navigator` property - gives info about the web browser:
  - `navigator.userAgent`
  - `navigator.language`
  - `navigator.hardwareConcurrency`
- Global `screen` property - info about user's display size
  - `screen.width`
  - `screen.height`
- `navigator` and `screen` objects are like env vars

Produces output:
- In the DOM
- In the console, but this is for debugging

## Errors

JS programs don't crash, they just don't do what they're supposed to and then log errors to the console:
- You can set a few properties on the `window` object to handle errors. Mostly useful for telemetry

```js
// log errors and unexpected failures
window.onerror()

// when a Promise is rejected and there is no .catch() 
window.onunhandledrejection()
window.addEventListener('unhandledrejection', function(e) {...})
```

## Web security

### Restrictions

- JS cannot read or write to the filesystem
- JS cannot access general-purpose networks. JS can only make HTTP reqs and use Websockets

### Same-origin policy

A JS script can only read properties of windows and documents that share the same origin:
- **origin** - protocol, host, and port of the URL that loaded the document
- origin of the document that the script is embedded in, not the script itself
  - If Host A serves a web page with a script loaded from Host B, then the script origin is Host A
- Different web server = different origin
- Different scheme = different origin
- Different port on same server = different origin
- iframes can't read properties of the page hosting them

Applies to HTTP reqs too:
- By default, JS can make HTTP reqs to the web server that loaded the document, but cannot make reqs to other web servers unless you use CORS or set the `document.domain`
  - `document.domain`: when a site has multiple subdomains (_docs.example.com_, _support.example.com_, _example.com_), the different sites might need to access properties from other subdomains. A script with the _docs.example.com_ origin can set `document.domain` to `example.com` to access those files.
  - CORS: **Cross-Origin Resource Sharing**. Lets a server decide which origins they can serve.
    - Adds `Origin` request header that lists origins they will support
    - Adds `Access-Control-Allow-Origin` response header

### Cross-site scripting

When an attacker injects HTML tags or scripts into your website:
- If you dynamically generate content based on user input, then you must sanitize it by removing any embedded HTML tags, or you are vulnerable
- Called 'cross-site' because more than one site is involved:
  - The site that injects HTML might get users to click on something
  - Then the site runs code from the malicious site. Then they can ready cookie info or track keystrokes, among other things
- Prevention options:
  - Remove HTML tags from untrusted data before you create dynamic content
  - Always display untrusted content in an iframe with the `sandbox` attribute set. This disables scripting and other things

