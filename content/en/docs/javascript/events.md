---
title: "Events"
weight: 80
description: >
---

There are 3 ways to specify an event:
1. HTML:
   ```html
   <p id="unique" onclick="magic()">Click here for magic!</p>
   ```
2. Javascript:
   ```js 
   document.getElementById("unique").onclick = function() { magic(); };
   ```
3. Event listeners:
    ```js 
    document.getElementById("unique").addEventListener("click", magic);
    ```

For example:

```js
function hello() {
    console.log("hello")
}

function goodbye() {
    console.log("goodbye")
}

window.addEventListener('click', hello)
window.onclick = function() { goodbye() }
```

## Event handlers 

Event handlers consists of two components: the event to listen for, and a callback function that fires when the event occurs.

In most cases, you pass the event object to callback function so you can access its properties. The parameter is commonly named `e` or `event`:

```js 
window.addEventListener('click', (e) => { console.log(e); })
```

### onload 

Fired after a specific element is loaded. It is most commonly used on the `window` element, but you can use it on any element.

`onload` behaves differently for the `window` and `document` objects, depending on the browser. If you add it to the `document` object, you can access any node in the DOM bc the event fires after the DOM loads.

Use `DOMContentLoaded()` method with `addEventListener` as a substitute for `document.onload`:

```js 
document.addEventListener("DOMContentLoaded", (e) => {
    console.log(e);
});

// or assign a function called unique() to the body

<body onload="unique()"></body>
```

### Mouse event handlers 

| Event | Description |
|-------|:------------|
| `ondbclick`    | when the mouse is double-clicked |
| `onmousedown`  | when the mouse clicks on top of an element without the click being released |
| `onmouseup`    | when the mouse click on top of an element is released |
| `onmouseenter` | when the mouse moves onto an element |
| `onmouseleave` | when the mouse leaves an element and all of its children |
| `onmousemove`  | when the mouse moves over an element |
| `onmouseout`   | when the mouse leaves an individual element |
| `onmouseover`  | when the mouse hovers over an element |

### Event target property

The event that you pass to the event handler callback has a number of properties, including the `target` property. The `target` is the HTML element that fired the event:

```js 
function triggerSomething() {
  console.dir(event.target);
}
```

The following example changes the innerHTML of the button's parent element when clicked:

```js 
<div id="welcome">Hi there!</div>
<form>
  <input type="text" name="firstname" placeholder="First name" />
  <input type="text" name="lastname" placeholder="Last name" />
  <input type="button" onclick="sendInfo()" value="Submit" />
</form>
<script>
  function sendInfo() {
    let p = event.target.parentElement;
    message("Welcome " + p.firstname.value + " " + p.lastname.value);
  }
  function message(m) {
    document.getElementById("welcome").innerHTML = m;
  }
</script>
```

### DOM event flow (event bubbling)

Event bubbling is when you have nested elements and each element has an event handler attached. When you trigger an event on a nested element, it fires its own event first, then its parent's event, then its parent's parent's, and so on. You can change this by using the 3rd parameter for `addEventListener`, `useCapture`:

```js
document.addEventListener("DOMContentLoaded", func(), useCapture;)
```

`useCapture` is a Boolean. This enables _event delegation_: instead of adding event handlers to every element in a block of HTML, you define a wrapper and assign the event to the wrapper element. This wrapper element applies the event handler to its child elements. 


### onchange and onblur

onchange is triggered when an element changes. For example, when the value of an input box changes. onblur 

### Key event handlers 

You can use this to restrict what characters an input box accepts. `event.key` returns the key you entered. You can use `onpaste` to restrict what users can paste.

Here is an example:

```html
<!doctype html>
<html>
  <body>
    <body>
      <div id="wrapper">JavaScript is fun!</div>
      <input type="text" name="myNum1" onkeypress="return numCheck()" onpaste="return false">
      <input type="text" name="myNum2" onkeypress="return numCheck2()" onpaste="return false">
      <script>
        function numCheck() {
            message(!isNaN(event.key));
            return !isNaN(event.key);
        }
        function numCheck2() {
            message(isNaN(event.key));
            return isNaN(event.key);
        }
        function message(m) {
            document.getElementById("wrapper").innerHTML = m;
        }
      </script>
  </body>
</html>
```

### Drag and drop elements 

Set `draggable` to true to indicate that an element can be dragged and dropped. The following example creates two divs, and one contains a nested `div` that with text. You can drag the nested div into the other top-level div by adding the following properties and functions:

- `ondrop`
- `ondragover`
- `ondragstart`: 
- `draggable`

```html 
<body>
  <div class="box" ondrop="dDrop()" ondragover="nDrop()">
    1
    <div id="dragme" ondragstart="dStart()" draggable="true">
      Drag Me Please!
    </div>
  </div>
  <div class="box" ondrop="dDrop()" ondragover="nDrop()">2</div>

  <script>
    let holderItem;
    function dStart() {
      holderItem = event.target;
    }
    function nDrop() {
      event.preventDefault();
    }
    function dDrop() {
      event.preventDefault();
      if (event.target.className == "box") {
        event.target.appendChild(holderItem);
      }
    }
  </script>
</body>
```

The script does the following:
1. When the nested div is selected, it triggers the ondragstart event. Its `dStart()` handler stores the `target` in the `holderItem` variable.
2. By default, HTML prevents dropping, so add the `ondrop` event to both top-level divs. Its `nDrop()` handler prevents this default behavior. 
3. In the target destination, add the ondrop event checks whether the target destination can accept the element that you are dropping. If it can, you append the item as a child to that element.

### Form submission

You can trigger a event when a user submits a form.with the `onsubmit` attribute:

```html
<form onsubmit="eHandler()">
```

The action and method attributes can redirect to another page:

```html 
<!doctype html>
<html>
  <body>
    <form action="redirectTarget.html" method="get">
      <input type="text" placeholder="name" name="name" />
      <input type="submit" value="send" />
    </form>
  </body>
</html>
```
In the previous example:
- `action` defines the page that we redirect to. This 
- `method` specifies how to send form data. It appends form data onto the `redirectTarget.html` page URL as query parameters in k/v pairs.

You can use the query parameters in `redirectTarget.html` as follows:

```html 
<!doctype html>
<html>
  <body>
    <script>
      let q = window.location.search; 
      let params = new URLSearchParams(q); 
      let name = params.get("name");
      console.log(name);
    </script>
  </body>
</html>
```

When `onsubmit` returns a Boolean, you can use it for form validation. In the following example, the `valForm()` function uses the event on the `wrapper` div, and the `id` values of its child elements to validate the fields:

```html 
<!doctype html>
<html>
  <body>
    <div id="wrapper"></div>
    <form action="anotherpage.html" method="get" onsubmit="return valForm()">
      <input type="text" id="firstName" name="firstName" placeholder="First name" />
      <input type="text" id="lastName" name="lastName" placeholder="Last name" />
      <input type="text" id="age" name="age" placeholder="Age" />
      <input type="submit" value="submit" />
    </form>
    <script>
      function valForm() {
        var p = event.target.children;
        if (p.firstName.value == "") {
          message("Need a first name!!");
          return false;
        }
        if (p.lastName.value == "") {
          message("Need a last name!!");
          return false;
        }
        if (p.age.value == "") {
          message("Need an age!!");
          return false;
        }
        return true;
      }
      function message(m) {
        document.getElementById("wrapper").innerHTML = m;
      }
    </script>
  </body>
</html>
```

## Examples 

### Dark mode 

```html 
<!DOCTYPE html>
<html>
<head>
    <title>Laurence Svekis</title>
</head>
<body>
    <script>
        let darkMode = false;
        window.onclick = () => {
            console.log(darkMode);
            if (!darkMode) {
                document.body.style.backgroundColor = "black";
                document.body.style.color = "white";
                darkMode = true;
            } else {
                document.body.style.backgroundColor = "white";
                document.body.style.color = "black";
                darkMode = false;
            }
        }
    </script>
</body>
</html>
```




```html 

```





### Build your own analytics

Record IDs, tabs, and class name of items clicked on:

```html 
<!doctype html >
<html>
<head>
    <title>JS Tester</title>
    <style>.box{width:200px;height:100px;border:1px solid black}</style>
</head>
<body> 
    <div class="container">
        <div class="box" id="box0">Box #1</div>
        <div class="box" id="box1">Box #2</div>
        <div class="box" id="box2">Box #3</div>
        <div class="box" id="box3">Box #4</div>
    </div> 
    <script>      
        const counter = [];  
        const main = document.querySelector(".container");
        main.addEventListener("click",tracker);
        function tracker(e){
            const el = e.target;
            if(el.id){
            const temp = {};
            temp.content = el.textContent;
            temp.id = el.id;
            temp.tagName = el.tagName;
            temp.class = el.className;
            console.dir(el);
            counter.push(temp);
            console.log(counter);
            }
        }
    </script>
</body>
</html>
```

### Star rater system 

```html

<!DOCTYPE html>
<html>
<head>
    <title>Star Rater</title>
    <style>
        .stars ul {
            list-style-type: none;
            padding: 0;
        }
        .star {
            font-size: 2em;
            color: #ddd;
            display: inline-block;
        }
        .orange {
            color: orange;
        }
        .output {
            background-color: #ddd;
        }
    </style>
</head>
<body>
    <ul class="stars">
        <li class="star">&#10029;</li>
        <li class="star">&#10029;</li>
        <li class="star">&#10029;</li>
        <li class="star">&#10029;</li>
        <li class="star">&#10029;</li>
    </ul>
    <div class="output"></div>
        <script>
        const starsUL = document.querySelector(".stars");
        const output = document.querySelector(".output");
        const stars = document.querySelectorAll(".star");
        stars.forEach((star, index) => {
            star.starValue = (index + 1);
            star.addEventListener("click", starRate);
        });
        function starRate(e) {
            output.innerHTML =
                `You Rated this ${e.target.starValue} stars`;
            stars.forEach((star, index) => {
                if (index < e.target.starValue) {
                    star.classList.add("orange");
                } else {
                    star.classList.remove("orange");
                }
            });
        }
    </script>
</body>
</html>
```