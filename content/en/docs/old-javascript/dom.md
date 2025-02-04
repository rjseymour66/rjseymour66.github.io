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

## Examples 

### Getting text content from click event

```html
<!doctype html>
<html>
<head>
    <title>JS Tester</title>
</head>
<body>
    <div>
        <button onclick="message(this)">Button 1</button>
        <button onclick="message(this)">Button 2</button>
    </div>
    <script>
        function message(el) {
            console.dir(el.textContent);
        }
    </script>
</body>
</html>
```

### Build table and retrieve data 

```html 
<!DOCTYPE html>
<html>
<head>
    <title>JavaScript</title>
</head>
<body>
    <div id="message"></div>
    <div id="output"></div>
    <script>
        const message = document.querySelector("#message");
        const myArray = ["Laurence", "Mike", "John", "Larry", "Kim",
                         "Joanne", "Lisa", "Janet", "Jane"];
        build();
        //addClicks();
        function build() {
            let html = "<h1>My Friends Table</h1><table>";
            myArray.forEach((item, index) => {
                html += `<tr class="box" data-row="${index+1}"
                         data-name="${item}" onclick="getData(this)">
                         <td>${item}</td>`;
                html += `<td >${index + 1}</td></tr>`;
            });
            html += "</table>";
            document.getElementById("output").innerHTML = html;
        }
        function getData(el) {
            let temp = el.getAttribute("data-row");
            let tempName = el.getAttribute("data-name");
            message.innerHTML = `${tempName } is in row #${temp}`;
        }
    </script>
</body>
</html>
```

### Add event listener to button with loop 

```html 
<!doctype html>
<html>
<head>
    <title>JS Tester</title>
</head>
<body>
    <div>
        <button>Button 1</button>
        <button>Button 2</button>
        <button>Button 3</button>
    </div>
    <script>
        const btns = document.querySelectorAll("button");
        btns.forEach((btn)=>{
            function output(){
                console.log(this.textContent);
            }
            btn.addEventListener("click",output);
        });
    </script>
</body>
</html>
```

### Add items to a list with input box and button 

```html 
<!DOCTYPE html>
<html>
<head>
    <title>JavaScript</title>
    <style>
    </style>
</head>
<body>
    <div id="message">Complete JavaScript Course</div>
    <div>
        <input type="text" id="addItem">
        <input type="button" id="addNew" value="Add to List"> </div>
    <div id="output">
        <h1>Shopping List</h1>
        <ol id="sList"> </ol>
    </div>
    <script>
        document.getElementById("addNew").onclick = function () {
            addOne();
        }
        function addOne() {
            var a = document.getElementById("addItem").value;
            var li = document.createElement("li");
            li.appendChild(document.createTextNode(a));
            document.getElementById("sList").appendChild(li);
        }
    </script>
</body>
</html>
```

### Collapsible accordian 

```html 
<!doctype html>
<html>
<head>
    <title>JS Tester</title>
    <style>
        .active {
            display: block !important;
        }
        .myText {
            display: none;
        }
        .title {
            font-size: 1.5em;
            background-color: #ddd;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="title">Title #1</div>
        <div class="myText">Just some text #1</div>
        <div class="title">Title #2</div>
        <div class="myText">Just some text #2</div>
        <div class="title">Title #3</div>
        <div class="myText">Just some text #3</div>
    </div>
    <script>
        const menus = document.querySelectorAll(".title");
        const openText = document.querySelectorAll(".myText");
        menus.forEach((el) => {
            el.addEventListener("click", (e) => {
                console.log(el.nextElementSibling);
                remover();
                el.nextElementSibling.classList.toggle("active");
            })
        })
        function remover() {
            openText.forEach((ele) => {
                ele.classList.remove("active");
            })
        }
    </script>
</body>
</html>
```

### Voting system (add to list, increase counter on clicks)

```html 
<!DOCTYPE html>
<html>
<head>
    <title>JavaScript</title>
</head>
<body>
    <div id="message">Complete JavaScript Course</div>
    <div>
        <input type="text" id="addFriend">
        <input type="button" id="addNew" value="Add Friend">
    </div>
    <table id="output"></table>
    <script>
        window.onload = build;
        const myArray = ["Laurence", "Mike", "John", "Larry"];
        const message = document.getElementById("message");
        const addNew = document.getElementById("addNew");
        const newInput = document.getElementById("addFriend");
        const output = document.getElementById("output");
        addNew.onclick = function () {
            const newFriend = newInput.value;
            adder(newFriend, myArray.length, 0);
            myArray.push(newFriend);
        }
        function build() {
            myArray.forEach((item, index) => {
                adder(item, index, 0);
            });
        }
        function adder(name, index, counter) {
            const tr = document.createElement("tr");
            const td1 = document.createElement("td");
            td1.classList.add("box");
            td1.textContent = index + 1;
            const td2 = document.createElement("td");
            td2.textContent = name;
            const td3 = document.createElement("td");
            td3.textContent = counter;
            tr.append(td1);
            tr.append(td2);
            tr.append(td3);
            tr.onclick= function () {
                console.log(tr.lastChild);
                let val = Number(tr.lastChild.textContent);
                val++;
                tr.lastChild.textContent = val;
            }
            output.appendChild(tr);
        }
    </script>
</body>
</html>
```

### Hangman 

```html 
<!doctype html>
<html><head>
    <title>Hangman Game</title>
    <style>
        .gameArea {
            text-align: center;
            font-size: 2em;
        }
        .box,
        .boxD {
            display: inline-block;
            padding: 5px;
        }
        .boxE {
            display: inline-block;
            width: 40px;
            border: 1px solid #ccc;
            border-radius: 5px;
            font-size: 1.5em;
        }
    </style>
</head>
<body>
    <div class="gameArea">
        <div class="score"> </div>
        <div class="puzzle"></div>
        <div class="letters"></div>
        <button>Start Game</button>
    </div>
    <script>
        const game = { cur: "", solution: "", puzz: [], total: 0 };
        const myWords = ["learn Javascript", "learn html",
                         "learn css"];
        const score = document.querySelector(".score");
        const puzzle = document.querySelector(".puzzle");
        const letters = document.querySelector(".letters");
        const btn = document.querySelector("button");
        btn.addEventListener("click", startGame);
        function startGame() {
            if (myWords.length > 0) {
                btn.style.display = "none";
                game.puzz = [];
                game.total = 0;
                game.cur = myWords.shift();
                game.solution = game.cur.split("");
                builder();
            } else {
                score.textContent = "No More Words.";
            }
        }
        function createElements(elType, parentEle, output, cla) {
            const temp = document.createElement(elType);
            temp.classList.add("boxE");
            parentEle.append(temp);
            temp.textContent = output;
            return temp;
        }
        function updateScore() {
            score.textContent = `Total Letters Left : ${game.total}`;
            if (game.total <= 0) {
                console.log("game over");
                score.textContent = "Game Over";
                btn.style.display = "block";
            }
        }
        function builder() {
            letters.innerHTML = "";
            puzzle.innerHTML = "";
            game.solution.forEach((lett) => {
                let div = createElements("div", puzzle, "-", "boxE");
                if (lett == " ") {
                    div.style.borderColor = "white";
                    div.textContent = " ";
                } else {
                    game.total++;
                }
                game.puzz.push(div);
                updateScore();
            })
            for (let i = 0; i < 26; i++) {
                let temp = String.fromCharCode(65 + i);
                let div = createElements("div", letters, temp,"box");
  
                let checker = function (e) {
                    div.style.backgroundColor = "#ddd";
                    div.classList.remove("box");
                    div.classList.add("boxD");
                    div.removeEventListener("click", checker);
                    checkLetter(temp);
                }
                div.addEventListener("click", checker);
            }
        }
        function checkLetter(letter) {
            console.log(letter);
            game.solution.forEach((ele, index) => {
                if (ele.toUpperCase() == letter) {
                    game.puzz[index].textContent = letter;
                    game.total--;
                    updateScore();
                };
            };
            )
        }
    </script>
</body>
</html>
```