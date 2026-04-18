---
title: "Toggling content visibility"
linkTitle: "Visibility"
weight: 90
# description:
---

## Hide content

[Hiding content responsibly](https://kittygiraudel.com/2021/02/17/hiding-content-responsibly/) provides a thorough overview.

Here are some general guidelines:

- Do not put `aria-hidden="true"` or `role="presentation"` on focusable, interactive elements like a button.
- If you hide content visually with `opacity`, `height`, or `transform`, also hide the content semantically with `visibility: hidden;` so screen readers do not read it.
- Do not hide elements semantically if they are referenced elsewhere in the document. For example, do not hide an input that has a visible label.

As an example, you want a skip link before the navigation to be readable by screen readers, but you do not want to display it visually until the user focuses it. The following example hides content visually while keeping it accessible to screen readers. Read about the technique in [The Anatomy of Visually-Hidden](https://www.tpgi.com/the-anatomy-of-visually-hidden/):

```css
.visually-hidden-sr {
  clip-path: inset(50%);        /* Clips all visual content (effectively hides it from view) */
  height: 1px;                  /* Makes the element tiny but still technically on the page */
  width: 1px;                   /* Makes the element tiny but still technically on the page */
  overflow: hidden;             /* Prevents scrollbars or text spill */
  position: absolute;           /* Removes the element from the normal flow so it does not affect layout */
  white-space: nowrap;          /* Prevents line breaks that could affect screen reader behavior */
}
```

The `.visually-hidden-sr` class is useful whenever you need screen readers to hear content that sighted users do not need to see. For example, a form with an error count might render `<span class="visually-hidden-sr">3 errors found.</span>` so a screen reader user hears the summary when the page reloads, while the visual design shows only the inline error messages.

### Images

When images or icons serve only a decorative purpose, exclude them from the accessibility tree so screen readers do not announce them. To hide an element from the AT:

- Leave the `alt` attribute empty
- Add `aria-hidden="true"`

```html
<img src="#" alt="">

<button>
    <svg aria-hidden="true"></svg>
</button>
```

## Disclosure widgets

A *disclosure widget* is another term for an accordion: a section of content that a button can expand or collapse.

### Native \<details> element

The native disclosure widget is the `<details>` element. This element has some notable behaviors:

- Page search (Ctrl + F) searches and highlights content in this element whether it is expanded or collapsed.
- Support across screen readers is inconsistent.

The visible label goes in `<summary>` and the hidden content is any other child element:

```html
<details>
    <summary>Show details</summary>
    <p>here is the content</p>
</details>
```

Style the open and closed states with these selectors:

```css
summary::marker {
  content: "+ ";
}

details[open] summary::marker {
  content: "- ";
}
```

You can select it and track its toggle state with JavaScript:

```js
const details = document.querySelector('details');

details.addEventListener('toggle', e => {
    console.log(details.open);                  // logs true or false
});
```


### Custom accordion

The following is a custom widget using these ARIA controls:

- `aria-expanded="false"`: Describes whether the content is visible.
- `aria-controls="content"`: Associates the button with the content it controls.

```html
<div class="disclosure">
    <button aria-expanded="false" aria-controls="content">
        Show details
    </button>
    <div class="disclosure-content" id="content">
        <p>Detailed content goes here....</p>
    </div>
</div>
```

Here is some basic CSS that shows and hides the content based on the ARIA attribute:

```css
[aria-expanded="false"] + .disclosure-content {
  display: none;
}
```

You can apply more complex CSS and JavaScript for a smoother animated transition. The key challenge is that you cannot animate `height` from `0` to `auto`, but you can animate grid row height. Create one row for the button and one for the content using `grid-template-rows`. By default the content row is `0fr`, which collapses it, and it transitions to `1fr` when expanded:

```css
.disclosure {
  --_height: 0fr;

  display: grid;
  justify-content: start;
  grid-template-rows: 1.4em var(--_height);
}

@media (prefers-reduced-motion: no-preference) {
  .disclosure {
    transition: visibility 0.3s, grid-template-rows 0.3s;
  }
}

.disclosure > [aria-expanded] {
  width: fit-content;
}

.disclosure > [aria-expanded="false"] + .disclosure-content {
  visibility: hidden;
}

.disclosure:has([aria-expanded="true"]) {
  --_height: 1fr;
}

.disclosure .disclosure-content {
  overflow: hidden;
}
```

The JavaScript toggles `aria-expanded` on each click. The comparison against the string `"false"` is intentional: `getAttribute` always returns a string, so you must compare against `"false"` (not the boolean `false`) to get the correct result:

```js
button.addEventListener('click', e => {
    button.setAttribute(
        'aria-expanded',
        button.getAttribute('aria-expanded') === 'false'
    );
});
```

### Multiple accordions

The following HTML wraps multiple accordion items in a `<section>`. The `aria-labelledby` attribute gives the section an accessible name by referencing the heading, so screen readers announce the section name when the user enters it:

```html
<section aria-labelledby="faq_heading" class="faq">
    <h2 id="faq_heading">Frequently asked questions</h2>
    <h3>First question</h3>
    <div class="faq-content">
        <p>First answer...</p>
    </div>
    <h3>Second question</h3>
    <div class="faq-content">
        <p>Second answer...</p>
    </div>
    <h3>Third question</h3>
    <div class="faq-content">
        <p>Third answer...</p>
    </div>
</section>
```

Here is the JavaScript. It replaces each heading's text with a `<button>` element so keyboard users can activate it. The buttons are created dynamically rather than written in HTML so the content degrades gracefully when JavaScript is not available:

```js
const faq = document.querySelector('.faq');                 // Select the entire FAQ section
const headings = faq.querySelectorAll('h3');

for (let i = 0; i < headings.length; i++) {                 // Add event listeners to all headings
    const button = document.createElement('button');        // Create a button to replace the heading's text
    const heading = headings[i];                            
    const content = heading.nextElementSibling;             // Get the content div
    const id = `faq_${i}`;                                  // Create a unique ID for the content

    button.setAttribute('aria-expanded', false);            // Set ARIA attr: not expanded by default
    button.setAttribute('aria-controls', id);               // Associate heading button with content by ID
    button.textContent = heading.textContent;               // Copy heading text as button label
    heading.innerHTML = "";                                 // Clear heading text
    heading.append(button);                                 // Replace heading text with the button

    content.setAttribute('id', id);
}

faq.addEventListener('click', e => {
    const button = e.target.closest("[aria-expanded]");

    if (button) {       // Only proceed if the heading button with ARIA was found
        const isOpen = button.getAttribute("aria-expanded") === 'false';
        button.setAttribute('aria-expanded', isOpen);
    }
});
```

Here is some basic CSS:

```css
.faq [aria-expanded] {
  all: unset;
}

.faq [aria-expanded]:focus-visible {
  outline: 0.25em solid;
}

h3:has([aria-expanded="false"]) + .faq-content {
  display: none;
}
```
