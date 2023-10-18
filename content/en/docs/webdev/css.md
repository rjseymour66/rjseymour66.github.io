---
title: "CSS"
# linkTitle: "CSS"
weight: 30
description: >
  Setting up a CSS file.
---

## Reset

- [Reboot, Resets, and Reasoning](https://css-tricks.com/reboot-resets-reasoning/)

  This CSS article provides some background on resets.

- [Meyers reset](https://meyerweb.com/eric/tools/css/reset/)

  Most popular, undoes default user agent styles.

- [Normalize](https://necolas.github.io/normalize.css/)

  Tweaks styles to make them consistent.

- [Browser default styles](https://browserdefaultstyles.com/)

  Look up elements and see default user agent styles, support, etc.

### User agent styles

[Chrome stylesheet](https://chromium.googlesource.com/chromium/blink/+/refs/heads/main/Source/core/css/html.css)

### Example

Add this reset to your file (adapted from [A Modern CSS Reset](https://andy-bell.co.uk/a-modern-css-reset/)):

```css
/* Box sizing rules */
*,
*::before,
*::after {
  box-sizing: border-box;
}

/* Remove default margin */
* {
  margin: 0;
  padding: 0;
  /* inherit font so you can define styles 
  for h1, h* elements instead of rely on user 
  agent defaults */
  font: inherit;
}

/* Remove list styles on ul, ol elements with a list role, which suggests default styling will be removed */
ul[role="list"],
ol[role="list"] {
  list-style: none;
}

/* Set core root defaults */
html:focus-within {
  scroll-behavior: smooth;
}

html,
body {
  height: 100%;
}

/* Set core body defaults */
body {
  text-rendering: optimizeSpeed;
  line-height: 1.5;
}

/* A elements that don't have a class get default styles */
a:not([class]) {
  text-decoration-skip-ink: auto;
}

/* Make images easier to work with */
img,
picture,
svg {
  max-width: 100%;
  display: block;
}

/* Remove all animations, transitions and smooth scroll for people that prefer not to see them */
@media (prefers-reduced-motion: reduce) {
  html:focus-within {
    scroll-behavior: auto;
  }

  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

## Set up variables

This example shows you how to set up variables for the entire site:

```css
:root {
  --clr-accent-500: hsl(12, 60%, 45%);
  --clr-accent-400: hsl(12, 88%, 59%);
  --clr-accent-300: hsl(12, 88%, 75%);
  --clr-accent-100: hsl(13, 100%, 96%);

  --clr-primary-400: hsl(228, 39%, 23%);

  --clr-neutral-900: hsl(232, 12%, 13%);
  --clr-neutral-100: hsl(0 0% 100%);

  --ff-primary: "Be Vietnam Pro", sans-serif;

  --ff-body: var(--ff-primary);
  --ff-heading: var(--ff-primary);

  --fw-regular: 400;
  --fw-semi-bold: 500;
  --fw-bold: 700;

  --fs-300: 0.8125rem;
  --fs-400: 0.875rem;
  --fs-500: 0.9375rem;
  --fs-600: 1rem;
  --fs-700: 1.875rem;
  --fs-800: 2.5rem;
  --fs-900: 3.5rem;

  --fs-body: var(--fs-400);
  --fs-primary-heading: var(--fs-800);
  --fs-secondary-heading: var(--fs-700);
  --fs-nav: var(--fs-500);
  --fs-button: var(--fs-300);

  --size-100: 0.25rem;
  --size-200: 0.5rem;
  --size-300: 0.75rem;
  --size-400: 1rem;
  --size-500: 1.5rem;
  --size-600: 2rem;
  --size-700: 3rem;
  --size-800: 4rem;
  --size-900: 5rem;
}

@media (min-width: 50em) {
  :root {
    --fs-body: var(--fs-500);
    --fs-primary-heading: var(--fs-900);
    --fs-secondary-heading: var(--fs-800);

    --fs-nav: var(--fs-300);
  }
}
```

## Forms

```html
<body>
   <h1>Registration Form</h1>
   <p>Please fill out this form with the required information</p>

   <form action="https://register-demo.freecodecamp.org" method="post">
      <fieldset>
         <label for="first-name">Enter Your First Name: <input id="first-name" type="text" name="first-name"
               required /></label>
         <label for="last-name">Enter Your Last Name: <input id="last-name" type="text" name="last-name"
               required /></label>
         <label for="email">Enter Your Email: <input id="email" type="email" name="email" required /></label>
         <label for="new-password">Create a New Password: <input id="new-password" type="password"
               pattern="[a-z0-5]{8,}" name="new-password" required /></label>
      </fieldset>
      <fieldset>
         <label for="personal-account"><input class="inline" id="personal-account" type="radio" name="account-type" /> Personal
            Account</label>
         <label for="business-account"><input class="inline" id="business-account" type="radio" name="account-type" /> Business
            Account</label>
         <label for="terms-and-conditions">
            <input class="inline" id="terms-and-conditions" type="checkbox" name="terms-and-conditions" required /> I accept the <a
               href="https://www.freecodecamp.org/news/terms-of-service/">terms and conditions</a>
         </label>
      </fieldset>
      <fieldset>
         <label for="profile-picture">Upload a profile picture: <input id="profile-picture" type="file"
               name="profile-picture" /></label>
         <label for="age">Input your age (years): <input id="age" type="number" min="13" max="120" name="age" /></label>
         <label for="referrer">How did you hear about us?
            <select id="referrer" name="referrer">
               <option value="">(select one)</option>
               <option value="1">freeCodeCamp News</option>
               <option value="2">freeCodeCamp YouTube Channel</option>
               <option value="3">freeCodeCamp Forum</option>
               <option value="4">Other</option>
            </select>
         </label>
         <label for="bio">Provide a bio:
            <textarea id="bio" rows="3" cols="30" name="bio" placeholder="I like coding on the beach..."></textarea>
         </label>
      </fieldset>

      <!-- The first input element with a type of submit is automatically set to submit its nearest parent form element. -->
      <input type="submit" value="Submit" />
   </form>

</body>
```

The related CSS stylesheet:

```css
body {
   width: 100%;
   height: 100vh;
   margin: 0;
   background-color: #1b1b32;
   color: #f5f6f7;
   font-family: Tahoma, Geneva, Verdana, sans-serif;
   font-size: 16px;
 }

label {
   display: block;
   margin: 0.5rem 0;
}

h1,
p {
   margin: 1em auto;
   text-align: center;
}

form {
   margin: 0 auto;
   max-width: 500px;
   min-width: 300px;
   width: 60vw;
   padding: 0 0 2em 0;
}

fieldset {
   border: none;
   padding: 2rem 0;
   border-bottom: 3px solid #3b3b4f;
}

fieldset:last-of-type {
   border-bottom: none;
}

/* 
fieldset:not(:last-of-type) {
   border-bottom: 3px solid #3b3b4f;
} */

input,
textarea,
select {
   width: 100%;
   margin: 10px 0 0 0;
   min-height: 2em;
}

.inline {
   /* unsets the 100% width */
   width: unset;
   margin: 0 0.5em 0 0;
   vertical-align: middle;
}

input,
textarea {
   background-color: #0a0a23;
   border: 1px solid #0a0a23;
   color: #fff;
   
}

input[type="submit"] {
   display: block;
   width: 60%;
   margin: 1em auto;
   height: 2em;
   min-width: 300px;
   font-size: 1.1rem;
   background-color: #3b3b4f;
   border-color: white;
}

input[type="file"] {
   padding: 1px 2px;
}

a {
   color: #dfdfe2;
}
```

[HTML elements reference](https://developer.mozilla.org/en-US/docs/Web/HTML/Element)
[CSS Cheat sheet](https://htmlcheatsheet.com/css/)

## Emmet

[Official docs](https://docs.emmet.io/)
[Emmet cheat sheet](https://docs.emmet.io/cheat-sheet/)
[Keybindings plugin](https://marketplace.visualstudio.com/items?itemName=agutierrezr.emmet-keybindings)

Find key bindings with Ctrl + k then Ctrl + s.

### Wrapper

```html
.wrapper>h1{Title}+.content
```

```html
<div class="wrapper">
  <h1>Title</h1>
  <div class="content"></div>
</div>
```

### Auto-gen

Creates button:

```html
button[type="button"]
```

#### Creates div

```
[data-selected]
```

#### Add text to auto-gen elements:

```html
header>nav>ul>li*3{test}
```

```html
<header>
  <nav>
    <ul>
      <li>test</li>
      <li>test</li>
      <li>test</li>
    </ul>
  </nav>
</header>
```

#### Dynamic auto-gen:

```html
header>nav>ul>li*3{List Item $}
```

```html
<header>
  <nav>
    <ul>
      <li>List Item 1</li>
      <li>List Item 2</li>
      <li>List Item 3</li>
    </ul>
  </nav>
</header>
```

#### Dynamic classes:

```html
header>nav>ul>li*3.class-${List Item $}
```

```html
<header>
  <nav>
    <ul>
      <li class="class-1">List Item 1</li>
      <li class="class-2">List Item 2</li>
      <li class="class-3">List Item 3</li>
    </ul>
  </nav>
</header>
```

#### Zero padding

```html
header>nav>ul>li*3.class-${List Item $$}
```

```html
<header>
  <nav>
    <ul>
      <li class="class-1">List Item 01</li>
      <li class="class-2">List Item 02</li>
      <li class="class-3">List Item 03</li>
    </ul>
  </nav>
</header>
```

#### Grouping

Group with parentheses:

```html
(header>nav)+main+footer
```

```html
<header>
  <nav></nav>
</header>
<main></main>
<footer></footer>
```

Complex example:

```html
(header>h2{Heading}+nav>li*5>a{Link $})+main+footer
```

```html
<header>
  <h2>Heading</h2>
  <nav>
    <li><a href="">Link 1</a></li>
    <li><a href="">Link 2</a></li>
    <li><a href="">Link 3</a></li>
    <li><a href="">Link 4</a></li>
    <li><a href="">Link 5</a></li>
  </nav>
</header>
<main></main>
<footer></footer>
```

### Forms

```html
form:post>.group>input:text
```

```html
<form action="" method="post">
  <div class="group"><input type="text" name="" id="" /></div>
</form>
```

```html
form:post>(.group>label+input:text)+(.group>label+input:number)
```

```html
<form action="" method="post">
  <div class="group">
    <label for=""></label><input type="text" name="" id="" />
  </div>
  <div class="group">
    <label for=""></label><input type="number" name="" id="" />
  </div>
</form>
```

## SVG (Scalable Vector Graphics)

SVGs scale to any size without losing quality or increasing file size, and you can modify them with CSS or JS.

Play around with it here: [SvgPathEditor](https://yqnn.github.io/svg-path-editor/).

### Links

#### Free libraries

- [Material icons](https://fonts.google.com/icons)
- [Feather Icons](https://feathericons.com/)
- [The Noun Project](https://thenounproject.com/browse/icons/term/free/)
- [Ionicons](https://ionic.io/ionicons)

#### Misc

- [Make Awesome SVG Animations with CSS](https://www.youtube.com/watch?v=UTHgr6NLeEw)
- [SVG filters](https://www.smashingmagazine.com/2015/05/why-the-svg-filter-is-awesome/)
- [d3 for data visualizations](https://d3js.org/)

### Use cases

- Icons
- Graphs/Charts
- Large, simple images
- Patterned backgrounds
- Applying effects to other elements via SVG filters

### Anti-use cases

Inefficient for storing complex images.

### Anatomy

```html
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <rect x="0" y="0" width="100" height="100" fill="burlywood" />
  <path
    d="M 10 10 H 90 V 90 L 10 60"
    fill="transparent"
    stroke="black"
    stroke-width="3"
  />
  <circle cx="50" cy="50" r="20" class="svg-circle" />
  <g class="svg-text-group">
    <text x="20" y="25" rotate="10" id="hello-text">Hello!</text>
    <use href="#hello-text" x="-10" y="65" fill="white" />
  </g>
</svg>
```

- `xmlns`: XML namespace. Specifies the XML dialect that you are using so the browser can render the SVG correctly.
- `viewBox`: Defines the bounds of the SVG---the positions of different points of the SVG elements in the following order:

  - `min-x`, `min-y`, `width`, and `height`

  Also defines the aspect ratio (ratio of width to height) and the origin (the image's origin of motion for an animation).

- `class`, `id`: Same funciton as in CSS or JS.
- `<circle>`, `<rect>`, `<path>`, `<text>`, [other elements](https://developer.mozilla.org/en-US/docs/Web/SVG/Element): Defined by the SVG namespace that let you build SVGs.

  CSS tricks [SVG Elements by Category](https://css-tricks.com/svg-properties-and-css/#svg-elements-by-category)

- `fill` and `stroke`: Properties that you can edit with CSS and JS.

  [Properties shared between CSS and SVG](https://css-tricks.com/svg-properties-and-css/#shared-properties)

  [SVG CSS Properties](https://css-tricks.com/svg-properties-and-css/#svg-css-properties)

### Embedding SVGs

1. Link with an `<img>` tag or with `background-image: url(./path/to/.svg);`
   - You cannot edit these SVGs.
2. Embed it in the HTML file.
   - Makes HTML harder to read
   - Page is less cacheable
   - Slows HTML loading

You can minimize these drawbacks with React or Webpack.

## Tables

[Table basics](https://developer.mozilla.org/en-US/docs/Learn/HTML/Tables/Basics)
[Table advanced](https://developer.mozilla.org/en-US/docs/Learn/HTML/Tables/Advanced)
[Data Table Design UX Patterns](https://pencilandpaper.io/articles/ux-pattern-analysis-enterprise-data-tables/)
