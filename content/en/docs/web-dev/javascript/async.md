---
title: "Asynchronous JS"
linkTitle: "Async"
weight: 80
description:
---

An asynchronous program must stop computing while it waits for data to arrive, or for some other event to occur:
- For example, JS in the browser is event-driven--it waits for the user to do something and then runs code
- Historically, these events were handled with callbacks

There are three important JS features that help with async programs:
- Promises: objects that represent the result of an async operation that has not yet completed
- async/await keywords: let you structure your Promise-based code as if it is synchronous
- async iterators and the `for/await` loop: work with streams of async events with loops, looks synchronous

## Callbacks

A callback is a function that is called into another function, and then executed at a later time, usually after some code runs:
- You don't know when the async function will run, but you know where

## Promises

A promise is an object that might produce a value in the future:
- Create a Promise that executes statements if it resolves with no errors, or executes other statements if it is rejected
- A promise is either pending, fulfilled, or rejected
- The return value from the promise's resolve function is sent to the `.then` function
- create with the `new Promise()` constructor
  - a Promise takes a resolve and reject function. The reject function is called if the Promise is fulfilled, reject is called if the Promise is rejected
  - You usually return the Promise from a function
  - When you have the Promise stored in a variable, you can all the `.then()` method to handle the fulfilled Promise (`resolve` function), and `.catch()` to handle the error (the `reject` function)
  - Instead of nesting callbacks, you can jsut chain `.then()` calls
- `Promise.all([promise1, promise2,...])` can call multiple Promises at once

```js
let p = new Promise((resolve, reject) => {
    let a = 1 + 1;
    if (a == 2) {
        resolve('Success');
    } else {
        reject('Failed');
    }
});

p.then((message) => {
    console.log('This is in the then ' + message);
}).catch((message) => {
    console.log('This is in the catch ' + message);
});
```

## async and await

These keywords let you write Promise-based async code tht looks like synchronous code blocks:
- Fulfilled Promises return a value like a synchronous function, and rejected Promises throw values like a synchronous function
  - `async` and `await` code hide the Promises syntax

### async expressions

You can only use `await` within functions that are declared asynchronous with the `async` keyword:
- Use `await` before you invoke a function that returns a Promise
- `await` does not cause your program to block -- the code is still asynchronous
- declaring a function with `async` means that the return value is always a Promise


```js
// --- All the functions are equivalent --- //
fetch(url)
    .then(resp => resp.json())
    .then(json => console.log(json.body));

async function fetchData(url) {
    try {
        let resp = await fetch(url);
        let json = await resp.json();
        console.log(json.body);
    } catch (error) {
        console.error("Error fetching data:", error);
    }
}
fetchData(url);

const fetchArrow = async (url) => {
    try {
        let resp = await fetch(url);
        let json = await resp.json();
        console.log(json.body);
    } catch (error) {
        console.log(`Error fetching data: ${error}`);
    }
};
fetchArrow(url);
```
