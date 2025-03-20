---
title: "Concurrency"
weight: 90
description: >
---


## Callbacks

A callback is a function that takes another function as an argument. The argument function is called when the code in the calling function completes.

For example, the following `getGrade` function calls the `judge` function as soon as the `switch` statement completes. Note that the parameter does not inlude the `()`--you invoke the callback in the function body with the `()`:

```js
function judge(grade) {
    switch (true) {
        case grade == "A":
            console.log("You got an", grade, ": amazing!");
            break;
        case grade == "B":
            console.log("You got a", grade, ": well done!");
            break;
      // more cases...
        default:
            console.log("An", grade, "! What?!");
    }
}
function getGrade(score, callback) {
    let grade;
    switch (true) {
        case score >= 90:
            grade = "A";
            break;
        case score >= 80:
            grade = "B";
            break;
        // more cases
        default:
            grade = "F";
    }
    callback(grade);
}
getGrade(85, judge);
```

Callbacks are useful for asynchronous code, where you are waiting for results from a database call before you execute the callback that processes the data. Examples include the native `setTimeout` and `setInterval` functions--they accept a function (callback) and length of time as parameters.

## Promises 

A promise is a special object that connects code that needs to produce a result, and code that needs to use that result it its next step.

When you create a promise, you do not know what its eventual value will be. For example, in the following code, `promise` is either going to succeed and run the `resolve` function, or it fails and calls the `reject` function:

```js 
let promise = new Promise(function (resolve, reject) {
    let x = 20;
    if (x > 10) {
        resolve(x); // on success
    } else {
        reject("Too low");  // on error
    }
});
```
If a promise has neither succeeded or failed, it is called 'pending'.

After you define the Promise, you can use `then` to create a promise that works with the result, whether the original Promise succeeds or fails:

```js
promise.then(
    function (value) {
        console.log("Success:", value);
    },
    function (error) {
        console.log("Error:", error);
    }
);
```
You can chain `then` calls to the original Promise declaration. When you chain `then` calls, the `return` statement is the input for the next `then` function. Use the `catch` function to execute if any of the chained `then` Promises are rejected: 

```js 
const promise = new Promise((resolve, reject) => {
    resolve("success!");
})
    .then(value => {
        console.log(value);
        return "we";
    })
    .then(value => {
        console.log(value);
        return "can";
    })
    .then(value => {
        console.log(value);
        return "chain";
    })
    .then(value => {
        console.log(value);
        return "promises";
    })
    .then(value => {
        console.log(value);
    })
    .catch(value => {
        console.log(value);
    })
```

## async and await TODO

Use `async` to make a function return a Promise. You can work with this Promise with `then` and `catch` functions, or you can use the `await` keyword to wait until the Promise completes. 

Example that I need to research: 

```js 
function saySomething(x) {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve("something" + x);
        }, 2000);
    });
}
async function talk(x) {
    const words = await saySomething(x);
    console.log(words);
}
talk(2);
talk(4);
talk(8);
```

## Event loop 

Javascript is a single-threaded language. This means that only one thing can happen at a time: tasks must wait for previously executing tasks to complete.

The single executor is called the _event loop_. The event loop executes all of the Javascript work. Even though JavaScript is single-threaded, it achieves concurrency with the _call stack_ and the _callback queue_.

### Call stack and callback queue 

The call stack is a queue of all actions that are pending execution. The event loop constantly monitors the call stack and completes pending tasks, one-by-one, from the top of the stack. 

When you use a callback, Javascript outsources the callback task to the browser's web API. When the callback completes, it goes into the callback queue. When the call stack is empty, the event loop checks the callback queue to see if there is any pending work. If there are callbacks waiting in the queue, they are executed one-by-one, from the top of the queue. After each callback is executed, the event loop checks the call stack to see if there is any work to do before executing another callback. 

