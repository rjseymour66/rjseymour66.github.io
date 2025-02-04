---
title: "Functions"
weight: 60
description: >
---

Functions are actions--a bundle of statements that are executed when called. 

Follow these guidelines when you name a function:
- Use camelCase.
- Describe what the function does.
- Use a verb to help describe what the function is doing. Ex: `sendMsg`.

## Writing functions 

```js
// standard function declaration
function nameOfFunction() {
  // func body
}

// assign anonymous function to a variable
let varFunction = function() {
  // func body
}

// params do not need types
function paramFunc(param1, param2) {
  // func body
}
```
## Default parameters

If you do not pass arguments to functions that require parameters, then Javascript assigns them the default type, `undefined`.

> `undefined` + `undefined` = `NaN`

Create default parameters with the assignment syntax:

```js
function addNums(x = 2, y = 3) {
  return x + y;
}
```

## Special functions 

### Arrow functions 

Arrow functions are a short-hand notation. They are useful for passing higher-order functions:

```js
// no params
() => single line func body 

// one param
param => single line func body

// params
(param1, param2) => single line func body

(param1, param2) => {
  // multiple 
  // lines of 
  // code
}
```

### Immediately invoked function expressions (IIFE)

These are anonymous functions that you invoke immediately. These are helpful when you want to initialize something, or when yiu want to create private and public variables and functions.

To create an IIFE, wrap an anonymous function in parentheses, and add another set of parentheses after the function definition to invoke the function immediately:

```js
(function () {
  console.log("IIFE")
})();
```

### Anonymous functions 

To declare an anonymous function, use the `function` keyword followed by the function definition. You can also assign the function to a variable:

```js 
function() {
  // function body
}

let funcVar = function() {
  // function body
}
```

### Callbacks 

A _callback_ is a function that you pass to and call from another function. These are very helpful with asynchronous functions. The following is a simple example:

```js
let cbFunc = () => console.log("This is a callback");
console.log("Waiting one second to call the callback...");
setTimeout(cbFunc, 1000);
```

Output: 
```shell 
$ node js/functions.js 
Waiting one second to call the callback...
This is a callback
```