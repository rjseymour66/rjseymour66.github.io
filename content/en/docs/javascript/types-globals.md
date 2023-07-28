---
title: "Types and globals"
weight: 20
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