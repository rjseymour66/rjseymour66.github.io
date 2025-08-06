---
title: "Responsive design"
linkTitle: "xResponsive"
weight: 100
# description:
---

Websites need to work on all devices, where you serve the same pages to all devices but they render differently based on viewport.

Use breakpoints to achieve responsive design. A _breakpoint_ is a browser width or height where the styles change to provide the best possible layout for that size.

## Three key principles

1. **Mobile-first approach**: Develop the mobile version before the desktop version. All versions must share the same HTML, so make sure you design mobile, tablet, and desktop viewports before you begin so you can properly structure the HTML.
2. **@media rule** (media queries): Write styles that apply to viewports of specified sizes. 
3. **Fluid layouts**: Containers scale differently based on viewport width

## Viewport meta tag

Tells mobile devices that your website is responsive. Otherwise, mobile devices will try to emulate a desktop browser:

```html
<head>
    ...
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    ...
  </head>
```
The `content` attribute tells the browser to do two things:
- `"width=device-width`: Use the device width as the assumed width when interpreting the CSS
- `initial-scale=1.0`: Set the zoom level to 100% when the page loads

> You can also include `user-scalable=no` to prohibit two-finger zoom on mobile devices. This is not recommended.


## Mobile first

Designing mobile first ensures that both desktop and mobile work:
- Screen space is limited
- Network is slower
- Mobile users have different set of interactive controls. For example, you can't hover
- Always make key action items large enough to easily tap with a finger

When a site works with mobile constraints, you use what is called 'progressive enhancement' to change the experience for desktop.

Mobile focuses on content, and is task-oriented. Always make sure that the important content is displayed first.


### Hamburger menu

Create a hamburger menu to display nav options when the user taps the icon:

```html
<body>
    <header id="header" class="page-header">
      <div class="title">
        <h1>Wombat Coffee Roasters</h1>
        <div class="slogan">We love coffee</div>
      </div>
    </header>

    <nav class="menu" id="main-menu">
      <button class="menu-toggle" id="toggle-menu">toggle menu</button>
      <div class="menu-dropdown">
        <ul class="nav-menu">
          <li><a href="/about.html">About</a></li>
          <li><a href="/shop.html">Shop</a></li>
          <li><a href="/menu.html">Menu</a></li>
          <li><a href="/brew.html">Brew</a></li>
        </ul>
      </div>
    </nav>
    <!-- ... -->
```

#### Styles

The `<nav>` element styles do most of the work:
* Create a containing block for the menu button
* Position the button
* Push the button text out of view, and keep it available for screen readers
* Add the hamburger icon with a pseudoclass
* Hide nav items by default
* Create a class that displays the nav items as block elements when toggled on
* Style the nav item `<a>` elements with padding to increase tappable area

```css
/* add position to create the toggle menu containing block */
.menu {
  position: relative;
}

/* position the toggle menu */
.menu-toggle {
  position: absolute;       /* move the toggle menu up into the header */
  top: -1.2em;
  right: 0.1em;

  border: 0;
  background-color: transparent;

/* force the font out of view but keep it for a11y */
  font-size: 3em;
  width: 1em;               /* constrained width and height*/
  height: 1em;
  line-height: 0.4;
  text-indent: 5em;         /* large text indent */
  white-space: nowrap; 
  overflow: hidden;         /* hide the overflow */
}

/* add the hamburger icon */
.menu-toggle::after {
  position: absolute;
  top: 0.2em;
  left: 0.2em;
  display: block;
  content: "\2261";
  text-indent: 0;
}

/* hide the list elements, by default */
.menu-dropdown {
  display: none;
  position: absolute;
  right: 0;
  left: 0;
  margin: 0;
}

/* display list elements when .is-open is toggled with JS */
.menu.is-open .menu-dropdown {
  display: block;
}

/* nav menu styles */
.nav-menu {
  margin: 0;
  padding-left: 0;
  border: 1px solid #ccc;
  list-style: none;
  background-color: #000;
  color: #fff;
}

/* apply border to all but the first list item */
.nav-menu > li + li {
  border-top: 1px solid #ccc;
}

/* make the links blocks to increase tap area */
.nav-menu > li > a {
  display: block;
  padding: 0.8em 1em;
  color: #fff;
  font-weight: normal;
}
```

#### Javascript

When the user taps the hamburger toggle menu, add a class to display the nav list:
```js
let button = document.querySelector('#toggle-menu');
button.addEventListener('click', e => {
    e.preventDefault();
    let menu = document.querySelector('#main-menu');
    menu.classList.toggle('is-open');
});
```

### Tables

Tables are a notorius pain in the ass. On mobile devices, you can display the tables in a list-type format. You have make all table elements display as block elements, and then hide the table heading off-screen (don't use `display: none` or it affects a11y):

```css
table {
  inline-size: 100%;
}

@media (max-width: 480px) {
  table,
  thead,
  tbody,
  tr,
  th,
  td {
    display: block;
  }

  thead tr {
    position: absolute;
    top: -9999px;
    left: -9999px;
  }

  tr {
    margin-block-end: 1em;
  }
}
```

## Media queries

Media queries let you create styles that apply to your HTML under specific circumstances, like screen size. Its a conditional check - if the condition in between the parentheses is met, then the rule is applied. For example, if the minimum width of the viewport is 560px, then the `h1` element that is a child of an element with the `.title` class is set to `2.5rem`:

```css
@media (min-width: 560px) {
    .title > h1 {
        font-size: 2.5rem;
    }
}
```

When possible, keep media queries short. One strategy is to change custom properties in media queries so you can override them in one spot:

```css
:root {
    --gap: 0.5rem;
}

@media (min-width: 560px) {
    :root {
        --gap: 1rem;
    }
}
```

### Adding breakpoints

> [freeCodeCamp article: The 100% correct way to do CSS breakpoints](https://www.freecodecamp.org/news/the-100-correct-way-to-do-css-breakpoints-88d6a5ba1862) says 600px, 900px, 1200px, and 1800px are the correct breakpoints.

Choose breakpoints that make sense for your design, don't get too overanalytical about it. In general, you want to start setting breakpoints for the part of your design that goes from stacked content on mobile to columns in larger viewports.

> Use liberal padding in larger viewports.

Mobile-first breakpoints are almost always `min-width`. If it is tedious to override rules, then use a `max-width` media query to apply mobile styles.

Each breakpoint should follow the mobile styles that it overrides so that the media query styles take precedence:

```css
main {
  padding: 1em;
}

@media (min-width: 560px) {
  main {
    padding: 2em 1em;
  }
}

@media (min-width: 650px) {
  main {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 1.5em;
    max-inline-size: 1400px;
    margin-inline: auto;
  }
}
```

### ems vs px

Use px for media queries, not ems/rems.

Previously, people thought it was best to use ems for media queries. Ems are based on the default font size (16px), but users can change the browser's default text size. This can change your breakpoints.

To avoid this, use px - px are a much more consistent unit across browsers.



### Clauses

Use `and` if you want to apply a style when the screen size meets two criteria:

```css
@media (min-width: 280px) and (min-width: 560px) {...}
```

Use a comma (`,`) or `or` if you want to apply a style when the viewport meets one of the criteria. The `or` keyword is a relatively new feature:

```css
@media (min-width: 280px), (min-width: 560px) {...}
@media (min-width: 280px) or (min-width: 560px) {...}
```

### Media features

A _media feature_ is the syntax that specifies which viewport you want to target. It is the `min-width` in `@media (min-width: 560px) {...}`. You have to provide explicit values to the media feature - you cannot use a custom property instead of a px value.

Other media features include:
- `(min-height: 560px)`
- `(max-height: 560px)`
- `(orientation: landscape)`
- `(orientation: portrait)`
- `(min-resolution: 2dppx)`
- `(max-resolution: 2dppx)`
- `(pointer: coarse)`
- `(pointer: fine)`


### Syntax

There are two ways to define the values passed to the media features:
- classic: `min-width: value`.
- range: `width = value`

```css
/* classic */
@media (min-width: 280px), (min-width: 560px) {...}
@media (min-width: 280px) or (min-width: 560px) {...}

/* range syntax */
@media (280px <= width < 560px) {...}
@media (width = 280px) {...}
```

Range is more intuitive because it is more familiar, easier to read/write, and you can be more explicit around rules that might conflict.


### Light and dark themes

Media queries can detect when the user has the OS set to light or dark mode:

```css
@media (prefers-color-scheme: dark) {
    /* rulesets */
}

@media (prefers-color-scheme: light) {
    /* rulesets */
}
```

### Media types

You can also pass `screen` and `print` to media queries. These media types don't require parentheses:

```css
@media print {...}
@media screen {...}
```

Use `print` for how you want the page to look if the user prints it.
- Use `display:none;` on images, navigation, footers, etc
- Change all fonts to black and remove background images

```css
@media print {
    * {
        color: black !important;
        background: none !important;
    }
}
```



## Fluid layouts

Containers should grow and shrink according to the size of the viewport:
- define containers in terms of percentages, not absolute values like px
- add padding to the sides of the content so they can grow to 100% - padding


## Responsive images

Always add this rule to your stylesheet to ensure that images don't overflow their container width:

```css
img { max-width: 100%; }
```

You should always optimize your images before you put them in a web page:
- Save for Web option in image editor
- Use a compression tool like https://tinypng.com/

Always use the appropriate size and resolution image for the screen size:

```css
.hero {
  ...
  background-image: url(images/coffee-beans-small.jpg);
}

@media (min-width: 560px) {
  .hero {
    ...
    background-image: url(/images/coffee-beans-medium.jpg);
  }
}

@media (min-width: 800px) {
  .hero {
    ...
    background-image: url(/images/coffee-beans.jpg);
  }
}
```

### srcset

If you add images with the HTML `<img>` tag, then you can apply different images per screen size with the `srcset` attribute:

```html
<img
    src="coffee-beans-small.jpg"
    alt="Coffee beans"
    srcset="
    /images/coffee-beans-small.jpg   560w,
    /images/coffee-beans-medium.jpg  800w,
    /images/coffee-beans.jpg        1280w
    "
/>
```