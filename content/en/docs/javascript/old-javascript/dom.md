---
title: "Document Object Model (DOM)"
weight: 70
description: >
---

The DOM takes an HTML page and turns it into a logical tree.  The Browser Object Model (BOM) holds all the methods and properties that Javascript can interact with in the browser, including the following: 
- DOM
- Previous pages visited 
- Size of the browser window

## Selecting page elements 

`document.querySelector()` selects the first element within the document that matches the specified selectors.
`document.querySelectorAll()` returns a static node list that represents a list of all elements in the documen that match the provided selectors.

## BOM 

The BOM is what lets JS communicate with the browser, including the following: 
- Window object 
- History
- Navigator 
- Location

To see all the properties of the BOM (an any other JS object), use `console.dir()`:

```js 
console.dir(window)
// access window properties with dot notation
window.history.length
// DOM representation:
window.document
// go back one page in history
window.history.go(-1)
```

### Window navigator 

Contains information about the browser, such as the what the browser is, the version, and the OS running the browser:

```js 
console.dir(window.navigator)
// navigator is globally available
console.dir(navigator)
```

### Window location object 

Contains the URL of the current web page. You can override parts of this to make the browser go to a different page:

```js 
console.dir(window.location)
// location is globally available
console.dir(location)
```

## Element manipulation with the DOM 

### innerText and innerHTML

`innerText` manipulates text between opening and closing of an element. It returns the content as plain text, so you cannot use this method if you want to add HTML to the page.

To add HTML to the page, you have to use `innerHTML`.

```js 
document.body.children.greeting.innerText = "<b>Bye!</b>"   // renders <b>Bye!</b>
document.body.children.greeting.innerHTML = "<b>Bye!</b>"   // renders Bye! in bold text
```

### Accessing elements 

It is important to remember that many of these methods return either an HTMLCollection or a NodeList. This means you can operate on them with array methods.
_HTMLCollection_
: A collection of document elements 

_NodeList_
: A collection of document nodes, such as element nodes, attribute nodes, and text nodes.

```js
// returns first element with the id
document.getElementById("two")  // returns the full HTML element: <div id="two" class="example">Hi!</div>

// returns HTMLCollection (js obj/list of nodes)
document.getElementsByTagName("div")
document.getElementsByTagName("div").item(2)            // returns third element in the HTMLCollection
document.getElementsByTagName("div").namedItem("two")   // returns element with id="two"

// returns HTMLCollection
document.getElementsByClassName("example")

// returns a NodeList of elements that match the CSS selector
// good alternative for getElementById() bc there should be 
// only one element with a specific id
document.querySelectorAll("p")
document.querySelectorAll("div.example")
document.querySelectorAll("div#id")

// returns a first HTML element first matching instance
document.querySelector("div")   // 
document.querySelector(".example")  //
```

### Click handler

A click handler connects a JS function to an HTML element:

```html
<!-- Inline handlers -->
<div id="one" onclick="alert('Ouch! Stop it!')">Don't click here!</div>
<!-- buttonClk is defined within the <script> tags -->
<button onclick="buttonClk()">Button</button>
```
Here is the same `buttonClk` handler using Javascript:
```js 
document.getElementById("click-btn").onclick = function () {alert("clicked the button")}
```

### this in the DOM

In the DOM, `this` refers to the element of the DOM that `this` belongs to. For example, the following function logs the HTML element:
```js 
function logThis(val) {
            console.log(val[.elementProp])
        }
```
HTML element: 

```html 
<button id="click-btn" onclick="logThis(this)">Button</button>
<!-- val.parentElement -->
<body>...</body>
```

This is useful when you need to get the value of an input box.

### style property 

Change the style of an element. The following snippet toggles whether a paragraph is displayed: 

```html 
<body>
  <script>
    function toggleDisplay(){
      let p = document.getElementById("magic");
      if(p.style.display === "none") {
        p.style.display = "block";
      } else {
        p.style.display = "none";
      }
    }
  </script>
  <p id="magic">I might disappear and appear.</p>
  <button onclick="toggleDisplay()">Magic!</button>
</body>
```

The following uses a `for` loop to apply styles to elements in the HTMLCollection that `getElementsByTagName` returns:

```js
function rainbowify(){
  let divs = document.getElementsByTagName("div");
  for(let i = 0; i < divs.length; i++) {
    divs[i].style.backgroundColor = divs[i].id;
  }
}
// ...
<div id="red"></div>
<div id="orange"></div>
<div id="yellow"></div>
<div id="green"></div>
<div id="blue"></div>
<div id="indigo"></div>
<div id="violet"></div>
```

### Changing class

Change the class of an element to edit its CSS.

Add a class with `classList.add("classname")`:

```js
function disappear(){
  document.getElementById("shape").classList.add("hide");
}
```

Remove a class with `classList.remove("classname")`:

```js 
function change(){
  document.getElementById("shape").classList.remove("blue");
}
```

### Toggling classes 

Toggling a class on and off an element is so common, there is a `toggle` method:

```js 
function changeVisibility(){
  document.getElementById("shape").classList.toggle("hide");
}
```

### Manipulating attributes 

The `setAttribute` method adds or changes element attributes. This method changes the HTML of the page--the in-browser DOM reflects any methods that you add or remove:

```js
function changeAttr(){
  let el = document.getElementById("shape");
  el.setAttribute("style", "background-color:red;border:1px solid black");
  el.setAttribute("id", "new");
  el.setAttribute("class", "circle");

}
```

## Event Listeners

An event is something that happens on a web page. Each HTML element has a set of event properties that you can set (`window.onload = ...`). Another option is that you can assign as an attribute one event handler to each HTML element. When you add an event handler to an HTML element, we call it "registering an event handler".

Javascript provides _event listeners_, a way to register event handlers that allows more than one handler per element. To register an event listener:
1. Select the element.
2. Chain `addEventListener(`_`event`_`, function)` to the selection statement.

For example:
```js
document.getElementById("square").addEventListener("click", changeColor);
```

### window.onload 

You can add an event listener by setting the event property of a certain object to a function. For example, the following function adds an event listener to a button when the page loads:

```html 
<script>
  window.onload = function() {
    document.getElementById("square").addEventListener("click", changeColor);
  }
  function changeColor(){
    let red = Math.floor(Math.random() * 256);
    let green = Math.floor(Math.random() * 256);
    let blue = Math.floor(Math.random() * 256);
    this.style.backgroundColor = `rgb(${red}, ${green}, ${blue})`;
  }
</script>
```

## Creating new elements 

To create an element and add it to the DOM:
1. Select an element that you want to nest the new element under (a parent element).
2. Create the element with `createElement` and add any content or styles.
3. Append the new element to a parent element in the DOM.

