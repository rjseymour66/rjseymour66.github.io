---
title: "Secrets"
# linkTitle: ""
weight: 5
# description:
---

## Vendor prefixes

Here are some helpful links to determin if you need an autoprefixer:
- [autoprefixer](https://github.com/postcss/autoprefixer)
- [-prefix-free](https://projects.verou.me/prefixfree/)

## Tips

### currentColor

Considered the first ever variable in CSS, this value is equal to the `color` property. The `color` property is inherited. This means that it applies either the `color` value for the current element or--if `color` is not defined--the initial color for that property.

### (avoid) media queries

> Now, you can use _container queries_.

Try to avoid media queries, when possible. You want to exhaust every possible way to make the website flexible without resorting to media queries. If you do use them, let the content dictate the breakpoints, not an arbitrary device.

Here are some tips to avoid them:
- Use percentages instead of fixed widths
- Use `max-width` instead of `width` so you can adapt to smaller viewports
- Always use `max-width: 100%;` for replaced content such as `img`, `object`, `video`, and `iframe`. Replaced elements are elements that are replaced by resources that the browser fetches.
- Use `column-width` instead of `column-count` so you can get one column on smaller resolutions.

### Use shorthands

Longhand properties only apply to that specific property--you don't reset all the other properties that might affect your styles. For example:

```css
body {
    background: red;            /* background will always be red */
    background-color: red;      /* other background-* props might affect the background */
}
```

You can also combine shorthand and longhand declarations to reduce the amount of edits. Here, we use the shorthand for the `url()` fallbacks, and the specific properties for specific styles:

```css
body {
    background: url(some.png) no-repeat top right,
                url(some.png) no-repeat bottom right,
                url(some.png) no-repeat bottom left;

    background-size: 2em 2em;
    background-repeat: no-repeat;
}
```

In the preceding example, you can add all the fallbacks to the `background` property, and add the individual styles to the other props. If you need to make an edit to `*-size` or `*-repeat`, then you edit only those properties.

## Backgrounds and borders

### No scroll background

This ruleset makes the background stick in the same spot, even if you scroll past it:

```css
body {
    background: url(/assets/Green_Grass.JPG);
    background-size: cover;
    background-attachment: fixed;
}
```

### Translucent borders

By default, borders extend underneath the border area, so it is hard to see transparent borders on elements with lighter backgrounds. To fix this, use the `background-clip` property to clip the background at the `padding-box`, rather than the `border-box`:

```css
.translucent-border {
  border: 20px solid hsla(0, 0%, 100%, 0.5);
  background: white;
  background-clip: padding-box;
}
```

### Multiple borders

The original fix was to use multiple `box-shadow` properties, but that is bad for multiple reasons:
- Shadows don't take up layout space, so you have to add additional margin to make the doc flow correctly
- Shadows can't capture mouse events

The fix is to use `outline`:

```css
.multiple-border {
  background: yellowgreen;
  border: 10px solid #655;
  outline: 5px solid deeppink;
}
```

You can also add `outline-offset` to add space between the border and the outline. Here, we use a negative value to make the outline look like stitching:

```css
.inner {
  background: yellowgreen;
  border: 10px solid #655;
  outline: 1px dashed deeppink;
  outline-offset: -24px;
}
```

### Flexible background positioning

When you need to position an image within a container, you often need to account for padding. For example, if you want to position an image in the bottom right of the container, you can explicitly enter the padding values in `background-position`:

```css
.background-img {
    padding: 10px;
    background: url(/assets/cancel-icon.svg) no-repeat yellowgreen;
    background-position: right 20px bottom 10px;      /* 20px from right, 10px from bottom */
}
```

This is difficult in case you want to change the padding. Instead, use the `calc()` function  to position the image from the top left corner of the container (`100%`), and subtract the padding. It's probably easier to use variables:

```css
.background-img {
    padding: 10px;
    background: url(/assets/cancel-icon.svg) no-repeat yellowgreen;
    background-position: calc(100% - 20px) calc(100% - 10px);
}
```

### Striped background

Create stripes by assigning equal or complementary (stop% + stop% = 100%) color stops to each color in `repeating- linear-gradient()`. You can control the height or width of the stripes by adding start and stop points after the stripe color.

```css
/* #5a 30px from the start of each gradient cycle */
.single-val {
    background: repeating-linear-gradient(#fb3, #58a 30px);
}

/* start #fb3 at 0 and end at 15px, start #58a at 15px and end at 30px */
.double-val {
    background: repeating-linear-gradient(#fb3 0 15px, #58a 0 30px);
}

/* horizontal */
.h-stripe {
    background: repeating-linear-gradient(
        #fb3 10% 20%,
        #58a 20% 30%,
        red 30% 40%
    );
}

/* vertical */
.v-stripes {
    background: repeating-linear-gradient(
        to right,
        #fb3 10% 20%,
        #58a 20% 30%,
        red 30% 40%
    );
}
```

If you want to simplify this, you can set a `background` color, then use `background-image` to create a semi-transparent white stripe and a completely transparent stripe. The semi-transparent white stipe is because you generally don't want to have stripes with starkly contrasting colors--you want it to be just a bit lighter:

```css
.nice-stripes {
    background: var(--background-clr);
    background-image: repeating-linear-gradient(
        30deg,
        hsla(0, 0%, 100%, 0.1) 0 15px,
        transparent 0 30px
    );
}
```

### Complex background patterns

[Patterns site](https://projects.verou.me/css3patterns/)

For repeating patterns, use one of the following strategies:
- `linear-gradient` and `background-size`. This seems more maintainable, because you specify the stops with `%`, and then you can play with the size of the pattern repeating with the `background-size` property. 
- `repeating-linear-gradient` and stops. To change the pattern, you have to set multiple values in the stops.

#### Tablecloth

You can either specify the size of the pattern with `linear-gradient` and `background-size`, or you can calculate it with the color stops and `repeating-linear-gradient`. Both of these rules create a tablecloth-like grid:

```css
/* each gradient uses 50% of the background-size */
.tablecloth {
    background: white;
    background-image: linear-gradient(
        90deg,
        rgba(200, 0, 0, 0.5) 50%,
        transparent 0
    ),
    linear-gradient(
        rgba(200, 0, 0, 0.5) 50%, 
        transparent 0);

    background-size: 30px 30px;         /* width height */
}

.tablecloth-repeating {
    background: white;
    background-image: repeating-linear-gradient(
        rgba(200, 0, 0, 0.5) 0 15px,
        transparent 15px 30px
        ),
        repeating-linear-gradient(
            90deg,
            rgba(200, 0, 0, 0.5) 0 15px,
            transparent 0px 30px
        );
}
```

#### Grid

These styles create a grid that looks like drafting paper:

```css
.grid {
    background: #58a;
    background-image: linear-gradient(white 1px, transparent 0),
        linear-gradient(90deg, white 1px, transparent 0);

    background-size: 30px 30px;
}

.grid-repeating {
    background: #58a;
    background-image: repeating-linear-gradient(
            white 0 1px,
            transparent 0 30px),
        repeating-linear-gradient(
            90deg, 
            white 0 1px, 
            transparent 0 30px);
}
```

#### Nested grid

This pattern makes it look like real drafting paper, where each square has smaller grids within. The `background-image` and `background-size` values map to each other in the order they are listed:

```css
.drafting-paper {
    background: #58a;
    background-image: 
        linear-gradient(white 2px, transparent 0),
        linear-gradient(90deg, white 2px, transparent 0),
        linear-gradient(hsla(0, 0%, 100%, 0.3) 1px, transparent 0),
        linear-gradient(90deg, hsla(0, 0%, 100%, 0.3) 1px, transparent 0);

    background-size: 75px 75px, 75px 75px,
                     15px 15px, 15px 15px;
}
```

#### Horizontal stripes

This pattern looks like lines on a piece of paper:

```css
.notebook {
    background-image: linear-gradient(white 1px, transparent 0);
    background-size: 30px 30px;
}
```

#### Polka dot

Add polka dots with `radial-gradient()`. Here, there are 2 different patterns. The first applies only one `radial-gradient`, and the second applies two for more polka dots.

The `background-position` of the second example applies different positioning for each `background-image` gradient. To make this work, the second set of values must be half the size of the `background-size`:

```css
.polka-dots {
    background: #655;
    background-image: radial-gradient(tan 30%, transparent 0);
    background-size: 30px 30px;
}

.double-dots {
    background: #655;
    background-image: 
        radial-gradient(tan 30%, transparent 0),
        radial-gradient(tan 30%, transparent 0);
    background-size: 30px 30px;
    background-position: 0 0, 15px 15px;
}
```

Changes to this require a lot of different edits, so you can create a `@mixin` in SASS:

```css
@mixin double-dot($size, $dot, $base, $accent) {
    background: $base;
    background-image: 
        radial-gradient($accent $dot, transparent 0),
        radial-gradient($accent $dot, transparent 0);
    background-size: $size $size;
    background-position: 0 0, $size/2 $size/2;
}

.container {
    @include double-dot(30px, 30%, #655, tan);
}
```

#### Checkerboard

Here is a basic checkerboard:

```css
.checker-bg {
    background: #eee;
    background-image:
        linear-gradient(45deg, #bbb 25%, transparent 0),
        linear-gradient(45deg, transparent 75%, #bbb 0),
        linear-gradient(45deg, #bbb 25%, transparent 0),
        linear-gradient(45deg, transparent 75%, #bbb 0);
    background-position: 0 0, 15px 15px, 15px 15px, 30px 30px;
    background-size: 30px 30px;
}
```

You can also simplify things and just embed and SVG:

```css
.svg {
    background: #eee
        url('data:image/svg+xml, \
        <svg xmlns="http://www.w3.org/2000/svg" \
            width="100" height="100" fill-opacity=".25"> \
            <rect x="50" width="50" height="50" /> \
            <rect y="50" width="50" height="50" /> \
        </svg>');
    background-size: 30px 30px;
}
```

### Continuous image borders

Set an image as the border with `backgound-clip` and `background-origin`. Apply two linear backgrounds to the element: the first is the background that looks like a nested div because its a linear gradient of the same colors, and the second is the image. Set the `background-clip` to `padding-box` on the first, and set it to `border-box` on the second.

Set the `background-origin` to `border-box` so it doesn't repeat within the image:

```css
.image-border {
    padding: 1em;
    border: 3em solid transparent;
    background: linear-gradient(white, white), url(/assets/background.jpg);
    background-size: cover;
    background-clip: padding-box, border-box;
    background-origin: border-box;
}
```

#### Vintage envelope

This reuses the background border idea but creates a striped border with `repeating-linear-gradient`:

```css
  padding: 1em;
  border: 1em solid transparent;
  background: linear-gradient(white, white) padding-box,
    repeating-linear-gradient(
        -45deg,
        red 0,
        red 12.5%,
        transparent 0,
        transparent 25%,
        #58a 0,
        #58a 37.5%,
        transparent 0,
        transparent 50%
      )
      0 / 5em 5em;
}
```

## Shapes

### Flexible ellipses

`border-radius` accepts values in multiple formats:
- `50% / 50%`: horizontal and vertical radii
- `a b c d`: corners beginning at top-left. If you provide less than four values, the values are doubled--`c` is applied to `c` and `d`, two values means `a` is applied to `a` and `c` and `b` is applied to `b` and `d`.
- `a b c d / v x y z`: horizonal and vertical radii for all four corners

Make sure you understand which 

```css
.half-ellipses {
    background: #fb3;
    width: 200px;
    height: 300px;
    border-radius: 50% / 100% 100% 0 0;
}

.left-ellispses {
    background: #fb3;
    width: 200px;
    height: 300px;
    border-radius: 100% 0 0 100% / 50%;
}

.quarter-ellipses {
    background: #fb3;
    width: 200px;
    height: 300px;
    border-radius: 100% 0 0 0;
}
```

### Diamond images

Do this with `clip-path`. Clipping paths lets you clip the element into the shape that we want.

This also includes a hover pseudo-class that reveals the entire image on hover:

```css
.picture {
    width: 400px;
    overflow: hidden;
}

.picture > img {
    max-width: 100%;
    clip-path: polygon(50% 0, 100% 50%, 50% 100%, 0 50%);
    transition: clip-path 300ms ease-in-out;
}

.picture > img:hover {
    clip-path: polygon(0 0, 100% 0, 100% 100%, 0 100%);
}
```

### Cutout corners

The easiest way to do this is with `clip-path`.

```css
.picture {
    padding: 20rem 30rem;
    background: #58a;
    clip-path: polygon(
        20px 0,
        calc(100% - 20px) 0,
        100% 20px,
        100% calc(100% - 20px),
        calc(100% - 20px) 100%,
        20px 100%,
        0 calc(100% - 20px),
        0 20px
    );
}
```

### Pie charts

Use `conic-gradient`:

```css
.pie {
  width: 100px;
  height: 100px;
  background: yellowgreen;
  border-radius: 50%;

  background-image: conic-gradient(
    deeppink 20%,
    #fb3 0,
    #fb3 30%,
    yellowgreen 0
  );
}
```

## Visual effects

### Shadows

Passing positive values to `box-shadow` can offset the shadow to the right and bottom. Negatives do the opposite. The last value you add is how much blur you want on the shadow. The shadow also respects the `border-radius` value.

If you want to increase the size of the shadow, add the fourth value before the color, the spread radius.

#### Duplicate offset (adjacent sides)

This creates a duplicate of the element that is offset by `50px`:

```css
.box {
    box-shadow: 50px 50px 0 20px black;
}
```

#### Shadow on one side

Use the spread radius here, and make one of the offset values `0`. Put the shadow on either side with a positive or negative spread radius:

```css
.box {
    box-shadow: 0 50px 0 -20px black;
}
```

#### Border/outline shadow

`box-shadow` works only on HTML elements. If you want to include a pseudo-element or a decoration like an outline, you have to use `filter: drop-shadow();`.

This takes an x-offset, y-offset, blur value, and color. This example creates a duplicate of the box and the outline offset:

```css
.box {
    filter: drop-shadow(50px 50px 0 black);
}
```

#### Frosted glass

If you have text that you need to display over a background, apply a backdrop filter to the container element:

```css
main {
    backdrop-filter: blur(10px);
}
```

#### Folded corner effect

Apply these styles to a div:

```css
.note {
  height: 20rem;
  width: 30rem;
  //   background: yellowgreen;

  position: relative;
  background: linear-gradient(-150deg, transparent 1.5em, yellowgreen 0);

  border-radius: 0.5em;
}

.note::before {
  content: "";
  position: absolute;
  top: 0;
  right: 0;
  background: linear-gradient(
      to left bottom,
      transparent 50%,
      rgba(0, 0, 0, 0.2) 0,
      rgba(0, 0, 0, 0.4)
    )
    100% 0 no-repeat;
  width: 1.73em;
  height: 3em;
  transform: translateY(-1.3em) rotate(-30deg);
  transform-origin: bottom right;
  border-bottom-left-radius: inherit;
  box-shadow: -0.2em 0.2em 0.3em -0.1em rgba(0, 0, 0, 0.15);
}
```

## Typography

### Custom underlines

Here are some properties that you can manipulate to manipulate text underlines:

```css
.underline {
  text-decoration: underline;
  text-decoration-style: dashed;
  text-underline-position: auto;
  text-decoration-thickness: 3px;
  text-underline-offset: 2px;
}
```

### Letterpress

This effect is supposed to give the impression of text pressed into the page. This works so long as the text is not completely black, and the background is not completely white. You need a dark text on a lighter background:

```css
.content {
    background: hsl(210, 13%, 60%);
    color: hsl(210, 13%, 30%);
    text-shadow: 0 1px 1px hsla(0, 0%, 100%, 0.8);
}
```

### Glowing text

Use the blur value in `text-shadow` with some good colors to create a glowing text effect:

```css
.glow {
    background: #203;
    color: #ffc;
    text-shadow: 0 0 0.1em, 0 0 0.3em;
}
```

Here it is as a hover state:

```css
.glow:hover {
  color: transparent;
  text-shadow: 0 0 0.1em white, 0 0 0.3em white;
}
```

### Extruding text

This effect makes the text look 3D. You create it by applying multiple `text-shadow` properties with increasing x- and y-offset values:

```css
.box {
    background: #58a;
    color: #ffc;
    text-shadow: 1px 1px black,
                 2px 2px black,
                 3px 3px black,
                 4px 4px black,
                 5px 5px black,
                 6px 6px black,
                 7px 7px black,
                 8px 8px black;
}
```


## User experience

### Interactive image comparison

