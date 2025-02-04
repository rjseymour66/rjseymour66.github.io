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