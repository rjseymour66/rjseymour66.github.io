---
title: "Patterns"
# linkTitle: "CSS"
weight: 200
# description: 
---

https://www.patterns.dev/

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


### Cookie builder 

Read a cookie by name, create a new cookie, set cookie expiration, and delete a cookie.

```html
<!doctype html>
<html>
<head>
    <title>Complete JavaScript Course</title>
</head>
<body>
    <script>
        console.log(document.cookie);
        console.log(rCookie("test1"));
        console.log(rCookie("test"));
        cCookie("test1", "new Cookie", 30);
        dCookie("test2");
        function cCookie(cName, value, days) {
            if (days) {
                const d = new Date();
                d.setTime(d.getTime() + (days * 24 * 60 * 60 * 1000));
                let e = "; expires=" + d.toUTCString();
                document.cookie = cName + "=" + value + e + "; path=/";
            }
        }
        function rCookie(cName) {
            let cookieValue = false;
            let arr = document.cookie.split("; ");
            arr.forEach(str => {
                const cookie = str.split("=");
                if (cookie[0] == cName) {
                    cookieValue = cookie[1];
                }
            });
            return cookieValue;
        }
        function dCookie(cName) {
            cCookie(cName, "", -1);
        }
    </script>
</body>
</html>
```

### Local storage shopping list 

```html 
<!doctype html>
<html>
<head>
    <title>JavaScript</title>
    <style>
        .ready {
            background-color: #ddd;
            color: red;
            text-decoration: line-through;
        }
    </style>
</head>
<body>
    <div class="main">
        <input placeholder="New Item" value="test item" maxlength="30">
        <button>Add</button>
    </div>
    <ul class="output">
    </ul>
    <script>
        const userTask = document.querySelector(".main input");
        const addBtn = document.querySelector(".main button");
        const output = document.querySelector(".output");
        const tasks = JSON.parse(localStorage.getItem("tasklist")) || [];
        addBtn.addEventListener("click", createListItem);
        if (tasks.length > 0) {
            tasks.forEach((task) => {
                genItem(task.val, task.checked);
            });
        }
        function saveTasks() {
            localStorage.setItem("tasklist", JSON.stringify(tasks));
        }
        function buildTasks() {
            tasks.length = 0;
            const curList = output.querySelectorAll("li");
            curList.forEach((el) => {
                const tempTask = {
                    val: el.textContent,
                    checked: false
                };
                if (el.classList.contains("ready")) {
                    tempTask.checked = true;
                }
                tasks.push(tempTask);
            });
            saveTasks();
        }
        function genItem(val, complete) {
            const li = document.createElement("li");
            const temp = document.createTextNode(val);
            li.appendChild(temp);
            output.append(li);
            userTask.value = "";
            if (complete) {
                li.classList.add("ready");
            }
            li.addEventListener("click", (e) => {
                li.classList.toggle("ready");
                buildTasks();
            });
            return val;
        }
        function createListItem() {
            const val = userTask.value;
            if (val.length > 0) {
                const myObj = {
                    val: genItem(val, false),
                    checked: false
                };
                tasks.push(myObj);
                saveTasks();
            }
        }
    </script>
</body>
</html>
```

### Form validator 

Validate form values before they are submitted. 

```html 
<!doctype html>
<html>
<head>
    <title>JavaScript Course</title>
    <style>
        .hide {
            display: none;
        }
        .error {
            color: red;
            font-size: 0.8em;
            font-family: sans-serif;
            font-style: italic;
        }
        input {
            border-color: #ddd;
            width: 400px;
            display: block;
            font-size: 1.5em;
        }
    </style>
</head>
<body>
    <form name="myform"> Email :
        <input type="text" name="email"> <span class="error hide"></span>
        <br> Password :
        <input type="password" name="password"> <span class="error hide"></span>
        <br> User Name :
        <input type="text" name="userName"> <span class="error hide"></span>
        <br>
        <input type="submit" value="Sign Up"> </form>
    <script>
        const myForm = document.querySelector("form");
        const inputs = document.querySelectorAll("input");
        const errors = document.querySelectorAll(".error");
        const required = ["email", "userName"];
        myForm.addEventListener("submit", validation);
        function validation(e) {
            let data = {};
            e.preventDefault();
            errors.forEach(function (item) {
                item.classList.add("hide");
            });
            let error = false;
            inputs.forEach(function (el) {
                let tempName = el.getAttribute("name");
                if (tempName != null) {
                    el.style.borderColor = "#ddd";
                    if (el.value.length == 0 && 
                    required.includes(tempName)) {
                        addError(el, "Required Field", tempName);
                        error = true;
                    }
                    if (tempName == "email") {
                        let exp = /([A-Za-z0-9._-]+@[A-Za-z0-9._-]+\.[A-Za-z0-9]+)\w+/;
                        let result = exp.test(el.value);
                        if (!result) {
                            addError(el, "Invalid Email", tempName);
                            error = true;
                        }
                    }
                    if (tempName == "password") {
                        let exp = /[A-Za-z0-9]+$/;
                        let result = exp.test(el.value);
                        if (!result) {
                            addError(el, "Only numbers and Letters",
                                     tempName);
                            error = true;
                        }
                        if (!(el.value.length > 3 &&
                        el.value.length < 9)) {
                            addError(el, "Needs to be between 3-8 " +
                                     "characters", tempName);
                            error = true;
                        }
                    }
                    data[tempName] = el.value;
                }
            });
            if (!error) {
                myForm.submit();
            }
        }
        function addError(el, mes, fieldName) {
            let temp = el.nextElementSibling;
            temp.classList.remove("hide");
            temp.textContent = fieldName.toUpperCase() + " " + mes;
            el.style.borderColor = "red";
            el.focus();
        }
    </script>
</body>
</html>
```


### JSON data files

Create a JSON file locally, connect to the JSON and data, and output the data from the JSON file into your console:

```js
let url = "people.json";
fetch(url)
.then(response => response.json())
.then(data => {
    console.log(data);
    data.forEach(person => {
        console.log(`${person.first} ${person.last} - ${person.topic}`);
    });
});
// people.json 
[
    {
        "first": "Laurence",
        "last": "Svekis",
        "topic": "JavaScript"
    },
    {
        "first": "John",
        "last": "Smith",
        "topic": "HTML"
    },
    {
        "first": "Jane",
        "last": "Doe",
        "topic": "CSS"
    }
]
```

### Make a list 

Create a list that saves to local storage so even if the page is refreshed, the data will persist within the browser. If the local storage is empty on the first load of the page, set up a JSON file that will be loaded to the local storage and saved as a default list to start the list:

```html 
<!DOCTYPE html>
<html>
<head>
    <title>JavaScript List Project</title>
</head>
<body>
    <div class="output"></div>
    <input type="text"><button>add</button>
    <script>
        const output = document.querySelector(".output");
        const myValue = document.querySelector("input");
        const btn1 = document.querySelector("button");
        const url = "list.json";
        btn1.addEventListener("click", addToList);
        let localData = localStorage.getItem("myList");
        let myList = [];
        window.addEventListener("DOMContentLoaded", () => {
            output.textContent = "Loading......";
            if (localData) {
                myList = JSON.parse(localStorage.getItem("myList"));
                output.innerHTML = "";
                myList.forEach((el, index) => {
                    maker(el);
                });
            } else {
                reloader();
            }
        });
        function addToList() {
            if (myValue.value.length > 3) {
                const myObj = {
                    "name": myValue.value
                }
                myList.push(myObj);
                maker(myObj);
                savetoStorage();
            }
            myValue.value = "";
        }
        function savetoStorage() {
            console.log(myList);
            localStorage.setItem("myList", JSON.stringify(myList));
        }
        function reloader() {
            fetch(url).then(rep => rep.json())
            .then((data) => {
                myList = data;
                myList.forEach((el, index) => {
                    maker(el);
                });
                savetoStorage();
            });
        }
        function maker(el) {
            const div = document.createElement("div");
            div.innerHTML = `${el.name}`;
            output.append(div);
        }
    </script>
</body>
</html>
```

## Events


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
