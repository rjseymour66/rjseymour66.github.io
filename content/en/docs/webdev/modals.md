---
title: "Modal"
weight: 70
description: >
  Creating modals with HTML, CSS, and Javascript.
---

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