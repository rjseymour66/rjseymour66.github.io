---
title: "Transitions"
linkTitle: ""
weight: 180
# description:
---

Transitions add motion and change to the page:
- tell the browser to ease one value into another when the value changes. Think state changes on a link
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
```


## Basics

Transitions morph styles from a first ruleset to a second ruleset, where the second ruleset is defines styles for a state change on an element.

1. Add the `transition-*` properties to the rulese that targets the element at all times. This is usually the ruleset for the element's original (static) state:
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


## Dropdown menu example

This example is a menu that opens when you click the menu button. The colors on the menu button and list item change when you hover the mouse on them:

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
    display: none;
    background-color: var(--background-color);
    width: 10em;
  }
  .dropdown.is-open .dropdown__drawer {             /* change li display to block with this class */
    display: block;
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