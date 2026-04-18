---
title: "Best practices"
weight: 140
description: >
  Practical guidelines for writing clear, safe, and maintainable JavaScript.
---

These practices summarize patterns that consistently produce readable, predictable code. They are not rules to memorize — they are trade-offs to understand, so you know when to apply them and when a specific situation calls for something different.

## Declarations

### Prefer `const`; use `let` when you need to reassign

`const` signals intent: this binding will not change. It prevents accidental reassignment and makes code easier to reason about. Reach for `let` only when you actually need to reassign the variable.

```js
// const works fine for objects and arrays — you can still mutate their contents
const user = { name: 'Ada', role: 'admin' };
user.role = 'viewer';   // fine — the binding is constant, not the object

// let for counters, accumulators, loop variables that change
let total = 0;
for (const item of cart) {
    total += item.price * item.qty;
}
```

Avoid `var`. It has function scope instead of block scope, which creates subtle bugs in loops and conditionals, and it hoists declarations to the top of the function in ways that obscure what the code does.

### Declare variables close to where you use them

A variable declared at the top of a function but only used 30 lines later makes readers track it across unrelated code. Declare it just before its first use.

```js
// harder to follow — reader must remember `filtered` across unrelated code
const filtered = items.filter(i => i.active);
doSomethingUnrelated();
doAnotherThing();
render(filtered);

// easier to follow — declaration and use are adjacent
doSomethingUnrelated();
doAnotherThing();
const filtered = items.filter(i => i.active);
render(filtered);
```

## Equality

### Always use `===`

The `==` operator performs type coercion before comparing, which produces counterintuitive results:

```js
0   == false    // true
''  == false    // true
[]  == false    // true
null == undefined // true

// None of these are true with ===
0   === false   // false
''  === false   // false
```

Use `===` everywhere. The only exception is `x == null`, which checks for both `null` and `undefined` in one comparison — a deliberate, well-known idiom:

```js
// Checks for both null and undefined
if (value == null) {
    return defaultValue;
}
```

## Functions

### Keep functions short and focused on one thing

A function that fetches data, transforms it, and updates the DOM is three functions waiting to be extracted. Each function should do one thing — and its name should describe exactly what that one thing is.

```js
// too much in one function
async function loadAndRender(userId) {
    const res = await fetch(`/api/users/${userId}`);
    const data = await res.json();
    const formatted = `${data.first} ${data.last}`;
    document.querySelector('#name').textContent = formatted;
    document.querySelector('#email').textContent = data.email;
}

// three focused functions
async function fetchUser(userId) {
    const res = await fetch(`/api/users/${userId}`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return res.json();
}

function formatUser(user) {
    return { displayName: `${user.first} ${user.last}`, email: user.email };
}

function renderUser({ displayName, email }) {
    document.querySelector('#name').textContent = displayName;
    document.querySelector('#email').textContent = email;
}

// composition is now obvious
const user = await fetchUser(userId);
renderUser(formatUser(user));
```

### Name functions after what they return, not how they work

A function named `getActiveUsers` communicates what you get back. A function named `filterArrayByActiveProperty` describes implementation detail that may change.

```js
// describes implementation — brittle name
const filterArrayByActiveProperty = users => users.filter(u => u.active);

// describes intent — survives refactoring
const getActiveUsers = users => users.filter(u => u.active);
```

## Objects and arrays

### Don't mutate objects or arrays you didn't create

Mutating an object passed in as a parameter can cause hard-to-trace bugs when the caller still holds a reference to the original. Return a new value instead:

```js
// mutates the original — callers may not expect this
function applyDiscount(cart, pct) {
    cart.items = cart.items.map(i => ({ ...i, price: i.price * (1 - pct) }));
    return cart;  // same reference, now modified
}

// returns a new cart — original is unchanged
function applyDiscount(cart, pct) {
    return {
        ...cart,
        items: cart.items.map(i => ({ ...i, price: i.price * (1 - pct) })),
    };
}
```

### Use spread and destructuring for clarity

Spread and destructuring make the shape of your data explicit at a glance:

```js
// update one field — create a new object, don't mutate
const updatedUser = { ...user, role: 'admin' };

// pull out only what you need
const { name, email } = user;

// rename during destructuring
const { first: firstName, last: lastName } = user;

// default values in destructuring
function greet({ name = 'stranger', language = 'en' } = {}) {
    return language === 'en' ? `Hello, ${name}` : `Hola, ${name}`;
}
```

## Async code

### Always handle Promise rejections

An unhandled rejection in Node.js crashes the process. In the browser, it logs a warning that's easy to miss in production. Every Promise chain needs either a `.catch()` or a `try/catch`.

```js
// unhandled — if this rejects, you'll never know
fetchDashboardData().then(render);

// handled
fetchDashboardData()
    .then(render)
    .catch(err => {
        console.error('Dashboard failed to load:', err);
        showErrorState();
    });
```

### Run independent async operations in parallel

`await` in sequence means each call waits for the previous one to finish. When operations are independent, use `Promise.all` to run them at the same time:

```js
// sequential — ~600ms total
const user    = await fetchUser(id);
const posts   = await fetchPosts(id);
const friends = await fetchFriends(id);

// parallel — ~200ms (the slowest one)
const [user, posts, friends] = await Promise.all([
    fetchUser(id),
    fetchPosts(id),
    fetchFriends(id),
]);
```

Use sequential `await` only when a later call depends on the result of an earlier one.

### Check `response.ok` when using `fetch`

`fetch` only rejects on network failure. A 404 or 500 response resolves — with `response.ok` set to `false`. Always check it:

```js
const res = await fetch('/api/data');
if (!res.ok) throw new Error(`HTTP ${res.status}`);
const data = await res.json();
```

## Common pitfalls

### `this` in callbacks

Regular functions have their own `this`, which breaks when you pass a method as a callback:

```js
class Timer {
    constructor() {
        this.seconds = 0;
    }

    start() {
        // BUG — `this` inside the callback is not the Timer instance
        setInterval(function () {
            this.seconds++;     // this is undefined (strict mode) or window
        }, 1000);

        // CORRECT option 1 — arrow function inherits `this` from start()
        setInterval(() => {
            this.seconds++;
        }, 1000);

        // CORRECT option 2 — bind
        setInterval(function () {
            this.seconds++;
        }.bind(this), 1000);
    }
}
```

Arrow functions do not have their own `this` — they capture it from the enclosing scope. This makes them the right choice for callbacks that need to reference the outer object.

### `NaN` comparisons

`NaN` is the only value in JavaScript that is not equal to itself:

```js
NaN === NaN     // false
NaN == NaN      // false

// Use Number.isNaN() — not the global isNaN(), which coerces first
Number.isNaN(NaN)           // true
Number.isNaN('hello')       // false (it's a string, not NaN)
isNaN('hello')              // true — coerces 'hello' to NaN first, unreliable
```

### Floating point arithmetic

JavaScript uses IEEE 754 floating point, so decimal arithmetic can produce unexpected results:

```js
0.1 + 0.2          // 0.30000000000000004
0.1 + 0.2 === 0.3  // false
```

For currency and anything precision-sensitive, work in integers (cents, not dollars) or use `toFixed()` for display:

```js
// work in cents
const price = 199;   // $1.99 in cents
const tax   = Math.round(price * 0.08);
const total = price + tax;

// display
console.log(`$${(total / 100).toFixed(2)}`);   // $2.15
```

### `delete` on arrays

`delete` removes an element but leaves a hole — the array length stays the same and the slot becomes `undefined`:

```js
const arr = [1, 2, 3];
delete arr[1];
console.log(arr);       // [ 1, <1 empty item>, 3 ]
console.log(arr.length) // 3

// Use splice to remove and reindex
arr.splice(1, 1);
console.log(arr);       // [ 1, 3 ]
```

## Security

### Never insert user input directly into the DOM with `innerHTML`

`innerHTML` executes any `<script>` tags or event handler attributes in the string, creating an XSS vulnerability:

```js
// UNSAFE — if userInput contains <img src=x onerror=stealCookies()>, it runs
container.innerHTML = userInput;

// SAFE — textContent treats everything as text, never as HTML
container.textContent = userInput;

// SAFE — build elements programmatically
const p = document.createElement('p');
p.textContent = userInput;
container.appendChild(p);

// If you genuinely need to render HTML from a user, sanitize it first
// with a library like DOMPurify before passing to innerHTML
container.innerHTML = DOMPurify.sanitize(userInput);
```

### Never use `eval()`

`eval()` executes an arbitrary string as JavaScript code. It creates XSS vulnerabilities, defeats optimization in the JS engine, and is almost never necessary. Use `JSON.parse()` for data, computed property names for dynamic keys, and `Function` only in controlled build tooling — never at runtime with user input.

```js
// UNSAFE
const key = userInput;
eval(`config.${key} = value`);

// SAFE — computed property name
config[key] = value;
```

## Performance

### Debounce event handlers that fire rapidly

`input`, `scroll`, and `resize` can fire dozens of times per second. Wrapping the handler in a debounce prevents unnecessary work:

```js
function debounce(fn, ms) {
    let timer;
    return (...args) => {
        clearTimeout(timer);
        timer = setTimeout(() => fn(...args), ms);
    };
}

const search = debounce(async (query) => {
    const results = await fetchResults(query);
    renderResults(results);
}, 300);

document.querySelector('#search').addEventListener('input', e => search(e.target.value));
```

### Use `{ passive: true }` on scroll and touch listeners

Marking a listener passive tells the browser it will never call `preventDefault()`, so the browser can start scrolling immediately without waiting for your handler to finish:

```js
window.addEventListener('scroll', updateProgressBar, { passive: true });
document.addEventListener('touchmove', trackSwipe, { passive: true });
```

### Prefer `textContent` over `innerHTML` when you don't need HTML

`textContent` skips HTML parsing and is faster. It's also safer — a default choice until you actually need markup.
