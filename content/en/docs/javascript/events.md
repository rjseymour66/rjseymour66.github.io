---
title: "Events"
linkTitle: "Events"
weight: 20
description:
---



JS uses event-driven programming, like all GUI apps. This means that the browser generates an event when something interesting happens to the document or browser:
- For example:
  - Browser finishes loading a document
  - User clicks button
  - Moves mouse across the screen
- Events can occur on any element within an HTML doc
- JS can register a function to invoke when a specific event happens

## Links

- [Eloquent Javascript "Handling Events"](https://eloquentjavascript.net/15_event.html)
- [MDN Intro to Events](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events)

## Event Model

There are three important parts of the event model:

- **Event type**: A string that specifies the type of event - also called 'event name'. Ex: 'mousemove', 'keydown', 'load'

- **Event target**: Object on which the event occurred, usually a Window, Document, or Element object but can also be a Worker object. Ex: Load event on window, selecting a `<button>` element.

- **Event handler, event listener**: Function that handles or responds to an event. Handler and listener are synonymous, but sometimes they are used according to how the function addresses the event.
  
  Applications register the handlers with the browser by specifying the event type and event target. When an event of that type occurs on the target, the browser calls the function.

  When a handler is invoked, we also call it 'fired', 'triggered', or 'dispatched'.

- **Event object**: A JS object that contains details about the event, and are passed as an arg to the event handler. All events have the following properties:
  - `type`: specifies the event type
  - `target`: event target
  
  Some objects have additional properties, such as the screen coordinates for a mouse event.

- **Event propagation**: This is the process that the browser uses to decide which objects to trigger event handlers on.
  
  Load or Worker events don't propagate.

  Some events in the document propagate, also called 'bubbling up' the document tree. An event handler can stop the propagation with a special method on the event object.

  _Event capturing_ is when an event handler registered to a container element can intercept an event before it gets to its target.

## Categories

There are a lot of event categories. This list groups events in general categories:
- Device-dependent input events: Tied to specific input device, like mouse or keyboard
- Device-independent input events: Not tied to a device. Ex: click events. These are often device-agnostic alternatives to device-dependent events. For example, 'input' event instead of 'keydown', or 'pointerdown', instead of 'mousedown'. These are useful for touch screens, stylus pens, etc.
- UI events: High-level events, often on form elements. Ex: 'focus', 'change', 'submit'.
- State-change events: Triggered by browser or network activity, not users. Signal a life-cycle or state-related change, such as 'load' or 'DOMContentLoaded'. For network connection, the browser fires 'online' and 'offline'.
- API-specific events: Web APIs, like audio or video, have their own event types. These events are because the API is asynchronous, and they were developed before Promises were made, so you needed to be able to determine when an event occurred.

## Registering event handlers

Two ways to register an event handler:
- Set property on the event target (old, legacy way)
- Pass the handler to the object's or element's `addEventListener()` method

### Set property

Not recommended because you can only register one event handler to the event target.

Set the property with `on<eventname>`. For example, `window.onload`:

```js
window.onload = function() {
    alert('The window just loaded!')
}
```

### Set event handler attributes

Not a best practice, avoid when possible. The way the browser executes these event handlers is confusing because it uses unexpected variables.

You just add the body of a function as an HTML element property. Then, the browser converts your string into a function behind the scenes. For example:

```js
<body onload="alert('The window loaded!');">
    ...
</body>
```

### addEventListener()

All objects that can be an event target have an `addEventListener()` method that you can use to register events for that target.

`<target>.addEventListener(<event-type>, <function>, <capture>)`

Takes 3 arguments:
1. Event type that you are registering the handler for. Ex: 'click'
2. Function that is invoked when the specified type of event occurs
3. Capturing definition. Accepts a Boolean or object

```js
let button = document.querySelector('.btn');
button.addEventListener('click', () => console.log(button.textContent));
button.addEventListener('click', () => console.log('registered another listener'));
```

When you register more than one handler to an object, the handlers fire in the order they are registered:
- You can't register more than one handler with identical arguments, it only fires once

Remove an event listener with `removeEventListener()`:
- Takes the same args as `addEventListener()`, including the third optional argument

```js
let clickCount = 0;

let logSecondListener = () => {
    console.log("Registered another listener");
    clickCount++;
    if (clickCount > 0) {
        button.removeEventListener('click', logSecondListener);
    }
};

let button = document.querySelector('.btn');
button.addEventListener('click', () => console.log(button.textContent));
button.addEventListener('click', logSecondListener);
```

The third argument is optional, and can either be a Boolean or an object that specifies the exact behavior you want:
```js
document.addEventListener('click', logSecondListener, {
capture: true,
once: true,
passive: true
});
```
If you pass `true`, then the event handler is registered as a _capturing handler_.

If you pass the object:
- `capture`: Boolean, determines whether the event handler is a _capturing handler_
- `once`: Boolean, automatically remove the handler after it is invoked once
- `passive`: Boolean, controls `preventDefault()` behavior. If `true`, the handler will never call `preventDefault()`. Lets the web browser know that it can use its default behavior when the handler is running.
  Firefox and Chrome make `touchmove` and `mousewheel` events passive by default, because the browser needs to be able to scroll when users are touching the screen. 

### Add handler on each node in group

1. Grab the elements with `querySelectorAll()` to get a nodelist.
2. Use `forEach()` to add an event listener to all the nodes:

   ```html
   <div id="container">
     <button id="1">Click Me</button>
     <button id="2">Click Me</button>
     <button id="3">Click Me</button>
   </div>
   ```

   ```js
   // buttons is a node list. It looks and acts much like an array.
   const buttons = document.querySelectorAll("button");

   // we use the .forEach method to iterate through each button
   buttons.forEach((button) => {
     // and for each one we add a 'click' listener
     button.addEventListener("click", () => {
       alert(button.id);
     });
   });
   ```

## Event handler invocation

- Event handlers should not return anything. If you do not want a handler to perform a default action, call `preventDefault()`.
- Event handlers are invoked in the order that they are registed.

### Event object

When the browser invokes an event handler, it passes an Event object as the single argument:

```js
button.addEventListener('click', e => console.log(e));
```
Event object useful properties:
- `type`: Type of event
- `target`: Object that the event occurred on
- `currentTarget`: If the event propagates, the object that the current event was registered on
- `timeStamp`: Timestamp (in milliseconds), but not absolute time. Calculate elapsed time between two events by subtracting the first `timeStamp` from the second.
- `isTrusted`: `true` if the event handler was dispatched from the browser, and `false` if dispatched from JS script.
- `bubbles`: Boolean, whether the event bubbles up the DOM.
- `cancelable`: Boolean, whether the event can be prevented.
- `defaultPrevented`: Boolean, whether `event.preventDefault()` was called.

Some events have special properties. For example, mouse events have the `clientX` property that shows the window coordinates.

#### Event handler context

The `this` value is the event target, unless you use an arrow function. Arrow functions have the same `this` value as the scope in which they are defined:

```js
button.addEventListener('click', function () { console.log(this); });   // <button class="btn">Click me!</button>
button.addEventListener('click', () => console.log(this));              // Window {window: Window, self: Window,...}
```

### Event propagation

Event propagation is not about triggering event handlers, its about how an event on a nested element goes up the DOM tree - it 'bubbles' up the DOM.

Event propagation doesn't happen on the Window or a different standalone object. It happens on the Document or a document Element object:
- Events 'bubble up' - the events on its parent, grandparent, etc elements are fired, all the way up to the Window object
- This is an alternative to registering event handlers on lots of elements.
  - register an event handler on an ancestor element instead of lots of its descendant elements. When you click on an descendant element, it bubbles up to the Window
- These events DO NOT bubble:
  - 'focus'
  - 'blur'
  - 'scroll;'
- 'load' event on a document elements bubble, but stop at the Document object - doesn't bubble up to the Window object

#### Phases of event propagation:
1. Capturing phase - event handlers registered as capturing handlers fire first. This is like bubbling in reverse: Window capturing handlers fire, then Document capturing handlers fire, etc, all the way down to the parent of the event target. Handlers on the event target are not fired.
   
   Capturing event handlers is useful in debugging and for handling mouse drags.
2. Invocation of event handlers of the target object
3. Event bubbling

### Event cancellation

The browser responds to many user events by default: entering text, scrolling, etc. 
- If you register a handler to one of these events, you can stop this default behavior with the `preventDefault()` method of the object.
- To stop event bubbling, call the event object's `stopPropagation()` method. Still lets you call other event handlers on the event object
- To stop all events on the event object, call `stopImmediatePropagation()`

```js
// --- stopPropagation() --- //
container.addEventListener('click', e => {
        console.log(`${e.target} triggered an ${e.type}`);
    }
);

button.addEventListener('click', e => {     // does not bubble up and fire container click event
    console.log('clicked the button');
    e.stopPropagation();
});

// --- stopImmediatePropagation() --- //
button.addEventListener('click', e => {
    console.log('clicked the button');
    e.stopImmediatePropagation();           // this handler was registered first, so
});                                         // fires before the second handler

button.addEventListener('click', () => console.log('second event listener'));   // this will not fire
```

### Event delegation

https://javascript.info/event-delegation