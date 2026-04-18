---
title: "Managing Focus"
linkTitle: "Focus"
weight: 70
# description:
---

Keyboard navigation is not just for users with motor disabilities. Power users, developers, and anyone who prefers not to reach for a mouse all rely on it. If a user cannot see where focus is on the page, they cannot navigate your site with a keyboard. Focus management is the foundation of keyboard accessibility.

## Tips

- Do not create custom modal elements if you can avoid it. The native `<dialog>` element manages focus automatically: it traps focus inside the dialog while it is open and returns focus to the triggering element when it closes. A custom modal requires you to implement all of that behavior yourself.

## Styles

Adding focus styles is one of the most important things you can do for keyboard accessibility. You need visible styles on elements so users can track which element is currently active as they tab through the page. Default browser styles are sufficient but inconsistent across browsers. Define your own to ensure consistency.

> Never remove an element's outline.

### :focus

Applies when the user focuses an element with the keyboard, mouse, or any other input method.

### :focus-visible

From MDN:

The `:focus-visible` pseudo-class applies while an element matches the `:focus` pseudo-class and the user agent determines via heuristics that the focus should be made evident on the element.

In practice, this means `:focus-visible` applies for keyboard users but not for mouse clicks. When a mouse user clicks a button, they can already see where they clicked. When a keyboard user tabs to a button, they need a visible ring to track their position. The heuristics include the following:

- The user is interacting with the page via keyboard or another non-pointing device
- The element supports keyboard input, such as an `<input>` or `<textarea>`
- Focus moved to a new element via JavaScript and the previous element showed a visible focus indicator
- The browser has a user preference to always show focus indicators

Use `:focus-visible` for visible focus rings so mouse users do not see unnecessary outlines while keyboard users always do:

```css
/* Apply focus ring only for keyboard users */
.button:focus-visible {
  outline: 0.25em dashed black;
}

/* Mouse users get a subtler shadow instead */
.button:focus:not(:focus-visible) {
  outline: none;
  box-shadow: 1px 1px 5px rgba(1, 1, 0, 0.7);
}
```

### :focus-within

Applies styles to any element whose descendants match `:focus`. If you have an `<input>` nested in a `<label>`, you can change the label's styles when the input receives focus by applying `:focus-within` to the label.

### Examples

Make sure your focus styles meet W3C color contrast guidelines. For *forced-colors mode* (a Windows accessibility setting that overrides all colors), combine `box-shadow` with a transparent outline. The transparent outline becomes visible in forced-colors mode even though it is invisible normally:

```css
/* Global focus-visible styles */
:focus-visible {
  outline: 0.25em solid;
  outline-offset: 0.25em;
}

/* Focus audio/video when a child element is focused */
:is(video, audio):focus-within {
    box-shadow: 0 0 10px 3px rgb(0 0 0 / 0.2);
}
```

## Make elements focusable

Most interactive elements are focusable by default. You need to explicitly manage focus in these situations:

- You need to move focus programmatically, but the target element is not natively focusable
- You want to add focus to a scrollable container so keyboard users can scroll it
- You are building a custom interactive widget

### tabindex

`tabindex` lets you control which elements are focusable and in what order. Apply `tabindex="0"` to add an element to the natural tab order determined by the HTML structure.

Apply `tabindex="-1"` to make an element focusable via the JavaScript `focus()` method without adding it to the keyboard tab order. This is the correct approach for elements that should receive focus programmatically, such as a modal dialog that should receive focus when it opens, but that users should not tab to directly.

> Do not assign a `tabindex` value higher than `0`. A positive value overrides the natural tab order and gives that element priority over others. This disrupts navigation in ways users do not expect.


## Skip content

Pages with long navigation menus or repeated header content force keyboard users to tab through many interactive elements to reach the main content. On a page with twelve navigation links, a keyboard user must press Tab twelve times on every page load before reaching the first paragraph.

A *skip link* solves this by jumping focus directly to the main content. The link should only be visible when it has focus, so it does not clutter the visual design for mouse users:

```html
<a href="#content" class="skip-link">Skip to content</a>
...
<p id="content">Here is the main content!</p>
```

The CSS hides the skip link when it is not focused or active:

```css
/* Visible styles */
.skip-link {
  background-color: #fff;
  position: absolute;
  padding: 0.2em;
  display: block;
}

/* Hide skip link */
.skip-link:not(:focus):not(:active) {
  clip-path: inset(50%);
  height: 1px;
  width: 1px;
  overflow: hidden;
  white-space: nowrap;
}
```
