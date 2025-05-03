---
title: "Toggling content visibility"
linkTitle: "Visibility"
weight: 90
# description:
---

## Hide content

[Hiding content responsibly](https://kittygiraudel.com/2021/02/17/hiding-content-responsibly/) provides an awesome overview.

Here are some general guidelines:
- Do not put `aria-hidden="true"` or `role="presentation"` on focusable, interactive elements like a button.
- If you hide content visually with `opacity`, `height`, or `transform`, hide the content semantically with something like `visibility: hidden;`
- Don't hide elements semantically if they're referenced elsewhere in the document. For example, don't hide an input that has a visible label.

As an example, you want the skip link before the nav to be machine readable, but you don't want to display menu content hiding until a button press. This depends on :

This example hides content but keeps it accessible to screen readers. Read about it in [anatomy of visually-hidden](https://www.tpgi.com/the-anatomy-of-visually-hidden/):

```css
.visually-hidden-sr {
  clip-path: inset(50%);        /* Clips all visual content (effectively hides it from view) */
  height: 1px;                  /* Makes the element tiny but still technically on the page */
  width: 1px;                   /* Makes the element tiny but still technically on the page */
  overflow: hidden;             /* Prevents scrollbars or text spill */
  position: absolute;           /* Removes the element from the normal flow so it doesn’t affect layout */
  white-space: nowrap;          /* Prevents line breaks that could affect screen reader behavior */
}
```

### Images

When you use images or icons for decorative content, you might want to hide it from the accessiblity tree (not machine-readable). To hide it from the AT:
- leave the `alt` tag empty
- add `aria-hidden="true"`

```html
<img src="#" alt="">

<button>
    <svg aria-hidden="true"></svg>
</button>
```

## Disclosure widgets

"Disclosure widget" is another term for "accordian".

### Native \<details> element

The native disclosure widget is the `<details>` element. This element has some strange behaviors:
- Page search (CTRL + f) searches and highlights content in this element whether it is expanded or not
- Doesn't work well with all screen readers

The visibile element is in `<summary>` and the hidden content is any other content:

```html
<details>
    <summary>Show details</summary>
    <p>here is the content</p>
</details>
```

Style it with these selectors:

```css
summary::marker {
  content: "+ ";
}

details[open] summary::marker {
  content: "- ";
}
```

You can select it and track its toggle state with JS:

```js
const details = document.querySelector('details');

details.addEventListener('toggle', e => {
    console.log(details.open);                  // logs true or false
});
```


### Custom accordian

Here is a custom widget. It uses the following ARIA controls:
- `aria-expanded="false"`: describes its state
- `aria-control="content"`: associates the button to the content div

```html
<div class="disclosure">
    <button aria-expanded="false" aria-control="content">
        Show details
    </button>
    <div class="disclosure-content" id="content">
        <p>Detailed content goes here....</p>
    </div>
</div>
```

Here is some basic CSS:

```css
[aria-expanded="false"] + .disclosure-content {
  display: none;
}
```

You can use more complex CSS and JS for a smoother transition:
- You can't transition the height of an element, so use a grid.
- Create a row for the button, and one for the content with `grid-template-rows`. By default, the content is `0fr`, but transistions to `1fr`.

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

This requires some JS to toggle the state with the `aria-expanded` attribute:

```js
button.addEventListener('click', e => {
    button.setAttribute(
        'aria-expanded',
        button.getAttribute('aria-expanded') === "false"
    );
});
```

### Multiple accordians

Here is the HTML for multiple accordians in a `<section>` element. Notice that `aria-labelledby` associates the section with the heading to give it an accessible name:

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

Here is the JS:

```js
const faq = document.querySelector('.faq');                 // Select the entire FAQ section
const headings = faq.querySelectorAll('h3');

for (let i = 0; i < headings.length; i++) {                 // Add event listeners to all headings
    const button = document.createElement('button');        // create a button to replace the heading's text
    const heading = headings[i];                            
    const content = heading.nextElementSibling;             // Get the content div
    const id = `faq_${i}`;                                  // Create UID for content

    button.setAttribute('aria-expanded', false);            // Set ARIA attr. not expanded by default
    button.setAttribute('aria-controls', id);               // ARIA to associate heading button with content id
    button.textContent = heading.textContent;               // Uses heading text as button text
    heading.innerHTML = "";                                 // Clears heading text
    heading.append(button);                                 // Replaces heading with clickable button

    content.setAttribute('id', id);
}

faq.addEventListener('click', e => {
    const button = e.target.closest("[aria-expanded]");

    if (button) {       // only proceed if the heading button w/ARIA was found
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