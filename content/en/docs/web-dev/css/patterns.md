---
title: "Patterns and components"
linkTitle: "Patterns"
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

### @counter-style custom bullets

Name the `@counter-style` rule so you can reference it in a `list-style` rule. The at-rule uses these properties to define its behavior:
- `symbol`: what we use to define the bullet style. For example, Unicode.
- `system`: determines how the browser converts the items position in the list to the visual representation on the screen. You can add a space-delimited list to the `symbol` property so the browser displays each item in the list, or add `cyclic` in the `system` property to loop through the the `symbol` values. If you add one value in the `symbol` property, then the browser loops through that.
- `suffix`: what comes between the `symbol` and the list item contents. By default, it is a bullet.


Here is a complete at-rule and its application:

```css
@counter-style emoji {
  symbols: "\2615";
  system: cyclic;
  suffix: " ";
}

article ul {
  list-style: emoji;
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

## Progress bar

You have to use vendor prefixes to style the progress bar:


Chrome properties:
- `::-webkit-progress-inner-element`: Outermost part of the progress element.
- `::-webkit-progress-bar`: Entire progress bar--the part below the inner-element. This is a child of `::-webkit-progress-inner-element`
- `::-webkit-progress-value`: Progress indicator and child of `::-webkit-progress-bar`.

Firefox properties:
`::-moz-progress-bar`: Progress indicator, similar to `::-webkit-progress-value`

```css
progress {
  height: 1.5em;
  width: 100%;

  border-radius: 20px;
  --webkit-appearance: none;        /* remove default chrome styles */
  --moz-appearance: none;           /* remove default firefox styles */
  appearance: none;
}

/* chrome */
::-webkit-progress-value {
  border-radius: 20px;
  background-color: #7be6e8;
}

/* firefox */
::-moz-progress-bar {
  border-radius: 20px;
  background-color: #7be6e8;
}

::-webkit-progress-bar {
  border-radius: 20px;
  background: #4db3ff;
  background: linear-gradient(to right, #128688 0%, #4db3ff 100%);
  border: none;
}

::-webkit-progress-inner-element {
  border-radius: 20px;
}
```

## Time element

This element includes the `datetime` attribute, which translates dates into a machine-readable format for better search engine results and custom features like reminders.

You can style it pretty much as you would any other element:

```html
<time datetime="2021-09-07">Tuesday, 5<sup>th</sup> September 2021</time>
```

```css
time {
  font-weight: 700;
  font-size: 1.5rem;
  font-family: "Oswald", sans-serif;
  text-align: center;
  text-transform: uppercase;

  border-top: 3px solid #333;
  border-bottom: 3px solid #333;
  padding: 12px 0;

  display: block;
}

time sup {
  font-size: 0.875rem;
  font-weight: normal;
}
```

## Blockquote

You can add quotations around a block quote (or any quote?) element with the `open-quote` and `close-quote` values in the `content` property:

```css
blockquote {
  /* styles */
}

blockquote::before {
  content: open-quote;
}

blockquote::after {
  content: close-quote;
}
```

## background shorthand

These are equivalent:

```css
.some-class {
    background: url(image/path) left top / cover blue;
}

.some-class {
    background-image: url(image/path);
    background-position: left top;
    background-size: cover;
    background-color: blue;
}
```
- `background-size: cover;` tells the browser to calculate the optimal size of the image to coer the entire element without distortion. If the image doesn't have the same aspect ratio to the element, parts of the element are clipped.
  - If you don't want to clip the image, use `contain` instead of `cover`
- `background-color: blue;` helps if the image takes too long to load, or if the image is transparent


## background-clip

Along with `text-fill-color`, lets you apply a background image to text. Meaning, you can have large text, make it transparent, and show an image through the text as if the text clipped out that portion of the image. These are experimental, so make sure you provide fallback properties:
- set `background-size: cover;` so the image covers the entire element. The browser automatically calculates the element width and height.
- set `background-clip: text;` so the image shows only behind the letters. You need to add vendor prefixes for this.
- set `<vendor>-text-fill-color: transparent;` so you can see teh image through the text. This supercedes `color`, so you can put it earlier in the cascade. Always add a appropriate `color` fallback in case this fails.

```css
h1 {
  ...
  background: url("images/bg-img.jpg");
  background-size: cover;
  -webkit-background-clip: text;
  background-clip: text;
  -moz-text-fill-color: transparent;
  -webkit-text-fill-color: transparent;
  color: white;
}
```

## Polka-dot background

You can create a simple polka-dot pattern with radial gradient. This rule creates a small circle that transitions into a transparent color, and repeats across the entire page:

```css
body {
  background-image: radial-gradient(var(--accent) 0.75px, transparent 0.75px);
  background-size: 15px 15px;
}
```

## Summary cards with transform

First, apply a background image and styles for the containing section. Here, we add the image with a size of `cover` so you can see as much of the image as possible within the containing element. We also add a `background-color` fallback in case the image doesn't load:

```css
.summary-card-class {
  background-image: url(/images/4.jpg);
}

main > section {
  background-size: cover;
  background-color: #3a8491;
  border-radius: 4px;
}
```

Next, style the content container. Here, it is a `<div>`. This adds margin, padding, and text styles to the content header and body text. We also add a transparent, nearly-black background to add contrast and improve readability:

```css
main > section > div {
  background-color: hsla(0, 0%, 0%, 0.75);
  margin: 1rem;
  padding: 1rem;
  color: whitesmoke;
  text-align: center;
  font-size: 14px;
  font-family: "Rubik", sans-serif;
}

section h2 {
  font-size: 1.3rem;
  font-weight: bold;
  line-height: 1.2;
}

section p {
  font-style: italic;
  font-size: 1.125rem;
  font-family: "Cardo", cursive;
  line-height: 1.35;
}
```

Now, style the links. These styles change the `<a>` tag into an inline block element so you can add padding to increase the clickable area. On focus, the 

```css
a {
  background-color: #ffa600;
  color: rgba(0 0 0 0.75);
  padding: 0.75rem 1.5rem;
  display: inline-block;
  border-radius: 4px;
  text-decoration: none;
}

a:hover {
  background-color: #e69500;
}

a:focus {
  outline: 1px dashed #e69500;
  outline-offset: 3px;
}
```
Format the content in the container with a grid. We use `calc()` to limit the grid size to 100% of the container, minus the padding and margin applied to the `<div>` containing element. We define the rows where the header and link take the minimum space their content requires (`min-content`), and the middle row takes the remainder (`auto`).

```css
main > section > div {
  background-color: hsla(0, 0%, 0%, 0.75);
  ...
  height: calc(100% - 4rem);
  display: grid;
  grid-template-rows: min-content auto min-content;
  align-items: center;
}
```

### Hide with transform

For screens Now, we hide the portion that we want to reveal on hover. To accomplish this, apply the `translateY()` to move the content vertically. Use `calc()` to move it by the card height (350px) minus the container top margin + container top padding + header size.

We also apply a compound `@media` query that works on devices that hover and frequently use hover (`hover:hover`), has a minimum width of 700px, and the browser does not prevent animations:

```css
@media (hover: hover) and (min-width: 700px) and (prefers-reduced-motion: no-preference) {
  main > section > div {
    transform: translateY(calc(350px - 6rem));
  }
}
```

Next, hide the trailing content that does not display until the user hovers. To do this, we set an explicit height on the inner `<div>` container, and hide the overflow.

Because some of the text content is visible, we also hide the non-header content by setting `opacity: 0;`. The additional `translateY` function adds more movement to the content with it displays during a hover action:

```css
@media (hover: hover) and (min-width: 700px) and (prefers-reduced-motion: no-preference) {
  main > section > div {
    ...
    height: 5rem;                       /* otherwise, content displays on hover or focus */
    overflow: hidden;                   /* hide overflow */
  }

  main > section > div > *:not(h2) {
    opacity: 0;
    transform: translateY(1rem);
  }
}
```
### Reveal with transform

Finally, we add styles to reveal the hidden content. We animate the changes with this rule that transitions all properties over 700ms, and it starts slowly, speeds up, then slows down at the end (`ease-in-out`):

```css
transition: all 700ms ease-in-out;
```

We want to reveal the content when the user hovers or when it is in keyboard focus. For the keyboard focus, you use `:focus-within` so we apply styles when a descendant of the element is currently in focus. To complete the transition:
- use `translateY(0)` to remove the vertical displacement
- set the height of the inner containter to the height of the outer container minus its padding and margin
- reset the opacity on the paragraph and link with `opacity: 1;`

```css
@media (hover: hover) and (min-width: 700px) and (prefers-reduced-motion: no-preference) {
  main > section > div {
    ...
    transition: all 700ms ease-in-out;              /* apply animation */
  }

  main > section > div > *:not(h2) {
    ...
    transition: all 700ms ease-in-out;              /* apply animation */
  }
  section:hover div,
  section:focus-within div {
    transform: translateY(0);                       /* remove vertical displacement */
    height: calc(350px - 4rem);                     /* container el height - (padding + margin) */
  }

  section:hover div > *:not(h2),
  section:focus-within div > *:not(h2) {
    opacity: 1;                                     /* reinstate content opacity */
    transform: translateY(0);                       /* remove vertical displacement */
  }
}
```

## Flip card with transform

The general process:
1. With absolute positioning, overlay the two elements that you want to flip, i.e. the front and the back of the card.
2. Place the card with `backface-visibility` and `transform`. `backface-visibility` works when you use a 3D transform property.
3. Animate the change with `transition`.

Here is the abbreviated HTML:

```html
    <section class="card-item">
      <!--Front of the card-->
      <section class="card-item__side front">
        ...
      </section>

      <!--Back of the card-->
      <section class="card-item__side back">
        ...
      </section>
    </section>
```

First, absolutely position the 'back' element and rotate it on its horizontal axis with `transform`. This rotation adds to the illusion that the card is actually being flipped over. We place this in a `hover` media query so machines with hover use the flip, but mobile devices (without hover capabilities) just display the images side-by-side:

```css
@media (hover: hover) {
  .back {
    position: absolute;
    top: 0;
    left: 0;
    transform: rotateY(-180deg);
  }
}
```

Next, apply the `transform` styles that work on hover:
- `transform-style` tells the browser that we are trying to create a 3D effect.
- `backface-visibility` works when you apply a 3D transform property, such as `rotateX()` or `rotateY()`. Here, we are telling the browser to hide the transformed element. In this case, it is the back of the card where we applied `transform: rotateY(-180deg);` previously in the `hover` media query.
- 

```css
@media (hover: hover) {
  ...
  .card-item {
    transform-style: preserve-3d;
  }

  .card-item__side {
    backface-visibility: hidden;
  }

  .card-item:hover {
    transform: rotateY(180deg);
  }
}
```

Now, we need to smoothly transition from the front of the card to the back of the card on hover. Do this with the `transform` property. Here, we add the `transition` property to the containing `card-item` element and add the `prefers-reduced-motion` condition to the media query:

```css
@media (hover: hover) and (prefers-reduced-motion: no-preference) {
    ...
  .card-item {
    transform-style: preserve-3d;
    transition: transform 350ms cubic-bezier(0.71, 0.03, 0.56, 0.85);
  }
    ...
  .card-item:hover {
    transform: rotateY(180deg);
  }
}
```
This rule:
- applies a transistion to the `.card-item:hover { transform: rotateY(180deg); }` declaration
- the animation takes 350ms
- uses a cubic-bezier timing function that consists of four points:
  - points one and four represent the start and end, respectively
  - points two and three are handles on the points that are represented with x and y coordinates
  - these are difficult to calculate by hand. Use [cubic-bezier](https://cubic-bezier.com/#.17,.67,.83,.67) or the browser dev tools.

Here is the complete ruleset:

```css
@media (hover: hover) and (prefers-reduced-motion: no-preference) {
  .back {
    position: absolute;
    top: 0;
    left: 0;
    transform: rotateY(-180deg);
  }

  .card-item {
    transform-style: preserve-3d;
    transition: transform 350ms cubic-bezier(0.71, 0.03, 0.56, 0.85);
  }

  .card-item__side {
    backface-visibility: hidden;
  }

  .card-item:hover {
    transform: rotateY(180deg);
  }
}
```

## focus-visible outline for keyboard nav

`focus-visible` can add an outline to UI elements when a user navigates to the element with the keyboard:

```css
*:focus-visible {
  outline: 1px dotted var(--primary);
  outline-offset: 3px;
}
```

## Tables

Tables have three sections:
- Header: `<thead>`
- Body: `<tbody>`
- Footer: `<tfoot>`

### tbody

You can target specific columns in a table with the `nth-of-type(<n>)` pseudo class. This rule says, "In the `tbody`, select the second element of type `td` and make the font weight bold":

```css
tbody td:nth-of-type(2) {
  font-weight: bold;
}
```

To give the rows a zebra-like style, use the `nth-of-type(<n>)` pseudo class with the `even` (or `odd`) keyword as an argument. This rule gives all even-numbered rows a different background color:

```css
tbody tr:nth-of-type(even) {
  background: #f2fcfc;
}
```

### Borders

By default, each table cell has a border. When you apply a visibile color to the border, it looks like each cell is a block. To get rid of these borders, collapse them. Then, you can add borders to only the top of the rows by targeting the `tr` element.

**Make sure you collapse the borders on the `table` element**:

```css
table {
  border-collapse: collapse;
}

tr {
  border-top: 1px solid #aeb7b7;
}
```

### Responsive tables

Tables do not work well on mobile, so you can make them behave like cards on a smaller screen. This is a special circumstance where you want to use a `max-width` media query (rather than a mobile-first `min-width`) to apply styles until a specific width.

To style the table for mobile:
- make the table cells and table rows `display: block;`. Normally, these are display type `table-cell` and `table-row`
- changing the display type means that the table does not take up its container's width. Make the table width `100%` to change this.
- because the first column contains an image, float it left and add margin so there is space between it and the text
- absolutely position the header off the page. Positioning leaves this available to screen readers. You can access the table header contents with the `data-name` attribute. Position it with a pseudo-selector that uses table cells for placement.
- align the table cell data to the right
- extract the header content with the `attr()` function and the `data-name` attribute. Float it left so it lines up along the floated image
- change the footer row into a flex container, and update the flex item spacing and add padding

```css
table {
  width: 100%;
}

@media (max-width: 549px) {
  td,
  tr {
    display: block;
  }
  table td > img {
    float: left;
    margin-right: 10px;
  }

  thead {
    position: absolute;
    left: -9999rem;
  }

  td {
    text-align: right;
    padding: 5px;
  }

  td[data-name]::before {
    content: attr(data-name) ":";
    float: left;
  }

  tfoot tr {
    display: flex;
    justify-content: space-between;
    align-items: baseline;
  }

  tfoot th {
    padding: 5px;
  }
}
```

## outline-offset

Use `outline-offset` to control where an element's outline is placed. Positive values move the outline farther from the element, and negative values place the outline within the element:

```css
.share__button:hover,
.share__button:focus-visible {
  cursor: pointer;
  outline: 1px solid black;
  outline-offset: -5px;
}
```

## Transparent borders to prevent content shift

If you add a border on a state like `:hover`, the content will shift after the state change if you don't add a transparent border to the stateless element:

```css
.share__link:link,
.share__link:visited {
  height: 48px;
  width: 48px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px solid transparent;
}
```

