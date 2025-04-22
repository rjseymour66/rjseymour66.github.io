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
```css
background: linear-gradient(<angle>, <start-color>, <end-color>);
```
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

### Background gradients

If you use a linear gradient as the background, you need to set the `min-height` to `100vh`. Otherwise, the gradient repeats if the content is not as large as the viewport:

```css
html,
body {
  margin: 0;
  padding: 0;
  min-height: 100vh;
}
```

If you set `background-repeat: no-repeat` and the viewport is larger than the content, there will be a white space below the content.

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

Radial gradients start at a point and proceeds outward in all directions. The first color is centered in the element and transitions evenly to the element's corners in an ellipses.
- accepts color stops
- can specify color spaces
- `circle` keyword makes the radial gradient a circle, not ellipses
- specify the size of the gradient by adding length values. one value makes the gradient a circle of that size. two values specify the horizontal and vertical measurements
- `at Xpx Ypx` specifies the location of the center of the gradient with coordinates
- `repeating-radial-gradient()` repeats the pattern in concentric rings

Use `radial-gradient()` for the `background-image` property. This example has a white center and a blue outside:

```css
/* basic gradient */
.fade {
  ...
  background-image: radial-gradient(white, blue);
}

/* circle */
.fade {
  ...
  background-image: radial-gradient(circle, white, midnightblue);
}

/* change location of center of gradient */
.fade {
  ...
  background-image: radial-gradient(50px at 25% 75%, white, midnightblue);
}

/* color stops */
.fade {
  ...
  background-image: radial-gradient(
    circle,
    midnightblue 0%,
    white 75%,
    red 100%
  );
}

/* color space and hue transistion */
.fade {
  ...
  background-image: radial-gradient(circle in oklch longer hue, yellow, blue);
}

/* repeating stripes - like hypnosis */
.fade {
  ...
  background-image: repeating-radial-gradient(
    circle,
    midnightblue 0 15px,
    white 15px 30px
  );
}

/* add depth to page */
@layer theme {
  :root {
    --bg-color-1: oklch(47% 0.1 238deg);
    --bg-color-2: oklch(32% 0.08 238deg);
  }
}

@layer global {
  body {
    ...
    background-image: radial-gradient(var(--bg-color-1), var(--bg-color-2));
    ...
  }
  ...
}
```

### Conic gradient

Changes color around the ellipse in a circular direction:
- `from Xdeg` changes the starting angle. By default, the angle is 0deg (straight up). `from 90deg` starts the gradient from the right side, horizontally
- `at 30px 45px` specifies center point distance from the top-left corner. Use `px` or `%`
- `in <color-space>` specifies color space

Use `conic-gradient()` for the `background-image` property:

```css
/* basic */
.fade {
  ...
  background-image: conic-gradient(white, blue);
}

/* softer transition */
.fade {
  ...
  background-image: conic-gradient(white, blue, white);
}

/* specify starting angle */
.fade {
  ...
  background-image: conic-gradient(from 90deg, white, blue, white);
}

/* change center point location */
.fade {
  ...
  background-image: conic-gradient(at 95% 60%, white, blue, white);
}

/* color space */
.fade {
  ...
  background-image: conic-gradient(from 90deg in oklch, yellow, blue, yellow);
}
```

## Shadows

Create shadow with these two properties:
- `box-shadow`: shadow on element's box shape
- `text-shadow`: shadow for rendered text

A shadow is by default the exact size and dimensions of the element, with a matching `border-radius`. You have to specify the horizontal and vertical offset, and the color of the shadow in the property. The blur radius and spread radius are optional:
- blur radius is how much of the edges are blurred for a softer shadow
- spread radius is the size of the shadow. Positive values make it larger in all directions, negative values make it smaller
- the larger the shadow offset, the higher the element appears on the page

```css
/* properties */
.shadow {
    box-shadow: <x-offset> <y-offset> <blur-radius> <spread-radius> <color>;
    box-shadow: 2px 2px 2px 1px black;
}
```

### Gradients and shadows for depth

> Making elements look 3D and lifelike is no longer common.

Give elements a curved, 3D appearance with gradient, shadow, and the `:active` pseudo-class:
- add blur to the shadow to make it look natural
- use `:active` to remove the shadow when the button is clicked and use an `inset` shadow to put the shadow in the element. 

This makes it look like the button is pressed:

```css
/* gradient and shadow for 3D appearance */
.button {
  ...
  background-image: linear-gradient(
    to bottom,
    oklch(57% 0.11 263deg),
    oklch(40% 0.13 263deg)
  );
  box-shadow: 0.1em 0.1em 0.5em oklch(26% 0.07 263deg);
}

/* appear pressed when active */
/* first shadow has no offset and slight blur, second has vertical offset */
.button:active {
  box-shadow: inset 0 0 0.5em oklch(26% 0.07 263deg),
    inset 0 0.5em 1em rgb(0 0 0 / 0.4);
}
```

### Flat design

Emphasizes vivid, uniform colors and simple appearance:
- fewer gradients, shadows, and rounded corners
- gradients from one color to similar color
- faint shadows


This is an example of flat design:
- shadow is straight down
- rgb values
- hover and active change the button to different shades of blue
```css
.button {
  padding: 0.8em;
  border: 0;
  font-family: Helvetica, Arial, sans-serif;
  color: white;
  background-color: oklch(57% 0.11 263deg);
  box-shadow: 0em 0.2em 0.2em rgb(0 0 0 /0.15);
}

.button:hover {
  background-color: oklch(53% 0.13 263deg);
}

.button:active {
  background-color: oklch(40% 0.13 263deg);
}
```

### Hybrid design

This approach combines realistic 3D design with flat design:
- thick bottom border made with `box-shadow` with no blur for cube-like object
- `:active` shifts the button down a few pixels to mimic a pressed button with `transform: translateY()`, then reduces the box shadow by the same amount of the transform (`0.1em`)
- adds text shadow

```css
.button {
  padding: 0.8em;
  border: 0;
  font-family: Helvetica, Arial, sans-serif;
  color: white;
  border-radius: 0.5em;
  background-color: oklch(57% 0.11 263deg);
  box-shadow: 0em 0.4em oklch(40% 0.13 263deg);
  text-shadow: 1px 1px oklch(40% 0.13 263deg);
}

/*  */
.button:active {
  background-color: oklch(53% 0.13 263deg);
  transform: translateY(0.1em);
  box-shadow: 0 0.3em oklch(40% 0.13 263deg);
}
```

## Blend modes

Because the `background` property accepts any number of values, you can use multiple values to blend backgrounds. This means that you apply two different backgrounds that are fully opaque.

Some use cases:
- Tint an image with a color or gradient
- Apply texture to an image
- Lighten, darken, or reduce contrast of image so the text in front is more readable
- Overlay a text banner while letting the image show through

Use the `background-blend-mode` property to blend the different `background-*` properties. Here, it merges multiple images:
- Most `background-*` properties accept multiple values
- `background-image`: accepts multiple values
- if you only pass one value to a property, then it applies to all `background-images`
- `background-size`: accepts cover and contain
  - `cover`: resizes the image to make it fill the container
  - `contain`: makes sure the entire image is visible, even if it doesn't fill the container
- `background-blend-mode` determines how the properties blend
```css
.blend {
  min-height: 70vmin;
  background-image: url(/images/bear.jpg), url(/images/bear.jpg);
  background-size: cover;
  background-repeat: no-repeat;
  background-position: -30vw, 30vw;
  background-blend-mode: difference;
}
```

### Tinting an image

`background-blend-mode` can work with `background-color` to merge an image with a hue:
- The `background-blend-mode: luminosity;` setting takes the luminosity from the front image (`background-image`) and merges it with the hue and saturation of the back layer (`background-color`). This means that it uses the background color with the contrast and brightness of the image

```css
.blend {
  min-height: 70vmin;
  background-image: url(/images/bear.jpg);
  background-color: #148;
  background-size: cover;
  background-repeat: no-repeat;
  background-position: center;
  background-blend-mode: luminosity;
}
```

### Adding texture to an image

This example repeats the texture image and uses `soft-light`. In general, use `soft-light` for darker images and `hard-light` for lighter images:

```css
.blend {
  min-height: 70vmin;
  background-image: url("images/scratches.png"), url(/images/bear.jpg);
  background-size: 200px, cover;
  background-repeat: repeat, no-repeat;
  background-position: center;
  background-blend-mode: soft-light;
}
```


### Blend mode cheatsheet

[Interactive blend tool](https://garden.bradwoods.io/notes/css/blend-modes)

| Type of Effect      | Blend Mode    | Description                                                                                                                                                          |
| ------------------- | ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Normal**          | `normal`      | Default rendering where the element does not blend with background.                                                                                                  |
| **Darkening**       | `darken`      | Selects the **darker** color from foreground and background. Selects the darker of the two colors.                                                                   |
|                     | `multiply`    | Multiplies color values, producing a **darker result**. The lighter the front color, the more the base color shows through.                                          |
|                     | `color-burn`  | Darkens the base color by **increasing contrast**.                                                                                                                   |
| **Lightening**      | `lighten`     | Selects the **lighter** color from foreground and background.                                                                                                        |
|                     | `screen`      | The opposite of multiply, producing a **brighter result**. The darker the front color, the more the base color shows through.                                        |
|                     | `color-dodge` | Lightens the base color by **reducing contrast**. Lightens the base color, decreasing contrast.                                                                      |
| **Contrast**        | `overlay`     | Mix of multiply and screen, creating **high contrast** effects. Increases contrast by applying multiply to dark colors and screen to light colors, at half strength. |
|                     | `soft-light`  | Softly **lightens or darkens** colors depending on contrast. Similar to hard-light but uses burn/ dodge instead of multiply/ screen.                                 |
|                     | `hard-light`  | Similar to overlay but with more **intense** contrast. Greatly increases contrast. Like overlay but applies multiply or screen at full strength.                     |
| **Inversion**       | `difference`  | Subtracts colors from each other, creating a **negative effect**.                                                                                                    |
|                     | `exclusion`   | Similar to difference but with **less contrast**.                                                                                                                    |
| **Component-Based** | `hue`         | Adopts the **hue** of the foreground while keeping background saturation & luminosity.                                                                               |
|                     | `saturation`  | Uses the **saturation** of the foreground while keeping background hue & luminosity.                                                                                 |
|                     | `color`       | Applies hue and saturation from the top color onto the bottom color.                                                                                                 |
|                     | `luminosity`  | Uses the **luminosity** of the foreground while keeping background hue & saturation.                                                                                 |

### mix-blend-mode

`background-blend-mode` is limited to one element. `mix-blend-mode` can blend multiple elements:
- You can put a heading in front of an image

```html
<div class="blend">
      <h1>Ursa Major</h1>
    </div>
```

```css
.blend {
  background-image: url(/images/bear.jpg);
  background-size: cover;
  background-position: center;
  padding: 50vmin 0 1em;
}

.blend > h1 {
  margin: 0;
  font-family: Helvetica, Arial, sans-serif;
  font-size: 6rem;
  text-align: center;
  mix-blend-mode: hard-light;
  background-color: oklch(25% 0.2 25deg);
  color: #808080;
  border: 0.1em solid #ccc;
  border-width: 0.1em 0;
}
```