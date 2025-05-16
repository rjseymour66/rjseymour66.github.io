---
title: "Menus and menu buttons"
linkTitle: "Menus"
weight: 50
# description:
---

## Navigation menus

When creating navigation menus with submenus, be careful with these ARIA attributes. Make sure you use `aria-expanded="true|false"` on the top level menu item. Use this attribute on click, not on focus.

> These ARIA attributes are for application menus and not navs:
> - `aria-haspopup="true"`: activated on click and only reveal a secret menu
> - `role="menu"`
> - `role="menuitem"`


For touch screens, top-level destination pages should have a table of contents rather than a submenu.


## Tables of content

The TOC should use a `<nav>` element, list, and group labeling mechanism. The screen reader announces this as "products navigation":

```html
<nav aria-lablledby="sections-heading">
    <h2 id="sections-heading">Products</h2>
        <ul>
            <li><a href="/path/to/page">Page 1</a></li>
            <li><a href="/path/to/page">Page 2</a></li>
            <li><a href="/path/to/page">Page 3</a></li>
        </ul>
</nav>
```

## Hamburger menu

The hamburger menu should be a button that is nested within the `<nav>` element and hidden with HTML and CSS:

```html
<nav id="navigation">
    <button aria-expanded="false">Menu</button>
    <ul hidden>
        <li><a href="/">Home</a></li>
        <li><a href="/about">About</a></li>
        <li><a href="/shop">Shop</a></li>
        <li><a href="/contact">Contact</a></li>
    </ul>
</nav>
```

Hide the button with CSS to hide it from SRs and remove it from the focus order:

```css
[hidden] {
    display: none;
}
```

Here is the JS to display the menu. Do not use an arrow function so `this` is the current context:

```js
const navButton = document.querySelector('#navigation button');

navButton.addEventListener('click', function () {
    let isExpanded = this.getAttribute('aria-expanded') === 'true';
    this.setAttribute('aria-expanded', !isExpanded);
    let menu = this.nextElementSibling;
    menu.hidden = !menu.hidden;
});
```

## aria-controls

`aria-controls` has poor support for SRs, but you should include it because some users expect it. Its purpose is to help SR users navigate from a controlling element to a controlled element:

```html
<nav id="navigation">
    <button aria-expanded="false" aria-controls="menu-list">Menu</button>
    <ul id="menu-list" hidden="hidden">
        <li><a href="/">Home</a></li>
        <li><a href="/about">About</a></li>
        <li><a href="/shop">Shop</a></li>
        <li><a href="/contact">Contact</a></li>
    </ul>
</nav>
```

## True menus (app menus)

Here is a [GitHub link](https://github.com/Heydon/inclusive-menu-button) that describes it better than this PDF.

These ARIA attributes are for choosing options in an application, such as the File, Edit, View, etc menus in MS Word. If the app you're building doesn't have much in common with a standard application like this, you likely won't need the attributes in this section.

```html
<button aria-haspopup="true" aria-expanded="false">
Difficulty
    <span aria-hidden="true">&#x25be;</span>
</button>
<div role="menu">
    <button role="menuitem" tabindex="-1">Easy</button>
    <button role="menuitem" tabindex="-1">Medium</button>
    <button role="menuitem" tabindex="-1">Incredibly Hard</button>
</div>
```

- `aria-haspopup` indicates that the button reveals a menu. It warns the user that when they press the button they are moved to the popup menu
- `aria-hidden` in the span makes sure that the SR does not announce the unicode for a downward-pointing arrow.
- `aria-expanded` tells users whether the menu is expanded or collapsed.
- add `menu` to the top-level element in the menu, and `menuitem` to each item in the menu.
- `tabindex="-1"` removes the Tab focus but lets us control the focus with JS.

### Keyboard and focus

