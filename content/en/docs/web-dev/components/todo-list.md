---
title: "Todo list"
# linkTitle: ""
weight: 40
# description:
---

This todo list example is really about making inclusive components that you create or delete.

## Empty state

Before you add items to your todo list, you will have an empty space. You need to inform users and screen readers there is a part of the page that can contain content, even if it does not currently contain content.

To accomplish this, place an element with an `empty-state` class after the `<ul>` element:

```html
<section aria-labelledby="todos-label">
    <h1 id="todos-label">My Todo List</h1>
    <ul>
        <!-- no todos yet -->
    </ul>
    <div class="empty-state">
        <p>Add your first todo&#x2193;</p>
    </div>
</section>
```

Pseudo-class selectors target element states, so apply styles when the `<ul>` is `:empty`:
- The first ruleset says "by default, the `.empty-state` element and an empty `<ul>` should be visually hidden and not accessible to screen readers."
- The second ruleset says "when the `<ul>` is empty, display the sibling element with the `.empty-state` class."

```css
.empty-state, ul:empty {
    display: none;
}

ul:empty + .empty-state {
    display: block;
}
```

## Forms for keyboard users

Use a form to users can press Enter and submit their todo. Placeholder text is not accessible because it disappears when users begin typing. Because the button in this case is descriptive enough for what we are trying to accomplish (it says "Add"), we do not have to add a label. However, as a best practice, we can add a label and hide it.

The `visually-hidden` class hides it from the user but makes it accessible to screen readers:

```html
<form action="#">
    <label for="add-todo" class="visually-hidden">Add a todo item</label>
    <input id="add-todo" type="text" placeholder="e.g. Adopt an owl" />
    <button type="submit">Add</button>
</form>
```

Here is the class that hides the label, and styles to make the placeholder text more accessible:

```css
.visually-hidden-sr {
    clip-path: inset(50%);        /* Clips all visual content (effectively hides it from view) */
    height: 1px;                  /* Makes the element tiny but still technically on the page */
    width: 1px;                   /* Makes the element tiny but still technically on the page */
    overflow: hidden;             /* Prevents scrollbars or text spill */
    position: absolute;           /* Removes the element from the normal flow so it doesn’t affect layout */
    white-space: nowrap;          /* Prevents line breaks that could affect screen reader behavior */
}

input::placeholder {
    color: #444;
    font-style: italic;
}
```
### aria-label

Alternately, you could just add the `aria-label` to the input element:

```html
<form action="#">
    <input type="text" aria-label="Write a new todo item" placeholder="e.g. Adopt an owl" />
    <button type="submit">Add</button>
</form> 
```

## Validation

Do not disable the submit button until the input is valid. Disabled buttons are not focusable by keyboard, so they might be missed entirely. It is better to allow users to enter invalid information and add `aria-invalid="true"` to the input element:

```html
<form action="#">
    <label for="add-todo" class="visually-hidden">Add a todo item</label>
    <input
        id="add-todo"
        type="text"
        aria-invalid="true"
        placeholder="e.g. Adopt an owl"
    />
    <button type="submit">Add</button>
</form>
```

## Feedback live region

A live region is an element that tells a screen reader to announce its contents when the contents change. You can use this to wrap status messages so they are announced when they appear visually.

To define a live region, add `role="status"` and `aria-live="polite"`. Using `polite` means that the heading is announced before the status message. This is in contrast to `aria-live="assertive"`, which immediately announces the status message.

We also hide the live region if there is another way that the todo list alerts readers that a todo was added:

```html
<div role="status" aria-live="polite" class="visually-hidden">
    <!-- live region contents -->
</div>
```

Here is a JS function that announces updated content in the live region:

```js
const liveRegion = document.querySelectorAll('role="status"]');

let addedFeedback = todoName => {
    liveRegion.textContent = `${todoName} added.`;
};
```

## Checking off todos

Here is a style that targets a label that is the sibling of a checked checkbox:

```css
:checked + label {
    text-decoration: line-through;
}
```

## Deleting todos

The button to delete a todo is often a trashcan SVG. Because screen readers cannot read an SVG without a label, we give it a label and hide it:

```html
<ul class="todo-list">
    <li>Pick up kids from school
        <button>
            <svg><!-- svg content --></svg>
            <span class="visually-hidden">delete {todo.name}</span>
        </button>
    </li>
    ...
```

### SVG bloat

Adding multiple instances of the same SVG can cause performance issues. To reduce this, we use a pattern where the SVG is defined as a symbol at the head of the document body and reused with the `<use>` tag.

This method requires some specific steps:
- add `style="display: none"` to the SVG image so it doesn't display
- remove all styling from the SVG except the display style and the `xmlns` value
- wrap the path in a `<symbol>` tag. `<symbol>` should include an `id` and a `viewBox` value. Material icons use `viewBox="0 -960 960 960"`

After you create the SVG, reference the `<symbol>`'s `id` value to in the `<use>` element to render the SVG:

```html
<body>
    <svg style="display: none" xmlns="http://www.w3.org/2000/svg">
        <symbol id="trash-icon" viewBox="0 -960 960 960">
            <path
                d="M280-120q-33 0-56.5-23.5T200-200v-520h-40v-80h200v-40h240v40h200v80h-40v520q0 33-23.5 56.5T680-120H280Zm400-600H280v520h400v-520ZM360-280h80v-360h-80v360Zm160 0h80v-360h-80v360ZM280-720v520-520Z"
                fill="red"
            />
        </symbol>
    </svg>

    <!-- ... -->

    <button>
        <svg>
            <use href="#trash-icon"></use>
        </svg>
        <span class="visually-hidden">delete</span>
    </button>
```

<svg width="24" height="24" viewBox="0 -960 960 960" fill="currentColor">
  <use href="#trash-icon" />
</svg>

### Focus

After you delete a todo, you need to decide where to focus because the element that was focused was just deleted. The best way to reorient users is to focus the page heading. This way, they can tab from the heading directly to either the first list item or the live region that says there are no items in the list.

Add `tabindex="-1"` to the element. This has the following benefits:
- Users can't focus the element with Tab.
- Only you can focus the element with JS.

```html
<h1 id="todos-label" tabindex="-1">My Todo List</h1>
```

You do not need to give this element focus styles because it is not interactive:

```css
[tabindex="-1"] { outline: none; }
```

Focusing in JS is simple:

```js
const h1 = document.querySelector('#todos-label');
h1.focus();
```

### Live region

You should also update the live region with a message:

```js
let deletedFeedback = (todoName) => {
    liveRegion.textContent = `${todoName} deleted.`;
};
```