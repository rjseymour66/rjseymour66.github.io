---
title: "Responsive design"
linkTitle: "Responsive"
weight: 100
# description:
---

Websites need to work on all devices, where you serve the same pages to all devices but they render differently based on viewport.

Use breakpoints to achieve responsive design. A _breakpoint_ is a browser width or height where the styles change to provide the best possible layout for that size.

## Three key principles

1. **Mobile-first approach**: Develop the mobile version before the desktop version. All versions must share the same HTML, so make sure you design mobile, tablet, and desktop viewports before you begin so you can properly structure the HTML.
2. **@media rule** (media queries): Write styles that apply to viewports of specified sizes. 
3. **Fluid layouts**: Containers scale differently based on viewport width

## Mobile first

Designing mobile first ensures that both desktop and mobile work:
- Screen space is limited
- Network is slower
- Mobile users have different set of interactive controls. For example, you can't hover
- Always make key action items large enough to easily tap with a finger

When a site works with mobile constraints, you use what is called 'progressive enhancement' to change the experience for desktop.

Mobile focuses on content, and is task-oriented. Always make sure that the important content is displayed first.

### Viewport meta tag

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