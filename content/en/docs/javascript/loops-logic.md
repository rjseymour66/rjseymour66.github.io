---
title: "Loops and logic"
weight: 50
description: >
  Loops and logic.
---

## Loops 

### while

```js
while (condition) {
  // code that gets executed as long as the condition is true
}
```

### do while 

`do while` loops always execute at least once: 

```js 
do {
  // code to be executed if the condition is true
} while (condition);
```

### for 
```js 
for (initialize variable; condition; statement) { 
  // code to be executed
}

for (let i = 0; i < cars.length; i++) {
  text += cars[i] + "<br>";
}
```

### for in 

```js 

let car = {
  model: "Golf",
  make: "Volkswagen",
  year: 1999,
  color: "black",
};

for (let prop in car) {
    console.log(car[prop])
}
// Output: 

// Golf
// Volkswagen
// 1999
// black

```
### for of 

You cannot manipulate values with this loop:

```js
for (let variableName of arr) {
  // code to be executed
  // value of variableName gets updated every iteration
  // all values of the array will be variableName once
}
```

## Logic 

### Switch statements

```js 
switch(expression) {
  case value1:
    // code to be executed
    break;
  case value2:
    // code to be executed
    break;
  case value-n:
    // code to be executed
    break;
  default:
    // code to be executed when no cases match
    break;
}

```