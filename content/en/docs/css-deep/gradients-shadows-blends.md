---
title: "Gradients, shadows, and blend modes"
linkTitle: "Gradients, shadows, blends"
weight: 160
# description:
---

## Gradients

The `background` property is shorthand for these eight properties:
- `background-image`: Specifies an image from a file or color gradient
- `background-position`: Initial position of the background image
- `background-size`: How large to render the background image within the element
- `background-repeat`: Whether to tile the image to fill the element
- `background-origin`: Whether the background positioning is relative to the element's `border-box`, `padding-box`, or `content-box`
- `background-clip`: Should the background fill the `border-box`, `padding-box`, or `content-box`
- `background-attachment`: Does the background image scroll with the element or stay fixed in place in the viewport?
- `background-color`: Solid background color that renders behind any image

If you need to use more than one of these, don't use the shorthand, use the individual properties. If you do use the shorthand, it will set the properties that you set and then reset the others to the initial value.

### linear-gradient()

Use `linear-gradient()` as a value in the `background-image` property. It takes three parameters:
1. angle
   - `to right`
   - `to top`
   - `to bottom`
   - `to bottom` [ `right` | `left` ]
   - degrees (most common): When you think of degrees, think of a circle, where the top is 0 and the bottom is 180
     - `0deg` (`to top`)
     - `90deg` (`to right`)
     - `180deg` (`to bottom`)
     - `360deg` (`to top`)
   - `rad`: radians, where one full circle is 6.2832 radians.
   - `turn`: One full turn is 360deg. Use a number less than 1 to represent the direction. For example, `0.25turn`
   - `grad`: One full circle is `400grad`, and `100grad` = `90deg`.
2. starting color
3. ending color

```css
.fade {
  height: 200px;
  width: 400px;
  background-image: linear-gradient(90deg, white, blue);
}
```

### Multiple color stops

You can add more colors to the linear gradient by just listing them as arguments. Each color is called a _color stop_:

```css
.fade {
  height: 200px;
  width: 400px;
  background-image: linear-gradient(to right, turquoise, white, blue);
}
```

You can set the position of the color stops with %, px, ems, or other length values. It seems to make the best sense to use the units that you used to define the element property:

```css
.fade {
  height: 200px;
  width: 100em;
  background-image: linear-gradient(
    to right,
    turquoise 0em,
    white 50em,
    blue 90em
  );
}
```

### Stripes

If you place two color stops at the same location, the gradient changes colors instantly. This example is turquoise to 40%, then changes to white, which goes to 60%, then changes at 60% to blue, until the end:

```css
.fade {
  height: 200px;
  width: 400px;
  background-image: linear-gradient(
    90deg,
    turquoise 40%,
    white 40% 60%,
    blue 60%
  );
}
```

### Repeating gradients

[Stripes in CSS](https://css-tricks.com/stripes-css/)

Create a repeating gradient with `repeating-linear-gradient()`. It functions like `linear-gradient()`, but the pattern repeats. This example creates a stiped bar:

```css
.fade {
  height: 1em;
  width: 400px;
  background-image: repeating-linear-gradient(
    -45deg,
    oklch(60% 0.1 257deg) 0px 10px,
    oklch(40% 0.1 257deg) 10px 20px
  );
  border-radius: 0.3em;
}
```

### Color interpolation

The color space that you use has a great effect on how the gradient displays. sRGB always has a 'gray dead zone' when you work with colors opposite each other on the color wheel. This is because when each of the values go up or down, they all meet in the middle at 127.5, creating gray.

You do not have this issue with OKLCH colors--these colors do not pass through the middle of the color wheel (where grey is).

Different color spaces is not yet supported in Firefox, so include a fallback. Specify the color space with `in <color-space>`. By default, the color space is sRGB:

```css
.fade {
  height: 1em;
  width: 400px;
  background-image: linear-gradient(90deg, yellow, blue);               /* fallback */
  background-image: linear-gradient(90deg in oklch, yellow, blue);      /* color space syntax */
}
```

For examples, go to [CodePen](https://codepen.io/keithjgrant/full/GRzjGqL).

### Circular hue gradients

In polar color spaces, hue is a circular value. This means that you can go the short way around the color wheel to get to the new color, or you can go the long way, through more colors.

You can specify the hue direction with these values:
- `shorter hue`
- `longer hue`
- `increasing hue`, which increases the hue angle (clockwise)
- `decreasing hue`, which decreases the hue angle (counter-clockwise)

For example:

```css
.fade {
  height: 1em;
  width: 400px;
  background-image: linear-gradient(90deg, yellow, blue);
  background-image: linear-gradient(90deg in oklch longer hue, yellow, blue);
}
```

### Radial gradients

