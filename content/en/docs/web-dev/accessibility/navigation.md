---
title: "Navigation"
# linkTitle: "Focus"
weight: 80
# description:
---


## Main nav

The main navigation of a site should follow this basic outline:

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
- You don't have to add `role="list"` to the navigation ul because the `<nav>` element maintains any native semantic information for list children, regardless of the styles.
- The skip link lets users skip over the interactive elements between the skip link and the target. Here, the target is the main navigation. Use this if there are more than two interactive elements before the main navigation.


## aria-current

Indicates to the screen reader which page (or element in a set) the user is on. This is a good alternative to `class="active"`, but you can use both if you'd like.

`aria-current` accepts the following values, each described in detail in the [MDN](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-current#values) docs:
- `page`
- `step`
- `location`
- `date`
- `time`
- `true`
- `false`



Use the `aria-current` selector to style the current page:

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

The `<list>` element has an implicit role of `list`. When a screen reader encounters a `list`, it announces how many items are in the list.

Because all lists have an implicit role of `list`, you might think that you don't have to explicitly add the `list` role. However, if you remove the list discs or change any other visual indicators that identify the element and its children as a list, you should include `role="list"`.

Alternately, you could set `list-style-type: "";`. This doesn't remove the semantic information from the list.

> If you wrap the list in a `<nav>` element, you do not have to add the `list` role.


## Examples

### Hamburger for narrow viewports

This involves the following steps:
1. Create styles for narrow viewports
2. Hide the nav list in a sidebar
3. Let users toggle its visibility

This example did not work at all, but here is are the styles to figure out later:

#### HTML

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
    <main role="Main">
      ...
    </main>
```

#### CSS

```css
nav {
  position: var(--nav-position), fixed;
  inset-block-start: 1rem;
  inset-block-end: 1rem;
  z-index: 1; /* added */
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

```js
const nav = document.querySelector('nav');
const list = nav.querySelector('ul');
const burgerTemplate = document.querySelector('#burger-template').content;
const burgerClone = burgerTemplate.cloneNode(true);
const button = burgerClone.querySelector('button');


nav.addEventListener('keyup', e => {
    if (e.code === 'Escape') {
        button.setAttribute('aria-expanded', false);
        button.focus();
    }
});

button.addEventListener('click', e => {
    const isOpen = button.getAttribute('aria-expanded') === true;
    button.setAttribute('aria-expanded', !isOpen);
});

nav.insertBefore(burgerClone, list);
```
