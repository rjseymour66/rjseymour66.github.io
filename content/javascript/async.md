---
title: "Asynchronous JS"
linkTitle: "Async"
weight: 110
description:
---

Asynchronous programs must wait for data to arrive or for some event to occur before they can continue. In the browser, JavaScript is event-driven: it waits for the user to act and then runs code in response. Callbacks originally handled these deferred operations.

Three language features now give you structured ways to work with async operations:

- **Promises**: Objects that represent the result of an async operation that has not yet completed
- **`async`/`await`**: Keywords that let you structure Promise-based code as if it were synchronous
- **Async iterators and the `for/await` loop**: Work with streams of async events in loops

## The core mental model

JavaScript runs on a single thread. It cannot literally pause and wait. Doing so would freeze the browser. Instead, async operations are *registered* and a callback (or Promise) runs when the result is ready. The rest of your code continues running in the meantime.

Think of it like ordering at a coffee shop:

- **Synchronous**: you stand at the counter and block the queue until your drink is ready.
- **Asynchronous (callback)**: you give them your name. They call it when it is ready.
- **Promise**: you get a numbered ticket stub. You can do other things. The ticket resolves when the drink is ready.
- **`async`/`await`**: same as a Promise, but reads like you are standing at the counter. The syntax hides the waiting.

## Callbacks

A callback is a function passed into another function, to be executed at a later time when an operation completes. You do not know *when* the async function will run, but you know *where*: inside the function you passed it to.

Callbacks work fine for a single async step, but chaining multiple steps produces deeply nested code known as *callback hell*:

```js
// Three sequential steps: deeply nested, error-prone to read
login(username, password, (err, user) => {
    if (err) return handleError(err);

    getProfile(user.id, (err, profile) => {
        if (err) return handleError(err);

        getRecommendations(profile.interests, (err, recs) => {
            if (err) return handleError(err);
            renderDashboard(user, profile, recs);
        });
    });
});
```

Each level adds indentation, and error handling requires a separate check at every step. Promises eliminate this by making each step return a value you can chain from.

## Promises

A Promise is an object that represents a value that is not yet available, but will be at some point in the future (or will fail trying). A Promise is always in one of three states:

| State | Meaning |
|:---|:---|
| **Pending** | The async operation has not finished yet |
| **Fulfilled** | The operation completed successfully. A value is available. |
| **Rejected** | The operation failed. A reason (error) is available. |

Once a Promise settles (fulfills or rejects), it never changes state.

You create a Promise with the `new Promise()` constructor, which takes a function with two parameters:

- `resolve(value)`: call this when the operation succeeds
- `reject(reason)`: call this when the operation fails

```js
const fetchScore = new Promise((resolve, reject) => {
    // simulate an async operation with setTimeout
    setTimeout(() => {
        const score = 87;
        if (score >= 0) {
            resolve(score);     // fulfills the Promise with the score
        } else {
            reject(new Error('Invalid score'));  // rejects with an error
        }
    }, 1000);
});

fetchScore
    .then(score => console.log(`Score: ${score}`))   // runs on fulfillment
    .catch(err  => console.error(err.message));       // runs on rejection
```

### Chaining .then()

Each `.then()` returns a *new* Promise. The value you return from one `.then()` callback becomes the input to the next. This lets you write sequential async steps as a flat chain instead of nested callbacks:

```js
// Three sequential steps: flat, readable, one error handler covers all of them
login(username, password)
    .then(user    => getProfile(user.id))
    .then(profile => getRecommendations(profile.interests))
    .then(recs    => renderDashboard(recs))
    .catch(err    => handleError(err));   // catches any rejection in the chain
```

The code performs the same three steps as the callback example above. The flat chain is dramatically easier to follow.

### Error handling

`.catch(fn)` is shorthand for `.then(null, fn)`. A single `.catch()` at the end of a chain handles any rejection that occurs at any step above it. Use `.finally()` for cleanup that must run regardless:

```js
fetch('/api/user')
    .then(res => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
    })
    .then(user => renderProfile(user))
    .catch(err => {
        console.error('Failed to load profile:', err.message);
        showErrorBanner();
    })
    .finally(() => hideLoadingSpinner());   // always runs
```

### Promise combinators

Run multiple Promises concurrently instead of sequentially when the operations are independent.

#### Promise.all: all must succeed

Resolves when *every* Promise resolves. Rejects immediately if *any* Promise rejects:

```js
// Fetch user, posts, and notifications in parallel: 3× faster than sequential
const [user, posts, notifications] = await Promise.all([
    fetch('/api/user').then(r => r.json()),
    fetch('/api/posts').then(r => r.json()),
    fetch('/api/notifications').then(r => r.json()),
]);
```

#### Promise.allSettled: collect all results regardless of failure

Resolves when every Promise settles. Never rejects. Each result has a `status` of `'fulfilled'` or `'rejected'`:

```js
const results = await Promise.allSettled([
    fetch('/api/analytics'),    // might fail: analytics is non-critical
    fetch('/api/user'),
    fetch('/api/config'),
]);

results.forEach(result => {
    if (result.status === 'fulfilled') {
        process(result.value);
    } else {
        console.warn('Non-critical failure:', result.reason.message);
    }
});
```

#### Promise.race: first one wins

Resolves or rejects with the result of the first Promise that settles. Useful for timeouts:

```js
function withTimeout(promise, ms) {
    const timeout = new Promise((_, reject) =>
        setTimeout(() => reject(new Error(`Timed out after ${ms}ms`)), ms)
    );
    return Promise.race([promise, timeout]);
}

// Request must complete within 5 seconds
const data = await withTimeout(fetch('/api/slow-endpoint').then(r => r.json()), 5000);
```

## async and await

`async`/`await` is syntactic sugar over Promises. It does not introduce new async behavior. It makes Promise-based code *read* like synchronous code. Under the hood, every `await` is a `.then()`.

The `async`/`await` syntax follows three rules:

- `await` can only be used inside a function declared with `async`.
- `await` pauses execution of the *current async function* only. The rest of your program continues running.
- An `async` function always returns a Promise, even if you return a plain value.

```js
// .then() chain and async/await are equivalent: same behavior, different syntax

// Promise chain
fetch(url)
    .then(resp => resp.json())
    .then(json => console.log(json.body));

// async/await
async function fetchData(url) {
    try {
        const resp = await fetch(url);
        const json = await resp.json();
        console.log(json.body);
    } catch (err) {
        console.error('Fetch failed:', err);
    }
}
```

### Error handling with try/catch

Use `try/catch` to handle rejected Promises in `async` functions. This replaces `.catch()` in the chain:

```js
async function loadUserDashboard(userId) {
    try {
        const res = await fetch(`/api/users/${userId}`);
        if (!res.ok) throw new Error(`HTTP ${res.status}`);

        const user = await res.json();
        renderDashboard(user);
    } catch (err) {
        console.error('Could not load dashboard:', err.message);
        showErrorState();
    } finally {
        hideSpinner();  // always runs
    }
}
```

### Sequential vs parallel execution

Awaiting independent operations in sequence is the most common `async/await` mistake. `await` inside a loop or in sequence causes each request to wait for the previous one:

```js
// SLOW: requests run one after another (sequential)
async function loadSlow() {
    const user    = await fetchUser();     // waits ~200ms
    const posts   = await fetchPosts();   // then waits ~150ms
    const friends = await fetchFriends(); // then waits ~100ms
    // Total: ~450ms
}

// FAST: requests run at the same time (parallel)
async function loadFast() {
    const [user, posts, friends] = await Promise.all([
        fetchUser(),
        fetchPosts(),
        fetchFriends(),
    ]);
    // Total: ~200ms (the slowest one)
}
```

Use `Promise.all` when the operations are independent. Use sequential `await` only when each step depends on the result of the previous one.

### Avoiding common mistakes

#### Forgetting `await`

```js
// BUG: result is a Promise object, not the parsed JSON
const data = fetch('/api/data').then(r => r.json());
console.log(data.name);  // undefined: data is a Promise, not an object

// CORRECT
const data = await fetch('/api/data').then(r => r.json());
console.log(data.name);  // works
```

#### `await` in a `forEach` loop: does not work

```js
// BUG: forEach does not wait for async callbacks
const ids = [1, 2, 3];
ids.forEach(async (id) => {
    const user = await fetchUser(id);  // runs concurrently, order not guaranteed
    console.log(user);
});

// CORRECT option 1: sequential (one at a time)
for (const id of ids) {
    const user = await fetchUser(id);
    console.log(user);
}

// CORRECT option 2: parallel (all at once)
const users = await Promise.all(ids.map(id => fetchUser(id)));
```

#### Unhandled rejections

Every `async` function that can fail needs either a `try/catch` or a `.catch()` at the call site. Unhandled rejections log errors and, in Node.js, crash the process:

```js
// BUG: if loadUser rejects, the error is silently swallowed
async function init() {
    loadUser();   // forgot await AND no .catch()
}

// CORRECT
async function init() {
    try {
        await loadUser();
    } catch (err) {
        console.error('Init failed:', err);
    }
}
```
