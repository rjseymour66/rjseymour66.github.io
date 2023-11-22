---
title: "Organization"
# linkTitle: "CSS"
weight: 20
description: >
  Different ways to organize your JS code.
---

## Links

- [Design patterns](https://www.patterns.dev/)

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
const person = {
    init: function (phrase) {
        this.phrase = phrase;
        return this;
    },
    speak: function () {
        console.log(this.phrase);
    }
};

const jack = Object.create(person).init('My name is Jack');
jack.speak();   // => My name is Jack
person.isPrototypeOf(jack); // => true
```
`Object.create` is the preferred method when you create an object with a prototype. It is much more aligned with the prototype philosophy than the `new` keyword, which is derived from class-based object instantiation. It is also easier to use than `setPrototypeOf()` when you create objects with prototypes.

#### Implementation

Below is a sample implementation of `Object.create()`:

```js
function objectCreate(proto) {
    const obj = {};
    Object.setPrototypeOf(obj, proto);
    return obj;
}
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

## Scope

- `var` is _function-scoped_, which means they are available anywhere within a function. This complicates `var` usage in nested functions like closures.
- `let` and `var` are _block-scoped_. This limits their scope to curly braces (`{ }`), which provides more control over scope and is optimal for nested functions.

## Closures

A closure is a combination of a function and its lexical environment (its surrounding state).

```js
let makeAdding = (firstNumber) => {
  const first = firstNumber;

  return function resulting(secondNumber) {
    const second = secondNumber;
    return first + second;
  };
};

const add5 = makeAdding(5);
console.log(add5(3)); // => 8
```

## Object shorthand notation

The following factor function uses the object shorthand notation:

```js
let createUser = (name) => {
  const discordName = `@${name}`;
  return { name, discordName };
};
```

When you have a variable with the same name as the object property that you are assigning it, you can use the shorthand. For example, these return statements are equivalent:

```js
// object initializer
return { name: name, discordName: discordName }; // => {name: 'jack', discordName: '@jack'}

// object shorthand
return { name, discordName }; // => {name: 'jack', discordName: '@jack'}
```

Note that the longer method doesn't require quotes (`" "`) around the object key names.

## Destructuring

[MDN article](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)

Destructuring is when you extract values from an object or arrary.

For objects, you can extract one of the object's properties into a variable that has the same name as that object property:

```js
const user = { name: "jack", age: 40 };

const { name, age } = user;
console.log(name); // => jack
console.log(age); // => 40
```

For arrays, you can extract elements in the array into variables. The array element is assigned to the variable with the same index:

```js
const array = [1, 2, 3, 4, 5];

const [zero, one, two, three, four] = array;

console.log(zero); // => 0
console.log(one); // => 1
console.log(two); // => 2
console.log(three); // => 3
console.log(four); // => 4
```

## Factory functions

### vs constructors

Prefer a factory function over a constructor because:

- A constructor looks like a normal JS function, but it does not behave like one. It requires the `new` keyword---if you forget to add this keyword, then you create hard-to-debug error messages.
- How a constructor interacts with `instanceof`. `instanceof` checks the presence of an object's prototype in the entire prototype chain. This does not confirm if the object was created with that constructor because you can reassign a constructor's prototype.

### Implementation

Prefer _factory functions_ because of the following:

- They do not use the `new` keyword--they're just functions that return objects.
- They can levy the power of closures.
- They do not use the prototype. The prototype model incurs a performance penalty when you create lots of objects.

```js
const User = function (name) {
  this.name = name;
  this.discordName = `@${name}`;
};

let createUser = (name) => {
  const discordName = `@${name}`;
  return { name, discordName };
};

let constructor = new User("jack");
console.log(constructor); // => User {name: 'jack', discordName: '@jack'}

let factoryFunc = createUser("jack");
console.log(factoryFunc); // => {name: 'jack', discordName: '@jack'}
```

### Private variables and functions

You can create private variables with closures. In the following factory function, the employee record keeps the salary information private. There are functions to retrieve the salary value (not the actual salary variable), and a function to increase the salary:

```js
let createEmployee = (name) => {
  const email = `${name}@company.com`;

  let salary = 100_000;
  const getSalary = () => salary;
  const giveMeritIncrease = () => (salary *= 1.05);

  return { name, email, getSalary, giveMeritIncrease };
};

const jack = createEmployee("jack");
jack.giveMeritIncrease();
jack.giveMeritIncrease();

const currentSalary = jack.getSalary();
console.log(currentSalary); // => 110250
```

This is a closure because the `getMerit` and `giveMeritIncrease` have access to the `salary` variable becuase of lexical scope.

### Inheritance and `Object.assign`

You can leverage inheritance with factory functions without using prototypes.

The following `createManager` function extends the the `createEmployee` function from the [Private variables and functions](#private-variables-and-functions) section. It accepts an additional argument, `pto`, and uses destructuring to extract the `email` and `getSalary` from the `createEmployee` function. Then, it adds a new function to increase the `pto`, `increasePTO`:

```js
let createManager = (name, pto) => {
  const { email, getSalary, giveMeritIncrease } = createEmployee(name);

  const increasePTO = () => (pto *= 1.1);
  return { name, email, getSalary, giveMeritIncrease, increasePTO };
};

const boss = createManager("sally", 15);
boss.giveMeritIncrease();
boss.giveMeritIncrease();
boss.giveMeritIncrease();
boss.giveMeritIncrease();
console.log(boss.getSalary()); // => 121550.625
console.log(boss.increasePTO()); // 16.5
```

You can also use `Object.assign`, a static method that copies all properties from a source object to a target object:

```js
Object.assign(target, source1, source2, /* …, */ sourceN);
```

The following function accepts the same parameters, but it uses `createEmployee` to create an employee. Then, it uses `Object.assign()` to create an empty target object, and then copies the manager and public methods into the the empty object:

```js
let createManager = (name, pto) => {
  const manager = createEmployee(name);

  const increasePTO = () => (pto *= 1.1);
  return Object.assign({}, manager, { increasePTO });
};
```

## Module pattern (IIFEs)

[MDN docs IIFE](https://developer.mozilla.org/en-US/docs/Glossary/IIFE)

You can hide variables and functions by wrapping a factory function in an immediately-invoked function expression (IIFE). Its 'immediately-invoked' because you place the `()` right after the function definition:

```js
const employee = (function (name) {
  const email = `${name}@company.com`;

  let salary = 100_000;

  const getName = () => name;
  const getSalary = () => salary;
  const giveMeritIncrease = () => (salary *= 1.05);

  return { name, email, getName, getSalary, giveMeritIncrease };
})("jack");

employee.giveMeritIncrease();
employee.giveMeritIncrease();

const currentSalary = employee.getSalary();
console.log(employee.getName()); // => jack
console.log(currentSalary); // => 110250
```

To pass parameters to an IIFE, include it in the parentheses at the end. The previous example passes `jack` as an argument as the `name` parameter.

## Scope

Scope answers the question, _where are my variables and functions available to me?_

### Global scope

Anything that is in the global scope is attached to the window object, except for variables that are declared with `let` and `const`. These variables are global, but not attached to the `window` object.

Also, you can't reassign `const`.

`let` and `var` are block-scoped.
`var` is function-scoped.

## Closures

A closure is _the ability to access a parent level scope from a child scope, even after the parent function has been terminated_.

The following closures are equivalent:

```js
// Example 1
function outer() {
  const outerVar = "Hey I am the outer Var";

  return function inner() {
    const innerVar = "Hey I am an inner var";

    console.log(outerVar);
    console.log(innerVar);
  };
}

// Example 2
function outer() {
  const outerVar = "Hey I am the outer Var";

  function inner() {
    const innerVar = "Hey I am an inner var";

    console.log(outerVar);
    console.log(innerVar);
  }

  return inner;
}
```

Look at `Example 2`. Here is how you can invoke both functions:

```js
function outer() {
  const outerVar = "Hey I am the outer Var";

  function inner() {
    const innerVar = "Hey I am an inner var";

    console.log(outerVar);
    console.log(innerVar);
  }

  return inner;
}

// invoke outer and save it in innerFn
const innerFn = outer();

// invoke innerFn()
innerFn();
```

In the previous example, you stored the `outer()` function in a variable and executed it, and then called that function at a later time with `innerFn()`.

Normally, anytime that a function executes, the variables that are scoped within that function are no longer accessible. With closures, variables in the outer function are available even after the function executes.


## Classes

A class is a complex function definition. To illustrate, consider a class named `User`:

```js
class User {
  constructor(name) {
    this.name = name;
  }
  sayHi() {
    alert(this.name);
  }
}

// class is a function
console.log(typeof User); // function

// ...or, more precisely, the constructor method
console.log(User === User.prototype.constructor); // true
```

This class creates a function named `User` and stores its class methods in the `User.prototype`.

> Class constructors always use strict.

### Syntax

The following example shows the basic class syntax:

```js
class ClassName {
  // class methods
  constructor() { ... }
  method1() { ... }
  method2() { ... }
  method3() { ... }
  ...
}
```

You create an object with the `new` syntax:

```js
const myObject = new ClassName();
```

### Constructors

You can derive a class from another class with the `extends` keyword. If your derived class does not provide custom initialization, you do not have to call `super()` in the constructor. If it does use customization, then you have to call it:

```js
//*********************************
// Base class
//*********************************
class Person {
  constructor(name) {
    this.name = name;
  }

  sayName() {
    console.log(`Hi, my name is ${this.name}.`);
  }
}

const a = new Person("jack");
a.sayName(); // => Hi, my name is jack.

//*********************************
// Straight inheritence
//*********************************
class NotCustomPerson extends Person {
  // nothing here
}

const b = new NotCustomPerson("rick");
b.sayName(); // => Hi, my name is rick.

//*********************************
// Inheritence with customizations
//*********************************
class CustomPerson extends Person {
  constructor(name, age) {
    super(name);
    this.age = age;
  }

  sayAge() {
    console.log(`I am ${this.age} years old.`);
  }
}

const c = new CustomPerson("steve", 40);
c.sayName(); // => Hi, my name is steve.
c.sayAge(); // => I am 40 years old.
```

You can call the getters and setters on the parent class, but you cannot use `this` when you call them. This is because `this` is initialized on the getters and setters before the constructor is called, but `this` is not fully initialized on the object (including the object constructor).

#### Class fields

Class fields are properties that are set on individual objects, not the prototype. They are enumerable, and they can be inherited in the prototype chain. Because of this, they cannot be named `prototype` or `constructor`. If you do not initialize the field with a value, it is `undefined`.

Class fields might require a polyfill, because they are new to Javascript:

```js
class User {
  name = "John";  // class field

  sayHi() {...}
}
```

`this` refers to the class instance that you created, while `super` refers to the `prototype` property of the base class. The `prototype` property contains the base class instance methods, but not its fields. This is because the instance fields are added before the constructor runs, but they are added _after_ `super` returns.

Fields are added one-by-one, in order from top to bottom. So, the bottom field can reference the top field, but not vice versa.

Derived classes do not invoke setters from the base class.

#### Class expression

You can also use a [class expression](https://javascript.info/class#class-expression) to define a class, but I do not see how this could be more useful than the standard syntax.

### Methods

Class methods are non-enumerable. This means that you cannot cycle through the methods with the `for...in` loop or `Ojbect.keys()` function.


### Getters and setters

There are _data_ properties and _accessor_ properties. Accessor properties look like normal functions, but they exist only to get or set a value.

Add getters and setters with the `get` and `set` keywords on an object, or use the `Object.defineProperty()` method.

#### `get` and `set`

The following snippet includes field validation in the set accessor. Also, both accessors begin the variable with an underscore (`_name`), which is a Javascript convention that specifies a variable as an internal variable that should not be accessed from outside the object:

```js
let user = {
  get name() {
    return this._name;
  },

  set name(value) {
    if (value.length < 4) {
      alert("Name is too short, need at least 4 characters");
      return;
    }
    this._name = value;
  },
};
```

#### `Object.defineProperty`

You can use `Object.defineProperty` to get a value on the object's prototype. You cannot use it on an object.

```js
let user = {
  name: "John",
  surname: "Smith",
};

Object.defineProperty(user, "fullName", {
  get() {
    return `${this.name} ${this.surname}`;
  },

  set(value) {
    [this.name, this.surname] = value.split(" ");
  },
});
```


### Private properties and methods

Private properties cannot be referenced outside of the class. To create a private property, prefix the field name with a hash (`#`). For example, `#privateField` or `#privateMethod(){...}`. Constructors cannot be private. The old convention is to prefix the field or method with an underscore.

Private properties are not part of prototypical inheritance.

### static methods and properties

[MDN docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/static)

Static methods are on the class itself--you cannot access them from an object of the class. Common use cases include the following:

- methods: create or clone objects
- properties: caches, fixed-configuration, other data that does not need to be replicated across instances.

Think about how some string methods are accessed on the instance of a string itself--`someString.slice(0, 5)`--whereas some methods are called on the String constructor--`String.fromCharCode(79, 100, 105, 110)`.