---
title: "Toggle buttons"
# linkTitle: "CSS in Depth"
weight: 30
# description:
---

Buttons are inherently accessible to keyboards and screen readers. If the button is not submitting a form, use `type=button`. This means that you don't have to add `e.preventDefault()` to keep the browser from submitting a form. 


### Pressed button styles

If you are creating a button that looks like an actual button--is depressed after a click--add `aria-pressed`. This makes some screen readers announce the button as a toggle button.

Use `position: relative;` so you can move the element without removing it from the doc flow. This is better for performance because it does not trigger GPU acceleration or trigger a new stacking context. In addition, its great for subtle static layout tweaks that need to include shadows or outlines. Shadows, outlines, and background layers are shifted with the element.

You can use `transform: translate();` for animation, transitions, or to increase performance.

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

Focus styles should not affect layout:
- `box-shadow` for focus styles because it respects the curved corners.
- `outline` and `outline-offset` to add transparent styles for forced-color and high-contrast mode.

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

### Toggle switch

> Compare with example from [Accessibility](http://localhost:1313/docs/web-dev/accessibility/buttons/#toggle-switch) for better final component with rounded corners.

This component is a toggle switch that has "on" and "off" text within the button. Add these attributes to a button to make it an accessible switch:
- `role="switch"`
- `aria-checked="true"`
- `aria-labelledby="<associated-label>"`


#### HTML

The markup places each button within a list item with the following properties and attributes:
- Each list item has a `<span>` that acts as a label for the button
- Each button uses the `role="switch"`, which communiates its state with the aria-checked attribute.
- Each button also uses spans to list on/off state

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

#### CSS

The CSS is pretty complicated, so each ruleset is associated with a number and described here:
1. Targets the on/off spans within the button using a descendant selector. When the switch has `aria-checked="true"`, the first child (`on`) gets a black background. When the switch has `aria-checked="false"`, last child (`off`) gets the black background.
2. Styles for the button. Use flex to center the spans within the button, if necessary.
3. Focus styles. These are transparent for forced-color mode.
4. Styles for the on/off spans. Make them an inline block so you can add padding.
5. List styles - these aren't specific to the buttons so no need to expand here. The last rule targets the 'off' span so there is more space between the switches.

```css
[role="switch"][aria-checked="true"] :first-child,      /* 1 */
[role="switch"][aria-checked="false"] :last-child {
  background: #000;
  color: #fff;
  border-radius: inherit;
}

[role="switch"] {                                       /* 2 */
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
  border: 3px solid #000;
  font-size: inherit;

  display: flex;
  align-items: center;
}

[role="switch"]:focus {                                 /* 3 */
  outline: 2px solid transparent;
  outline-offset: 2px;
  box-shadow: 0 0 0 0.25rem skyblue;
}

[aria-checked] span {                                   /* 4 */
  font-size: 0.75rem;
  font-weight: bold;
  display: inline-block;
  border-radius: 3px;
  padding: 0.25rem;
}

/* List styles */                                       /* 5 */
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
  margin-left: 0.5rem;
}
```

#### Javascript

This JS selects all buttons on the page. `querySelectorAll` returns a NodeList, so we convert it to an array and add the click event to each item in the array with a `forEach`.

We saved whether the `aria-checked` state attribute is `"true"` in a variable. When a button is clicked, it toggles that value and updates the `aria-checked` state attribute. So if the value is `true`, then it is set to `false`, and vice versa:

```js
const toggles = Array.from(document.querySelectorAll('[role="switch"]'));

toggles.forEach(toggle => {
    toggle.addEventListener('click', e => {
        let isChecked = toggle.getAttribute('aria-checked') === 'true';
        toggle.setAttribute('aria-checked', !isChecked);
    });
});
```


### References

- [Replacing Radio Buttons Without Replacing Radio Buttons](https://www.sitepoint.com/replacing-radio-buttons-without-replacing-radio-buttons/)
- [WTF, forms?](http://wtfforms.com/)