---
title: "Basics"
weight: 30
description: >
  Types and globals.
---

## Comments

```js
// Single comment
/* Multi-line comment. Make sure this is the correct 
commenting style.
*/
```

## Essentials 

```js
let name = "My name";
name = "Rename";
```

`let` and `const` have block scope, `var` has global or function scope.

Prefer `const` by default — use `let` only when you need to reassign a value. This makes your intent clear and prevents accidental reassignment:

```js
const MAX_RETRIES = 3;          // value never changes
let currentAttempt = 0;         // will increment in a loop

// const with objects/arrays: the reference is fixed, but contents are mutable
const user = { name: 'Alice' };
user.name = 'Bob';              // allowed
user = {};                      // TypeError: Assignment to constant variable
```

## Strings 

```js
let doubleQuote = "Don't you dare do that"
let name = "Jimmy"
// template strings
let sentence = `My name is ${name}`;
```

```js 
// Combine strings
s1.concat(s2)
```

```js
// split string into array by a sep
// s.split("sep")


let s = "This is going to be an array"
let arr = s.split(" ")

console.log(arr);
```

Output: 
```shell 
[
  'This',  'is',
  'going', 'to',
  'be',    'an',
  'array'
]
```

### join

Convert array to string: 

```js
let letters = ["s", "t", "r", "i", "n", "g"]
let word = letters.join("")
let comma_word = letters.join()

console.log("word:", word);
console.log("comma_word:", comma_word);
```

Output: 
```shell 
word: string
comma_word: s,t,r,i,n,g
```
### indexOf and lastIndexOf

Find the index of a letter or word in a string with `indexOf()` (or `lastIndexOf`):

```js
let  word = "Javascript"
let r = word.indexOf("r")
let x = word.indexOf("x")

console.log(r);
console.log(x);
```
If the letter is not in the string, then it returns a `-1`:
Output: 
```shell 
6
-1
```

If you want to use a regex, use the `search()` method. It behaves exactly as `indexOf()`. Note that `indexOf()` is faster than `search`.

### charAt

`charAt` finds the character at the specified index:

```js
let line = "To be, or not to be"
let c = line.charAt(1)

console.log(c); // o
```

Output: 
```shell 
o
```

### slice

`slice(start, end)` can create a new string from an existing string--it does not alter the existing string. If you omit `end`, it slices from `start` to the end of the string.

`start` is inclusive. `end` is uninclusive:


```js
let s = "Mississippi"

let sub1 = s.slice(2, 4)
let sub2 = s.slice(5, 7)
let sub3 = s.slice(4)

console.log(sub1);    // iss
console.log(sub2);    // ss
console.log(sub3);    // issippi
```
### replace 


`replace(old, new)` changes only the first occurence, and `replaceAll(old, new)` replace every occurence:

```js
let old = "This oh is oh my hello to oh you doh!"

let rep = old.replace("oh", "x")
let repAll = old.replaceAll("oh", "x")

console.log(rep);     // This x is oh my hello to oh you doh!
console.log(repAll);  // This x is x my hello to x you dx!
```
### toUpperCase and toLowerCase
`toUpperCase` and `toLowerCase`:


```js
let lower = "abcdefghijklmnopqrstuvwxyz"
let upper = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"

let lowToUpper = lower.toUpperCase();
let upToLower = upper.toLowerCase();

console.log(lowToUpper);  // ABCDEFGHIJKLMNOPQRSTUVWXYZ
console.log(upToLower);   // abcdefghijklmnopqrstuvwxyz
```
### trim, trimStart, trimEnd

`trim()` removes whitespace from both ends of a string. `trimStart()` and `trimEnd()` remove from one side only. Useful when handling user input from form fields:

```js
let raw = "   user@example.com   ";
console.log(raw.trim());        // "user@example.com"
console.log(raw.trimStart());   // "user@example.com   "
console.log(raw.trimEnd());     // "   user@example.com"
```

### includes

`includes(str)` returns `true` if the string contains `str`. Unlike `indexOf`, it returns a Boolean instead of an index — cleaner for simple checks:

```js
let url = "https://api.example.com/users";

if (url.includes('/users')) {
    console.log('User API endpoint');
}
// true
```

### padStart and padEnd

`padStart(length, char)` and `padEnd(length, char)` pad a string to a minimum length. Useful for formatting numbers or building fixed-width output:

```js
// Zero-pad an order number for display
let orderId = String(42);
console.log(orderId.padStart(6, '0'));  // "000042"

// Progress indicator
let pct = '75%';
console.log(pct.padEnd(10, '-'));       // "75%-------"
```

### startsWith and endsWith

`startsWith(word)` and `endsWith(word)` return a boolean that indicates whether the string starts or ends with `word`. Note that it is case-sensitive, and punctuation counts as part of the string:

```js
let s = "The string is the thing."

let starts = s.startsWith("the")
let ends = s.endsWith("thing.")

console.log(starts);  // false
console.log(ends);    // true
```

## Numbers

```js
let intNr = 1;
let decNr = 1.5;
let expNr = 1.4e15;
let octNr = 0o10; //decimal version would be 8
let hexNr = 0x3E8; //decimal version would be 1000
let binNr = 0b101; //decimal version would be 5
```

### Global functions

Many of these are global functions:

```js
isNan(x)  // is x a number, returns boolean
isInteger(x)  // is x a Number

let x = 1.2345
let twoDec = x.toFixed(2) // 1.23
let precision = x.toPrecision(2)  // 1.2
```

## Boolean 

Use `true` or `false`.

## Data types

```js
let str = "Hello";
let nr = 7;
let bigNr = 12345678901234n;
let bool = true;
let sym = Symbol("unique");
let undef = undefined;
let unknown = null;
 
console.log("str", typeof str);
console.log("nr", typeof nr);
console.log("bigNr", typeof bigNr);
console.log("bool", typeof bool);
console.log("sym", typeof sym);
console.log("undef", typeof undef);
console.log("unknown", typeof unknown);
```

### Converting (casting)

```js
String(val)
Number(val)
Boolean(val)
```

## Equality

The double equals (`==`) checks for equal value, not data type. The triple equals (`===`) checks both value and data type:

```js 
let x = 5
let y = '5'

x == y
// true

x === y
// false
```


## Type coercion gotchas

The `==` operator silently converts types before comparing. This produces results that are hard to predict:

```js
0 == false        // true  — 0 is falsy
'' == false       // true  — empty string is falsy
null == undefined // true  — special case
[] == false       // true  — array coerces to ''
[] == ![]         // true  — both coerce to 0

// Avoid == entirely. Use === everywhere.
0 === false       // false — different types
```

The values `0`, `''`, `null`, `undefined`, `NaN`, and `false` are all _falsy_. Everything else is _truthy_. This matters for `if` checks:

```js
const username = '';

// Dangerous: 0 is a valid count, but this branch won't run
if (!username) {
    console.log('No username provided');
}

// Explicit check is clearer
if (username === '') {
    console.log('No username provided');
}
```

## Optional chaining and nullish coalescing

Optional chaining (`?.`) lets you safely access nested object properties without throwing a `TypeError` when an intermediate value is `null` or `undefined`:

```js
// Without optional chaining — crashes if user is null
const city = user.address.city;

// With optional chaining — returns undefined instead of throwing
const city = user?.address?.city;

// Also works for method calls and array access
const firstTag = post?.tags?.[0];
const label = button?.getAttribute?.('aria-label');
```

Nullish coalescing (`??`) provides a fallback when a value is `null` or `undefined` — but *not* when it is `0` or `''`:

```js
const timeout = config.timeout ?? 3000;   // use 3000 only if timeout is null/undefined
const retries = config.retries ?? 3;

// Compare with ||, which also triggers on 0 and ''
const timeout2 = config.timeout || 3000;  // replaces 0 with 3000 — usually wrong
```

Combine them for safe property access with a default:

```js
const displayName = user?.profile?.nickname ?? 'Anonymous';
```

## URI and URL encoding 

URLs use _percent encoding_. For example, a space is encoded as `%20`:

```js 
let uri = "https://www.example.com/submit?name=rick randolph";
console.log("URI:", uri);

let encoded_uri = encodeURI(uri)
console.log("Encoded:", encoded_uri);

let decoded_uri = decodeURI(encoded_uri)
console.log("Decoded:", decoded_uri);
```

Output: 
```shell 
URI: https://www.example.com/submit?name=rick randolph
Encoded: https://www.example.com/submit?name=rick%20randolph
Decoded: https://www.example.com/submit?name=rick randolph
```




## Math 

```js
let highest = Math.max(2, 56, 12, 1, 233, 4); // 233
let lowest = Math.min(2, 56, 12, 1, 233, 4);  // 1
let result = Math.sqrt(64);   // 8
let result2 = Math.pow(5, 3); // 5^3 = 125

let x = Math.round(6.4);  // 6
let y = Math.round(8.8);  // 9
let x = Math.ceil(6.4);   // 7
let y = Math.ceil(8.8);   // 9
let x = Math.floor(6.4);  // 6
let y = Math.floor(8.8);  // 8
```


## Date 

Create dates with the built-in `Date` object and its methods:

```js

let today = new Date()    // 2023-03-09T15:45:44.414Z
let secsSinceUnixEpoch = Date.now()   // 1678376744419
let unixEpochPlus1000 = new Date(1000)  // 1970-01-01T00:00:01.000Z
let stringDate = new Date("Thurs Mar 09 2023 10:43:00 GMT+0200"); // 2023-03-09T08:43:00.000Z


let specificDate = new Date(2023, 2, 9, 10, 45, 0, 0);  // 2023-03-09T15:45:00.000Z
/*** accepted formats ***/
// new Date(year, monthIndex)
// new Date(year, monthIndex, day)
// new Date(year, monthIndex, day, hours)
// new Date(year, monthIndex, day, hours, minutes)
// new Date(year, monthIndex, day, hours, minutes, seconds)
// new Date(year, monthIndex, day, hours, minutes, seconds, milliseconds)
```

Get and set date elements: 

```js
let d = new Date();
console.log("Day of week:", d.getDay());              // Day of week: 4
console.log("Day of month:", d.getDate());            // Day of month: 9
console.log("Month:", d.getMonth());                  // Month: 2
console.log("Year:", d.getFullYear());                // Year: 2023
console.log("Seconds:", d.getSeconds());              // Seconds: 15
console.log("Milliseconds:", d.getMilliseconds());    // Milliseconds: 672
console.log("Time:", d.getTime());                    // Time: 1678377375672

// change these methods to set* to change the date:
d.setFullYear(2040)   // 2040-03-09T15:58:22.622Z
```
### Parse dates

`parse()` can parse strings into epoch dates.


```js
let d1 = Date.parse("June 5, 2021");  // 1622851200000
let d2 = Date.parse("6/5/2021");      // 1622851200000
```

### Date to string

`toDateString()` and `toLocaleDateString()` convert an epoch date into a string:

```js 
let d = new Date();

console.log(d.toDateString());        // Thu Mar 09 2023
console.log(d.toLocaleDateString());  // 3/9/2023
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

A switch statement is a good fit when you have a fixed set of named states. A real-world example: mapping HTTP status codes to user-facing messages:

```js
function statusMessage(code) {
    switch (code) {
        case 200: return 'OK';
        case 201: return 'Created';
        case 400: return 'Bad request — check your input';
        case 401: return 'Unauthorized — please log in';
        case 403: return 'Forbidden';
        case 404: return 'Not found';
        case 500: return 'Server error — try again later';
        default:  return `Unexpected status: ${code}`;
    }
}
```

## Functions

Functions are reusable blocks of code that execute when called. Follow these naming conventions:
- Use camelCase.
- Use a verb to describe what the function does: `fetchUser`, `validateEmail`, `formatDate`.

```js
// Standard function declaration — hoisted to the top of its scope
function greet(name) {
    return `Hello, ${name}!`;
}

// Function expression — not hoisted, assigned to a variable
const greet = function(name) {
    return `Hello, ${name}!`;
};

// Parameters do not need type annotations
function add(x, y) {
    return x + y;
}
```

### Argument object

Each function has an `arguments` object that stores every value passed in, accessible by index. This predates rest parameters — prefer the modern `...args` syntax in new code:

```js
// Legacy — avoid in new code
function sumAll() {
    let total = 0;
    for (let i = 0; i < arguments.length; i++) {
        total += arguments[i];
    }
    return total;
}

// Modern — use rest parameters instead
function sumAll(...nums) {
    return nums.reduce((total, n) => total + n, 0);
}

sumAll(1, 2, 3, 4);  // 10
```

### Hoisting and strict mode

Hoisting moves `var` declarations and function declarations to the top of their scope at parse time. This makes code hard to reason about:

```js
// This works because the declaration is hoisted — but it's confusing
console.log(greet('Alice'));  // 'Hello, Alice!'

function greet(name) {
    return `Hello, ${name}!`;
}

// var is also hoisted — initialized as undefined
console.log(score);   // undefined, not ReferenceError
var score = 10;
```

Use `let` and `const` to avoid hoisting surprises. Add `'use strict'` at the top of files that don't use ES modules — strict mode disables several error-prone behaviors:

```js
'use strict';
```

### Default parameters

If you call a function without providing an argument, JavaScript assigns that parameter `undefined`. Default parameter values guard against this:

```js
// Without default — breaks when timeout is undefined
function fetchWithTimeout(url, timeout) {
    // timeout is NaN if not passed
}

// With defaults — safe and self-documenting
function fetchWithTimeout(url, timeout = 5000, retries = 3) {
    console.log(`Fetching ${url} with ${timeout}ms timeout, ${retries} retries`);
}

fetchWithTimeout('/api/data');
// Fetching /api/data with 5000ms timeout, 3 retries
```

## Special functions

### Arrow functions

Arrow functions are a concise syntax for function expressions. They do not bind their own `this` — they inherit it from the surrounding scope (see the `this` section in Objects):

```js
// No params
const logTimestamp = () => console.log(Date.now());

// One param — parentheses optional
const double = n => n * 2;

// Multiple params
const add = (a, b) => a + b;

// Multi-line body requires curly braces and explicit return
const buildUrl = (base, path, id) => {
    const normalized = path.startsWith('/') ? path : `/${path}`;
    return `${base}${normalized}/${id}`;
};
```

Arrow functions are most useful as callbacks, where the concise syntax reduces noise:

```js
const prices = [9.99, 14.49, 3.75, 22.00];

const discounted = prices
    .filter(p => p < 15)
    .map(p => p * 0.9)
    .map(p => p.toFixed(2));

console.log(discounted);  // ['8.99', '13.04', '3.38']
```

### Immediately invoked function expressions (IIFE)

An IIFE wraps a function in parentheses and invokes it immediately. Use IIFEs to create a private scope or to initialize something once without polluting the global namespace:

```js
// Basic IIFE
(function () {
    console.log('Runs immediately');
})();

// IIFE with initialization — common before ES modules
const config = (function () {
    const apiBase = 'https://api.example.com';
    const version = 'v2';

    return {
        endpoint: (path) => `${apiBase}/${version}/${path}`,
        timeout: 5000,
    };
})();

console.log(config.endpoint('users'));  // https://api.example.com/v2/users
```

### Anonymous functions

An anonymous function has no name. You assign it to a variable or pass it directly as a callback:

```js
// Assigned to a variable
const multiply = function(a, b) {
    return a * b;
};

// Passed inline as a callback
setTimeout(function() {
    console.log('Delayed output');
}, 1000);
```

In modern code, prefer arrow functions for anonymous callbacks — they are shorter and avoid `this` binding issues.

## Error handling

Use `try/catch` to handle errors from code that might fail, such as network requests, JSON parsing, or operations on potentially null values. The optional `finally` block runs whether or not an error occurs:

```js
try {
    somethingRisky();
} catch (e) {
    if (e instanceof TypeError) {
        // wrong type passed to a function
    } else if (e instanceof RangeError) {
        // value out of allowed range
    } else if (e instanceof EvalError) {
        // eval() error
    } else {
        throw e;  // re-throw unknown errors
    }
} finally {
    console.log('Runs regardless of success or failure');
}
```

A real-world example: parsing user-supplied JSON safely:

```js
function parseConfig(raw) {
    try {
        const config = JSON.parse(raw);
        if (!config.apiKey) throw new TypeError('Missing apiKey in config');
        return config;
    } catch (e) {
        if (e instanceof SyntaxError) {
            console.error('Invalid JSON — could not parse config');
        } else {
            console.error(e.message);
        }
        return null;
    }
}
```

You can also throw errors deliberately to signal invalid states:

```js
function divide(a, b) {
    if (b === 0) throw new RangeError('Division by zero');
    return a / b;
}

try {
    divide(10, 0);
} catch (e) {
    console.error(e.message);  // Division by zero
}
```