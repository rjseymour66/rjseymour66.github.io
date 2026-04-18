---
title: "Buttons"
# linkTitle: ""
weight: 50
# description:
---

According to the WCAG, buttons need to meet these six baseline criteria:

Must convey a semantic button role programmatically
: Apply the `<button>` element so it has the implicit `button` role.

Must have a concise and straightforward accessible name
: Label the button, or the user and screen reader will not know what it does.

Communicates its state (pressed, etc)
: Apply ARIA attributes to convey the button's state, or the state of a different element that the button controls.

Recognizable as a button
: Buttons should look like buttons. Users spend time on other sites that follow the same general visual conventions, and they expect your site's buttons to match.

Colors must have sufficient contrast
: Must have a contrast ratio of 4.5:1 for normal text and 3:1 for large text.
  - The background color or outline of a focus indicator or border must have a contrast ratio of 3:1.

Must be focusable and allow activation through click, touch, or key events
: A button is an *interactive element*, which means users must be able to perform identical actions with the mouse and keyboard by pressing `Enter` or `Space`. To do this, make it tabbable.

The following examples cover the two main button types and ARIA state communication:

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

- **Submitting a form**: You do not have to set this to `type="submit"` if it is within a form.
- **Running JavaScript after user interaction**: Set this to `type="button"`. Examples include:
  - Opening a dialog
  - Toggling visibility on other elements
  - Running other JavaScript functions

## Labeling

When deciding how to label a button, consider these methods, in order of preference:

1. Native HTML techniques
2. `aria-labelledby` pointing to existing text
3. Visibly hidden content (for example, in a `<span>`)
4. `aria-label`

If the button contains text, the text is the label:

```html
<button type="button">Save</button>
```

If the button has an image, its `alt` text is the label. These images are called *functional images*. The `alt` text should describe the button's purpose, not the image itself:

```html
<button type="button"><img src="/path/to/svg" alt="Download"></button>
```

If you apply an SVG, apply `aria-labelledby` to create a reference to the SVG title. Here, the ARIA label references the `id` in the `<title>` element, which provides the accessible name **Download**:

```html
<button type="button">
    <svg aria-labelledby="title">
        <title id="title">Download</title>
    </svg>
</button>
```

If you do not want to include the image in the accessibility tree (AT), omit the `alt` content from an `img`, or add `aria-hidden` to an SVG. Do this when the image is redundant or does not provide additional information:

```html
<button type="button">
    <span class="visually-hidden">Download</span>
    <img src="/path/to/svg" alt="" />
</button>
```

Your button might include a decorative icon, like a downward arrow on a "Download" button. Omit this icon from the AT when the button also includes visible text, so screen readers do not announce both:

```html
<button type="button">
    Download
    <svg aria-hidden="true">
        ...
    </svg>
</button>
```

## Styling

Default browser button styles are plain, but you should still apply the `<button>` element. Remove its default styles and apply your own.

Never apply a `<div>` as a button for the following reasons:

- It is not focusable by default, so keyboard users cannot reach it.
- You cannot activate it with `Enter` or `Space`, which means keyboard users cannot trigger it.
- Screen readers do not announce it as a button, or may not announce it at all.
- Naming generic elements with `aria-label` is invalid per the ARIA specification.

The following CSS resets browser button styles to a clean baseline you can build on:

```css
button {
    background: none;
    border: 1px solid transparent;      /* shows outline in forced-colors mode */
    font: inherit;
    padding: 2rem 4rem;
}
```

## States and properties

`aria-expanded`
: *State attribute.* Indicates whether a button expands or collapses the element it controls. Set to `true` or `false`.

`aria-controls`
: *Property attribute.* Creates a relationship between a button and another element. The related element must have an `id` attribute whose value matches the `aria-controls` value exactly.

`aria-pressed`
: *State attribute.* Indicates the current "pressed" or "toggled" state of a toggle button. Set to `true` or `false`.

`aria-checked`
: *State attribute.* Indicates the current "checked" state of a checkbox, radio button, or other widget such as a toggle switch.

`aria-haspopup`
: *Property attribute.* Indicates that a button controls an interactive popup element. Supports these values:
  - `true` (same as `menu`)
  - `false`
  - `menu` (same as `true`)
  - `dialog`
  - `grid`
  - `listbox`
  - `tree`


### Hide/unhide list

A button that toggles the visibility of another element needs to communicate that element's state. To do this:

- Add `aria-expanded` to the button, not the expanded element.
- Hide the list by setting `aria-expanded="false"` on the button.
- Add a click event to the button that toggles the `aria-expanded` value.

Here is the HTML. It is an abbreviated navigation that shows or hides a list when the button is activated. Note that `aria-controls` and `id` must match exactly:

```html
<nav>
    <button aria-expanded="false" aria-controls="main-nav">
        Navigation
    </button>
    <ul id="main-nav">
        <li>1</li>
        <li>2</li>
        <li>3</li>
    </ul>
</nav>
```

The CSS applies styles based on the `aria-expanded` attribute value. It hides the adjacent `<ul>` when the attribute is `false`:

```css
[aria-expanded="false"] + ul {
  display: none;
}
```

Finally, add the JavaScript that toggles the `aria-expanded` value on each click:

```js
const button = document.querySelector("button");

button.addEventListener('click', (e) => {
    const isExpanded = button.getAttribute('aria-expanded') === 'true';
    button.setAttribute('aria-expanded', !isExpanded);
});
```

> If the button is pressed to mark something as a favorite, apply `aria-pressed="true"` in place of `aria-expanded="true"`.

### Toggle switch

You can create a toggle switch with HTML, CSS, and JavaScript. The following HTML applies `aria-checked` rather than `aria-pressed` because the switch represents a persistent on/off state, similar to a checkbox:

```html
<button class="toggle-btn" id="toggle" aria-checked="false">
    Functional cookies
</button>
```

The CSS creates the visual switch using pseudo-elements on the button. Each numbered comment corresponds to a rule below:

1. Button reset styles. Unset all defaults and make it a flex container to center the label and switch.
2. Create the pill-shaped background of the switch using `::before`.
3. Create the circular indicator using `::after`. Its size is the background height minus twice the offset, leaving a thin gap between the indicator and the background edge. The offset also sets the initial `left` position so the indicator is not flush with the background.
4. Apply a focus outline to the background pseudo-element when the button receives keyboard focus.
5. Apply a soft shadow when the button is hovered or focused.
6. Change the background color when the switch is toggled on.
7. Move the indicator to the right when the switch is toggled on.

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
  transition: background 0.3s, box-shadow 0.3s;
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
  transition: translate 0.3s;
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

Here is the JavaScript. It reads the current `aria-checked` string and sets the opposite value:

```js
const toggle = document.querySelector('#toggle');

toggle.addEventListener('click', (e) => {
    const isChecked = toggle.getAttribute('aria-checked') === 'true';
    toggle.setAttribute('aria-checked', !isChecked);
});
```

## Make a div a button

In general, always apply `<button>` for button behavior. If you must apply a `<div>`, add all of the following so it behaves exactly like a native button:

- `role="button"` to announce it as a button to screen readers
- `tabindex="0"` to make it focusable by keyboard
- Explicit focus styles so keyboard users can see it
- A `keydown` event listener that activates on the `Enter` key
- A `keyup` event listener that activates on the `Space` key
