---
title: "Objects and Classes"
weight: 40
description: >
  Objects and Classes.
---

## Objects


Objects have properties that are assigned values.

```js
// Update object properties with dot notation
> person
{ name: 'Steve', age: 40 }
> person.age = 41

> person
{ name: 'Steve', age: 41 }

// Accessing arrays in objects
let company.activities[1]

// Accessing objects in arrays
let streetName = addresses[0].street

// Accessing objects in arrays in objects
  company = { 
            companyName: "Healthy Candy",
            activities: [ 
                "food manufacturing", 
                "improving kids' health", 
                "manufacturing toys"],
            address: [{
                street: "2nd street",
                number: "123",
                zipcode: "33116",
                city: "Miami",
                state: "Florida"
            },
            {
                street: "1st West avenue",
                number: "5",
                zipcode: "75001",
                city: "Addison",
                state: "Texas"
            }],
            yearOfEstablishment: 2021 
            };

let streetName = company.address[1].street // 1st West avenue

```

### Convert object to array 

```js

let car = {
  model: "Golf",
  make: "Volkswagen",
  year: 1999,
  color: "black",
};

// convert object keys to array
> let keys = Object.keys(car)
> keys
[ 'model', 'make', 'year', 'color' ]

// convert object values to array
> let values = Object.values(car)
> values
[ 'Golf', 'Volkswagen', 1999, 'black' ]

// convert object keys/values to array
> let keyVals = Object.entries(car)
> keyVals
[
  [ 'model', 'Golf' ],
  [ 'make', 'Volkswagen' ],
  [ 'year', 1999 ],
  [ 'color', 'black' ]
]

for (let [key, val] of Object.entries(car)) {
    console.log(key, ":", val)
}
// Output:

// model : Golf
// make : Volkswagen
// year : 1999
// color : black
```

## Classes

Classes are blueprints for objects. It includes properties (attributes) and methods that model behavior. To make a property private, place a `#` before it. Here is an example class:

```js 
class Person {
  #firstname;
  #lastname;
  constructor(firstname, lastname) {
    this.#firstname = firstname;
    this.#lastname = lastname;
  }
  get firstname() {
    return this.#firstname;
  }
  set firstname(firstname) { 
      if(firstname.startsWith("M")){
        this.#firstname = firstname; 
      } else {
        this.#firstname = "M" + firstname; 
      }
  }
  get lastname() {
    return this.#lastname;
  }
  set lastname(lastname) {
    this.#lastname = lastname;
  }
}
```

### Inheritance 

When you inherit from a class to create a subclass, you use the `extends` keywor, and the `super` function in the constructor. The `super` function calls the parent class's constructor and methods. You do not need to explicitly set parent class attributes in a subclass's constructor, or define methods from the parent class that you do not want to override:

```js 
class Motorcycle extends Vehicle {
  constructor(color, currentSpeed, maxSpeed, fuel) {
    super(color, currentSpeed, maxSpeed);
    this.fuel = fuel;
  }
  doWheelie() {
    console.log("Driving on one wheel!");
   }
}
```

### Prototypes TODO

### Built-in methods 

You can chain any method that returns a result:
```js
let s = "Hello";
console.log(
  s.concat(" there!")
  .toUpperCase()
  .replace("THERE", "you")
  .concat(" You're amazing!")
);
```