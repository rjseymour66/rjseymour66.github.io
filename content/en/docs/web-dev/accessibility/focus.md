---
title: "Managing Focus"
linkTitle: "Focus"
weight: 70
# description:
---

## Tips

- Don't create custom modal elements, or you have to manage the focus. The native `<dialog>` element manages focus automatically.

## Styles

Adding focus styles is one of the most important things you can do for keyboard accessibility. You need to add styles to elements so users can tab to elements on the page. Default user agent styles are sufficient, but not great.

> Don't remove an element's outline!

### :focus

Applies when the user focuses an element with the keyboard, mouse, or any other input.

### :focus-visible

From `MDN`:

The `:focus-visible` pseudo-class applies while an element matches the :focus pseudo-class and the UA (User Agent) determines via heuristics that the focus should be made evident on the element.

The heurisitics include the following:
- user interacts with the page using a keyboard or another nonpointing device
- the element supports keyboard input, such as an input or textarea element
- if you move focus to a new element with JS, and the previous element showed focus, the new element shows focus too
- if there is a user preference in the browser

### :focus-within

Applies styles to any element whose descendants match `:focus` conditions. So if you have an input nested in a label, and you select the input, you can change the label styles by using `:focus-within` on it.

### Examples

Make sure your focus styles meet W3C color contrast guidelines. For forced contrast mode, combine `box-shadow` and transparent outlines:

```css
/* global focus-visible styles */
:focus-visible {
  outline: 0.25em solid;
  outline-offset: 0.25em;
}

/* focus audio/video when child element is focused */
:is(video, audio):focus-within {
    box-shadow: 0 0 10px 3px rgb(0 0 0 / 0.2);
}
```

These declarations provide different styles for keyboard and non-keyboard users:

```css
/* keyboard */
.button:focus-visible {
  outline: 0.25em dashed black;
}

/* non-keyboard */
.button:focus:not(:focus-visible) {
  outline: none;
  box-shadow: 1px 1px 5px rgba(1, 1, 0, 0.7);
}
```

## Make elements focusable

Most interactive elements are focusable by default, but here are some circumstances that you need to add focus:
- you need to move focus, but the element or its children are not focusable
- want to add focus to scrollable areas
- you use a custom element

### tabindex

`tabindex` lets you make an element focusable. Use `tabindex=0` to add the element to its natural sequence in the tab order, which is determined by the HTML structure.

Use `tabindex="-1"` to make an element focusable with JS `focus()` but NOT keyboard accessible, or remove an element from the tab order but still access with JS.

> Do not assign a tabindex value higher than `0`. This alters the natural sequence of the tab order and the element gets preference over others. This can confuse navigation.


## Skip content

Some parts of a web page contain multiple interactive elements, and tabbing through them all can be cumbersome. For example, a navigation menu. You can add "skip links" to these areas that link to the next section in the content so users don't have to tab through all the interactive elements.

Skip links should only be visible when it is focus:

```html
<a href="#content" class="skip-link">Skip to content</a>
...
<p id="content">Here is the main content!</p>
```

The CSS hides the skip link when it is not active or in focus:

```css
/* visible styles */
.skip-link {
  background-color: #fff;
  position: absolute;
  padding: 0.2em;
  display: block;
}

/* hide skip link */
.skip-link:not(:focus):not(:active) {
  clip-path: inset(50%);
  height: 1px;
  width: 1px;
  overflow: hidden;
  white-space: nowrap;
}
```