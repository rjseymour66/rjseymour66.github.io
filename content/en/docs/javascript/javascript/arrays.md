---
title: "Arrays"
# linkTitle: "CSS"
weight: 20
description: >
  Helpful Array methods with examples.
---

## filter()

```js
// Array.prototype.filter()
// 1. Filter the list of inventors for those who were born in the 1500's

const p = inventors.filter(inventor => {
  return inventor.year > 1499 && inventor.year < 1600;
});
```

## map()

```js
// Array.prototype.map()
// 2. Give us an array of the inventors first and last names

const firstAndLast = inventors.map(inventor => `${inventor.first} ${inventor.last}`);
```

## sort()

```js
// Array.prototype.sort()
// 3. Sort the inventors by birthdate, oldest to youngest

const age = inventors.sort(function (a, b) {
  return a.year > b.year ? 1 : -1;
});
```

## reduce()

```js
// Array.prototype.reduce()
// 4. How many years did all the inventors live all together?

// As a for loop
let totalYears = 0;

for (let i = 0; i < inventors.length; i++) {
  totalYears += inventors[i].year;
}

// with reduce()
const totalYears = inventors.reduce((total, inventor) => {
    return total + (inventor.passed - inventor.year);
}, 0);
```

Reduce an object:

```js
// 8. Reduce Exercise
// Sum up the instances of each of these
const data = ['car', 'car', 'truck', 'truck', 'bike', 'walk', 'car', 'van', 'bike', 'walk', 'car', 'van', 'car', 'truck'];

// start with a blank object
// if there is not an obj of that type, make an entry for it
// increment the number of items 
const transportation = data.reduce((obj, item) => {

    if (!obj[item]) {
    obj[item] = 0;
    }
    obj[item]++;
    return obj;
}, {});
```


## sort()

```js
// 5. Sort the inventors by years lived

const oldest = inventors.sort((a, b) => {
  const thisPerson = a.passed - a.year;
  const nextPerson = b.passed - b.year;

  return thisPerson > nextPerson ? -1 : 1;
});
```

## Array.from() with NodeList

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

## sort()

```js
// 7. sort Exercise
// Sort the people alphabetically by last name
const alpha = people.sort((lastOne, nextOne) => {
  const [aLast, aFirst] = lastOne.split(", ");
  const [bLast, bFirst] = nextOne.split(", ");
  return aLast > bLast ? 1 : -1;
});
```

## some()

```js
// Array.prototype.some() // is at least one person 19 or older?

const isAdult = people.some(person => {
  const currentYear = (new Date()).getFullYear()
  return (currentYear - person.year) >= 18
})
```

## every()

```js
// Array.prototype.every() // is everyone 19 or older?

const allAdults = people.every(person => {
  const currentYear = (new Date()).getFullYear()
  return (currentYear - person.year) >= 18
})
```

## find()

```js
// Array.prototype.find()
// Find is like filter, but instead returns just the one you are looking for
// find the comment with the ID of 823423

const comment = comments.find(comment => comment.id === 823423);
```

## findIndex()

```js
// Array.prototype.findIndex()
// Find the comment with this ID
// delete the comment with the ID of 823423

const index = comments.findIndex(comment => comment.id === 823423);
```