---
title: "Scripting documents"
linkTitle: "Scripting"
weight: 60
description:
---

JavaScript was created to build interactive web applications from static HTML
documents. Every `Window` object has a `document` property that references the
`Document` object. The `Document` object is the central object in the Document
Object Model (DOM) and provides the API you use to manipulate document content.

## Selecting document elements

CSS *selectors* describe elements or sets of elements within a document. The
following examples show common selector patterns:

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

`querySelector()` and `querySelectorAll()` search for elements matching a CSS
selector. They are the preferred selection methods and can be called on
`document` or any element object to search within that element's descendants:

- `querySelector()` returns the first matching element, or `null` if none
  is found.
- `querySelectorAll()` returns a `NodeList` of all matching elements. A
  `NodeList` is indexed and has a `length` property, so `for` loops work
  directly. To use array methods, convert it first with `Array.from()`.

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

### Real-world example: filterable list

One listener on the input filters all items in real time. Store the searchable
value in a `data-` attribute so the filter logic doesn't depend on visible text
formatting:

```js
const input = document.querySelector('#filter-input');
const items = document.querySelectorAll('.user-item');

input.addEventListener('input', () => {
    const query = input.value.toLowerCase();

    items.forEach(item => {
        const name = item.dataset.name.toLowerCase();
        item.hidden = !name.includes(query);
    });
});
```

```html
<input id="filter-input" placeholder="Search users…">
<ul>
  <li class="user-item" data-name="alice johnson">Alice Johnson</li>
  <li class="user-item" data-name="bob smith">Bob Smith</li>
  <li class="user-item" data-name="carol white">Carol White</li>
</ul>
```

`element.closest(selector)` walks up the DOM tree from the element and returns
the first ancestor that matches the selector, including the element itself. If
no match is found, it returns `null`. In this way it is the complement of
`querySelector()`, which searches downward through descendants.

`element.matches(selector)` returns a Boolean indicating whether the element
matches the given CSS selector. It is commonly used in event delegation to
check whether a clicked target is the intended element type.

### Element selection methods

These methods are older alternatives to `querySelector` and `querySelectorAll`.
Unlike `querySelectorAll()`, they return live `NodeList` objects that
automatically update when the document structure changes.

```js
let a = document.getElementById('idname');           // get element with id="idname"
let b = document.getElementsByName('nameAttr');      // document.querySelectorAll('*[name="nameAttr"]')
let c = document.getElementsByTagName('h1')          // get all h1 elements
let d = document.getElementsByClassName('container') // get elements with class="container"
```

## Document structure and traversal

The DOM Traversal API lets you access the relatives of an `Element` object:
its parents, children, and siblings. The API exposes this structure through
properties, not methods.

### Ignore Text and Comment nodes

The following properties skip `Text` and `Comment` nodes and return only
`Element` objects:

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

All `Node` objects, including `Text` and `Comment` nodes, expose properties for
traversing the document. These properties are sensitive to changes in the DOM,
so use them carefully when the document structure may change:

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
el.nextSibling.nodeType     // Node type number of the next sibling

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

An HTML element consists of a tag name and a set of name/value pairs called
*attributes*. You can work with attributes through `Element` class methods or
as properties on the `HTMLElement` object that represents the element. Working
with attributes as properties is often easier.

### Element class methods

The `Element` class provides four methods for reading and writing attributes
directly:

```js
el.getAttribute('class')            // get value of class attr
el.setAttribute('id', 'new-id');    // add new id="new-id" attr
el.hasAttribute('id')               // Boolean, whether el has id attr
el.removeAttribute('id');           // remove id attr from el
```

### Attributes as element properties

You can access and set most attributes with dot (`.`) notation. A few rules
apply:

- You cannot delete attributes with dot notation. Use `removeAttribute()`
  instead.
- Some HTML attribute names map to differently-named JavaScript properties.
  For example, `<input value="val">` maps to the JS property `defaultValue`.
- Multi-word HTML attribute names use camelCase in JavaScript. For example,
  `font-family` becomes `fontFamily`.
- HTML attribute names that are JavaScript reserved words are prefixed with
  `html`. For example, `<label for="val">` uses the JS property `htmlFor`.
- Properties that represent strings return strings. Properties that represent
  Boolean or numeric values return those types.

```js
// read id attr
let myId = p.id;
console.log(myId === 'my-id');  // true

// set form attrs
let f = document.querySelector('.form');
f.action = 'https://www.example.submit';
f.method = 'POST';
```

### Class attribute

`class` is a reserved word in JavaScript, so use the `className` property to
get or set an element's class attribute. The value is a space-delimited string
of class names, not a single class.

The `classList` property returns the classes as an iterable, Array-like object
with four methods: `add()`, `remove()`, `contains()`, and `toggle()`.

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

> Here is a good [Web Dev Simplified](https://blog.webdevsimplified.com/2020-10/javascript-data-attributes/) post that explains data attributes.

HTML lets you add custom attributes to any element as long as you prefix them
with `data-`. These are called *dataset attributes*. The `Element` object exposes
them through its `dataset` property, where each `data-` attribute maps to a
property name. Hyphenated attribute names become camelCase. For example,
`data-my-data` is accessible as `dataset.myData`.

```js
<div data-my-data="custom attribute">
    ...
</div>

let div = document.querySelector('div');
console.log(div.dataset.myData);            // custom attribute
```

## Element content

### `innerHTML` for HTML content

You can set and return element content as a string of HTML or plain text:

- **Never insert untrusted content with `innerHTML`.** An attacker can inject
  `<script>` tags or event handler attributes, which executes arbitrary code
  (XSS). Use `textContent` for plain text, or `createElement` for structured
  content. If you must render HTML from an untrusted source, sanitize it first
  with a library like [DOMPurify](https://github.com/cure53/DOMPurify):

  ```js
  // UNSAFE — never do this with user input
  el.innerHTML = userInput;

  // SAFE — plain text only
  el.textContent = userInput;

  // SAFE — structured content via DOM API
  const p = document.createElement('p');
  p.textContent = userInput;
  container.appendChild(p);

  // SAFE — HTML from untrusted source, sanitized
  // el.innerHTML = DOMPurify.sanitize(userInput);
  ```

- `innerHTML` returns element content as a string of HTML markup. Appending
  text this way is inefficient because the browser must serialize the HTML
  to a string, append the text, then parse it back into an element.
- `outerHTML` returns the element's HTML including its own opening and closing
  tags. Setting `outerHTML` replaces the element entirely.
- `insertAdjacentHTML()` inserts HTML markup at a position relative to the
  element. It takes two arguments:
  - The position: `beforebegin`, `afterbegin`, `beforeend`, or `afterend`
  - The HTML string to insert

The following examples show `innerHTML`, `outerHTML`, and `insertAdjacentHTML()`
in use:

```js
p.innerHTML    // This is a <span>paragraph</span>.
p.outerHTML    // <p class="paragraph">This is a <span>paragraph</span>.</p>


const html = "<p>I inserted this HTML</p>";
const span = "<span>Inserted span</span> ";
p.insertAdjacentHTML('beforebegin', html);   // insert html before opening <p> tag
p.insertAdjacentHTML('afterbegin', span);    // insert span after opening <p> tag
```

### `textContent` for plain text

`textContent` reads or writes plain text in an element without processing HTML
tags. It works for both `Element` and `Text` nodes and returns all text content
across all descendant elements. **Do not** use `innerText` as an alternative.

```js
let text = para.textContent;
para.textContent += "this is a test";
```

## Creating, inserting, deleting nodes

### `createElement()`

The `Document` class provides methods for creating `Element` objects. The
`Element` and `Text` objects provide methods for inserting, deleting, and
replacing nodes in the document tree.

`createElement()` creates a new element. To add content to it, use `append()`
to add text strings or child elements after existing content, or `prepend()` to
add them before. Both methods accept multiple arguments. `createTextNode()` is
rarely needed because `append()` and `prepend()` accept plain strings directly.

```js
let newP = document.createElement('p');
let em = document.createElement('em');
em.append('fantastic');
newP.append('This is ', em, '!');
newP.prepend('Hey! ');

console.log(newP.textContent);              // Hey! This is fantastic!
console.log(newP.innerHTML);                // Hey! This is <em>fantastic</em>!
```

### `append()` and `prepend()`

Both methods accept any number of string and element arguments:

- **`append()`:** Adds text strings or elements after the last child of the
  element
- **`prepend()`:** Adds text strings or elements before the first child of the
  element

```js
em.append('fantastic');
newP.append('This is ', em, '!');
newP.prepend('Hey! ');

console.log(newP.textContent);              // Hey! This is fantastic!
console.log(newP.innerHTML);                // Hey! This is <em>fantastic</em>!
```

### `before()` and `after()`

To insert an `Element` or `Text` node in the middle of a container element's
child list, use `before()` or `after()`. Both methods accept any number of
string and element arguments and work on both `Element` and `Text` nodes.

If you pass an existing element, the browser moves it to the new position rather
than copying it. To copy it instead, pass `cloneNode(true)`:

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

### `remove()` and `replaceWith()`

`remove()` removes the element from the document and takes no arguments.
`replaceWith()` replaces the element with any number of elements or strings.

The DOM also offers older methods that are harder to use and are not needed
when the modern methods above are available:

- `appendChild()`
- `insertBefore()`
- `replaceChild()`
- `removeChild()`

The following examples show `remove()` and `replaceWith()` in use:

```js
para.remove();                          // remove para el
para.replaceWith(h2);                   // replace para el with h2 el, remove h2
para.replaceWith(h2.cloneNode(true));   // replace para el with h2 el, but don't remove h2
```

### `append()` vs `appendChild()`

> Not needed? See remove() first.

`appendChild()` appends a single node to the end of a parent's child list.
`append()` does the same but accepts multiple arguments and plain strings. The
following table summarizes their differences:

| Differences     | `append()`                | `appendChild()`          |
| :-------------- | :------------------------ | :----------------------- |
| Return value    | `undefined`               | The appended Node object |
| Input           | Multiple Node Objects     | A single Node object     |
| Parameter types | Accept Node and DOMString | Only Node                |

### `parentNode` methods

`parentNode.appendChild(childNode)` appends `childNode` as the last child of
`parentNode`.

> Append child nodes to the parent element before you append the parent element
> to the DOM.

`parentNode.insertBefore(newNode, referenceNode)` inserts `newNode` into
`parentNode` before `referenceNode`.

`parentNode.removeChild(child)` removes `child` from `parentNode` and returns
a reference to the removed child:

```js
const linkPara = document.querySelector("p");
linkPara.parentNode.removeChild(linkPara);
```

### Remove all child elements

When you create and append nodes dynamically, clear the container's children
before refreshing the list:

```js
const container = document.querySelector('.grid-container');

while (container.firstChild) {
    container.removeChild(container.firstChild);
}
```

The loop removes the first child on each iteration until no children remain.

## CSS

JavaScript can modify CSS directly without editing stylesheets. Common uses
include:

- Hiding elements by setting `display` to `none`
- Positioning elements dynamically with `position` set to `absolute`,
  `relative`, or `fixed`
- Transforming elements with `transform` to shift, scale, or rotate
- Triggering CSS transitions to animate property changes from JavaScript

### Classes

The most straightforward way to manipulate styling is by adding and removing
classes with the `classList` methods:

```js
let div = document.querySelector('div');

div.classList.add('test-class');        // adds class="test-class"
div.classList.remove('container');      // removes class="container"
div.classList.contains('test-class');   // true - checks if class is in classList
div.classList.toggle('test-class');     // removes class="test-class"
div.classList.contains('test-class');   // false

// hide p element on click event w/hidden class
let b = document.querySelector('.btn');
let p = document.querySelector('.text');

b.addEventListener('click', () => {
    console.log(p);
    p.classList.toggle('hidden');
});
```

### Inline styles

The DOM defines a `style` property on all `Element` objects. The `style`
property is not a string: it is a `CSSStyleDeclaration` object, a parsed
representation of the element's inline CSS. All values are specified as strings
and must include units. For example, `el.style.borderRadius = "8px"`. CSS
kebab-case property names become camelCase in JavaScript. For example,
`font-family` becomes `fontFamily`.

```js
// format
el.style.<property> = "value";

// position a paragraph on the page
let displayAt = (para, x, y) => {
    para.style.display = 'block';
    para.style.position = 'absolute';
    para.style.left = `${x}px`;
    para.style.top = `${y}px`;
};

displayAt(p, 43, 34);

// setting shortcut properties
e.style.margin = `${top}px ${right}px ${left}px ${bottom}px`;
```

### Computed styles

Computed styles are the final values of CSS properties after all stylesheets
have been applied and resolved by the browser. Use
`window.getComputedStyle(element, pseudoelement)` to read the style the browser
used to render an element:

```js
let title = document.querySelector('.title');
let tStyles = window.getComputedStyle(title);
```

Computed styles differ from inline styles in three ways:

- Computed styles are read-only. You cannot change them.
- Computed styles are absolute. The browser converts all values to `px`,
  `rgb()`, or `rgba()`.
- The `cssText` property of a computed style is `undefined`.

### Scripting stylesheets

You can give a `<style>` tag an `id` attribute and then read or modify it with
JavaScript. Setting the `disabled` property on a stylesheet element deactivates
it without removing it from the DOM, which is useful for switching between
themes. You can also inject new stylesheets directly into the DOM. To inspect
or modify a stylesheet's rules programmatically, search for `CSSStyleSheet`.

The following example toggles between two stylesheets by enabling one and
disabling the other:

```js
// could be light and dark themes, not button colors
let toggleStyles = () => {
    let red = document.querySelector('#red-button');        // different stylesheets
    let blue = document.querySelector('#blue-button');

    if (red.disabled) {
        blue.disabled = true;
        red.disabled = false;
    } else {
        blue.disabled = false;
        red.disabled = true;
    }
};

b.addEventListener('click', toggleStyles);
```

The following example swaps the active theme by replacing an existing `<link>`
element or injecting a new one:

```js
// different way to change themes
let setTheme = (name) => {
    // create new link element
    let link = document.createElement('link');
    link.id = 'theme';
    link.rel = 'stylesheet';
    link.href = `${name}.css`;

    // look for existing theme - all themes use id="theme"
    let currentTheme = document.querySelector('#theme');
    if (currentTheme) {
        currentTheme.replaceWith(link);
    } else {
        // insert new link in head
        document.head.append(link);
    }
};

setTheme('other-style');
```

### Real-world example: toast notification

A toast is a temporary message that appears and fades out automatically. It
creates a DOM element, inserts it, then removes it after a delay. No HTML
template is needed:

```js
function showToast(message, type = 'info', duration = 3000) {
    const toast = document.createElement('div');
    toast.textContent = message;
    toast.className = `toast toast--${type}`;
    Object.assign(toast.style, {
        position: 'fixed',
        bottom: '1.5rem',
        right: '1.5rem',
        padding: '0.75rem 1.25rem',
        borderRadius: '6px',
        color: 'white',
        background: type === 'error' ? '#c62828' : '#323232',
        transition: 'opacity 0.3s ease',
        opacity: '1',
    });

    document.body.appendChild(toast);

    setTimeout(() => {
        toast.style.opacity = '0';
        toast.addEventListener('transitionend', () => toast.remove(), { once: true });
    }, duration);
}

// Usage
showToast('Settings saved.');
showToast('Upload failed — please try again.', 'error');
showToast('New message from Alice.', 'info', 5000);
```

### Animation and events

Revisit when I understand transitions and keyframes.
