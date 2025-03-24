---
title: "Patterns"
# linkTitle: "CSS in Depth"
weight: 300
# description:
---

## Navigation menu

When creating menu items:
- Apply padding to the internal `<a>` tags to provide more clickable surface area.
- Make the `<a>` tags block elements with `display: block;`. This means that you can control their height with their padding and content.
- Add space between flex items with the `gap` property. Define `gap` as a variable.
- Use `margin: auto` to fill available space between flex items, like place a flex item on the other side of a flex container.

Here is the HTML:

```html
<nav>
    <ul class="site-nav">
        <li><a href="/">Home</a></li>
        <li><a href="/features">Features</a></li>
        <li><a href="/pricing">Pricing</a></li>
        <li><a href="/support">Support</a></li>
        <li class="nav-right">
            <a href="/about">About</a>
        </li>
    </ul>
</nav>
```

Horizontal nav styling. The `ul` is the flex container, while each `li` is a flex item:

```css
:root {
    --gap-size: 1.5rem;
}

.site-nav {
    display: flex;
    gap: var(--gap-size);
    padding: 0.5rem;
    list-style-type: none;
    background-color: #5f4b44;
}

.site-nav > .nav-right {
    margin-inline-start: auto;
/*  margin-left: auto; */
}

.site-nav > li > a {
    display: block;
    padding: 0.5em 1em;
    background-color: #cc6b5a;
    color: white;
    text-decoration: none;
}
```

## Tables

- [Table basics](https://developer.mozilla.org/en-US/docs/Learn/HTML/Tables/Basics)
- [Table advanced](https://developer.mozilla.org/en-US/docs/Learn/HTML/Tables/Advanced)
- [Data Table Design UX Patterns](https://pencilandpaper.io/articles/ux-pattern-analysis-enterprise-data-tables/)


## `dialog` element

Read this [blog post](https://blog.webdevsimplified.com/2023-04/html-dialog/).

You can use the `<dialog>` element instead of HTML, CSS, and JS.

Code dump:

```html
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="styles.css">
    <!-- <script src="library.js" defer></script> -->
    <title>Library | Odin</title>

    <style>
        .modal {
            padding: 0.5rem;
            /* top: 50%; */
            /* left: 50%; */
            /* translate: -50% -50%; */
            border: 1px solid black;
            border-radius: 0.25rem;

        }

        .modal::backdrop {
            background-color: rgba(0, 0, 0, .3);
        }
    </style>
</head>

<body>

    <button class="modal-open">Open</button>
    <dialog class="modal">
        <div>This is a modal</div>
        <button class="modal-close">Close</button>
    </dialog>

</body>

<script>
    const modal = document.querySelector('.modal');
    const openButton = document.querySelector('.modal-open');
    const closeButton = document.querySelector('.modal-close');

    openButton.addEventListener('click', () => {
        modal.showModal();
    });

    closeButton.addEventListener('click', () => {
        modal.close();
    });

    modal.addEventListener("click", e => {
        const dialogDimensions = modal.getBoundingClientRect();
        if (
            e.clientX < dialogDimensions.left ||
            e.clientX > dialogDimensions.right ||
            e.clientY < dialogDimensions.top ||
            e.clientY > dialogDimensions.bottom
        ) {
            modal.close();
        }
    })
</script>
```

## HTML, CSS, JS

An example of a `position: fixed` element is a modal. Here is the HTML:

```html
...
<div>
    <div class="modal" id="modal"></div>
    <div class="modal-backdrop"></div>
    <div class="modal-body">
        <button class="modal-close" id="close" />
        ... modal contents
    </div>
</div>
```

You position in in the viewport using the following:

```css
/* The modal-backdrop (grayed-out area behind the actual modal) */
/* The backdrop covers the entire viewport. */
.modal-backdrop {
    position: fixed;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    background-color: rgba(0, 0, 0, 0.5);
}

/* Like the backdrop, the modal-body is positioned within the viewport */
.modal-body {
    position: fixed;
    top: 3em;
    right: 20%;
    bottom: 3em;
    left: 20%;
    background-color: #fff;
    /* allow scroll if necessary */
    overflow: auto; 
}
```

Use JS to grab the `modal` id, then use `display: none` or `display: block` to toggle it on or off.

## Images

### Circular image

Many profile images use a circular image, where the image is centered. The `height` and `width` must be equal, and center the image with `object-fit: cover;`

```css
img {
    width: 5em; /* adjust width and height as needed */
    height: 5em;
    border-radius: 50%;
    object-fit: cover;
}
```

## Lists

### Custom bullets

This example creates custom bullets using the `::before` pseudo-class:

1. Remove the bullets and padding from the `ol` element or list class.
2. Add relative positioning to the `li` to create a containing block for the bullet.
3. Create the `::before` pseudo-class on the `li` element, and add the following styles:
   - bullet content
   - absolute positioning
   - to center the bullet, apply `top: 50%;` and `transform: translateY(-50%);`. When an `li` takes more than one line, this places the bullet directly in the center of the wrapped line

```css
ul {
    list-style-type: none;
    padding-inline: 0;
}

ul > li {
    position: relative;
    padding-inline-start: var(--padding-list-align);
    line-height: var(--line-height-lg);
}

ul > li::before {
    content: "\2022";
    font-weight: var(--fw-heavy);
    color: var(--clr-accent-markers);
    position: absolute;
    left: 0;
    top: 50%;
    transform: translateY(-50%);
}
```

### Custom numbers

This example adds custom numbers to an `ol`:
1. Remove the numbers from the agent styles, remove padding, then create a custom counter. Optionally, you can set the counter to start on a new number.
2. Add relative positioning to the `li`, and set the `counter-increment` to the custom counter you defined on the parent `ol` element.
3. Create the custom numbers with the `::before` pseudo-class, and add the following styles:
   - content--add your custom counter with the `counter()` function, and add in quotations any additional content to follow the number. For example, a period (`.`).
   - absolute position the pseudo-element
   - position with `left` and `top`, etc.

```css
ol {
    list-style-type: none;
    padding-inline: 0;
    counter-reset: custom-counter;  /* counter-reset: custom-counter 10; */
}

ol > li {
    position: relative;
    padding-inline-start: var(--padding-list-align);
    line-height: var(--line-height-lg);
    counter-increment: custom-counter;
}

ol > li::before {
    content: counter(custom-counter) ".";
    font-weight: var(--fw-heavy);
    color: var(--clr-accent-markers);
    position: absolute;
    left: 0;

    padding-inline-start: var(--marker-padding);
}
```