---
title: "Arrays"
weight: 120
description: >
  Helpful Array methods with examples.
---

You can create an array with the `new` keyword, but the array literal syntax is more common:

```js
let arr1 = new Array("purple", "green", "yellow");
let arr2 = new Array(10);       // creates an array with 10 undefined values

let bestAr = ["black", "orange", "pink"];
```

A single array can hold elements of different types:

```js
let arr = ["hi there", 5, true];
console.log(typeof arr[0]); // string
console.log(typeof arr[1]); // number
console.log(typeof arr[2]); // boolean
```

You can change the values of elements in a `const` array, but you cannot reassign it to a new array:

```js
// let array: can be reassigned
let lArr = [1, 2, 3, 4, 5];
lArr = ["one", "two", "three", "four", "five"];
console.log(lArr); // ['one', 'two', 'three', 'four', 'five']

// const array: elements can change, but the binding cannot be reassigned
const cArr = [1, 2, 3, 4, 5];
cArr = ["one", "two", "three", "four", "five"]; // TypeError: Assignment to constant variable
```

## Properties

### length

`length` returns the number of elements in an array:

```js
const fruit = ["apple", "banana", "pear"];
console.log(fruit.length); // 3
```

### push()

`push()` adds one or more elements to the end of an array and returns the new length:

```js
const arr = ["a", "b", "c"];
arr.push("d");               // adds "d" to the end, returns 4
const len = arr.push("e");   // len === 5
```

### splice()

`splice(start, deleteCount, ...items)` inserts elements at a specific index and optionally removes elements at the same position. Pass `0` as the second argument to insert without removing anything:

```js
let fruits = ['apple', 'banana', 'orange'];
fruits.splice(2, 0, 'kiwi', 'grape');
console.log(fruits); // ['apple', 'banana', 'kiwi', 'grape', 'orange']

fruits.splice(2, 1, 'pear'); // removes 1 element at index 2, inserts 'pear'
console.log(fruits); // ['apple', 'banana', 'pear', 'grape', 'orange']
```

Pass only the first two arguments to delete a range of elements. The second argument is the number of elements to remove, not an end index:

```js
let nums = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
nums.splice(1, 5);     // removes 5 elements starting at index 1
console.log(nums);      // [0, 6, 7, 8, 9]
```

### concat()

`concat()` combines two or more arrays into a new array without modifying the originals:

```js
let a = [1, 2, 3];
let b = [4, 5, 6];
let c = a.concat(b);
console.log(c); // [1, 2, 3, 4, 5, 6]
```

### pop()

`pop()` removes the last element from an array and returns it:

```js
let c = [1, 2, 3, 4, 5, 6];
c.pop();               // returns 6, c is now [1, 2, 3, 4, 5]
const p = c.pop();     // p === 5, c is now [1, 2, 3, 4]
```

### shift()

`shift()` removes the first element from an array and returns it:

```js
let c = [1, 2, 3, 4, 5, 6];
c.shift();             // returns 1, c is now [2, 3, 4, 5, 6]
const s = c.shift();   // s === 2, c is now [3, 4, 5, 6]
```

### delete

The `delete` operator removes an element at a given index but does not shift the remaining elements. The slot becomes empty:

```js
let nums = [0, 6, 7, 8, 9];
delete nums[0];
console.log(nums); // [<1 empty item>, 6, 7, 8, 9]
```

### find()

`find()` accepts a predicate function and returns the first element that satisfies it. If no element matches, it returns `undefined`:

```js
let ar = [1, 2, 3, 4, 5];

const lost  = ar.find(e => e === 6); // undefined
const found = ar.find(e => e === 1); // 1
```

### indexOf()

`indexOf()` returns the index of the first matching element, or `-1` if the element is not present. Pass a second argument to start the search at a specific position:

```js
let ar = [1, 2, 3, 4, 5];
ar.indexOf(4);     // 3
ar.indexOf(9);     // -1
ar.indexOf(1, 3);  // -1: searches from index 3 onward, so 1 is not found
```

### lastIndexOf()

`lastIndexOf()` returns the index of the last occurrence of an element in an array, or `-1` if the element is not present:

```js
let a = [1, 2, 3, 4, 5, 1];
a.lastIndexOf(1); // 5
```

### sort()

`sort()` orders array elements alphabetically or numerically in place:

```js
let a = [2, 4, 6, 3, 5, 2, 5, 8];
a.sort(); // [2, 2, 3, 4, 5, 5, 6, 8]

let names = ['bill', 'albert', 'sara', 'megan'];
names.sort(); // ['albert', 'bill', 'megan', 'sara']
```

### reverse()

`reverse()` reverses the order of array elements in place without sorting:

```js
let forward = [0, 1, 2, 3, 4, 5];
forward.reverse(); // [5, 4, 3, 2, 1, 0]
```

## Operators

### ...spread

The spread operator (`...`) expands an array into individual elements. Precede any array name with three dots to spread it:

```js
let a = [5, 6, 7];
let nums = [1, 2, 3, 4, ...a];

console.log(nums); // [1, 2, 3, 4, 5, 6, 7]
```

You can also pass an array as individual function arguments by spreading it at the call site:

```js
let addTwo = (x, y) => { return x + y; };
let two = [4, 7];

addTwo(...two); // 11
```

### rest parameter

The rest parameter (`...`) collects any remaining function arguments into an array. Unlike spread, which expands an array outward, rest gathers individual values inward:

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

`forEach()` executes a callback on every element in an array:

```js
const arr = [1, 2, 3];
arr.forEach(e => console.log(e));
// 1
// 2
// 3
```

### filter()

`filter()` accepts a predicate function and returns a new array containing only the elements for which the function returns `true`:

```js
let arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
let isEven = (el) => { return el % 2 == 0; };
let newArr = arr.filter(isEven);
console.log(newArr); // [2, 4, 6, 8, 10]
```

You can also pass an inline function directly to `filter()`. This example filters an `inventors` array for those born in the 1500s:

```js
const p = inventors.filter(inventor => {
    return inventor.year > 1499 && inventor.year < 1600;
});
```

### map()

`map()` executes a function on every element and returns a new array with the results. Arrow functions with a single expression do not require an explicit `return`. Multi-line arrow functions do:

```js
let arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
let mapArr = arr.map(x => x * 2);
console.log(mapArr); // [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
```

You can also transform arrays of objects. This example extracts first and last names from an `inventors` array:

```js
const firstAndLast = inventors.map(inventor => `${inventor.first} ${inventor.last}`);
```

### sort()

`sort()` with no arguments orders elements alphabetically or numerically in place. For custom ordering, pass a comparator function that accepts two elements — commonly named `a` and `b`. The function must return:

- A positive number if `a` should come after `b`
- A negative number if `a` should come before `b`

In practice: write an expression comparing `a` and `b`. Return `1` to place `a` after `b`, and `-1` to place `a` before `b`.

This example sorts an `inventors` array by birth year, oldest first. When `a.year` is greater than `b.year`, `a` should come later, so the function returns `1`:

```js
const ordered = inventors.sort((a, b) => {
    if (a.year > b.year) {
        return 1;
    } else {
        return -1;
    }
});

// Equivalent using a ternary:
const age = inventors.sort((a, b) => a.year > b.year ? 1 : -1);
```

For ascending numeric sorts, return `1` when `a` is greater than `b`. This example sorts inventors by years lived, longest first:

```js
const oldest = inventors.sort((a, b) => {
    const thisPerson = a.passed - a.year;
    const nextPerson = b.passed - b.year;
    return thisPerson > nextPerson ? -1 : 1;
});
```

You can also sort by derived string values. This example sorts a `people` array (formatted as `"Last, First"`) alphabetically by last name using destructuring:

```js
const alpha = people.sort((lastOne, nextOne) => {
    const [aLast, aFirst] = lastOne.split(", ");
    const [bLast, bFirst] = nextOne.split(", ");
    return aLast > bLast ? 1 : -1;
});
```

### reduce()

`reduce()` calculates a running total by applying a function to each element and an accumulator. It is equivalent to a `for` loop that tracks a running total:

```js
let totalYears = 0;

for (let i = 0; i < inventors.length; i++) {
    totalYears += inventors[i].year;
}
```

`reduce()` accomplishes the same goal declaratively. The first callback argument is the *accumulator* (the running total), and the second is the current element. The value after the function definition is the accumulator's initial value:

```js
const totalYears = inventors.reduce((total, inventor) => {
    return total + (inventor.passed - inventor.year);
}, 0);
```

You can also reduce into an object. This example counts occurrences of each item in an array:

```js
const data = ['car', 'car', 'truck', 'truck', 'bike', 'walk', 'car', 'van',
              'bike', 'walk', 'car', 'van', 'car', 'truck'];

const transportation = data.reduce((obj, item) => {
    if (!obj[item]) {   // if item is not already in obj
        obj[item] = 0;  // add item to the obj
    }
    obj[item]++;        // if it is in the obj, increment its value
    return obj;
}, {});
```

### Array.from() with NodeList

DOM query methods like `querySelectorAll()` return a `NodeList`, not an array. `Array.from()` converts a `NodeList` into a true array so you can chain array methods on it. The spread operator produces the same result:

```js
const category = document.querySelector('.mw-category');

// Convert NodeList to array
const links = Array.from(category.querySelectorAll('a'));

// Equivalent using spread:
// const links = [...category.querySelectorAll('a')];

const de = links
    .map(link => link.textContent)
    .filter(streetName => streetName.includes('de'));
```

### some()

`some()` returns `true` if at least one element in the array satisfies the predicate function, and `false` otherwise:

```js
const isAdult = people.some(person => {
    const currentYear = (new Date()).getFullYear();
    return (currentYear - person.year) >= 18;
});
```

### every()

`every()` accepts a predicate function and returns `true` only when every element satisfies it. It returns `false` as soon as any element fails:

```js
let arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
let isEven = (el) => { return el % 2 == 0; };
let everyArr = arr.every(isEven);
console.log(everyArr); // false
```

You can also pass an inline predicate. This example checks whether every person in an array is an adult:

```js
const allAdults = people.every(person => {
    const currentYear = (new Date()).getFullYear();
    return (currentYear - person.year) >= 18;
});
```

### find()

`find()` returns the first element that satisfies the predicate rather than a new array of all matches. It is like `filter()`, but stops at the first result:

```js
const comment = comments.find(comment => comment.id === 823423);
```

### findIndex()

`findIndex()` returns the index of the first element that satisfies the predicate, or `-1` if no match is found. It is useful when you need to remove or update a specific element by position:

```js
const index = comments.findIndex(comment => comment.id === 823423);
```

### lastIndexOf()

`lastIndexOf()` returns the index of the last occurrence of the argument, or `-1` if the value is not in the array:

```js
let arr = ["one", "two", "three", "four", "two"];
let lastTwo = arr.lastIndexOf("two");
console.log(lastTwo); // 4
```

### at()

`at(index)` returns the element at the given index and supports negative indexing (`-1` is the last element, `-2` is second to last). This is cleaner than `arr[arr.length - 1]`:

```js
const colors = ['red', 'green', 'blue', 'yellow'];

console.log(colors.at(0));    // 'red'
console.log(colors.at(-1));   // 'yellow': last element
console.log(colors.at(-2));   // 'blue': second to last

// Compare with the old pattern:
colors[colors.length - 1];    // 'yellow': verbose
colors.at(-1);                // 'yellow': clear
```

### flatMap()

`flatMap(fn)` maps each element to an array, then flattens one level. It is equivalent to `.map().flat()` but more efficient. Apply it when your transform function naturally produces multiple values per input:

```js
// Split sentences into individual words
const sentences = ['hello world', 'foo bar baz'];
const words = sentences.flatMap(s => s.split(' '));
// ['hello', 'world', 'foo', 'bar', 'baz']

// Expand order items into individual line items
const orders = [
    { id: 1, items: ['shirt', 'hat'] },
    { id: 2, items: ['shoes'] },
];
const allItems = orders.flatMap(order => order.items);
// ['shirt', 'hat', 'shoes']
```

## Real-world method chaining

Array methods compose well. Chaining `filter → map → reduce` is a common pattern for transforming and summarizing data:

```js
const transactions = [
    { type: 'purchase', amount: 29.99, category: 'clothing' },
    { type: 'refund',   amount: 15.00, category: 'electronics' },
    { type: 'purchase', amount: 89.95, category: 'electronics' },
    { type: 'purchase', amount: 12.49, category: 'clothing' },
    { type: 'purchase', amount: 249.00, category: 'electronics' },
];

// Total spent on electronics purchases only
const electronicsTotal = transactions
    .filter(t => t.type === 'purchase' && t.category === 'electronics')
    .map(t => t.amount)
    .reduce((sum, amount) => sum + amount, 0);

console.log(electronicsTotal.toFixed(2));   // 338.95

// Summarize total by category
const byCategory = transactions
    .filter(t => t.type === 'purchase')
    .reduce((totals, t) => {
        totals[t.category] = (totals[t.category] ?? 0) + t.amount;
        return totals;
    }, {});

// { clothing: 42.48, electronics: 338.95 }
```

Each method returns a new array without mutating the original. You can apply multiple transforms safely, and the chain reads top-to-bottom like a description of what you want.
