---
title: "Foundations"
# linkTitle: "CSS"
weight: 10
description: >
  Beginner information about web development.
---

## Links

- [DOM tree and nodes](https://www.digitalocean.com/community/tutorials/understanding-the-dom-tree-and-nodes#html-terminology)
- [Javascript DOM tutorial](https://www.javascripttutorial.net/javascript-dom/)
- [Eloquent Javascript "The Document Object Model"](https://eloquentjavascript.net/14_dom.html)
- [Eloquent Javascript "Handling Events"](https://eloquentjavascript.net/15_event.html)
- [DOM Enlightenment](https://domenlightenment.com/)
- [MDN Intro to Events](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events)



## DOM

- Web browsers parse HTML to create the DOM. The DOM nodes are objects with properties and methods.
- Javascript alters the DOM, not the HTML.
- `element.querySelectorAll()` returns a nodelist. This is not an array--it does not have the normal array methods. To convert to an array, use `Array.from()` or the spread operator.
- Use `defer` to make sure that the HTML is parsed before you manipulate the DOM.

```js
<head>
  <script src="js-file.js" defer></script>
</head>
```

### Add/remove DOM elements

- `parentNode.appendChild(childNode)` - appends childNode as the last child of parentNode.
  > Append child nodes to the parent element before you append the parent element to the DOM.
- `parentNode.insertBefore(newNode, referenceNode)` - inserts newNode into parentNode before referenceNode.

- `parentNode.removeChild(child)` - removes child from parentNode on the DOM and returns a reference to child.

  ```js
  const linkPara = document.querySelector("p");
  linkPara.parentNode.removeChild(linkPara);
  ```
### Remove all child elements

If you are creating nodes with Javascript and appending them to a parent (like a contaier), then you need to remove all child nodes before you refresh the elements:

```js
const container = document.querySelector('.grid-container');

while (container.firstChild) {
    container.removeChild(container.firstChild);
}
```
So, while the container has a first child, remove the first child from the container.

### append() vs appendChild()

- `appendChild()` appends a node to the parent's list of child nodes.
- `append()` inserts a node after the parent node's last child

| Differences     | `append()`                | `appendChild()`          |
| :-------------- | :------------------------ | :----------------------- |
| Return value    | `undefined`               | The appended Node object |
| Input           | Multiple Node Objects     | A single Node object     |
| Parameter types | Accept Node and DOMString | Only Node                |

### CSS

- You add styles with JS as inline styles. Inline styles are avialable through the `style` object on element node objects.
  - These style names don't use hyphens, they use camelCase. Just remove the hyphen and use camelCase.
  - Use hyphens if you use bracket notation. For example, `div.style['background-color']`.

#### get/set/remove properties

```js
// adds the indicated style rule
div.style.color = "blue";

// adds several style rules
div.style.cssText = "color: blue; background: white;";

// adds several style rules
div.setAttribute("style", "color: blue; background: white;");

// if id exists, update it to 'theDiv', else create an id
// with value "theDiv"
div.setAttribute("id", "theDiv");

// returns value of specified attribute, in this case
// "theDiv"
div.getAttribute("id");

// removes specified attribute
div.removeAttribute("id");
```

### classes

```js
div.classList.add("new");
// adds class "new" to your new div

div.classList.remove("new");
// removes "new" class from div

div.classList.toggle("active");
// if div doesn't have class "active" then add it, or if
// it does, then remove it
```

### Text

A text node is the actual text inside an element or attribute.

```js
div.textContent = "Hello World!";
// creates a text node containing "Hello World!" and
// inserts it in div

div.innerHTML = "<span>Hello World!</span>";
// renders the HTML inside div
```

> Use `textContent` to add text bc there are security issues with `innerHTML`.

## Events

[HTML DOM Events](https://www.w3schools.com/jsref/dom_obj_event.asp)

Add Events in the following 3 ways:

1. Add function attributes directly to HTML:
   ```html
   <button onclick="alert('Hello World')">Click Me</button>
   ```
   This means you can set only one `onclick` event on the HTML element.
2. Set properties, like `onclick`, to the DOM nodes in JS:

   ```html
   <button id="btn">Click Me</button>
   ```

   ```js
   const btn = document.querySelector("#btn");
   btn.onclick = () => alert("Hello World");
   ```

   You can still only set one `onclick` property.

   > `getElementById` is an older method and is used less today.

3. Attach event listeners (preferred):

   ```html
   <button id="btn">Click Me Too</button>
   ```

   ```js
   const btn = document.querySelector("#btn");
   btn.addEventListener("click", () => {
     alert("Hello World");
   });
   ```

   The event listener separates concerns, and you can add as many as you want.

### Event object

You can get the element that fires the event with `e.target`. For example:

```js
btn.addEventListener("click", function (e) {
  e.target.style.background = "blue";
});
```

### Listeners on groups of nodes

1. Grab the elements with `querySelectorAll()` to get a nodelist.
2. Use `forEach()` to add an event listener to all the nodes:

   ```html
   <div id="container">
     <button id="1">Click Me</button>
     <button id="2">Click Me</button>
     <button id="3">Click Me</button>
   </div>
   ```

   ```js
   // buttons is a node list. It looks and acts much like an array.
   const buttons = document.querySelectorAll("button");

   // we use the .forEach method to iterate through each button
   buttons.forEach((button) => {
     // and for each one we add a 'click' listener
     button.addEventListener("click", () => {
       alert(button.id);
     });
   });
   ```

## Objects

Define an object with curly braces and an optional list of properties that are key/value pairs.

### Properties

Multiword properties must use quotes:

```js
// you can use a comma after the last property
let user = {
  name: "John",
  age: 30,
  "likes birds": true,
};
```

#### Add or remove properties

```js
// add
user.isAdmin = true;

// delete
delete user.age;
```

#### Square brackets and properties

```js
let user = {};

// set
user["likes birds"] = true;

// get
alert(user["likes birds"]); // true

// delete
delete user["likes birds"];
```

#### undefined

If you try to access a property that doesn't exist, you get `undefined`:

```js
$ let user = {};
undefined
$ user
{}
$ user.noProp;
undefined
```

#### Key in test

```js
$ let user = {name: 'Jake', age: 40};
undefined
$ "name" in user
true
$ "age" in user
true
```

### Create an object

```js
let user = new Object();
let user = {};
```

The following are equal:

```js
let createUser = (name, age) => {
  return {
    name: name,
    age: age,
  };
};

let createUser = (name, age) => {
  return {
    name,
    age,
  };
};
```

### for...in loop

```js
let user = {
    name: 'Ryan',
    age: 40,
    likes: 'stuff',
}

$ for (let key in user) {
    console.log(key)
}
name
age
likes
```

If the keys can be converted to or from an integer, then JS automatically orders them. Otherwise, they are listed in the order that you created them:

```js
$ let nums = {
  10: "ten",
  5: "five",
  1: "one",
};


$ for (let key in nums) {
  console.log(key);
}
1;
5;
10;
```