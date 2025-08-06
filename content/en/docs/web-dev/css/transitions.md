---
title: "Transitions"
linkTitle: "xTransitions"
weight: 180
# description:
---

Transitions add motion and change to the page:
- tell the browser to ease one value into another when the value changes. Think state changes on a link--you could have a 0px border radius on a button, and then when you hover you transition into a 5px border radius.
- `transition-*` properties add motion to the page.

## Accessibility

Some users set up their OSs to prevent certain motions on the screen. Check whether these settings exist with the `prefers-reduced-motion` media query as part of your `reset` styles:

```css
@layer reset {
    @media (prefers-reduced-motion: reduce) {
        *,
        *::before,
        *::after {
            animation-duration: 0.01ms !important;
            animation-iteration-count: 1 !important;
            transition-duration: 0.01ms !important;
            scroll-behavior: auto !important;
        }
    }
}

/* basic example */
@media (prefers-reduced-motion: reduce) {
  rect { animation: none; }
}
```
> It might also help to include a switch that lets users opt in/out or pause animations.

The browser reviews animations and determines whether to enable them based on the following:
- How fast it is
- How long it is
- How much of the viewport it uses
- What the flash rate is
- How essential it is to the functioning of the site or understanding of the content

### View in the browser

To view your settings with accessibility settings applied:
1. Open Chrome tools.
2. Select the vertical ellipses.
3. Go to More tools > Rendering.
4. Scroll to **Emulate CSS media feature prefers-reduced-motion** and select **prefers-reduced-motion: reduce**.

## Basics

Transitions morph styles from a first ruleset to a second ruleset, where the second ruleset defines styles for a state change on an element.

1. Add the `transition-*` properties to the ruleset that targets the element at all times. This is usually the ruleset for the element's original (static) state:
   - `transition-property` to define which property you want to transition after a state change. Can be a specific property, or `all` for all properties defined on the state.
   - `transition-duration` is the amount of time it takes to transition into the properties for the new state. Takes `s` or `ms`.

2. On the new state's ruleset, define the properties that you want to change.

### Example

When you hover over this button, the border radius and background color change to the styles defined in the `:hover` state ruleset. Border radius changes even though it is not defined on the static element ruleset:

```css
button {
  padding: 0.3em 1em;
  border: 0;
  font-size: 1rem;
  color: white;
  background-color: oklch(74% 0.11 195deg);
  transition-property: all;
  transition-duration: 0.5s;
}

button:hover {
  border-radius: 1em;
  background-color: oklch(55% 0.16 24deg);
}
```

### transition

Not all properties can be animated (apply a transition to it). For example, you can't animate the `display` property:
- Look in the MDN docs for details about whether you can animate the property. If it has an animation type of `discrete`, it cannot be animated.
- In general, properties that accept a length, number, color, or `calc()` function can be animated
- When you create a transition, slow it down to 2 or 3 seconds to make sure it is behaving the way you want it to.

`transition` property is shorthand for these values:

```css
transition: <affected-property> <duration> <timing-function> <delay>
/* for example */
transition: background-color 0.3s linear 0.5s;

/* target multiple properties */
transition: background-color, color, font-size 0.3s linear 0.5s;

/* different transitions to different properties */
transition: border-radius 0.3s linear, background-color 0.6s ease;
/* equivalent in longhand */
transition-property: border-radius, background-color;
transition-duration: 0.3s, 0.6s;
transition-timing-function: linear, ease;
```

- _affected property_ is the property that you want to change
- _duration_ is how long the transition takes. It is a time value expressed in s or ms, cannot be 0.
  - **For hover effects, use a transition duration between 200-500ms or it will seem like your website is slow.**
- _timing-function_ controls the rate of change between the transition.
- _delay_ lets you specify a time value before the transition begins

## Timing functions

The timing function defines how the property value transitions from one value to another:
- does it change at a steady speed or start slowly and accelerate?

See [MDN docs](https://developer.mozilla.org/en-US/docs/Web/CSS/transition-timing-function) for detailed descriptions.

Possible values:
- `linear`: changes at a constant rate
- `ease`: increases in velocity until the middle, then slows down
- `ease-in`: starts slow but then accelerates until transition completes
- `ease-out`: starts quickly but then decelerates until transition completes
- `ease-in-out`: starts slowly, accelerates, then decelerates until complete

### Use cases

- Linear: Color changes and fade in/out effects
- Decelerating: User-initiated changes. Use a flavor of `ease-out`. This lets them see a response to their actions quickly
- Accelerating: System-initiated changes. Use a flavor of `ease-in`. This draws the user's attention at first and then speeds up to complete the transition.

### cubic-bezier()

Create a custom timing function with `cubic-bezier()` function. Timing functions are based on Bezier curves, which calculate a property's value as a function of change over time.

You can experiment with timing functions in the DevTools pane:
1. Go to the element with the `transition` property.
2. Select the box with the curved line to open the cubic bezier editor.

You can also experiment at [cubic-bezier.com](https://cubic-bezier.com/#.17,.67,.83,.67).

### step()

Steps are not very practical, but here are some ideas on [CSS Tricks](https://css-tricks.com/clever-uses-step-easing/).

Instead of a fluid transition, you can transition in discrete steps. This takes two parameters: the number of steps, and the `start` or `end` keyword that indicates whether the change should occur at the start or end of each step:

```css
.box {
  position: absolute;
  left: 0;
  height: 50px;
  width: 50px;
  background-color: oklch(70% 0.18 145deg);
  transition: all 1s steps(3);
}
```

## Examples

### Fade in/out menu

This example is a menu that opens when you click the menu button. It has these transitions:
- The colors on the menu button and list item change when you hover the mouse on them
- menu fades in and out when you click the toggle button
  - transition `opacity` from `0` to `1`
  - remove the menu drawer from the page with `visibility`, which is animatable--unlike `display`. `visibility` accepts either `visible` or `hidden`. If an element is `hidden`, it is still in the document flow, but this doesnt matter with the menu we are creating because it is absolutely positioned. 
- set the `transition` to `transition: opacity 0.2s linear, visibility 0s linear 0.2s;`
  - When the menu closes, you transition the opacity for 0.2s, and then transitions the visibility in 0s, but after a 0.2s delay. This delay is enough time for the opacity to fade out, and then the visiblity completely removes it afterwards.
  - When the menu opens, you set the `visibility` to visible, and remove the delay by setting `transition-delay: 0s`

```html
<div class="dropdown" aria-haspopup="true">                             <!-- dropdown container -->
      <button class="dropdown__toggle" type="button">Menu</button>      <!-- dropdown button -->
      <div class="dropdown__drawer">                                    <!-- container for menu lis-->
        <ul class="menu" role="menu">                                   <!-- menu items -->
          <li role="menuitem">
            <a href="/features">Features</a>
          </li>
          <li role="menuitem">
            <a href="/pricing">Pricing</a>
          </li>
          <li role="menuitem">
            <a href="/support">Support</a>
          </li>
          <li role="menuitem">
            <a href="/about">About</a>
          </li>
        </ul>
      </div>
    </div>
    <p><a href="/read-more">Read more</a></p>
    <script>
        let toggle = document.getElementsByClassName("dropdown__toggle")[0];    // toggle .is-open class on click
        let dropdown = toggle.parentElement;
        toggle.addEventListener("click", function (e) {
            dropdown.classList.toggle("is-open");
        });
    </script>
```

```css
@layer modules {
  .dropdown {
    --border-color: oklch(61% 0.08 314deg);
    --text-color: oklch(39% 0.06 314deg);
    --text-color-focused: oklch(39% 0.2 314deg);
    --background-color: white;
    --highlight-color: oklch(95% 0.01 314deg);
  }
  .dropdown__toggle {
    display: block;
    padding: 0.5em 1em;
    border: 1px solid var(--border-color);
    color: var(--text-color);
    background-color: var(--background-color);
    font: inherit;
    text-decoration: none;
    transition: background-color 0.2s linear;       /* transition color on button when state changes */
  }
  .dropdown__toggle:hover {
    background-color: var(--highlight-color);       /* color button transitions to on hover */
  }
  .dropdown__drawer {
    position: absolute;
    background-color: var(--background-color);
    width: 10em;
    visibility: hidden;                                             /* hidden by default */
    opacity: 0;                                                     /* completely opaque by default */
    transition: opacity 0.2s linear, visibility 0s linear 0.2s;     /* opacity transition in 0.2s */
  }                                                                 /* delay visibility transition for 0.2s */
  .dropdown.is-open .dropdown__drawer {
    visibility: visible;
    opacity: 1;
    transition-delay: 0s;                                           /* remove transition delay so its immediately visible */
  }

  .menu {
    padding-left: 0;
    margin: 0;
    list-style: none;
  }
  .menu > li + li > a {
    border-top: 0;
  }

  .menu > li > a {
    display: block;
    padding: 0.5em 1em;
    color: var(--text-color);
    background-color: var(--background-color);
    text-decoration: none;
    transition: all 0.2s linear;                    /* transition menu item color on state change */
    border: 1px solid var(--border-color);
  }

  .menu > li > a:hover {
    background-color: var(--highlight-color);       /* menu item background-color transitions on hover */
    color: var(--text-color-focused);               /* menu item color transitions on hover */
  }
}
```

### Sliding menu

This menu needs to transition the menu height from `0` to `auto`, but you can't transition from an explicit length (`0`) to `auto`. You need JS to figure out what the height should be with the menu's `scrollHeight` property.

This example reuses the HTML/CSS from the previous section with updated JS:

```js
let toggle = document.getElementsByClassName("dropdown__toggle")[0];
let dropdown = toggle.parentElement;
let drawer = document.getElementsByClassName('dropdown__drawer')[0];
// get the scrollHeight
let height = drawer.scrollHeight;

// if the is-open class is present, set the element height to the 
// scrollHeight value. Otherwise, the height is 0.
toggle.addEventListener("click", function (e) {
    dropdown.classList.toggle("is-open");
    if (dropdown.classList.contains('is-open')) {
        drawer.style.setProperty('height', height + 'px');
    } else {
        drawer.style.setProperty('height', '0');
    }
});
```

