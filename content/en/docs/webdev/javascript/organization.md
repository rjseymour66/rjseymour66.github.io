---
title: "Organization"
# linkTitle: "CSS"
weight: 20
description: >
  Different ways to organize your JS code.
---

## Objects and object constructors

### Object literals

```js
const myObject = {
  property: 'Value!',
  otherProperty: 77,
  "obnoxious property": function() {
    // do stuff!
  }
};

// dot notation
myObject.property; // 'Value!'

// bracket notation
myObject["obnoxious property"]; // [Function]
```

### Object constructors

Object constructors are functions that use `this` to bind properties (state) and functions (behavior) to objects (places in memory):

```js
function Player(name, marker) {
  this.name = name;
  this.marker = marker;
  this.sayName = function() {
    console.log(name)
  };
}
```

Create a new object with the `new` keyword:

```js
const player = new Player('jack', 'O');
player.sayName(); // 'jack'
```

The `new` keyword tells the browser that the `Player` function is a constructor, so look for the constructor property on the object's prototype

## Prototype

[javascript.info article](https://javascript.info/prototype-inheritance)

Objects have a special hidden property `[[Prototype]] that is either set to `null` or it references another object.

_The `prototype` is another object that the original object inherits from&mdash;the original object has access to all the `prototype`'s methods and properties._

To explain, take this example:

```js
function Player(name, marker) {
  this.name = name;
  this.marker = marker;
  this.sayName = function() {
    console.log(name)
  };
}

const player = new Player('jack', 'O');
```
The _original object_ in this scenario is `player`. This object inherits (has access to) all properties or functions of the `prototype`, which in this case is `Player`.

To get the prototype:

```js
// returns 'true' if the prototype of player is Player
Object.getPrototypeOf(player) === Player.prototype;
```
The `getPrototypeOf()` function returns the `.prototype` property of the Object Constructor. So, the Player prototype contains a constructor function, and `player` object was created with that constructor function.

> An equivalent but deprecated way to perform this check is with the `__proto__` object property:
>
>`player.__proto__ === Player.prototype`


### Inheritance

Prototypal inheritance is the sharing of properties and methods through generalized objects that you can clone and extend. This is different from class-based languages like Java, where the class provides a blueprint for the object.

Inheritance helps save memory because the Prototype object's properties and functions are stored in a single, shared object (the Prototype).

It also provides each object with methods on the `Object.prototype`. `Object.prototype` is the top of the object hierarchy&mdash;it is similar to the root `Object` class in Java. This inheritance hierarchy is called the _prototype chain_.To view the `Object.prototype`:

```js
console.log(Object.prototype);
// logs the following:

{constructor: ƒ, __defineGetter__: ƒ, __defineSetter__: ƒ, hasOwnProperty: ƒ, __lookupGetter__: ƒ, …}
constructor: ƒ Object()
hasOwnProperty:ƒ hasOwnProperty()
isPrototypeOf:ƒ isPrototypeOf()
propertyIsEnumerable: ƒ propertyIsEnumerable()
toLocaleString: ƒ toLocaleString()
toString: ƒ toString()
valueOf: ƒ valueOf()
__defineGetter__: ƒ __defineGetter__()
__defineSetter__: ƒ __defineSetter__()
__lookupGetter__: ƒ __lookupGetter__()
__lookupSetter__: ƒ __lookupSetter__()
__proto__: (...)
get __proto__: ƒ __proto__()
set __proto__: ƒ __proto__()
```

You can set the prototype of an object with the `Object.setPrototypeOf(target, source)` function:

```js
function Person(name) {
    this.name = name;
}

Person.prototype.sayName = function () {
    console.log(`Hello, I'm ${this.name}!`);
};

function Player(name, marker) {
    this.name = name;
    this.marker = marker;
}

Player.prototype.getMarker = function () {
    console.log(`My marker is '${this.marker}'`);
};

// Now make `Player` objects inherit from `Person`
Object.setPrototypeOf(Player.prototype, Person.prototype);
```
> Always make sure you set up inheritance before you add methods or you will get an error.


### Adding methods

It is a common best practice in JS to define only properties in the constructor, and then add methods to the prototype.

You can add functions to the prototype that are not available in the constructor:

```js
Player.prototype.newFunc = function () {
    console.log('This function is not in the Player constructor!');
};
```

Now, when you `console.log(Object.getPrototypeOf(player));`, you get `newFunc` and the `constructor`:

```js
{newFunc: ƒ, constructor: ƒ}
    newFunc: ƒ ()
    constructor: ƒ Player(name, marker)
    ...
```

You can use the `call()` method to chain constructors (copy properties from one constructor to another). The following example creates a `Person` object with a `greet` method:

```js
function Person(name) {
    this.name = name;
}

let me = new Person('jack');
console.log(me);    // Person {name: 'jack'}

Person.prototype.greet = function () {
    return `Hello, my name is ${this.name}`;
};

console.log(me.greet());    // Hello, my name is jack
```

Next, create a Teacher object that uses `call()` to copy the properties from the `Person` object constructor. Then, create a `dismiss()` function on the `Teacher` prototype:

```js
// create a teacher
function Teacher(name, subject) {
    Person.call(this, name);
    this.subject = subject;
}

// declare method unique to Teacher
Teacher.prototype.dismiss = function () {
    return `${this.name} dismisses ${this.subject} class at 2pm.`;
};
```
When you create a `Teacher` object, you can call the `dismiss()` method, but not the `Person`'s `greet()` method:

```js
const teacher = new Teacher('Ms. Smith', 'Math');

console.log(teacher);   // Teacher {name: 'Ms. Smith', subject: 'Math'}
console.log(teacher.dismiss()); // Ms. Smith dismisses Math class at 2pm.
console.log(teacher.greet());   // TypeError: teacher.greet is not a function at <line>

```

This is because `call()` only chains the constructors, not the methods on a prototype. To inherit a prototype's methods, use `Object.setPrototypeOf()`:

```js
// set the prototype of Teacher to Person
Object.setPrototypeOf(Teacher.prototype, Person.prototype);

console.log(teacher.greet());   // Hello, my name is Ms. Smith
```

### `Object.create()`

[Object.create() MDN docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create)

Object.create() is a static method on the `Object` creates a new object that uses an existing object as its prototype. Pass the prototype as an argument to the `create()` function. This works easiest when the prototype is an object literal:

```js

```

### Iterate over inherited properties

You can get all the property keys of an object:

```js
function Person(name) {
    this.name = name;
}

function Teacher(name, subject, age) {
    Person.call(this, name);
    this.subject = subject;
    this.age = age;
}

const person = new Person('steve');
console.log(Object.keys(person));   // ['name']

const teacher = new Teacher('rick', 'Math', 35);
console.log(Object.keys(teacher));  // ['name', 'subject', 'age']
```
You can also iterate over properties with the following syntax:

```js
for (let prop in teacher) console.log(prop);
// name
// subject
// age
```

## `this` keyword

All of this info is gleaned from [Gentle explanation of `this`](https://dmitripavlutin.com/gentle-explanation-of-this-in-javascript/).

In more "traditional" languages like Java, `this` represents the instance of the current object in the class method.

In Javascript, `this` is the context of a function invocation. Javascript has four types of function invocations:
- function: `alert('Invoke')`
- method: `console.log('Invoke this')`
- constructor: `new ObjectName('invoke', 'args')`
- indirect: `alert.call(undefined, 'Invoke')`

### Function invocation

A _function invocation_ is when an expression that evaluates to a function object is executed with parentheses that contain optional arguments.

#### Definition

`this` is the global object in a function invocation. For example, the `window` object is the global object in a browser:

```js
function myName(name) {
    console.log(this === window);   // true
    this.name = name; // assigns a name to the window
}

myName('Willie the window object'); // function invocation
console.log(window.name);   // Willie the window object
```
In `'strict'` mode, `this` is `undefined` in a function invocation.

#### Beware

When you use `this` in an inner function, its value does not change just because it is nested within a different context. For example, the following `calculate` function is nested within a method invocation. Because it is a function invocation, `this` is still equal to the global window object:

```js
const numbers = {
    numberA: 5,
    numberB: 10,

    sum: function () {
        console.log(`method invocation: ${this === numbers}`); // => true

        function calculate() {
            // this is window or undefined in strict mode
            console.log(`numbers: ${this === numbers}`); // => false
            console.log(`window: ${this === window}`);   // => true
            return this.numberA + this.numberB;
        }
        return calculate();
    }
};
```
The easiest way to resolve this is to use an [arrow function](#arrow-function), which resolves `this` to its executing context:

```js
const numbers = {
    numberA: 5,
    numberB: 10,

    sum: function () {

        const calculate = () => {
            console.log(`numbers: ${this === numbers}`); // => true
            return this.numberA + this.numberB;
        };
return calculate();
    }
};
```

### Method invocation

A _method invocation_ is when a expression that is a property of an object that evaluates to a function is invoked with parentheses and optional arguments.

#### Definition

`this` is the object that owns the method. This is true even for methods that are part of the `class` syntax or methods that are inherited from a prototype object. `this` is always the object before the dot (`.`) in method invocation.

#### Beware

If you assign a method to a lone variable, `this` is no longer the object that created the method&mdash;`this` is the global object:

```js
const lone = obj.MethodName();
lone(); // this is the global object or undefined in strict mode
```

This also occurs when you pass an object method as a parameter to a function. When passed as a parameter, a method is separated from an object:

```js
function Pet(type, legs) {
    this.type = type;
    this.legs = legs;

    this.logInfo = function () {
        console.log(this === myCat); // => false
        console.log(`The ${this.type} has ${this.legs} legs`);
    };
}

const myCat = new Pet('Cat', 4);
// logs "The undefined has undefined legs"
// or throws a TypeError in strict mode
setTimeout(myCat.logInfo, 1000);
```
You can fix this with an arrow function:

```js
function Pet(type, legs) {
    ...
    this.logInfo = () => {...};
}
```

### Constructor invocation

A _constructor invocation_ occurs when the `new` keyword is followed by an expression that evaluates to a function object with parentheses and optional arguments. For example:

```js
const pet = new Pet('dog', 'fido');
```

> A constructor call creates a new empty object, which inherits properties from the constructor's prototype.

#### Definition

`this` is the object that the constructor invocation creates. This rule also applies to class syntax constructors.

#### Beware

Don't forget to use the `new` keyword. Otherwise, the constructor correctly assigns the properties, but the absence of `new` makes the constructor a function invocation. `this` is the global window object in a function invocation.

### Indirect invocation

An _indirect invocation_ is when a function is called using the `.call()` or `.apply()` methods.

`call(thisArg, arg1, arg2, ...)`
: `thisArg` is the context of the invocation, and the other args are passed as arguments to the called function.

`.apply(thisArg, [arg1, arg2, ...])`
: `thisArg` is the context of the invocation that accepts an array of args passed to the called function.

#### Definition

`this` is the first argument in `.call()` or `.apply()`. For example, the context for the following function invocation's `this` defaults to the window object. You can change that context with `.call()` by passing the desired context to `.call()`, and then pass any arguments that the `sayHi()` function accepts as args following the new context. The same logic applies to `apply()`, but you pass the caller arguments as an array:

```js
const dog = { name: 'fido' };

function sayHi(greeting) {
    console.log(this === dog);  // => true
    return greeting + this.name;
}

sayHi.call(dog, 'Howdy, ');     // => Howdy, fido
sayHi.apply(dog, ['Howdy, ']);  // => Howdy, fido
```

A more practical example is calling the prototype's constructor in the inheriting object's constructor:

```js
function Person(name) {
    this.name = name;
}

function Teacher(name, subject, age) {
    Person.call(this, name);    // this === current Teacher object
    this.subject = subject;
    this.age = age;
}
```
### Bound function

A _bound_ function is a function whose context or arguments are bound to specific values with `.bind()`.

`.bind(thisArg, [arg1, arg2, ...])`
: `thisArg` is the context of the invocation that accepts an optional array of args to bind to.
  `.bind()` returns a new function with a predefined `this` value.

#### Definition

`this` is the first argument in `.bind()` when you are invoking a bound function. It lets you predefine the `this` value for the caller.

> **IMPORTANT**
> 
> `.bind()` creates a permanent context link that you cannot change with `.call()` or `.apply()`.

For example, the `concat()` function prepends the word 'Hello' to the argument and then returns the new string. Because it is a function invocation, `this` is the global window object. You can call `.bind()` with the new context:

```js
function concat(word) {
    'use strict';
    return `${this} ${word}`;   // => undefined
}

concat('World!');    // => undefined World!

const hello = concat.bind('Hello,');
hello('World');     // => Hello, World!
```
In the previous example, `.bind()` literally changes the value of `this`. `.bind()` lets you change the context on functions that share code and scope.


#### Beware

If you extract a bound function into a variable, the binding context does not persist.

### Arrow function

An arrow function is a short-form function that binds `this` to its current context. You cannot use the [`arguments` object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments) with arrow functions, you must use the rest syntax (`...args`).

#### Definition

`this` is the enclosing context where the arrow function is defined. It doesn't create its own context, it takes the context from the outer function that defines it. This means that it resolves `this` _lexically_.

> **IMPORTANT**
> Remember these two gotchas:
> - You cannot use an arrow function as a constructor.
> - An arrow function creates a permanent context link that you cannot change with `.call()` or `.apply()`.

#### Beware

You cannot use an arrow function to declare a method on an object. Instead, you must use `function` expressions. This is because the arrow function uses the global context (window object) as its context, not the caller. For example:

```js
function Period(hours, minutes) {
    this.hours = hours;
    this.minutes = minutes;
}

// INCORRECT! Uses global context
Period.prototype.format = () => {
    return this.hours + ' hours and ' + this.minutes + ' minutes';
};

// CORRECT! Uses caller's context during method invocation
Period.prototype.format = function () {
    return this.hours + ' hours and ' + this.minutes + ' minutes';
};

const walkPeriod = new Period(2, 30);
```