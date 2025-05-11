---
title: "Toggle buttons"
# linkTitle: "CSS in Depth"
weight: 30
# description:
---

Compare with this and come up with something good:

http://localhost:1313/docs/web-dev/accessibility/buttons/#toggle-switch




- use `type=button` so you don't have to add `e.preventDefault()` to keep the browser from submitting a form
- buttons are inherently accessible to keyboards and screen readers
- `aria-pressed` makes some screen readers announce the button as a toggle button


- Use transform: translate() for animation, transitions, or performance.
- Use position: relative for subtle static layout tweaks that need to include shadows or outlines.


### Links

- [Replacing Radio Buttons Without Replacing Radio Buttons](https://www.sitepoint.com/replacing-radio-buttons-without-replacing-radio-buttons/)
- [WTF, forms?](http://wtfforms.com/)




### Pressed button styles

Use `position` so you can move the element without removing it from the doc flow. This is better for performance because it does not trigger GPU acceleration or trigger a new stacking context.

Also, shadows, outlines, and background layers are shifted with the element.



```css
[aria-pressed] {
    padding: 1rem 2rem;
    color: #fff;
    border-radius: 4px;
    border: none;
    font-family: sans-serif;
    font-weight: 700;
    font-size: 1.25rem;
    background: #000;

    position: relative;
    top: -0.25rem;
    left: -0.25rem;
    box-shadow: 0.125em 0.125em 0 #fff,
                0.25em 0.25em #000;
}

[aria-pressed="true"] {
    box-shadow: inset 0 0 0 0.15rem #000,
                inset 0.25em 0.25em 0 #fff;
}
```

### Focus styles

- focus styles should not affect layout
- use box-shadow here bc outline only draws a box around an element, does not respect the curved corners
- use transparent outline for high contrast mode

```css
[aria-pressed="true"]::after {
    content: "\2713";
    position: absolute;
    top: -2px;
    right: -50px;
    color: #000;
    font-size: 3rem;
}

[aria-pressed]:focus {
    outline: 2px solid transparent;
    box-shadow: 0 0 0 0.25rem skyblue;
}

[aria-pressed="true"]:focus {
    box-shadow: 0 0 0 0.25rem skyblue,
                inset 0 0 0 0.15rem #000,
                inset 0.25em 0.25em 0 #fff;
}
```

### Labels

- Never change pressed state and label together because you communicate the state when you change a label

- for voice recognition software, you need to ID buttons by vocalizing the label, so it is better to switch the label instead of the state
- for translation, use a hidden span bc 


If you only change the label for a button, the label is not announced after it is changed--you have to unfocus and refocus it.

### Final

```html
<section class="toggle-section">
    <h2>Notifications</h2>
    <ul>
        <li>
            <span id="notify-email">Notify by email</span>
            <button
                role="switch"
                aria-checked="true"
                aria-labelledby="notify-email"
            >
                <span>on</span>
                <span>off</span>
            </button>
        </li>
        <li>
            <span id="notify-sms">Notify by SMS</span>
            <button
                role="switch"
                aria-checked="true"
                aria-labelledby="notify-sms"
                >
                <span>on</span>
                <span>off</span>
            </button>
        </li>
    </ul>
</section>
```

```css
.visually-hidden {
    position: absolute;
    clip-path: inset(50%);
    height: 1px;
    width: 1px;
    overflow: hidden;
    white-space: nowrap;
}

[role="switch"][aria-checked="true"] :first-child,
[role="switch"][aria-checked="false"] :last-child {
    background: #000;
    color: #fff;
    padding: 0.35em;
    border-radius: inherit;
}

[role="switch"] {
    padding: 0.75rem 1rem;
    border-radius: 4px;
    border: 3px solid #000;
}

[role="switch"]:focus {
    outline: 2px solid transparent;
    box-shadow: 0 0 0 0.25rem skyblue;
}

.toggle-section {
    display: flex;
    flex-direction: column;
    gap: 2rem;
    justify-content: center;
}

ul {
    margin: unset;
    padding: 0;
}

li {
    list-style: none;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0.5rem 0;

    border-bottom: 1px solid black;
}

li span:last-of-type {
    margin-left: 1rem;
}
```


```js
const toggles = document.querySelectorAll('[role="switch"]');

Array.from(toggles).forEach(toggle => {
    toggle.addEventListener('click', e => {
        let isChecked = toggle.getAttribute('aria-checked') === 'true';
        toggle.setAttribute('aria-checked', !isChecked);
    });
});
```