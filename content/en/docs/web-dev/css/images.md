---
title: "Images and SVGs"
linkTitle: "Images"
weight: 220
# description:
---

## Images

To change the size of an image without changing its proportions, set the `width` to the size you want, and set the `height` to `auto`:

```css
img {
  height: auto;
  width: val;
}
```
Setting the height and width on an image also helps the browser calculate the image size while it loads the other content. This might prevent strange rendering behavior during page loads.

### object-position

An image might distort when you set its height and width dimensions. To prevent this, set `object-fit: cover;` to maintain the object's original aspect ratio while fitting inside the specified dimensions.

When you apply `object-fit: cover;`, the browser clips the image. To change the image's position inside the container and manimpulate what part of the image is clipped, use the `object-position` property.

### filters

[Codepen examples](https://codepen.io/michaelgearon/pen/porovxJ)  

Change how an image looks with the `filter` property, which takes a function that alters the image. If you apply filters to many images on a page, it might significantly impact page performance.

Use grayscale to make an image black and white:

```css
img {
  filter: grayscale(100%);
}
```

Full list of available functions here: https://developer.mozilla.org/en-US/docs/Web/CSS/filter#functions

### Broken image links

First, always provide a value to the `alt` attribute so users can see or hear a description of the image that should be displayed.

Use the JS `onerror` attribute to hide the image if it does not display:

```html
<img
  src="images/image.jpg"
  alt="failed to load image"
  onerror="this.style.display='none'"
/>
```

### Floating images

Apply the `float` property. Make sure to add margin to the text side of the image so there is space between the image and text:

```css
figure {
  float: left;
  margin-right: 24px;
}
```

### aspect-ratio

The aspect ratio is the proportional relationship of the image width and height, which is equal to the width / height. It isn't necessary many times, but including it reduces layout shifts when the browser has to reload. This happens because the browser knows the image size and can save room while it is being loaded.

### shape-outside

In some cases, you might have to wrap text around a square or rectangular image. You can do this with the `shape-outside` property, which accepts any of the following:
- circle or ellipse
- polygon
- alpha channel of the image (?)
- path
- box model values like `margin-box`, `content-box`, etc.
- linear-gradient

This example uses the `circle()` function to define a circle. This function takes the radius of the circle, which we set at `50%`. It also uses the `clip-path` property to clip the image into a shape:

> I did not get this to work correctly.

```css
img.compass {
  aspect-ratio: 1;
  float: right;
  shape-outside: circle(50%);
  clip-path: circle(50%);
  margin-left: 1rem;
}
```

Another option is combining `shape-outside` with a box-model value and `border-radius`. This `border-radius` clips the image and gives the image a circular shape, and `shape-outside: margin-box;` tells the text to flow around the image's margin-box size:

```css
img.compass {
  aspect-ratio: 1;
  float: right;

  margin-left: 1rem;
  border-radius: 50%;
  shape-outside: margin-box;
  border: 1px solid red;
}
```


### flow-root

When you use floating images, you can prevent them from overflowing out of their containers by applying `flow-root` to their containers:

```css
main > *,
section {
  display: flow-root;
}
```

## SVG (Scalable Vector Graphics)

SVGs scale to any size without losing quality or increasing file size, and you can modify them with CSS or JS.

Play around with it here: [SvgPathEditor](https://yqnn.github.io/svg-path-editor/).


### Links

#### Free libraries

- [Material icons](https://fonts.google.com/icons)
- [Feather Icons](https://feathericons.com/)
- [The Noun Project](https://thenounproject.com/browse/icons/term/free/)
- [Ionicons](https://ionic.io/ionicons)

#### Misc

- [Make Awesome SVG Animations with CSS](https://www.youtube.com/watch?v=UTHgr6NLeEw)
- [SVG filters](https://www.smashingmagazine.com/2015/05/why-the-svg-filter-is-awesome/)
- [d3 for data visualizations](https://d3js.org/)

### Use cases

- Icons
- Graphs/Charts
- Large, simple images
- Patterned backgrounds
- Applying effects to other elements via SVG filters

### Anti-use cases

Inefficient for storing complex images.

### Anatomy

Written in XML-based markup, so you can place it directly in HTML:
- commonly used for logos, icons, loaders
- recoloring is easy
- good for loaders bc you can add animations to them

It consist of _vectors_ on a _Cartesian plane_:
- vector: mathematical formula that defines a geometric primitive
- Cartesian coordinate system: grid-based system that definees a point by using a pair of numerical coordinates based on the points distance from two perpendicular axes


```html
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <rect x="0" y="0" width="100" height="100" fill="burlywood" />
  <path
    d="M 10 10 H 90 V 90 L 10 60"
    fill="transparent"
    stroke="black"
    stroke-width="3"
  />
  <circle cx="50" cy="50" r="20" class="svg-circle" />
  <g class="svg-text-group">
    <text x="20" y="25" rotate="10" id="hello-text">Hello!</text>
    <use href="#hello-text" x="-10" y="65" fill="white" />
  </g>
</svg>
```

- `xmlns`: XML namespace. Specifies the XML dialect that you are using so the browser can render the SVG correctly.
- `viewBox`: Defines the bounds of the SVG---the positions of different points of the SVG elements in the following order:

  - `min-x`, `min-y`, `width`, and `height`. `min-x` and `min-y` are the distance from the top-left corner of the container.

  Similar to a "pan and zoom" tool.
  
  Also defines the aspect ratio (ratio of width to height) and the origin (the image's origin of motion for an animation).

- `class`, `id`: Same funciton as in CSS or JS.
- `<circle>`, `<rect>`, `<path>`, `<text>`, `<circle>`, `<ellipse>`, `<line>`, `<polyline>`, `<polygon>`, [other elements](https://developer.mozilla.org/en-US/docs/Web/SVG/Element): Defined by the SVG namespace that let you build SVGs.
  
  Paths are for irregular shapes that you would see if you created an icon in Inkscape.

  CSS tricks [SVG Elements by Category](https://css-tricks.com/svg-properties-and-css/#svg-elements-by-category)

- `fill` and `stroke`: Properties that you can edit with CSS and JS.

  [Properties shared between CSS and SVG](https://css-tricks.com/svg-properties-and-css/#shared-properties)

  [SVG CSS Properties](https://css-tricks.com/svg-properties-and-css/#svg-css-properties)

### Styling SVGs

You can style SVGs with CSS, but it depends on how you add the SVG to your site.
1. Link with an `<img>` tag or with `background-image: url(./path/to/.svg);`
   - You cannot edit these SVGs.
   - you can use CSS to change the size of the svg, but you can't change the shape
2. Embed it in the HTML file, which means to place the XML directly in the HTML file.
   - Makes HTML harder to read
   - Page is less cacheable
   - Slows HTML loading
   - Not good separation-of-concerns

You can minimize these drawbacks with React or Webpack.


You can also style SVGs with presentation attributes, such as `<rect fill="blue">`:

```css
rect:nth-child(1)  { fill: #1a9f8c }          /* element selector */
.svg-class:nth-child(1)  { fill: #1a9f8c }    /* class selector */
```

For more information, see the [MDN docs](https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Attribute).

### Floating SVGs

You can float an SVG and wrap text around it by importing it with the `shape-outside` property and the `url()` function.

> When you use `url()` with `shape-outside`, its important that the SVG resource is fetched by the browser, not read from the file system. Reading from the file system violates CORS policy.
>
> Note that the resource being fetched is the SVG path, not the actual image. You still need to have the image bundled with your website resources (HTML, CSS, etc.) so it renders in the browser.

In addition to fetching the SVG shape, you have to add shape-margin to move the text away from the SVG path. We also add some `margin-right` (the image is floated left) so the text appears to flow more naturally around the image. Make sure your standard `margin*` value is less than or equal to your `shape-margin`, or the text won't wrap uniformly:

```css
img.dog {
  aspect-ratio: 126/161;
  float: left;
  shape-outside: url(https://raw.githubusercontent.com/michaelgearon/Tiny-CSS-Projects/main/chapter-07/before/img/dog.svg);
  shape-margin: 1em;
  margin-right: 1em;
}
```