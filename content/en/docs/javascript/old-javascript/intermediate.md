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
