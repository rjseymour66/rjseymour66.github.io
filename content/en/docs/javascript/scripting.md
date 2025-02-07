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

### Ignore Text and Comment Nodes

```js
el.parentNode                       // Parent Element or Document object
el.children                         // NodeList of child Element objects
el.childElementCount                // Number of child elements, equal to children.length 
el.firstElementChild                // First child Element object, or null
el.lastElementChild                 // Last child Element object, or null
el.nextElementSibling               // Sibling immediately after Element object, or null
el.previousElementSibling           // Sibling immediately before Element object, or null
el.children[0].nextElementSibling   // Element immediately after first child of el
```

### Node property traversal

All Node objects, including Text and Comment nodes, have properties that you can traverse the document with:
- These properties are very sensitive to changes in the DOM

```js
el.parentNode               // Parent Node, or null for Document object
el.childNodes               // Read-only NodeList of all children of el
el.firstChild               // First child Node, or null
el.lastChild                // Last child Node, or null
el.nextSibling              // Sibling Node immediately after el Node, or null
el.previousSibling          // Sibling Node immediately before el Node, or null
el.nodeType                 // Number that specifies the node type:
                            // (1) Element, (3) Text, (8) Comment, (9) Document
el.nodeValue                // Text content of Text or Comment Node
el.nextSibling.nodeType     // HTML tag of an Element

// finding Text and Comment Nodes
function traverseDOM(node) {
    node.childNodes.forEach(child => {
        if (child.nodeType === Node.TEXT_NODE && child.nodeValue.trim() !== "") {
            console.log("Text Node:", child.nodeValue);
        }
        if (child.nodeType === Node.COMMENT_NODE) {
            console.log("Comment Node:", child.nodeValue);
        }
        traverseDOM(child); // Recursively check child nodes
    });
}

traverseDOM(document.body);
```

## Attributes

HTML elements are a tag name and a set of name/value pairs called attributes:
- YOu can use Element class methods to get, set, and test HTML attributes, but they are also available as properties of the HTMLEement object that represents the element
- Working with them as properties is often easier

### Element class methods

```js
el.getAttribute('class')            // get value of class attr
el.setAttribute('id', 'new-id');    // add new id="new-id" attr
el.hasAttribute('id')               // Boolean, whether el has id attr
el.removeAttribute('id');           // remove id attr from el
```

### Attributes as element properties

You can access and set attributes with dot (`.`) notation:
- cannot delete attributes, need `removeAttribute()` method
- some HTML attribute names map to properties that are named differently
  - `<input value="val">` is the JS property `defaultValue`
- If an HTML attr is more than one word long, convert it to camelCase to use it with JS
- If HTML attr is a JS reserved word, prefix with `html`
  - `<label for="val">` needs the JS property `htmlFor`
- Properties use string values, but Boolean and number properties use those types instead (i.e. Boolean uses bool)

```js
// set id attr
let myId = p.id;
console.log(myId === 'my-id');  // true

// set form attrs
let f = document.querySelector('.form');
f.action = 'https://www.example.submit';
f.method = 'POST';
```

### Class attribute

`class` is a reserverd word in JS, so use the `className` property to set and return the element class attribute:
- The value is a space-delimited list of class names, not a single class
- `classList` property returns the classes in an iterable Array-like object
  - `classList` has `add()`, `remove()`, `contains()`, and `toggle()` methods

```js
<div class="container new-class">
    ...
</div>

let div = document.querySelector('div');

div.classList.add('test-class');                        // adds class="test-class"
div.classList.remove('container');                      // removes class="container"
console.log(div.classList.contains('test-class'));      // true - checks if class is in classList
console.log(div.classList.toggle('test-class'));        // removes class="test-class"
console.log(div.classList.contains('test-class'));      // false
```

### Data attributes

HTML lets you add a custom attribute as long as you prefix it with `data-`:
- Called "dataset attributes"
- Element objects have a `dataset` property that contains properties that correspond to these `data-` attributes
  - hypenated data attrs use camelCase in JS
  - `dataset.myData` represents `data-my-data`

```js
<div data-my-data="custom attribute">
    ...
</div>

let div = document.querySelector('div');
console.log(div.dataset.myData);            // custom attribute
```

## Element content

### innerHTML for HTML content

You can set and return element content as a string of HTML or plain text:
- NEVER INSERT USER INPUT INTO THE DOCUMENT
- `innerHTML` returns the content as a string of HTML markup
  - Not efficient to append text because this method it has to serialize the HTML string as text, append the text, then parse it back into an HTML element
- `outerHTML` includes the enclosing element too when you get it. When you set it, it replaces the element entirely
- `insertAdjacentHTML()` lets you insert HTML markup adjacent to the specified element.
  - Takes two args:
    - first is where you insert the HTML (`beforebegin`, `afterbegin`, `beforeend`, `afterend`)
    - second is the HTML to insert

```js
p.innerHTML    // This is a <span>paragraph</span>.
p.outerHTML    // <p class="paragraph">This is a <span>paragraph</span>.</p>


const html = "<p>I inserted this HTML</p>";
const span = "<span>Inserted span</span> ";
p.insertAdjacentHTML('beforebegin', html);   // insert html before opening <p> tag
p.insertAdjacentHTML('afterbegin', span);    // insert span after opening <p> tag
```
### textContent for plain text

`textContent` inserts or queries plain text in an element so you do not have to deal with HTML tags:
- Works for Element and Text nodes
- Gets all text content in all decendant elements
- DO NOT use `innerText`

```js
let text = para.textContent;
para.textContent += "this is a test";
```

## Creating, inserting, deleting nodes

### createElement()

The Document class provides methods for creating Element objects, and the Element and Text object have methods for inserting, deleting, and replacing nodes in the document tree:
- `createElement()` creates an element
  - `append()` Element nodes: appends text strings or other elements to the arg. Takes multiple args
  - `prepend()` Element nodes: prepends text strings or other elements to the arg. Takes multiple args
- `createTextNode()` doesn't really have a use because you can add text to elements with `append()` and `prepend()`

```js
let newP = document.createElement('p');
let em = document.createElement('em');
em.append('fantastic');
newP.append('This is ', em, '!');
newP.prepend('Hey! ');

console.log(newP.textContent);              // Hey! This is fantastic!
console.log(newP.innerHTML);                // Hey! This is <em>fantastic</em>!
```

### append() and prepend()

- `append()` Element nodes: appends text strings or other elements to the arg. Takes multiple args
- `prepend()` Element nodes: prepends text strings or other elements to the arg. Takes multiple args

```js
em.append('fantastic');
newP.append('This is ', em, '!');
newP.prepend('Hey! ');

console.log(newP.textContent);              // Hey! This is fantastic!
console.log(newP.innerHTML);                // Hey! This is <em>fantastic</em>!
```

### before() and after()

To insert an Element or Text node in the middle of a container element's child list, use `before()` or `after()`:
- Select the element with `querySelector()`, then use `before()` or `after()`
- Both methods take any number of string and element arguments
- Works on Element and Text nodes
- If you grab an existing element and move it, it is not copied, it is moved to the new location
  - `cloneNode(true)` copies the element 

```js
let para = document.querySelector('.paragraph');

// create new p el, add text, place after existing para el
let newPara = document.createElement('p');
newPara.textContent = "This is also a paragraph!";
para.after(newPara, document.createElement('hr'));

// moves h2 el, doesn't copy it
let h2 = document.querySelector('.inner-h2');
para.before(h2);

// clones h2 element and injects in document
let h2 = document.querySelector('.inner-h2');
para.before(h2.cloneNode(true));
```
### remove() and replaceWith()

Remove or replace an element:
- `remove()` takes no args
- `replaceWith()` takes any number of element and strings
- DOM also offers other methods that are more difficult to use and shouldn't be needed:
  - `appendChild()`
  - `insertBefore()`
  - `replaceChild()`
  - `removeChild()`

```js
para.remove();                          // remove para el
para.replaceWith(h2);                   // replace para el with h2 el, remove h2
para.replaceWith(h2.cloneNode(true));   // replace para el with h2 el, but don't remove h2
```

