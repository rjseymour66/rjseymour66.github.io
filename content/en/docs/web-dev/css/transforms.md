---
title: "Transforms"
linkTitle: "xTransforms"
weight: 190
# description:
---

The `transform` property can change or distort the shape or position of an element on the page by rotating, scaling, or skewing the element in two or three dimensions:
- most commonly used with transitions or animations
- the document flow does not change when you transform an element. No other elements can use the space occupied by the transformed element
- set margin to prevent a transformed element from overlapping other elements
- you can't transform inline elements. Either change it to an `inline-block` or `block` element, or change it to a flex or grid item.

## Basics

`transform` takes a function that specifies how the element should be transformed:

```css
transform: rotate(90deg);
```

There are [several transform functions](https://developer.mozilla.org/en-US/docs/Web/CSS/transform-function), but here are the most typical:
- `rotate`: spins the element the specified degrees on an axis
- `translate`: moves the element left, right, up, down, similar to positioning
- `scale`: shrinks or expands the element
- `skew`: distorts the shape of the element, sliding its top or bottom edge in opposite directions

```css
@layer modules {
  .card {
    max-inline-size: 300px;
    padding: 0.5em;
    margin-inline: auto;
    background-color: white;
    transform: rotate(15deg);               /* rotate right */
    transform: translate(20px, 40px);       /* move 20px right, 40px down */
    transform: scale(0.3);                  /* shrink to 30% size */
    transform: skew(20deg);                 /* skew top of image 20deg to left */
  }
}
```

### Transform origin

The transform origin is the axis of rotation, or the place where the scaling or skewing begins for the transform.
- by default, is the center.
- the origin stays locked in place and the rest of the element transforms around it (except with `translate()`)
- change with the `transform-origin` property

Specify with keywords like `top`, `right`, `bottom`, `left`, `right center`, `right bottom`, or percentages:

```css
@layer modules {
  .card {
    max-inline-size: 300px;
    padding: 0.5em;
    margin-inline: auto;
    background-color: white;
    transform: rotate(15deg);
    transform-origin: top left;
  }
}
```

This example combines `@keyframes` and animations to create a transform. You create a `@keyframes` rule that applies a `transform` property at different stages, then you set the `transform-origin` on the element itself to define the transform from the center of the element:

```css
@keyframes doScale {
  from {
    transform: scaleY(1);
  }
  50% {
    transform: scaleY(0.2);
  }
  to {
    transform: scaleY(1);
  }
}
rect {
  animation: doScale 2.2s infinite;
  transform-origin: center;
}
```

### Multiple transforms

Apply multiple transforms by separating the functions by a space. Each transform value is applied from **right to left**:
- generally, `translate()` should come after `rotate()` or `scale()`, or it might translate on the wrong axis.

```css
@layer modules {
  .card {
    ...
    transform: translate(50px, 0) rotate(15deg);
  }
}
```

### Standalone properties

You don't need the `transform` property to apply these functions, they have their own properties you can add directly to the ruleset. Use these when applying simple transforms:
- `translate`
- `rotate`
- `scale`

When you use these properties, they are applied after `transform` properties in this order:
1. `scale`
2. `rotate`
3. `translate`

```css
@layer modules {
  .card {
    ...
    translate: 50px, 0;     /* horizontal, vertical */
    rotate: 15deg;
  }
}
```

## Transforms in motion

### Scaling icons

Scale icons with transforms. If you changed the height and width, it would impact the document flow:

```css
.nav-links__icon {
    transition: scale 0.2s ease-out;
}

.nav-links a:hover > .nav-links__icon,          /* mouse hover */
.nav-links a:focus > .nav-links__icon {         /* keyboard focus */
    scale: 1.3;
}
```

### Fade in/out labels

Hover and focus pseudo-classes are used for the transforms so the menu items appear as soon as they are moused over or focused with the keyboard:

```css
@media (min-width: 480px) {
    .nav-links {
        display: block;
        padding: 1em;
        margin-block-end: 0;
    }

    .nav-links__label {
        display: inline-block;              /* inline block so you can apply tranforms */
        margin-left: 1em;
        padding-right: 1em;
        opacity: 0;                         /* hidden by default */
        translate: -1em;                    /* hidden label is moved left */
        transition: transform 8.4s cubic-bezier(0.2, 0.9, 0.3, 1.3),        /* creates a bouncing effect for label */
        opacity 0.4s linear;
    }

    .nav-links:hover .nav-links__label,     /* transition on the entire list */
    .nav-links:focus .nav-links__label {
      opacity: 1;
      translate: 0;
    }
}
```


### Staggering transitions

You can stagger the transition-delay on each element so that it appears on hover or focus in a smooth, wave-like transition:

> Do this with SCSS.

```css
.nav-links__label {
    display: inline-block;
    margin-left: 1em;
    padding-right: 1em;
    opacity: 0;
    translate: -1em;
    transition: translate 0.4s cubic-bezier(0.2, 0.9, 0.3, 1.3),
    opacity 0.4s linear;
}

.nav-links:hover .nav-links__label,
.nav-links:focus .nav-links__label {
    opacity: 1;
    translate: 0;
}

.nav-links > li:nth-child(2) .nav-links__label {
    transition-delay: 0.1s;
}
.nav-links > li:nth-child(3) .nav-links__label {
    transition-delay: 0.2s;
}
.nav-links > li:nth-child(4) .nav-links__label {
    transition-delay: 0.3s;
}
.nav-links > li:nth-child(5) .nav-links__label {
    transition-delay: 0.4s;
}
```

## Performance

You might be wondering why you would want to use a transform like `translate` or `scale`, when you can achieve nearly identical outcomes with positioning or changing the heigth/width of an object. The reason is that **transforms have a much better performance** than the alternatives.

### Rendering pipeline

> When you use transitions or animations, use `opacity` or a transform property because they get their own paint layer, which means that layout does not need to be recomputed for any affected elements.
> For details, see [CSS Triggers](https://csstriggers.com/).

Rendering is the process of translating computed styles into pixels on the screen. It has three stages:
1. **Layout**: The browser calculates how much space each elements uses on the screen because the placement of all elements in the document flow is affected by other elements.
   
   When you change the height or width of an element, its layout is recomputed. When a layout change occurs, the browser has to _reflow_ the page, which requires that it recomputes the layout of all elements that were moved as a result of the resized element.
2. **Paint**: Filling in pixels with color, but only in memory--not on the screen. The page is painted in layers. Changes in this stage are less expensive than layout changes. For example, if you change a background color, the layout does not need to be recomputed.

   An element can be painted in its own layer. This is called _hardware acceleration_ because it lets the browser send its rendering to the system's GPU, rather than the CPU used to pain the main layers.
3. **Composite**: The browser takes all painted layers and draws them in order to create the final image that will be displayed on the screen.

   If an element uses `opacity` or a transform property, it is promoted to its own layer and uses the GPU for rendering. This means that the main layer doesn't need to rerender if you change one of these animated properties.

   One-time changes don't usually impact performance, but animations can if the screen requires lots of updates. The screen updates about 60 times/sec, so animated changes should compute this fast.

### will-change

The `will-change` property can promote an element to its own paint layer. Only apply it if you see performance issues due to an animation.


This rule means that you expect the `transform` property to change on an element:
```css
.element {
    will-change: transform;
}
```

## 3D transforms

3D transforms use x, y, and z dimensions. You can do this with `translateX()`, `translateY()`, or `translateZ()`:
- z dimension moves the element closer or further from the user

These rules are all equivalent:

```css
transform: translate(15px, 50px);
transform: translateX(15px) translateY(50px);
translate: 15px, 50px;

/* add translateZ() to the end */
transform: translate(15px, 50px, 30px);
transform: translateX(15px) translateY(50px) translateZ(30px);
translate: 15px, 50px 30px;
```

### rotate()

You can also rotate elements on different axes:
- `rotateX()` rotates along a horizontal axis, like flipping it over a bar
- `rotateY()` rotates along a vertical axis, like flipping it around a pole
- `rotateZ()` is equal to `rotate()`, which rotates along the z-axis

You can specify the axis in the rule:

```css
rotate: z 20deg;
rotate: x 30deg;
```

### Perspective

Specify perspective in two ways:
- `perspective` property
- `perspective()` transform function: applies perspective to each individual element. **If you use multiple transform functions, you must use this last (first in list, directly after property name)**.
  - Does not work with `rotate`

Perspective is the distance between the screen and the 3D scene that you created in the browser. The higher the perspective, the farther away the screen, and then the more subtle the perspective:
- 3D transforms without perspective appear flat


#### On each element

The `perspective()` function adds perspective on each individual box element in a container row:

```css
.row {
  display: flex;
  gap: 4em;
  justify-content: center;
}

.box {
  box-sizing: border-box;
  width: 150px;
  padding: 60px 0;
  text-align: center;
  background-color: oklch(60% 0.12 158deg);
  transform: perspective(200px) rotateX(30deg);
}
```

#### On container element (shared perspective)

Use the `perspective` property on the container, so each box element has the same perspective--shared vanishing point:

```css
.row {
  display: flex;
  gap: 4em;
  justify-content: center;
  perspective: 200px;
}

.box {
  box-sizing: border-box;
  width: 150px;
  padding: 60px 0;
  text-align: center;
  background-color: oklch(60% 0.12 158deg);
  transform: rotateX(30deg);
}
```

### perspective-origin

The `perspective-origin` property lets you shift the vanishing point. By default, the vanishing point is straight ahead. It accepts these keywords and values:
- `top`
- `left`
- `bottom`
- `right`
- `center`
- percentage values, measured from element's top-left corner

```css
.row {
  display: flex;
  gap: 4em;
  justify-content: center;
  perspective: 200px;
  perspective-origin: left bottom;
}
```

### backface-visibility

`backface-visibility: hidden`

If you spin an element more than 90 degrees with `rotateX()` and `rotateY()`, you see the back of the element. This is like a mirror image of the original. You can hide this with the `hidden` property.

Use this to create an animation that looks like a card is flipping around: [Card flip](https://3dtransforms.desandro.com/card-flip)

### transform-style

Good for building nested elements in 3D. Not very common in real-world scenarios, but apply like this:

```css
.element {
    transform-style: presever-3d;
}
```

[Examples and deep explanation!](https://davidwalsh.name/3d-transforms)