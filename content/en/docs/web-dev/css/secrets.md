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