---
title: "Animations"
# linkTitle: ""
weight: 200
# description:
---

Keyframe animations let you make an element take a roundabout path from location to location, or you might want the element to return to its previous location.

A keyframe is a way to define multiple properties that change over a period of time. For example, you can change the background color of an element multiple times over a specified period, or you could rotate an element over a period with multiple `transform` declarations.

## Keyframes

A _keyframe_ is a specific point in an animation:
- you define keyframes, and the browser fills in the points in between so the changes look smooth. This is called _in-betweening_.
  - transforms only let you define the starting and ending point. Keyframes let you define multiple points
  - If you repeat an animation, make sure the ending value matches the beginning value so changes are smooth
- Rules in a keyframe are a very high-priority origin in the cascade, so they take precedence over other declarations. If a box is green before an animation is applied that makes the box red, then the box is red for the duration of the animation and goes back to green at the end.

### @keyframes

There are two main properties:
- `@keyframe` at-rule that defines the animation
- `animation` property that applies the animation to an element.

The `@keyframes` at-rule needs a name, and then you define different steps (keyframes) with percentages. Each keyframe describes what the step should look like when it completed that percentage of its motion.

```css
@keyframes <keyframe-name> {
    0%   { /* ruleset */ }
    25%  { /* ruleset */ }
    50%  { /* ruleset */ }
    100% { /* ruleset */ }
}
```

This keyframe animation moves a box 50px from left to right in 3 keyframes. The color also changes, but because you don't define the color in the second step, the browser fills in the blanks to smoothly transition the colors defined at the first and last step. When the browser fills in the gaps like this, it is called _in-betweening_.

Finally, you apply the animation to an element with the `animation` property, which says "apply the `over-and-back` keyframe animation to this element, make it last 1.5 seconds, the movement should be steady (linear), and repeat it three times":

```css
@keyframes over-and-back {
  0% {
    background-color: hsl(0, 50%, 50%);
    transform: translate(0);
  }
  50% {
    transform: translate(50px);
  }
  100% {
    background-color: hsl(270, 50%, 90%);
    transform: translate(0);
  }
}

.box {
  width: 100px;
  height: 100px;
  background-color: green;
  animation: over-and-back 1.5s linear 3;
}
```

### animation

> To prevent seizures, the WC3 recommends that you don't create an animation that flashes more than three times in a 1-second period.

The `animation` property is shorthand for [multiple properties](https://developer.mozilla.org/en-US/docs/Web/CSS/animation):

```css
.element {
    animation: name duration timing-function delay iteration-count direction fill-mode;
}
```
- `name`: Name of the function defined in the `@keyframe` at-rule
- `duration`: How long the animation lasts
- `timing-function`: How the animation accelerates/decelerates. Use a bezier curve or transition timing function (`ease-in`, etc)
- `iteration-count`: Number of times the animation repeats. By default, is `1`.
- `direction`: Whether the animation plays forward, backward, or alternate. [MDN docs](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-direction)
- `fill-mode`: HOw the animation applies styles to its target before and after execution. [MDN docs](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-fill-mode).


## Animating 3D transforms

> WKND107SPP622

First, put your design and layout in place.

- Puts `perspective` on the grid container so all elements appear to come in from the same location
- Applies an animation to each item

1. Step 1:
   - The negative `translateZ()` value makes it come from a distance, instead of from the viewport
   - `rotateY()` makes the card come in completely sideways, with any angle coming from the shared perspective added to the container
2. Step 2:
   - brings the cards almost completely to the screen (`-160px`)
   - rotates them slightly
3. Step 3:
   - places the card elements completely flat

```css
.flyin-grid__item {
  animation: fly-in 600ms ease-in;
}

@keyframes fly-in {
  0% {
    transform: translateZ(-800px) rotateY(90deg);
    opacity: 0;
  }
  56% {
    transform: translateZ(-160px) rotateY(87deg);
    opacity: 1;
  }
  100% {
    transform: translateZ(0) rotateY(0);
  }
}
```

Here is another example that staggers changes to the height of SVG rect objects:

```css
rect:nth-child(1) {
  fill: #1a9f8c;
  animation-delay: 0;
}
rect:nth-child(2) {
  fill: #1eab8d;
  animation-delay: 200ms;
}
rect:nth-child(3) {
  fill: #20b38e;
  animation-delay: 400ms;
}
...
rect:nth-child(11) {
  fill: #128688;
  animation-delay: 2s;
}
```

### animation properties

See the [MDN docs](https://developer.mozilla.org/en-US/docs/Web/CSS/animation#constituent_properties) for a list of all animation properties and a description.

### animation-delay

Apply `animation-delay` to an item to delay the animation. This lets you create staggered animations that display in a wave:

```css
.flyin-grid__item {
  animation: fly-in 600ms ease-in;
}

.flyin-grid__item:nth-child(2) {
  animation-delay: 0.15s;
}

.flyin-grid__item:nth-child(3) {
  animation-delay: 0.3s;
}

.flyin-grid__item:nth-child(4) {
  animation-delay: 0.45s;
}
```


### animation-fill-mode

`animation-fill-mode` applies styles before, after, or during a keyframe. Accepts these values:
- `none`: default
- `forwards`
- `backwards`
- `both`

For definitions, see the [MDN docs](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-fill-mode).

Styles defined in an animation are only applied during the animation. You might need to apply these styles before the animation. For example, if an element is zooming in on the screen, it will appear in its final location before the animation, then zoom in, then display normally. It makes more sense to apply the first keyframe's styles before the animation begins so the element does not begin in its final location.

This example uses the `backwards` value to apply keyframe properties before the animation begins. Apply `animation-fill-mode` to the static element under the `animation` property:

```css
.flyin-grid__item {
  animation: fly-in 600ms ease-in;
  animation-fill-mode: backwards;
}
```

This essentially makes the animation pause on the first frame until it begins.

## Practical usage

Refer to the [Animation MDN docs](https://developer.mozilla.org/en-US/docs/Web/API/Animation) for details about how to interact with animations with JS.

### Waiting on server response 

When a user action sends a request to a server, you want to communicate that the user is waiting on the server response. You can achieve this with an `is-loading` class that displays a spinner on the screen.

Here, a button displays a spinner when waiting on a server response:
- make the button text transparent so you can replace it with the spinner
- make the button a containing element
- create a spinner out of a pseudo-element and position it absolutely
- create keyframes that rotate the pseudo-element
  - it applies the animation until the class is removed (`infinite`)

```css
button.is-loading {
  position: relative;
  color: transparent;
}

button.is-loading::after {
  position: absolute;
  content: "";
  display: block;
  width: 1.4em;
  height: 1.4em;
  top: 50%;                               /* 1/2 the buttons width */
  left: 50%;                              /* 1/2 the buttons height */
  margin-left: -0.7em;                    
  margin-top: -0.7em;                     
  border-top: 2px solid white;            /* top border makes it look like a crescent */
  border-radius: 50%;
  animation: spin 0.5s linear infinite;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}
```

Here is the javascript to add the `is-loading` class:

```js
let input = document.querySelector('#trip');
let button = document.querySelector('#submit-button');

button.addEventListener('click', (e) => {
    e.preventDefault();
    button.classList.add('is-loading');
    button.disabled = true;
    input.disabled = true;

    // submit form data to server
});
```

### Shake element to get user attention

If a user needs to enter text into an element or save their work, you can animate an element to shake and draw their attention.

Here, the keyframe shakes the button left and right by `0.4em`. There are multiple steps defined within some rulesets because the actions are the same. Notice that the odd percentages shift to the left (negative values), while the positive shift the element to the right (positive values). The values at 80% and 90% shorten the movement so it is less intense at the end, which makes it slow down more smoothly.

```css
.shake {
  animation: shake 0.7s linear;
}

@keyframes shake {
  0%,
  100% {
    transform: translateX(0);
  }
  10%,
  30%,
  50%,
  70% {
    transform: translateX(-0.4em);
  }
  20%,
  40%,
  60% {
    transform: translateX(0.4em);
  }
  80% {
    transform: translateX(0.3em);
  }
  90% {
    transform: translateX(-0.3em);
  }
}
```

Add Javascript to addd the `shake` class. This script causes the button to shake after the user stops typing for 1 second:
- use a timeout to add the `shake` class
- clear the timeout (reset it) when the button is selected or the user resumes typing
- remove the class with the `animationend` event type, so when the animation completes, the class is removed and it stops shaking. This means that you can add the class again

```js
let input = document.querySelector('#trip');
let button = document.querySelector('#submit-button');

let timeout = null;

button.addEventListener('click', (e) => {
    e.preventDefault();
    clearTimeout(timeout);                          // clear timeout when button pressed
    button.classList.add('is-loading');
    button.disabled = true;
    input.disabled = true;
});

input.addEventListener('keyup', function () {
    clearTimeout(timeout);                          // reset timeout if user types again
    timeout = setTimeout(function () {
        button.classList.add('shake');              // add .shake when user hasn't typed for 1s
    }, 1000);
});

button.addEventListener('animationend', function () {
    button.classList.remove('shake');               // remove .shake when animation completes
});
```

### Start animations based on scrolling

The `animation-timeline: scroll();` property is not widely supported, so you need to do this with JS until it is.

You might want an animation to fire when you scroll past an element in either direction. This example spins a box positioned at the top-right of the viewport when you scroll.

`scroll()` without an argument binds the animation to the closest ancestor element in the DOM that has a scrollbar. It also accepts these values:
- `nearest`: default behavior
- `root`: animates using the document root
- `self`: animates relative to the scroll position of the current element

You can combine the previous values with these values to control the scroll direction:
- `block` (default direction)
- `inline`
- `y`
- `x`

For example, `scroll(inline root)` scrolls relative to the inline scrolling of th root element

```css
body {
  margin: unset;
  height: 200lvh;
}

.indicator {
  position: fixed;
  top: 15px;
  right: 15px;
  height: 50px;
  width: 50px;
  background-color: oklch(74% 0.08 260deg);
  animation-name: spin;
  animation-timeline: scroll();
}

@keyframes spin {
  from {
    rotate: 0deg;
  }
  to {
    rotate: 360deg;
  }
}
```