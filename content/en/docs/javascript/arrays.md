---
title: "Arrays"
weight: 30
description: >
  Arrays and array methods.
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

// push adds elements to the end of the array and
// it returns the new length of the array
array.push(val)
let lenArray = array.push(val)

// splice adds elements at a specific index, and optionally deletes
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

// concat combines arrays
> let a = [1, 2, 3]
> let b = [4, 5, 6]
> let c = a.concat(b)
> c
[ 1, 2, 3, 4, 5, 6 ]

// .pop() deletes the last element from the array and returns it
> c.pop()
6
> c
[ 1, 2, 3, 4, 5 ]
> let p = c.pop()
> p
5
> c
[ 1, 2, 3, 4 ]

// .shift() deletes the first element from the array
[ 1, 2, 3, 4, 5, 6 ]
> c.shift()
1
> s = c.shift()
2
> s
2
> c
[ 3, 4 ]

// the delete operator removes an item from an array but does not 
// rearrange other elements
[ 0, 6, 7, 8, 9 ]
> delete nums[0]
true
> nums
[ <1 empty item>, 6, 7, 8, 9 ]

// .find takes a function that identifies an element in an array and returns 
// the first instance of the element if it is found. If does not find the 
// element, it returns 'undefined'

> let ar = [1, 2, 3, 4, 5]

> let lost = ar.find(e => e === 6)
> lost

> let found = ar.find(e => e === 1)
> found
1

// indexOf() returns the index of the element in the array if the element
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

// .lastIndexOf() returns the index of the last occurence of an element in an array
[ 1, 2, 3, 4, 5, 1 ]
> a.lastIndexOf(1)
5



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

// reverse reverses the order of the array elements without sorting
> forward = [0, 1, 2, 3, 4, 5]
[ 0, 1, 2, 3, 4, 5 ]

> forward.reverse()
[ 5, 4, 3, 2, 1, 0 ]

```

## Functions 

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


## Multidimensional arrays 

```js
> a1 = [1, 2, 3]
[ 1, 2, 3 ]

> a2 = [4, 5, 6]
[ 4, 5, 6 ]

> a3 = [7, 8, 9]
[ 7, 8, 9 ]

> mAr = [a1, a2, a3]
[ [ 1, 2, 3 ], [ 4, 5, 6 ], [ 7, 8, 9 ] ]
```

## Built-in methods 

### forEach

```js
arr.forEach(func) // executes func on every element in the array

```

### filter

The `filter` method takes a function as an argument. This function must return a `Boolean`. Any item in the array that returns `true` will be in the filtered array:

```js 
let arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
let isEven = (el) => {return el % 2 == 0}
let newArr = arr.filter(isEven)
console.log(newArr);
```

Output: 

```shell 
[ 2, 4, 6, 8, 10 ]
```

### every

The `every()` method returns a `boolean` and accepts a function that returns a `boolean`. It returns `true` when each element in the array returns a `true`, and `false` otherwise:

```js 
let arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
let isEven = (el) => {return el % 2 == 0}
let everyArr = arr.every(isEven)
console.log(everyArr);
```
Output: 

```shell 
false
```
### copyWithin

The `copyWithin()` method can replace a part of an array with another part of an array:

### map

`map()` accepts a function, and executes that function on all elements in an array to return a new array:

```js 
let arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
let mapArr = arr.map(x => x * 2);
console.log(mapArr);
```

Output: 

```shell 
[ 2,  4,  6,  8, 10, 12, 14, 16, 18, 20 ]
```
### lastIndexOf

`lastIndexOf()` returns the index of the last occurence of the argument:

```js
let arr = ["one", "two", "three", "four", "two"]
let lastTwo = arr.lastIndexOf("two")
console.log(lastTwo);
```

Output:

```shell 
4
```

If you pass a value that is not in the array, it returns a `-1`.