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

Works with `object-fit`, which tells the browser to calculate the optimum size of the image based on the dimensions provided so that it does not distort. `object-position` changes where the image is positioned inside the container to manipulate which part of the image is clipped.

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

  - `min-x`, `min-y`, `width`, and `height`

  Also defines the aspect ratio (ratio of width to height) and the origin (the image's origin of motion for an animation).

- `class`, `id`: Same funciton as in CSS or JS.
- `<circle>`, `<rect>`, `<path>`, `<text>`, [other elements](https://developer.mozilla.org/en-US/docs/Web/SVG/Element): Defined by the SVG namespace that let you build SVGs.

  CSS tricks [SVG Elements by Category](https://css-tricks.com/svg-properties-and-css/#svg-elements-by-category)

- `fill` and `stroke`: Properties that you can edit with CSS and JS.

  [Properties shared between CSS and SVG](https://css-tricks.com/svg-properties-and-css/#shared-properties)

  [SVG CSS Properties](https://css-tricks.com/svg-properties-and-css/#svg-css-properties)

### Embedding SVGs

1. Link with an `<img>` tag or with `background-image: url(./path/to/.svg);`
   - You cannot edit these SVGs.
2. Embed it in the HTML file.
   - Makes HTML harder to read
   - Page is less cacheable
   - Slows HTML loading

You can minimize these drawbacks with React or Webpack.