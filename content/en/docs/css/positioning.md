---
title: "Positioning"
linkTitle: "Positioning"
weight: 5
description: >
  Notes about basic Positioning.
---

Initially, every element has static positioning:

```css
position: static
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

## Absolute positioning

The location of absolute positioned elements is based on the closest positioned ancestor element. This ancestor element is its _containing block_.

Absolute positioning is used frequently with JS to build menus, tooltips, info boxes, etc.

### Absolute example

In this example, we will position a 'close' button for the modal described in the Fixed section. The 'close' icon looks like this:

```css
.modal-close {
    position: absolute;
    top: 0.3em;
    right: 0.3em;
    padding: 0.3em;
    cursor: pointer;
}
```

When the browser reads this `position: absolute`, it searches up the DOM heirarchy until it finds a positioned element and uses that as the positioning reference, or _containing block_.

## Relative positioning

Applying relative positioning does not impact the elements around the positioned elements. It moves the element relative to its original position in the document flow.

The main use for relative positioning is to create a containing block for an absolutely positioned element.

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
...
.dropdown {
    display: inline-block;
    position: relative;
}
...

.dropdown-menu {
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


## Stacking contexts with z-index

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
--z-one:    100;
--z-two:    200;
--z-three:  300;
--z-four:   400;
```