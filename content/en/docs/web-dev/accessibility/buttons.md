---
title: "Buttons"
# linkTitle: ""
weight: 50
# description:
---

According to the WCAG, buttons need to meet these six baseline criteria:

Must convey a semantic button role programmatically:
: Use the `<button>` element so it has the implicit `button` role

Must have a concise and straightforward accessible name
: Label the button, or the user or screen reader won't know what to do with it

Communicates its state (pressed, etc)
: Use ARIA attributes to convey the button's state or the state of a different element that the button controls

Recognizable as a button
: Buttons should look like buttons. Users spend time on other sites that use the same general style of button, and they 
expect your site's buttons to look like theirs.

Colors must have sufficient contrast
: Must have a contrast ratio of 4:5:1 for normal text, 3:1 for large text.
  - background color or outline of a focus indicator or border should have contrast ration of 3:1

Must be focusable and allow activation through click, touch, or key events
: A button is an _interactive element_, which means you have to be able to perform identical actions with the mouse and keyboard by pressing `Enter` or `space`. To do this, make it tabbable.

```html
<!-- submit button -->
<form action="">
        <label for="email">Email</label>
        <input type="email" name="email" id="email" />
        <button>Sign up</button>
</form>

<!-- JS button -->
<button type="button" id="js-handle">Print</button>

<!-- convey state -->
<button type="button" aria-pressed="true">Mute</button>
```

The button typically has two use cases:
- submitting a form - you don't have to set this to `type="submit"` if its within a form
- running JS after user interaction. Set this to `type="button"`. Examples include:
  - Opening a dialog
  - toggling visibility on other elements
  - running other JS functions

## Labeling

When deciding how to label a button, consider these methods, in order of preference:
1. Native HTML techniques
2. `aria-labelledby` pointing to existing text
3. Visibly hidden content (ex: in a `<span>`)
4. `aria-label`

If the button contains text, the text is the label:

```html
<button type="button">Save</button>
```

If the button has an image, its `alt` text is the label. These images are called _functional_ images. The `alt` text should describe its purpose:

```html
<button type="button"><img src="/path/to/svg" alt="Download"></button>
```

If you use an SVG, use `aria-labelledby` to create a reference to the SVG title. Here, the ARIA label references the `id` in the `<title>` element, which provides the accessible name **Download**:

```html
<button type="button">
    <svg aria-labelledby="title">
        <title id="title">Download</title>
    </svg>
</button>
```

If you don't want to include the image in the accessibility tree (AT), omit the `alt` content from an `img`, or add `aria-hidden` to an SVG. Do this when the image is redundant or doesn't provide additional information:

```html
<button type="button">
    <span class="visually-hidden">Download</span>
    <img src="/path/to/svg" alt="" />
</button>
```

Your button might include an icon, like an arrow pointing down for a "download" button. Omit this from the AT if the button includes text:

```html
<button type="button">
    <span class="visually-hidden">Download</span>
    <svg aria-labelledby="title">
        ...
    </svg>
</button>
```

## Styling

Default button styles are pretty bad, but you still want to use a `<button>` element. You just need to remove its styles and add your own.

Never use a `<div>` as a button for the following reasons:
- It's not focusable by default
- You can't activate it with `Enter` or `Space`
- Screen readers do not announce it as a button, or at all
- It's invalid to name generic elements with `aria-label` 

Here are all the styles that you need to define to style a button:

```css
button {
    background: none;
    border: 1px solid transparent;      /* shows outline in forced-colors mode */
    font: inherit;
    padding: 2rem 4rem;
}
```

## States and properties

aria-expanded
: State attribute, indicates whether a button expands or collapses the element it controls. Set to `true` or `false`.

aria-controls
: Property attribute, creates a relationship between a button and another element. The related element must have an `id` attribute equal to `aria-controls`.

aria-pressed
: State attribute, indicates current "pressed" or "toggled" state of a toggle button. Set to `true` or `false`.

aria-checked
: State attribute, indicates current "checked" state of a checkbox, radio button, and other widgets, like toggle switches.

aria-haspopup
: Property attribute, indicates that a button controls an interactive pop-up element. Supports these values:
  - `true` (same as `menu`)
  - `false`
  - `menu` (same as `true`)
  - `dialog`
  - `grid`
  - `listbox`
  - `tree`


### Hide/unhide list

If a button toggles the visibility of another element needs to communicate the element's state. To do this:
- Add `aria-expanded` to the button, not the expanded element
- Hide the list using the `aria-expanded="false"` attribute and value
- Use a click event on the button that toggles the `aria-expanded` attribute value

Here is the HTML. It is an abbreviated nav that displays or hides a list using the button:

```html
<nav>
    <button aria-expanded="false" aria-controls="main-nav">
        Navigation
    </button>
    <ul id="main_nav">
        <li>1</li>
        <li>2</li>
        <li>3</li>
    </ul>
</nav>
```

The CSS applies styles using the `aria-expanded` attribute. Here, it uses the attribute selector and applies to the adjacent `<ul>` a rule that hides the list when the ARIA attr is set to `false`:

```css
[aria-expanded="false"] + ul {
  display: none;
}
```

Finally, add the JS that toggles the `aria-expanded` value on a click event:

```js
const button = document.querySelector("button");

button.addEventListener('click', (e) => {
    const isExpanded = button.getAttribute('aria-expanded') === 'true';
    button.setAttribute('aria-expanded', !isExpanded);
});
```

> If the button is pressed to indicate that something was added as a favorite, use `aria-pressed=true` in place of `aria-expanded="true"`.

### Toggle switch

You can create a toggle switch with simple HTML, semi-complicated CSS, and simple JS. Here is the HTML--note how its not type="button", but rather role="button", and it uses the `aria-checked` attribute:

```html
<button class="toggle-btn" id="toggle" role="button" aria-checked="false">
    Functional cookies
</button>
```

The CSS is pretty complicated. You have to create a toggle switch and its background from pseudo-elements on the button. Here is a numbered description of each rule:
1. Button reset styles. We unset all styles and make it a flex container to center the label and the switch
2. Create the background of the switch
3. Create the moveable indictor of the switch. The size is calculated by taking the height of the background pseudo-element and subtracting 2x the toggle offset. This leaves a thin strip of background between the indicator and its background. You also use the offset when setting the `left` positioning of the element so it is not flush with the background.
4. Focus styles applied to the background
5. Apply a blur when you focus or hover
6. Change the color of the background when the switch is toggled to the right
7. Moves the indicator to the right

```css
.toggle-btn {                                           /* 1 */
  --toggle-offset: 0.125em;
  --toggle-height: 1.6em;
  --toggle-background: oklab(0.82 0 0);

  all: unset;
  align-items: center;
  display: flex;
  gap: 0.5em;
  position: relative;
}

.toggle-btn::before {                                   /* 2 */
  background: var(--toggle-background); 
  border-radius: 4em;
  content: "";
  display: inline-block;
  height: var(--toggle-height);
  transition: background 0.3s box-shadow 0.3s;
  width: 3em;
}

.toggle-btn::after {                                    /* 3 */
  --_size: calc(var(--toggle-height) - (var(--toggle-offset) * 2));

  background: #fff;
  border-radius: 50%;
  content: "";
  height: var(--_size);
  left: var(--toggle-offset);
  position: absolute;
  transform: translate 0.3s;
  top: var(--toggle-offset);
  width: var(--_size);
}

.toggle-btn:focus-visible::before {                     /* 4 */
  outline: 2px solid;
  outline-offset: 2px;
}

.toggle-btn:is(:focus-visible, :hover)::before {        /* 5 */
  box-shadow: 0px 0px 3px 1px rgb(0 0 0 /0.3);
}

[aria-checked="true"] {
  --toggle-background: oklab(0.7 -0.18 0.17);           /* 6 */
}

.toggle-btn[aria-checked="true"]::after {               /* 7 */
  translate: 100% 0;
}
```

Here is the JS. It just toggles the `isChecked` attribute that stores the `aria-checked` value:

```js
const toggle = document.querySelector('#toggle');

toggle.addEventListener('click', (e) => {
    const isChecked = toggle.getAttribute('aria-checked') === 'true';
    toggle.setAttribute('aria-checked', !isChecked);
    console.log(toggle);
});
```

## Make a div a button

In general, you should always use buttons as buttons. If you have to use a div, make sure it does these things so it behaves exactly like a button:
- `role="button"`
- Focusable
- Has focus styles
- `keydown` event is listening for the Enter key
- `keyup` event is listening for the Space key