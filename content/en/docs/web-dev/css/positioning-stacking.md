---
title: "Positioning and stacking context"
linkTitle: "xPositioning"
weight: 90
# description:
---

By default, all elements are `position: static;`. Anytime you change the `position` value, the element becomes _positioned_. Positioned elements are removed from the document flow so that you can place them anywhere on the screen.

Initially, every element has static positioning:

```css
position: static;
```

## Cheatsheet

| Positioning | Containing Block                                                                 | Use cases                                         |
| :---------- | :------------------------------------------------------------------------------- | :------------------------------------------------ |
| Fixed       | Viewport                                                                         | modal<br>navigation bars<br>floating chat buttons |
| Absolute    | Closest-positioned ancestor element<br>(usually a `relative` positioned element) | popup menus<br>tooltips<br>"info" boxes           |
| Relative    | element with `position: absolute;`                                               | dropdown menus                                    |
| Sticky      |                                                                                  | section headings                                  |

## Fixed

> Containing block: viewport

Positions elements arbitrarily within the viewport (the _containing block_). Fixed elements stay in the same place as the user scrolls.

Removed from the normal flow of the document and positioned relative to the viewport (the viewport is the _containing block_) with `top`, `right`, `bottom`, and `left`. These settings also size the element. If you set `left` and `right` to `5em`, then the width of the fixed element is 10em less than the viewport width.

`inset` is the shorthand and it accepts values in the same format as the `padding` or `margin` shorthand:

```css
.class {
  inset: <top> <right> <bottom> <left>;
}
```

To make the element take up the entire viewport, use `inset: 0;`. This is helpful to darken the background for a smaller modal element.

Fixed elements are removed from the document flow -- static elements display as if the fixed elements do not exist.

For static side navs, add a margin to the content to make sure that it doesn't flow behind it.

### Modal example

This is for demo purposes only. Instead, you would use a `<dialog>` element.

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
var button = document.getElementById("open");
var close = document.getElementById("close");
var modal = document.getElementById("modal");

button.addEventListener("click", function (event) {
  modal.classList.add("is-open");
  document.body.classList.add("no-scroll");
});

close.addEventListener("click", function (event) {
  modal.classList.remove("is-open");
  document.body.classList.remove("no-scroll");
});
```

### Content determines size

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

## Absolute

Absolute positioned elements are positioned based on their closest-positioned ancestor element. That ancestor element is called the _containing block_. The browser will look up the DOM hierarchy until a position ancestor is found. If no ancestor is found, it uses the _initial containing block_, which is an area equal to the viewport.

The `inset` property values place the absolute positioned element within that containing block.

> Usually, an `absolute` element's containing block is set to `relative`.

Position something at an exact point on the screen without affecting any elements around it. It removes it from the normal document flow and positions it relative to an ancestor element.

#### Use cases

- popup menus
- tooltips
- info boxes

### Modal close button

In this example, we will position a 'close' button for the modal described in the Fixed section. The `.modal-close` class hides the 'close' text off the screen with the `overflow` property, and the `::after` psuedo class absolutely positions the `X` icon:

```css
/* positions the clickable area in the top-right
of the modal. Indents it 10em to hide the <button>Close</button>
while allowing a screenreader to read it*/
.modal-close {
  position: absolute;
  top: 0.3em;
  right: 0.3em;
  padding: 0.3em;
  border: 0;
  font-size: 2em;
  height: 1em;
  width: 1em;
  text-indent: 10em;
  overflow: hidden;
  background-color: transparent;
}

/* Adds the X icon */
.modal-close::after {
  position: absolute;
  line-height: 0.5;
  top: 0.2em;
  left: 0.1em;
  text-indent: 0;
  content: "\00D7";
}
```

When the browser reads this `position: absolute`, it searches up the DOM heirarchy until it finds a positioned element and uses that as the positioning reference, or _containing block_.

## Relative

The main use for relative positioning is to establish a containing block for an absolutely positioned element.

Applying relative positioning does not impact the elements around the positioned elements. It moves the element relative to its original position in the document flow.

You cannot size the element with the `inset` property---you can only move it in relation to its original location. For example, adding `top: 1em;` moves the element down 1 em. If you apply both `top` and `bottom`, then `bottom` is ignored. If you apply both `left` and `right`, then right is ignored.

#### Use cases

[Custom select dropdown](https://www.webaxe.org/accessible-custom-select-dropdowns/) menus.

### Dropdown menu

A dropdown consists of a containing div that is positioned `relative`. The following elements are within this container:

- button that you click to open the menu
- div that contains a ul. Set this div to `display: none;`. When it is visible, set the display to `block`. This div needs to display exactly below the button that opens the menu, so compute the side of the button element and use apply that value to this div's `top` property.

In the following example:

- `dropdown` is the relative-positioned containing block
- `dropdown-menu` is the absolute-positioned element that is hidden initially

```html
<main class="container">
  <nav>
    <div class="dropdown" id="dropdown">
      <button class="dropdown-toggle" id="dropdown-toggle">Main Menu</button>
      <div class="dropdown-menu">
        <ul class="submenu">
          <li><a href="/">Home</a></li>
          <li><a href="/coffees">Coffees</a></li>
          <li><a href="/brewers">Brewers</a></li>
          <li><a href="/specials">Specials</a></li>
          <li><a href="/about">About us</a></li>
        </ul>
      </div>
    </div>
  </nav>
</main>
```

The following CSS creates the positioned context:

```css
.dropdown {
  display: inline-block;
  position: relative;
}

.dropdown-toggle {
  padding: 0.5em 1.5em;
  border: 1px solid #ccc;
  background-color: #eee;
  border-radius: 0;
}

/* div that wraps the ul */
.dropdown-menu {
  display: none;
  position: absolute;
  left: 0;
  top: 2.1em;
  inline-size: max-content;
  min-inline-size: 100%;
  background-color: #eee;
}

/* toggle class with js */
.dropdown.is-open .dropdown-menu {
  display: block;
}

/* menu list items */
.submenu {
  padding-inline-start: 0;
  margin: 0;
  list-style-type: none;
  border: 1px solid #999;
}

.submenu > li + li {
  border-top: 1px solid #999;
}

/* menu links - make block for larger clickable area */
.submenu > li > a {
  display: block;
  padding: 0.5em 1.5em;
  background-color: #eee;
  color: #369;
  text-decoration: none;
}

.submenu > li > a:hover {
  background-color: #fff;
}
```

Javascript:

```js
const dropdownToggle = document.querySelector("#dropdown-toggle");
const dropdown = document.querySelector("#dropdown");

dropdownToggle.addEventListener("click", (e) => {
  dropdown.classList.toggle("is-open");
});
```

#### Create CSS triangle

[CSS Tricks: Shapes of CSS](https://css-tricks.com/the-shapes-of-css/)

You can add CSS to look like a caret that rotates when clicked. Use this if your site did not import icon packets already, and you want to reduce requests:

```css
.dropdown-toggle {
  padding: 0.5em 2em 0.5em 1.5em; /* add extra space to the right of the text */
  border: 1px solid #ccc;
  background-color: #eee;
  border-radius: 0;
}

/* create an empty content box with thick border */
.dropdown-toggle::after {
  content: "";
  position: absolute;
  right: 1em;
  top: 0.9em;
  border: 0.3em solid;
  border-color: black transparent transparent;
}

/* reverse border color when menu is open */
.dropdown.is-open .dropdown-toggle::after {
  top: 0.6em;
  border-color: transparent transparent black;
}
```

## Sticky positioning

This is a hybrid between relative and fixed positioning. The element scrolls until it reaches a specified spot in the viewport and then it 'sticks' in place:
- They are not taken out of the document flow.
- Behaves like `static` elements until you scroll past its parent element. Then, it behaves like a `fixed` element and stays at that offset in the parent element
- You do not need a containing block for `sticky` positioning.

Sticky elements always remain within the bounds of their parent element:
- The parent element must be taller than the sticky element

An element with this class scrolls normally until it gets `1em` from the top of the viewport. Then, it 'sticks' in place and scrolls until the parent element scrolls out of the viewport:

```css
.sticky {
  position: sticky;
  top: 1em;
}
```

#### Use cases

- Section headings

## z-index

[MDN docs](https://developer.mozilla.org/en-US/docs/Web/CSS/position)

[CSS Tricks](https://css-tricks.com/absolute-relative-fixed-positioining-how-do-they-differ/)

[Fixed vs Sticky](https://www.kevinpowell.co/article/positition-fixed-vs-sticky/)

`position: static;` is the default mode for all elements. `top`, `right`, `bottom`, and `left` do not affect static elements.

When you position an element, you remove it from the document flow. When you remove an element from the document flow, you become responsible for all the things that the document flow normally does for you. This includes making sure that the positioned element does not:

- Overflow outside the browser viewport and then hidden from the user
- Cover important content

When the browser parses HTML, it creates a DOM tree and a _render tree_ that represents the appearance and position of each element. The browser paints elements in the order in which they are listed in the HTML. The position determines the order that the browser _paints_ the elements:

1. Non-positioned elements.
2. Positioned elements using `z-index`.

The browser paints positioned elements in the order that they appear in the HTML. **Relative and absolute positioning remove the element from the document flow**. This might cause issues with other positioned elements.

### Stacking context

Elements with a `z-index` establish a stacking context. A _stacking context_ is an element or a group of elements that are painted together by the browser. Each stacking conetxt has a root element, and all its descendants are part of the stacking context. It's sort of like a namespace for positioned elements - `z-index` values in one stacking context have no effect on `z-index` values in another stacking context:
- Elements with a higher `z-index` are positioned in front of elements with a lower `z-index`.
- Elements with a negative `z-index` are positioned behind static elements.
- No element outside the stacking context can be positioned between elements in the stacking context.

> Fixed and sticky positioning always create a stacking context, even though they do not require a `z-index` setting.

Elements within the stacking context are stacked in this order:

1. root element of the stacking context
2. positioned elements with negative z-index and their children
3. non-positioned elements
4. positioned elements witha z-index of `auto` and their children
5. positioned elements with a positive z-index


Ways to create a stacking context, other than `z-index`:
- `opacity` less than `1`
- `transform` property
- `filter` property

#### Best practice

Put your `z-index` values in variables:

```css
--z-loading-indicator: 100;
--z-nav-menu: 200;
--z-dropdown-menu: 300;
--z-modal-backdrop: 400;
--z-modal-body: 410;
```