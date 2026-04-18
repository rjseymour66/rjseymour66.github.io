---
title: "Events"
linkTitle: "Events"
weight: 50
description:
---

JavaScript is event-driven: the browser generates an event whenever something changes in
the document or the browser itself. A page finishing its load, a user clicking a button,
or a cursor moving across the screen all produce events. Any HTML element can be an event
target, and you register functions — called event handlers — to run when a specific event
occurs on a specific element.

## Further reading

- [Eloquent JavaScript: "Handling Events"](https://eloquentjavascript.net/15_event.html) —
  a deep dive into the event model with interactive examples.
- [MDN: Introduction to Events](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events) —
  the reference guide for event types, properties, and browser compatibility.

## Event model

The JavaScript event model has five core concepts.

The **event type** is a string that names the kind of event — for example, `mousemove`,
`keydown`, or `load`. You'll also see it called the event name.

The **event target** is the object on which the event occurred. Targets are usually a
`Window`, `Document`, or `Element`, but a `Worker` object can also be a target.

The **event handler** (also called an event listener) is the function that responds to
an event. You register a handler by telling the browser which event type to watch for
and which target to watch. When that event fires on that target, the browser calls your
function. You'll also hear this described as the handler being fired, triggered, or
dispatched.

The **event object** is a JavaScript object the browser passes to your handler as its
argument. Every event object has at least two properties:

- `type`: the event type string
- `target`: the object the event occurred on

Some events carry additional properties. Mouse events, for example, include `clientX`
and `clientY` for the cursor's window coordinates.

**Event propagation** is how the browser decides which handlers to call beyond the
immediate target. Most DOM events bubble up the document tree — a click on a `<button>`
inside a `<div>` also triggers any click handlers on that `<div>`, its parent, and so on
up to `Window`. A handler can stop this process using a method on the event object.
`load` and `Worker` events don't propagate. Event capturing runs the same process in
reverse: a handler on a container element intercepts the event before it reaches its
target.

## Event categories

Browser events fall into five general categories.

- **Device-dependent input events** are tied to a specific input device such as a mouse
  or keyboard. Examples include `mousedown` and `keydown`.

- **Device-independent input events** are not tied to a specific device, which makes
  them useful for touch screens and stylus input. For example, `pointerdown` works as a
  device-agnostic alternative to `mousedown`, and `input` works as an alternative to
  `keydown`.

- **UI events** are high-level events on form elements, such as `focus`, `change`, and
  `submit`.

- **State-change events** are triggered by browser or network activity, not user
  interaction. They signal lifecycle changes — for example, `load` and `DOMContentLoaded`
  fire when a page finishes loading, and `online`/`offline` fire when the network
  connection changes.

- **API-specific events** are defined by individual Web APIs such as the Audio or Video
  APIs. These exist because many Web APIs were designed before Promises, so they relied
  on events to signal when asynchronous operations completed.

## Registering event handlers

You have two ways to register an event handler: assign a function directly to a property
on the event target, or pass your handler to the target's `addEventListener()` method.
Prefer `addEventListener()` in almost all cases.

### Assign a property on the event target

Each event type has a corresponding property named `on` + the event name — for example,
`onclick`, `onkeydown`, or `onload`. Assigning a function to that property registers it
as the handler:

```js
window.onload = function () {
    document.querySelector('#app').classList.remove('loading');
};
```

The limitation is that you can only assign one handler per event type this way. A second
assignment overwrites the first. Use `addEventListener()` instead when you need multiple
handlers or cleaner separation of concerns.

#### onload

The `onload` event fires after an element finishes loading. On `window`, it fires after
the entire page — including images and stylesheets — has loaded. On `document`, its
behavior varies by browser, so avoid `document.onload`.

Instead, use `DOMContentLoaded` with `addEventListener`. It fires as soon as the HTML is
parsed and the DOM is ready, without waiting for images or other resources:

```js
document.addEventListener('DOMContentLoaded', () => {
    document.querySelector('#search-input').focus();
});
```

This is the standard pattern for any setup code that needs the DOM but doesn't depend on
images or external resources being fully loaded.

### Set event handler attributes

You can also register handlers as HTML attributes directly on elements:

```html
<button onclick="handleSubmit()">Submit</button>
```

Avoid this approach. The browser wraps your attribute string in a function with an unusual
scope chain — the element itself and any ancestor `<form>` are in scope, which can shadow
variable names in surprising ways. It also mixes behavior into your markup, making the
code harder to maintain. Use `addEventListener()` instead.

### addEventListener()

Any object that can be an event target has an `addEventListener()` method. Call it with
the event type string, the handler function, and an optional third argument that controls
capturing behavior:

```js
const button = document.querySelector('.btn');
button.addEventListener('click', () => console.log(button.textContent));
button.addEventListener('click', () => console.log('second listener'));
```

You can register multiple handlers on the same target for the same event type — they fire
in registration order. Registering the same handler function with identical arguments more
than once has no effect; it fires only once.

Remove a handler with `removeEventListener()`, passing the same arguments you used to
register it. Because anonymous functions can't be referenced again, you need a named
function to remove a listener:

```js
let clickCount = 0;

const logOnce = () => {
    console.log('Listener fired');
    clickCount++;
    if (clickCount === 1) {
        button.removeEventListener('click', logOnce);
    }
};

const button = document.querySelector('.btn');
button.addEventListener('click', () => console.log(button.textContent));
button.addEventListener('click', logOnce);
```

The third argument to `addEventListener()` is either a Boolean or an options object:

```js
document.addEventListener('click', handler, {
    capture: true,
    once: true,
    passive: true,
});
```

Pass `true` to register the handler as a capturing handler (equivalent to
`{ capture: true }`). The options object supports three properties:

- `capture`: registers the handler in the capturing phase rather than the bubbling phase.
- `once`: removes the handler automatically after it fires once.
- `passive`: promises the browser your handler won't call `preventDefault()`, which lets
  it optimize scrolling and touch behavior. Firefox and Chrome make `touchmove` and
  `mousewheel` passive by default for this reason.

**`once` — run a handler exactly one time**

Use `once: true` for setup steps that should only happen once, like showing a welcome
tooltip or tracking a first interaction:

```js
// Show an onboarding tooltip only on the user's first click
document.addEventListener('click', () => {
    showOnboardingTooltip();
}, { once: true });

// Log when a video is first played, not on every resume
videoEl.addEventListener('play', () => {
    analytics.track('video_first_play', { id: videoEl.dataset.id });
}, { once: true });
```

**`passive: true` — keep scrolling smooth**

Scroll and touch handlers can block the browser's rendering pipeline unless you explicitly
promise not to call `preventDefault()`. Mark them passive to keep scrolling responsive:

```js
window.addEventListener('scroll', () => {
    updateProgressBar();
}, { passive: true });
```

### Register a handler on multiple elements

Use `querySelectorAll()` to get a `NodeList`, then call `forEach()` to attach a handler
to each element:

```html
<div id="toolbar">
    <button data-command="bold">Bold</button>
    <button data-command="italic">Italic</button>
    <button data-command="underline">Underline</button>
</div>
```

```js
const buttons = document.querySelectorAll('#toolbar button');

buttons.forEach((button) => {
    button.addEventListener('click', () => {
        document.execCommand(button.dataset.command);
    });
});
```

This works well for static sets of elements. If elements are added or removed at runtime,
use event delegation on the parent instead — covered in the next section.

### Mouse events reference

| Event        | Fires when                                                          |
| ------------ | ------------------------------------------------------------------- |
| `dblclick`   | The user double-clicks an element.                                  |
| `mousedown`  | A mouse button is pressed on an element, before it's released.      |
| `mouseup`    | A pressed mouse button is released over an element.                 |
| `mouseenter` | The cursor moves onto an element. Does not bubble.                  |
| `mouseleave` | The cursor leaves an element and all its children. Does not bubble. |
| `mousemove`  | The cursor moves while over an element. Fires continuously.         |
| `mouseout`   | The cursor leaves an element or any of its children. Bubbles.       |
| `mouseover`  | The cursor enters an element or any of its children. Bubbles.       |

Use these strings with `addEventListener()` — for example,
`button.addEventListener('dblclick', handler)`.

## Event handler invocation

The browser ignores any value an event handler returns. To cancel the browser's default
behavior for an event — such as preventing a form from submitting or a link from
navigating — call `preventDefault()` on the event object inside the handler. Don't rely
on `return false`; that pattern works in jQuery but not with native `addEventListener()`.

Handlers on the same target fire in the order they were registered.

### Event object

When the browser calls your handler, it passes an `Event` object as the first argument:

```js
button.addEventListener('click', (e) => {
    console.log(e.target, e.type, e.timeStamp);
});
```

Every event object exposes these core properties:

- `type`: The event type string, such as `'click'` or `'keydown'`.
- `target`: The element where the event originated.
- `currentTarget`: The element the handler is attached to. During bubbling, `target` and
  `currentTarget` differ — `target` stays fixed on the origin element while
  `currentTarget` changes as the event travels up the DOM.
- `timeStamp`: Milliseconds elapsed since the page loaded. Subtract two timestamps to
  calculate the time between events.
- `isTrusted`: `true` if the browser generated the event; `false` if your code dispatched
  it programmatically.
- `bubbles`: Whether the event bubbles up the DOM.
- `cancelable`: Whether you can suppress the browser's default behavior with
  `preventDefault()`.
- `defaultPrevented`: Whether `preventDefault()` has already been called on this event.

Some event types carry additional properties. Mouse events include `clientX` and `clientY`
for the cursor's coordinates relative to the viewport, and `KeyboardEvent` includes `key`
for the pressed key's value.

### Event handler context

Inside a regular function handler, `this` refers to the element the handler is attached
to — the same as `e.currentTarget`. Inside an arrow function, `this` inherits from the
surrounding lexical scope:

```js
button.addEventListener('click', function () {
    console.log(this);          // the <button> element
});

button.addEventListener('click', () => {
    console.log(this);          // the enclosing scope's this (often Window)
});
```

This matters most in class-based components. If you define a handler as an arrow function
class field, `this` reliably refers to the class instance — no `.bind()` needed:

```js
class Modal {
    constructor() {
        document.addEventListener('keydown', this.handleKey);
    }

    handleKey = (e) => {
        if (e.key === 'Escape') this.close();
    };
}
```

### Event propagation

When an event fires on a DOM element, it travels through three phases.

In the **capturing phase**, the event descends from `Window` down through the DOM to the
event target. Handlers registered with `{ capture: true }` fire during this phase.
Capturing is useful for intercepting events before they reach their target — for example,
tracking mouse drags across a container regardless of which child element the drag started
on.

In the **target phase**, the handlers attached directly to the element that triggered the
event fire.

In the **bubbling phase**, the event travels back up the DOM — from the target's parent,
to its grandparent, and so on up to `Window`. Any handler for the same event type on
those ancestor elements fires in turn. This is what makes event delegation possible: you
can register one handler on a parent and respond to events from any descendant.

Not all events bubble. `focus`, `blur`, and `scroll` don't propagate. The `load` event
bubbles on document elements but stops at `Document` — it doesn't reach `Window`.

### Event cancellation

The browser performs a default action for many events: following a link on `click`,
submitting a form on `submit`, scrolling on `wheel`. Call `preventDefault()` on the event
object to suppress that default action while still allowing your handler — and any other
handlers — to run.

Call `stopPropagation()` to prevent the event from bubbling further up the DOM. Other
handlers attached to the same element still fire; only ancestor handlers are skipped.

Call `stopImmediatePropagation()` to do both: stop bubbling and prevent any remaining
handlers on the same element from firing.

```js
const container = document.querySelector('#container');
const button = document.querySelector('#container button');

// stopPropagation: button click doesn't reach the container handler
container.addEventListener('click', () => {
    console.log('container clicked');
});

button.addEventListener('click', (e) => {
    e.stopPropagation();
    console.log('button clicked — container handler skipped');
});

// stopImmediatePropagation: second button handler never fires
button.addEventListener('click', (e) => {
    e.stopImmediatePropagation();
    console.log('first handler — stops here');
});

button.addEventListener('click', () => {
    console.log('second handler — never fires');
});
```

### Event delegation

Event delegation uses bubbling to handle events on many elements with a single handler on
their parent. Rather than attaching a listener to every child, you attach one to the
ancestor and inspect `e.target` to determine which child was clicked.

This is especially useful when child elements are added or removed dynamically — new
elements are covered automatically because they bubble up to the same parent.

```js
const list = document.querySelector('#todo-list');

list.addEventListener('click', (e) => {
    if (e.target.matches('.delete-btn')) {
        e.target.closest('li').remove();
    }

    if (e.target.matches('.complete-btn')) {
        e.target.closest('li').classList.toggle('done');
    }
});

function addTodo(text) {
    const li = document.createElement('li');

    const span = document.createElement('span');
    span.textContent = text;           // textContent avoids XSS with user-provided text

    li.append(
        span,
        Object.assign(document.createElement('button'), {
            className: 'complete-btn',
            textContent: 'Done',
        }),
        Object.assign(document.createElement('button'), {
            className: 'delete-btn',
            textContent: 'Delete',
        }),
    );

    list.appendChild(li);
}
```

## Custom events

`CustomEvent` lets you define and dispatch your own event types, which is the cleanest
way to communicate between components without importing or directly calling each other:

```js
function notifyUserLoggedIn(user) {
    const event = new CustomEvent('userLoggedIn', {
        bubbles: true,
        detail: { user },       // payload accessible at e.detail
    });
    document.dispatchEvent(event);
}

document.addEventListener('userLoggedIn', (e) => {
    const { user } = e.detail;
    updateNavBar(user);
});

notifyUserLoggedIn({ name: 'Alice', role: 'admin' });
```

## Debouncing events

High-frequency events like `input`, `scroll`, and `resize` fire many times per second.
Debouncing delays your handler until the event stops firing for a set period:

```js
function debounce(fn, delay) {
    let timer;
    return (...args) => {
        clearTimeout(timer);
        timer = setTimeout(() => fn(...args), delay);
    };
}

const searchInput = document.querySelector('#search');

const search = debounce(async (query) => {
    if (!query) return;
    const results = await fetch(`/api/search?q=${encodeURIComponent(query)}`);
    const data = await results.json();
    renderResults(data);
}, 300);

searchInput.addEventListener('input', (e) => search(e.target.value));
```

Without debouncing, every keystroke fires a network request. With a 300ms delay, the
handler waits until the user pauses.
