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

> NOTE
> CSS In-depth recommended adding this to each stylesheet:
> ```css
> button,
> input {
>   font: inherit;
> }
> ```
> This might be taken care of by the `* {... font: inherit; }` ruleset below.

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
  line-height: 1.2;
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

/* button, */
input,
select,
textarea {
  width: 150px;
  padding: 0;
  margin: 0;
  box-sizing: border-box;

  font-family: inherit;
  font-size: 100%;

  /* removes system-level styling */
  appearance: none;
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

- [Table basics](https://developer.mozilla.org/en-US/docs/Learn/HTML/Tables/Basics)
- [Table advanced](https://developer.mozilla.org/en-US/docs/Learn/HTML/Tables/Advanced)
- [Data Table Design UX Patterns](https://pencilandpaper.io/articles/ux-pattern-analysis-enterprise-data-tables/)


## CSS Units


## Text and typography

Good article: [Modern CSS Techniques to Improve Legibility](https://www.smashingmagazine.com/2020/07/css-techniques-legibility/)

If your `font-family` property setting uses this hierarchy:

1. First setting
2. Fallback font
3. Default user agent style

You can make sure you get the font that you want with the [system font stack](https://css-tricks.com/snippets/css/system-font-stack/), which is a long list of fallback fonts.

With variables:

```css
:root {
  --system-ui: system-ui, "Segoe UI", Roboto, Helvetica, Arial, sans-serif,
    "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
}

.element {
  font-family: var(--system-ui);
}
```

Without variables:

```css
/* Define the "system" font family */
@font-face {
  font-family: system-ui;
  font-style: normal;
  font-weight: 300;
  src: local(".SFNSText-Light"), local(".HelveticaNeueDeskInterface-Light"),
    local(".LucidaGrandeUI"), local("Ubuntu Light"), local("Segoe UI Light"),
    local("Roboto-Light"), local("DroidSans"), local("Tahoma");
}

/* Now, let's apply it on an element */
body {
  font-family: "system-ui";
}
```

### Online fonts

To prevent long lists of font-family values, you can include an online font library:

- [Google Fonts](https://fonts.google.com/)
- [Font Library](https://fontlibrary.org/)
- [Adobe Fonts (not free)](https://fonts.adobe.com/)

You can include them with link tags in the HTML or `@import` statement at the top of the stylesheet.

### Downloaded fonts

A downloaded font is a font that you downloaded from the web. This requires that you use the `@font-face` rule, then use the `font-family` property:

```css
@font-face {
  font-family: my-cool-font;
  src: url(../fonts/the-font-file.woff);
}

h1 {
  font-family: my-cool-font, sans-serif;
}
```

Here are some links:

- [W3 general info](https://www.w3schools.com/css/css3_fonts.asp)
- [Font file formats](https://fileinfo.com/filetypes/font)

## Font styles

### Italics

- `font-style`:
  - If italics are for styling purposes, use `font-style: italic;`
  - If italics are for emphasis, use `<em>` tags
- `letter-spacing`: space between letters
- `line-height`: space between lines in wrapped text.
- `text-transform`: Changes the case of the text (ex: all UPPERCASE)
- `text-shadow`: Shadow around text. `text-shadow: offset-x | offset-y | blur-radius | color` [MDN docs](https://developer.mozilla.org/en-US/docs/Web/CSS/text-shadow)
- `ellipsis`: Use this with other properties to put an ellipsis in place of overflowing text:
  ```css
  .overflowing {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
  ```
  You need this because text will write outside of its container, by default. See [CSS Tricks](https://css-tricks.com/snippets/css/truncate-string-with-ellipsis/) for in-depth info.

## CSS properties

### background

[MDN docs](https://developer.mozilla.org/en-US/docs/Web/CSS/background)
[CSS Tricks](https://css-tricks.com/almanac/properties/b/background/)

Includes 8 properties that can do the following:

- change background color
- add background images
- change position and size of background images
- repeat or tile the images
- create background layers

### box-shadow

[MDN docs](https://developer.mozilla.org/en-US/docs/Web/CSS/box-shadow)

Adds effects around elements.

### overflow

[MDN docs](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow)
[CSS Tricks](https://css-tricks.com/almanac/properties/o/overflow/)

How an elment behaves when its content is too big.

### opacity

[MDN docs](https://developer.mozilla.org/en-US/docs/Web/CSS/opacity)

Sets the opacity.

## Advanced selectors

[MDN combinators](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Selectors/Combinators)

### Pseudo-classes

[MDN classes vs elements](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Selectors/Pseudo-classes_and_pseudo-elements)
[MDN docs](https://developer.mozilla.org/en-US/docs/Web/CSS/Pseudo-classes#location_pseudo-classes)

Classes use a single colon and target elements that already exist in the HTML. Some common ones are:

- `:focus `
- `:hover `
- `:active`

Structural pseudo-classes:

- `:root` that represents the very top level of your document
- `:first-child`
- `:last-child`
- `:empty` for elements that have no children at all
- `:only-child` for elements that do not have any siblings
- `:nth-child`

  ```css
  .myList:nth-child(5) {
    /* Selects the 5th element with class myList */
  }

  .myList:nth-child(3n) {
    /* Selects every 3rd element with class myList */
  }

  .myList:nth-child(3n + 3) {
    /* Selects every 3rd element with class myList, beginning with the 3rd */
  }

  .myList:nth-child(even) {
    /* Selects every even element with class myList */
  }
  ```

### Pseudo-elements

[MDN docs](https://developer.mozilla.org/en-US/docs/Web/CSS/Pseudo-elements)

Elements use a double colon and target elements that do not normally exist or are not elements at all:

- `::marker` style `li` bullets or numbers
- `::first-letter`
- `::first-line`
- `::selection` change styling when user selects text on the page.
- `::before`
- `::after`

### Advanced selectors

[Complete list](https://css-tricks.com/almanac/selectors/)
[MDN attribute selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors)
[Complex Selectors](https://learn.shayhowe.com/advanced-html-css/complex-selectors/)
[CSS Tricks: Taming Advanced CSS Selectors](https://www.smashingmagazine.com/2009/08/taming-advanced-css-selectors/)
[CSS Tricks: The Skinny on CSS Attribute Selectors](https://css-tricks.com/attribute-selectors/)

Select and style any attribute:

- `[attribute]`
- `selector[attribute]`
- `[attribute="value"]`

```css
[src] {
  /* This will target any element that has a src attribute. */
}

img[src] {
  /* This will only target img elements that have a src attribute. */
}

img[src="puppy.jpg"] {
  /* This will target img elements with a src attribute that is exactly "puppy.jpg" */
}
```

- `[attribute^="value]` match from the start
- `[attribute$="value"]` match from the end
- `[attribute*="value]` match anywhere inside the string

```css
[class^="aus"] {
  /* Classes are attributes too!
    This will target any class that begins with 'aus':
    class='austria'
    class='australia'
  */
}

[src$=".jpg"] {
  /* This will target any src attribute that ends in '.jpg':
  src='puppy.jpg'
  src='kitten.jpg'
  */
}

[for*="ill"] {
  /* This will target any for attribute that has 'ill' anywhere inside it:
  for="bill"
  for="jill"
  for="silly"
  for="ill"
  */
}
```

## Positioning



[MDN docs](https://developer.mozilla.org/en-US/docs/Web/CSS/position)

[CSS Tricks](https://css-tricks.com/absolute-relative-fixed-positioining-how-do-they-differ/)

[Fixed vs Sticky](https://www.kevinpowell.co/article/positition-fixed-vs-sticky/)

`position: static;` is the default mode for all elements. `top`, `right`, `bottom`, and `left` do not affect static elements.

When you position elements, you remove them from the normal document flow. This means that the positioned elements do not affect other elements, and other elements do not affect the positioned element.

### z-index

Relative and absolute positioning remove the element from the document flow. This might cause issues with other positioned elements.

When HTML is parsed, the browser creates the DOM tree and a render tree. The render tree represents the physical appearance and position of each element, as well as the order that the browser paints each element.

First, the browser paints all non-positioned elements, then positioned elements according to the _z-index_. Elements that the browser paints later appear in front of previously painted elements, should they overlap.

Elements with a `z-index` establish a _stacking context_. A stacking context is a n element or a group of elements that are painted together with a browser. Elements with a higher `z-index` are positioned in front of elements with a lower `z-index`. Elements with a negative `z-index` are positioned behind static elements. No element outside the stacking context can be positioned between elements in the stacking context.

If an element is positioned within another element's stacking context, then the nested element is painted with the parent element. For example, if a div uses `position: relative;` to act as the containing div for an `abolute` positioned element, then the `absolute` element is painted with its `relative` parent.

Elements within the stacking context are stacked in this order:
- root element of the stacking context
- positioned elements with negative z-index and their children
- non-positioned elements
- positioned elements witha  z-index of `auto` and their children
- positioned elements with a positive z-index

### Cheatsheet 

| Positioning | Use cases | Containing block | Description |
|:------------|:----------|:-----------------|:------------|
| Fixed       | modal, navigation bars, floating chat buttons   | Viewport         | Positioned relative to viewport. |
| Absolute    | popup menus, tooltips, "info" boxes | Closest-positioned ancestor element. |  |
| Relative    | dropdown menus | element with `position: absolute;` |  |
| Sticky      | section headings |  |  |


### Relative

> Usually establishes the containing block for an absolute element.

Positions relative to the parent element and removes it from the doc flow.

You cannot size the element with the `inset` property---you can only move it in relation to its original location. If you apply both `top` and `bottom`, then `bottom` is ignored. If you apply both `left` and `right`, then right is ignored.

#### Create a dropdown menu

A dropdown consists of a containing div that is positioned `relative`. The following elements are within this container:
- button that you click to open the menu
- div that contains a ul. Set this div to `display: none;`. When it is visible, set the display to `block`. This div needs to display exactly below the button that opens the menu, so compute the side of the button element and use apply that value to this div's `top` property.
  - a ul with the following styles:
  ```css
  .ul-class {
    padding-inline-start: 0;
    margin: 0;
    list-style-type: none;
    border: 1px solid #999;
  }
 
  .ul-class > li + li {
    border-top: 1px solid #999;
  }
 
  .ul-class > li > a {
    display: block;
    padding: 0.5em 1.5em;
    background-color: #eee;
    color: #369;
    text-decoration: none;
  }
  
  .ul-class > li > a:hover {
    background-color: #fff;
  }
  ```

#### Use cases

[Custom select dropdown](https://www.webaxe.org/accessible-custom-select-dropdowns/) menus.

### Absolute

Absolute positioned elements are positioned based on their closest-positioned ancestor element. That ancestor element is called the _containing block_. The `inset` property values place the absolute positioned element within that containing block.

> Usually, an `absolute` element's containing block is set to `relative`.

Position something at an exact point on the screen without affecting any elements around it. It removes it from the normal document flow and positions it relative to an ancestor element.

#### Use cases

- modals
- image with a caption on top of it
- icons on top of other elements

### Fixed

Fixed elements stay in the same place as the user scrolls.

Removed from the normal flow of the document and positioned relative to the viewport (the viewport is the _containing block_) with `top`, `right`, `bottom`, and `left`. These settings also size the element. If you set `left` and `right` to `5em`, then the width of the fixed element is 10em less than the viewport width.

`inset` is the shorthand and it accepts values in the same format as the `padding` or `margin` shorthand:

```css
.class {
  inset: <top> <right> <bottom> <left>;
}
```

To make the element take up the entire viewport, use `inset: 0;`. This is helpful to darken the background for a smaller modal element.

#### Create a modal

A modal consists of a div container named 'modal' or something intuitive (add `aria-modal=true`, too). Place the modal HTML right before the closing `</body>` tag.

The following elements are nested in this div container:
- empty div that serves as the modal backdrop. This needs to be fixed with an `inset: 0;`.
- div that contains the modal contents. Set the `display: none;` so you can make it visible with JS (described next).
- A button to display the modal. The button can add a class that changes the modal display from `none` to `block`.
- A button that closes the modal, and a class that changes the display back to `none`.
- A class that prevents the screen from scrolling.
- JS that adds and removes the class that displays the modal and the class that prevents scrolling.

Here is a complete example:

```html
<!-- <header>...</header> -->
<div class="modal" id="modal" role="dialog" aria-modal="true">
    <div class="modal-backdrop"></div>
    <div class="modal-body">
        <!-- modal body contents -->
    </div>
</div>
<!-- <main> ... </main> -->
```

```css
.modal {
  display: none;
}

.modal.is-open {
  display: block;
}

.modal-backdrop {
  position: fixed;
  inset: 0;
  background-color: rgb(0 0 0 / 0.5);
}

.modal-body {
  position: fixed;
  inset-block: 3em; /* inset top and bottom */
  inset-inline: 20%; /* inset left and right */
  padding: 2em 3em;
  background-color: #fff;
  overflow: auto;
}

body.no-scroll {
  overflow: hidden;
}
```

Here is the JS. You grab the buttons that open and close the modal, and make them add and remove a class that changes the modal display. You can also add a class to the body to prevent scrolling:
```js
var button = document.getElementById('open');
var close = document.getElementById('close');
var modal = document.getElementById('modal');

button.addEventListener('click', function (event) {
    modal.classList.add('is-open');
    document.body.classList.add('no-scroll');
});

close.addEventListener('click', function (event) {
    modal.classList.remove('is-open');
    document.body.classList.remove('no-scroll');
});
```


#### Position elements whose contents determine size 

You can specify the sides that you need to place the element and a width, and let the contents determine the element size:

```css
.fixed-position {
  position: fixed;
  top: 1em;
  right: 1em;
  width: 20%;
}
```
This is helpful for fixed navs or fixed side-navs.

> To prevent other content from overflowing behind a fixed nav, add margin to the content's container.


#### Use cases

- navigation bars
- floating chat buttons

### Sticky

Behave like `static` elements until you scroll past them, then they behave like `fixed` elements and stay at the offset that you set for them. They are not taken out of the document flow.

Sticky elements always remain within the bounds of their parent elements.

#### Use cases

- section headings

## CSS Functions

[Practical use cases](https://moderncss.dev/practical-uses-of-css-math-functions-calc-clamp-min-max/)
[MDN complete list of functions](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Functions)
[min(), max(), clamp() info](https://web.dev/articles/min-max-clamp)

### calc()

You can nest `calc()`:

```css
:root {
  --header: 3rem;
  --footer: 40px;
  --main: calc(100vh - calc(var(--header) + var(--footer))); /*
  --main: calc(100vh - (3rem + 40px))
  */
}
```

### min()

Applies the smaller value to the element:

```css
img {
  width: min(150px, 100%);
}
```

If there is 150px available to the image, then it uses it. Otherwise, the image is set to 100% of its parent element.

You can also see this as "at maximum, the image is 150px wide"

### max()

Applies the larger value to the element:

```css
img {
  width: min(100px, 4em, 50%);
}
```

If there is enough viewport size to apply 4em or 50%, then it is applied. If there isn't, the image is 100px wide.

### clamp()

Makes elements fluid and responsive. Accepts 3 values:

- smallest val
- ideal val
- largest val

```css
h1 {
  font-size: clamp(320px, 80vw, 60rem); /*
  font-size: clamp(smallest, ideal, largest); */
}
```

## Custom properties

CSS variables that you can use within the context or scope that they are defined.

### Fallback values

```css
.fallback {
  --color-text: white;

  background-color: var(--undeclared-property, black);
  color: var(--undeclared-again, var(--color-text, yellow));
}
```

### Scope

The scope of a custom property includes the selector that the custom property was delcared in, and any descendants of that selector.

To make custom properties globally available, put them on the `:root` selector:

```css
:root {
  --custom-prop: value;
}
```

### Themes with custom properties

You can add dark and light themes with custom properties. First, create the theme colors with class styles on the `:root` element:

```css
:root.dark {
  --border-btn: 1px solid rgb(220, 220, 220);
  --color-base-bg: rgb(18, 18, 18);
  --color-base-text: rgb(240, 240, 240);
  --color-btn-bg: rgb(36, 36, 36);
}

:root.light {
  --border-btn: 1px solid rgb(36, 36, 36);
  --color-base-bg: rgb(240, 240, 240);
  --color-base-text: rgb(18, 18, 18);
  --color-btn-bg: rgb(220, 220, 220);
}

/* apply custom props to elements below */
```

Then, use JS to grab the root element (`documentElement`), and toggle the theme:

```js
let setTheme = () => {
  const root = document.documentElement;
  const newTheme = root.className === "dark" ? "light" : "dark";
  root.className = newTheme;

  document.querySelector(".theme-name").textContent = newTheme;
};

document.querySelector(".theme-toggle").addEventListener("click", setTheme);
```

#### prefers-color-scheme media query

You can also set the theme according to the user agent settings. If you have the user agent set to dark, then this media query sets the styles to the dark theme. The media query accepts only `light` and `dark` as arguments:

```css
:root {
  --border-btn: 1px solid rgb(36, 36, 36);
  --color-base-bg: rgb(240, 240, 240);
  --color-base-text: rgb(18, 18, 18);
  --color-btn-bg: rgb(220, 220, 220);
  --theme-name: "light";
}

@media (prefers-color-scheme: dark) {
  :root {
    --border-btn: 1px solid rgb(220, 220, 220);
    --color-base-bg: rgb(18, 18, 18);
    --color-base-text: rgb(240, 240, 240);
    --color-btn-bg: rgb(36, 36, 36);
    --theme-name: "dark";
  }
}

/* element styles below */
```

## Browser compatibility

[Can I use](https://caniuse.com/)

- Major web browsers pretty much share the same compatibility features.
- On IoS, Safari is the only supported browser. When you install Chrome or Firefox, they use the Safari rendering engine.

  For more info, [read this](https://adactio.com/journal/17428).