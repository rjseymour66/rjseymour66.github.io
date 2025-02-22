---
title: "Positioning and stacking context"
linkTitle: "Positioning"
weight: 80
# description:
---

| Positioning | Use cases                                     | Containing block                     | Description                      |
| :---------- | :-------------------------------------------- | :----------------------------------- | :------------------------------- |
| Fixed       | modal, navigation bars, floating chat buttons | Viewport                             | Positioned relative to viewport. |
| Absolute    | popup menus, tooltips, "info" boxes           | Closest-positioned ancestor element. |                                  |
| Relative    | dropdown menus                                | element with `position: absolute;`   |                                  |
| Sticky      | section headings                              |                                      |                                  |

Initially, every element has static positioning:

```css
position: static;
```

Anytime you change the `position` value, the element becomes _positioned_. Positioned elements are removed from the document flow so that you can place them anywhere on the screen.

## Fixed positioning

Positions elements arbitrarily within the viewport (the _containing block_) using the following properties:

```
top:
bottom:
left:
right:
```

Fixed elements are removed from the document flow -- static elements display as if the fixed elements do not exist.

For static side navs, add a margin to the content to make sure that it doesn't flow behind it.

### Fixed example

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

## Absolute positioning

The location of absolute positioned elements is based on the closest positioned ancestor element. This ancestor element is its _containing block_.

Absolute positioning is used frequently with JS to build menus, tooltips, info boxes, etc.

### Absolute

Absolute positioned elements are positioned based on their closest-positioned ancestor element. That ancestor element is called the _containing block_. The `inset` property values place the absolute positioned element within that containing block.

> Usually, an `absolute` element's containing block is set to `relative`.

Position something at an exact point on the screen without affecting any elements around it. It removes it from the normal document flow and positions it relative to an ancestor element.

#### Use cases

- modals
- image with a caption on top of it
- icons on top of other elements

### Absolute example

In this example, we will position a 'close' button for the modal described in the Fixed section. The 'close' icon looks like this:

```css
/* positions the clickable area in the top-right
of the modal. Indents it 10em to hide the <button>Close</button>
while allowing a screenreader to read it*/
.modal-close {
  position: absolute;

  top: 0.3rem;
  right: 0.3rem;

  padding: 1.75rem;
  border: none;
  height: 1rem;
  width: 1rem;

  text-indent: 10em;
  overflow: hidden;
  background-color: transparent;
}

/* Adds the X icon w black circle bg*/
.modal-close::after {
  position: absolute;
  line-height: 0.5;
  top: 0.5rem;
  right: 0.5rem;
  text-indent: 0;
  font-size: 2.5rem;
  cursor: pointer;

  background-color: lightgray;
  border-radius: 50%;
  padding: 0.5rem;

  content: "\00d7";
}
```

When the browser reads this `position: absolute`, it searches up the DOM heirarchy until it finds a positioned element and uses that as the positioning reference, or _containing block_.

## Relative positioning

Applying relative positioning does not impact the elements around the positioned elements. It moves the element relative to its original position in the document flow.

The main use for relative positioning is to create a containing block for an absolutely positioned element.

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

### Relative example

In the following example:

- `dropdown` is the relative-positioned containing block
- `dropdown-menu` is the absolute-positioned element that is hidden initially

```html
<div class="container">
  <nav>
    <div class="dropdown">
      <div class="dropdown-label">Main Menu</div>
      <div class="dropdown-menu">
        <ul class="submenu">
          <li><a href="#">Home</a></li>
          <li><a href="#">Coffees</a></li>
          <li><a href="#">Brewers</a></li>
          <li><a href="#">Specials</a></li>
          <li><a href="#">About us</a></li>
        </ul>
      </div>
    </div>
  </nav>
</div>
```

The following CSS creates the positioned context:

```css
... .dropdown {
  display: inline-block;
  position: relative;
}
... .dropdown-menu {
  display: none;
  position: absolute;
  left: 0;
  top: 2.1em;
  min-width: 100%;
  background-color: #eee;
}

/* when you hover on dropdown, the menu displays. Toggle with JS */
.dropdown:hover > .dropdown-menu {
  display: block;
}
```

## Sticky positioning

This is a hybrid between relative and fixed positioning. The element scrolls until it reaches a specified spot in the viewport and then it 'sticks' in place.

You do not need a containing block for `sticky` positioning.

Behave like `static` elements until you scroll past them, then they behave like `fixed` elements and stay at the offset that you set for them. They are not taken out of the document flow.

Sticky elements always remain within the bounds of their parent elements.

#### Use cases

- section headings

## z-index

When you remove an element from the document flow, you become responsible for all the things that the document flow normally does for you. This includes making sure that the positioned element does not:

- Overflow outside the browser viewport and then hidden from the user
- Cover important content

When the browser parses HTML, it creates a _render tree_ that represents the appearance and position of each element. The browser paints elements in the order in which they are listed in the HTML. The position determines the order that the browser _paints_ the elements:

- First, it paints non-positioned elements.
- Next, it paints positioned elements using `z-index`.

Elements with a `z-index` establish a _stacking context_. A stacking context is a n element or a group of elements that are painted together with a browser. Elements with a higher `z-index` are positioned in front of elements with a lower `z-index`. Elements with a negative `z-index` are positioned behind static elements. No element outside the stacking context can be positioned between elements in the stacking context.

#### Best practice

Put your `z-index` values in variables:

```css
--z-loading-indicator: 100;
--z-nav-menu: 200;
--z-dropdown-menu: 300;
--z-modal-backdrop: 400;
--z-modal-body: 410;
```

<!-- **************************************************************************** -->

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
- positioned elements witha z-index of `auto` and their children
- positioned elements with a positive z-index
