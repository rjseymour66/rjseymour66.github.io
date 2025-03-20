---
title: "Masks, shapes, and clipping"
linkTitle: "Masks, shapes, clipping"
weight: 170
# description:
---

## Filters

`filter` property to apply blur, color shift, or desaturation to an element:

- Use multiple filters at the same time by separating them with a space. This applies filter effects from left to right
- Use filters subtly, like whenn a user hovers over an image

### Types of filters

| Function example                  | Description                                                    | Parameter(s)                                                                                                                                                                                                                                               |
| :-------------------------------- | :------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| blur(10px)                        | Applies a Gaussian blur                                        | The higher the length, the stronger the blur effect.                                                                                                                                                                                                       |
| brightness(150%)                  | Increases or decreases the brightness                          | A value lower than 100% reduces the brightness; a value greater than 100% increases it.                                                                                                                                                                    |
| contrast(150%)                    | Increases or decreases the contrast                            | A value lower than 100% reduces contrast (fades the image); a value greater than 100% increases it.                                                                                                                                                        |
| drop-shadow(10px 10px 15px black) | Adds a drop shadow, similar to the drop-shadow property        | The first two parameters are lengths indicating the x- and y-offsets, respectively. The third is a blur amount (optional). The fourth is a color (optional). Unlike the drop-shadow property, the spread radius value and inset keyword are not supported. |
| grayscale(50%)                    | Desaturates the color                                          | A value between 0% and 100%, which produces a fully grayscale image.                                                                                                                                                                                       |
| hue-rotate(30deg)                 | Shifts the hue value of every pixel                            | Any angle indicating how far around the color wheel to shift colors.                                                                                                                                                                                       |
| invert(100%)                      | Inverts the colors                                             | A value of 100% completely inverts the colors. A value between 0% and 100% applies the effect at the indicated strength.                                                                                                                                   |
| opacity(50%)                      | Makes the element transparent, similar to the opacity property | A value of 0% makes the element completely transparent; 100% is fully opaque.                                                                                                                                                                              |
| saturate(150%)                    | Increases or decreases the color saturation                    | Values higher than 100% increase the image color saturation; values lower than 100% desaturate the image. saturate(25%) is equivalent to grayscale(75%).                                                                                                   |
| sepia(100%)                       | Replaces color with a sepia-tone effect                        | A value between 0% and 100%, controlling the strength of the effect.                                                                                                                                                                                       |

### blur()

Takes a single parameter that is a length that indicates the strength of the blur--how many pixels will blend together:

```css
/* basic blur */
img {
  filter: blur(5px);
}

/* blur all images except img you are hovering over */
.gallery:hover img {                  /* blur all images in div */
  filter: blur(3px) grayscale(50%);
}

.gallery img:hover {                  /* no blur on img hovering over */
  filter: none;
}

.gallery img {                        /* blur transition */
  transition: filter 1s;
}
```

### Backdrop filter

Applies a filter behind some content, rather than the content directly
- takes same functions as `filter`
- useful when placing text in front of a background image, notification messages, application control buttons

This example uses a background image, and then a blurred background so the text displays properly

```html
<div class="box">
  <div class="box__content">
    <h1>Common Kingfisher</h1>
    <p>
      The Common Kingfisher is a bird native found in Europe, sia, and north
      Africa. It has bright blue plumage with an orange belly.
    </p>
  </div>
</div>
```

```css
/* background image */
.box {
  padding: 30px;
  background-image: url(/images/bird.jpg);
  background-size: cover;
  background-position: center 30%;
}

/*  */
.box__content {
  max-inline-size: 500px;
  margin-inline: auto;
  padding: 50px 30px 70px;
  border-radius: 10px;
  background-color: oklch(100% none none / 0.3);
  -webkit-backdrop-filter: blur(10px);
  backdrop-filter: blur(10px);
}
```


## Masks

All elements are rendered as rectangles, but using a mask lets you use an image as a reference for the browser to hide or show portions of an element. For example, if you have a star image, you can use it as a mask over any other image to either hide or show a star-shaped portion of the background image:
- `mask-image` is similar to `background-image`
- `mask-size` is similar to `background-size` and accepts the same values
- masks are invisible, so if yourre working with them it might help to use `background-image` until you are sure of the styles you want to apply, then change it to `mask-image`

This example applies a mask to an image:

```css
.mask {
  width: 300px;
  min-height: 300px;
  background-image: url(/images/bird.jpg);          /* underlying image */
  background-size: cover;
  background-position: 100%;
  -webkit-mask-image: url(images/star-mask.png);
  -webkit-mask-size: 100% 100%;
  mask-image: url(images/star-mask.png);            /* overlying mask */
  mask-size: 100% 100%;
}
```

### Mask properties

mask
: A shorthand property for all of the following properties.

mask-image
: Specifies the image to use for the mask. This may be an image file referenced via `url()`, a gradient function, or the ID selector of an SVG `<mask>` element defined on the page (for example, `url(#my-svg-mask>)`). Multiple masks can be provided, each separated by a comma.

mask-position
: Specifies x and y values of the mask position relative to the top-left corner `mask-origin`. The keyword values `top`, `right`, `bottom`, `left`, and `center` are also supported.

mask-size
: Sets the size of the image mask. This can be x and y values as a length or percentage or one of the keywords `cover`, `contain`, or `auto`. If you use the mask shorthand property, this must follow the `mask-position`, separated by a slash, such as `mask: url(/mask.png) 10px 10px / 90%`.

mask-repeat
: Specifies whether the mask image should tile for a repeating effect. The initial value is repeat. Other values are `no-repeat`, `repeat-x`, `repeat-y`, `space`, and `round`. `space` and `round` will repeat the mask as much as the element will allow without clipping a portion of it off; `space` adds a gap between each copy of the mask, while `round` allows the repeated mask to stretch to fill the element.

mask-origin
: Specifies the origin used for positioning the mask. The initial value is `content-box`, but you can change this to `padding-box` or `border-box`.

mask-clip
: Specifies the area of the element that will be affected by the mask. The initial value is `border-box`, but you can change this to `padding-box` or `content-box`.

mask-mode
: Specifies whether to use the alpha channel or the luminance of the mask image to define the mask. This can be set to `alpha`, `luminance`, or `match-source`. The initial value is `match-source`, which uses an alpha channel for image masks but a luminance channel for SVG sources referenced via an ID selector (see `mask-image`).

mask-composite
: If multiple mask images are defined, this specifies how they are to be combined. Supported values are `add` (the initial value), `subtract`, `intersect`, and `exclude`. See https://mng.bz/qOWr for a closer look at this.

### Gradient masks

Apply gradients to `mask-image` like you would `background-image`. These examples apply different gradient types with `mask-image`:

```css
/* transparent at top, black at bottom */
.mask {
  width: 300px;
  min-height: 300px;
  background-image: url(/images/bird.jpg);
  background-size: cover;
  background-position: 100%;
  -webkit-mask-image: linear-gradient(to bottom, transparent, black);
  mask-image: linear-gradient(to bottom, transparent, black);
}

/* radial for vignette effect */
.mask {
  width: 300px;
  min-height: 300px;
  background-image: url(/images/bird.jpg);
  background-size: cover;
  background-position: 100%;
  mask-image: radial-gradient(black 40%, transparent 70%);
}

/* stripes */
.mask {
  width: 300px;
  min-height: 300px;
  background-image: url(/images/bird.jpg);
  background-size: cover;
  background-position: 100%;
  mask-image: repeating-linear-gradient(
    45deg,
    transparent 0px 30px,
    black 30px 60px
  );
}
```

### Luminance masks

`mask-mode` property lets you use luminance with masks.

Masks are applied as transparent or opaque, but you can use luminance--black and white--when applying masks:
- Helpful when you don't have a transparent mask available
- `mask-mode` is `alpha` by default, accepts `luminance`

```css
.mask {
  width: 300px;
  min-height: 300px;
  background-image: url(/images/bird.jpg);
  background-size: cover;
  background-position: 100%;
  -webkit-mask-image: linear-gradient(to bottom, black, white);
  mask-image: linear-gradient(to bottom, black, white);
  mask-mode: luminance;
}
```

## Clipping paths

Use the `clip-path` property.

Method to selectively hide a portion of an element. Its like a mask, but instead of using an image to mask portions of the image, you use a mathematical formula.


### Firefox dev tool

Firefox has a helpful clip-path tool if you need to test polygonal clipping:
1. Right-click an element and select **Inspect**.
2. In the CSS rules pane, find the `clip-path` property, and select the polygon before the value.
3. Click and drag it in the viewport. Firefox updates the coordinate values in the CSS rules pane.

### clip-path

[CSS Tricks article](https://css-tricks.com/cut-corners-using-css-mask-and-clip-path-properties/)

Sets the clipping path:
- setting it to any value other than `none` creates a new stacking context
- usually you want to set the clipping size to the full width and height of the element you are clipping


clip-path types:
- `circle()` function accepts a length of the radius, and x y coords to move its center
- `ellipse()` function creates an oval shape: `ellipse(<horizontal-radius> <vertical-radius>);`
- `polygon()` function accetps any number of x and y coords, separated by a comma
  - `polygon(50% 0%, 100% 100%, 0% 100%);` creates a triangle (top, bottom-right, bottom-left)

```css
/* defines the radius in px */
.clipped {
  clip-path: circle(398px);
}

/* circle edges reach edges farthest to the center  */
.clipped {
  clip-path: circle(farthest-side);
}

/* circle edges reach edges closest to the center */
.clipped {
  clip-path: circle(closest-side);
}

/* 229px circle, center moved to x=337px y=293px */
.clipped {
  clip-path: circle(229px at 337px 293px);
}

/* oval shape, changed center */
.clipped {
  clip-path: ellipse(300px 170px at 360px 290px);
}

/* polygon clip */
.clipped {
  clip-path: polygon(380px 50px, 650px 210px, 520px 500px, 20px 360px);
}
```

### clip-path types

inset()
: Clips to a rectangle within the element, inset from its edges by the specified amount. Providing one value, such as inset(15px), places the clip path that far in on each side of the element. Providing two, three, or four values allows you to control the various sides independently, similar to the way the padding property can accept different values for each side.

path()
: Clips to a given SVG path string for example, path("M68,312C17,239 117,63 223,62C328,61 409,276 370,319C330,365 118,384 68,312Z"). SVG paths are generally too cryptic to edit by hand; you will need to export them from vector-editing software.

margin-box
: Clips to the element’s margin box.

border-box
: Clips to the element’s border box.

padding-box
: Clips to the element’s padding box.

content-box
: Clips to the element’s content box.


## Floats and shapes

All elements are rectangles, but the `shape-outside` property lets you define a more complex shape that inline content can wrap around:
- **requires that the element is floated**.
- usually used with a mask, clip path, or border radius that follows the same contours

### Floating

Use `float: left;` or `float: right;`.

Floating pulls an element to one side and lets inline content flow around its margin box:
- removes the element from the normal document flow so the content can wrap
- floated element's position depends on where it is in the HTML
- if you float multiple elements in the same direction, they sit beside each other
- wrap `img` in a div or a normal element, because they are sometimes difficult to work with than standard elements when wrapping content
  - **floating elements do not add height to a parent container. If you float an element in a div, the div has a height of 0**


#### Clear fix

`clear:both` on the `::after` pseudo-element forces an element to appear beneath a floated element. This extends the height of the container element to the bottom of its floated child element:

```css
.float-parent::after {
  display: block;
  content: "";
  clear: both;
}
```

### shape-outside

Defines the shape of the text that wraps around a floated element. Accepts most of the same values as `clip-path`--it does not accept the `path()` function for SVGs:
- this means that you can apply the same value for `shape-outside` as you did to the element's clip path
- might help to set it as a custom property
- if the floated element has a border radius, try `margin-box`, `border-box`, `padding-box`, or `content-box`
- `shape-margin` applies margin to the floated element, but it does not extend outside its `margin-box`

```css
/* custom property */
.img-div {
  float: left;
  margin-right: 15px;
  --shape: circle(50%);
  clip-path: var(--shape);
  shape-outside: var(--shape);
}

.img-div > img {
  display: block;
  height: 350px;
  width: 350px;
  object-fit: cover;
}

/* border-radius */
.img-div {
  float: left;
  margin-right: 15px;
  shape-outside: margin-box;      /* wrap around margin-box on element */
  border-radius: 50%;             /* set border radius for text */
}

.img-div > img {
  display: block;
  height: 350px;
  width: 350px;
  object-fit: cover;
  border-radius: 50%;             /* set border radius for image */
}
```

### Shapes and gradient masks

You can apply properties like `linear-gradient()` to the text and image:
- text shape is defined at where the gradient either reaches the end, or 0% opacity
- `shape-image-threshold` property can change where the edge of the image is so you can overlap the image with text

```css
/* linear-gradient property */
.img-div {
  float: left;
  --gradent: linear-gradient(65deg, black 0 375px, transparent 375px);
  mask: var(--gradent);
  shape-outside: var(--gradent);
  shape-margin: 15px;
}

.img-div > img {
  display: block;
  height: 350px;
  object-position: -70px 0;     /* manually position image in element */
}

/* shape-image-threshold */
.img-div {
  float: left;
  --gradient: linear-gradient(65deg, black 0%, transparent 75%);
  mask: var(--gradient);
  shape-outside: var(--gradient);
  shape-margin: 15px;
  shape-image-threshold: 0.15;
}

.img-div > img {
  display: block;
  height: 350px;
  object-position: -70px 0;
}
```