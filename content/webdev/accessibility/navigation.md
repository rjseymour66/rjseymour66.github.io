---
title: "Navigation"
# linkTitle: "Focus"
weight: 80
# description:
---


## Main nav

The main navigation of a site should follow this basic outline. The `aria-label="Main"` attribute distinguishes this navigation landmark from other `<nav>` elements on the page (such as breadcrumbs or a footer navigation), so screen reader users can jump directly to the right one:

```html
<header>
    <a href="#main-nav" class="skip-link">Skip to navigation</a>
    <a href="#">Website logo</a>
    <a href="#">another link</a>
    <nav id="main-nav" aria-label="Main">
        <ul>
            <li><a href="#">Home</a></li>
            <li><a href="#" aria-current="page">Products</a></li>
            <li><a href="#">Team</a></li>
            <li><a href="#">Contact</a></li>
        </ul>
    </nav>
</header>
```

- You do not need to add `role="list"` to the navigation `<ul>` because the `<nav>` element preserves the native semantic information for list children, regardless of CSS styles applied.
- The skip link lets users jump past interactive elements between the skip link and the target. Apply one if there are more than two interactive elements before the main navigation.


## aria-current

`aria-current` tells a screen reader which item in a set of related elements is the active one. It is a better alternative to `class="active"` because class names are invisible to assistive technologies. You can apply both if you need the class for styling.

For example, on a product site with a four-item navigation, a screen reader user who cannot see the visual "active" highlight needs `aria-current="page"` to know they are viewing the Products page. Without it, they must read the URL or page title to orient themselves.

`aria-current` accepts the following values, each described in the [MDN docs](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-current#values):

- `page`
- `step`
- `location`
- `date`
- `time`
- `true`
- `false`

Apply the `aria-current` attribute selector to style the current page in CSS:

```css
a {
  border-block-end: 3px solid var(--border-color, transparent);
  color: var(--text);
  display: inline-block;
  font-size: 1.4rem;
  padding: 0.5em;
  text-decoration: none;
}

a:focus-visible {
  outline: 0.25em solid currentColor;
  outline-offset: 0.125em;
}

[aria-current="page"] {
  --border-color: var(--highlight);
  --text: var(var(--highlight));
}
```

## Lists

The `<ul>` and `<ol>` elements have an implicit role of `list`. When a screen reader encounters a list, it announces how many items are in it, and announces each item's position as the user navigates (for example, "item 2 of 5").

Because all lists have an implicit `list` role, you might think you never need to add it explicitly. However, if you remove the list bullets or change other visual indicators that identify the element as a list, some browsers (notably Safari with VoiceOver) remove the semantic role. In that case, add `role="list"` explicitly.

Alternatively, set `list-style-type: "";` to an empty string. This keeps the semantic information intact without displaying bullets.

> If you wrap the list in a `<nav>` element, you do not need to add `role="list"`.


## Examples

### Hamburger for narrow viewports

> **Note:** This example is a work in progress. The JavaScript toggle has a known bug (see below) and the full implementation requires additional testing.

This pattern involves the following steps:

1. Create styles for narrow viewports.
2. Hide the nav list in a sidebar.
3. Let users toggle its visibility with a button.

#### HTML

The button lives inside a `<template>` element so JavaScript can inject it only in environments that support scripting. This is a progressive enhancement approach: the navigation list is always visible, and the hamburger button only appears when JavaScript runs.

```html
<header>
      <nav aria-label="Main" id="main-nav">
        <ul id="main-nav-list">
          <li><a href="#">Home</a></li>
          <li><a href="#" aria-current="page">Products</a></li>
          <li><a href="#">Team</a></li>
          <li><a href="#">Contact</a></li>
        </ul>

        <template id="burger-template">
          <button
            type="button"
            aria-expanded="false"
            aria-label="Menu"
            aria-controls="main-nav-list"
          >
            <svg viewBox="-5 0 10 8" width="40" aria-hidden="true">
              <line
                y2="6.5"
                stroke="#000"
                stroke-width="10"
                stroke-dasharray="1.5 1"
              />
            </svg>
          </button>
        </template>
      </nav>
    </header>
    <main>
      ...
    </main>
```

#### CSS

CSS custom properties drive the responsive behavior. The narrow viewport defaults set the navigation to a fixed sidebar. The `@media` query at `48em` switches to a horizontal row by resetting the custom properties:

```css
nav {
  position: var(--nav-position), fixed;
  inset-block-start: 1rem;
  inset-block-end: 1rem;
  z-index: 1;
}

ul {
  display: flex;
  flex-direction: var(--nav-list-layout, column);
  flex-wrap: wrap;
  gap: 1rem;
  list-style: none;
  margin: 0;
  padding: 0;
}

nav ul {
  background: hsl(0 0% 100%);
  box-shadow: var(--nav-list-shadow, -5px 0 11px 0 hsl(0 0% 0%/0.2));
  display: flex;
  flex-direction: var(--nav-list-layout, column);
  flex-wrap: wrap;
  gap: 1rem;
  height: var(--nav-list-height, 100dvh);
  list-style: none;
  margin: 0;
  padding: var(--nav-list-padding, 2rem);
  position: var(--nav-list-position, fixed);
  inset-block-start: 0;
  inset-block-end: 0;
  width: var(--nav-list-width, min(22rem, 100vw));
  visibility: var(--nav-list-visibility, hidden);
}

[aria-expanded="true"] + ul {
  --nav-list-visibility: visible;
}

@media (min-width: 48em) {
  nav {
    --nav-position: static;
    --nav-button-display: none;
  }
  nav ul {
    --nav-list-layout: row;
    --nav-list-position: static;
    --nav-list-padding: 0;
    --nav-list-height: auto;
    --nav-list-width: 100%;
    --nav-list-shadow: none;
    --nav-list-visibility: visible;
  }
}

nav ul:first-child {
  --nav-list-layout: row;
  --nav-list-position: static;
  --nav-list-padding: 0;
  --nav-list-height: auto;
  --nav-list-width: 100%;
  --nav-list-shadow: none;
  --nav-list-visibility: visible;
}

nav button {
  all: unset;
  display: var(--nav-button-display, flex);
  position: relative;
  z-index: 1;
}

button:focus-visible {
  outline: 0.25em solid currentColor;
  outline-offset: 0.125em;
}

ul {
  visibility: var(--nav-button-display, hidden);
}
```

#### JS

The JavaScript injects the button from the template and handles two events. The `keyup` handler closes the menu when the user presses `Escape` and returns focus to the button. The `click` handler toggles `aria-expanded`.

Note: the comparison `button.getAttribute('aria-expanded') === 'true'` is required because `getAttribute` always returns a string, not a boolean. Comparing against the boolean `true` would always evaluate to `false` and the menu would never close.

```js
const nav = document.querySelector('nav');
const list = nav.querySelector('ul');
const burgerTemplate = document.querySelector('#burger-template').content;
const burgerClone = burgerTemplate.cloneNode(true);
const button = burgerClone.querySelector('button');


nav.addEventListener('keyup', e => {
    if (e.code === 'Escape') {
        button.setAttribute('aria-expanded', 'false');
        button.focus();
    }
});

button.addEventListener('click', e => {
    const isOpen = button.getAttribute('aria-expanded') === 'true';
    button.setAttribute('aria-expanded', !isOpen);
});

nav.insertBefore(burgerClone, list);
```
