---
title: "Arrays"
# linkTitle: "CSS"
weight: 100
description: >
  Helpful Array methods with examples.
---

Can use the `new` keyword, but much easier to just assign an array to a variable:

```js 
arr1 = new Array("purple", "green", "yellow");
arr2 = new Array(10);       // creates an array with 10 undefined values

bestAr = ["black", "orange", "pink"];
```

A single array can hold elements of a different type: 
```js 
let arr = ["hi there", 5, true];
console.log(typeof arr[0]); // string
console.log(typeof arr[1]); // number
console.log(typeof arr[2]); // boolean
```

You can change the values of elements of a `const` array, but you can not reassign it to a new array:

```js 
// let array
let lArr = [1, 2, 3, 4, 5]
lArr = ["one", "two", "three", "four", "five"]

lArr
(5) ['one', 'two', 'three', 'four', 'five']

// cannot reassign const array
const cArr = [1, 2, 3, 4, 5]
cArr = ["one", "two", "three", "four", "five"]
VM1349:1 Uncaught TypeError: Assignment to constant variable.
    at <anonymous>:1:6
```

## Properties 

fruit = ["apple", "banana", "pear"]
(3) ['apple', 'banana', 'pear']
fruit.length
3


```js
arr.length          // length is a technically a property

// **************** push ****************//
// adds elements to the end of the array and
// it returns the new length of the array
array.push(val)
let lenArray = array.push(val)

// **************** splice ****************//
// adds elements at a specific index, and optionally deletes
// any elements with the second argument
// splice(start, numDel, els...)
> let fruits = ['apple', 'banana', 'orange']
> fruits.splice(2, 0, 'kiwi', 'grape')

> fruits
[ 'apple', 'banana', 'kiwi', 'grape', 'orange' ]

> fruits.splice(2, 1, 'pear')
[ 'kiwi' ]

> fruits
[ 'apple', 'banana', 'pear', 'grape', 'orange' ]

// Use splice and only the first two arguments to delete a range of elements from an array.
// Both arguments are inclusive.
> nums = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
> nums.splice(1, 5)
[ 1, 2, 3, 4, 5 ]
> nums
[ 0, 6, 7, 8, 9 ]


// **************** concat ****************//
// concat combines arrays
> let a = [1, 2, 3]
> let b = [4, 5, 6]
> let c = a.concat(b)
> c
[ 1, 2, 3, 4, 5, 6 ]


// **************** pop() ****************//
// deletes the last element from the array and returns it
> c.pop()
6
> c
[ 1, 2, 3, 4, 5 ]
> let p = c.pop()
> p
5
> c
[ 1, 2, 3, 4 ]


// **************** shift ****************//
// deletes the first element from the array
[ 1, 2, 3, 4, 5, 6 ]
> c.shift()
1
> s = c.shift()
2
> s
2
> c
[ 3, 4 ]

// **************** delete ****************//
// the delete operator removes an item from an array but does not 
// rearrange other elements
[ 0, 6, 7, 8, 9 ]
> delete nums[0]
true
> nums
[ <1 empty item>, 6, 7, 8, 9 ]


// **************** find ****************//
// takes a function that identifies an element in an array and returns 
// the first instance of the element if it is found. If does not find the 
// element, it returns 'undefined'

> let ar = [1, 2, 3, 4, 5]

> let lost = ar.find(e => e === 6)
> lost

> let found = ar.find(e => e === 1)
> found
1


// **************** indexOf() ****************//
// returns the index of the element in the array if the element
// is in the array, or a -1 if it is not present
[ 1, 2, 3, 4, 5 ]
> ar.indexOf(4)
3

> ar.indexOf(9)
-1

// add a second argument to indexOf() that specifies which position to begin
// searching the array
> ar.indexOf(1, 3)
-1


// **************** lastIndexOf() ****************//
// returns the index of the last occurence of an element in an array
[ 1, 2, 3, 4, 5, 1 ]
> a.lastIndexOf(1)
5


// **************** sort() ****************//
// sort orders arrays alphabetically or numerically
> a = [2, 4, 6, 3, 5, 2, 5, 8]
[
  2, 4, 6, 3,
  5, 2, 5, 8
]
> a.sort()
[
  2, 2, 3, 4,
  5, 5, 6, 8
]


[ 'bill', 'albert', 'sara', 'megan' ]
> names.sort()
[ 'albert', 'bill', 'megan', 'sara' ]

// **************** reverse() ****************//
// reverses the order of the array elements without sorting
> forward = [0, 1, 2, 3, 4, 5]
[ 0, 1, 2, 3, 4, 5 ]

> forward.reverse()
[ 5, 4, 3, 2, 1, 0 ]

```

## Operators

### ...spread

The `spread` operator spreads an array into individual elements. To do this, precede the name of an array with three dots (`...`):

```js
> let a = [5, 6, 7]
> let nums = [1, 2, 3, 4, ...a]

> nums
[
  1, 2, 3, 4,
  5, 6, 7
]
```

You can use the `spread` operator as function arguments:

```js 
> let addTwo = (x, y) => {return x + y}
> let two = [4, 7]

> addTwo(...two)
11
```

### rest parameter

The `rest` parameter is similar to spread, but it accepts a variable number of arguments:

```js 
function someFunction(param1, ...param2) {
  console.log(param1, param2);
}
someFunction("hi", "there!", "How are you?");

// Output:
// hi [ 'there!', 'How are you?' ]
```

## Methods

### forEach()

`.forEach()` executes a function on every item in an array:

```js
const arr = [1, 2, 3]
arr.forEach(e => console.log(e))

// Output:
1
2
3
```

### filter()

The `filter` method takes a function as an argument. This function must return a `Boolean`. Any item in the array that returns `true` will be in the filtered array:

```js 
let arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
let isEven = (el) => {return el % 2 == 0}
let newArr = arr.filter(isEven)
console.log(newArr);

// Output: 

shell 
[ 2, 4, 6, 8, 10 ]

// Array.prototype.filter()
// 1. Filter the list of inventors for those who were born in the 1500's

const p = inventors.filter(inventor => {
  return inventor.year > 1499 && inventor.year < 1600;
});
```

### map()

`map()` accepts a function, and executes that function on all elements in an array to return a new array:

```js 
let arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
let mapArr = arr.map(x => x * 2);
console.log(mapArr);

// Output: 
 
[ 2,  4,  6,  8, 10, 12, 14, 16, 18, 20 ]

// Array.prototype.map()
// 2. Give us an array of the inventors first and last names

const firstAndLast = inventors.map(inventor => `${inventor.first} ${inventor.last}`);
```

### sort()

`sort` takes a function that takes two arguments to compare elements in an array. These are commonly represented as `a` and `b`. If the function returns `1`, then `a` comes first in the resulting array. Otherwise, `b` comes first.

In the following example, we want to sort an array of inventor objects by age, where the first object was born the earliest, and the last object was born the latest. We compare `a.year` and `b.year`:
- If `a.year` is greater than `b.year`, we want `a` to come after `b`. So, we return a number greater than `0` (we return `1`).
- If `a.year` is less than `b.year`, we want it to come first, so we return `-1`. 

In other words, write an expression that compares `a` and `b`. If you want `a` to come before `b` based on that criteria, return `-1`. Otherwise, return `1`.

> Sorting numeric arrays depends on whether you want an ascending or descending sort. For ascending, you check whether `a` is greater than `b`. If `a` is greater, return `1` so it comes after `b`.

```js
// Array.prototype.sort()
// 3. Sort the inventors by birthdate, oldest to youngest

const ordered = inventors.sort((a, b) => {
  if (a.year > b.year) {
    return 1;
  } else {
    return -1;
  }
})

const age = inventors.sort(function (a, b) {
  return a.year > b.year ? 1 : -1;
});
```

### reduce()

The `reduce` function lets you calculate a sum using a running total. It is equivalent to a for loop that uses a variable to store the running total. For example:

```js
let totalYears = 0;

for (let i = 0; i < inventors.length; i++) {
  totalYears += inventors[i].year;
}
```

`reduce` accomplishes the same goal declaratively with arrow functions:

```js
const totalYears = inventors.reduce((total, inventor) => {
    return total + (inventor.passed - inventor.year);
}, 0);
```

Here, `total` is the _accumulator_, which is the variable that stores the running total. The function's return value is added to the accumulator. `inventor` is an element in the array, equivalent to the `inventors[i]` element in the for loop. After the function definition, there is a number. This is the starting value for the accumulator. For example, the initial value of `totalYears`  in the for loop.

Reduce an object:

```js
// 8. Reduce Exercise
// Sum up the instances of each of these
const data = ['car', 'car', 'truck', 'truck', 'bike', 'walk', 'car', 'van', 'bike', 'walk', 'car', 'van', 'car', 'truck'];

// start with a blank object
// if there is not an obj of that type, make an entry for it
// increment the number of items 
const transportation = data.reduce((obj, item) => {

    if (!obj[item]) {     // if item is not already in obj
      obj[item] = 0;      // add item to the obj
    }
    
    obj[item]++;          // if it is in the obj, increment its value
    return obj;
}, {});
```


### sort()

```js
// 5. Sort the inventors by years lived

const oldest = inventors.sort((a, b) => {
  const thisPerson = a.passed - a.year;
  const nextPerson = b.passed - b.year;

  return thisPerson > nextPerson ? -1 : 1;
});
```

### Array.from() with NodeList

```js
// 6. create a list of Boulevards in Paris that contain 'de' anywhere in the name
// https://en.wikipedia.org/wiki/Category:Boulevards_in_Paris
// const category = document.querySelector('.mw-category');

// NodeList to Array
Array.from()
const links = Array.from(category.querySelectorAll('a'));

// spread takes every item out of an iterable and puts them in an array.
// const links = [...category.querySelectorAll('a')];
const de = links
  .map(link => link.textContent)
  .filter(streetName => streetName.includes('de'));
```

### sort()

```js
// 7. sort Exercise
// Sort the people alphabetically by last name
const alpha = people.sort((lastOne, nextOne) => {
  const [aLast, aFirst] = lastOne.split(", ");
  const [bLast, bFirst] = nextOne.split(", ");
  return aLast > bLast ? 1 : -1;
});
```

### some()

```js
// Array.prototype.some() // is at least one person 19 or older?

const isAdult = people.some(person => {
  const currentYear = (new Date()).getFullYear()
  return (currentYear - person.year) >= 18
})
```

### every()

The `every()` method returns a `boolean` and accepts a function that returns a `boolean`. It returns `true` when each element in the array returns a `true`, and `false` otherwise:

```js 
let arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
let isEven = (el) => {return el % 2 == 0}
let everyArr = arr.every(isEven)
console.log(everyArr);

// Output: 
false

// Array.prototype.every() // is everyone 19 or older?

const allAdults = people.every(person => {
  const currentYear = (new Date()).getFullYear()
  return (currentYear - person.year) >= 18
})
```

### find()

```js
// Array.prototype.find()
// Find is like filter, but instead returns just the one you are looking for
// find the comment with the ID of 823423

const comment = comments.find(comment => comment.id === 823423);
```

### findIndex()

```js
// Array.prototype.findIndex()
// Find the comment with this ID
// delete the comment with the ID of 823423

const index = comments.findIndex(comment => comment.id === 823423);
```

### lastIndexOf

`lastIndexOf()` returns the index of the last occurence of the argument:

```js
let arr = ["one", "two", "three", "four", "two"]
let lastTwo = arr.lastIndexOf("two")
console.log(lastTwo);

// Output:
4
```

If you pass a value that is not in the array, it returns a `-1`.