---
title: "Scripting documents"
linkTitle: "Scripting"
weight: 30
description:
---

JS was created to create interactive web applications from static HTML documents:
- Every Window object has a `document` property that references the Document object
- The Document object is the central object in the Document Object Model (DOM), which lets you manipulate document content

## Selecting Document Elements

CSS gives us _selectors_, which help us descibe elements or sets of elements within a document:

```js
div                             // any <div>
#nav                            // id="nav"
.warning                        // class="warning"
p[lang="fr"]                    // p with lang="fr" attribute
*[name="x"]                     // any element with name="x" attribute
span.fatal.error                // <span> with class="fatal error"
span[lang="fr"].warning         // <span> with lang="fr" and class="warning"
#log span                       // <span> descendant of element with id="log"
#log>span                       // <span> child of element with id="log"
body>h1:first-child             // first <h1> in the body
img + p.caption                 // <p class="caption"> immediately following an <img>
h2 ~ p                          // <p> after an <h2> and is a sibling of the <h2>
button, input[type=button]      // either a button element or <input type="button">
```
`querySelector()` and `querySelectorAll()` search for Element objects. You can find an element in the document that matches the CSS selector. These are the preferred selection functions:
- `querySelectorAll()` returns a NodeList, not an array of Element objects.
  - NodeList has a length property and is indexed, so `for` loops work
  - Need to convert to array with `Array.from()` method
- can invoke these methods on `document` or an element object to select a descendant of the object

```js
let header = document.querySelector('.one');            // get element with class="one"
console.log(header);

let titles = document.querySelectorAll('h1, h2, h3');   // get all h1, h2, h3 elements
titles.forEach(t => console.log(t));

// returns a NodeList
let titles = document.querySelectorAll('h1, h2, h3');
titles.forEach(t => console.log(t));
console.log(titles.length);

// convert to array to use array methods
let ar = Array.from(titles);

// call querySelector on Element object
let inner = document.querySelector('.inner-div');
let innerh2 = inner.querySelector('h2');
```

`event.target.closest('<element>)` element selector either matches the element object that it is invoked on, or it finds the parent of the element argument:
- If there are no matches, it returns `null`
- is almost the opposite of `querySelector()`

`event.matches('<element>)` returns a Boolean telling whether the event is of the type of element passed as the argument.

### Element selection methods

These methods are legacy alternatives to `querySelector` and `querySelectorAll`:
- They all return a NodeList, but they return 'live' NodeLists change if the document structure changes

```js
let a = document.getElementById('idname');          // get element with id="idname"
let b = document.getElementByName('nameAttr');      // document.querySelectorAll('*[name="nameAttr"]')
let c = document.getElementByTagName('h1')          // get all h1 elements
let d = document.getElementByClassName('container') // get elements with class="container"
```

## Document structure and traversal

There is a Traversal API that lets you access relatives (sibling, parent, etc) of an Element object. The Traversal API uses properties, not methods:
- Ignores Text Nodes