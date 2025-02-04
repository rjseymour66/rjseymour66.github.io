---
title: "Intermediate JS"
weight: 80
description: >
---

## Functions and the argument object 

Each function has a custom `arguments` object that stores the values passed into each function as an array. Access them with index notation:

```js
function test(a, b, c) {
  console.log("first:", a, arguments[0]);
  console.log("second:", b, arguments[1]);
  console.log("third:", c, arguments[2]);
}
test("fun", "js", "secrets");
```

It is now more common (i.e. modern) to use the rest paramter (...param) that arguments.

## Hoisting and Strict mode 

Hoisting is when Javascript moves declarations to the top of the scope in which they are defined. It makes code less readable and is generally confusing. When you declare variables and functions, use `let` instead of `var`.

To use `strict` mode, add the following to the top of a `.js` file:

```js 
"use strict";
```

`strict` mode is considered a best practice. It fixes many of the odd behaviors that Javascript has (such as hoisting). It is a good precursor to TypeScript or other frameworks.

## Debugging 

Open the browser console > Sources tab, and right-click on the line to add a breakpoint where you need to debug.

## Error handling 

Use a `try/catch` block with an optional `finally` block to handle potential errors:

```js 
try {
  somethingVeryDangerous();
} catch (e) {
  if (e instanceof TypeError) {
    // deal with TypeError exceptions
  } else if (e instanceof RangeError) {
    // deal with RangeError exceptions
  } else if (e instanceof EvalError) {
    // deal with EvalError exceptions
  } else {
    //deal with all other exceptions
    throw e; //rethrow
  }
} finally {
  console.log("Error or no error, I will be logged!");
}
```
The `finally` block is executed if the function succeeds or if it throws an error. 

You can also throw an error on purpose: 

```js 
function somethingVeryDangerous() {
  throw RangeError();
}
```

## Cookies 

A cookie is a small data file that is stored on your computer and used by websites. They store data about website users. Each cookie is a string with a special pattern--key/value pairs separated by semi-colons:

```js 
document.cookie = "name=Maaike;favoriteColor=black";
```

For some browsers (Chrome), you cannot set cookies from the client side, only the server. The following code can read from the cookie:

```js 
let cookie = decodeURIComponent(document.cookie);
let cookieList = cookie.split(";");
for (let i = 0; i < cookieList.length; i++) {
  let c = cookieList[i];
  if (c.charAt(0) == " ") {
    c = c.trim();
  }
  if (c.startsWith("name")) {
    alert(c.substring(5, c.length)); 
  }
}
```
The following example uses a cookie to greet the user: 

```js
<!DOCTYPE html>
<html>
  <body>
    <input onchange="setCookie(this)" />
    <button onclick="sayHi('name')">Let's talk, cookie!</button>
    <p id="hi"></p>
    <script>
      function setCookie(e) {
        document.cookie = "name=" + e.value + ";";
      }
      function sayHi(key) {
        let name = getCookie(key);
        document.getElementById("hi").innerHTML = "Hi " + name;
      }
      function getCookie(key) {
        let cookie = decodeURIComponent(document.cookie);
        let cookieList = cookie.split(";");
        for (let i = 0; i < cookieList.length; i++) {
          let c = cookieList[i];
          if (c.charAt(0) == " ") {
            c = c.trim();
          }
          if (c.startsWith(key)) {
            console.log("hi" + c);
            return c.substring(key.length + 1, c.length);
          }
        }
      }
    </script>
  </body>
</html>
```

## Local storage 

Cookies save data, but the more modern way is to use local storage (both methods share the same privacy issues). Local storage can save key-value pairs in a web browser and use them in a new session. Local storage data does not need to be passed around with every HTTP request, like cookies do. 

The localStorage object is a property of the window object. Use the setItem() and getItem() methods to set and get values in local storage:

```html
<!DOCTYPE html>
<html>
  <body>
    <div id="stored"></div>
    <script>
      let message = "Hello storage!";
      localStorage.setItem("example", message);
      if (localStorage.getItem("example")) {
        document.getElementById("stored").innerHTML =
          localStorage.getItem("example");
      }
    </script>
  </body>
</html>
```
You can also retrieve and remove data with the key index. This is helpful when looping through data: 

```js
window.localStorage.key(0);   // get the key
window.localStorage.getItem(window.localStorage.key(0));    // get the value associated with the key
window.localStorage.removeItem("name"); // remove the item with the name key
window.localStorage.clear();    // remove all key-value pairs
```

## JSON 

Convert a string to a JSON object with `JSON.parse(str)`:

```js 
let str = "{\"name\": \"Maaike\", \"age\": 30}";
let obj = JSON.parse(str);
```

Convert a JSON object to a string with `.stringify()`

```js 
let dog = {
    "name": "champ",
    "breed": "dachshund"
};
let strdog = JSON.stringify(dog);
console.log(typeof strdog);
console.log(strdog);
```

## Examples 

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